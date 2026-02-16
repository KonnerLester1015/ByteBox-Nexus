---
title: "Resend Approval Emails"
---

## Overview

In ServiceNow, approval workflows are critical for ensuring that requests and changes are properly reviewed. However, approvers may sometimes miss or overlook the initial approval email, leading to delays in request fulfillment or change execution. To address this, you can automate the process of resending approval notifications for pending approvals that are more than 7 days old.

This guide outlines how to create a **Scheduled Script Execution** record in ServiceNow that runs daily, checks for 'requested' approvals older than 7 days, and resends the appropriate email notifications.

## 1. Problem Statement

Approvers occasionally miss or accidentally delete the original approval notification email. This can result in outstanding approvals remaining unaddressed, causing bottlenecks in workflows and delayed service delivery.

## 2. Solution: Scheduled Script Execution

To automate the process, we will create a scheduled job in ServiceNow that:

- Runs daily within a defined schedule.
- Identifies all approval records (`sysapproval_approver`) in the 'requested' state that have not been updated in the last 7 days.
- Triggers the appropriate email notification event for each qualifying record.

### 2.1 Schedule Condition

To avoid triggering notifications during extended periods of company leave such as holidays, shutdowns, or weekends the script is configured to run only during a defined work schedule.

This **"8-5 weekdays excluding holidays"** schedule reflects standard working hours and business expectations. The goal is to avoid sending unnecessary reminders when no action is expected.

Use the following condition script to check if the current time falls within the defined schedule window:

```javascript {linenos=table,linenostart=1}
var answer = false;
var schedule = new GlideSchedule("090eecae0a0a0b260077e1dfa71da828");
// Check if the current date and time fall within the schedule
var currentDateTime = new GlideDateTime();
if (schedule.isInSchedule(currentDateTime)) {
    // Return true if the schedule is active
    answer = true;
} else {
    // Return false if the schedule is not active
    answer = false;
}
```

### 2.1 Script to Run

If the schedule condition is met, the following script will execute. It finds all 'requested' approvals older than 7 days and resends the appropriate email notification:

```javascript {linenos=table,linenostart=1}
// Create a GlideRecord object for the 'sysapproval_approver' table
var record = new GlideRecord('sysapproval_approver');

// Add a query to filter records with the 'state' field set to 'requested'
record.addQuery('state', 'requested');

// Add an encoded query to filter out records updated in the last 7 days
record.addEncodedQuery('sys_updated_onRELATIVELT@dayofweek@ago@7');

// Execute the query
record.query();

// Iterate through the results
while (record.next()) {
    if (record.source_table == "change_request") {
        gs.eventQueue("change_request.approval.inserted", record, gs.getUserID(), gs.getUserName());
        gs.addInfoMessage("Change approval email resent.");
    } else {
        gs.eventQueue("approval.inserted", record, gs.getUserID(), gs.getUserName());
        gs.addInfoMessage("Approval email resent.");
    }
}
```

### 3 Verification

After implementing the scheduled job:

- Monitor the approval records to ensure that pending approvals older than 7 days receive new email notifications.
- Check the ServiceNow system logs for info messages confirming that approval emails have been resent.
- Approvers should receive the new notifications in their inbox, helping to keep workflows moving efficiently.

By automating the resending of approval notifications, you can reduce delays, improve visibility, and ensure that critical requests and changes are not overlooked.