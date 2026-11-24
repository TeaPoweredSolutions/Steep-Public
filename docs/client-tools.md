# Client tools and Secure Client Access

Client tools work on devices directly, for the cases MECM can't handle by itself: a device with no client, a client that doesn't report, or logs MECM doesn't collect.

| Tool | What it does |
|---|---|
| **Read OS details** | Reads the Windows version, revision (UBR), edition and architecture, for devices MECM has no revision for. The dashboards use it at once. The client dashboards use values from MECM discovery/DDRs and hardware inventory and these values will be missing on the dashboard if the system does not have a working client or discovery data in MECM. |
| **Gather logs** | Brings back a zip of the device's troubleshooting logs. |
| **Trace hardware inventory** | Follows a device's last inventory report from the client to the site, and says where it broke. |
| **Software update scan report** | Shows what the device's last software update scan found. |
| **Software update patching report** | Traces each software update deployment through the client's logs, and gives each update's final status. |
| **Secure Client Push** | Installs, repairs or upgrades the MECM client, like MECM's client push. |
| **Secure Client Repair** | Repairs BITS, WMI, the .NET Framework, the MECM client and the Windows Update Agent, least invasive first. |
| **Read installed MSI** | Reads an application's MSI from a device that has it installed (from Choose MSI on the Applications pages). |
| **Client actions** | Runs the MECM client's own actions: health check, machine policy, hardware inventory, software update scan. |
| **Approve, Block** | Approves, unapproves, blocks or unblocks clients in MECM. |

!!! info
    Run them from **Clients → Client tools**. Set them up in **Configure → Client tools**.

## The Client tools page

**Clients → Client tools** lists every fix and tool. Choose one and click **Open**, or double-click it. **History** below the list shows every run.

- **Fixes** are grouped like the problems on Client health. Each shows how many devices in the current scope have that problem. Fixes with no affected devices are greyed out, but still open.
- **Tools** each open a page where you name the devices to run on, with the tool's history.
- **Investigate** on a Client health row opens that problem's fix page. Clicking a device name opens it with that device at the top.

A fix page shows:

- how many devices have each problem it covers
- why the problem happens and how to fix it
- the affected devices, with an XLSX/CSV export. When a page covers several problems, they're numbered and **Show** filters to one.
- a tick box per device. **Test** checks whether Steep can connect to the ticked devices (or every row shown).
- on some pages, **Fix the ticked devices**: client actions or Secure Client Push

!!! info
    Dates show on your own clock (your browser's time zone), with the time on hover.

### Test: is the device reachable?

**Test** asks, from the Steep server and without signing in, whether each device answers on file sharing (TCP 445), which Secure Client Access needs. No ping is used.

| Result | Meaning |
|---|---|
| Reachable | File sharing answers. |
| No file sharing | The device answers, but not on 445: a firewall. |
| No answer | Off, away or blocked. |
| Name not found | DNS has no address for it. |

!!! info
    Results are kept for 10 minutes. MECM's own status at the last collection is in the **Last online** column.

## Secure Client Access

Client tools sign in to devices themselves. They're off until an Architect sets up at least one way to sign in:

| Way to sign in | How it works |
|---|---|
| **Each device's own Windows LAPS password** | Read from AD only when a tool needs it. By default, the device resets it within a few minutes of the run, and Steep also flags it as expired in AD. See [Why one password shouldn't open every device](why-secure-client-access.md). |
| **Global local admin accounts**, such as your MECM client push accounts | For devices where you don't, or can't, use LAPS. |

With both, Steep tries LAPS first, then each global account in turn.

- **Before any password is read**, Steep checks that the device answers on TCP 445. If it doesn't, the run stops: no password is read, and no account is used.
- Steep never shows, logs or audits a password. It audits which account signed in, to which device, and why.
- How the password is protected: [password-security.md](password-security.md).

### Which accounts Steep uses for LAPS

On Configure → Client tools, tick which accounts may read (and decrypt) LAPS passwords, and which may reset them. For each device, Steep checks its computer object's permissions in AD and uses the first ticked account allowed to read, and the first allowed to reset. There's no OU mapping to maintain. If no ticked account is allowed, the run says so and names the device's OU.

