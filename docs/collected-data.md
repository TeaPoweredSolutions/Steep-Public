# What Steep collects

Steep reads from MECM, Active Directory and Microsoft's public release and security data.

- **Collecting only reads.** A collection never changes anything in MECM or AD. Changes you start in Steep (an ADR, a client repair) use the accounts you allow and are audited.
- **Current state only.** Each collection replaces the last one. Steep keeps history only for:
    - the daily counts behind the trend charts
    - each device's identifiers (below)
    - the problems flagged on each device over time
    - software metering usage
    - the record of what Steep did

!!! info
    The examples below are real, from a lab. Every property by source, and where Steep uses it: [data-schema.md](data-schema.md).

## Device export

Patch compliance, Update deployment, Client health and OOB alerts all export the same columns, as XLSX or CSV. Only which devices are included changes: everything, a selected group, or problems only.

Example: CLIENT1, a Windows 11 client.

| Column | Example (CLIENT1) | Comes from |
|---|---|---|
| Device | CLIENT1 | MECM / AD |
| Domain | corp.contoso.com | AD / MECM |
| Needs attention | Yes | Steep |
| Operating system | Windows 11 25H2 | Steep (from build, SKU and branch) |
| Edition | Enterprise Evaluation | MECM hardware inventory (OperatingSystemSKU) |
| Channel | GeneralAvailability | MECM discovery (OSBranch) |
| Architecture | X64 | MECM hardware inventory (SystemType) |
| Language | en-US | MECM hardware inventory (OSLanguage) |
| Build | 10.0.26200.6584 | MECM discovery (BuildExt) |
| Installed update | Initial release | Steep + Microsoft release data |
| Installed update date | 2025-09-30 | Microsoft release data |
| Installed KB | *(blank)* | Microsoft release data |
| Patch Tuesday (B) release | 2026-09 B | Microsoft release data |
| Patch Tuesday (B) revision | 9445 | Microsoft release data |
| Latest release | 2026-09 OOB | Microsoft release data |
| Latest revision | 9457 | Microsoft release data |
| Compliance | Behind | Steep |
| Compliance detail | Has the initial 25H2 release; missing 2026-09 B. | Steep |
| Releases behind | 19 | Steep |
| Days behind | 349 | Steep |
| Missing exploited CVEs | CVE-2025-24990, CVE-2025-47827, … (13) | Microsoft security data (MSRC) |
| Missing CVE fixes | 1796 | Microsoft security data (MSRC) |
| Highest missing severity | Critical | Microsoft security data (MSRC) |
| Health issues | *(blank)* | Steep |
| Domain controller | No | AD (userAccountControl) |
| Site system roles | *(blank)* | MECM site systems |
| Liveness | Active | Steep |
| AD activity | Active | Steep (from AD) |
| Last seen | 2026-10-12 07:07:56Z | MECM / AD |
| MECM hierarchy | Lab | Steep connection name |
| Site code | CHQ | MECM |
| Resource ID | 16777220 | MECM |
| MECM client | Yes | MECM discovery (Client) |
| Client active | Yes | MECM client status |
| Client version | 5.00.9141.1011 | MECM client status |
| Production client version | 5.00.9141.1011 | Site database (AutoClientUpgradeConfigs) |
| Client approved | Yes | MECM client status |
| Client blocked | No | MECM client status |
| Obsolete MECM records | 0 | MECM (records marked obsolete) |
| Duplicate MECM records | 0 | MECM |
| Client check result | Passed | MECM client status (ccmeval) |
| Last client check | 2026-11-15 02:33:21Z | MECM client status |
| Client certificate | Self-signed | MECM client status |
| Boundary groups | Corp Boundary Group | MECM client status |
| Online now | No | MECM client status |
| Last policy request | 2026-10-12 07:07:56Z | MECM client status |
| Last hardware inventory | 2026-10-12 06:10:09Z | MECM client status |
| Last update scan | 2026-10-12 06:05:01Z | Site database (v_UpdateScanStatus) |
| Update scan error | *(blank: no error)* | Site database (v_UpdateScanStatus) |
| Last logon user | CLIENT1\Administrator | MECM discovery |
| Primary users | *(blank)* | MECM user device affinity |
| AD password last set | 2026-10-09 23:29:33Z | AD (pwdLastSet) |
| AD last logon | 2026-10-09 23:29:33Z | AD (lastLogonTimestamp) |
| AD account disabled | No | AD (userAccountControl) |
| AD OU | CN=Computers,DC=corp,DC=contoso,DC=com | AD (distinguishedName) |
| AD distinguished name | CN=CLIENT1,CN=Computers,DC=corp,DC=contoso,DC=com | AD (distinguishedName) |
| SID | S-1-5-21-3963388308-182971869-1286981655-1602 | MECM / AD |
| Classification notes | *(blank)* | Steep |

