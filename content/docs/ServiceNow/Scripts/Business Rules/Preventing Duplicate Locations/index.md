---
title: "Preventing Duplicate Locations"
---

## Overview

This document outlines a solution to enforce data integrity on the **Location** (`cmn_location`) table by preventing the creation of duplicate records. 

The logic ensures that no two locations can share the same **Name** and **Parent** combination. This is a critical validation because ServiceNow's `full_name` field (e.g., *USA / California / San Diego*) is calculated **after** the record is inserted, making standard uniqueness checks on that field unreliable during the initial save.

## Problem Statement

In many ServiceNow environments, users accidentally create duplicate location records because:
- The `full_name` field is not yet populated during a `before` Business Rule.
- Global organizations often have multiple rooms or sub-sites with the same short name (e.g., "Conference Room A"), but they reside under different parent locations.
- Standard "Unique" field indexes on the `name` column alone are too restrictive, while checks on `full_name` fail to trigger accurately on new inserts.

The goal is to provide **real time feedback** to the user and abort the database action if a duplicate hierarchy is detected.

## Why Validate Name + Parent (Not Full Name)?

The `full_name` field is a "Calculated" field or populated via an `After` Business Rule process. If you run a script `before` the insert, `current.full_name` is effectively empty. By validating the **Name** and **Parent** fields specifically, we bypass this limitation and check the data that *will* eventually form the full name, ensuring integrity before the record ever hits the database. Additionally the us of the 'unique' index on the `full_name` field is not possible due to the system creating the location record and then calculating the `full_name` after the fact, which would allow duplicates and then throw an error after the fact, which is not ideal for user experience nor data integrity.

---

### Business Rule: Prevent Duplicate Location

**Table:** Location (`cmn_location`)  
**When:** Before (Insert and Update)

```javascript {linenos=table,linenostart=1,filename="Prevent Duplicate Location.js"}
(function executeRule(current, previous /*null when async*/) {

    // Initialize GlideRecord to check for existing records with the same naming structure
    var locGR = new GlideRecord('cmn_location');
    locGR.addQuery('name', current.name);
    
    // Validating against the parent sys_id
    locGR.addQuery('parent', current.parent); 
    
    // Ensure we aren't flagging the record we are currently updating
    locGR.addQuery('sys_id', '!=', current.getUniqueValue());
    
    locGR.query();

    if (locGR.next()) {
        // Construct a helpful message for the end user
        vvar parentDisplay = current.parent.getDisplayValue() || "No Parent";
        var errorMsg = gs.getMessage(
            "A location named '{0}' already exists under the parent '{1}'. Please use a unique combination.", 
            [current.name.toString(), parentDisplay]
        );
        
        gs.addErrorMessage(errorMsg);
        
        // Stop the record from being created or updated
        current.setAbortAction(true);
    }

})(current, previous);
```