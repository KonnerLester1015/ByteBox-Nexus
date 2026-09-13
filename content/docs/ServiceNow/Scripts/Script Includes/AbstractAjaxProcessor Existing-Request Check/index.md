---
title: "AbstractAjaxProcessor Existing-Request Check"
---

## Overview

This pattern is a building block for **Just-In-Time (JIT) permission requests**. These are catalog items that grant temporary, time-limited access and are meant to only ever have one active grant per user at a time. Before the user finishes filling out a new request, it warns them if they already have an active RITM for that same catalog item, so they understand a new submission will override the existing one rather than stack on top of it.

The pattern has two halves:

1. A **Script Include** extending `AbstractAjaxProcessor` (marked Client Callable) that queries `sc_req_item` for an active request on a specific catalog item (`cat_item`) submitted by the current user (`requested_for`), then looks up a variable's stored value (like `end_time`) via the `sc_item_option_mtom` join table. It returns a small JSON payload describing whether a request exists and its relevant details.
2. An **onLoad Catalog Client Script** that calls the Script Include via `GlideAjax`, and if `exists` comes back `true`, shows an informational field message warning the user.

A closely related variant extends the same idea to a *capacity* check rather than a simple existence check: instead of just detecting one active request, it counts how many active requests exist system-wide for the catalog item, and reports `max_reached = true` once two or more exist. along with the two most recent `end_time` values, sorted chronologically and joined with "and" for display. This is useful when the restriction is a shared pool limit rather than one-per-user.

## Problem Statement

Any catalog item used to grant time-limited access needs a way to enforce one active grant at a time, or a shared limit when access has a limited capacity. Without this check, a user who already has active access could submit another request and unintentionally change their existing expiration time without realizing it.

Showing a warning before the user completes the request helps prevent confusion about why their access may end earlier or later than expected.

Since multiple catalog items need to perform the same sc_req_item and sc_item_option_mtom lookup, that logic is handled in a single Script Include and reused across items. Each catalog item simply passes its own hardcoded catalog item and variable sys_id.

{{< tabs >}}

  {{< tab name="Script Include (Existence Check)" >}}

```javascript {linenos=table,linenostart=1,filename="CheckExistingJitRequest.js"}
/**
 * Script Name: CheckExistingJitRequest
 * Description: This Script Include checks if the current user has an active JIT (Just-In-Time) access request (RITM) for a specific catalog item.
 *              If one exists, it retrieves the value of the `end_time` variable from the associated request item.
 *              This script is used to inform users in the Service Portal that their existing access may be overridden.
 * 
 * Usage: This script is used as a server-side processor callable from a GlideAjax call in a Catalog Client Script.
 *      Example record:
 *         Name: CheckExistingJitRequest
 *         Table: sys_script_include (Script Include) 
 * 
 * Context: Called from an `onLoad` Catalog Client Script on a JIT-access catalog item form. 
 *          It uses GlideRecord to search the `sc_req_item` table for active requests created for the logged-in user and retrieves the `end_time` from the `sc_item_option_mtom` table.
 * 
 * Author: Konner Lester
 * Date Created: May 23, 2025
 * Last Modified: May 23, 2025
 * 
 * Note: 
 * - This Script Include must be marked as **Client Callable**.
 * - The `catItemId` and `endTimeVariableId` values are hardcoded and must match the catalog item and variable sys_ids.
 * - The `end_time` variable must exist on the catalog item and be correctly referenced in `sc_item_option_mtom`.
 */

var CheckExistingJitRequest = Class.create();
CheckExistingJitRequest.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {

    checkExisting: function() {
        var userId = gs.getUserID();
        var catItemId = '23ac29d33bad2610fd6edba693e45ab6'; // Catalog item sys_id
        var endTimeVariableId = '2d1ef2373b256a10fd6edba693e45aa9'; // Variable sys_id for end_time

        var result = {
            exists: false,
            end_time: ''
        };

        var ritmGr = new GlideRecord('sc_req_item');
        ritmGr.addQuery('active', true);
        ritmGr.addQuery('cat_item', catItemId);
        ritmGr.addQuery('requested_for', userId);
        ritmGr.orderByDesc('sys_created_on'); // Just in case there are multiple
        ritmGr.query();

        if (ritmGr.next()) {
            result.exists = true;

            // Look up the associated variable value
            var mtomGr = new GlideRecord('sc_item_option_mtom');
            mtomGr.addQuery('request_item', ritmGr.sys_id);
            mtomGr.addQuery('sc_item_option.item_option_new', endTimeVariableId);
            mtomGr.query();

            if (mtomGr.next()) {
                result.end_time = mtomGr.sc_item_option.value + ''; // Convert to string
            }
        }

        return new JSON().encode(result);
    }
});
```

{{< /tab >}}

  {{< tab name="Script Include (Request Limit Variant)" >}}

