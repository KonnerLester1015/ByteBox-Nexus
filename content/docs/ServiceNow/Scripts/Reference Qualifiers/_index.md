---
title: Reference Qualifiers
type: docs
prev: docs/servicenow/scripts/
---

## Overview

Reference Qualifiers are scripts attached to reference fields and variables (`sys_dictionary` entries or catalog item variables) that dynamically restrict which records a user can pick. Instead of a static filter, the qualifier script runs each time the field is displayed and returns an encoded query, most commonly a `sys_idIN...` list, built from the current record's other field values.

This section covers patterns for building dynamic, interdependent reference qualifiers.

{{< cards >}}

{{< card link="combine-group-membership-for-a-qualifier/" title="Combine Group Membership for a Qualifier" icon="users" >}}

{{< /cards >}}
