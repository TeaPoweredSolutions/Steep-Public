# Steep release notes

## 0.7.0 (2026-11-30)

The first public release: a free **Time Limited Preview** that works until 2026-12-23. Steep shows Configuration Manager (MECM) teams what the MECM console doesn't: the devices, operating systems and applications that have quietly fallen through the cracks. This is everything Steep does today, page by page, in the order of the menu.

### Overview

- **Impact**: what Steep did and surfaced, and how your environment changed: devices patched, upgraded or repaired, client health fixes, vulnerable applications upgraded or removed, with daily trends. Every device gets its own Steep device ID, matched through renames, reimages, client reinstalls and hardware changes.

### Clients

- **Patch compliance**: every OS variant (product, edition, channel, build and revision) with device counts. Each device is rated *Current*, *Within grace*, *Behind* or *Out of support* by its build revision against Microsoft's latest release, not by WSUS scan results. Devices missing fixes for actively exploited CVEs are called out, and Windows 10 ESU and hotpatch devices are rated correctly. A total vulnerabilities counter and a daily trend chart.
- **Client health**: whether every computer in AD is in MECM with a working client. Checks for no client, not discovered, not approved, blocked, obsolete or duplicate records, inactive clients, old policy and inventory, no client certificate, failed or old client health checks (ccmeval), no boundary group, a client version below production, failing update scans, failed update installations and .NET Framework update failures. Thresholds are configurable, with counters and a daily trend chart. **Investigate** opens the fix page for a problem.
- **Client tools**: a page for each problem, with the devices that have it and the tools that fix it, plus the tools on their own. All of them sign in to devices with **Secure Client Access**: each device's own Windows LAPS password, read only when needed, never stored or logged, and reset by the device within minutes of the run.
    - **Secure Client Repair**: repairs BITS, WMI, the .NET Framework, the MECM client and the Windows Update Agent, least invasive first, up to the level you allow. Restarts happen only if you allow them, and Steep carries the repair on afterwards.
    - **Secure Client Push**: installs, repairs or upgrades the MECM client like MECM's client push, but with LAPS instead of a global admin account.
    - **Software update scan report**: the device's last software update scan as one page: scan source and catalog version, deployments, every update as Installed, Required or Missing, and why expected updates are missing.
    - **Software update patching report**: traces each software update deployment through the client's logs and gives each update's final status, with the log lines behind it and a zip of the logs.
    - **Trace hardware inventory**: follows a device's last inventory report from the client to the site database and says where it broke.
    - **Read OS details**, **Gather logs**, **Read installed MSI**, the MECM client actions (health check, machine policy, hardware inventory, update scan), and **Approve**, **Unapprove**, **Block** and **Unblock**.
    - A history of every run, kept on the page and on the device.

### Updates

- **Update deployment**: for each OS family, whether the latest Patch Tuesday update is synced and deployed in MECM, and if not, the last one that was. Shows "Product not enabled in SUP" when an OS's updates are missing because its product isn't selected on the software update point.
- **ADR wizard**: builds automatic deployment rules from lab-checked templates for each OS and architecture (Windows 11, Windows 10 and Windows Server), with a preview of the updates each rule picks from your catalog. **Deploy to MECM** creates the software update point products, package, collection, ADR and deployment, and records each change in the audit log.
- **OOB alerts**: every out-of-band and preview release since Patch Tuesday, the CVEs each is first to fix, and which ones justify breaking the normal patch cycle.

### Application Management

- **Vulnerabilities**: vulnerable applications and installations, with CVEs from the NVD, CISA's Known Exploited Vulnerabilities catalog and EPSS scores. Counters and a daily trend chart.
- **Families**: MECM applications matched to installed software by product code and MSI UpgradeCode, grouped into product families, with **MSI** and **Metered** badges and software metering usage per version.
- **Application Management**: choose the **production version** every device should run (Steep supersedes the older versions in MECM) and **pin** older versions to the devices that need them, by AD group, OU or collection, with the reason recorded. Pinning a version with known CVEs needs an acknowledgement. Converts user deployments to device deployments. Steep suggests; engineers decide.
- **Applications**: every installed application and version, with suggestions for new families, and Excel and CSV export.
- **MSI Auto-packager**: creates MECM applications from MSI files, with detection, icons, install logs, repair, distribution and metering rules set up for you.
- **Software metering**: usage per program and device for 13 months, so you can see whether anyone still uses an old or vulnerable version before removing it.

### Configure

- **Collectors**: when each collection runs, its last results, and **Collect now**.
- **Connection accounts**: the accounts Steep uses, encrypted with Windows DPAPI and never shown, exported or logged. Steep checks each account in AD hourly and warns before it's locked out, disabled or expired.
- **Connections**: MECM hierarchies (a read account, an optional write account and the site database), AD domains, the accounts for content source shares, and the vulnerability data sources.
- **Client tools**: which accounts may read and reset LAPS passwords, fallback admin accounts, resetting the LAPS password after every use (on by default), whether LSA protection is on, and Automatic Secure Client Push for the OUs you choose (report only by default).
- **Permissions**: Steep roles (Architect, Client Administrator, Application Administrator, Windows Updates Administrator, Reader) follow MECM's administrative users. A new install grants only Architects from MECM; an Architect turns on the others.
- **Settings**: thresholds, retention, collection priority, the SUP/WSUS manager (a software update point sync on Patch Tuesday, timed for Microsoft's release), software metering, application packaging, the web console certificate, and the daily check for new Steep versions.
- **License**: license status, the license terms, and the third-party licenses and acknowledgements.
- **Audit**: every change, by an engineer or by automation, with filters and Excel or CSV export. Steep Architects only.
- **Telemetry**: preview and download an anonymous zip to share with Tea Powered Solutions if you choose to. Devices, applications, versions and builds are replaced by random IDs that only your Steep can map back. Nothing is sent automatically.

### Help

- **Docs**: the offline help (the docs for your version, served by Steep itself, with search, no internet needed) and a link to the online docs for the latest release.
- **Contact**: where to ask questions, report problems or security issues, and reach Tea Powered Solutions: GitHub, email and Reddit.

### Installation and operations

- **One PowerShell script** installs, upgrades or removes Steep as a Windows service. Steep brings its own .NET runtime, so nothing is installed machine-wide or shared with MECM. `-CertificateThumbprint` chooses the console's HTTPS certificate.
- **Least privilege**: MECM *Read-only Analyst*, AD read, and SELECT on only the site database views Steep reads (`Grant-SteepSiteDatabaseRead_0.7.0.sql`; `Revoke-SteepSiteDatabaseRead_0.7.0.sql` removes it).
- **Its own database** (SQLite): about 1.2 GB at 150,000 devices, 1.5 GB with software metering. Daily maintenance gives free space back to Windows and checks the database weekly.
- **Customer-hosted**: nothing is sent to Tea Powered Solutions unless you send it, and the license is never checked online. Once a day Steep asks GitHub whether a newer version exists, sending nothing about your environment; you can turn this off.
- **Trial mode** without a license: real counts and statuses, with OS, patch, CVE and device details hidden.
