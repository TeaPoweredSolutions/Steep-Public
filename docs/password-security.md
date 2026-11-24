# How Steep protects passwords

Steep handles two kinds of passwords:

- **Connection accounts** you save in Steep, for MECM, the site database, Active Directory, source shares and client tools.
- **Windows LAPS passwords** of devices, which Steep reads from AD when a client tool signs in to a device.

## Connection accounts

| Protection | How |
|---|---|
| Encrypted for this server only | Windows DPAPI, for the Steep service account (SYSTEM). Only SYSTEM on the Steep server can decrypt them. A copied database is useless elsewhere. |
| Bound to Steep | DPAPI's additional entropy, so other software running as SYSTEM can't decrypt them by accident. |
| Locked-down storage | Kept in `steep.db` in `C:\ProgramData\TeaPowered\Steep`, a folder limited to SYSTEM and Administrators. |
| Write-only | Can be set or replaced, never shown, exported or logged. |
| Audited | Every change records who changed which account, and when. Never the password. |
| Used only for their own connection | Decrypted only when a collection, tool or action needs that connection. |
| Encrypted in memory | Site database, source share and client tools passwords are decrypted straight into memory Windows keeps encrypted for Steep alone (`CryptProtectMemory`). Steep hands them to Windows only for the sign-in, then overwrites them with zeros. |

!!! info
    The NVD API key, if you add one, is stored the same way. After moving Steep to a new server, enter the passwords again.

Each MECM hierarchy, site database, domain and source share can have its own account. See [permissions.md](permissions.md).

## LAPS passwords

### Never stored

Steep reads a device's LAPS password from AD only when a tool runs against that device, uses it for that run, and discards it. It's never written to the database, logs, exports or audit log.

### Read only by accounts you allow

- You choose which accounts may read LAPS passwords, and which may reset them. AD decides which devices each one covers. Steep checks before it asks.
- Encrypted LAPS passwords are decrypted on the Steep server, as the reading account. It must be an authorized password decryptor in your LAPS policy.
- Steep checks the device answers on the network first. If it doesn't, no password is read.
- Steep connects only to the device name AD holds for that computer.

### Protected in memory

- Steep reads the password right before it signs in.
- The raw data from AD, and the decrypted copy, are overwritten with zeros as soon as the password is read from them. They're pinned in memory, so no moved copies are left behind.
- From then on, the password stays in memory Windows keeps encrypted for Steep alone. No plain-text copy is made.
- Steep signs in once per run and reuses that sign-in for every step.
- When the run ends, Steep signs out and overwrites the password with zeros.

### Reset after use

With **Reset the LAPS password after every use** (on by default):

1. Right after signing in, before the tool does anything, Steep sets up a scheduled task on the device to reset the password and clean up.
2. When the run ends, Steep flags the password for reset in AD, tells the device the run is over, and signs out. Within about two minutes, the task resets the password (Windows LAPS stores the new one in AD at once), cleans up and removes itself.
3. If the run never ends normally (a restart, or Steep stopped), the task does the same shortly after the run's time limit, or when the device next starts.

!!! info
    A password Steep used stops working within a few minutes. If the device can't reach a domain controller then, it sets a new password at its next LAPS cycle, as flagged in step 2.

How the task works: [Resetting the LAPS password](client-tools.md#resetting-the-laps-password).

### Audited

Every use is recorded in the run's history and the audit log: which device, and which account read its password. The history also says whether the password was flagged and whether the device was told to reset it. Never the password.

## The Steep server

- **Crash dumps:** the installer limits Steep.exe's crash dumps to small ones, without the contents of its memory. Uninstalling removes the setting.
- **LSA protection:** Configure → Client tools shows whether LSA protection (RunAsPPL) is on. The installer recommends it when it's off. Steep never changes it.

## Recommendations

- Use a **dedicated, least-privilege account** for each connection. Give LAPS read rights only to the accounts client tools need.
- Keep **Reset the LAPS password after use** on, and give the reset account rights on every OU Steep reads passwords from.
- Keep **LAPS password encryption** on, with a small group of authorized decryptors.
- Limit who is a local **Administrator** on the Steep server, as on any site server.
- Turn on **Credential Guard** on the Steep server, as Microsoft recommends for servers that handle administrative credentials.
- Turn on **LSA protection** (RunAsPPL) on the Steep server before using client tools. It needs a restart and blocks sign-in add-ons Microsoft hasn't signed, so check first. Run the commands in an elevated PowerShell on the Steep server:
    1. Turn on audit mode: set the DWORD `AuditLevel` to `8` in `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\LSASS.exe`, and restart.

        ```powershell
        $key = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\LSASS.exe'
        if (-not (Test-Path $key)) { New-Item -Path $key | Out-Null }
        New-ItemProperty -Path $key -Name AuditLevel -PropertyType DWord -Value 8 -Force | Out-Null
        Restart-Computer
        ```

        !!! tip "Give audit mode time"
            Leave audit mode on for at least a week before you check, so everything that signs in through LSA has a chance to run: backups, antivirus and agent updates, patching and its restarts, and any monthly or scheduled jobs. An add-on that hasn't run yet won't show up in the log.

    2. Look for events **3065** and **3066** in Event Viewer, under Applications and Services Logs → Microsoft → Windows → CodeIntegrity → Operational. Each names an add-on that would be blocked. None means nothing would be affected.

        ```powershell
        Get-WinEvent -FilterHashtable @{ LogName = 'Microsoft-Windows-CodeIntegrity/Operational'; Id = 3065, 3066 } -ErrorAction SilentlyContinue |
            Format-List TimeCreated, Id, Message
        ```

    3. Turn it on with Group Policy (Computer Configuration → Administrative Templates → System → Local Security Authority → **Configure LSASS to run as a protected process**), or set the DWORD `RunAsPPL` in `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` to `1` (`2` on Windows Server 2025, which can be turned off again without a UEFI change). Then remove the audit setting and restart.

        ```powershell
        # 2 on Windows Server 2025 and later (can be turned off again without a UEFI change), 1 on older servers
        $os = Get-CimInstance -ClassName Win32_OperatingSystem
        $value = if ($os.ProductType -ne 1 -and [int]$os.BuildNumber -ge 26100) { 2 } else { 1 }
        New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -Name RunAsPPL -PropertyType DWord -Value $value -Force | Out-Null
        "RunAsPPL set to $value"
        Remove-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\LSASS.exe' -Name AuditLevel -ErrorAction SilentlyContinue
        Restart-Computer
        ```

    4. Check it worked: Windows logs event **12** (Wininit) in the System log, "LSASS.exe was started as a protected process". Configure → Client tools in Steep shows it too.

        ```powershell
        Get-WinEvent -FilterHashtable @{ LogName = 'System'; ProviderName = 'Microsoft-Windows-Wininit'; Id = 12 } -MaxEvents 1 |
            Format-List TimeCreated, Message
        ```
