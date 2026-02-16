---
title: "Uninstalling Applications"
---

## Overview

Uninstalling an application in ServiceNow requires elevated administrative privileges. While the process is generally straightforward, complications can arise when managing the underlying data structures, specifically regarding the "Retain tables and data" configuration.

This guide covers the standard uninstallation procedure and provides a solution for scenarios where the system prevents the automated removal of application tables.

## Standard Uninstallation Procedure

1. Log in to your instance with the **admin** role.
2. Navigate to **All > Application Manager**.
3. Select the **Installed** tab to view applications currently residing on the instance.
4. Locate and click on the specific application you wish to remove.
5. In the **Quick Actions** section, click **Uninstall**.

### Data Retention Configuration
Upon clicking Uninstall, a confirmation dialog will appear listing all associated files, including table records, fields, and cross-scope definitions.

* **Retain tables and data:**:
    * **Enabled (Checked):** Preserves the physical tables and their records in the database after the application logic is removed.
    * **Disabled (Unchecked):** Permenantly drops the tables and deletes all associated data from the database.

{{< callout type="warning" >}}
  In certain instances, the **Retain tables and data** checkbox may be greyed out and locked in the "Enabled" state. This typically occurs when the application contains **Import Set Tables**. ServiceNow enforces this to prevent the accidental corruption of data pipelines. Refer to the Troubleshooting section below for a workaround.
{{< /callout >}}

![Uninstall Application Uninstall Confirmation](/images/UninstallApplicationUninstallConfirmation.png)

6. Configure the retention checkbox as desired.
7. Click **OK** to begin the uninstallation process.
8. **Verification:** Once the process completes, verify the removal by checking the **Installed** tab or navigating to **System Definition > Tables** to ensure the schema has been handled according to your selection.

![Uninstall Application Success Confirmation](/images/UninstallApplicationSuccessConfirmation.png)

---

## Troubleshooting: Greyed Out Retention Checkbox

If the application includes an Import Set Table, you must manually decommission the table before the system will allow you to uninstall the application and its data simultaneously.

### Manually Deleting Import Set Tables
1. Navigate to **System Definition > Tables**.
2. Filter the list where the **Application** matches the scope you are uninstalling.
3. Locate tables where the **Extends table** (Super class) is `Import Set Row` (`sys_import_set_row`).
4. Open the table record.
5. Click the **Delete** UI Action.
6. When prompted, type `delete` in the confirmation field and click **OK**.
7. Return to the Application Manager and restart the uninstallation; the **Retain tables and data** checkbox will now be editable.