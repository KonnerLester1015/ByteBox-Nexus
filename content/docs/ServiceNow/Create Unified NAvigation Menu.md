---
title: "Create Unified Navigation Menu"
---
By default, ServiceNow includes the `All`, `Favorite`, `History`, `Workspaces`, and `Admin` menus in the Now Experience Framework. However, custom menus can be created to cater to different user types working within the platform side of the product (not the Service Portal).

For instance, you can create custom menus for Approvers or Fulfillers to include links to dashboards or tables relevant to their roles. You can also create custom menu items with specific access limitations. For example, a "Fulfillers" menu could include links to common tables like the task table and submenus for each assignment group with unique links, such as a Computer Technician group needing access to the CMDB to modify computer information.

The primary purpose of Unified Menus is to provide user groups with easy access to necessary links without the need to manually favorite them, aiding new hires and streamlining access to custom applications.

Below are the steps to create a custom Unified Navigation Menu called "Approver." This menu will only be visible to users with the `admin` role and will contain links to administrative items such as Notifications, System Logs, and System Properties.

## Procedure
### Create Menu
1. Log in to your instance.
2. Navigate to `All -> Now Experience Framework -> Unified Navigation Menu Configuration`.

{{< callout type="info" >}}
  This will display all Out-of-box menus and any custom menus.
{{< /callout >}}

3. Click **New**.
4. Enter "Approver" in the **Name** field.
5. Click the pencil icon on the **Visible Role** field to show the slush bucket.
6. Search for and add the business_stakeholder role.

{{< callout type="info" >}}
  This role should be assigned to all approvers. Any user without this role will be unable to view the menu (excluding Admins).
{{< /callout >}}

7. Click Done.
8. Click Submit.
   
{{< callout type="warning" >}}
  You may be redirected to the Unified Navigation Menu list. If so, click back into the Approver record.
{{< /callout >}}

9. In the "Unified Navigation Configurable Menu Items" related list, click New.
10. Select Module in the Type field.
11. Enter "My approvals" in the Module field.
12. Click Submit.

The **Approver** menu will now be visible and, when clicked, will show the "My approvals" module.