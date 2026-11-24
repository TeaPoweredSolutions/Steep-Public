# Web console certificate

Steep's console runs over HTTPS. This page covers what a certificate needs so browsers accept it, and how to issue one from an Active Directory Certificate Services (AD CS) enterprise CA.

Settings → Web console, and `Steep.exe certificate` on the server, check the certificate Steep uses and list anything a browser will object to.

## Do you need a new certificate?

By default, Steep uses the management point's certificate. It finds it the way MECM does: the certificate IIS has bound to the site's HTTPS client port (Site Properties → Ports, 443 by default), on the Default Web Site or a custom one, including host name (SNI) bindings. Settings → Web console shows where it looked. Until Steep has read MECM, or on a server that isn't a management point, it looks on port 443.

| Your site | What to do |
|---|---|
| **PKI (HTTPS), with the MP's certificate from your CA** | It usually works as it is. Check Settings shows no warnings. The most common problem is a missing Subject Alternative Name (SAN): MECM accepts the FQDN in the subject, browsers don't. |
| **Enhanced HTTP** | The certificate on 443 is MECM's self-signed "SMS Role SSL Certificate", issued by **SMS Issuing**. MECM clients trust it; browsers don't, and it has no SAN. Get a certificate from your CA (below). |
| **No IIS on the server** | Get a certificate from your CA and choose it in Steep. Until then, the console only opens on the server itself, at `http://localhost:8443`. |

!!! info
    Steep uses exactly one certificate: the one bound to the management point's HTTPS port, or the one whose thumbprint you enter. It never picks one by template or purpose.

## Certificate requirements for browsers

All of these must be true for Edge, Chrome or Firefox to open the console without a warning:

| Requirement | Why |
|---|---|
| **A SAN with every name people type**: the FQDN (`mecm01.corp.contoso.com`), plus the short name (`mecm01`) or any DNS alias they use | Browsers only check the SAN. They ignore the subject's common name. |
| **Enhanced Key Usage includes Server Authentication** (1.3.6.1.5.5.7.3.1). The certificate console calls it "Intended Purposes"; templates call it "Application Policies". | Browsers only accept certificates issued for server authentication. |
| **Issued by a CA the workstation trusts** | Domain members trust an enterprise CA automatically. Other computers need its root certificate in Trusted Root Certification Authorities. |
| **Not expired**, and the CA's revocation list (CRL) reachable from workstations | Some browsers and policies refuse certificates whose revocation can't be checked. |
| **RSA key of at least 2048 bits** | Smaller keys are rejected. |

!!! warning
    On the Steep server, the certificate must be in the computer's Personal store (certlm.msc), with its private key. Requesting it through certlm.msc or autoenrollment (via a [certificate template](#the-certificate-template) on your domain CA) puts it there. Importing a .cer file doesn't: it has no private key. The computers people browse from only need to trust your CA, which domain members do automatically.

Opening the console by IP address always shows a warning, unless the IP is in the SAN. Use a name.

## The certificate template

Your PKI team may already have one, often the template for MECM's HTTPS site systems ("ConfigMgr Web Server Certificate" in Microsoft's example). If it puts DNS names in the SAN, use it.

Otherwise, in the Certification Authority console: **Certificate Templates** → **Manage** → right-click **Web Server** → **Duplicate Template**, and set:

| Tab | Setting |
|---|---|
| **Compatibility** | Whatever your CA and PKI team use. |
| **General** | A name, such as **Steep Web Console**. Validity per your policy (1–2 years is common). |
| **Request Handling** | Purpose: **Signature and encryption**. Leave **Allow private key to be exported** unticked. |
| **Cryptography** | Minimum key size **2048** (RSA). |
| **Subject Name** | Either option below. |
| **Extensions** | **Application Policies**: Server Authentication (already there). **Key Usage**: Digital signature and Key encipherment (the default). |
| **Security** | A group with the Steep servers' computer accounts (for example MECM's **ConfigMgr IIS Servers**): **Read** and **Enroll**, plus **Autoenroll** with "Build from this Active Directory information". |

Then **Certificate Templates** → **New** → **Certificate Template to Issue**, and choose the new template. Restart the Steep server (or run `gpupdate /force`) so it picks up the group membership.

### Subject Name: which option?

| Option | SAN | Renewal |
|---|---|---|
| **Supply in the request** (Microsoft's ConfigMgr example) | You type the names, so you can add the short name and aliases. | Manual: request a new certificate before the old one expires. |
| **Build from this Active Directory information**, with **DNS name** ticked | The server's FQDN only, so people must use the full name. | Automatic, with autoenrollment. |

## Requesting the certificate

On the Steep server:

1. Run **certlm.msc**, right-click **Personal** → **All Tasks** → **Request New Certificate**.
2. Choose **Active Directory Enrollment Policy**, then the **Steep Web Console** template.
3. With "Supply in the request": click **More information is required to enroll for this certificate**. Under **Alternative name**, choose **DNS** and add each name (`mecm01.corp.contoso.com`, `mecm01`). Leave the subject empty, or set **Common name** to the FQDN.
4. Click **Enroll**.

## Using it in Steep

| Option | How | Renewals |
|---|---|---|
| **Bind it to the management point's HTTPS port** | IIS Manager → the site MECM uses → Bindings → https → the new certificate. Keep Steep on **Use the IIS HTTPS certificate**. | Picked up within an hour. |
| **Choose it in Steep only** | Settings → Web console → **A specific certificate**, and paste the thumbprint. IIS is left alone. | Enter the new thumbprint after every renewal. |
| **Choose it when installing** | `.\Install-Steep.ps1 -AcceptLicenseTerms -CertificateThumbprint <thumbprint>`. Leaving the switch out on an upgrade keeps the earlier choice. | Run the installer again with the new thumbprint. |
| **Choose it from the command line** | See below. | Enter the new thumbprint after every renewal. |

!!! warning "Changing the MP's binding"
    MECM's site systems also use the certificate bound on the management point, and always prefer a PKI certificate over the enhanced HTTP one. Check with your MECM administrator before changing it.

From an elevated PowerShell in `C:\Program Files\TeaPowered\Steep`:

```powershell
.\Steep.exe config set web.certificatethumbprint <thumbprint>
.\Steep.exe config set web.certificatesource Thumbprint
Restart-Service Steep
```

To go back to following IIS: `.\Steep.exe config set web.certificatesource IisHttpsBinding`.

After any change, restart the Steep service, then check Settings → Web console or run `Steep.exe certificate`.

## Microsoft's documentation

- [Example web server certificate deployment for ConfigMgr site systems](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/network/example-deployment-of-pki-certificates#deploy-the-web-server-certificate-for-site-systems-that-run-iis)
- [PKI certificate requirements for Configuration Manager](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/network/pki-certificate-requirements)
- [Enhanced HTTP and the SMS Issuing certificate](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/enhanced-http#validate-the-certificate)
