---
title: "Combine Group Membership for a Qualifier"
---

## Overview

This pattern splits a common reference qualifier need into two reusable pieces:

1. A classless **Script Include**, `CheckGroupMembership`, that exposes one on-demand function, `getGroupMembersRefQual(group_id)`. Given a group's sys_id, it queries `sys_user_grmember` for active members of that group and returns a ready-to-use `sys_idIN...` encoded query string of their user sys_ids.
2. A **Reference Qualifier** script — used here on a "Manager" variable in a catalog item - that calls `getGroupMembersRefQual()` twice (once per group), then merges and de-duplicates the two result sets before returning a single combined `sys_idIN...` string.

Because the membership lookup lives in the Script Include, any other reference qualifier that needs "members of group X" (or "members of group X or Y") can reuse the same function instead of re-writing the `sys_user_grmember` query each time.

## Problem Statement

A reference field sometimes needs to be scoped to more than one group's membership. For example, showing both "Approving Managers" and "Non-Approving Managers" in a single Manager picker. Combining two independent group lookups into one qualifier means running each query, merging the results, and removing duplicates (since a user could belong to both groups) before returning the final `sys_idIN` list.

{{< tabs >}}

  {{< tab name="Script Include" >}}

```javascript {linenos=table,linenostart=1,filename="CheckGroupMembership.js"}
// Create a new class called CheckGroupMembership
var CheckGroupMembership = Class.create(); 
CheckGroupMembership.prototype = {
	initialize: function() {},

	// Function to get the reference qualifier for group members
    getGroupMembersRefQual: function(group_id){
		// Check if the group_id is null or empty
        if(gs.nil(group_id)){
			return 'sys_id=NULL';
		}
		// Create array to store the user IDs of group members
        var memberIDs = [];
        // Create a new GlideRecord object for the 'sys_user_grmember' table
		var grMember = new GlideRecord('sys_user_grmember');
		grMember.addQuery('group', group_id);
		grMember.addQuery('user.active', true);
		grMember.query();
		
        // Iterate through the results
        while(grMember.next()){
			// Add the user ID to the memberIDs array
            memberIDs.push(grMember.getValue('user'));
		}
		// Return a reference qualifier string that matches the user IDs in the memberIDs array
        return 'sys_idIN' + memberIDs.join(',');
	},

	type: 'CheckGroupMembership'
};
```

{{< /tab >}}

  {{< tab name="Reference Qualifier Usage" >}}

```javascript {linenos=table,linenostart=1,filename="Login ID Manager"}
/**
 * Script Name: Get Group Members Reference Qualifier
 * Description: This script retrieves the members of the All Approving managers and All non-Approving managers groups and combines them into a single reference qualifier string.
 * 
 * Usage: This script is utilized within the ServiceNow instance in Manager Variable
 *      Example record:
 *         Name: Manager
 *         Table: item_option_new
 * 
 * Context: This is used in a reference field to only show users in a specific group.
 * 
 * Author: Konner Lester
 * Date Created: 5/7/2025
 * Last Modified: 5/7/2025
 * 
 */

javascript: 
var group1 = new global.CheckGroupMembership().getGroupMembersRefQual('70cc1f671bfb1110976764e9bc4bcbff');
var group2 = new global.CheckGroupMembership().getGroupMembersRefQual('f6a4cddf1b704610efa3326ecc4bcbae');

// Combine the results, avoiding duplicates
var combinedGroups = group1.split(',').concat(group2.split(',')).filter(function(item, pos, self) {
    return self.indexOf(item) == pos;
}).join(',');

// Return the final query string
'sys_idIN' + combinedGroups;
```

{{< /tab >}}

{{< /tabs >}}

{{< callout type="info" >}}
Note that `group1` and `group2` each already start with the `sys_idIN` prefix, so `.split(',')` on the first element leaves `sys_idIN<firstUserId>` intact. The de-duplication and `join()` treat it like any other array entry, and the final `'sys_idIN' + combinedGroups` prepends the prefix a second time. This works out because `sys_idINsys_idIN<id>,<id>...` still parses fine as the first `IN` value, but if you reuse this pattern elsewhere, it's cleaner to strip the `sys_idIN` prefix before merging.
{{< /callout >}}
