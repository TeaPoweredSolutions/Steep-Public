# Installing Steep

Steep runs as a Windows service, ideally on the **MECM primary site server**. It brings its own .NET runtime, so nothing is installed machine-wide or shared with MECM.

## Requirements

| Requirement | Details |
|---|---|
| Server | Windows Server 2016 or later, 64-bit. Recommended: the primary site server, which has the management point's certificate. |
| Rights to install | Local Administrator on the server |
| Disk | About 400 MB for the program. The database grows with devices and history: tens of MB on a small site, about 1.2–1.5 GB at 150,000 devices (see [scalability.md](scalability.md)). |
| Network | SMS Provider (DCOM/WMI), domain controllers (LDAP 389, signed and sealed), SQL Server (1433) for the site database, and outbound HTTPS (443) to the addresses below |
| Console port | TCP 8443 by default. Never 80, 443, 8530 or 8531, which IIS, WSUS and the MECM Admin Service use. |
| Accounts | At least one read-only account for MECM (Read-only Analyst) and AD. Separate accounts per connection are supported and recommended. See [permissions.md](permissions.md). |
| Configuration Manager console (optional) | Only for Application Management's "Supersede with Production". Steep loads the console's libraries from where the console is installed on its server. |

### Outbound internet access

Steep downloads public reference data over HTTPS (443). Nothing about your environment is sent to Tea Powered Solutions.

| Address | What Steep reads | When |
|---|---|---|
| `learn.microsoft.com` | Windows release information: versions, builds, end-of-support dates, hotpatch calendars | Always (required) |
| `api.msrc.microsoft.com` | Microsoft's monthly security updates and the CVEs each fixes | Always (required) |
| `services.nvd.nist.gov` | CVEs for the applications Steep tracks (a fixed list, the same for every customer) | Public vulnerability feeds on |
| `www.cisa.gov` | CISA's Known Exploited Vulnerabilities catalog | Public vulnerability feeds on |
| `api.first.org` | EPSS scores. Steep sends the IDs of CVEs found on installed software, never device names. | Public vulnerability feeds on, EPSS set to "Ask only for the CVEs found in my environment" (the default) |
| `epss.empiricalsecurity.com` | EPSS's full daily file (about 3 MB). Nothing is sent. | Public vulnerability feeds on, EPSS set to "Always download every score", or more than 2,000 CVEs |
| `api.github.com` | The latest Steep version number, so Architects know when to upgrade. Nothing about your environment is sent, and Steep never updates itself. | Daily and at start-up, while **Check for new Steep versions** is on (Configure → Settings) |
| `aka.ms` → `download.microsoft.com` | Microsoft's .NET Framework Repair Tool, kept only if Microsoft signed it | Only when an engineer clicks **Download from Microsoft**. Offline: copy it into `%ProgramData%\TeaPowered\Steep\RepairTools`. |

**EPSS:** by default, Steep asks FIRST's API for the CVEs in your environment, 80 per request. To send no CVE IDs at all, set the EPSS feed to always download the full daily file (Configure → Connections → Vulnerability data). Above 2,000 CVEs, Steep downloads the full file anyway, as FIRST asks. steep.log says which method was used.

To turn off the version check: Configure → Settings, or `.\Steep.exe config set updates.check false`.

## Install

1. Extract `Steep-<version>.zip` to a folder on the server.
2. Read the license terms (`Steep-License-Terms.md` in the package).
3. Open **PowerShell as Administrator** in that folder and run:

    ```powershell
    .\Install-Steep.ps1 -AcceptLicenseTerms
    ```

4. Open the address the installer prints, and sign in with your Windows account. Everyone accepts the license terms once, the first time they open Steep.
5. On **Connection accounts**, add the accounts Steep will use.
6. On **Connections**, add your MECM hierarchies, site databases and AD domains. **Find site database** locates the site database for you.
7. Click **Collect now**, or wait for the first scheduled collection.

### Installer switches

| Switch | What it does |
|---|---|
| `-AcceptLicenseTerms` | **Required.** Without it, the installer shows where the terms are and changes nothing. |
| `-CertificateThumbprint <thumbprint>` | The console's HTTPS certificate. By default, Steep uses the certificate IIS has bound for the management point. See [web-console-certificate.md](web-console-certificate.md). |
| `-AllowedPrincipals 'CORP\MECM Admins'` | Users or groups made Steep Architects. The installing account is always added. MECM's Full Administrators and Infrastructure Administrators also become Steep Architects automatically, once Steep has read MECM (unless their MECM scopes are limited). |
| `-Port <number>` | The console port (default 8443). Refuses a port in use, including WSUS and management point ports. |
| `-OpenFirewall` | Allows the console port inbound, domain network profile only. Useful only with an HTTPS certificate. |

