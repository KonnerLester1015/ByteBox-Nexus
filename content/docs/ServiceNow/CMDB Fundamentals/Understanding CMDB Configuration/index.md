---
title: "Understanding CMDB Configurations"
---

## Executive Summary

ServiceNow provides several configuration management tools designed to ensure that the **Configuration Management Database (CMDB)** remains accurate, reliable, and compliant. These tools (**CI Class Manager**, **Identification and Reconciliation Engine (IRE)**, **CMDB 360 / Multisource CMDB**, and **Dynamic Reconciliation Rules**) work together to govern how Configuration Items (CIs) are created, identified, updated, and maintained.

This document explains how these components interact to manage CMDB data quality, prevent duplication, control updates from multiple data sources, and ensure the CMDB accurately reflects the IT environment.

---

## CI Class Manager

### Overview

The **CI Class Manager** provides a centralized interface for managing CMDB class hierarchies, attributes, rules, and health metrics. It enables CMDB administrators to view, edit, and configure the structure and behavior of CI classes.

**Key Benefits:**

- Centralized management of the CMDB class hierarchy  
- Visibility into parent-child relationships between classes  
- Control over class definitions, attributes, and associated rules  
- Support for class extensibility and customization  
- Tools to improve CMDB data accuracy and health  

### Basic Info Tab

Displays high level information about the selected CI class, including:

- **Display Name:** Name of the class as shown in the CMDB.  
- **Principal Class:** Restricts which CIs appear in ITSM forms such as Incident or Change.  
- **Table Name:** The database table associated with the CI class.  
- **Managed By Group:** Defines which group is responsible for managing all CIs in the class.  
- **Default Product Model:** Sets a default model for standardization.  
- **Icon:** Associates a visual icon with the class.
- **Extensible Flag:** Determines whether the class can be extended. 

### Attributes Tab

Manages class specific attributes that describe a CI:

- View, add, or modify attributes.  
- Set attributes as **mandatory** or **optional**.  
- Extend attributes from parent classes or define unique ones for a subclass.  

### Identification Rules Tab

Defines how CIs are uniquely identified within the CMDB. Identification rules ensure that each CI record represents a distinct item.

**Options:**

- **Allow Null Attributes:** Enables matching when at least one identifying attribute is populated.  
- **Allow Fallback to Parent’s Rule:** Uses parent class identification logic if a direct match is not found.  

### Reconciliation Rules Tab

Determines how updates from multiple data sources are merged and prioritized. Each rule specifies which discovery sources can update particular CI attributes and their precedence order.

### Health Tab

The **Health** tab allows configuration of metrics and KPIs that measure CMDB quality. Health is evaluated using three dimensions:

- **Completeness:** Ensures all required fields are populated.  
- **Compliance:** Verifies that data adheres to defined audits.  
- **Correctness:** Detects duplicate, orphaned, or stale CIs.  

---

## Identification and Reconciliation Engine (IRE)

### Overview

The **Identification and Reconciliation Engine (IRE)** is the framework that processes payloads from various discovery and integration sources to maintain data accuracy in the CMDB. It ensures consistent identification, reconciliation, and updating of CI records from multiple data sources.

**Used to:**

1. Reduce duplicate CIs  
2. Prevent conflicting updates to attributes  
3. Stop attribute flapping between sources  

### Core Processes

1. **Identification:** Determines whether a discovered CI already exists or should be created. This step uses identification rules to ensure each CI is uniquely recognized across all sources.  
2. **Reconciliation:** Controls which data sources are allowed to update which CI attributes. Reconciliation rules define data source priority and prevent unauthorized updates.  
3. **Deduplication:** Groups duplicate CIs into de-duplication tasks for review and cleanup.  
4. **Reclassification:** Reassigns CIs to a different class if their type changes (e.g., upgrading from a generic server to a Windows Server).  

{{< callout type="info" >}}
  **Preventing Unauthorized Inserts:** Administrators can configure **IRE Data Source Rules** to block inserts from certain sources, ensuring that only trusted systems can add new CIs.
{{< /callout >}}

---

## Identification and Reconciliation Rules

### Identification Rules

Identification rules ensure that each Configuration Item is uniquely represented in the CMDB.

**Benefits:**

- Maintain data integrity by preventing duplicate or overloaded CIs  
- Provide consistency across data sources  
- Automate CI recognition and reduce manual effort  

**Common Issues Addressed:**

