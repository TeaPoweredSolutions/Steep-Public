<!-- A copy of src/TeaPowered.Steep.Service/Steep-License-Terms.md, the terms Steep shows and each user accepts.
     Edit that file and copy it here: a test checks they match. -->

# Steep license terms

Edition: 2

## License, terms and liability

Steep is © 2026 Jonathan Dabian, doing business as Tea Powered Solutions ("we", "us"). All rights reserved.

Steep is licensed, not sold. A Steep license lets your organization use Steep for the number of devices and the
period you purchased; the free public preview covers any number of devices. You may not copy, resell, sublicense or
redistribute Steep, or reverse engineer it except where the law allows it regardless of these terms.

**Security research.** Security research on Steep is allowed, provided you follow a legal, responsible and
reasonable vulnerability disclosure process: report what you find to us privately, and give us at least 90 days, or
until a fix is released, before any public disclosure. We don't offer bug or vulnerability bounties.

**Preview, license and expiry.** Steep can run as a free public preview or with a license, and works until the date
shown on Configure → License. When a preview or license ends, Steep keeps working for 30 more days, then switches to
trial mode. Steep keeps all your data. Changes Steep made to your MECM environment stay in place, but Steep no longer
manages them. Licensing Steep again restores all of its features and your data.

**What Steep changes.** Steep can create, change, publish or deploy collections, updates, applications, packages,
baselines, task sequences, software metering rules and scripts, distribute content, and change certain settings in
your hierarchy (such as the software update point). Steep shows you what it will change and asks you to confirm before
it changes MECM.

**Accounts.** Steep supports separate accounts for each system it reads from (MECM, Active Directory and the MECM
site database), and separate MECM accounts for reading and writing. Steep only ever reads the site database. If you
don't want Steep to change anything, don't give it a MECM account with write access, and use only the dashboards,
which need read access alone.

**Use at your own risk.** Steep is pre-release software and may contain errors. Before you use a feature that changes
MECM, make sure you understand what it will do. Test with a pilot collection or a lab first.

**Your responsibility.** You decide what Steep reads from and writes to your infrastructure, and we're not liable for
disruption caused by how you use Steep. It's your responsibility to understand what Steep will do before you use it to
make changes in your environment. Steep's information comes from other systems in your environment and from public
sources, and can be incomplete, late or wrong: check everything at its source before you act on it or tell Steep to
make changes.

**No warranty.** Steep is provided "as is", without warranties of any kind, express or implied, including
merchantability, fitness for a particular purpose and non-infringement.

**Limitation of liability.** To the fullest extent the law allows, we're not liable for any indirect, incidental,
special or consequential damages, or for lost data, profits or business, or service interruptions, arising from using
or being unable to use Steep, even if advised of the possibility. Our total liability is limited to the greater of the
license fees you paid for Steep in the 12 months before the claim, or US$100.

**Changes to these terms.** These terms may be updated with new versions of Steep. When they change, Steep asks you to
accept the new version; continuing to use Steep also means you accept it.

## Support

If you've purchased Steep and have trouble installing, configuring or using it as documented, contact us and we'll
make a reasonable effort to resolve it. Free public preview users are welcome to ask for help too; we help on a
best-effort basis, without any obligation.

**What we may need from you.** Troubleshooting may need information about Steep, its log files, or information from
the systems it connects to (Active Directory, MECM and SQL Server), and sometimes a screen-sharing session with your
engineers. You stay in control of any screen-sharing session. If we can't get the information or access we need, we
may not be able to resolve the issue.

**Sensitive information.** Send only what's needed, and remove anything you don't want to share before you send it.
You're responsible for what you share with us, or with anyone else who troubleshoots Steep for you. We use what you
send only to help you, and delete it when the issue is closed. Never post logs or screenshots publicly.

**Refunds.** Troubleshooting results aren't guaranteed. If you've purchased Steep and we can't get it working as
documented, we'll refund the unused part of your license period.

## Privacy

Your environment's data stays on your Steep server. Steep doesn't "phone home": it sends no device names, user names,
domain or site names, vulnerabilities found in your environment, or installed-software lists to us or anyone else.

To keep its reference data current, Steep downloads public data from Microsoft (learn.microsoft.com,
api.msrc.microsoft.com), NVD, CISA and FIRST (api.first.org, epss.empiricalsecurity.com). These downloads are the same
for every customer, with one exception: to look up exploit-probability (EPSS) scores, Steep sends FIRST the IDs of the
CVEs found on your installed software (for example "CVE-2026-12345"), with no device or software names. When there are
more than 2,000 CVEs to look up, Steep downloads FIRST's full daily file instead. If sending CVE IDs concerns you, set
EPSS scores to "Always download every score" on Configure → Connections → Vulnerability data: Steep then always
downloads the full daily file (a few MB) and sends nothing. The "Turn off" button for public feeds, in the same place,
stops all of these downloads except Microsoft's.

Once a day, Steep asks GitHub (api.github.com) for the version number of the latest Steep release, so it can tell your
Steep Architects when a newer version is available. The request is the same for every customer: it carries nothing about
your environment, not even the version of Steep you run, and Steep compares the versions on your server. As with any web
request, GitHub sees your server's public IP address. Steep only notifies: it never downloads or installs anything by
itself. To turn this off, clear "Check for new Steep versions" on Configure → Settings.

Review the terms of use of these public feeds (listed under Third-party licenses and acknowledgements) and make sure
you agree to them. If you don't, turn the public feeds off.

A future version of Steep will add optional telemetry: counts only, such as how many OS versions weren't being patched
and were then corrected, patch deployment health and vulnerable application counts. It will never include device,
user, domain or application names, specific vulnerabilities found, or anything else that identifies your
organization, and you'll be able to turn it off.

## Third-party licenses and acknowledgements

Steep uses and ships with publicly available software and data. That software and data, with links to their licenses
and terms of use, are listed on Steep's License page and in Steep's documentation on GitHub. Review those licenses
and terms, and make sure you agree to them, before you use Steep.

## Contact, support and security reports

To ask for information, purchase Steep, report a problem (functional or security-related) or request support, email
Steep_by_TeaPowered@outlook.com. We may also follow Steep's GitHub and other channels such as LinkedIn, Slack and
Reddit; Steep's documentation on GitHub has the details.

When you report a problem or ask for support, include as much detail as you can, such as logs and screenshots (see
Sensitive information, above). We may ask for a screen-sharing session to resolve your issue.

Report security vulnerabilities in Steep privately to Steep_by_TeaPowered@outlook.com, so we can discuss and fix them
before anything is posted publicly.
