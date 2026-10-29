---
title: "User Criteria Check"
---

## Overview

This pattern restricts which **choices a user can see in a catalog variable** based on whether they match a **User Criteria** record. The catalog item stays open to everyone, but certain options in a Select Box variable are shown only to users who qualify. Everyone else sees the item with those options removed.

The pattern has two halves:

1. A reusable **Script Include** extending `AbstractAjaxProcessor` (marked Client Callable). It takes a User Criteria sys_id, checks that the value is a well-formed sys_id for an existing `user_criteria` record, and then calls the platform's `sn_uc.UserCriteriaLoader.userMatches()` API to check whether the current user matches. It returns `'true'` or `'false'`. The core logic is also exposed as a server-side `matches()` method, so business rules, Flow Designer script steps, and other Script Includes can use the same check without GlideAjax.
2. An **onLoad Catalog Client Script** that sends the User Criteria sys_id to the Script Include via `GlideAjax`. If the answer is anything other than `'true'`, it removes the restricted choices from the variable with `g_form.removeOption()`.

## Problem Statement

User Criteria in ServiceNow controls who can see an entire catalog item or category. It doesn't control individual choices *inside* a variable. Teams often want one catalog item where most options are available to everyone, while a few sensitive or specialized options are limited to a specific audience, such as members of a group, department, or location.

The User Criteria API (`sn_uc.UserCriteriaLoader`) and `gs` are server-side only, so a Catalog Client Script can't check User Criteria membership directly. A client-callable Script Include fills that gap.

Many catalog items may need the same check against different User Criteria records, so the sys_id is passed in as a parameter instead of being hardcoded. Each client script supplies its own User Criteria sys_id and its own list of restricted choices, and the Script Include stays generic.

{{< tabs >}}

  {{< tab name="Script Include" >}}

```javascript {linenos=table,linenostart=1,filename="UserCriteriaCheckAjax.js"}
/**
 * Script Name: UserCriteriaCheckAjax
 * Description: This Script Include checks whether a user matches a given User Criteria record.
 *              The User Criteria sys_id is passed in as a parameter, making the Script Include reusable
 *              across any catalog item or script that needs a User Criteria membership check.
 *
 * Usage: This script is used as a server-side processor callable from a GlideAjax call in a Catalog Client Script.
 *        It can also be called directly from other server-side scripts.
 *      Example record:
 *         Name: UserCriteriaCheckAjax
 *         Table: sys_script_include (Script Include)
 *
 * Context: Called from an `onLoad` Catalog Client Script to determine whether restricted variable choices
 *          should be shown to the current user. Uses `sn_uc.UserCriteriaLoader.userMatches()` to evaluate membership.
 *
 * Note:
 * - This Script Include must be marked as **Client Callable**.
 */
var UserCriteriaCheckAjax = Class.create();
UserCriteriaCheckAjax.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {

    /**
     * Client-callable (GlideAjax) entry point.
     * Checks whether the CURRENT logged-in user matches the given User Criteria.
     *
     * Params:
     *   sysparm_criteria_id - sys_id of a user_criteria record
     *
     * Returns: 'true' or 'false' (string)
     */
    userMatchesCriteria: function() {
        var criteriaId = this.getParameter('sysparm_criteria_id');
        return this.matches(criteriaId, gs.getUserID()) ? 'true' : 'false';
    },

    /**
     * Server-side reusable method.
     * Usage from other server scripts:
     *   new UserCriteriaCheckAjax().matches('<criteria_sys_id>');               // current user
     *   new UserCriteriaCheckAjax().matches('<criteria_sys_id>', '<user_id>');  // specific user
     *
     * @param {String} criteriaId - sys_id of a user_criteria record
     * @param {String} [userId]   - sys_user sys_id (defaults to current user)
     * @returns {Boolean}
     */
    matches: function(criteriaId, userId) {
        criteriaId = (criteriaId || '').toString().trim();
        userId = userId || gs.getUserID();

        if (!this._isValidCriteria(criteriaId)) {
            gs.warn('UserCriteriaCheckAjax: invalid or unknown user criteria sys_id "' + criteriaId + '"');
            return false;
        }

        return sn_uc.UserCriteriaLoader.userMatches(userId, [criteriaId]) === true;
    },

    /**
     * Confirms the value is a well-formed sys_id and an existing user_criteria record.
     */
    _isValidCriteria: function(criteriaId) {
        if (!/^[0-9a-f]{32}$/i.test(criteriaId)) {
            return false;
        }
        var gr = new GlideRecord('user_criteria');
        return gr.get(criteriaId);
    },

    type: 'UserCriteriaCheckAjax'
});
```

