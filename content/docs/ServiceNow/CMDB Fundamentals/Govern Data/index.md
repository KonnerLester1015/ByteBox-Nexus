---
title: "CMDB Data Governance"
---

## Overview

Effective governance is essential to maintaining a reliable and trustworthy Configuration Management Database (CMDB). Without proper governance, CMDB data can quickly become incomplete, inconsistent, or outdated leading to inaccurate reporting, failed automations, and poor operational decisions.

ServiceNow provides built-in capabilities to monitor and enforce data quality through **CMDB Health**, which evaluates the CMDB across three key dimensions:

- **Completeness** – Are required and recommended fields populated?
- **Compliance** – Do CIs meet defined standards and policies?
- **Correctness** – Is the data accurate, current, and properly related?

Together, these metrics ensure that CMDB data is not only present, but also usable and reliable for IT operations.

---

## CMDB Health Scorecards

### I. Completeness

The **Completeness** scorecard measures how well Configuration Items (CIs) are populated with required and recommended data.

#### Key Metrics

- **Required Fields**
  - Defined by setting the *Mandatory* attribute at the dictionary level
  - Measures the percentage of CIs missing required data

- **Recommended Fields**
  - Defined in the CI Class Manager
  - Measures the percentage of CIs missing suggested (but not enforced) data

#### Governance Considerations

Completeness ensures that critical operational data is available for:

- Incident and change routing  
- Ownership and accountability  
- Asset tracking and reporting  

#### Key Questions

- Are all required fields populated for each CI?  
- Are important non-discoverable fields (e.g., ownership, support group) filled in?  

---

### II. Compliance

The **Compliance** scorecard evaluates how well CIs align with defined governance standards and desired state configurations.

#### Key Metrics

- **Audit**
  - Compares actual CI values against expected values defined in audit templates
  - Can include both template based and scripted audits

#### Governance Considerations

Compliance ensures that:

- CIs adhere to organizational policies  
- Configurations meet security and operational standards  
- The CMDB reflects the intended state of the environment  

#### Key Questions

- Are CIs configured according to defined standards?  
- Do they meet desired state expectations?  

---

### III. Correctness

The **Correctness** scorecard validates the accuracy and integrity of CMDB data.

#### Key Metrics

- **Staleness**
  - Identifies CIs that have not been updated within a defined timeframe  
  - Based on the `sys_updated_on` field  
  - Default threshold: 60 days  

- **Orphaned CIs**
  - CIs without required relationships (e.g., no parent or dependency)  
  - Must be defined through custom orphan rules (By default there are no orphan rules in the base system)

- **Duplicate CIs**
  - Identifies multiple records representing the same CI  
  - Based on identification rules

#### Governance Considerations

Correctness ensures that:

- Data reflects the current state of the environment  
- Relationships are meaningful and intact  
- Duplicate or unused records do not degrade CMDB quality  

#### Key Questions

- Are CIs up-to-date and accurate?  
- Are duplicates minimized?  
- Do CIs have valid relationships?  

---

## Data Governance Processes

### Duplicate Remediation

Duplicate records significantly impact CMDB reliability and reporting. ServiceNow provides two primary methods to remediate these duplicates:

1. **Duplicate CI Remediation**
This guided process assists in resolving duplicates by offering options to merge, delete, or retain records. The workflow involves reviewing duplicate CIs, comparing their attributes, and selecting the best action to maintain data integrity. A **Main CI** is designated based on specific criteria: the most recently updated record, the record with the most existing relationships, or the oldest record (created first).

2. **De-duplication Templates**
Templates enable bulk resolution by defining identification criteria and automated actions, such as merging or deleting. These can be assigned to open de-duplication tasks or published to run on a scheduled basis, providing an efficient, automated way to manage duplicates at scale.

---

### Reclassification

Reclassification is the process of changing the class of a Configuration Item (CI) when it has been misclassified or when new information becomes available.

Types of Reclassification:

- **Upgrade**
  - Moves a CI to a more specific (child) class  
  - Example: `Server → Windows Server`  

- **Downgrade**
  - Moves a CI to a more general (parent) class  
  - May result in loss of attributes not present in the parent class  
  - Example: `Windows Server → Server`

- **Switch**
  - Moves a CI to a different branch in the class hierarchy
  - May result in attribute data loss
  - Example: `Linux Server → Windows Server`  

#### Global System Properties

These properties control whether automatic reclassification occurs:

- `glide.class.upgrade.enabled`  
- `glide.class.downgrade.enabled`  
- `glide.class.switch.enabled`  

Setting these to **false**:

- Prevents automatic reclassification  
- Generates reclassification tasks instead  

You can further refine behavior using these properties to allow data updates while specifically blocking the class change:

- `glide.identification_engine.update_without_switch_enabled`  
- `glide.identification_engine.update_without_downgrade_enabled`  
- `glide.identification_engine.update_without_upgrade_enabled`  

When set to **true**:

- CI attributes are updated  
- Reclassification is skipped  

---

#### Reclassification Restriction Rules

Reclassification restriction rules allow control at the **class level**, rather than globally.

- Restrict downgrade and switch actions between specific classes 
- Prevent unwanted data loss during reclassification  
- Allow attribute updates while blocking class changes  

---

## Summary

CMDB data governance is centered around ensuring data quality, consistency, and reliability. By leveraging CMDB Health and supporting processes:

- **Completeness** ensures required data is populated  
- **Compliance** ensures adherence to standards  
- **Correctness** ensures data accuracy and integrity  

Supporting processes such as duplicate remediation and reclassification controls further strengthen governance by maintaining clean, accurate, and well-structured data.

A mature CMDB requires continuous monitoring and enforcement of these principles to remain a trusted source of truth for the organization.