```javascript {linenos=table,linenostart=1,filename="CheckJitRequestLimit.js"}
/**
 * Script Name: CheckJitRequestLimit
 * Description: This Script Include checks if the current user has an active JIT (Just-In-Time) access request (RITM) for a specific catalog item.
 *              If the user already has an active request, it returns max_reached = false and does nothing further.
 *              If not, it checks if there are two or more active requests for the catalog item. If so, it returns max_reached = true
 *              and provides the end_time values of the two most recent requests (sorted chronologically and joined with "and").
 *              This script is used to inform users in the Service Portal when the shared request capacity has been reached.
 * 
 * Usage: This script is used as a server-side processor callable from a GlideAjax call in a Catalog Client Script.
 *      Example record:
 *         Name: CheckJitRequestLimit
 *         Table: sys_script_include (Script Include) 
 * 
 * Context: Called from an `onLoad` Catalog Client Script on a JIT-access catalog item form. 
 *          It uses GlideRecord to search the `sc_req_item` table for active requests created for the logged-in user and retrieves the `end_time` from the `sc_item_option_mtom` table.
 * 
 * Author: Konner Lester
 * Date Created: June 10, 2025
 * Last Modified: June 10, 2025
 * 
 * Note: 
 * - This Script Include must be marked as **Client Callable**.
 * - The `catItemId` and `endTimeVariableId` values are hardcoded and must match the catalog item and variable sys_ids.
 * - The `end_time` variable must exist on the catalog item and be correctly referenced in `sc_item_option_mtom`.
 * - If the current user has an active request, the script returns early with max_reached = false.
 * - If two or more active requests exist, the script returns max_reached = true and the sorted end_time values of the two most recent requests, joined with "and".
 */
var CheckJitRequestLimit = Class.create();
CheckJitRequestLimit.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {

    checkCapacity: function() {
        var catItemId = 'a0583bd83bc6aa10fd6edba693e45ac2'; // Catalog item sys_id
        var endTimeVariableId = 'b458b71c3bc6aa10fd6edba693e45a2e'; // Variable sys_id for end_time

        var result = {
            max_reached: false,
            end_time: ''
        };
        var userId = gs.getUserID();

        // Check if current user has an active request
        var userRitmGr = new GlideRecord('sc_req_item');
        userRitmGr.addQuery('active', true);
        userRitmGr.addQuery('cat_item', catItemId);
        userRitmGr.addQuery('requested_for', userId);
        userRitmGr.query();
        if (userRitmGr.hasNext()) {
            // User already has an active request, do nothing else
            return new JSON().encode(result);
        }

        var ritmGr = new GlideRecord('sc_req_item');
        ritmGr.addQuery('active', true);
        ritmGr.addQuery('cat_item', catItemId);
        ritmGr.orderByDesc('sys_created_on');
        ritmGr.query();

        var activeCount = 0;
        var ritmSysIds = [];

        while (ritmGr.next()) {
            activeCount++;
            if (ritmSysIds.length < 2) {
                ritmSysIds.push(ritmGr.sys_id + '');
            }
        }

        if (activeCount >= 2) {
            result.max_reached = true;
        }

        // Collect end_time values from up to 2 requests
        var endTimes = [];
        for (var i = 0; i < ritmSysIds.length; i++) {
            var mtomGr = new GlideRecord('sc_item_option_mtom');
            mtomGr.addQuery('request_item', ritmSysIds[i]);
            mtomGr.addQuery('sc_item_option.item_option_new', endTimeVariableId);
            mtomGr.orderByDesc('sys_created_on');
            mtomGr.query();

            if (mtomGr.next()) {
                endTimes.push(mtomGr.sc_item_option.value + '');
            }
        }
	
		// Helper to parse "June 9, 2025 at 5:54 PM" to a Date object
		function parseEndTime(str) {
			// Remove "at" and replace with space, then parse
			// Example: "June 9, 2025 at 5:54 PM" -> "June 9, 2025 5:54 PM"
			var cleaned = str.replace(' at ', ' ');
			return new Date(cleaned);
		}

		endTimes.sort(function(a, b) {
			return parseEndTime(a) - parseEndTime(b);
		});

		if (endTimes.length === 2) {
            result.end_time = endTimes[0] + " and " + endTimes[1];
        } else {
            result.end_time = endTimes.join('');
        }

		return new JSON().encode(result);
    }
});
```

{{< /tab >}}

  {{< tab name="Client Script" >}}

```javascript {linenos=table,linenostart=1,filename="Existing JIT Request In Progress.js"}
/**
 * Script Name: Existing JIT Request In Progress
 * Description: This client script checks if the currently logged-in user already has an active JIT (Just-In-Time) access request. 
 *              If one exists, it retrieves the scheduled end time and displays a warning message to the user on the catalog item form.
 * 
 * Usage: This script is utilized as a Catalog Client Script (type: onLoad) on a JIT-access catalog item.
 *      Example record:
 *         Name: Existing JIT Request In Progress
 *         Table: catalog_script_client (Catalog Client Scripts) 
 * 
 * Context: Triggered when the catalog item form is loaded in the Service Portal. 
 *          Relies on a Script Include (`CheckExistingJitRequest`) to perform the server-side GlideRecord lookup.
 * 
 * Author: Konner Lester
 * Date Created: May 23, 2025
 * Last Modified: May 23, 2025
 */
function onLoad() {
    var ga = new GlideAjax('CheckExistingJitRequest');
    ga.addParam('sysparm_name', 'checkExisting');
    ga.getXMLAnswer(function(response) {
        if (!response) return;

        var data = JSON.parse(response);
        if (data.exists === true) {
            var message = "You already have an active request for this access";
            
            if (data.end_time) {
                message += " scheduled to end on " + data.end_time + ".";
            } else {
                message += ".";
            }

            message += " Submitting a new request will override the current expiration, and access will be revoked based on this new request.";

            g_form.showFieldMsg('duration', message, 'info');
        }
    });
}
```

{{< /tab >}}

{{< /tabs >}}
