---
title: "Auto-Approve via Delegate or Duplicate"
---

## Overview

These two **Business Rules** both run on insert/update of `sysapproval_approver` and automatically close out an approval record as `approved` when a real approval decision has already effectively been made elsewhere, so the same approver isn't pestered twice or blocked on a delegate who already acted for them.

1. **Delegate Approval Check** — checks the `sys_user_delegate` table for an active delegate relationship for the current approver. If that delegate has already approved the same document (matched on `document_id` or `sysapproval`), the current approval record is auto-approved with a work note explaining it was "No Longer Required" because the delegate already acted.
2. **Duplicate Approval Requests** — checks whether the *same* approver already has an `approved` record for the same document. If so, this (presumably duplicate) approval record is auto-approved with a work note explaining it was "Not Longer Required" due to the earlier approval on the same record by the same user.

## Problem Statement

Approval workflows can generate more than one `sysapproval_approver` record pointing at the same underlying document for the same person — for example, if a delegate relationship is set up mid-flight, or if a workflow re-triggers approvals. Without these checks, an approver (or their delegate) could be asked to approve the same thing multiple times, or a pending approval could sit unresolved even though an equivalent approval already happened. Both rules resolve this by watching for a matching prior `approved` record and short-circuiting the new one to `approved` automatically.

{{< tabs >}}

  {{< tab name="Delegate Approval Check" >}}

```javascript {linenos=table,linenostart=1,filename="Delegate Approval Check.js"}
/**
 * Script Name: Delegate Approval Check
 * Description: This script checks if a delegate has previously approved the same record for the out-of-office user. 
 *              If a previous approval by a delegate is found, the approval is marked as 'approved', and a work note is added.
 * 
 * Usage: This script is utilized within the ServiceNow instance as a Business Rule. It is triggered when an approval request is created or updated.
 *      Example record:
 *         Name: Delegate Approval Check
 *         Table: sysapproval_approver 
 * 
 * Context: This script is intended to be used in the context of ServiceNow's approval process. It is designed to prevent redundant approvals when a delegate has already approved for the user.
 * 
 * Author: Konner Lester
 * Date Created: 05/07/2025
 * Last Modified: 05/07/2025
 * 
 */

// Execute the check
checkDelegateApproval();

function checkDelegateApproval() {
    // Ensure there is a record linked to the approval
    if (current.document_id || current.sysapproval) {
        
        // Query the sys_user_delegate table to find the delegate relationships
        var delegateGr = new GlideRecord('sys_user_delegate');
        delegateGr.addQuery('user', current.approver);
        delegateGr.addEncodedQuery('ends>=javascript:gs.beginningOfToday()');
        delegateGr.query();

        // If there is a delegate relationship, check for prior approvals
        while (delegateGr.next()) {
            var delegateUser = delegateGr.delegate;

            var approvalGr = new GlideRecord('sysapproval_approver');
            
            if (!current.document_id.nil()) {
                approvalGr.addQuery('document_id', current.document_id);
            } else if (!current.sysapproval.nil()) {
                approvalGr.addQuery('sysapproval', current.sysapproval);
            }
            
            approvalGr.addQuery('approver', delegateUser); // Look for approvals from the delegate
            approvalGr.addQuery('state', 'approved');
            approvalGr.query();
            
            if (approvalGr.next()) {
                // If a previous approval is found by the delegate, set this approval to 'approved'
                current.state = 'approved';
                current.u_work_notes = "Approval marked by system as 'No Longer Required' because the delegate (" + delegateUser.getDisplayValue() + ") approved in an earlier stage.";
                break; // Exit loop if we find a valid delegate approval
            }
        }
    }
}
```

{{< /tab >}}

  {{< tab name="Duplicate Approval Requests" >}}

```javascript {linenos=table,linenostart=1,filename="Duplicate Approval Requests.js"}
/**
 * Script Name: Duplicate Approval Requests
 * Description: This script checks for duplicate approval requests for the same record by the same user. If a duplicate is found, it marks the current approval request as 'approved' and adds a work note indicating that the approval was marked as 'Not Longer Required' due to a previous approval.
 * 
 * Usage: This script is utilized within the ServiceNow instance as a Business Rule. It is triggered when an approval request is created or updated.
 *      Example record:
 *         Name: Duplicate Approval Requests
 *         Table: sys_script 
 * 
 * Context: This script is intended to be used in the context of ServiceNow's approval process. It is designed to prevent duplicate approval requests from being sent for the same record by the same user. 
 * 
 * Author: Konner Lester
 * Date Created: 12/22/2022 
 * Last Modified: 04/12/2023 
 * 
 */

//Check to see if user has previously approved
approveDuplicateApproval();

function approveDuplicateApproval(){
    //Must have link to record being approved
    if(current.document_id || current.sysapproval){
        //Query for approval records for this user/record
        var app = new GlideRecord('sysapproval_approver');
        //Handle empty document_id and sysapproval fields
        if(!current.document_id.nil()){
            app.addQuery('document_id', current.document_id);
        }
        else if(!current.sysapproval.nil()){
            app.addQuery('sysapproval', current.sysapproval);
        }
        app.addQuery('approver', current.approver);
        app.addQuery('state', 'approved');
        //Optionally restrict to current workflow
        //app.addQuery('wf_activity.workflow_version', current.wf_activity.workflow_version);
        app.query();
        if(app.next()){
            //If previous approval is found set this approval to 'approved'
            current.state = 'approved';
            current.u_work_notes = "Approval marked by system as 'Not Longer Required' due to a previous approval on the same record by the same user.";
        }
    }
}
```

{{< /tab >}}

{{< /tabs >}}

{{< callout type="info" >}}
Both rules match on `document_id` when present and fall back to `sysapproval` — the two fields ServiceNow uses depending on which approval engine (legacy Workflow vs. Flow Designer) generated the record.
{{< /callout >}}
