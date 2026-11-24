# Steep

*by Tea Powered Solutions*

**Steep shows Configuration Manager (MECM) teams what the MECM console doesn't: the devices, operating systems and applications that have quietly fallen through the cracks.**

MECM's software update compliance comes from WSUS scan results: "applicable" or "not applicable". Those results are often wrong. Steep combines MECM data with Active Directory and Microsoft's own release and security data. It rates each device by its OS build revision, which shows its real patch level.

Example: Windows 10 21H2 and Windows 10 Enterprise LTSC 2021 share build 19044. The console's "Operating System Build" column looks the same for both. Steep tells you the 21H2 device is a GA system that was left behind, not LTSC.

---

## Features

### Dashboards

| Dashboard | What it shows / does |
|---|---|
| **Patch compliance** | Every OS variant (product, edition, channel, build, revision) with device counts. Each device is *Current*, *Within grace*, *Behind* or *Out of support* against the latest Patch Tuesday release. Devices missing fixes for **actively exploited CVEs** are called out. Windows 10 ESU and hotpatch devices are rated correctly. |
| **Update deployment** | For each OS family: whether the latest Patch Tuesday update is synced and deployed in MECM, and if not, the last one that was. |
| **WSUS validation** | Patching dashboards are product aware and will show you operating system products that are not synced in your Software Update Point
| **OOB alerts** | Every out-of-band and preview release since Patch Tuesday, the CVEs each is first to fix, and which ones justify breaking the normal patch cycle. |
| **Client health** | Whether every computer in AD is in MECM with a working client: no client, not discovered, not approved, blocked, duplicate, inactive, old policy, old inventory, failing update scans, failed update installs and more. Thresholds are configurable. |
| **Application vulnerabilities** | Vulnerable applications and installations, with CVEs from NVD, CISA's exploited list and EPSS scores. |
| **Client tools** | Run automated agent repairs, diagnostics, or collect logs from systems
| **Trends and Impact** | Daily charts of compliance, client health and vulnerabilities, and what Steep fixed or surfaced. |

!!! info
    Every dashboard has a **scope picker** (one MECM hierarchy or AD domain) and **Excel and CSV export**, for all devices or only problem ones.

### Client tools and Secure Client Access

- **Secure Client Access** signs in to devices with each device's own **Windows LAPS** password. Steep reads it only when needed, never stores or logs it, and has the device reset it within a few minutes of the run. [Why one password shouldn't open every device](why-secure-client-access.md).
- **Secure Client Push** installs or repairs the MECM client like MECM's client push, but with LAPS instead of a global admin account. **Automatic Secure Client Push** does it for computers in the OUs you choose.
- **Secure Client Repair** repairs BITS, WMI, the .NET Framework, the MECM client and the Windows Update Agent, least invasive first, up to the level you allow.

| Tool | What it does |
|---|---|
| **Read OS details** | Reads the Windows version, revision, edition and architecture, for devices MECM has no revision for. The dashboards use it at once. |
| **Gather logs** | Brings back a zip of the device's troubleshooting logs: MECM client, Windows Update, CBS, setup and event logs. |
| **Trace hardware inventory** | Follows a device's last inventory report from the client to the site database, and says where it broke. |
| **Software update scan report** | Shows what the device's last software update scan found: every update, its status, and why expected updates are missing. |
| **Software update patching report** | Traces each software update deployment through the client's logs, and gives each update's final status with the log lines behind it. |
| **Secure Client Push** | Installs, repairs or upgrades the MECM client, by hand or automatically for the OUs you choose. |
| **Secure Client Repair** | Repairs BITS, WMI, the .NET Framework, the MECM client and the Windows Update Agent, least invasive first, up to the level you allow. |
| **Client actions** | Runs the MECM client's own actions from the fix pages: health check, machine policy, hardware inventory, software update scan. |
| **Approve and Block** | Approves, unapproves, blocks or unblocks clients in MECM. |
| **Read installed MSI** | Reads an application's MSI from a device that has it installed, for Application Management. |

How each tool works: [client-tools.md](client-tools.md).

!!! tip "Retire your client push accounts"
    With Secure Client Access, you can remove MECM client push accounts that are local administrators everywhere. A stolen LAPS password opens one device, and only for a few minutes.

!!! note "LAPS support"
    Steep supports Windows LAPS with passwords stored in Active Directory. Entra ID is planned. Legacy Microsoft LAPS isn't supported.

### Software updates

- **ADR wizard:** builds automatic deployment rules from lab-checked templates for each OS, with a preview of the updates they pick.
- **SUP/WSUS manager:** can start a software update point sync on Patch Tuesday, timed for Microsoft's release.

### True patching compliance

Steep is **product aware**. It rates each device by what it actually is, not by what WUA scans report:

| Steep reads | Why it matters |
|---|---|
| **Type**: workstation or server | Different products, with their own release schedules and end-of-support dates. |
| **Channel**: GA or LTSC/LTSB | The same build can be either. One may be out of support while the other still gets updates. |
| **Edition**: Enterprise, Education, Pro and so on | Editions of the same release reach end of support at different times. |
| **Release**: 22H2, 24H2, 25H2, 26H2 (from the build number) | Each release has its own updates and end-of-support date. |
| **Build revision**: the last part of the version, for example 26200.**6584** | Each monthly cumulative update sets it to a known number, so the revision says exactly which update is installed. |

