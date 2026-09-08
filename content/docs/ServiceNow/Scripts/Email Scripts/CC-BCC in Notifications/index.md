---
title: "CC/BCC in Notifications"
---

## Overview

The Notification record form does not expose a CC or BCC field directly. To add a CC or BCC recipient to a notification, you must use an Email Script that calls `email.addAddress()`.

## Creating the Email Script

Create a new Email Script (**System Notification > Email > Notification Email Scripts**) with the following template:

```javascript {filename="CC_BCC_Script.js", linenos=table}
(function runMailScript(/* GlideRecord */ current, /* TemplatePrinter */ template,
          /* Optional EmailOutbound */ email, /* Optional GlideRecord */ email_action,
          /* Optional GlideRecord */ event) {

         email.addAddress("cc","email_address","name_of_recipent");
         email.addAddress("bcc","email_address","name_of_recipent");

         //Example
        //email.addAddress("cc","Konner.Lester@example.com","Konner Lester");


})(current, template, email, email_action, event);
```

`email.addAddress()` takes three arguments:

- **Type** — `"cc"` or `"bcc"`
- **Email address** — the recipient's email address
- **Name** — the recipient's display name

## Adding the Script to a Notification

Once the Email Script is created, reference it from the notification's **Message HTML** field using the `mail_script` syntax:

```
${mail_script:Name_of_script}
```

Replace `Name_of_script` with the name of the Email Script record you created.

{{< callout type="info" >}}
`mail_script` calls can be placed anywhere in the Message HTML body. The script itself does not render visible output unless you also call `template.print()` — it only needs to run to add the CC/BCC addresses to the outbound email.
{{< /callout >}}
