# Dynamic Description on Catalog Tasks
## Overview

This document describes a method for dynamically populating the **Short Description** and **Description** fields of ServiceNow catalog tasks. By using a templating system with variable substitution, reference lookups, and conditional logic, administrators can create a more flexible and informative user experience without requiring manual input for every task.

The process is driven by two Business Rules that parse a template attached to a catalog item. This allows for automated population of task descriptions based on:

- **Variable Substitution (`VAR`)**: Directly inserts the value of a variable from the catalog item.
- **Reference Lookups (`REF`)**: Performs a GlideRecord lookup to retrieve the display value of a referenced record.
- **Conditional Logic (`CONSW`)**: Allows for different templates to be used based on the value of a specific variable.

---

## Templates

The templating system supports two main types:

1. **Non-CONSW Templates**: Used for simple, direct variable substitutions.
2. **CONSW Templates**: Enables conditional logic to switch between different templates based on a variable's value.

### Non-CONSW Template Example

This template is for a simple access request, where values are populated directly from variables.

**Short Description Template:**

```plain
Request for VAR.access_level access to VAR.select_requested_folder_path.
```

**Description Template:**

```plain
Please provide REF.requested_for(sys_user) VAR.access_level access to the folder path at VAR.select_requested_folder_path.
Provided Justification: VAR.Description.
```

**Example Output:**

- Short Description: `Request for change access to T:\IT\SysLogs.`
- Description: `Please provide John Doe change access to the folder path at T:\IT\SysLogs. Provided Justification: I am needing access to this folder because I am a new engineer team member.`

### CONSW Template Example

The `CONSW` (Conditional Switch) template allows you to use different descriptions based on the value of a variable. In this example, a variable named `action` determines which of the four templates is used.

**Short Description Template:**

```plain
CONSW.action(new_server_virtual_machine_request:1,change_to_existing_server_virtual_machine:2,server_migration:3,decom_VM:4)

1)Provision Server: "VAR.name"  
2)Modify Server: "VAR.what_is_the_name_of_the_server_you_are_requesting_a_change_for"  
3)Migrate Server: "VAR.name_of_server_you_are_migrating_from"  
4)Decommission Server: "VAR.server_to_be_decommissioned"  
$)
```

**Description Template:**

```plain
CONSW.action(new_server_virtual_machine_request:1,change_to_existing_server_virtual_machine:2,server_migration:3,decom_VM:4)

1)A request has been submitted to provision a new VAR.server_type server named "VAR.name" with the VAR.operating_system operating system.  
The server is needed by VAR.date_needed_by.

2)A change to the "VAR.what_is_the_name_of_the_server_you_are_requesting_a_change_for" server is being requested.  
The requested change is described as: VAR.what_is_the_change_being_requested.  
Justification for this change is: VAR.justification.  
This change is needed by VAR.date_needed_by.

3)A request has been submitted to migrate the server "VAR.name_of_server_you_are_migrating_from" to "VAR.name_of_server_you_are_migrating_to".  
The migration will include the following: VAR.what_needs_to_be_migrated_services_accounts_printers_etc.  
This migration is needed by VAR.date_needed_by.

4)A request has been submitted to decommission the server "VAR.server_to_be_decommissioned".  
The reason for decommissioning is: VAR.reason_for_decommission.  
The expected date of decommission is VAR.decommission_date.  

$)
```

**Example Output:**

- Short Description: `Provision Server: "MyNewServer01"`
- Description: `A request has been submitted to provision a new production server named "MyNewServer01" with the Windows Server 2022 operating system. The server is needed by 2025-05-30.`

**Example Output 2:**

- Short Description: `Decommission Server: "SQLProd01"`
- Description: `A request has been submitted to decommission the server "SQLProd01". The reason for decommissioning is: This is an old server that has been replaced. The expected date of decommission is 2025-05-27.`

## Technical Implementation

This process is powered by two Business Rules that run on the sc_task table before a record is inserted. The dynamic description will only be applied if the description field is empty, which means any description set by a flow or another process will not be overwritten.

**Note:** Only one CONSW block per template is currently supported.

**Handling Missing Values:**

If a variable (`VAR`) or reference (`REF`) value cannot be resolved, the system replaces it with a blank string. This ensures that tasks are still created without errors, but it may result in empty sections in the description. For example:
Template: `Please provide VAR.access_level access.`
Output (if `access_level` is empty): `Please provide access`.
To avoid unclear task descriptions, ensure required variables are validated before submission.

{{< tabs >}}

  {{< tab name="Autofill Description Script" >}}

```javascript {linenos=table,linenostart=1,filename="Autofill Description.js"}
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

{{< tab name="Autofill Short Description Script" >}}
This Business Rule mirrors the logic of the description script but is specifically for the short description field. It uses the same conditional logic to select the correct template and then performs variable and reference substitutions.

```javascript {linenos=table,linenostart=1,filename="Autofill Short Description.js"}
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

{{< /tabs >}}

