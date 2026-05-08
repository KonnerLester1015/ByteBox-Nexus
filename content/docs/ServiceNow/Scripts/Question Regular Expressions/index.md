---
title: "Question Regular Expressions"
description: "Reusable regex patterns for validating ServiceNow question fields."
type: docs
prev: docs/servicenow/scripts
---

# ServiceNow Question Regular Expressions

A single reference page for common regex patterns you can reuse across question fields (like catalog item variables).

{{< callout type="info" >}}
**Note:** A helpful resource for testing and building regex patterns is [regex101.com](https://regex101.com/). It provides real-time feedback and explanations for your regex.
{{< /callout >}}

## How Question Regular Expressions Work

1. Create a record in **Service Catalog > Catalog Variables > Variable Validation Regex** with the regex and validation message.
2. Reference that regex record on a question field (for example, a catalog item variable).
3. User input is validated in real time and on submit using the provided message.

## Server Name Validation

Enforces server naming with only letters, numbers, and hyphens. No leading or trailing hyphens.

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  Server names often drift from naming standards and introduce invalid characters or whitespace.

  ### The Solution
  Apply this question regex to standardize server names across catalog items and forms.
  {{< /tab >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="ServerName.regex"}
^(?!-)[A-Za-z0-9-]{3,}(?<!-)$
```

  **Validation Message**
  Must be 3-15 chars (A-Z, 0-9, -), no leading/trailing hyphens or spaces

  **Note**
  This pattern enforces a minimum of 3 characters with no maximum. Change `{3,}` to `{3,15}` if you want a 15-character max. In my case the variable hasa "max_length=15" variable attribute that handles the max length, so I only enforce the minimum in the regex. Adjust as needed for your use case.
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - app-01
  - WEB01
  - db-prod-1

  **Invalid**
  - -web01
  - web01-
  - web 01
  - web_01
  {{< /tab >}}

{{< /tabs >}}

## Network Share Name Validation

Allows letters, numbers, underscores, and hyphens with no spaces.

{{< tabs >}}

  {{< tab name="Usage" >}}
  ### The Problem
  Network share names often include spaces or special characters that break scripts and automation.

  ### The Solution
  Apply this regex to allow only safe characters for share names and follow a consistent naming convention.
  {{< /tab >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="NetworkShareName.regex"}
^[A-Za-z0-9_-]{1,80}$
```

  **Validation Message**
  Must be 1-80 chars (A-Z, 0-9, _, -), no spaces or special characters
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - share
  - share_01
  - SHARE-ARCHIVE

  **Invalid**
  - share name
  - share!
  - share/01
  {{< /tab >}}

{{< /tabs >}}
