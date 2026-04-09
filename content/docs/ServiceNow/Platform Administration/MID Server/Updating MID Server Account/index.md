---
title: "Updating MID Server Account"
---

## Overview

This guide explains how to update the username and password used by an already established MID Server using a working sequence that updates the configuration file first, then the ServiceNow user record, and finally restarts the MID Server host service.

Common reasons for updating credentials include:

- Service account password rotation
- Migration to a new service account standard
- Security remediation after an audit finding

## Before You Start

Prepare these items first:

- Confirm new service account username and password
- Local administrator/root access on the MID host
- ServiceNow admin access to validate MID status after restart
- A maintenance window if the MID Server handles production workflows

{{< callout type="warning" >}}
Per ServiceNow documentation the MID Server goes down immediately when you save the ServiceNow user record. However, in my expierience, the MID Server remains `Up` and continues to pick up jobs for a short time after the user record is updated. Either way complete the password update in the instance and the restart as quickly as possible.
{{< /callout >}}

## Update Procedure

### 1. Identify the MID Server and service account

1. In ServiceNow, navigate to `MID Server > Servers`.
2. Locate the MID Server you want to update.
3. Note the `Logged in user`, `Host name`, and `Home directory` fields.

### 2. Update the user/password in the config.xml file

Complete the following steps for all MID Servers that use this service account:

1. On the MID Server host, go to the `Home directory`.
2. Open the `config.xml` file in a text editor.
3. Locate the `mid.instance.username` and/or `mid.instance.password` parameters.
4. For a password change, replace the encrypted value with the new password in plain text.

{{< callout type="info" >}}
Note that the password being in plain text in the `config.xml` file is expected. The MID Server agent encrypts the password when it reads the file, so do not be alarmed by this.
{{< /callout >}}

Before:

```xml
<parameter name="mid.instance.password" secure="true" value="encrypted:7cHG3x8Ssx9m84qHaHlgKQ=="/>
```

After:

```xml
<parameter name="mid.instance.password" secure="true" value="newpassword"/>
```

Important notes:

- Delete the `encrypted:` prefix. Authentication fails if this prefix remains.
- Escape special XML characters. For example, use `&amp;` for `&` and `&lt;` for `<`.
- Do not accidentally delete the `/>` at the end of the line. Removing it causes an XML parser error in the MID Server agent log.
- Save the file.
- Do not restart the MID Server yet. The password change does not take effect until the final restart step.

### 3. Verify the service account configuration

1. Go to `System Security > Users and Groups > Users`.
2. Open the user record for the MID Server service account, which is the `Logged in user` from Step 1.
3. Verify the following:

- The user has the `mid_server` role
- The user is active
- The user is not locked out
- The `User ID` matches the value in the `config.xml` file, case-sensitive

### 4. Update the password in the instance

{{< callout type="warning" >}}
Complete this step and the next step as quickly as possible. The MID Server goes down when you save the user record.
{{< /callout >}}

1. In the `Password` field, enter the new password. *Note: To be able to set the password you must fill in the password field from the list view*
2. Select `Update`.

### 5. Restart the MID Server

On the MID Server host, restart the MID Server service:

- Windows: use the Services control panel or MMC snap-in
- Linux: use the appropriate service command for your distribution

After the restart, the MID Server should reconnect and return to `Up` status.

### 6. Validate in ServiceNow

1. Return to `MID Server > Servers`.
2. Open the MID Server record and confirm the status has returned to `Up`.
3. Check that the latest check-in time is current.
4. Review the MID Server agent log if the server does not reconnect.

### 7. Test capability

Run a small Discovery or orchestration test to confirm the MID Server is processing jobs correctly.

## If Authentication Fails

Use this checklist:

- Confirm the `User ID` in ServiceNow matches the `config.xml` value exactly
- Verify the password was entered correctly in both the host file and the ServiceNow user record
- Review the MID host logs for exact login or XML parser errors

## Appendix

- [ServiceNow knowledge article: MID Server password update guidance](https://noderegister.service-now.com/kb?id=kb_article_view&sysparm_article=KB0746702)
- [ServiceNow MID Server Product Documentation](https://www.servicenow.com/docs/r/servicenow-platform/mid-server/mid-server-landing.html)