Steep reads these from MECM's own discovery and inventory data (or with **Read OS details** for devices MECM has no revision for), and matches the revision against Microsoft's release information. It knows each device's installed update, the latest one, how many releases and days behind it is, and which exploited CVEs it's missing, without relying on scan results.

#### Why scan compliance can't be trusted

MECM's compliance comes from the Windows Update Agent scanning against WSUS. It answers "is anything **in this deployment** missing?", not "is this device patched?":

- **It isn't product aware.** If the update for a device's OS isn't synced or deployed, nothing is missing from the device's point of view, so it reports compliant. A new Windows release whose product isn't selected on the software update point is a common example.
- **Out-of-support versions look compliant.** Once Microsoft stops releasing updates for a version, no new update applies to it, so it can show compliant forever.
- **Stranded builds look compliant.** A device missing a prerequisite update, or servicing stack, finds every newer update "not applicable", and reports compliant while it falls further behind every month.
- **Old scans keep old answers.** Compliance reflects the last successful scan. A device that hasn't scanned for weeks keeps reporting its old state, or shows Unknown, which hides how far behind it is.
- **A catalog that's behind can't see new updates.** If the software update point hasn't synced, the newest updates aren't in the catalog, so devices can't report them missing.
- **Devices without a working client aren't counted.** Steep finds them through AD and can query their details to validate their level directly.
- **Build numbers are shared.** Windows 10 21H2 and Windows 10 Enterprise LTSC 2021 are both build 19044, so the console's "Operating System Build" column can't tell them apart even if you're looking for it.

### Application management

- Matches MECM applications to installed software by product code and MSI UpgradeCode, not by names people typed. Groups them into product families and labels 32-bit and 64-bit.
- **Vulnerabilities per application:** the CVEs for each installed version, from the vulnerability feeds, with known-exploited ones flagged and EPSS scores.
- **Production version:** choose the version every device should run. Steep supersedes the older versions with it in MECM, so devices on older versions upgrade.
- **Pinned versions:** keep an older version on the devices that need it, chosen by AD group, OU or collection, with the reason recorded. Pinning a version with known CVEs needs an acknowledgement.
- **MSI Auto-packager** creates MECM applications from MSIs.
- **Software metering:** MSI and Metered badges, and usage per version and device for 13 months.
    - Pinned an old version with known vulnerabilities? Use metering to check whether anyone still uses it. If nobody does, remove it and reduce your risk.
- Steep suggests; engineers decide. Nothing changes automatically. Decisions can be exported and imported.

### Operations

- **Microsoft's own data:** release information, end-of-servicing dates and the MSRC Security Update Guide. No account or API key needed.
- **Multiple environments:** several MECM hierarchies, site databases and AD domains. Devices are matched by computer-account SID, so renames don't break them.
- **Scheduled collectors**, plus **Collect now** on each dashboard.
- **Auditing:** every action, by an engineer or by automation, is in the audit log, with filters and export. Activity is logged in CMTrace format.
- **Simple installation:** one PowerShell script installs, upgrades or removes Steep. Steep brings its own .NET runtime, so nothing is installed machine-wide or shared with MECM.

---

## Security

- **Read-only is useful.** The MECM write account is optional. Read, write, SQL, AD and source share accounts can all be separate.
- **Least privilege.** Reading needs MECM *Read-only Analyst*, read access to the site database, and AD read. SQL access can be limited to the views Steep reads.
- **Roles from MECM.** Users sign in with their Windows accounts. Steep roles (Architect, Client Administrator, Application Administrator, Windows Updates Administrator, Reader) follow MECM's administrative users. A new install grants only Architects from MECM; an Architect turns on the others.
- **Protected passwords.** Encrypted with Windows DPAPI for Steep's service account. Never shown, exported or logged. A copied database is useless elsewhere.
- **Locked-down server.** The data folder is limited to SYSTEM and Administrators. The console runs over HTTPS on its own port (never 80 or 443), using the site server's certificate. Without a certificate, it only opens on the server itself.
- **Customer-hosted.** Everything runs in your network. Nothing is sent to Tea Powered Solutions unless you send it. The license is never checked online. Once a day, Steep asks GitHub if a newer version exists, sending nothing about your environment; you can turn this off.
- **Supply chain.** Packages are pinned to nuget.org, the build treats warnings as errors, and over 1,000 automated tests run with every build, many built from real lab data.

See [permissions.md](permissions.md) and [password-security.md](password-security.md).

## Scale

- **Designed for up to 150,000 devices:** an estimated **1.2 GB** of disk (1.5 GB with software metering) and **1.5–2 GB** of memory.
- **Fast collection:** devices and installed software are read from the site database in batches. The SMS Provider still decides which devices are included, so MECM security applies. If SQL can't be reached, Steep falls back to the SMS Provider.
- **Changes only:** AD every hour and installed software every day, with a full read daily.
- **Instant dashboards:** worked out once after each collection, then served from memory.
- **Its own database:** SQLite. Steep never uses MECM's SQL Server and needs no SQL license.

*Figures at scale are estimates until the 150,000-device scale test.* See [scalability.md](scalability.md).

---

## Licensing

- **Free to try.** Without a license, Steep runs in trial mode: real counts, with OS, patch, CVE and device details hidden.
- **Licensed per site and MECM client.** Licenses are signed. Going over the client count never turns anything off: Steep shows a reminder, and the difference is settled at renewal.

See [licensing.md](licensing.md).
