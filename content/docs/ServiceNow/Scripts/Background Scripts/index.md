---
title: "Background Script Repository"
description: "A collection of reusable ServiceNow background scripts for administration and troubleshooting."
weight: 10
type: docs
prev: docs/servicenow/scripts
---

A curated collection of snippets for quick platform administration.

{{< callout type="info" >}}
**Note:** Always run scripts in a sub-production instance first. Use `gs.info()` to verify data before executing updates.
{{< /callout >}}

## Approval Audit Report (RITM)
Quickly see a summary of all Catalog Item approvals for a specific user, including counts by item type and record reference.

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  When auditing a user's activity or preparing for offboarding, it's difficult to see a high-level summary of what types of requests they are approving without looking at dozens of individual records.

  ### The Solution
  This script filters the `sysapproval_approver` table for a specific user and class (RITM), then aggregates the results by Catalog Item name to provide a clear summary.
  {{< /tab >}}

  {{< tab name="Script" >}}
  **Please note the highlighted line(s) for where to change inputs**

  ```javascript {linenos=table,hl_lines=[2],linenostart=1,filename="ApprovalAudit.js"}
// Set the Approver Sys ID you want to audit 
var approverSysId = '9d4f2e1c8a3b5f7e6c2a8b4d1f9e3c7a';

var grApproval = new GlideRecord('sysapproval_approver');
grApproval.addEncodedQuery('approver=' + approverSysId + '^sysapproval.sys_class_name=sc_req_item');
grApproval.query();

// Look up Approver Name for the Header
var approverName = approverSysId;
var grUser = new GlideRecord('sys_user');
if (grUser.get(approverSysId)) {
    approverName = grUser.getDisplayValue();
}

var counts = {}; 
var logOutput = "";

// --- HEADER SECTION ---
logOutput += '=======================================================================================================================\n';
logOutput += 'SERVICENOW APPROVAL AUDIT REPORT\n';
logOutput += 'Approver: ' + approverName + ' (' + approverSysId + ')\n';
logOutput += 'Generated on: ' + new GlideDateTime().getDisplayValue() + '\n';
logOutput += '=======================================================================================================================\n\n';

// Detailed Table Header
logOutput += pad('Approval Sys ID', 35) + ' | ' + pad('RITM Sys ID', 35) + ' | ' + 'Catalog Item\n';
logOutput += '-----------------------------------------------------------------------------------------------------------------------\n';

while (grApproval.next()) {
    var ritmSysId = grApproval.getValue('sysapproval');
    var grRITM = new GlideRecord('sc_req_item');
    
    if (grRITM.get(ritmSysId)) {
        var catItemName = grRITM.cat_item.getDisplayValue();
        
        if (!counts[catItemName]) {
            counts[catItemName] = 1;
        } else {
            counts[catItemName]++;
        }
        
        logOutput += pad(grApproval.getUniqueValue(), 35) + ' | ' + 
                     pad(ritmSysId, 35) + ' | ' + 
                     catItemName + '\n';
    }
}

// Print Main Report
gs.info(logOutput);

// --- AGGREGATED SUMMARY ---
var summary = "-----------------------------------------------------------------------------------------------------------------------\n";
summary += "SUMMARY BY ITEM TYPE\n";
summary += "-----------------------------------------------------------------------------------------------------------------------\n";

for (var item in counts) {
    summary += pad(item, 35) + ": " + counts[item] + "\n";
}
summary += "-----------------------------------------------------------------------------------------------------------------------\n";
summary += "Total Unique Items: " + Object.keys(counts).length + "\n";
summary += "Total Records Found: " + grApproval.getRowCount() + "\n";

gs.info(summary);

// Padding helper
function pad(str, length) {
    str = str || "";
    while (str.length < length) {
        str += " ";
    }
    return str;
}
```
  {{< /tab >}}

