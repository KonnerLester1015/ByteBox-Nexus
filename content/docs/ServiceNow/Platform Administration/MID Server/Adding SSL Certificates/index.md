---
title: "Adding SSL Certificates"
---

# Adding SSL Certificates to ServiceNow MID Server

ServiceNow MID Servers rely on a private Java Runtime Environment (JRE) to handle outbound HTTPS connections. Unlike web browsers, this environment does not automatically inherit trusted certificates from the host OS (Windows/Linux).

This guide covers the process of manually trusting a Certificate Authority (CA) to resolve handshake failures in integrations, Discovery, or Service Mapping.

## Common Use Cases
- **Integration Failures:** REST/SOAP messages returning Response Code: -1 or Untrusted Certificate.
- **Discovery Issues:** Failure to connect to vCenter, cloud endpoints, or internal management consoles.
- **Certificate Rotation:** A service updated its certificate, but the MID Server lacks the new Intermediate or Root CA.

## Prerequisites
- Administrative rights on the host
- The CA certificate file (usually `.crt`, `.cer`, `.pem`)
- Identification of the MID Server's folder containing the JRE Keytool (eg., C:\ServiceNow\MID_Prod\agent\jre\bin)
- Identification of the MID Server's JRE cacerts file (eg., C:\ServiceNow\MID_Prod\agent\jre\lib\security\cacerts)
- Knowledge of the keystore password (default is `changeit`)

## Procedure

1. Open Command Prompt or Terminal as Administrator/root.
2. Navigate to the JRE bin directory of the MID Server:

   ```sh
   cd C:\ServiceNow\MID_Prod\agent\jre\bin
   ```

3. Run the keytool command to import the CA certificate

     ```sh
     keytool -import -alias my_new_ca -file "C:\temp\ca_cert.cer" -keystore "..\lib\security\cacerts"
     ```

4. When prompted, enter the keystore password (default is `changeit`).
5. When prompted, type `yes` to trust the certificate.

## Verification
You can verify the certificate was added successfully by listing the contents of the keystore:

```sh
keytool.exe -list -keystore "..\lib\security\cacerts" -storepass changeit
```

{{< callout type="warning" >}}
Dont forget to run this command from where the MID Server's JRE Keytool is located, and ensure you have the correct path to the cacerts file.
{{< /callout >}}

## References
- [ServiceNow Documentation: Add SSL certificates for the MID Server](https://www.servicenow.com/docs/r/servicenow-platform/mid-server/add-ssl-certificates.html)
- [Java keytool Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
