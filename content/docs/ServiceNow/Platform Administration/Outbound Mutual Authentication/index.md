---
title: "Outbound Mutual Authentication"
---

## Overview

Mutual authentication (also known as two-way TLS or mTLS) ensures that both the client and the server verify each other’s identities before establishing a secure connection. While standard outbound traffic only requires ServiceNow to trust the external server, mutual authentication requires the external server to trust ServiceNow as well.

This guide covers how to configure ServiceNow as a client to support mutual authentication for outbound web services

## Prerequisites

- A valid X.509 certificate and private key provided by your organization or a trusted Certificate Authority (CA)
- The target external endpoint URL supporting mutual authentication.

## Procedure
### Step 1: Import the Certificate
1. Navigate to **All > System Security > Certificates** in the ServiceNow application.
2. Click on **New** to create a new certificate record.
3. Fill in the required fields:
   - **Name**: A descriptive name for the certificate.
   - **Type**: Select **PKCS12 Key Store**
   - **Key store password**: Enter the password for the PKCS12 file.
4. Click the paperclip icon to upload the PKCS12 file containing the certificate and private key (e.g., `client_certificate.p12`).
5. Click **Submit** to save the certificate record.
  
### Step 2: Create a Protocol Profile
1. Navigate to **All > System Security > Protocol Profiles**.
2. Click on **New** to create a new protocol profile.
3. Fill in the required fields:
   - **Protocol**: Enter a unique name to identify this HTTPS protocol (e.g., `httpsapp`). THis name then will be used in urls to reference this protocol (e.g., `httpsapp://api.example.com`).
   - **Keystore**: Select the certificate record you created in Step 1.
4. Click **Submit** to save the protocol profile.

### Step 3: Use the Protocol Profile in Outbound Web Services
1. Navigate to the outbound web service configuration (e.g., **All > System Web Services > Outbound > REST Message**).
2. Select the REST message you want to configure for mutual authentication or create a new one.
3. Select the **Use mutual authenticiation** checkbox.
4. In the **Protocol** field, enter the name of the protocol profile you created in Step 2 (e.g., `httpsapp`).
5. Click **Save**

## References
- [ServiceNow Documentation: Configure mutual authentication](https://www.servicenow.com/docs/r/zurich/platform-security/authentication/c_MutualAuthentication.html)
- [Create a protocol profile](https://www.servicenow.com/docs/r/zurich/api-reference/web-services/t_CreateAProtocolProfile.html)
- [Enable mutual authentication](https://www.servicenow.com/docs/r/zurich/api-reference/web-services/t_EnableMutualAuth.html)