{{< tab name="Sample Output" >}}
```text {linenos=table,linenostart=1}
*** Script: =======================================================================================================================
SERVICENOW APPROVAL AUDIT REPORT
Approver: Jane Smith (9d4f2e1c8a3b5f7e6c2a8b4d1f9e3c7a)
Generated on: 04-23-2026 14:49:21
=======================================================================================================================

Approval Sys ID                     | RITM Sys ID                         | Catalog Item
-----------------------------------------------------------------------------------------------------------------------
7e2b4a9f1c8d3e5a6f2c8b1d4e9a3c7b    | 3a9f2e1c8b5d7a6c2f4e9b3d1a8c5f7e    | Folder Permissions
5c8d1e9f2a4b6e3c7d5a9f1e3b6c8d2a    | 9e3f1a2c7b5d8a4e6f2c9b1d3e5a7c8f    | Application Request
2d6a9e1c3f8b5a7e4c2f9d1b6a8e3c5f    | 8f5c2a9d1e7b3a6c4f2e8d5a1b9c3e7d    | Internet and External Email
4b7c2e9a1f3d6e5c8a2b7f1d4a9e3c6b    | 1c5e8d2f9a3b6c7e2d4a5f8b1e9c3a6f    | Application Request
9a3e1f7c2d6b4a8e5c1f3d9b2e6a7c5f    | 6d2b9f1a3c7e5a8d2c4f1b9e3a6c8d5e    | Application Request
3f8c1e5a9b2d7c4e6a1f3d8b5a2c9e7f    | 7e9d1c3a5f2b8e6c4d1a9f3e2b7c5a8d    | Folder Permissions
1a6f3c2e9d5b7a4c8e1f6d3a9b2c5e7f    | 4c7b1f9e2a6d3c8e5a1f7d2b9c3e6a8b    | Application Request
8e5a2c1f9d3b7e4a6c2f8d1a5b9e3c7a    | 2f4d7a1c9b3e5a8c6e1f3d9a2b5c7e8d    | Application Request
6c2d1a9f3e7b5c8e4a2f1d9b6e3c7a5f    | 5a8f3c1d9e2b6a7c4e1f3d8a2c5b9e6d    | Application Request
3b9c4e1f7a2d6c5a8e1b3f9d2a5c7e4f    | 1e6d3a8c2f9b5e7a4c1d6f3b8a2e9c5d    | Folder Permissions
7d2a5f1c9e3b6a8d4c1f2e9a3b6c7d5e    | 9c1f3e5a2d7b4a8e6c2f1d9a3b5c7e8d    | Internet and External Email
4a8b1d6f3c2e9a5f7d1b4c9e2a6d3f8c    | 2d5e9a1c3f7b6a8e4c1f3d9b2a5c7e8d    | Internet and External Email

*** Script: -----------------------------------------------------------------------------------------------------------------------
SUMMARY BY ITEM TYPE
-----------------------------------------------------------------------------------------------------------------------
Folder Permissions                 : 3
Application Request                : 6
Internet and External Email        : 3
-----------------------------------------------------------------------------------------------------------------------
Total Unique Items: 3
Total Records Found: 12
```
{{< /tab >}}

{{< /tabs >}}