The rights each account needs are in [permissions.md](permissions.md#client-tools-secure-client-access).

### Verify access

Configure → Client tools → **Verify access** signs in to one device, one way, and checks:

| Check | What it proves |
|---|---|
| File sharing (ADMIN$) | Steep can sign in over SMB and is an administrator. A test file is written and deleted. |
| Service Control Manager | Steep may create services, which every tool needs. Nothing is created. |
| WMI (Win32_OperatingSystem) | The quick path for Read OS details. Optional. |

Each check is audited. A LAPS password used for it is reset like any other use.

### Requirements and what can block it

Secure Client Access needs TCP 445 from the Steep server, a local administrator account on the device, and AD rights for LAPS. Some security settings block it, such as "Deny access to this computer from the network" for local accounts. Both lists are in [permissions.md](permissions.md#client-tools-secure-client-access).

## Who can use it

| Action | Who |
|---|---|
| Set up Secure Client Access and the tools, Verify access, Automatic Secure Client Push settings | Steep Architects |
| Run client tools and client actions, Secure Client Repair, download gathered logs and Microsoft's repair tools | Steep Client Administrators and Architects |
| Read installed MSI | Steep Client Administrators and Architects, with Manage applications |
| Automatic Secure Client Push's queue, and **Pause all automatic remediation** | Steep Client Administrators and Architects |
| See run history, open scan and patching reports, export a scan report's updates | Anyone with Steep access |
| Download a patching report's logs | Steep Client Administrators and Architects |

A new install grants only Steep Architects from MECM. An Architect turns on Steep Client Administrators (from MECM's Operations Administrator role) on Configure → Permissions, or adds people by hand.

**Trial mode** (no license):

- Only **Read OS details** runs, so you can check Secure Client Access works. Steep refuses every other tool on the server, not just on the page.
- Run history shows only Read OS details, with device names and versions hidden. Gathered logs can't be downloaded.
- Automatic Secure Client Push keeps its queue but never pushes.
- **Test** still works.

## Fix page tools

### Client actions

| Action | What runs on the device | Offered on |
|---|---|---|
| **Run the client health check** | `CcmEval.exe`. The run shows the result and any check that failed. | Client health check, Inactive clients |
| **Machine policy retrieval** | Machine Policy Retrieval and Evaluation Cycle | Client health check, Inactive clients, Hardware inventory, Software update scan, Install failures |
| **Hardware inventory** | Hardware Inventory Cycle. **Full resync** sends everything, not only changes. | Hardware inventory |
| **Software update scan** | Software Updates Scan Cycle, then Assignments Evaluation | Software update scan, Install failures |

- They're the same actions as the Configuration Manager control panel's Actions tab.
- Four devices run at a time. **History** shows what each device said.
- A run fails clearly if the device has no client, or SMS Agent Host isn't running.
- The fix page's dates change only after the client reports to the site and Steep's next MECM collection.
- Each run is audited.

### Approve, Unapprove, Block and Unblock

The **Client approval and blocking** page offers only the changes that apply:

| The client is | Offered |
|---|---|
| Not approved | **Approve**, **Block** |
| Blocked | **Unblock**, then **Approve** if it's still not approved |
| Approved | **Block**, **Unapprove** |

The **AD** column says where each device's account is: **In AD**, **Not in AD**, **Unknown domain** (one Steep doesn't read) or **Workgroup**. Unknown domains and workgroups are listed first: an unapproved client you don't recognize there is the one to block.

- Steep makes the change in MECM with the hierarchy's **write account**, as the console does. Nothing runs on the device.
- The page updates at once, without waiting for a collection.
- A blocked client can't get policy or content. It stays blocked after a reinstall with the same identity.
- Each change is audited.

### Trace hardware inventory

On the **Hardware inventory** page, **Trace hardware inventory** follows each ticked device's last report from the client to the site, the way Philip Loftin's [Troubleshooting Hardware Inventory in MECM](https://blog.loftinnc.com/troubleshooting-hardware-inventory-in-mecm) does by hand:

1. **BITS** on the device: running and not disabled.
2. **InventoryAgent.log**: the last hardware inventory cycle, its report type, and "Successfully sent report". Three or more full or resync reports in a row are flagged.
3. **CcmMessaging.log**: the upload of that report.
4. **MP_Hinv.log** on the device's management point: the report, and the MIF file it became.
5. **Status messages** from the data loader about that MIF:
    - **2719**: larger than the site's **Max MIF Size**
    - **2700**: invalid or corrupt; moved to BADMIFS
    - **2703**: couldn't be processed (inventory the site doesn't have, or a syntax error); moved to BADMIFS. The site asks the client for a resync itself.
6. **The site server**: the current Max MIF Size (with Recast Software's 50 MB recommendation when it's lower), **dataldr.log**'s lines about the MIF, and the file in **BADMIFS** with its size.

It only reads. The result is in History: what's wrong and what to do, then each step with ✓ or ✗.

**Reports the site rejected:** every MECM collection also reads messages 2700, 2703 and 2719 from the last 7 days. The Hardware inventory page shows how many per site. The messages name only the MIF, never the device, so use the trace to find which devices.

To fix reports that are too large, raise `Max MIF Size` under `HKLM\SOFTWARE\Microsoft\SMS\Components\SMS_INVENTORY_DATA_LOADER` on the site server (default 5,000,000 bytes; Recast recommends 50 MB, `0x3200000`), or find the inventory class that makes them large. A few 2703s are routine. Many that keep coming usually mean a custom inventory class doesn't match what clients report.

## Read installed MSI

From **Choose MSI** → **Read it from a device that has it installed**: pick the installed product and a device that has it. For applications whose installer is a setup.exe that installs an MSI.

- Steep copies the product's MSI from Windows Installer's cache, and the programs it installed, and reads them on the Steep server.
- It only reads. Nothing on the device changes, and nothing from the device is run.
- Steep never uses WMI's Win32_Product class, which can make Windows Installer repair products.

## Gather logs

Built-in rules (each can be disabled on Configure → Client tools, but not removed):

- the MECM client and ccmsetup logs
- CBS, DISM and Windows Update (its .etl files, and the decoded WindowsUpdate.log)
- setupapi and Windows Setup (Panther)
- the computer's Group Policy `Registry.pol` (a corrupt one stops policy and software updates)
- Steep's own client log
- the System, Application, Setup, Windows Update, Group Policy and BITS event logs, last 14 days

Add your own paths there too, or **Add the install log folder** (Settings → Install logs).

- Files past the size limit are left out and listed in `Steep-summary.txt` inside the zip.
- Zips are kept in Steep's data folder for the number of days you set. Every download is logged.

## Software update scan report

Shows what the MECM client found at its last software update scan, as one page. It only reads.

- Run it from **Tools → Software update scan report**, or from the *Software update scan* and *Software update install failures* fix pages.
- **View report** in the run's history opens it. Reports are kept for 90 days.

The report has:

- **Scan details:** OS and build revision, last restart, pending restart, the scan source (the software update point), the last scan, and the device's catalog version next to the software update point's. A warning shows when they differ.
- **Deployments:** each software update deployment to the device, whether it's compliant, Available or Required, its start and its deadline.
- **Updates:** every update the scan found, with KB, name, status and (for cumulative updates) the build revision. Filter by KB, name or status. **Export updates (CSV)**. Updates for other Windows versions are hidden; **Show updates for other Windows versions** shows them.
- **Windows Update install history**, with the error for any install that failed.

| Status | Meaning |
|---|---|
| Installed | The scan found it installed. |
| Required | Missing, and deployed to the device. |
| Missing | Missing, and not deployed to the device. |
| Not required | Not in the scan: Windows Update found it not applicable. |
| Not in catalog | Not in the scan, and not in the site's catalog (or expired there). |
| Not reported | Not in the scan, and the device hasn't scanned since the site received it. |

The latest cumulative updates for the device's Windows version (the last three Patch Tuesdays, plus any newer out-of-band release) are highlighted. Those the scan didn't list are at the top, marked **from the site**, with the reason. Windows Update doesn't report updates it found not applicable, so Steep works that out from the site's catalog.

!!! info
    Can't find an update you expect? Either the software update point's catalog doesn't have it (products, classifications or sync), or Windows Update found it not applicable.

## Software update patching report

Traces a device's software update deployments through its MECM client logs, as one page: each deployment, each update's final status, and the log lines behind each step. It only reads.

- Run it from **Tools → Software update patching report**, from the *Software update scan* and *Software update install failures* fix pages, or from a scan report.
- **View report** in the run's history opens it. Reports and their logs are kept for 90 days.

It follows the log flow in Manish Bangia's [Track SCCM Software Update patch deployment through client log flow](https://www.manishbangia.com/track-sccm-software-update-patch-deployment-through-client-log-flow/): PolicyAgent, ScanAgent, UpdatesStore, UpdatesDeployment, UpdatesHandler, WUAHandler, CAS, LocationServices, ContentTransferManager, DataTransferService and RebootCoordinator. Steep only interprets lines whose exact text it has seen in real client logs. Every other line that names an update is still listed, as written.

The report has:

- **Summary:** how many updates have each status, the last restart, and any pending restart. It says when a server can't restart on its own.
- **A card per deployment:** Available or Required, start, deadline, when its policy arrived, its last state, and its updates with their final status. **Steps** shows each update's way from policy to state message, with the log line for each step. Updates with a problem open by themselves.
- **Deployments whose policy isn't on the device:** deployments the site targets at the device that never reached it.
- **Content downloads:** distribution points found, download jobs, and whether the content is in the client cache.
- **Restarts:** RebootCoordinator.log's restart lines.
- **Logs read:** how far back each log reaches.
- **Download the logs (zip).**

| Status | Meaning |
|---|---|
| Installed | Installed (after its restart, when one was needed). |
| Installed, restart pending | Installed, waiting for a restart. |
| Installation failed | The install failed or couldn't start. The error is shown in words. |
| Interrupted by a restart | The device restarted during the install, and nothing says it finished. |
| Content not found on a distribution point | No distribution point has the content for this device's boundary group. |
| Downloading, Waiting to install, Installing | Where it is now. |
| Waiting for a maintenance window | Waiting for the next maintenance window. |
| Waiting for the user | Waiting for someone to log on or off. |
| Waiting for the deadline | Available, or Required with a deadline ahead, and not started in Software Center. |
| Not active yet | The deployment's start time hasn't come. |
| Required, not started | The scan found it missing, and the logs say nothing more. |
| Not required | Windows Update found it not applicable. |
| Superseded | A newer update replaces it. |
| Unknown | Nothing says anything about it (the logs may have rolled over). |

## Secure Client Repair

Repairs work on devices whose MECM client or Windows components are broken. They don't need the MECM client or WMI.

Each repair is a list of steps, least invasive first. Each step has a **level**:

| Level | What it may do |
|---|---|
| 0 | Diagnose: read only |
| 1 | Restart or start a service |
| 2 | Clear a cache or a queue of stuck work |
| 3 | Reset a component |
| 4 | A destructive reset. The confirmation says what is lost. |

- You choose the highest level Steep may use. Levels 3 and 4 ask for confirmation.
- Steep always diagnoses first, then works up the steps and **stops as soon as nothing is wrong**.
- Steps above your level don't run, but check whether they would have helped. The run lists them as **Needs a higher level**, with what they would do.

| Run status | Meaning |
|---|---|
| Succeeded | Fixed, nothing wrong, or diagnosed as asked. |
| Not fixed | The repair ran but couldn't fix the problem: a cause outside the device, a failed step, or a higher level needed. |
| Restart pending | Waiting for a restart. |
| Failed | Steep couldn't run the repair or get its result. |

**On the device**, under `%windir%\Steep` (only administrators can write there):

| File | Purpose |
|---|---|
| `Repair\ledger.json` | The last 50 runs and every step. A run interrupted by a restart carries on after its last step. |
| `Repair\running.lock` | Exists while a repair runs, so a second one waits. A stale lock is cleared by the next run. |
| `Logs\SteepRepair.log` | What happened, in CMTrace format. Gather logs brings it back. |
| `Backups\` | Anything a destructive step moved aside. |

Before it imports anything, each run checks its PowerShell modules' SHA-256 against the copies Steep shipped. (The scripts aren't code-signed yet.)

### Restarts

Only levels 3 and 4 can restart a device. Nothing below ever does. Every restart box on the repair page starts unticked:

- **Allow a restart outside a maintenance window:** Workstations, Servers.
- **Allow a restart inside a maintenance window:** Workstations, Servers. With only this ticked, a device without a window isn't restarted.
- **When:** after a number of minutes (60 by default), or at the start of the next maintenance window. Windows for all deployments or software updates count.
- **Tell the signed-in user:** Windows' own restart notice. With nobody signed in, the restart comes after 1 minute.

The device schedules the restart itself (`shutdown /a` cancels it). Afterwards, Steep checks the device answers and runs the same repair again, which carries on where it stopped. A device that hasn't restarted is tried every hour for up to two days.

Defaults for automatic repairs are on Configure → Client tools → **Secure Client Repair: automatic repairs**, including **Only run repairs while a maintenance window is open** (on for workstations and servers).

### BITS repair

For downloads that don't move: MECM content that never arrives, a client install stuck downloading, or a queue full of old jobs.

| Level | Steps |
|---|---|
| 0 · Diagnose | The BITS service and its startup type; every job in the queue, by state; the stuck ones |
| 1 · Start BITS | Sets BITS back to Manual if disabled and starts it; resumes ccmsetup's download |
| 2 · Cancel stuck downloads | Cancels MECM downloads in error, suspended, or not moving for a day (MECM retries them). Cancels other downloads in error or older than 7 days. Never cancels a download that's moving. |
| 4 · Reset the BITS queue | Only if BITS still doesn't work. Moves the queue database aside and starts BITS empty. **Every download on the device is lost.** If BITS doesn't start, the old queue is put back. |

Fixed when BITS isn't disabled, its queue can be read, no MECM download is stuck, no other download is in error or older than 7 days, and the queue isn't nearly full.

If a Group Policy disables BITS, it will disable it again: check the device's policies.

### MECM client repair

For clients that don't work: no policy, not reporting, SMS Agent Host stopped or crashing, or shown as inactive.

**Level 0 · Diagnose** checks:

- **The client:** SMS Agent Host, its version against the production client, a stuck ccmsetup and its last return code, crashes in the last 14 days, and proof it works: a client ID, an assigned site, a known management point, machine policy, and PolicyAgent.log written in the last 24 hours.
- **Known traps:** provisioning mode left on, a wrong trusted root key, a corrupt `Registry.pol`, the domain trust, the clock, free disk space, a pending restart.
- **Dependencies:** the client's services, WMI, MECM downloads in BITS, .NET, the last ccmeval result.
- **The client authentication certificate**, against Microsoft's PKI requirements (PKI clients only).
- **Maintenance windows**, and when each opens next.

| Level | Steps |
|---|---|
| 1 · Restart and switch back | Client services set back and started; provisioning mode off (never during a task sequence); the clock resynced if it's more than 5 minutes off; ccmsetup's download resumed; ccmeval and a machine policy request. Nothing is lost. |
| 2 · Clear and reset | Stuck MECM downloads cancelled; a corrupt local `Registry.pol` moved aside and rebuilt; a wrong trusted root key removed so the client fetches it again; the client's policy reset; `ccmrepair.exe`. |
| 3 · Reinstall the client | `ccmsetup /uninstall`; WMI salvaged if inconsistent; the client reinstalled with the device's site line. **`SMSCFG.INI` and the certificates are kept**, so the client keeps its identity and MECM record. Steep waits up to 15 minutes for it to register. |
| 4 · Clean everything and reinstall | Only if the reinstalled client still hasn't registered. Removes everything the client left, including **`SMSCFG.INI`** (a new identity, so MECM makes a new record). Registry keys are exported first; `ccmcache` is kept. An invalid client certificate is replaced safely. WMI reset if still broken (then a restart). Then the reinstall. |

- Reinstalling needs the device's site line, as for Secure Client Push. A device with no assigned site is refused.
- Fixed when the client proves it works (the five checks above).
- Other Client health problems (a disabled AD account, unapproved, blocked, obsolete, duplicate) are warned about, not blocking. Automatic repairs wait until they're resolved.

### WMI repair

For devices where WMI doesn't answer, hangs or reports a damaged repository. The MECM client keeps its data in WMI, so a broken WMI means a broken client.

| Level | Steps |
|---|---|
| 0 · Diagnose | The WMI service; `winmgmt /verifyrepository`; the repository's size (over 1 GB usually means something keeps filling it); test queries on `root\cimv2` and `root\ccm`, 60 seconds each; WMI crashes in the last 14 days |
| 1 · Restart WMI | Sets WMI back to Automatic if disabled, restarts it and the services that depend on it |
| 3 · Salvage the repository | Only if the repository is inconsistent or `root\cimv2` doesn't answer. Backs it up, runs `winmgmt /salvagerepository`, and recompiles only the autorecover MOFs. |
| 4 · Reset the repository | Only if WMI is still broken. Backs it up and runs `winmgmt /resetrepository`. **The MECM client's data is lost: reinstall the client afterwards.** |

- "Couldn't check" is never "broken": nothing is salvaged or reset because a check couldn't run.
- **Site systems and domain controllers are never changed.** On those, only level 0 runs. Repair them by hand.

### .NET Framework repair

For devices whose .NET Framework updates keep failing. Read the error on **Software update install failures** first: a missed restart only needs the install retried. **.NET Framework repair…** on that page opens the repair with the ticked devices.

| Level | Steps |
|---|---|
| 0 · Diagnose | The .NET Framework 4 version and whether 3.5 is installed; core files; a pending restart; .NET errors that stopped the MECM client in the last 14 days |
| 1 · Run the .NET Framework Repair Tool | Runs Microsoft's [.NET Framework Repair Tool](https://support.microsoft.com/help/2698555) silently, even when level 0 finds nothing. It may need a restart. Its log goes to Steep's log folder. |

Fixed when .NET Framework 4.6.2 or later is registered, its core files are there, and no .NET error stopped the client in 14 days. Retry the failed update afterwards.

**The tool comes from Microsoft.** **Download from Microsoft** on the repair page fetches it, and Steep keeps it only if Microsoft signed it. Without internet access, copy it into `%ProgramData%\TeaPowered\Steep\RepairTools`. Each download is audited.

### Windows Update Agent repair

For devices whose software update scans fail, or report no scan or an old one. **Windows Update Agent repair…** on the *Software update scan* page opens it with the ticked devices.

**Level 0 · Diagnose** checks:

- **Services:** Windows Update, BITS, Cryptographic Services, Windows Modules Installer (and Update Orchestrator and Windows Update Medic, reported only).
- **Where Windows Update points:** the MECM client's software update point against Windows Update's policy, and **which Group Policy object sets it**. A domain GPO pointing elsewhere is named.
- **Whether the software update point's WSUS answers**, with the same first call a scan makes. HTTP 503 means WsusPool is stopped, 500 that WSUS failed (often SUSDB), 404 that there's no WSUS there.
- **The last scan** and its error.
- **Catalog versions:** the client's against its software update point's, and that point's against the top one's.
- **The store:** `SoftwareDistribution`'s size, `DataStore.edb`, free space, a pending restart, the component store.

| Level | Steps |
|---|---|
| 1 · Services and a scan | Disabled services set back to their defaults, then a software update scan through the MECM client (up to 20 minutes). The scan always runs: a successful scan is the proof. |
| 2 · Clear caches and scan again | A corrupt `Registry.pol` replaced; Windows Update's stuck downloads cancelled; its download cache cleared; another scan |
| 3 · Reset Windows Update | `SoftwareDistribution` and `catroot2` **moved** aside; the WSUS client ID cleared (exported first); a full scan. Windows' update history in Settings starts empty; MECM keeps its own. |

- **When the cause isn't on the device, levels 2 and 3 don't run.** An unreachable or failing software update point, a catalog behind the top one, a domain GPO pointing elsewhere, or a broken MECM client is reported with what to fix.
- Fixed when no service is disabled, `Registry.pol` is valid, Windows Update points at the client's software update point, the last scan succeeded, and the catalog is current.
- It never restarts the device.

## Secure Client Push

Installs the MECM client with ccmsetup.exe, like MECM's client push. On a device that has a client, ccmsetup repairs or upgrades it.

**Where to start it:** the **Secure Client Push** tool page (type device names), or **Fix the ticked devices** on the *Computers without a MECM client* (install) and *Client version* (upgrade) pages.

### One line per primary site

Each primary site has its own folder and ccmsetup properties (Configure → Client tools). **Discover from MECM** fills them in from the site's production client package and Client Push Installation Properties, adding `/mp:` when none is named.

Which site a device uses:

| The device | Site used |
|---|---|
| Has an assigned site in MECM | That site (works in a CAS hierarchy too). |
| Has no assigned site | **You choose** in the confirmation. Steep doesn't guess. |
| Has records naming different sites | **Refused.** Fix discovery or boundaries first, or the client would keep switching sites. |

Fix pages have a **Site** drop-down when their devices span sites.

### Running it

- The confirmation names the devices and shows the ccmsetup command line for each site.
- You can change the command line for one run (for example to add `/forceinstall`). **Back to the site's line** undoes it. The change is audited and reaches the device only as ccmsetup's arguments.
- Steep waits for ccmsetup to finish and reports its return code: 0 is success, 7 is success with a restart needed.

### Automatic Secure Client Push

Installs the client on computers without one, in the OUs you choose, like MECM's automatic site-wide client push but with Steep's sign-in. An Architect sets it up on **Configure → Client tools → Automatic Secure Client Push**. It's off until then, and **Report only** (list, don't push) is on until the Architect turns it off.

| Setting | Behaviour |
|---|---|
| Which computers | Enabled AD computers with no MECM client, in the chosen OUs, used in the last 30 days, of a kind you ticked: **Workstations**, **Servers**, **Site system servers**, **Domain controllers**. Domain controllers and site systems each need their own tick. |
| Which site | The computer's assigned site. Computers with none are skipped and listed under *Needs an engineer*. |
| When | A pass every 30 minutes. Each pass checks which computers answer on TCP 445 (up to 500, longest-waiting first) and pushes to at most 20. |
| After a push | No new push to that computer for 24 hours. A client showing up in the next collections marks it **Fixed**. |
| Offline computers | Tried every pass for 7 days, then left for 30 days. |
| Needs an engineer | After 2 failed pushes in a row, 3 pushes in 90 days, or a client that goes missing again within 30 days. **Allow automation again**, **Exclude** and **Include again** are on the fix page. |

All numbers are settings.

- The **Computers without a MECM client** fix page shows the automation's state, the last and next pass, **Run a pass now**, the queue, *Needs an engineer*, and computers the settings leave out.
- **Pause all automatic remediation** on the Client tools page stops every automation at once.
- Every automatic push is a normal Secure Client Push run, shown as **Steep (automatic)** in history and the audit log.

## How the tools work (technical details)

This section is for engineers and security teams who want to know exactly what Steep does on a device and what it reads from MECM and AD. What each tool shows and how to use it is in the sections above; the permissions are in [permissions.md](permissions.md#client-tools-secure-client-access).

### Signing in and running on a device

Every tool that works on a device goes through the same steps, the way MECM's client push does:

1. **Reachable?** Steep checks, from the Steep server and without credentials, that the device answers on file sharing (TCP 445). If it doesn't, the run stops: no password is read and no account is used.
2. **Sign in.** Steep signs in to the device once for the whole run, with the device's LAPS password or a stored account, as a network sign-in (the mechanism `runas /netonly` uses). Every later step reuses that sign-in. A local account (the LAPS account) signs in with NTLM; a domain account with Kerberos where it can.
3. **Arm the LAPS reset** (LAPS sign-ins only, with "Reset the LAPS password after every use" on): see [Resetting the LAPS password](#resetting-the-laps-password) below.
4. **Copy.** Through the **ADMIN$** share, Steep copies its client agent (`SteepClientAgent.exe`, about 10 KB, built for .NET Framework 4.7.2, which every supported Windows has), the tool's PowerShell script, and any files the tool needs (ccmsetup.exe, the repair modules, Microsoft's .NET Framework Repair Tool) to `%windir%\Steep\Jobs\<id>`. Only administrators can write under `%windir%`.
5. **Run.** Through the Service Control Manager (over SMB named pipes), Steep creates and starts the service `SteepClientAgent_<id>` ("Steep client agent (temporary)"), set to start on demand and to run as **LocalSystem**. The agent runs the script with Windows PowerShell 5.1, as SYSTEM, and stops. Anything an engineer typed (extra log paths, ccmsetup properties) reaches the script only as quoted text, never as code.
6. **Bring back and clean up.** Steep waits for the script (each tool has its own time limit), copies back the result files the script wrote in its folder, then **deletes the service and the folder**.

Each script also writes a line or two to `%windir%\Steep\Logs\ClientTools.log` on the device, which is kept (and collected by Gather logs). No WMI or other RPC ports are needed for any of this; WMI is used only for Read OS details' quick path.

**Verify access** (Configure → Client tools) signs in one way to one device and checks each part on its own: it writes and deletes a test file through ADMIN$, opens the Service Control Manager with the right to create services (creating nothing), and reads `Win32_OperatingSystem` over WMI (optional).

### Reading the LAPS password

- Steep reads the computer object's permissions (`nTSecurityDescriptor`, permissions only) to pick the first ticked account allowed to read the password and the first allowed to reset it. The ticked accounts' group memberships are read to resolve nested groups.
- It reads `userAccountControl` (to refuse domain controllers), `operatingSystem`, `operatingSystemVersion` and `dNSHostName` (the only name Steep connects to).
- It reads the password from `msLAPS-Password`, or from `msLAPS-EncryptedPassword`, decrypted on the Steep server while signed in as the reading account (which must be an authorized decryptor). The account name comes from the same attribute.
- How the password is protected in memory and cleared afterwards is in [password-security.md](password-security.md).

### Resetting the LAPS password

Right after signing in with a LAPS password, before the tool does anything else:

1. A small job registers a scheduled task in the Task Scheduler folder `\Steep\`, as SYSTEM, that runs `finish.ps1` every minute and at every start of Windows, for up to three days. Each run of the task exits straight away until one of two things happens.
2. When the run ends normally, Steep writes `done.txt` next to `finish.ps1`, flags the password as expired in AD with the reset account (`msLAPS-PasswordExpirationTime`, the only AD attribute Steep writes), and signs out.
3. If Steep never writes `done.txt` (the device restarted, Steep stopped, the run hung), the task acts anyway once the time in `deadline.txt` passes: the tool's own time limit plus a margin.
4. The task then stops and removes the services the run created and their programs and scripts (keeping any results a cut-off run left, for Steep to collect later), resets the LAPS password with `Reset-LapsPassword` (Windows LAPS stores the new one in AD at once; `Invoke-LapsPolicyProcessing` where that command isn't there), and removes its own files and the task.

Nothing in the task or its files is secret. If the task can't be created, Steep cleans up and stops the run before the tool starts.

### Read OS details (technical)

- **Quick path:** over WMI (DCOM) from the Steep server, it reads `Win32_OperatingSystem`, `Win32_ComputerSystem.SystemType`, and the registry's `UBR` and `DisplayVersion` (`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion`). Nothing is copied or started on the device.
- **Otherwise**, through the client agent: the same registry values, plus `Win32_OperatingSystem` and `Win32_ComputerSystem` read on the device; if WMI on the device fails, the registry alone (with `ProductOptions` for workstation or server).
- The result goes straight into Steep's record of the device, so the dashboards use the revision at once. It changes nothing on the device.

### Gather logs (technical)

- Copies the enabled built-in rules' files and the paths you added, recursively, into a staging folder in the job's folder, stopping at the size limit (files left out are listed in `Steep-summary.txt`).
- Exports the event logs with `wevtutil epl`, limited to the last 14 days.
- Decodes `WindowsUpdate.log` with `Get-WindowsUpdateLog` (given up after three minutes; the .etl files are in the zip either way).
- Zips it all on the device (`Compress-Archive`) and brings back the zip, which is kept in Steep's data folder. The staging folder is removed with the job folder.

### Trace hardware inventory (technical)

- **On the device** (client agent): the BITS service and its startup type, and `InventoryAgent.log` and `CcmMessaging.log` (rolled-over copies first, read without locking them).
- **On the management point** (signed in there the same way): `MP_Hinv.log`, for this client's report and the MIF file it became.
- **On the site server** (signed in there the same way): `Max MIF Size` under `HKLM\SOFTWARE\Microsoft\SMS\Components\SMS_INVENTORY_DATA_LOADER`, `dataldr.log`, and the names and sizes of files in `inboxes\auth\dataldr.box\BADMIFS`.
- **From MECM**, with the hierarchy's read account: `SMS_StatusMessage` and `SMS_StatMsgInsStrings` (SMS_INVENTORY_DATA_LOADER messages 2700, 2703 and 2719; every collection reads these too) and `SMS_Site` (the site server's name).
- It only reads.

### Approve, Unapprove, Block and Unblock (technical)

- Nothing runs on the device, and Secure Client Access isn't used.
- With the hierarchy's **write account**, through the SMS Provider: `SMS_Collection.ApproveClients` (Approve and Unapprove) and `SMS_Collection.BlockClients` (Block and Unblock), the same calls the console makes.
- Steep then reads the device's record back from MECM and updates its own copy.

### Client actions (technical)

- The client agent calls `SMS_Client.TriggerSchedule` in `root\ccm` on the device, as SYSTEM, with each action's standard schedule ID (machine policy retrieval and evaluation, hardware inventory, software updates scan, software updates assignments evaluation): the same IDs the Configuration Manager control panel uses.
- **Full resync** first deletes the client's record of its last hardware inventory report (`InventoryActionStatus` for the hardware inventory action, in `root\ccm\invagt`), so the next report is a full one.
- **Run the client health check** runs `%windir%\CCM\CcmEval.exe` and reads `CcmEvalReport.xml`.
- The run fails with a clear message if there's no client, or if SMS Agent Host (CcmExec) isn't running.

### Read installed MSI (technical)

- **On the device** (client agent, read only): finds the product in Windows Installer's records (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18`, per-machine installs only) and its cached MSI in `C:\Windows\Installer`; finds where each program (.exe) the MSI installs was put, from Windows Installer's component records; and brings back a copy of the cached MSI and of those programs.
- **On the Steep server:** reads the copies (nothing from the device is run), keeps the MSI in its data folder while a choice uses it, and deletes the program copies.
- Steep never uses WMI's `Win32_Product` class, which makes Windows Installer check (and sometimes repair) every installed product.

### Software update scan report (technical)

- **On the device** (client agent, as SYSTEM, read only), from the MECM client's own records in WMI: its software update source and last scan (`CCM_UpdateSource`, `CCM_ScanUpdateSourceHistory`), every update the scan found (`CCM_UpdateStatus`), the updates deployed to the device (`CCM_TargetedUpdateEx1`), its software update deployments and their compliance (`CCM_UpdateCIAssignment`, `CCM_AssignmentCompliance`), whether a restart is pending, and the last 30 entries of Windows Update's install history.
- **From MECM**, with the hierarchy's read account: each software update point's catalog version (`vSMS_SUPSyncStatus`, or `SMS_SUPSyncStatus`), and for the latest cumulative updates, the site's catalog and this device's state for them (`v_UpdateInfo` and `v_UpdateComplianceStatus`, or `SMS_SoftwareUpdate` and `SMS_UpdateComplianceStatus`).

### Software update patching report (technical)

- **On the device** (client agent, as SYSTEM, read only), from the MECM client's records in WMI: its software update deployments and the updates in each (`CCM_UpdateCIAssignment`, `CCM_AssignmentCompliance`), the update store (`CCM_UpdateStatus`), the client's current state of deployed updates that aren't compliant yet (`CCM_SoftwareUpdate`), its state messages for deployments and updates (`CCM_StateMsg`), the client cache (`CacheInfoEx`), and whether a restart is pending.
- **Logs:** it copies the client logs the report follows (with their rolled-over copies), `CBS.log` and the CbsPersist files, and Windows Update's ETL files from the deployments' window (at most 30 days back), and decodes `WindowsUpdate.log` from them on the device (up to five minutes), up to 150 MB in all. The logs are read without locking them. Steep reads them on the Steep server and keeps them as the report's zip.
- **From MECM**, with the hierarchy's read account: the software update deployments (single updates and update groups) that target the device and MECM's names for the deployment and update states, from the site database (`v_CIAssignment`, `v_CIAssignmentTargetedMachines`, `v_StateNames`). Without the site database, it reads the deployments from the SMS Provider (`SMS_CIAssignmentBaseClass` with `SMS_FullCollectionMembership`), which has no state names.

### Secure Client Repair (technical)

- **What travels with a repair:** Steep's PowerShell repair modules, and what the repair needs (ccmsetup.exe for the MECM client repair, Microsoft's .NET Framework Repair Tool for the .NET Framework repair). Before importing anything, the run checks each module's SHA-256 against the copy Steep shipped; ccmsetup.exe and the .NET tool are hash-checked the same way. The .NET tool's Microsoft signature is also checked on the Steep server each time it's sent.
- **What stays on the device**, under `%windir%\Steep`: the run record (`Repair\ledger.json`), a lock while a repair runs (`Repair\running.lock`), the repair log (`Logs\SteepRepair.log`, CMTrace format) and anything a step moved aside (`Backups\`). See [Secure Client Repair](#secure-client-repair) above.
- **From MECM**, with the hierarchy's read account: the production client version (MECM client repair), each software update point's catalog version (`vSMS_SUPSyncStatus`, or `SMS_SUPSyncStatus`, for the Windows Update Agent repair) and the device's site line (for a reinstall).
- **Read as the Steep service account** (the Steep server's computer account on the network): ccmsetup.exe from the site's client folder, and the site's trusted root key from `\\<site server>\SMS_<site>\bin\i386\MobileClient.tcf`.
- **Restarts** are scheduled by the device itself with `shutdown.exe /r /t <seconds> /d p:4:1`. Afterwards Steep checks the device answers on TCP 445 and runs the same repair again, which carries on after the last step it finished. Nothing is left on the device to run at start-up.

### Secure Client Push (technical)

- **Discover from MECM**, with the hierarchy's read account: `SMS_Site` (which sites are primaries), `SMS_SCI_Component` (`SMS_CLIENT_CONFIG_MANAGER`, the Client Push Installation Properties), `SMS_SCI_SysResUse` (management points) and `SMS_Package` (the production client package's source folder).
- **The site a device uses** comes from its MECM record (`SMSAssignedSites`, from `v_RA_System_SMSAssignedSites` or the SMS Provider).
- **ccmsetup.exe** is read from the site's client folder **as the Steep service account** and copied to the device with the job.
- **On the device**, the client agent runs `ccmsetup.exe` with the site's properties, waits for ccmsetup (which hands over to its own service) to finish, and reads its return code from `%windir%\ccmsetup\Logs\ccmsetup.log`, by the entries' own times. 0 is success, 7 is success with a restart needed. It also reports whether SMS Agent Host is running.

### Automatic Secure Client Push (technical)

- **Which computers** comes from Steep's own collected data: the AD computers collector (`operatingSystem`, `userAccountControl`, `pwdLastSet`, `lastLogonTimestamp`) and MECM's list of site systems. It makes no extra AD or MECM reads.
- **Each pass** checks from the Steep server, without credentials, which waiting computers answer on TCP 445, then runs a normal Secure Client Push for those that answered, through the same queue as manual runs.
