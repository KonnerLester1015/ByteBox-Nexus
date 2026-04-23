---
title: "RITM Approval History"
---

## Overview

This Email Script generates a formatted approval history table for email notifications sent to stakeholders regarding Requested Item (RITM) approvals.

The script iterates through approval records linked to an RITM and displays each approver's name, decision (approved/rejected/requested), and action timestamp in chronological order.

### Use Case

When an RITM is approved, rejected, or receives new approvers, stakeholders receive a notification email. This script populates the email body with a complete audit trail of all approval activity, ensuring recipients have full visibility into the approval workflow without logging into ServiceNow.

To actual use the script in an email notification you can use this `${mail_script:scriptName}` syntax in the email template, where `scriptName` is the name of this Email Script record.

## Implementation

### Overview

The script queries the `sysapproval_approver` table for all approvals related to the current RITM record. It filters for approvals in approved, rejected, and requested states and sorts chronologically, and formats each entry for email display.

### Parameters

- `current` — the RITM record (contains `sys_id`)
- `template` — email template object used to render content
- `email` — email object
- `email_action` — action trigger information
- `event` — event context

### Script

```javascript {filename="RITM_Approval_History.js", linenos=table}
(function runMailScript(current, template, email, email_action, event) {

    var limit = 20;
    var approvers = new GlideRecord('sysapproval_approver');
    
    // Querying approvals linked to this RITM
    var qc = approvers.addQuery('sysapproval', current.sys_id);
    qc.addOrCondition('document_id', current.sys_id);
    
    approvers.addNotNullQuery('approver');
    approvers.addQuery('state', 'IN', 'approved,rejected,requested');
    
    // Sort by the date/time the record was updated
    // Use 'sys_updated_on' for the timestamp
    approvers.orderBy('sys_updated_on'); 
    
    approvers.setLimit(limit);
    approvers.query();

    if (approvers.hasNext()) {
        while (approvers.next()) {
            var name = approvers.approver.getDisplayValue();
            var state = approvers.state.getDisplayValue();
            var date = approvers.sys_updated_on.getDisplayValue();
            
            // Format: Approver Name - Approved (2026-04-02 12:00:00)
            template.print(name + ' - ' + state + ' (' + date + ')<br/>');
        }
        
        if (approvers.getRowCount() > limit) {
            template.print('Additional approvers are available in the system record.<br/>');
        }
    } else {
        template.print('No approvers found.<br/>');
    }

})(current, template, email, email_action, event);
```

## How It Works

### Step 1: Initialize Query

A `GlideRecord` query on `sysapproval_approver` establishes the baseline for fetching approval records related to the current RITM.

```javascript {linenos=table,linenostart=4}
var approvers = new GlideRecord('sysapproval_approver');
    
// Querying approvals linked to this RITM
var qc = approvers.addQuery('sysapproval', current.sys_id);
qc.addOrCondition('document_id', current.sys_id);
```

This handles multiple approval linkage patterns (direct `sysapproval` reference or document reference).

### Step 2: Filter For Valid Approvals

The query filters for:

- Approvals with a non-null `approver` (ensures approver is assigned)
- States of `approved`, `rejected`, or `requested`

```javascript {linenos=table,linenostart=10}
approvers.addNotNullQuery('approver');
approvers.addQuery('state', 'IN', 'approved,rejected,requested');
```

### Step 3: Sort By Timestamp

Results are sorted chronologically by `sys_updated_on`, presenting the approval history in the order decisions were made.

```javascript {linenos=table,linenostart=15}
approvers.orderBy('sys_updated_on');
```

### Step 4: Apply Limit

A configurable limit (default 20) prevents extremely long emails for items with many approvers. An overflow message notifies the recipient if additional approvals exist.

```javascript {linenos=table,linenostart=17}
approvers.setLimit(limit);
```

### Step 5: Render Output

For each approval, the script prints:

```javascript {linenos=table,linenostart=21}
while (approvers.next()) {
    var name = approvers.approver.getDisplayValue();
    var state = approvers.state.getDisplayValue();
    var date = approvers.sys_updated_on.getDisplayValue();
    
    // Format: Approver Name - Approved (2026-04-02 12:00:00)
    template.print(name + ' - ' + state + ' (' + date + ')<br/>');
}
```

## Appendix

- [ServiceNow Email Scripts Documentation](https://developer.servicenow.com/dev.do#!/learn/courses/australia/app_store_learnv2_automatingapps_australia_automating_application_logic/app_store_learnv2_automatingapps_australia_notifications/app_store_learnv2_automatingapps_australia_notification_email_scripts)