## Group Approval Audit Report
Summarize approvals for a specific group, showing counts by Catalog Item and listing each approval record and by member breakdown.

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  When auditing group owned approvals, it is difficult to see which Catalog Items the group is responsible for without manually reviewing each record. Additionally, tracking individual accountability within the group specifically, identifying which members are actively approving or rejecting requests is highly manual.

  ### The Solution
  This script queries the `sysapproval_group` table for a specific Assignment Group filtering on Requested Items (sc_req_item). It aggregates the data by Catalog Item type and outputs a detailed breakdown listing the Group Approval Sys ID, the RITM Sys ID, and the specific group Approval State (e.g., Requested, Approved, Rejected).
  {{< /tab >}}

  {{< tab name="Script" >}}
  **Please note the highlighted line(s) for where to change inputs**

  ```javascript {linenos=table,hl_lines=[2],linenostart=1,filename="GroupApprovalAudit.js"}
// Set the Group Sys ID you want to audit 
var groupSysId = '0a06b711873f5d50b1f432ec0ebb3591';

// 1. Initialize Member Stats with current group members from sys_user_grmember
var memberStats = {};
var grMember = new GlideRecord('sys_user_grmember');
grMember.addQuery('group', groupSysId);
grMember.query();
while (grMember.next()) {
    var memberName = grMember.user.getDisplayValue();
    memberStats[memberName] = { approved: 0, rejected: 0 };
}

// 2. Query Group Approvals
var grApproval = new GlideRecord('sysapproval_group');
grApproval.addEncodedQuery('assignment_group=' + groupSysId + '^parent.sys_class_name=sc_req_item');
grApproval.query();

// Look up Group Name for the Header
var groupName = groupSysId;
var grGroup = new GlideRecord('sys_user_group');
if (grGroup.get(groupSysId)) {
  groupName = grGroup.getDisplayValue();
}

var counts = {}; 
var logOutput = "";

// --- HEADER SECTION ---
logOutput += '=========================================================================================================================================\n';
logOutput += 'SERVICENOW GROUP APPROVAL AUDIT REPORT\n';
logOutput += 'Assignment Group: ' + groupName + ' (' + groupSysId + ')\n';
logOutput += 'Generated on: ' + new GlideDateTime().getDisplayValue() + '\n';
logOutput += '=========================================================================================================================================\n\n';

// Detailed Table Header
logOutput += pad('Group Approval Sys ID', 35) + ' | ' + pad('RITM Sys ID', 35) + ' | ' + pad('Approval State', 16) + ' | ' + 'Catalog Item\n';
logOutput += '-----------------------------------------------------------------------------------------------------------------------------------------\n';

while (grApproval.next()) {
  var ritmSysId = grApproval.getValue('parent');
  var approvalState = grApproval.getDisplayValue('approval');
  var grRITM = new GlideRecord('sc_req_item');
  
  if (grRITM.get(ritmSysId)) {
      var catItemName = grRITM.cat_item.getDisplayValue();
      
      if (!counts[catItemName]) {
          counts[catItemName] = 1;
      } else {
          counts[catItemName]++;
      }
      
      logOutput += pad(grApproval.getUniqueValue(), 35) + ' | ' + 
                   pad(ritmSysId, 35) + ' | ' + 
                   pad(approvalState, 16) + ' | ' + 
                   catItemName + '\n';
  }
}

// Print Main Report
gs.info(logOutput);

// 3. Gather Individual User Approval Stats (Batch Query to optimize performance)
var grUserApproval = new GlideRecord('sysapproval_approver');
grUserApproval.addEncodedQuery('group.assignment_group=' + groupSysId + '^group.parent.sys_class_name=sc_req_item^stateINapproved,rejected');
grUserApproval.query();

while (grUserApproval.next()) {
    var approverName = grUserApproval.approver.getDisplayValue();
    var state = grUserApproval.getValue('state');
    
    // If a user did an approval historically but is no longer in the group, add them dynamically
    if (!memberStats[approverName]) {
        memberStats[approverName] = { approved: 0, rejected: 0 };
    }
    
    if (state === 'approved') {
        memberStats[approverName].approved++;
    } else if (state === 'rejected') {
        memberStats[approverName].rejected++;
    }
}

// --- AGGREGATED SUMMARY BY ITEM ---
var summary = "-----------------------------------------------------------------------------------------------------------------------\n";
summary += "SUMMARY BY ITEM TYPE\n";
summary += "-----------------------------------------------------------------------------------------------------------------------\n";

for (var item in counts) {
  summary += pad(item, 35) + ": " + counts[item] + "\n";
}
summary += "-----------------------------------------------------------------------------------------------------------------------\n";
summary += "Total Unique Items: " + Object.keys(counts).length + "\n";
summary += "Total Records Found: " + grApproval.getRowCount() + "\n\n";

// --- MEMBER BREAKDOWN SUMMARY ---
summary += "-----------------------------------------------------------------------------------------------------------------------\n";
summary += "MEMBER APPROVAL BREAKDOWN\n";
summary += "-----------------------------------------------------------------------------------------------------------------------\n";
summary += pad('Group Member', 35) + ' | ' + pad('Approved', 12) + ' | ' + 'Rejected\n';
summary += "-----------------------------------------------------------------------------------------------------------------------\n";

for (var member in memberStats) {
    summary += pad(member, 35) + ' | ' + 
               pad(memberStats[member].approved.toString(), 12) + ' | ' + 
               memberStats[member].rejected + '\n';
}
summary += "-----------------------------------------------------------------------------------------------------------------------\n";

gs.info(summary);

// Padding helper
function pad(str, length) {
  str = str || "";
  while (str.length < length) {
      str += " ";
  }
  return str;
}
```
  {{< /tab >}}

{{< tab name="Sample Output" >}}
```text {linenos=table,linenostart=1}
*** Script: =========================================================================================================================================
SERVICENOW GROUP APPROVAL AUDIT REPORT
Assignment Group: HR (f5bf7yr74hdjkwei3832ec0ebb3520)
Generated on: 05-18-2026 12:57:06
=========================================================================================================================================

Group Approval Sys ID               | RITM Sys ID                         | Approval State   | Catalog Item
-----------------------------------------------------------------------------------------------------------------------------------------
ea0f78bfasdasfsdfsdfsdsfafa4e32a    | 9ffe849dngu4navbxklfxv4fafa4e339    | Approved         | Login
49df03ncu48fn4ndb1f432ec0ebb357f    | 3228e85787sj378dbfgh4bvj0ebb352e    | Approved         | Application
sdfsdfsdfkmkfe1028d04266cebb35f9    | d4fa8a6287c73hf784n2c82934bb35a8    | Approved         | SAP 
925b74b72fjsdfngkjndfg4fafa4e399    | 707828r3h78r234hr723r82dafa4e3fa    | Requested        | Login

*** Script: -----------------------------------------------------------------------------------------------------------------------
SUMMARY BY ITEM TYPE
-----------------------------------------------------------------------------------------------------------------------
Login                              : 2
Application                        : 1
SAP                                : 1
-----------------------------------------------------------------------------------------------------------------------
Total Unique Items: 3
Total Records Found: 4

-----------------------------------------------------------------------------------------------------------------------
MEMBER APPROVAL BREAKDOWN
-----------------------------------------------------------------------------------------------------------------------
Group Member                        | Approved     | Rejected
-----------------------------------------------------------------------------------------------------------------------
John Doe                            | 1            | 0
Jane Smith                          | 2            | 0
-----------------------------------------------------------------------------------------------------------------------
```
{{< /tab >}}

