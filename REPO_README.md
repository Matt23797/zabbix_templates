# Zabbix Templates

A collection of Zabbix monitoring templates built and tested in a real ISP/NOC environment. All templates use the HTTP agent where possible — no SNMP required unless noted.

**Zabbix version:** 8.0+

---

## Templates

### [`/mikrotik`](./mikrotik)
Templates for MikroTik RouterOS devices via the REST API. Requires RouterOS 7.1+.

| Template | Description |
|----------|-------------|
| `MikroTik by REST API` | General-purpose — system vitals, interfaces, DHCP, firewall connections |
| `Mikrotik RouterOS by HTTP` | WAN/ISP-focused — health sensors, BGP, OSPF, default route, interface traffic |
| `Mikrotik-Routing-by-HTTP` | Supplemental routing/compliance — BGP peers, OSPF neighbors, script and scheduler auditing |

See [`/mikrotik/README.md`](./mikrotik/README.md) for setup instructions and macro reference.

---

### [`/truenas`](./truenas)
Template for TrueNAS SCALE via the REST API. No SNMP configuration required — just an API key.

| Template | Description |
|----------|-------------|
| `Template Storage TrueNAS SCALE` | System vitals, ZFS pools, boot pool, disks, datasets, services, alerts |

See [`/truenas/README.md`](./truenas/README.md) for setup instructions and macro reference.

---

## General Notes

**HTTP agent items** poll device APIs directly. The Zabbix server or proxy needs network access to the monitored device on the relevant port (typically 80 or 443).

**Credentials** are stored as host-level macros marked `SECRET_TEXT` — they are not exported in template YAML and must be set per host.

**Fail-silent design** — where a feature isn't supported by a particular device (e.g. BGP on a device with no BGP config, or health sensors on a VM), endpoints return empty data and no unsupported items or errors are created.

---

## Contributing

Issues and pull requests welcome. If a template saves you time, consider sponsoring via GitHub Sponsors or Ko-fi.
