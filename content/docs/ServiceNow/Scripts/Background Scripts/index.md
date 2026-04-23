---
title: "Background Script Repository"
description: "A collection of reusable ServiceNow background scripts for administration and troubleshooting."
weight: 10
type: docs
prev: docs/servicenow/scripts
---

# ServiceNow Background Scripts

A curated collection of snippets for quick platform administration.

{{< callout type="info" >}}
**Note:** Always run scripts in a sub-production instance first. Use `gs.info()` to verify data before executing updates.
{{< /callout >}}

## Approval Audit Report (RITM)
*Summarizes catalog item approvals for a specific sys_id.*

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  When auditing a user's activity or preparing for offboarding, it's difficult to see a high-level summary of what types of requests they are approving without looking at dozens of individual records.

  ### The Solution
  This script filters the `sysapproval_approver` table for a specific user and class (RITM), then aggregates the results by Catalog Item name to provide a clear summary.
  {{< /tab >}}

  {{< tab name="Script" >}}
  **Please note the highlight line(s) for where to change inputs**

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