{{< /tabs >}}

## Validate Schedule
Ensures a requested date has sufficient lead time based on a specific schedule (e.g. business hours, exluding weekends and holidays) and a defined minimum duration (e.g. 3 business days).

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  When enforcing lead times for processes like Change Management or Catalog Requests, it is difficult to accurately calculate if a user has provided enough notice. A simple date subtraction doesn't account for weekends, holidays, or after-hours.

  ### The Solution
  This script uses ServiceNow's `GlideSchedul` API to calculate the exact amount of working time between a start date and an end date based on a specific schedule (e.g., 8 AM - 5 PM). It then checks if that working duration is at least 3 business days (which equals 27 business hours in an 8-5 schedule) and returns a success or failure message.
  {{< /tab >}}

  {{< tab name="Script" >}}
  **Please note the highlight line(s) for where to change inputs**

  ```javascript {linenos=table,hl_lines=[4,5,6,7,8],linenostart=1,filename="ValidateSchedule.js"}
// --- CONFIGURATION ---

var config = {
    start: '2026-03-31 08:00:00', // Mocking submission time
    end: '2026-04-02 17:00:00',   // Date Needed By
    scheduleId: '090eecae0a0a0b260077e1dfa71da828', // 8-5 weekdays
    daysRequired: 3,
    hoursInWorkDay: 9 // 8am to 5pm is 9 hours
};

var startGDT = new GlideDateTime();
startGDT.setDisplayValue(config.start); // Treats '08:00:00' as 8 AM YOUR time

var endGDT = new GlideDateTime();
endGDT.setDisplayValue(config.end);   // Treats '17:00:00' as 5 PM YOUR time

var schedule = new GlideSchedule(config.scheduleId);

// 1. Calculate the duration based on the schedule

var duration = schedule.duration(startGDT, endGDT);
var totalWorkMS = duration.getNumericValue(); // Total duration in milliseconds
var totalWorkHours = totalWorkMS / (1000 * 60 * 60);
var requiredHours = config.daysRequired * config.hoursInWorkDay;

// 2. Formatting for the report

var report = "LEAD TIME VALIDATION\n";
report += "---------------------------------\n";
report += "Start Date     : " + startGDT.getDisplayValue() + "\n";
report += "End Date       : " + endGDT.getDisplayValue() + "\n";
report += "Work Hours Found: " + totalWorkHours.toFixed(2) + "\n";
report += "Work Hours Req : " + requiredHours + " (" + config.daysRequired + " days)\n";
report += "---------------------------------\n";

// 3. Validation Logic

if (totalWorkHours >= requiredHours) {
    report += "RESULT: ✔ Sufficient lead time provided.";
} else {
    var missingHours = requiredHours - totalWorkHours;
    report += "RESULT: ✘ Insufficient lead time. Missing " + missingHours.toFixed(2) + " business hours.";
}

gs.info(report);
```
  {{< /tab >}}

{{< tab name="Sample Output" >}}
### Sufficent Lead Time Example
The following example has 4/1/2026 at 8 AM as the start date and 4/6/2026 at 5 PM as the end date. In this case this spans across 5 calendar days but only includes 3 full business days (4/1, 4/2, and 4/6) because 4/3 is a holiday and 4/4-4/5 are a weekend.
```text {linenos=table,linenostart=1}
*** Script: LEAD TIME VALIDATION
---------------------------------
Start Date     : 04-01-2026 08:00:00
End Date       : 04-06-2026 17:00:00
Work Hours Found: 27.00
Work Hours Req : 27 (3 days)
---------------------------------
RESULT: ✔ Sufficient lead time provided.
```

### Insufficent Lead Time Example
The following example has 4/1/2026 at 8 AM to 4/3/2026 at 5 PM, which includes 2 full business days however the 3rd day (4/3) is not covered due to it being a holiday.
```text {linenos=table,linenostart=1}
*** Script: LEAD TIME VALIDATION
---------------------------------
Start Date     : 04-01-2026 08:00:00
End Date       : 04-03-2026 17:00:00
Work Hours Found: 18.00
Work Hours Req : 27 (3 days)
---------------------------------
RESULT: ✘ Insufficient lead time. Missing 9.00 business hours.
```

{{< /tab >}}


{{< /tabs >}}