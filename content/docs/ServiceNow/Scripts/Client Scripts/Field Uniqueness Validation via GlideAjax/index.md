---
title: "Field Uniqueness Validation via GlideAjax"
---

## Overview

This pattern validates a user-entered field value against a table server-side, before the form is allowed to submit, using a paired **onChange** + **onSubmit** Catalog Client Script and a shared **Script Include**. The example here validates a new distribution group's email address against `sys_user_group` (to prevent creating a group with an email that's already in use), via a `CheckGroupEmailExists` Script Include:

- **onChange** — fires as soon as the user finishes typing an email address. It calls `CheckGroupEmailExists.checkEmail()` via `GlideAjax`, and if a match is found, clears the field and shows an inline field error immediately, so the user gets fast feedback while still on the field.
- **onSubmit** — performs the same check again at submit time as a safety net (covering cases like autofill, paste, or a value that was valid when last checked but became a duplicate since), and blocks submission if a duplicate is found.
- **Script Include** — `CheckGroupEmailExists`, extending `AbstractAjaxProcessor`, does the actual `sys_user_group` lookup for an active group with the given email and returns `'true'`/`'false'` as a string.

## Problem Statement

Some catalog variables need to be unique or otherwise validated against a table that isn't exposed as a simple reference field — for instance, a free-text "email address" variable that must not collide with an existing group's email. Client-side JavaScript alone can't query server tables, so the check has to go through `GlideAjax` to a Script Include. Checking only `onChange` isn't fully reliable (a user could bypass the change event or the value could go stale), so pairing it with an `onSubmit` check guarantees the value is verified again immediately before the record is saved.

{{< tabs >}}

  {{< tab name="onChange Client Script" >}}

```javascript {linenos=table,linenostart=1,filename="Validate Distribution Email - onChange.js"}
/**
 * Script Name: Validate Distribution Email - onChange
 * Description:
 *     Catalog client onChange script for ServiceNow that checks if the entered distribution email address already exists as a group.
 *     When the email_address field value changes, it calls the 'CheckGroupEmailExists' Script Include via GlideAjax.
 *     If the email exists, an error message is shown on the field; otherwise, any existing error message is hidden.
 *
 * Usage:
 *     Attach as an onChange client script to the 'email_address' field in a ServiceNow catalog item.
 *
 * Author: Konner Lester
 * Date Created: 8/1/2025
 * Last Modified: 8/4/2025
 */
function onChange(control, oldValue, newValue, isLoading) {
   if (isLoading || newValue == '') {
      return;
   }

   var ga = new GlideAjax('CheckGroupEmailExists');
   ga.addParam('sysparm_name', 'checkEmail');
   ga.addParam('sysparm_email', newValue);
   ga.getXMLAnswer(function(response) {
      if (response === 'true') {
        g_form.clearValue('email_address');
        g_form.showFieldMsg('email_address', 'This email is already in use by another group.', 'error');
      } else {
        g_form.hideFieldMsg('email_address', true);
      }
   });
}
```

{{< /tab >}}

  {{< tab name="onSubmit Client Script" >}}

```javascript {linenos=table,linenostart=1,filename="Validate Distribution Email - onSubmit.js"}
/**
 * Script Name: Validate Distribution Email - onSubmit
 * Description:
 *     Catalog client onSubmit script for ServiceNow that checks if the entered distribution email address already exists as a group before allowing form submission.
 *     If the email exists, an error message is shown and submission is prevented.
 *
 * Usage:
 *     Attach as an onSubmit client script to the relevant catalog item.
 *
 * Author: Konner Lester
 * Date Created: 8/4/2025
 * Last Modified: 8/4/2025
 */
function onSubmit() {
    var email = g_form.getValue('email_address');
    if (!email) {
        return true; // Allow submission if field is empty
    }

    var isValid = null;

    var ga = new GlideAjax('CheckGroupEmailExists');
    ga.addParam('sysparm_name', 'checkEmail');
    ga.addParam('sysparm_email', email);
    ga.getXMLAnswer(function(response) {
        if (response === 'true') {
            g_form.clearValue('email_address');
            g_form.showFieldMsg('email_address', 'This email is already in use by another group. Please try again.', 'error');
            isValid = false;
        } else {
            g_form.hideFieldMsg('email_address', true);
            isValid = true;
        }
    });

    // Prevent submission until the async check completes
    return false;
}
```

{{< callout type="warning" >}}
`getXMLAnswer()` is asynchronous, so the `isValid` flag set inside its callback isn't available by the time `onSubmit()` returns — this script always `return false`s and relies on the callback to correct course (e.g. re-submitting or clearing the field) rather than blocking synchronously on the ajax result. Keep this in mind if you adapt the pattern: a fully synchronous block requires `GlideAjax`'s synchronous mode or restructuring the submit flow.
{{< /callout >}}

{{< /tab >}}

  {{< tab name="Script Include" >}}

```javascript {linenos=table,linenostart=1,filename="CheckGroupEmailExists.js"}
/**
 * Script Name: CheckGroupEmailExists
 * Description:
 *     Script Include for ServiceNow that checks if an active group exists with the specified email address.
 *     Designed to be called via GlideAjax from client scripts.
 *
 * Methods:
 *     checkEmail() - Returns 'true' if an active group with the given email exists, otherwise 'false'.
 *                    Expects 'sysparm_email' as a parameter.
 *
 * Usage:
 *     var ga = new GlideAjax('CheckGroupEmailExists');
 *     ga.addParam('sysparm_name', 'checkEmail');
 *     ga.addParam('sysparm_email', 'group@example.com');
 *     ga.getXMLAnswer(function(response) { ... });
 *
 * Author: Konner Lester
 * Date Created: 8/1/2025
 * Last Modified: 8/1/2025
 */

var CheckGroupEmailExists = Class.create();
CheckGroupEmailExists.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
	checkEmail: function() {
		var email = this.getParameter('sysparm_email');
		if (!email)
			return 'false';

		var grp = new GlideRecord('sys_user_group');
		grp.addQuery('email', email);
		grp.addQuery('active', true);
		grp.setLimit(1);
		grp.query();
		return grp.hasNext() ? 'true' : 'false';
   }
});
```

{{< /tab >}}

{{< /tabs >}}
