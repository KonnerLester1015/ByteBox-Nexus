---
title: "Question Regular Expressions"
description: "Reusable regex patterns for validating ServiceNow question fields."
type: docs
prev: docs/servicenow/scripts
---

A single reference page for common regex patterns you can reuse across question fields (like catalog item variables).

{{< callout type="info" >}}
**Note:** A helpful resource for testing and building regex patterns is [regex101.com](https://regex101.com/). It provides real-time feedback and explanations for your regex.
{{< /callout >}}

## How Question Regular Expressions Work

1. Create a record in **Service Catalog > Catalog Variables > Variable Validation Regex** with the regex and validation message.
2. Reference that regex record on a question field (for example, a catalog item variable).
3. User input is validated in real time and on submit using the provided message.

## Email Address Validation

Validates a single email address with common username and domain rules.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="EmailAddress.regex"}
^(?![\.])[a-zA-Z0-9!#$%*\/?|^{}\`~&'\+\-=_.]+(?<![.]+)@(?![\.-])[a-zA-Z0-9-_\.]+(\.[a-zA-Z0-9_]+)$
```

  **Explanation**
  Validates a single email by checking a safe local part, one @, and a dot-separated domain.

  **Validation Message**
  Not a valid email
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - user@example.com
  - first.last+tag@sub.domain.net

  **Invalid**
  - .user@example.com
  - user@-domain.com
  - user@domain
  {{< /tab >}}

{{< /tabs >}}

## IPv4 Address Validation

Accepts valid IPv4 addresses only.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="IPv4Address.regex"}
^\s*(((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?))\s*$
```

  **Explanation**
  Matches four dot-separated octets, each constrained to 0-255.

  **Validation Message**
  Not a valid IPv4 address
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - 192.168.10.5
  - 10.0.0.25

  **Invalid**
  - 256.1.1.1
  - 10.0.0
  {{< /tab >}}

{{< /tabs >}}

## IPv6 Address Validation

Accepts valid IPv6 addresses only.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="IPv6Address.regex"}
^\s*((([0-9A-Fa-f]{1,4}:){7}([0-9A-Fa-f]{1,4}|:))|(([0-9A-Fa-f]{1,4}:){6}(:[0-9A-Fa-f]{1,4}|((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){5}(((:[0-9A-Fa-f]{1,4}){1,2})|:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){4}(((:[0-9A-Fa-f]{1,4}){1,3})|((:[0-9A-Fa-f]{1,4})?:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){3}(((:[0-9A-Fa-f]{1,4}){1,4})|((:[0-9A-Fa-f]{1,4}){0,2}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){2}(((:[0-9A-Fa-f]{1,4}){1,5})|((:[0-9A-Fa-f]{1,4}){0,3}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){1}(((:[0-9A-Fa-f]{1,4}){1,6})|((:[0-9A-Fa-f]{1,4}){0,4}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(:(((:[0-9A-Fa-f]{1,4}){1,7})|((:[0-9A-Fa-f]{1,4}){0,5}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:)))(%.+)?\s*$
```

  **Explanation**
  Matches standard IPv6 forms with optional compressed groups and embedded IPv4 where allowed.

  **Validation Message**
  Not a valid IPv6 address
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - 2001:0db8:85a3:0000:0000:8a2e:0370:7334
  - fe80::1

  **Invalid**
  - 2001:db8:85a3::8a2e::7334
  - 12345::abcd
  {{< /tab >}}

{{< /tabs >}}

## Network Share Name Validation

Allows letters, numbers, underscores, and hyphens with no spaces.

Network share names often include spaces or special characters that break scripts and automation. Apply this regex to allow only safe characters for share names and follow a consistent naming convention.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="NetworkShareName.regex"}
^[A-Za-z0-9_-]{1,80}$
```

  **Explanation**
  Restricts to 1-80 characters and allows only letters, numbers, underscores, and hyphens.

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

## Phone Number Validation (US)

Validates 10-digit US phone numbers with optional country code and common separators.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="PhoneNumber.regex"}
^(\+\d{1,2}\s?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}$
```

  **Explanation**
  Allows optional country code, optional parentheses, and common separators for a 10-digit US number.

  **Validation Message**
  Please enter a valid phone number
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - 5551234567
  - (555) 123-4567
  - +1 555-123-4567

  **Invalid**
  - 555-12-3456
  - 555-abc-1234
  {{< /tab >}}

{{< /tabs >}}

## Server Name Validation

Enforces server naming with only letters, numbers, and hyphens. No leading or trailing hyphens.

Server names often drift from naming standards and introduce invalid characters or whitespace. Apply this question regex to standardize server names across catalog items and forms.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="ServerName.regex"}
^(?!-)[A-Za-z0-9-]{3,}(?<!-)$
```

  **Explanation**
  Requires at least 3 characters, allows letters/numbers/hyphens, and blocks leading or trailing hyphens.

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

## URL Validation

Matches URLs with http, https, ftp, or a leading www.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="URL.regex"}
(((ftp|http|https):\/\/)|(www\.))([-\w\.\/#$\?=+@&%_:;]+)
```

  **Explanation**
  Accepts URLs starting with a scheme or www, then matches common URL-safe characters.

  **Validation Message**
  Not a valid URL
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - https://example.com
  - http://example.com/path?x=1
  - www.example.com

  **Invalid**
  - example.com
  - htp://example.com
  {{< /tab >}}

{{< /tabs >}}

## US Zip Code Validation

Accepts 5-digit ZIP codes and ZIP+4.

{{< tabs >}}

  {{< tab name="Regex" >}}
  **Regex**
  ```text {linenos=table,linenostart=1,filename="USZipCode.regex"}
^[0-9]{5}(?:-[0-9]{4})?$
```

  **Explanation**
  Matches a 5-digit ZIP with an optional 4-digit extension.

  **Validation Message**
  Incorrect zip code
  {{< /tab >}}

  {{< tab name="Sample Values" >}}
  **Valid**
  - 90210
  - 90210-1234

  **Invalid**
  - 9021
  - 90210-123
  {{< /tab >}}

{{< /tabs >}}
