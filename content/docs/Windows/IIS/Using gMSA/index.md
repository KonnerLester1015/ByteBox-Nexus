---
title: Using Group Managed Service Accounts
type: docs
sidebar:
  open: false
---

To set up an IIS site to use a Group Managed Service Account (gMSA), follow these steps:

## Overview

Let's set an example where we have a Windows Server 2019 machine named `WEB01` and we have multiple IIS sites running; in particular we have one called `MySite` that we want to have access to a network share. To do this we will create a gMSA called `gmsa-mysite` and assign it to the application pool of `MySite`.

{{< callout >}}
  Technically you could just set the computer object to have permissions to the share, but this is not a good practice. Using a gMSA allows you to have a dedicated account for the application pool that can be managed and rotated automatically by Active Directory. It also means if you ever need to move the application pool to another server, you can just add that server to the security group and it will work without having to change any passwords or permissions.
{{< /callout >}}

## Procedure

1. **Create the Security Group.**
We will create a security group in Active Directory that will contain the IIS server(s) that will be using the gMSA.

```PowerShell
New-ADGroup -Name "gmsa-mysite-group" -GroupScope Global -GroupCategory Security -Path "OU=Groups,DC=yourdomain,DC=com"
```

2. **Add the IIS server to the security group.**

```PowerShell
Add-ADGroupMember -Identity "gmsa-mysite-group" -Members "WEB01$"
```

3. **Create gMSA.**
Now we will create the gMSA and specify that only members of the security group we just created can retrieve the password for the gMSA.

```PowerShell
New-ADServiceAccount -Name gmsa-mysite -DNSHostName gmsa-mysite.yourdomain.com -PrincipalsAllowedToRetrieveManagedPassword "gmsa-mysite-group"
```

4. **Grant Permissions to the gMSA.**
Ensure that the gMSA has the necessary permissions to access the network share. You can do this by adding the gMSA to the appropriate security group or directly granting it access to the share.

5. **Reboot or purge the Kerberos ticket cache on IIS server.**
After creating the gMSA, you may need to reboot the IIS server or purge the Kerberos ticket cache to ensure that the server is aware of its new security group and gMSA membership. You can purge the Kerberos ticket cache by running the following commands in an elevated PowerShell prompt:

```PowerShell
klist -li 0x3e7 purge
gpupdate /force
```

6. **Test the gMSA account on the IIS server.**
Before configuring the application pool, test that the gMSA can be used on the IIS server by running:

```PowerShell
Test-ADServiceAccount -Identity gmsa-mysite
```

You should see a result of `True`, indicating that the gMSA password can be retrieved and used on the IIS server.

7. **Configure the Application Pool**
Open IIS Manager, navigate to the application pool for `MySite`, and change the identity to use the gMSA. In the "Advanced Settings" of the application pool, set the "Identity" to "Custom account" and enter the gMSA in the format `domain\\gmsa-mysite$`.

{{< callout type="warning" >}}
  Note that you need to include the trailing `$` when specifying the gMSA account in IIS.
{{< /callout >}}

## References
- [Configure gMSA](https://blog.admindroid.com/configure-managed-service-accounts-in-active-directory/)
