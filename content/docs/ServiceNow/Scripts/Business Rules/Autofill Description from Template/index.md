---
title: "Autofill Description from Template"
---

## Overview

Catalog items can store a dynamic template string (in custom fields `u_dynamic_description_template` and `u_dynamic_short_description_template` on the catalog item) that describes how to build the `description` and `short_description` of a resulting RITM from its variables. Two **Business Rules** — running on `sc_req_item` — read that template, substitute in variable values, and write the result back onto the record.

The templates are written in a small text mini-language with three tokens:

- **`VAR.field`** — replaced with the raw value of the catalog variable named `field`.
- **`REF.field(table)`** — treats `field` as a reference variable (a sys_id) and replaces the token with that record's display value, looked up on `table`.
- **`CONSW.field(value1:1,value2:2,...)`** — a "conditional switch." It reads the current value of variable `field`, maps it to a numbered template block (`value1` maps to block `1`, `value2` to block `2`, etc.), and replaces the *entire* template with just that block before the `VAR`/`REF` substitution runs. Blocks are written inline as `1)...text...`, `2)...text...`, terminated by a trailing `$)`.

## Problem Statement

Different catalog items need very different description text, and a single item can even need different text depending on a variable's value (e.g. a "Server Request" item behaves differently for "new server" vs. "decommission"). Hardcoding this logic per catalog item in separate business rules would be unmaintainable or would need to have custom flows for each catalog item instead of being able to use an single flow for many items. Instead, the wording lives as data template string stored on the catalog item and one generic business rule engine parses and fills it in, so admins can add or edit request-type wording without touching script.

If no template is defined at all, both rules fall back to a plain variable (`current.variables.Description` for the description, `current.request_item.u_task_short_description` for the short description).

{{< tabs >}}

  {{< tab name="Description Business Rule" >}}

```javascript {linenos=table,linenostart=1,filename="Autofill Description.js"}
/**
 * Script Name: Autofill Description
 * Description: This script automatically fills the description field of a ServiceNow record based on a dynamic template defined in the catalog item.
 * 
 * Usage: This script is utilized within the ServiceNow instance in the context of a business rule.
 *      Example record:
 *         Name: Autofill Description
 *         Table: sys_script
 * 
 * Author: Konner Lester
 * Date Created: 05/13/2025
 * Last Modified: 05/13/2025
 * 
 */
(function executeRule(current, previous) {
    var template = current.request_item.cat_item.u_dynamic_description_template;

    if (template) {
        // Check for Conditional logic
        var conswMatch = template.match(/CONSW\.(\w+)\((.*?)\)/);
        if (conswMatch) {
            var conswField = conswMatch[1];
            var conswConditions = conswMatch[2]; 

            // Get the value of the Conditional field
            var conswValue = current.variables[conswField] ? current.variables[conswField].toString() : "";

            // Parse the conditions
            var conditionMap = {};
            conswConditions.split(",").forEach(function(condition) {
                var [value, templateId] = condition.split(":");
                conditionMap[value] = templateId;
            });

            // Determine the template to use based on the Conditional value
            var selectedTemplateId = conditionMap[conswValue];
            if (selectedTemplateId) {
                // Extract the corresponding template
                var templateRegex = new RegExp(`^${selectedTemplateId}\\)([\\s\\S]*?)(?=\\d\\)|\\$\\))`, 'm');
                var selectedTemplateMatch = template.match(templateRegex);
                if (selectedTemplateMatch) {
                    template = selectedTemplateMatch[1].trim(); // Use the selected template
                }
            } else {
                template = ""; // No matching template, fallback to empty
            }
        }

        // Replace variables in the template
        var description = template.replace(/(VAR|REF)\.(\w+)(?:\((\w+)\))?/g, function(match, type, variableName, tableName) {
            // Get the value of the variable from current.variables
            var variableValue = current.variables[variableName];

            if (type === "VAR") {
                // Handle regular variables
                return variableValue !== undefined ? variableValue : ""; // Replace with value or empty string if undefined
            } else if (type === "REF" && tableName) {
                // Handle reference variables
                if (variableValue) {
                    var gr = new GlideRecord(tableName);
                    gr.addQuery("sys_id", variableValue);
                    gr.query();
                    if (gr.next()) {
                        return gr.getDisplayValue(); // Return the display value of the referenced record
                    }
                }
                return ""; // Replace with an empty string if the reference is invalid
            }

            return ""; // Default to an empty string for unmatched cases
        });

        // Set the description field
        current.description = description;
    } else {
        current.description = current.variables.Description; // Fallback value
    }
})(current, previous);
```

{{< /tab >}}

  {{< tab name="Short Description Business Rule" >}}

