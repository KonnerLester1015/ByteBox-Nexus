# Maintain ServiceNow Approvers
## Overview

This guide outlines the process for maintaining an up-to-date list of ServiceNow Approvers using a scheduled script execution record. The goal is to ensure that only users who have received approval requests in the last 30 days remain in a custom group called "ServiceNow Approver Users" group. This helps optimize license usage by removing inactive approvers, while a separate flow is responsible for adding new approvers and their delegates as needed.

## Problem Statement

ServiceNow Approvers are assigned the Business Stakeholder license, which incurs a cost. Over time, users who no longer participate in approval workflows may remain in the approvers group, leading to unnecessary license consumption. To minimize costs and maintain an accurate list of active approvers, it is necessary to regularly remove users who have not received approval requests within a configurable time window (e.g., 30 days).

{{< tabs >}}

  {{< tab name="Granting Approver Role" >}}**Solution: Process Engine**
  
To dynamically grant the approver role only when needed (rather than providing standing access), use an automated flow or script. In this implementation, a Flow called "Assign Approver License" is triggered whenever a record is created on the approval table (`sysapproval_approver`). The flow logic:

- Checks if the approver has any delegates. If so, it adds the delegate(s) to the "ServiceNow Approver Users" group.
- Adds the primary approver to the group if they are not already a member.

This ensures that only users actively involved in approval processes (and their delegates) are granted the necessary roles and group membership.

Refer to the following flow diagram for an overview of the logic:

![Example Flow Diagram](/images/AssignApproverLicense.png)

{{< /tab >}}

{{< tab name="Maintaining Approver Role" >}}**Solution: Scheduled Script Execution**

A scheduled script execution record is used to automate the maintenance of the approvers group. The script:

- Retrieves all current members of the "ServiceNow Approver Users" group.
- Identifies users (and their active delegates) who have received at least one approval request in the last 30 days.
- Removes users from the group if they have not received an approval request in that period.

> **Note:** This script only removes inactive users. Adding users to the group is handled by a separate flow triggered when a new approval record is created.

Below is the script used in the scheduled job. Update the `groupId` variable to match your approver group.

```javascript {linenos=table,linenostart=1,filename="Maintain ServerNow Approver Users.js"}
// This script will get the list of members in the specified group and will also check if they have recieved at least one approval request in the last 30 days. If they have not, they will be removed from the group.    
var groupId = 'G8dfi3rcVn2917Mqy9DX214ecc4bcbac'; //ServiceNow Approver Users group

// Get the list of members in the ServiceNow Approver Users group
var members = [];
var groupMembers = new GlideRecord('sys_user_grmember');
groupMembers.addQuery('group.sys_id', groupId);
groupMembers.query();

while (groupMembers.next()) {
    // Get the user sysid from the group member record
    var userId = groupMembers.getValue('user');
    members.push(userId);
    var member = new GlideRecord('sys_user');
    if (member.get(userId)) {
        gs.info('Current Members: ' + member.getDisplayValue()); // Print member display name
    }
}

// Get the list of all approval records in the last 30 days. Add the approver sys_id to an array
var allApproversList = [];
var gaApprovalList = new GlideAggregate('sysapproval_approver');
gaApprovalList.addEncodedQuery('sys_created_onONLast 30 days@javascript:gs.beginningOfLast30Days()@javascript:gs.endOfLast30Days()');
gaApprovalList.groupBy('approver');
gaApprovalList.query();

while (gaApprovalList.next()) {
    // Get the approver sys_id from the approval record
    var approverSysId = gaApprovalList.getValue('approver');
    allApproversList.push(approverSysId);

    // Check for active delegates for this approver
    var delegateGR = new GlideRecord('sys_user_delegate');
    delegateGR.addQuery('user', approverSysId);
    delegateGR.addEncodedQuery('ends>=javascript:gs.beginningOfToday()^starts<=javascript:gs.endOfToday()');
    delegateGR.query();

    while (delegateGR.next()) {
        // If there is an active delegate, add the delegate's sys_id to the approvers list
        var delegateSysId = delegateGR.getValue('delegate');
        allApproversList.push(delegateSysId);
        var delegateUser = new GlideRecord('sys_user');
        if (delegateUser.get(delegateSysId)) {
            gs.info('Delegate added: ' + delegateUser.getDisplayValue());
        }
    }

    // Print the approver's display name
    var approver = new GlideRecord('sys_user');
    if (approver.get(approverSysId)) {
        gs.info('Approvers in the last 30 days: ' + approver.getDisplayValue()); // Print approver display name
    }
}

// Ensure allApproversList contains unique values;
var uniqueApprovers = new ArrayUtil().unique(allApproversList);

// Compare if user is in both the members and uniqueApprovers arrays
// If a member is not in the uniqueApprovers, they will be removed from the group
var approversToRemove = [];
for (var i = 0; i < members.length; i++) {
    if (uniqueApprovers.indexOf(members[i]) == -1) {
        approversToRemove.push(members[i]);
    }
}

// Remove the approvers from the group
// Loop through the approversToRemove array and remove each user from the group
var grRemoveApprover = new GlideRecord('sys_user_grmember');
grRemoveApprover.addQuery('group', groupId);
grRemoveApprover.addQuery('user', 'IN', approversToRemove);
grRemoveApprover.query();

while (grRemoveApprover.next()) {
    grRemoveApprover.deleteRecord();
}
```

{{< /tab >}}

{{< /tabs >}}

