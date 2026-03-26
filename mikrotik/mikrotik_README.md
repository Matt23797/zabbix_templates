# Zabbix Templates — MikroTik RouterOS (REST API)

A collection of Zabbix templates for monitoring MikroTik RouterOS devices via the REST API. No SNMP required — all monitoring is done over HTTP using a read-only API user.

**Requires:** RouterOS 7.1+ · Zabbix 8.0

---

## Templates

### `MikroTik by REST API` (LAN/general)
General-purpose template for LAN-facing or homelab MikroTik devices. Covers system vitals, interface monitoring, DHCP, and firewall connections.

**Monitored:**
- CPU load and temperature
- Memory — free, total, utilization %
- Storage — free and total
- Uptime, RouterOS version
- Active firewall connection count
- DHCP active lease count
- Network interface discovery — traffic in/out (bps), errors, running state

**Macros:**

| Macro | Default | Description |
|-------|---------|-------------|
| `{$API_SCHEME}` | `http` | `http` or `https` |
| `{$API_HOST}` | `192.168.88.1` | Router IP or hostname |
| `{$API_PORT}` | `80` | REST API port (80 for http, 443 for https) |
| `{$API_USER}` | `zabbix` | Read-only API username |
| `{$API_PASSWORD}` | _(secret)_ | API password |
| `{$CPU_HIGH}` | `90` | CPU warning threshold (%) |
| `{$MEM_HIGH}` | `90` | Memory warning threshold (%) |
| `{$TEMP_HIGH}` | `70` | CPU temperature threshold (°C) |
| `{$IF_ERRORS_WARN}` | `2` | Interface error rate warning (pps) |

---

### `Mikrotik RouterOS by HTTP` (WAN/ISP)
ISP and WAN-focused template for CCR-class or edge routers. Includes hardware health sensor discovery, per-interface traffic monitoring, BGP session monitoring, OSPF neighbor monitoring, and default route tracking.

**Monitored:**

*System*
- Version, board name, model, serial number, identity
- Uptime, CPU load %, memory utilization %, storage free/total
- Bad blocks
- Firmware upgrade available (triggers when current ≠ upgrade firmware)
- Hardware health sensors via LLD (temperature, voltage, fan RPM — discovers whatever the hardware exposes)

*Routing*
- Default route active + gateway
- BGP session discovery — established state, prefix count, uptime
- OSPF neighbor discovery — state, state change count

*Interfaces*
- Per-interface discovery (excludes disabled interfaces automatically)
- Traffic in/out (bps), RX/TX errors, RX/TX drops, TX queue drops, link-downs
- Triggers for link down and link flapping

**Macros:**

| Macro | Default | Description |
|-------|---------|-------------|
| `{$ROS.URL}` | `http://192.168.1.1` | Base URL, no trailing slash |
| `{$ROS.USER}` | `monitoring` | Read-only API username |
| `{$ROS.PASS}` | _(secret)_ | API password |
| `{$ROS.CPU.WARN}` | `80` | CPU warning threshold (%) |
| `{$ROS.CPU.HIGH}` | `95` | CPU high threshold (%) |
| `{$ROS.MEM.WARN}` | `80` | Memory warning threshold (%) |
| `{$ROS.MEM.HIGH}` | `90` | Memory high threshold (%) |
| `{$ROS.REBOOT.WINDOW}` | `300` | Uptime (s) below which unexpected reboot is flagged |

> **Note on BGP/OSPF:** On devices without BGP or OSPF configured, these endpoints return empty arrays and fail silently — no unsupported items or errors.

---

### `Mikrotik-Routing-by-HTTP` (Routing/compliance)
Supplemental template focused on routing protocol state and configuration compliance. Designed to be linked alongside other MikroTik templates rather than used standalone.

**Monitored:**

*BGP*
- BGP peer discovery — per-peer session state
- Trigger: session not established

*OSPF*
- OSPF neighbor discovery — per-neighbor state and cost
- Trigger: neighbor not in Full state, cost changed

*Configuration compliance*
- System scheduler — entry count and names
- System scripts — count and names
- Trigger: script count changed (flags unexpected additions or removals)

**Macros:**

| Macro | Default | Description |
|-------|---------|-------------|
| `{$ROS_USER}` | `zabbix` | API username |
| `{$ROS_PASS}` | _(secret)_ | API password |

---

## Setup

### 1. Create a read-only API user on the RouterOS device

```
/user group add name=zabbix-ro policy=read,api,rest-api
/user add name=zabbix group=zabbix-ro password=yourpassword
```

### 2. Enable the REST API service

For HTTP (lab/internal only):
```
/ip service enable www
```

For HTTPS (recommended for production):
```
/ip service enable www-ssl
```

### 3. Import the template(s)

In Zabbix: **Configuration → Templates → Import**

Import whichever templates apply to your device. For a full ISP router you might apply both `Mikrotik RouterOS by HTTP` and `Mikrotik-Routing-by-HTTP`.

### 4. Create a host and set macros

Add the router as a host in Zabbix and set the required macros (URL/credentials) at the host level. The interface type doesn't matter — HTTP agent items don't use the Zabbix agent interface.

---

## Compatibility

| RouterOS Version | Status |
|-----------------|--------|
| 7.1 – 7.x | ✅ Supported |
| 6.x and earlier | ❌ REST API not available |

Tested on RouterOS 7.20.8 (long-term) and Zabbix 8.0.

### Hardware coverage

The templates are designed to work across MikroTik hardware families:

| Hardware | Notes |
|----------|-------|
| CCR1036, CCR1072 | Full support — health sensors (voltage, temp, fan), high core count |
| CCR2004 | Full support |
| L009, hEX, RB series | Full support — health sensors limited to what hardware exposes |
| Virtual/CHR | No health sensor data — fails silently |

---

## A note on macro naming

These templates were built at different times and use slightly different macro naming conventions:

- `MikroTik by REST API` uses `{$API_HOST}`, `{$API_USER}`, `{$API_PASSWORD}`
- `Mikrotik RouterOS by HTTP` uses `{$ROS.URL}`, `{$ROS.USER}`, `{$ROS.PASS}`
- `Mikrotik-Routing-by-HTTP` uses `{$ROS_USER}`, `{$ROS_PASS}`

If applying multiple templates to the same host, set all relevant macros. A future revision will standardize macro naming across the collection.

---

## License

MIT — free to use, modify, and distribute. Attribution appreciated but not required.

---

## Feedback & Contributions

Issues, suggestions, and pull requests welcome. If these templates save you time, consider sponsoring via GitHub Sponsors or Ko-fi.
