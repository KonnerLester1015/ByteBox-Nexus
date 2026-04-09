---
title: "Creating MID Server"
---

## Overview

A MID Server (Management, Instrumentation, and Discovery Server) is a Java based agent installed in your private network that enables ServiceNow to securely communicate with internal systems.

You should deploy a MID Server when ServiceNow needs to:

- Discover or map infrastructure that is not directly reachable from the internet
- Run orchestration activities against internal servers or endpoints
- Connect to internal data sources through IntegrationHub or custom integrations
- Keep traffic outbound only from your network to ServiceNow over HTTPS

## Prerequisites

Before setup, confirm the following:

- A ServiceNow admin account with access to MID Server records
- A host (Windows or Linux) that can reach your ServiceNow instance over HTTPS
- Java runtime version supported by your ServiceNow release
- A dedicated local/domain service account for running the MID service
- Firewall and proxy rules permitting outbound communication to ServiceNow

## Setup Steps

TO BE ADDED

## Appendix

- [ServiceNow MID Server Product Documentation](https://www.servicenow.com/docs/r/servicenow-platform/mid-server/mid-server-landing.html)
