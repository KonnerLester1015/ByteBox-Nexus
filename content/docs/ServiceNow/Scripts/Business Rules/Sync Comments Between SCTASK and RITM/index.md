---
title: "Sync Comments Between SCTASK and RITM"
---

## Overview

This document describes the solution for **bi-directional syncing of comments** between ServiceNow `sc_task` (SCTASK) records and their parent `sc_req_item` (RITM) records.

Two **Business Rules** are used to keep comments in sync:

1. **Copy Comments from SCTASK to RITM**  
   Triggered when a comment is added on an SCTASK. The comment is copied to the parent RITM if it is not already present.

2. **Copy Comments from RITM to SCTASK**  
   Triggered when a comment is added on an RITM. The comment is copied to all **active** SCTASKs linked to that RITM if it is not already present.

## Problem Statement

By default, comments in ServiceNow do not automatically propagate between related SCTASK and RITM records. This can lead to:

- Incomplete communication between requesters and fulfillers.
- The need for manual duplication of comments.
- Potential delays or misunderstandings in request fulfillment.

The goal of this solution is to **automatically keep comments synchronized** in both directions, reducing manual effort and ensuring consistent communication.

{{< tabs >}}

  {{< tab name="Copy Comments from SCTASK to RITM" >}}
  
```javascript {linenos=table,linenostart=1,filename="Copy Comments from SCTASK to RITM.js"}
(function executeRule(current, previous /*null when async*/) {
    // Get the latest journal entry (comment) from the SCTASK
    taskCommentRaw = current.comments.getJournalEntry(1);

    // Remove timestamp and author from the SCTASK comment
    var reg_exp = new RegExp('\n');
    var i = taskCommentRaw.search(reg_exp);
    var taskCommentCleaned = '';
    if (i > 0) {
        taskCommentCleaned = taskCommentRaw.substring(i + 1, taskCommentRaw.length).trim();
    }

    // Query for the parent RITM record using the SCTASK's parent field
    var ritmGR = new GlideAggregate('sc_req_item');
    ritmGR.addQuery('sys_id', current.parent);
    ritmGR.query();

    if (ritmGR.next()) {
        // Get the latest journal entry (comment) from the RITM
        ritmCommentRaw = ritmGR.comments.getJournalEntry(1);
        var ritmCommentCleaned = '';

        // Remove timestamp and author from the RITM comment
        var j = ritmCommentRaw.search(reg_exp);
        if (j > 0) {
            ritmCommentCleaned = ritmCommentRaw.substring(j + 1, ritmCommentRaw.length).trim();
        }

        // If the cleaned SCTASK comment is not already present in the RITM comment, copy it over
        if (taskCommentCleaned && taskCommentCleaned !== ritmCommentCleaned) {
            ritmGR.comments = taskCommentCleaned;
            ritmGR.update();
        }
    }

})(current, previous);
```

{{< /tab >}}

{{< tab name="Copy Comments from RITM to SCTASK" >}}

```javascript {linenos=table,linenostart=1,filename="Copy Comments from RITM to SCTASK.js"}
(function executeRule(current, previous /*null when async*/) {
    // Get the latest journal entry (comment) from the RITM
    var ritmCommentRaw = current.comments.getJournalEntry(1);

    // Remove timestamp and author from the RITM comment
    var reg_exp = new RegExp('\n');
    var i = ritmCommentRaw.search(reg_exp);
    var ritmCommentCleaned = '';
    if (i > 0) {
        ritmCommentCleaned = ritmCommentRaw.substring(i + 1, ritmCommentRaw.length).trim();
    }

    // Find all active SCTASK records linked to this RITM
    var taskGR = new GlideRecord('sc_task');
    taskGR.addQuery('request_item', current.sys_id);
    taskGR.addQuery('active', true); // Only update active tasks
    taskGR.query();

    while (taskGR.next()) {
        // Get the latest journal entry (comment) from the SCTASK
        var taskCommentRaw = taskGR.comments.getJournalEntry(1);
        var taskCommentCleaned = '';

        // Remove timestamp and author from the SCTASK comment
        var j = taskCommentRaw.search(reg_exp);
        if (j > 0) {
            taskCommentCleaned = taskCommentRaw.substring(j + 1, taskCommentRaw.length).trim();
        }

        // If the cleaned RITM comment is not already present in the SCTASK comment, copy it over
        if (ritmCommentCleaned && ritmCommentCleaned !== taskCommentCleaned) {
            taskGR.comments = ritmCommentCleaned;
            taskGR.update();
        }
    }
})(current, previous);

{{< /tab >}}

{{< /tabs >}}