{{< /tab >}}

  {{< tab name="Client Script" >}}

```javascript {linenos=table,linenostart=1,filename="Hide Restricted Options By User Criteria.js"}
/**
 * Script Name: Hide Restricted Options By User Criteria
 * Description: This client script checks whether the currently logged-in user matches a specific User Criteria record.
 *              If the user does not match, it removes restricted choices from a Select Box variable on the catalog item form.
 *
 * Usage: This script is utilized as a Catalog Client Script (type: onLoad, UI Type: All) on a catalog item or variable set.
 *      Example record:
 *         Name: Hide Restricted Options By User Criteria
 *         Table: catalog_script_client (Catalog Client Scripts)
 *
 * Context: Triggered when the catalog item form is loaded in the native UI or Service Portal.
 *          Relies on a Script Include (`UserCriteriaCheckAjax`) to perform the server-side User Criteria evaluation.
 *
 * Note:
 * - `g_form.removeOption()` matches on the choice VALUE, not the label. Use the values from the
 *   variable's Question Choices.
 * - This applies to Select Box variables. Lookup Select Box and Reference variables should be filtered
 *   with a reference qualifier instead.
 * - Removing options client-side is a usability control, not a security control. If it matters that
 *   non-matching users can never submit a restricted value, also validate on the server.
 */
function onLoad() {
    var VARIABLE_NAME = 'request_type';                     // Select Box variable to filter
    var CRITERIA_ID = '<user_criteria_sys_id>';             // sys_id of the User Criteria record

    // Choice VALUES that only users matching the User Criteria should see
    var RESTRICTED_OPTIONS = [
        'restricted_option_1',
        'restricted_option_2',
        'restricted_option_3'
    ];

    var ga = new GlideAjax('UserCriteriaCheckAjax');
    ga.addParam('sysparm_name', 'userMatchesCriteria');
    ga.addParam('sysparm_criteria_id', CRITERIA_ID);
    ga.getXMLAnswer(function(answer) {
        if (answer === 'true') {
            return; // User matches the criteria, so leave every option visible
        }

        for (var i = 0; i < RESTRICTED_OPTIONS.length; i++) {
            g_form.removeOption(VARIABLE_NAME, RESTRICTED_OPTIONS[i]);
        }
    });
}
```

{{< /tab >}}

  {{< tab name="Server-Side Usage" >}}

```javascript {linenos=table,linenostart=1,filename="Server-Side Example.js"}
/**
 * Example: Reusing the same User Criteria check from a server-side script
 * (business rule, Flow Designer script step, another Script Include, etc.)
 */
var criteriaId = '<user_criteria_sys_id>';
var checker = new UserCriteriaCheckAjax();

// Check the current user
if (checker.matches(criteriaId)) {
    gs.info('Current user matches the User Criteria.');
}

// Check a specific user (server-side only)
var requestedFor = current.variables.requested_for + '';
if (!checker.matches(criteriaId, requestedFor)) {
    gs.info('Requested-for user does not match the User Criteria.');
}
```

{{< /tab >}}

{{< /tabs >}}

## Considerations

- **Why not call User Criteria from the client?** `gs` and `sn_uc.UserCriteriaLoader` exist only on the server. GlideAjax is the supported way for a client script to get a server-side answer.
- **Server-side enforcement.** Hiding choices only changes what the user sees. For stricter control, call `matches()` from an onSubmit validation (via GlideAjax), a catalog data policy, or the item's flow, and reject restricted values from users who don't match.