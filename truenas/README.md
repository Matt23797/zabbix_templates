# Zabbix Template — TrueNAS SCALE (HTTP Agent)

Monitor TrueNAS SCALE via the REST API using Zabbix HTTP agent items. No SNMP configuration required — just an API key.

**Tested on:** TrueNAS SCALE 25.10.2.1 (Goldeye) · Zabbix 8.0

---

## What's Monitored

### System
| Item | Description |
|------|-------------|
| Version | Firmware version string |
| Hostname | System hostname |
| Uptime | Uptime in seconds (graphable) |
| Physical memory | Total RAM in bytes |
| CPU cores | Core count |
| Load average (1m / 5m / 15m) | System load averages |
| ECC memory | Whether ECC RAM is present |
| Active alert count | Number of active TrueNAS alerts |

### Boot Pool
| Item | Description |
|------|-------------|
| Status | ONLINE / DEGRADED / FAULTED |
| Healthy | Boolean health flag |
| Size / Allocated / Free | Boot pool capacity |
| Scrub state | Last scrub state |
| Scrub errors | Scrub error count |
| Vdev read / write / checksum errors | Per-vdev error counters |

### Storage Pools (LLD)
Discovers all ZFS data pools automatically.

| Item | Description |
|------|-------------|
| Status | ONLINE / DEGRADED / FAULTED |
| Healthy | Boolean health flag |
| Size / Allocated / Free | Pool capacity in bytes |
| Fragmentation | Pool fragmentation percentage |
| Scrub state | Last scrub state |
| Scrub errors | Scrub error count |

### Disks (LLD)
Discovers all disks by serial number.

| Item | Description |
|------|-------------|
| Size | Disk size in bytes |
| Pool assignment | Which pool the disk belongs to |

> **Note:** S.M.A.R.T attributes and disk temperatures require physical disks. These items return no data on virtual machines.

### Datasets (LLD)
Discovers all ZFS datasets automatically.

| Item | Description |
|------|-------------|
| Used | Bytes used |
| Available | Bytes available |
| Used by snapshots | Bytes consumed by snapshots |
| Compression ratio | Current compression ratio |
| Encrypted | Whether the dataset is encrypted |

### Services (LLD)
Discovers all configured services (SMB, NFS, SSH, iSCSI, FTP, SNMP, UPS, NVMe-oF).

| Item | Description |
|------|-------------|
| State | RUNNING / STOPPED |
| Enabled | Whether the service is set to start at boot |

---

## Triggers

| Trigger | Severity | Condition |
|---------|----------|-----------|
| Unexpected reboot | Warning | Uptime resets below threshold |
| Elevated load average | Warning | 5m avg load > `{$TRUENAS.LOAD.WARN}` |
| High load average | High | 5m avg load > `{$TRUENAS.LOAD.HIGH}` |
| Active system alerts | Average | Alert count > 0 |
| Boot pool not ONLINE | High | Boot pool status != ONLINE |
| Boot pool unhealthy | High | Boot pool healthy = false |
| Boot pool scrub errors | Average | Scrub error count > 0 |
| Boot pool vdev errors | Average | Any vdev read/write/checksum errors > 0 |
| Pool not ONLINE | High | Pool status != ONLINE (per pool) |
| Pool unhealthy | High | Pool healthy = false (per pool) |
| Pool capacity warning | Warning | Allocated > `{$TRUENAS.POOL.CAPACITY.WARN}`% |
| Pool capacity critical | High | Allocated > `{$TRUENAS.POOL.CAPACITY.HIGH}`% |
| Pool scrub errors | Average | Scrub error count > 0 (per pool) |
| Service enabled but not running | High | Service is enabled but state != RUNNING |

---

## Setup

### 1. Generate an API Key in TrueNAS

Go to **Credentials → API Keys → Add**. Give it a descriptive name (e.g. `zabbix-monitoring`) and copy the key — it's only shown once.

> For least-privilege access, create a dedicated read-only user account and link the API key to that account rather than using the admin account.

### 2. Import the Template

In Zabbix: **Configuration → Templates → Import** → select `truenas_scale_template.yaml`.

### 3. Add a Host

Create a new host in Zabbix and assign the template. The interface type doesn't matter — HTTP agent items don't use the Zabbix agent interface. An Agent interface with the TrueNAS IP is fine.

### 4. Set the Required Macros

Set these macros on the host (or at a host group / global level):

| Macro | Example | Description |
|-------|---------|-------------|
| `{$TRUENAS.URL}` | `http://192.168.1.100` | Base URL, no trailing slash. Use `https://` if you have a valid cert. |
| `{$TRUENAS.API.KEY}` | `1-abc123...` | API key from step 1 |

### 5. Optional Threshold Macros

These have sensible defaults but can be overridden per-host:

| Macro | Default | Description |
|-------|---------|-------------|
| `{$TRUENAS.POOL.CAPACITY.WARN}` | `80` | Pool capacity warning threshold (%) |
| `{$TRUENAS.POOL.CAPACITY.HIGH}` | `90` | Pool capacity high threshold (%) |
| `{$TRUENAS.LOAD.WARN}` | `2` | Load average warning threshold |
| `{$TRUENAS.LOAD.HIGH}` | `4` | Load average high threshold |
| `{$TRUENAS.REBOOT.WINDOW}` | `300` | Uptime (seconds) below which a reboot is flagged |

---

## Compatibility

| TrueNAS Version | Status |
|-----------------|--------|
| 24.10 (Electric Eel) | ✅ Supported |
| 25.04 (Fangtooth) | ✅ Supported |
| 25.10 (Goldeye) | ✅ Supported |
| 26.x and later | ⚠️ Not supported — see below |

### TrueNAS 26 and the REST API

This template uses the TrueNAS REST API (`/api/v2.0`). TrueNAS deprecated the REST API in 25.04 and has announced full removal in TrueNAS 26, replacing it with a WebSocket-based JSON-RPC 2.0 API.

**What this means:**
- The template works fine on all current stable releases (24.10, 25.04, 25.10)
- TrueNAS 25.10.1+ will generate a daily alert when REST API endpoints are accessed — this is cosmetic and does not affect functionality
- A v2 of this template using Zabbix external check scripts over WebSocket is planned for TrueNAS 26 compatibility

### Zabbix Version

Tested on Zabbix 8.0. Should work on 6.4+ but not tested.

---

## Polling Intervals

| Master Item | Interval | Rationale |
|-------------|----------|-----------|
| System info | 60s | Load averages need frequent polling for useful graphs |
| Alert list | 60s | Alerts should surface quickly |
| Service list | 60s | Service state changes should surface quickly |
| Pool list | 120s | Pool health is relatively stable |
| Boot pool state | 120s | Same as above |
| Dataset list | 300s | Dataset usage changes slowly |
| Disk list | 300s | Disk inventory rarely changes |

All dependent items inherit their master item's interval and add zero additional API load.

---

## License

MIT — free to use, modify, and distribute. Attribution appreciated but not required.

---

## Feedback & Contributions

Issues, suggestions, and pull requests welcome. If this template saves you time, consider sponsoring via GitHub Sponsors or Ko-fi.
