# Scalability

How much disk, memory and time Steep needs as environments grow.

Sizes are measured on lab databases and on 100,000 generated rows per table, then projected per device. Times and memory at scale are **estimates** until the scale test (150,000 devices × 150 installed products).

## Disk

Assuming 150 installed products per device, 20 tracked applications per device, and one flag a month per device kept 730 days. "With metering" adds 20 metered files, half used on each device, every month.

| Devices | Database | With metering |
|---|---|---|
| 10,000 | **~0.1 GB** | **~0.12 GB** |
| 50,000 | **~0.43 GB** | **~0.53 GB** |
| 150,000 | **~1.2 GB** | **~1.5 GB** |

!!! info
    At 100 metered files, add about 1.2 GB at 150,000 devices.

Allow about **twice the database size** free:

- SQLite's change file (`steep.db-wal`) grows briefly during a large first collection.
- Upgrading to 0.7.0 rebuilds the database once.
- Plus `steep.log` (5 MB by default, Settings → Retention).

!!! info
    Space freed by retention clean-up goes back to Windows after the daily clean-up, when it's more than 20% of the file. See [Database maintenance](installation.md#database-maintenance).

### What takes the space

| Data | Size | Grows with |
|---|---|---|
| MECM device record | ~1.4 KB | Devices |
| AD computer record | ~710 bytes | AD computers |
| Installed software (product numbers per device, packed) | ~2.7 bytes per entry: ~410 bytes per device | Devices × products |
| Each distinct product version, stored once | ~290 bytes | Distinct product versions |
| Steep device IDs (for trends and telemetry) | ~1.1 KB per device | Devices |
| Per-device application versions | ~115 bytes per tracked app | Devices × tracked apps |
| Flag history | ~85 bytes per flag | Devices × problems × days kept |
| Software metering usage | ~200 bytes per metered file per device | Devices × metered files used |
| Action log | ~350 bytes per record | Fixes Steep does or surfaces × days kept |
| Microsoft release, security and vulnerability data | 1.1 MB in total (compressed) | Nothing: the same everywhere |
| Update catalog, applications, site settings | ~5–25 MB in total | Catalog size, number of applications |
| Trends, settings, decisions, run history, audit | A few MB | Days kept, decisions made |

How Steep keeps it small:

- **Each product version is stored once** (name, publisher, version, product code, uninstall string). Each device keeps only a packed list of its products' numbers. Installed software used to be 97% of the data.
- **Dates, device references and repeated names are numbers**, lists are packed, reference data is compressed, and the database uses 8 KB pages.

Per table: [data-schema.md](data-schema.md#sizes).

## Memory and time

| Devices | Memory (peak) | Dashboards after a collection | Applications page |
|---|---|---|---|
| 10,000 | **~250 MB** | ~1–3 s | ~1 s |
| 50,000 | **~700 MB** | ~5–10 s | ~3 s |
| 150,000 | **~1.5–2 GB** | ~15–30 s | ~5–10 s |

- **Dashboards are worked out once after each collection**, then served from memory, so clicking around is instant.
- Steep prepares them in the background as soon as a collection finishes, and at start-up. Nobody waits unless they open a page during that work.
- Settings → Collection → **Low priority** runs that work below normal priority. Recommended for large environments.
- It uses one CPU core. Collections mostly wait on MECM, SQL and AD.
- Device dashboards don't load installed software. Only the Applications page and its export do.
- Devices on the same OS build and revision share one list of missing CVEs.

## Collections

| Collector | 10,000 devices | 150,000 devices | Notes |
|---|---|---|---|
| AD computers, changes only (hourly) | Seconds | Seconds | Only changed computers |
| AD computers, full read (daily) | ~2–5 s | ~20–60 s | LDAP paging |
| MECM installed software, first read | ~10–30 s | ~2–5 min | ~22.5 million rows at 150,000 devices |
| MECM installed software, daily | Seconds | ~10–40 s | Only devices whose inventory changed |
| MECM devices & client health (every 4 hours) | ~30–60 s through WMI | ~3–12 min through WMI; well under a minute from the site database | Choose "Site database (SQL)" on Connections |
| Software updates, applications | Seconds to a minute | Same | Depends on the catalog and applications, not devices |

## Large hierarchies

- **Read devices from the site database:** on Connections, choose "Site database (SQL)" for the hierarchy. The SMS Provider still decides which devices are included, so MECM security applies. Fields come from SQL, 1,000 devices per query. If SQL can't be reached, Steep falls back to the SMS Provider for that run.
- **Audit log:** kept permanently. Very large environments may need it aged later.