```javascript {linenos=table,linenostart=1,filename="Autofill Short Description.js"}
/**
 * Script Name: Autofill Short Description
 * Description: This script automatically fills the short description field of a ServiceNow record based on a dynamic template defined in the catalog item.
 * 
 * Usage: This script is utilized within the ServiceNow instance in the context of a business rule.
 *      Example record:
 *         Name: Autofill Short Description
 *         Table: sys_script
 * 
 * Author: Konner Lester
 * Date Created: 05/13/2025
 * Last Modified: 05/13/2025
 * 
 */
(function executeRule(current, previous) {
    var template = current.request_item.cat_item.u_dynamic_short_description_template;

    if (template) {
        // Check for CONSW logic
        var conswMatch = template.match(/CONSW\.(\w+)\((.*?)\)/);
        if (conswMatch) {
            var conswField = conswMatch[1]; // The variable to evaluate (e.g., test)
            var conswConditions = conswMatch[2]; // The conditions and templates (e.g., new_server_virtual_machine_request:1,...)

            // Get the value of the CONSW field
            var conswValue = current.variables[conswField] ? current.variables[conswField].toString() : "";

            // Parse the conditions
            var conditionMap = {};
            conswConditions.split(",").forEach(function(condition) {
                var [value, templateId] = condition.split(":");
                conditionMap[value] = templateId;
            });

            // Determine the template to use based on the CONSW value
            var selectedTemplateId = conditionMap[conswValue];
            if (selectedTemplateId) {
                // Extract the corresponding template
                var templateRegex = new RegExp(`^${selectedTemplateId}\\)([\\s\\S]*?)(?=\\d\\)|\\$\\))`, 'm');
                var selectedTemplateMatch = template.match(templateRegex);
                if (selectedTemplateMatch) {
                    template = selectedTemplateMatch[1].trim(); // Use the selected template
                } 
            } else {
                template = ""; // No matching template, fallback to empty
            }
        }

        // Replace variables in the template
        var shortDescription = template.replace(/(VAR|REF)\.(\w+)(?:\((\w+)\))?/g, function(match, type, variableName, tableName) {
            // Get the value of the variable from current.variables
            var variableValue = current.variables[variableName];

            if (type === "VAR") {
                // Handle regular variables
                return variableValue !== undefined ? variableValue : ""; // Replace with value or empty string if undefined
            } else if (type === "REF" && tableName) {
                // Handle reference variables
                if (variableValue) {
                    var gr = new GlideRecord(tableName);
                    gr.addQuery("sys_id", variableValue);
                    gr.query();
                    if (gr.next()) {
                        return gr.getDisplayValue(); // Return the display value of the referenced record
                    }
                }
                return ""; // Replace with an empty string if the reference is invalid
            }

            return ""; // Default to an empty string for unmatched cases
        });

        // Set the short description field
        current.short_description = shortDescription;
    } else {
        current.short_description = current.request_item.u_task_short_description; // Fallback value
    }
})(current, previous);
```

{{< /tab >}}

  {{< tab name="Template Syntax Examples" >}}

Templates are stored as plain text on the catalog item. A few representative examples from the description and short description template libraries:

```text {linenos=table,linenostart=1,filename="Server Request Description Template"}
CONSW.test(new_server_virtual_machine_request:1,change_to_existing_server_virtual_machine:2,server_migration:3,decom_VM:4)

1)A request has been submitted to provision a new VAR.server_type server named "VAR.name" with the VAR.operating_system operating system. 
The server is needed by VAR.date_needed_by.

2)A change to the "VAR.what_is_the_name_of_the_server_you_are_requesting_a_change_for" server is being requested. 
The requested change is described as: VAR.what_is_the_change_being_requested. 
Justification for this change is: VAR.Justification. 
This change is needed by VAR.date_needed_by.

3)A request has been submitted to migrate the server "VAR.name_of_server_you_are_migrating_from" to "VAR.name_of_server_you_are_migrating_to". 
The migration will include the following: VAR.what_needs_to_be_migrated_services_accounts_printers_etc. 
This migration is needed by VAR.date_needed_by.

4)A request has been submitted to decommission the server "VAR.server_to_be_decommissioned". 
The reason for decommissioning is: VAR.reason_for_decommission. 
The expected date of decommission is VAR.decommission_date.

$)
```

```text {linenos=table,linenostart=1,filename="Login ID Description Template"}
CONSW.login_id_type(sia:1,temp:2,intern:3,guest_account:4,service_account:5)

1)A request has been submitted to create a SIA Login ID for VAR.first_name VAR.last_name, with a badge number of VAR.badge_number. The associate is part of the REF.department(cmn_department) department, and the assigned manager is REF.manager(sys_user).

2)A request has been submitted to create a Contractor Login ID for VAR.first_name VAR.last_name. The contractor's email is VAR.contractor_email_address, and the SIA Contact is REF.sia_contact(sys_user).

3)A request has been submitted to create an Intern Login ID for VAR.first_name VAR.last_name, who will be working in the REF.department(cmn_department) department. The intern's badge number is VAR.badge_number, and the assigned manager is REF.manager(sys_user).

4)A request has been submitted to create a Guest Account for VAR.first_name VAR.last_name, employed by VAR.company. The guest's contact email is VAR.email, and the SIA Contact for the account is REF.sia_contact(sys_user).

5)A request has been submitted to create a Service Account for the purpose of "VAR.what_is_this_account_being_used_for." The account's point of contact is REF.sia_contact(sys_user).

$)
```

```text {linenos=table,linenostart=1,filename="Server Request Short Description Template"}
CONSW.test(new_server_virtual_machine_request:1,change_to_existing_server_virtual_machine:2,server_migration:3,decom_VM:4)

1)Provision Server: "VAR.name"
2)Modify Server: "VAR.what_is_the_name_of_the_server_you_are_requesting_a_change_for"
3)Migrate Server: "VAR.name_of_server_you_are_migrating_from"
4)Decommission Server: "VAR.server_to_be_decommissioned"
$)
```

Templates with no `CONSW` block (like the Folder Permissions or Guest Wifi templates) skip straight to `VAR`/`REF` substitution against the whole string:

```text {linenos=table,linenostart=1,filename="Folder Permissions Description Template"}
Please provide REF.requested_for(sys_user) VAR.access_level access to the folder path at VAR.select_requested_folder_path.
Provided Justification: VAR.Description.
```

{{< /tab >}}

{{< /tabs >}}