## Device identifiers

Steep keeps each device's hardware and account identifiers in order to follow systems through renames, reimages and client reinstalls. They aren't in the exports.

- Each device gets a **Steep device ID**, a random GUID.
- Steep remembers every identifier value it has seen for the device, and when.
- It notes renames, reimages, client reinstalls, and hardware and AD account changes.
- A device gone from MECM and AD for 30 days is marked retired. Its history is kept.

Example: CM1, the Primary site server in Microsoft's MECM Hydration Kit lab.

| Identifier | Example (CM1) | Comes from |
|---|---|---|
| SMBIOS GUID | 7982D194-B90C-4140-BE29-0AB6F2442F00 | MECM (SMS_R_System.SMBIOSGUID / v_R_System.SMBIOS_GUID0) |
| MECM client ID | GUID:3E816EA1-CC21-4E03-9502-50339425240C | MECM (SMSUniqueIdentifier / SMS_Unique_Identifier0) |
| Hardware ID | 2:549D1FA0E96F89FF571E0C2ABDF3926F59647F30 | MECM (HardwareID / Hardware_ID0) |
| AD object GUID | *(the computer object's objectGUID)* | AD (objectGUID) |
| SID | *(already collected; the AD computer account's SID)* | MECM / AD |

## Applications export

The Applications page exports one row per installed product per device: all of them, or one product family's.

Example: 7-Zip on CLIENT1, installed from Software Center.

| Column | Example (CLIENT1) | Comes from |
|---|---|---|
| Device | CLIENT1 | MECM |
| Domain | corp.contoso.com | AD / MECM |
| Family | 7-Zip 26.04 (x64 edition) | Steep (MSIs sharing an UpgradeCode) |
| Status | In a family | Steep |
| Match | Product code (certain) | Steep |
| Name | 7-Zip 23.01 (x64 edition) | MECM hardware inventory |
| Publisher | Igor Pavlov | MECM hardware inventory |
| Version | 23.01.00.0 | MECM hardware inventory |
| Architecture | 64-bit | MECM hardware inventory (which Add/Remove Programs view lists it) |
| Product code | {23170F69-40C1-2702-2301-000001000000} | MECM hardware inventory |
| Uninstall key | *(blank: it has a product code)* | MECM hardware inventory |
| Uninstall string | MsiExec.exe /I{23170F69-40C1-2702-2301-000001000000} | MECM hardware inventory |
| Confirmed CVEs | 4 | Steep (NVD affected versions; blank when not assessed) |
| Actively exploited CVEs | 0 | Steep (CISA KEV) |
| Unconfirmed CVEs | 1 | Steep (NVD without version data) |
| Vulnerability check | Checked | Steep: Checked, or why it wasn't (not in Steep's product list, version can't be compared) |
| Inventory | Installed Software | Which inventory class reported it |
| MECM application | 7-Zip 23.01 (x64 edition) | MECM applications |
| MECM revision | 2 | MECM applications |
| MECM hierarchy | Lab | Steep connection name |
| Resource ID | 16777220 | MECM |

## Telemetry export

Configure → Telemetry (Steep Architects only) builds an anonymous zip you can choose to email to Tea Powered Solutions. Nothing is sent automatically.

| The zip contains | The zip never contains |
|---|---|
| A manifest, and the catalog of what each counter and action means | Names of devices, applications, hierarchies or domains |
| Daily trend counts, and how much they changed | Versions, OS builds or revisions |
| If you tick them: action-log records (what Steep did and the fixes it surfaced) | CVE IDs |
| Severity levels, counts, exact times and result codes | Your license ID |
| Your Steep version, environment ID and peak client count | Who exported it |

!!! info
    Every device, application, version, OS build, hierarchy and domain is replaced by a random ID that only your Steep can map back. The original data is not hashed, or encrypted and it is impossible to reverse because the GUIDs in the export are random and unique to your Steep instance. The page previews every file before you download it, and each export is audited.