- **Duplicate CIs:** Multiple records representing a single item.  
- **Overloaded CIs:** One record incorrectly representing multiple distinct items.  

### Reconciliation Rules

Reconciliation rules determine which data source is the authoritative source for updating specific attributes or CI classes.

**Key Benefits:**

- Maintain data accuracy and reliability  
- Resolve conflicts between discovery sources  
- Control update precedence and prevent overwrites  

Reconciliation occurs **after identification** but before the CI is updated in the CMDB. It applies only to updates, not to new CI creation.

---

## Data Refresh Rules

**Data Refresh Rules** define how long a higher priority data source has to update a CI before lower priority sources are allowed to make changes.  

If the higher priority source does not perform an update within the configured interval, lower priority sources can refresh the data to ensure that the CMDB remains current.

These rules are configured in the **Reconciliation Rules** tab within the CI Class Manager.

---

## CMDB 360 / Multisource CMDB

### Overview

The **CMDB 360 (Multisource CMDB)** enhances visibility into CI attribute updates from multiple data sources. While reconciliation rules allow only one discovery source to update a CI, CMDB 360 stores information from all sources including those whose updates were rejected.

This data is stored in the **CMDB MultiSource Data [cmdb_multisource_data]** table, allowing administrators to query and report on all attribute values provided by various data sources.

### Historical Context

Before the introduction of CMDB 360 (Paris release), ServiceNow only tracked the last source that updated a CI in the **Data Source Histories [cmdb_datasource_last_update]** table.  
CMDB 360 now captures all incoming data, providing a complete historical and comparative view of multisource updates accessible from CI records, CI Class Manager, and CMDB Workspace.

{{< callout type="warning" >}}
  CMDB 360 is **not** enabled by default in the base system. It must be activated to take advantage of multisource data tracking.
{{< /callout >}}

---

## Dynamic Reconciliation Rules

### Overview

**Dynamic Reconciliation Rules** extend the static reconciliation process by analyzing Multisource CMDB data to determine the most suitable value when multiple sources report data. They evaluate attribute values based on logical conditions rather than static priorities.

{{< callout type="info" >}}
  When both static and dynamic reconciliation rules exist, **dynamic rules take precedence.**
{{< /callout >}}

### Supported Rule Types

- **First Reported:** Uses the first value reported by any source.  
- **Most Reported:** Selects the value reported by the majority of sources.  
- **Last Reported:** Uses the most recently reported value.  
- **Largest Value:** Selects the highest numerical or measurable value.  
- **Smallest Value:** Selects the lowest numerical or measurable value.  

### Advantages

- Ensures accuracy when multiple valid values exist  
- Automates decision making for conflicting attribute data  
- Reduces manual review and improves CMDB reliability  

---

## Relationships and Dependencies

**Dependent Relationships** describe how Configuration Items rely on or interact with one another. They play a crucial role in impact analysis and service dependency mapping.

**Key Functions:**

- **Dependency Mapping:** Identifies interdependencies among CIs (e.g., an application depends on a specific server).  
- **Impact Analysis:** Evaluates downstream effects when a CI fails or is modified.  
- **Lifecycle Management:** Ensures that dependent CIs are handled correctly during decommissioning or replacement.  

---

## CMDB Health and Governance

Maintaining CMDB health ensures the database remains a trustworthy source of truth. ServiceNow provides built-in dashboards and KPIs to monitor quality.

**CMDB Health Dimensions:**

- **Completeness:** Required fields are populated.  
- **Compliance:** Data adheres to established policies and audits.  
- **Correctness:** Data accurately represents the real-world configuration, free of duplicates and stale entries.  

Regular audits, certification policies, and automated health scorecards help maintain ongoing governance and visibility into CMDB reliability.

---

## Key Takeaways

- The **CI Class Manager** is used to structure, define, and manage CI classes and their rules.  
- The **Identification and Reconciliation Engine (IRE)** ensures that CI data is identified, merged, and updated correctly.  
- **Reconciliation and Data Refresh Rules** control which data sources are authoritative and when updates can occur.  
- **CMDB 360 / Multisource CMDB** provides complete transparency into multisource attribute values.  
- **Dynamic Reconciliation Rules** improve accuracy by using intelligent value selection methods.  
- **CMDB Health Management** ensures data remains compliant, complete, and correct over time.

A well configured CMDB supports all downstream ITSM and ITOM processes, maintaining data integrity and operational efficiency across the organization.
