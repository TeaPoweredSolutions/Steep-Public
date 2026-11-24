# Data schema: what Steep collects, stores and uses

Every property Steep reads from MECM, Active Directory and Microsoft: where it's stored, and what uses it. The permissions are in [permissions.md](permissions.md); examples are in [collected-data.md](collected-data.md).

| Verdict | Meaning |
|---|---|
| **Used** | A dashboard, check or match depends on it. |
| **Export only** | Only appears as a column in an export. |
| **Not collected** | Steep doesn't read it. |

## Sizes

Measured on three lab databases and on 100,000 generated rows per table, then projected to 150,000 devices. The sizes include SQLite's indexes and page overhead.

| Data | Measured on disk (0.7.0) | At 150,000 devices × 150 installed products | Before 0.7.0 |
|---|---|---|---|
| MECM device record | ~1.4 KB per device (the record itself is ~1.3 KB) | ~210 MB | ~310 MB |
| AD computer record | ~710 bytes per computer | ~105 MB | ~115 MB |
| Installed software: each device's list of product numbers | ~410 bytes per device (~2.7 bytes per entry) | ~60 MB | ~210 MB |
| Installed software: each product version, stored once | ~290 bytes per product version | ~30 MB at 100,000 product versions | ~30 MB |
| Steep device IDs (`SteepDevices`, `SteepDeviceIdentifiers`, `SteepDeviceEvents`) | ~1.1 KB per device (6 identifiers of ~140 bytes) | ~165 MB | ~270 MB |
| Per-device application versions (`DeviceAppVersions`) | ~115 bytes per tracked app per device (most of it the app key, e.g. `upgrade:{UpgradeCode}`) | ~345 MB at 20 tracked apps per device | ~640 MB |
| Flag history (`FlagHistory`) | ~85 bytes per flag raised | ~310 MB if each device clears one flag a month, kept 730 days | ~1.1 GB |
| Software metering usage (`MeteringUsage`) | ~200 bytes per metered file × device used in all 13 months (~105 bytes for 6 months) | ~300 MB at 20 metered files, half used, each used every month | ~2.5 GB |
| Action log (`ActionLog`) | ~350 bytes per record | Depends on how many fixes Steep surfaces; measured in the scale test | |
| Update catalog | ~290 bytes per update | Catalog size, not device count: ~5 MB | |
| Applications | ~1.1 KB per application | Application count: ~1–5 MB | |
| Microsoft release and security data, application vulnerability data | 1.1 MB, compressed (24 months of Security Update Guide data plus 37 applications' CVEs) | Same | 7.8 MB |
| Trends, settings, decisions, run history, audit | Kilobytes | A few MB | |

!!! info
    In total about **1.2 GB** at 150,000 devices, or about **1.5 GB** with software metering in use (20 metered files, half used, each used every month; ~1.2 GB more at 100 metered files). Before 0.7.0 it was about 2.7 GB, or 5 GB with metering. The assumed rows per device for flag history, application versions and metering are estimates until the scale test.

How 0.7.0 stores things:

- **Dates** are numbers: the UTC time in milliseconds with the UTC offset it was written with, so they sort by the real moment whatever the time zone (8 bytes instead of 33 characters). Calendar days are numbers like 20261122.
- **Devices** have a number inside the database (`SteepDevices.Number`, never reused); the other device tables refer to it instead of the 36-character Steep device ID. The Steep device ID stays on `SteepDevices`, and telemetry carries it.
- **Repeated names** (flag names, severities, hierarchy names) are stored once in `StoredNames` and referred to by number.
- **Installed software**: each device's product numbers are a packed binary list of the gaps between them.
- **Software metering usage**: one record per metered file per device, holding up to 13 calendar months (launches, Remote Desktop launches, time open, last use), people and SYSTEM or computer accounts apart. Which account used it isn't kept: the vulnerability and the fix are the device's.
- **Reference data** (Microsoft release and security data, vulnerability data) is compressed.

!!! info
    SQLite never shrinks its file by itself: space freed by retention clean-up is reused, but the file stays at its largest size. Since 0.7.0 the database uses incremental auto-vacuum, and after each daily retention clean-up Steep gives free space back to Windows when it's more than 20% of the file (see [Database maintenance](installation.md#database-maintenance)).

## MECM devices & client health (collector: MECM devices)

Through the SMS Provider (WMI), filtered by the account's MECM security role. One record per device.

### SMS_R_System (discovery data)

| Property | Stored as | Used for | Verdict |
|---|---|---|---|
| ResourceID | ResourceId | Key; joins every other class and view | Used |
| Name | Name | Device name everywhere | Used |
| SID | Sid | Matching to AD computers | Used |
| FullDomainName | FullDomainName | Matching to AD when there's no SID; domain scope | Used |
| SMBIOSGUID | SmbiosGuid | Steep device ID (main hardware key; trends and telemetry) | Used |
| SMSUniqueIdentifier | MecmClientId | Steep device ID (MECM client ID; changes on reinstall/reimage) | Used |
| HardwareID | HardwareId | Steep device ID (MECM's conflicting-record key) | Used |
| Client | IsClient | "No client" health checks | Used |
| Obsolete | IsObsolete | Choosing the main record when a device has duplicates | Used |
| BuildExt | BuildExt | OS build **and revision** (patch level): the only source of the revision | Used |
| Build | Build | OS build when BuildExt is missing | Used |
| OSBranch | OsBranch | LTSC vs GA channel | Used |
| OperatingSystemNameandVersion | OsNameAndVersion | Server vs workstation (English on every client) | Used |
| LastLogonUserName, LastLogonUserDomain | LastLogonUser | Device export | Export only |

### SMS_G_System_OPERATING_SYSTEM (hardware inventory)

| Property | Stored as | Used for | Verdict |
|---|---|---|---|
| BuildNumber | InvBuildNumber | OS build when discovery data has none; inventory health | Used |
| OperatingSystemSKU | OperatingSystemSku | Edition (Enterprise, LTSC, Server…), servicing, evaluation | Used |
| ProductType | ProductType | Workstation / server / domain controller | Used |
| OSLanguage | OsLanguage | Language breakdown on Patch compliance; export | Used |
| Caption | (not stored) | Nothing: Operating system, Edition and Channel already say the same | **Not collected** |

### Other classes

| Class | Property | Stored as | Used for | Verdict |
|---|---|---|---|---|
| SMS_G_System_COMPUTER_SYSTEM | SystemType | SystemType | Architecture (x64 / ARM64 / x86) on Patch compliance | Used |
| SMS_Admin, SMS_Role (collector: MECM administrative users, hourly) | LogonName, AdminSid, IsGroup, DisplayName, RoleNames; RoleID, RoleName, IsBuiltIn | MecmAdministrator | Steep roles (Permissions) | Used |
| SMS_CombinedDeviceResources | ClientActiveStatus | IsActive | Client activity checks | Used |
| | LastPolicyRequest | LastPolicyRequest | "No policy request" check; liveness | Used |
| | LastHardwareScan | LastHardwareScan | "Old hardware inventory" check | Used |
| | CNIsOnline | IsOnline | Device export ("Online now") | Export only |
| | CNLastOnlineTime | LastOnline | Liveness (dormant / stale; device export only) | Used |
| | ClientVersion | ClientVersion | Device export | Export only |
| | LastActiveTime | LastActivity | Client tools fix pages ("Last activity"), device export | Export only |
| | IsVirtualMachine | IsVirtualMachine | Client tools fix pages ("Virtual"), device export | Export only |
| | IsApproved | IsApproved | "Client approved = no" check | Used |
| | IsBlocked | IsBlocked | "Client blocked = yes" check | Used |
| | ClientCheckPass | ClientCheckResult | "Client health check failed / no results" (1 passed, 2 failed, 3 no results) | Used |
| | LastClientCheckTime | LastClientCheck | "Client health check out of date" | Used |
| | ClientCertType | ClientCertType | "No client certificate" (0 none, 1 self-signed, 2 PKI) | Used |
| | BoundaryGroups | BoundaryGroups | "Not in a boundary group" | Used |
| v_UpdateComplianceStatus (site database) | ResourceID where Status = 2 and LastEnforcementMessageID in 6, 11, 14 | UpdateInstallFailed | "Failures reported on any software update installation" | Used |
| AutoClientUpgradeConfigs (site database table) | ClientVersion | ProductionClientVersion | "Client version below production client version" | Used |
| SMS_G_System_WORKSTATION_STATUS | TimeStamp | (fills LastHardwareScan) | Hardware inventory time when client status has none | Used |
| SMS_UserMachineRelationship | UniqueUserName (IsActive = 1) | PrimaryUsers | Device export | Export only |

### Site database (SQL, required)

| View | Columns | Stored as | Used for | Verdict |
|---|---|---|---|---|
| v_UpdateScanStatus | ResourceID, LastScanTime, LastErrorCode | LastUpdateScan, LastUpdateScanError | "Missing / failing update scan" checks | Used |

!!! info
    The SMS Provider has no class for update scan status, which is why SQL is required.

### Devices from the site database

On Connections, a hierarchy can read devices from the site database ("Connection protocol: Site database (SQL)"). The SMS Provider is then asked only for `SMS_R_System.ResourceID`, so MECM security roles and scopes still decide which devices are included. The fields above come from these views, for those devices only:

| View | Columns |
|---|---|
| `v_R_System` | Name0, SID0, Full_Domain_Name0, Client0, Obsolete0, BuildExt, Build01, OSBranch01, Operating_System_Name_and0, User_Name0, User_Domain0, SMBIOS_GUID0, SMS_Unique_Identifier0, Hardware_ID0 |
| `v_GS_OPERATING_SYSTEM` | BuildNumber0, OperatingSystemSKU0, ProductType0, OSLanguage0 |
| `v_GS_COMPUTER_SYSTEM` | SystemType0 |
| `v_CombinedDeviceResources` (joined on MachineID) | ClientActiveStatus, LastPolicyRequest, LastHardwareScan, CNIsOnline, CNLastOnlineTime, ClientVersion, LastActiveTime, IsVirtualMachine, IsApproved, IsBlocked, ClientCheckPass, LastClientCheckTime, ClientCertType, BoundaryGroups |
| `v_GS_WORKSTATION_STATUS` | TimeStamp |
| `v_UserMachineRelationship` | UniqueUserName, where RelationActive = 1 |
| `v_RA_System_SMSAssignedSites` (optional) | SMS_Assigned_Sites0 |

!!! info
    If the site database can't be read, or is missing devices the SMS Provider lists, Steep falls back to the SMS Provider and logs a warning.

## MECM installed software (collector: MECM installed software)

The large one. Stored per device (one row per device, holding its list), so a delta rewrites only the devices whose inventory changed. With a site database (the normal path), only devices whose `v_GS_WORKSTATION_STATUS.TimeStamp` changed are read, limited to devices the SMS Provider returned (RBAC).

| Site database view (SMS Provider class) | Column | Stored as | Used for | Verdict |
|---|---|---|---|---|
| v_GS_WORKSTATION_STATUS | TimeStamp | (item version) | Which devices changed since the last read | Used |
| v_GS_INSTALLED_SOFTWARE (SMS_G_System_INSTALLED_SOFTWARE) | SoftwareCode0 | ProductCode | Matching to MECM deployment types; identity | Used |
| | ARPDisplayName0, else ProductName0 | Name | Identity; families; suggestions | Used |
| | Publisher0 | Publisher | Identity; suggestions | Used |
| | ProductVersion0 | Version | Versions in families | Used |
| | UninstallString0 | UninstallString | Export; future source of product codes for EXE-wrapped MSIs | Used |
| | InstallSource0 | (not stored) | Nothing | **Not collected** |
| v_GS_ADD_REMOVE_PROGRAMS / _64 (SMS_G_System_ADD_REMOVE_PROGRAMS / _64) | ProdID0 | ProductCode, or UninstallKey (EXE installers) | Matching; identity | Used |
| | DisplayName0 | Name | Identity | Used |
| | Publisher0 | Publisher | Identity | Used |
| | Version0 | Version | Versions | Used |
| | (which view) | Architecture | 32- vs 64-bit, kept separate | Used |
| (which class) | | Source | Installed Software vs Add/Remove Programs; export | Used |

**Not collected: the install source.** It's mostly a temporary path (`C:\Windows\ccmcache\<n>\`, which MECM deletes) or `C:\` for manual installs, so nothing Steep can rely on.

**Uninstall string:** the same on every device for a product version. For MSIs it's `MsiExec.exe /I{ProductCode}`, which can supply product codes.

## AD computers (collector: AD computers)

LDAP, signed and sealed. Changes only (by `uSNChanged`) every hour; a full read daily.

| Attribute | Stored as | Used for | Verdict |
|---|---|---|---|
| name | Name | Device name; matching | Used |
| objectSid | Sid | Matching to MECM | Used |
| objectGUID | ObjectGuid | Steep device ID for devices only in AD (trends and telemetry) | Used |
| distinguishedName | DistinguishedName | Device export (OU) | Export only |
| operatingSystem | OperatingSystem | Server vs workstation, for devices only in AD | Used |
| operatingSystemVersion | OperatingSystemVersion | OS build for devices only in AD | Used |
| pwdLastSet | PasswordLastSet | Activity (machine password age) | Used |
| lastLogonTimestamp | LastLogonTimestamp | Activity ("AD active / inactive") | Used |
| userAccountControl | Disabled, IsDomainController | Disabled accounts; domain controllers | Used |
| dNSHostName | DnsHostName | Matching servers to their MECM site system roles (Client health: site systems without a client); correct when a server's DNS suffix differs from its AD domain | Used |
| whenCreated | (not stored) | Nothing | **Not collected** |
| uSNChanged (rootDSE: highestCommittedUSN, dsServiceName, invocationId) | Collector state | Deltas | Used |

## Per hierarchy, not per device

Sized by the catalog or the number of applications, not by devices.

| Collector | Source | Properties | Notes |
|---|---|---|---|
| MECM software updates & site settings | SMS_SoftwareUpdate (Windows cumulative updates only) | CI_ID, ArticleID, LocalizedDisplayName, IsDeployed, IsExpired, NumMissing, NumPresent: used. DatePosted (the console's "Date Released"), IsSuperseded: stored for later features | Update deployment dashboard |
| | SMS_SCI_Component (discovery data manager; WSUS configuration manager) | Client push settings; supersedence rules | Client health (push), Update deployment |
| | SMS_SystemResourceList | ServerName, SiteCode, RoleName | Site systems without a client |
| MECM applications | SMS_Application (latest) | CI_ID, ModelName, LocalizedDisplayName, Manufacturer, SoftwareVersion, CIVersion, IsDeployed, IsExpired | All used (families, revisions) |
| | SMS_DeploymentType (latest) + SDMPackageXML | Name, technology, revision; detection product codes, install command, content location | All used; definitions re-read only when the revision changes |
| | The MSI each deployment type names | ProductCode, UpgradeCode, ProductVersion, ProductName, Manufacturer; File table programs (.exe name, version, language) | Families by UpgradeCode; MSI and Metered badges |
| | SMS_MeteredProductRule | RuleID, ProductName, FileName, OriginalFileName, FileVersion, LanguageID, SiteCode, Enabled | Metered badges |
| | SMS_MeteredFiles, SMS_MeteredUser (site database: v_MeteredFiles, v_MeteredUser) | MeteredFileID, FileName, FileVersion, LanguageID; MeteredUserID, Domain, UserName, FullName | Kept once seen, so stored usage keeps its names |
| | SMS_MonthlyUsageSummary (site database: v_MonthlyUsageSummary, only devices the SMS Provider returned) → **MeteringUsage table** | FileID, ResourceID, MeteredUserID, TimeKey (yyyymm), UsageCount, TSUsageCount, UsageTime (seconds), LastUsage | Since 0.7.0: one record per file × device holding up to 13 calendar months (this one included), the accounts added together, people and SYSTEM or computer accounts (from SMS_MeteredUser) kept apart; which person used it isn't kept. The current and previous month are re-read each collection; closed months are kept after MECM deletes them, until they're 13 months old. **Per device: never in telemetry.** Scale: ~1.5 million records (~300 MB when each is used every month) at 150,000 devices × 20 metered files, half used |
| (live, Metering page) | SMS_MeterData | ResourceID, MeteredUserID, FileID, StartTime, EndTime, StillRunning, InTSSession | Recent launches; MECM keeps them 5 days; not stored |
| (made by Steep) | **Chosen MSIs** (`steep.chosen-msi`, per hierarchy) | Application ModelName, deployment type CI_ID (or program name), source (content source or device), content source and path, or device, cached MSI path, installed version and Steep's copy (data folder InstalledMsis, by SHA-256), program headers and installed paths read from the device, SHA-256, size, last write, the MSI's properties and programs, who, when, checks accepted, last file problem | Choose MSI (design-msi-metering.md part 4): applied to the deployment type wherever applications load; re-checked each applications collection; never replaced by a collection |

## Microsoft (collector: Microsoft release & security data)

Not per environment: the same everywhere (1.1 MB in the database, compressed, since 0.7.0).

- **Windows release information pages** (learn.microsoft.com): stored as downloaded, so parser improvements apply to cached data.
- **MSRC Security Update Guide** (CVRF API): each month's document is ~20 MB; Steep keeps only the Windows vulnerabilities, fixes and exploitation status, and re-downloads a month only when Microsoft revises it.

See [scalability.md](scalability.md) for sizes at 10,000, 50,000 and 150,000 devices. Sizes at scale are estimates until proven in a production environment.
