# Permissions

Steep connects to MECM, SQL, Active Directory, file shares and devices with accounts you choose. This page lists what each account needs.

## Before you start: how much Steep can do

Each account gets the least privilege it needs. Together, they can make Steep powerful. Protect it like a site server.

| Part of Steep | What it can do | How to limit it |
|---|---|---|
| The Steep service | Runs as SYSTEM on its server. Local administrators there control Steep and every account saved in it. | Limit who is a local administrator on the Steep server. |
| The MECM write account | Operations Administrator can deploy applications and updates. Deploying software runs code as SYSTEM, so this is administrator access to every MECM client. | Leave the write account out if you only need reporting. Without it, Steep only reads MECM. |
| Secure Client Access | Signs in to devices as a local administrator. | Grant LAPS read rights only on the OUs you need. |
| Steep roles | Decide who can use all of the above through Steep. Every change is audited. | Give Steep roles only to the people who need them. |

## Accounts at a glance

| Account | Used for | Needs |
|---|---|---|
| MECM read account | Reading MECM through the SMS Provider | [Read account](#mecm-read-account) |
| MECM write account (optional) | Every change Steep makes in MECM | [Write account](#mecm-write-account) |
| Site database account | Reading the site database in SQL | [Site database account](#site-database-account) |
| AD account | Reading computers and accounts from AD | Read access to the domain (Domain Users is enough) |
| Source share accounts | Reading and writing application content | [Source share accounts](#source-share-accounts) |
| LAPS read account | Reading (and decrypting) devices' Windows LAPS passwords, so client tools can sign in to them | [LAPS accounts](#client-tools-secure-client-access) |
| LAPS reset account | Resetting a device's LAPS password after Steep uses it. Can be the same account as the read account. | [LAPS accounts](#client-tools-secure-client-access) |
| Global local admin account (optional) | A domain account in the local Administrators group on all devices (or several accounts with different scopes), such as your existing MECM client push accounts. For devices where you don't, or can't, use LAPS. | [Client tools](#client-tools-secure-client-access) |

!!! tip
    Best practice: one dedicated account per connection, so each can be audited and revoked on its own. One account for everything also works.

## MECM read account

| Requirement | Why |
|---|---|
| MECM security role **Read-only Analyst** | Read access to MECM through the SMS Provider. |
| Member of the local **SMS Admins** group on every SMS Provider server | Remote WMI access to `root\sms`. MECM usually adds administrative users itself, but it can lag. |
| Member of the local **Distributed COM Users** group on every SMS Provider server | Remote DCOM access for accounts that aren't administrators. It may not be needed in every environment. |
| **Not** a local administrator on the SMS Provider or site server | This account doesn't need it. |
| **Share: Read** and **NTFS: Read & execute, List folder contents, Read** on the application content source share | The Applications collector reads each MSI deployment type's MSI. Without access, those applications show "MSI couldn't be read" and aren't grouped into families. |

!!! info
    Steep connects to the SMS Provider on the MECM hierarchy server named in Connections. A few Application Management actions use the Configuration Manager console's own code, which may pick another SMS Provider. So add the read and write accounts to both "SMS Admins" and "Distributed COM Users" groups on every SMS Provider.

## MECM write account

Optional, one per MECM hierarchy (Connections → MECM hierarchies). Without it, Steep only reads that hierarchy.

Steep uses it for:

- Application Management and the MSI Auto-packager
- The ADR wizard and the Patch Tuesday software update sync
- Software metering rules and the metering maintenance task
- Approve, Unapprove, Block and Unblock

| Requirement | Why |
|---|---|
| MECM security role **Operations Administrator** | Every change Steep makes through the SMS Provider. Full Administrator also works but isn't needed. |
| Member of the local **SMS Admins** and **Distributed COM Users** groups on every SMS Provider server | As for the read account. |
| **Share: Read and Change** on the application content source share | The MSI Auto-packager copies installers there. Full Control isn't needed. |
| **NTFS: Modify, Read & execute, List folder contents, Read, Write** on the content source folder (This folder, subfolders and files) | Creating the publisher, name and version folders and copying files. |
| **No SQL Server permissions** | Steep never writes to the site database. Every change goes through the SMS Provider. |

!!! tip "Both accounts need an entry"
    The read and write accounts each need their own entry on the source share and folder. *Add* entries; never replace the existing ones.

## Site database account

The site database connection (Connections → Site databases) has its own account. It's required: some data, such as software update scan status, exists only in the site database, and SQL is much faster than the SMS Provider for large sites.

Give the account one of:

- **db_datareader** on the site database (`ConfigMgr_<site code>`). Simplest.
- **SELECT on the twenty views and one table Steep reads.** Most granular. A Steep version that reads a new view says so in its release notes.

| View or table | What it holds | What Steep uses it for |
|---|---|---|
| `v_R_System` | Each device's discovery record: name, SID, domain, client flags, OS build and revision, last user | Devices and client health |
| `v_GS_OPERATING_SYSTEM` | OS inventory: build, edition, workstation or server, language | OS classification on Patch compliance |
| `v_GS_COMPUTER_SYSTEM` | Computer inventory: system type | Architecture (x64, ARM64, x86) |
| `v_CombinedDeviceResources` | Client status: active, approved, blocked, last policy request, last hardware scan, online, version, health check, certificate, boundary groups | Client health checks and the device export |
| `v_UserMachineRelationship` | Primary users | The device export |
| `v_RA_System_SMSAssignedSites` | Each device's assigned site | Secure Client Push (optional: without it, the engineer picks the site) |
| `v_GS_WORKSTATION_STATUS` | When hardware inventory was last processed | Reading installed software only for devices that changed |
| `v_GS_INSTALLED_SOFTWARE` | Installed software: product code, name, publisher, version, uninstall string | Applications, product families, application vulnerabilities |
| `v_GS_ADD_REMOVE_PROGRAMS`, `v_GS_ADD_REMOVE_PROGRAMS_64` | Add/Remove Programs, 32-bit and 64-bit | Same as installed software, plus the architecture |
| `v_UpdateScanStatus` | Last software update scan time and error | Missing and failing update scan checks |
| `v_UpdateComplianceStatus` | Failed software update installations | The "Failures reported on any software update installation" check |
| `v_UpdateInfo` | Update titles and KB numbers | Naming failed updates; the .NET Framework update failures check |
| `vSMS_SUPSyncStatus` | Each software update point's catalog version, source and last sync | The Windows Update Agent repair |
| `v_CIAssignment`, `v_CIAssignmentTargetedMachines` | Software update deployments and the devices they target | The Software update patching report |
| `v_StateNames` | MECM's names for deployment and update states | The patching report shows states in words |
| `v_MeteredFiles`, `v_MeteredUser`, `v_MonthlyUsageSummary` | Software metering files, users and monthly usage | Software metering on the Applications pages |
| `AutoClientUpgradeConfigs` (table) | The production client version | The "Client version below production client version" check |

Windows authentication (a stored domain account) and SQL Server logins both work.

### Granting access: Grant-SteepSiteDatabaseRead_<version>.sql

The script is in the Steep package, next to Install-Steep.ps1. Its name has the Steep version, for example `Grant-SteepSiteDatabaseRead_0.7.0.sql`. Use the one from the package of the version you run: each release grants the views that version reads.

1. Open it in SQL Server Management Studio, connected to the site database's SQL Server as a sysadmin.
2. Set the values at the top:
    - the database name, for example `CM_ABC`
    - the account, `DOMAIN\name`
    - `@OnlySteepViews`: 1 for the views only, 0 for db_datareader
3. Run it.

It creates the login and database user if they're missing, grants the access, and lists what the account can read. For a SQL Server login, create the login first.

### Removing access: Revoke-SteepSiteDatabaseRead_<version>.sql

Deleting the login in SSMS isn't enough. The database user stays behind with its permissions, and they come back if the login is created again. Use the script, as a sysadmin:

| Setting | What it does |
|---|---|
| `@RemoveEverything = 0` | Revokes only what the grant script gave. The user and login stay. |
| `@RemoveEverything = 1` | Drops the database user, then the login. It keeps the login if it has a user in another database, holds a server role, or is the account running the script. |

!!! warning
    Be very careful with `@RemoveEverything = 1` if the account is shared with other applications or people.

**Test run:** both scripts have `@WhatIf`. With `@WhatIf = 1`, nothing changes and the Messages tab lists each statement. The grant script defaults to 0, the revoke script to 1.

## Source share accounts

Connections → **Source share** sets the accounts Steep uses on content source shares. By default, each is the hierarchy's own read or write account. Either can be another stored account or the Steep service account.

| Account | Used for | Needs |
|---|---|---|
| Read | Reading MSIs in application content, Choose MSI, metering rule offers | Read on the application content |
| Write | Copying new content (MSI Auto-packager), creating ADR package folders | Share: Read and Change, NTFS: Modify |

Accounts in use show "Source read" and "Source write" badges on Connection accounts, and can't be deleted.

## Scopes

Steep sees and changes MECM through its own accounts, not as the person using Steep. So:

- Give Steep's read and write accounts the **Default** scope and the **All Systems** and **All Users and User Groups** collections.
- Steep shows every device it collects to everyone with access. A user with limited MECM scopes can see more in Steep than in the console.
- Users whose MECM roles have limited scopes get no Steep access until an Architect adds them, and then only as Steep Readers.

!!! tip
    Don't restrict Steep's own accounts to non-default scopes. Dashboards would leave out devices, and changes could have unintended effects.

### The MECM administrative users list

Configure → Permissions shows MECM's administrative users and groups in three groups:

| Group | Meaning | Actions |
|---|---|---|
| **Enabled in Steep** | They get at least one Steep role. | **Add role** gives another role. **Remove from Steep** takes all roles away. |
| **Access not configured** | Found in MECM, but their roles aren't granted from MECM, or they were removed. | **Add role** or **Add to Steep**. |
| **Scoped users** | MECM limits their scopes. | **Add to Steep** as Steep Readers only, after an acknowledgement. |

- **Added by hand** is for users and groups that aren't MECM administrative users.
- Nothing on this page changes MECM.
- Until the "MECM administrative users" collector first runs, every MECM entry counts as scoped. Local administrators on the Steep server and the Steep service account always keep access.

## Connection account health

Steep checks every saved account in AD at start-up, every hour, and when you open Connection accounts. It reads the account with the AD connection's own account, never by signing in as it, so it can't add to the bad-password count.

| Badge | Meaning |
|---|---|
| Red | Locked out, disabled, expired, or password expired |
| Amber | The account or password expires within 7 days |
| OK | Fine. The tooltip shows when the password and account expire. |
| Not checked | Not a domain account (a SQL login) |

A red account also shows an alert on Connection accounts and a red badge on Configure.

## How passwords are stored

- Encrypted with Windows DPAPI for the Steep service account. Only that account, on that server, can decrypt them.
- Bound to Steep, so other software running as SYSTEM can't decrypt them.
- Write-only: never shown, exported or logged.
- Every change is audited.
- After moving Steep to another server, or changing its service account, enter the passwords again.
- The data folder (`%ProgramData%\TeaPowered\Steep`) is limited to SYSTEM and Administrators.

Domain accounts can sign in to SQL Server with Windows authentication. Steep uses a network-only sign-in for them (like `runas /netonly`).

More in [password-security.md](password-security.md).

## Client tools (Secure Client Access)

Client tools sign in to devices with each device's Windows LAPS password, a stored local administrator account (such as a client push account), or both. How each tool works is in [client-tools.md](client-tools.md#how-the-tools-work-technical-details).

### Requirements

| Requirement | Why |
|---|---|
| The Steep server is in the devices' domain, or one they trust | Encrypted LAPS passwords can only be decrypted on a domain-joined server. |
| TCP 445 (SMB) from the Steep server to each device | Steep copies files through ADMIN$ and starts a temporary service. Same as MECM's client push. |
| TCP 135 and dynamic RPC (optional) | WMI, for Read OS details' quick path. Every tool works without it. |
| The signing-in account is a local administrator on the device | The LAPS-managed account is one. A stored account must be in the local Administrators group. |
| The device is one Steep has collected from AD or MECM | Steep never sends credentials to a name someone typed. |
| Windows LAPS, with passwords in AD | Windows 10, Windows 11, Windows Server 2019 or later. Legacy Microsoft LAPS and Windows LAPS with passwords stored in Entra ID are not supported. Support for Windows LAPS with passwords stored in Entra ID is on Steep's roadmap. |

LAPS accounts, ticked on Configure → Client tools:

| Right | How to grant it |
|---|---|
| Read the password | `Set-LapsADReadPasswordPermission` on the devices' OUs |
| Reset the password | `Set-LapsADResetPasswordPermission` on the devices' OUs. It doesn't need to read the password. |
| Decrypt (encrypted passwords) | Make the reading account an authorized password decryptor in the LAPS policy. It also needs "Access this computer from the network" on the Steep server (Domain Users have it by default). |

Also:

- Steep never reads a domain controller's LAPS password. It uses a stored account there, if you have one.
- **Trace hardware inventory** also signs in to the device's management point and the site server.
- **Secure Client Push** reads ccmsetup.exe from each site's client folder (`\\<site server>\SMS_<code>\Client`) as the Steep server's computer account. With Steep on the site server, this already works. Otherwise, *add* Share: Read and NTFS: Read & execute for `<server>$`. The MECM client repair reads `MobileClient.tcf` the same way.

### What can block Secure Client Access

Steep signs in to devices with a network logon, like MECM's client push. Settings that refuse network logons for local accounts block LAPS. Settings that refuse them for everyone block every account.

| Setting | Effect | What to do |
|---|---|---|
| **Deny access to this computer from the network** with *Local account* (S-1-5-113) or *Local account and member of Administrators group* (S-1-5-114). Set by Microsoft's security baselines and CIS. | Blocks LAPS. | Use a stored domain account, or exempt the LAPS account if your policy allows. |
| **The LAPS-managed account is disabled**, for example the built-in Administrator by a baseline | LAPS still changes its password, but nobody can sign in with it. | Have LAPS manage an enabled account. |
| **Remote UAC for local accounts** (`LocalAccountTokenFilterPolicy`; "Apply UAC restrictions to local accounts on network logons") | Local administrators other than the built-in Administrator get "access denied". | Your security team's decision. Domain accounts aren't affected. |
| **NTLM restrictions** ("Restrict NTLM: Incoming NTLM traffic" on devices, "Outgoing NTLM traffic to remote servers" on the Steep server) | Local accounts can only use NTLM over the network, so LAPS is blocked. | Allow NTLM for these connections, or use a domain account. |
| **Admin shares turned off** (`AutoShareWks` or `AutoShareServer` = 0) | No ADMIN$, so Steep can't copy its files. | Turn admin shares back on. |
| **Firewall** | No file sharing from the Steep server. | Allow File and Printer Sharing (SMB-In) from the Steep server. |
| **Endpoint protection** | Some EDR products block or alert on a remote service. | Allow `SteepClientAgent_*` services and `%windir%\Steep\Jobs\*\SteepClientAgent.exe`. |
| **LAPS not applied**, or backing up to Entra ID instead of AD | No LAPS password for Steep to read. | Steep tries your stored accounts instead, if any. |

!!! info
    "Log on as a service" doesn't matter. Steep's temporary service runs as LocalSystem, which doesn't need that right.
