---
title: "Check Current Time is in Schedule"
---

## Overview

To make a Scheduled Job only run during a set schedule, add a conditional script that checks whether the current system time falls within the schedule of a selected `cmn_schedule` record. This uses the `GlideSchedule` API and its `isInSchedule()` method.

## Testing in a Background Script

Use this version to test a schedule directly from a Background Script, replacing the sys_id with your own `cmn_schedule` record:

```javascript {linenos=table,linenostart=1}
var schedule = new GlideSchedule("090eecae0a0a0b260077e1dfa71da828");
var currentDateTime = new GlideDateTime();
if (schedule.isInSchedule(currentDateTime)) {
    gs.print("The schedule is active.");
} else {
    gs.print("The schedule is not active.");
}
```

## Using as a Scheduled Job Condition

Scheduled Jobs support a **Condition** script that must evaluate to `true` for the job to run. Use this version in the Condition field to gate execution on the schedule:

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

{{< callout type="info" >}}
The sys_id passed to `GlideSchedule` must reference a record on the `cmn_schedule` table. You can find the sys_id by opening the schedule record and checking the URL, or by right-clicking the form header and selecting **Copy sys_id**.
{{< /callout >}}
