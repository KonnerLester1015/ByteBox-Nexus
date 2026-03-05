---
title: "Custom Transformation"
---

## Overview

Maintaining consistent data formatting is essential for a healthy CMDB and reference tables such as Location. Variations in casing, delimiters, and spacing (for example x1,y5, x1 y5, or X1Y5) can introduce reporting inconsistencies and complicate automation.

ServiceNow’s Field Transformation framework provides a structured method to normalize data before it is stored. Transformations allow administrators to automatically convert multiple representations of a value into a single standardized format, improving searchability and reporting accuracy.

This guide demonstrates how to implement a custom transformation that standardizes coordinate values into a consistent format such as:

- X1 Y5

The transformation handles several variations including:

- x1,y5
- x1 y5
- x1y5
- mx1 y1

All matching patterns are normalized into a consistent uppercase, space delimited coordinate format.

---

## Architecture

A transformation is composed of three primary records:

| Component                | Table                | Purpose                                                                 |
|--------------------------|----------------------|-------------------------------------------------------------------------|
| Transform Definition     | `fn_transform_def`   | Contains the transformation logic.                      |
| Transformation    | `fn_transform_config`| Specifies the table, field, and mode (Active, Test, or Off) for the transformation. |
| Transforms                | `fn_transform`       | Determines which records should be processed by the transformation.      |

This layered approach separates logic, configuration, and scope, allowing transformations to be reused across multiple fields or tables.

## Transform Definition

The Transform Definition contains the core logic used to modify field values. This script behaves similarly to a reusable function that receives the current field value and returns the transformed result.

ServiceNow includes several out-of-box definitions such as Trim, Round, and Change Case, but custom definitions can be created to support organization specific formatting rules.

### Implementation: Coordinate Formatter

This custom definition uses a Regular Expression (Regex) to detect coordinate patterns and enforce a standardized uppercase format with a single space delimiter.

In development I heavliy used [Regex101.com](https://regex101.com/) to test and validate the regular expression, ensuring it accurately targeted only the intended coordinate patterns.

**Script:**
```javascript {filename="CoordinateCorrections,js",linenos=table}
/**
 * Custom Transformation for Coordinate Formatting
 * Detects: x0,y1 | mx1 y1 | x5y2 | MXY2x2 | x3, t3
 * Transforms to: X0 Y1 | MX1 Y1 | X5 Y2 | MXY2 X2 | X3 T3
 */

function(variables, value, parameters) {
    // Cast to string to ensure JS string methods work
    var val = '' + value;
    
    // 1. Matches 1-3 letters + digits
    // 2. Matches zero or more commas/spaces [, ]*
    // 3. Matches the second set of 1-3 letters + digits
    var coordRegex = /\b([a-z]{1,3}\d+)[, ]*([a-z]{1,3}\d+)\b/i;

    var match = coordRegex.exec(val);

    if (match) {
        try {
            // match[1] is the first coordinate, match[2] is the second
            // We force a space between them and convert to uppercase
            var formattedCoord = (match[1] + ' ' + match[2]).toUpperCase();
            
            // Replaces the coordinate part and returns the cleaned string
            return val.replace(match[0], formattedCoord).trim();
            
        } catch (e) {
            parameters.put('errorMessage', 'Coordinate transform failed: ' + e.message);
            return value;
        }
    }

    // If no pattern matches (e.g. "Main Office"), return original value
    return value;
}
```

---

## Transformation

The Transformation record determines where the transformation logic is applied within the platform.

| Field      | Description                                      | Example              |
|------------|--------------------------------------------------|----------------------------|
| Name       | Descriptive name for the configuration           | Coordinate Corrections     |
| Table      | Target table containing the field                | Location [cmn_location]    |
| Field      | Field whose values will be transformed           | Name                      |
| Status     | Determines whether the transformation runs       | Active                    |

When a record is evaluated, the system executes the definition script and replaces the field value with the returned result.

## Transform

The Transform record controls which records are evaluated by the transformation.

For this implementation, a transform named **All Locations** was created.

Configuration details:

- **Name:** All Location
- **Def:** Coordinate Formatter
- **Condition:** (None)
- **Active:** True

Leaving the condition builder empty ensures the transformation is evaluated against every record in the Location table.

{{< callout >}}
  If more granular control is needed, conditions can be added to limit execution to specific records (for example, only newly created locations or specific location types).
{{< /callout >}}

---

## Why Use Transformations Instead of Business Rules?

While similar functionality could be implemented using **Before Business Rules**, the transformation framework provides several advantages.

1. **Purpose-built normalization**

Transformations are specifically designed for data standardization and normalization, reducing the need for custom scripting across multiple rules.

2. **Reusable logic**

Transform definitions can be reused across different fields or tables without duplicating scripts.

3. **Centralized visibility**

Transformation records provide a centralized view of which fields are automatically normalized within the platform.

## Result

After the transformation is applied, all coordinate style location names are automatically normalized.

| Input     | Transformed Value |
|-----------|--------------|
| x1,y5     | X1 Y5        |
| x1y5      | X1 Y5        |
| mx1 y1    | MX1 Y1       |
| X3, t3    | X3 T3        |

This ensures consistent formatting for:

- Catalog selection variables
- Reporting and filtering
- Automated workflows relying on exact string matches
