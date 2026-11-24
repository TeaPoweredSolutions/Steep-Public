# Why one password shouldn't open every device: Secure Client Access and Windows LAPS

MECM's client push needs an account that is a local administrator on every computer it installs the client on. In most environments, that's one or two "client push" accounts in the local Administrators group of every workstation and server. They are some of the most powerful credentials an organization has. They rarely change, and many people and systems know them.

This page explains the risk, what the 2025 cyberattack on Jaguar Land Rover shows about stolen credentials, and how Steep's Secure Client Access uses each device's own Windows LAPS password instead.

## The problem with a global local admin account

A client push account is over-privileged by design: it must be an administrator anywhere MECM might install a client. That makes it a perfect target.

| Risk | Why |
|---|---|
| **One password, every device** | Whoever has the password, or its hash, is an administrator on every computer. One leak is the whole estate. |
| **It rarely changes** | Changing it means updating MECM, scripts and tools, so it's often left for years. A password leaked years ago may still work. |
| **Many people know it** | MECM administrators, engineers, contractors and former staff may have seen it, scripted it or saved it. |
| **It's sent over the network all the time** | Every push signs in to a remote computer with it. A push aimed at a computer an attacker controls can capture it. Researchers have shown how to make MECM push to a machine of the attacker's choosing. |
| **Its use looks normal** | An administrator signing in to many computers is exactly what this account does every day. |

Such passwords escape in ordinary ways: a script or share, a phished engineer, an infostealer on a contractor's laptop, a cracked hash, or an insider. Once it's out, the attacker doesn't need a software vulnerability. They just sign in.

## The Jaguar Land Rover cyberattack (2025)

On 31 August 2025, attackers got into Jaguar Land Rover's (JLR) IT environment. On 1 September, JLR shut down its global IT systems and stopped production in the UK, Slovakia, Brazil and India. A group calling itself "Scattered Lapsus$ Hunters" claimed responsibility. Investigators later linked the attack to Russian hackers.

### How the attackers got in

JLR hasn't published a full technical account. What has been reported points at credentials, not an exotic exploit:

- In March 2025, the Hellcat ransomware group claimed to have stolen JLR data, reportedly using **credentials harvested by infostealer malware** from a third party with access to JLR's Jira.
- Analysis of the September attack describes "social engineering, credential abuse, weak segmentation, and inadequate detection", not a zero-day. Attackers reportedly used **valid usernames and passwords, some with administrator rights**. **Passwords stolen in 2021 still worked in 2025.** Broad permissions let them move through to core infrastructure.
- The groups behind the name are known for **stolen, bought or phished credentials**: impersonating staff to help desks, phishing, MFA fatigue, SIM swapping, and recruiting insiders.

!!! note "Lapsus$ is not LAPS"
    Despite the similar name, the Lapsus$ group has nothing to do with Windows LAPS. We know of no report that anyone has broken Windows LAPS. A password that differs on every device and changes after every use is worth very little to an attacker who steals it.

Two of those points describe a global client push account exactly: administrator rights everywhere, and a password that rarely changes.

### The cost

The shutdown lasted about five weeks. JLR began a phased restart in October 2025, and output returned to normal in mid-November.

| Who | Impact |
|---|---|
| **Jaguar Land Rover** | About **£50 million a week** in losses while production was stopped. **£196 million** of attack-related costs and an **underlying loss of £485 million** in the quarter to September 2025, with revenue down about **24% to £4.9 billion** (tariffs and older Jaguar models also played a part). Wholesale volumes down **43.3%** in the three months to December 2025. By September 2026, annual pre-tax profit had fallen to **£14 million from £2.5 billion**, with the attack one of several causes, and thousands of staff were offered redundancy. |
| **JLR's suppliers** | More than **2,700 UK organisations** materially affected, according to the Cyber Monitoring Centre. Supply-chain workers laid off in their hundreds; smaller suppliers at risk of going out of business. |
| **The British government and public** | A government-guaranteed loan of up to **£1.5 billion** so JLR could keep paying suppliers. The Cyber Monitoring Centre estimated the cost to the UK economy at about **£1.9 billion**, the most economically damaging cyber event in British history. The Bank of England named it as one reason UK growth slowed. |

## How Secure Client Access reduces the risk

Steep's Secure Client Push and client tools can sign in to devices without a global admin account. **Secure Client Access** uses each device's own **Windows LAPS** password:

| Protection | How |
|---|---|
| **A different password on every device** | Windows LAPS gives each computer's local administrator its own random password, stored in AD. A leak opens one device, not the estate. |
| **Read only when needed** | Steep reads a password only when a tool is about to use it, with an account AD allows, and only after the device answers on the network. Steep never shows, logs or stores it. |
| **Reset after every use** | Before the tool starts, Steep sets up a task on the device. Within about two minutes of the run ending, it resets the LAPS password. If the run is cut off, it resets it anyway after the run's time limit. Steep also flags the password as expired in AD. Reset after use is on by default. |
| **No long-lived password to steal** | No client push password in scripts, shares or notes, and no password from 2021 still working in 2025. |
| **Controlled in AD** | Read and reset rights are granted per OU with the standard Windows LAPS permissions. Encrypted passwords can only be decrypted by your LAPS policy's authorized decryptors. |
| **Domain controllers excluded, everything audited** | Steep never uses a domain controller's LAPS password. Every use is audited with the device, the engineer and the reason. |

This covers workstations and servers alike. How Steep protects the password while it uses it: [password-security.md](password-security.md).

Once your devices are covered, you can remove the global client push accounts from MECM and from your devices' local Administrators groups. That closes one of the easiest routes to administrator rights on every machine. If your policy doesn't allow LAPS passwords to be used this way, Steep can still use stored administrator accounts, and can try LAPS first and fall back to them only where needed.

No single control stops every attack. Phishing-resistant MFA, least privilege, network segmentation and tested backups all matter. But a password that is different on every device and changes after every use is one an attacker can't reuse across your estate.

Setup, AD permissions and the Windows settings that can block it: [client-tools.md](client-tools.md) and [permissions.md](permissions.md#client-tools-secure-client-access).

## Sources

- [Jaguar Land Rover cyberattack](https://en.wikipedia.org/wiki/Jaguar_Land_Rover_cyberattack), Wikipedia (timeline, attribution, JLR quarterly results, supplier layoffs, Bank of England).
- [Jaguar Land Rover cyber attack](https://shattered.io/jaguar-land-rover-cyber-attack/), Shattered (Cyber Monitoring Centre estimate, 2,700 organisations, £1.5 billion loan guarantee, attackers' methods).
- [JLR breach breakdown](https://treblle.com/blog/jlr-breach-breakdown-analysis), Treblle (March 2025 infostealer credentials, credential abuse with administrator rights, passwords from 2021 still working).
- [Thousands at Jaguar Land Rover offered redundancies after falling profits](https://www.theguardian.com/business/2026/sep/05/thousands-jaguar-land-rover-offered-redundancies-after-falling-profits), The Guardian, 5 September 2026 (profit fall, redundancies, savings target).
