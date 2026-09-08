---
title: "Add Placeholder Text in String Field"
---

## Overview

To add example or placeholder text to a field, create a Client Script that uses the `.placeholder` property on the field's control object. Placeholder text is purely cosmetic it is not treated as an actual value for the field. For example, a field with placeholder text is still enforced as mandatory, and the placeholder will not be submitted if the user leaves the field blank.

## Template

```javascript {linenos=table,linenostart=1}
var field = g_form.getControl('fieldName');
field.placeholder = "Placeholder text";
```

## Example

The following `onLoad` Client Script adds multi-line placeholder text to the `u_asset_s` field on the Hardware Disposal Order table, showing users the expected format for entering assets:

```javascript {linenos=table,linenostart=1,filename="Placeholder Field Text.js"}
function onLoad(){
    var field = g_form.getControl('u_asset_s');
    field.placeholder = "**Example Entry:**\nThinkStation P360 Ultra - RX58K2LP\nThinkPad P15v - GT91M4QZ";
}
```