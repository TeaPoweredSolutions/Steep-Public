# Licensing

## Trial mode

Steep installs and runs without a license, in **trial mode**.

| Works as usual | Hidden or off |
|---|---|
| Real counts and statuses on every dashboard ("3 devices behind, 4 missing exploited fixes") | Operating systems, versions, builds, revisions, KBs, CVEs and device names |
| Collections, trends, permissions and the activity log | On the Applications page: versions, product codes, UpgradeCodes, and products that name an OS or KB |
| **Read OS details**, to check Secure Client Access works | Every other client tool. Gathered logs can't be downloaded. |
| | Exports, and application decisions (read-only) |

!!! info
    Hidden values are never sent to the browser (a blurred placeholder is), so they can't be revealed.

## The license

A license is a `.lic` file, or the same text pasted. A Steep Architect imports it on **Configure → License**, or with `Steep.exe license import <file>`.

| A license names | What it means |
|---|---|
| **One or more sites** | A MECM site code and an AD domain, for example `CHQ@corp.contoso.com`. Steep is licensed while **one** of them is connected: the site code as a MECM hierarchy **and** the domain as an AD domain, on Connections. Everything else you connect is covered too. (The site code alone isn't enough: P01 is MECM's suggested code, so many companies share it.) |
| **A number of MECM clients** | Counted across every hierarchy and domain. Going over never turns anything off. Well over, Steep shows a reminder, which an Architect can dismiss until 30 days before expiry. The difference is settled at renewal. |
| **An end date** | A warning starts 30 days before. After the end date, Steep keeps working for 30 more days with a warning, then returns to trial mode. Your data is kept. |

!!! warning "Migrating to a new site code or domain"
    If you change the site code **or** the AD domain, the license no longer matches once the old one is removed from Connections, and Steep switches to trial mode (your data is kept). Before you remove the old one, ask us for a new license: Configure → License shows the details to send.

!!! info
    Configure → License shows the status, what the license covers, the current and peak client counts, and the details to send when you ask for a license. Imports, and changes between licensed and trial, are audited.

## License terms

Configure → License shows Steep's license terms, one card per section, and who has accepted them. The same text is in [license-terms.md](license-terms.md), and the third-party list in [third-party-notices.md](third-party-notices.md).

- Each person accepts once, the first time they use Steep.
- A small change to the terms shows a banner (never a block) to people who accepted earlier terms. A Steep Architect, or `Install-Steep.ps1 -AcceptLicenseTerms`, can accept it for everyone.
- A major change (a new edition of the terms) asks everyone to accept again.

## Public preview

The public preview has a **preview license** built in:

- It covers any site and any number of MECM clients.
- It's used only while no license is imported. An imported license always wins.
- It expires on **2026-12-23**, with warnings from 2026-12-16.
- Steep then keeps working with a warning until 2027-01-22, then returns to trial mode. Your data is kept.