### The console address

- **With a certificate:** `https://<server FQDN>:8443`. Use the full name the certificate is issued to.
- **Without one:** only on the server itself, at `http://localhost:8443`. Choose a certificate (installer switch or Settings → Web console), then restart the service.
- `Steep.exe url` prints the address at any time.

!!! info "HTTPS warnings"
    Browsers may warn about the management point's certificate, for example a self-signed one or one without a Subject Alternative Name. See [HTTPS for the console](#https-for-the-console).

### What the installer does

- Copies the program to `C:\Program Files\TeaPowered\Steep`.
- Creates `C:\ProgramData\TeaPowered\Steep` (database, settings, encrypted accounts, and the log `Logs\steep.log`) and limits it to **SYSTEM and Administrators**. By default, every local user could read it.
- Registers the **Steep** service: LocalSystem, automatic (delayed) start, restarts if it fails.
- Writes only small crash dumps of Steep.exe, without memory contents.
- With `-OpenFirewall`, opens the console port on the domain profile only.

## Upgrade

Run `Install-Steep.ps1 -AcceptLicenseTerms` from the new package. It stops the service, replaces the program, upgrades the database and starts the service. Settings, accounts and history are kept.

- If the license terms changed, accepting them here accepts them for everyone who accepted before. A major change asks everyone to accept again.
- If a release reads new site database views, its release notes say so. Re-run the new package's `Grant-SteepSiteDatabaseRead_<version>.sql` if you granted access to Steep's views only. Install-Steep.ps1 names the new views in red when you upgrade.

## Back up and restore

Everything Steep keeps is in `C:\ProgramData\TeaPowered\Steep`. The program folder can always be reinstalled.

| File | What it holds |
|---|---|
| `steep.db` | Settings, connections, encrypted accounts and the NVD API key, collected data, trends, application decisions, and the audit log |
| `license.lic` | The imported license |
| `Logs\steep.log` (and older `steep*.lo_`) | The activity log. Useful for support, not needed to restore. |

**Back up:** stop the service, copy the folder, start the service. A copy taken while Steep runs can be caught mid-write.

```powershell
Stop-Service Steep
Copy-Item 'C:\ProgramData\TeaPowered\Steep' 'D:\Backup\Steep' -Recurse -Force
Start-Service Steep
```

!!! info
    The copy holds your device inventory. Protect it like you would protect a backup of the MECM site database.

**Restore on the same server:** install Steep (the same version or newer), stop the service, copy the files back, start the service.

**Restore on another server:** the same steps, then re-enter every connection account's password and the NVD API key. Passwords can only be decrypted on the server that saved them. Everything else carries over, including the license.

Application families and decisions can also be moved on their own: **Export decisions** on Configure → Settings, then import on the other server.

## Database maintenance

Steep looks after its own database. Once a day, after the retention clean-up (Settings → Retention), it:

- gives free space back to Windows when it's more than 20% of the file
- refreshes SQLite's query statistics
- empties the change file (`steep.db-wal`) into the database
- once a week, checks the database for damage. A problem goes to steep.log and the audit log; restore from a backup.

!!! info
    Each run is logged in steep.log under Retention, with the database size.

## Uninstall

```powershell
.\Uninstall-Steep.ps1             # keeps settings, accounts and history
.\Uninstall-Steep.ps1 -RemoveData # also deletes the database, settings, license, accounts and logs (cannot be undone)
```

## License

Steep runs in trial mode until a license is imported (Configure → License, or `Steep.exe license import <file>`). Trial mode shows real counts, but hides OS, patch, CVE and device details, and turns exports off. See [licensing.md](licensing.md).

## Who can use the console

Users sign in with their Windows accounts. Steep gives each person **Steep roles** from MECM's **Administrative Users**, read every hour. Roles given to a group apply to its members, nested groups included.

| MECM security role | Steep role | Can |
|---|---|---|
| Full Administrator, Infrastructure Administrator | Steep Architects | Everything, including Configure |
| Application Administrator, Application Author, Application Deployment Manager, Operations Administrator | Steep Application Administrators | Application families and decisions; start collections |
| Software Update Manager, Operations Administrator | Steep Windows Updates Administrators | Windows updates actions; start collections |
| Operations Administrator | Steep Client Administrators | Client tools and client health repairs |
| Read-only Analyst, or any other MECM role | Steep Readers | Read every page and export; change nothing |

- **A new install grants only Steep Architects from MECM.** An Architect turns on the other roles in **Configure → Permissions**.
- **Always Architects:** local Administrators on the Steep server (elevated sign-in only) and the Steep service account, so nobody is locked out.
- **Scoped administrators** (MECM users without the default scopes) get no access until an Architect adds them, and then only as Steep Readers. Steep doesn't apply MECM scopes: everyone with access sees every device. See [permissions.md](permissions.md#scopes).
- **Custom MECM roles** give Steep Readers unless an Architect maps them.
- **Narrowing it down:** on Configure → Permissions, an Architect can remove an MECM user or group from Steep, or untick "Granted from MECM" for a role.
- **Added by hand:** people who aren't MECM administrative users, on the Permissions page or with the command line.

From an elevated PowerShell:

```powershell
& "C:\Program Files\TeaPowered\Steep\Steep.exe" access add "CORP\MECM Admins" Architect
& "C:\Program Files\TeaPowered\Steep\Steep.exe" access list
& "C:\Program Files\TeaPowered\Steep\Steep.exe" access remove "CORP\someone"
```

Roles: Reader (the default), ApplicationAdministrator, UpdatesAdministrator, ClientAdministrator, Architect. Changes apply when the person reloads Steep. Refused users see which account they used and why.

!!! note "Local administrators and UAC"
    Browsing to Steep on its own server sends your non-elevated sign-in, so administrator rights don't apply. That's why the installer makes the installing account an Architect.

## HTTPS for the console

By default, Steep uses the certificate IIS has bound to the management point's HTTPS port (Site Properties → Ports, 443 by default), including host name (SNI) bindings. It checks every hour and picks up renewals. On a server that isn't a management point, it looks on port 443. Restart the service after changing the setting (Settings → Web console).

Settings → Web console, `Steep.exe certificate` and the installer all check the certificate and list what browsers will object to. The two common problems:

| Problem | Fix |
|---|---|
| **MECM's enhanced HTTP certificate** ("SMS Role SSL Certificate", issued by SMS Issuing). MECM clients trust it; browsers don't. | Request a web server certificate from your CA ([Microsoft's example](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/network/example-deployment-of-pki-certificates#deploy-the-web-server-certificate-for-site-systems-that-run-iis)). Bind it in IIS, or choose it in Steep. |
| **No Subject Alternative Name (SAN).** Browsers ignore the subject name. | Put the server's FQDN, and any short name people type, in the SAN. |

!!! info
    No warnings in Settings means the Steep server trusts the certificate. Each browser decides for itself: domain computers trust an enterprise CA automatically; other computers need its root certificate imported.

Details and certificate template settings: [web-console-certificate.md](web-console-certificate.md).

## Connection accounts

- Add accounts on **Connection accounts** in the console. Passwords are encrypted for SYSTEM on this server. Moving Steep means entering them again.
- Domain accounts (Windows authentication) and SQL Server logins both work.
- **Best practice:** one dedicated, least-privilege account per connection. See [permissions.md](permissions.md).
- **Source share:** both the read and write accounts need their own Share and NTFS entries. See [Source share accounts](permissions.md#source-share-accounts).

!!! tip "Add a site database connection"
    Reading the site database is much faster than the SMS Provider, and some checks need it. Grant access with `Grant-SteepSiteDatabaseRead_<version>.sql` (in the package). See [permissions.md](permissions.md#site-database-account).

## Limiting what Steep sees

**MECM:** give Steep's accounts the default scopes and All Systems. Decide what each person can do with Steep roles instead.

!!! warning "Don't restrict Steep's accounts to non-default scopes"
    Dashboards would quietly leave out devices, and differences between scopes can lead to unintended changes.

**Active Directory:** under Connections → Active Directory domains, list the OUs to collect computers from (sub-OUs included). MECM devices outside them are left out of the dashboards and listed in a note on Client health.

## Verifying the installation

- `Get-Service Steep` shows **Running**.
- Event Viewer → Windows Logs → Application, source **Steep**: start-up messages and warnings, such as no HTTPS certificate.
- `& "C:\Program Files\TeaPowered\Steep\Steep.exe" version` shows the version.
- The console status bar shows the last collection and any source that failed.
- `%ProgramData%\TeaPowered\Steep\Logs\steep.log` is Steep's log, in CMTrace format.
