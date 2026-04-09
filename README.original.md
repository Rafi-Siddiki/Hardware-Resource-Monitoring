# Zabbix# Zabbix Monitoring Templates and Deployment

This repository contains my Zabbix deployment assets, custom templates, and supporting documentation for monitoring server and storage hardware using both SNMP and Zabbix Agent 2.

It includes vendor-specific templates for Dell, HPE, IBM, Lenovo, Synology, and an agent-based Proxmox RAID SMART monitoring implementation.

## What this repository covers

- Zabbix deployment with PostgreSQL and Zabbix Web
- Vendor-specific SNMP templates
- Agent-based Proxmox RAID physical disk monitoring
- Unified numeric state mapping for dashboards
- Honeycomb/tile-friendly status values
- Documentation for template creation workflow

## Included templates

| Vendor / Platform | Method | Purpose |
|---|---|---|
| Dell PowerEdge R740 | SNMP | Server health, RAID, disks, sensors |
| HPE ProLiant DL380 | SNMP | Unified hardware monitoring with normalized dashboard items |
| IBM IMM | SNMPv3 | Unified hardware and RAID monitoring |
| Lenovo XCC | SNMPv3 | Unified hardware and RAID monitoring |
| Synology NAS | SNMP | NAS, disks, RAID, thermal, SSD wear |
| Proxmox RAID SMART | Zabbix Agent 2 | RAID physical disk health and SSD wear using smartctl |

## Monitoring approach

### 1. SNMP-based vendor monitoring
For physical servers and appliances, I used each vendor's MIBs to identify useful OIDs and then built or customized Zabbix templates to collect hardware, RAID, health, sensor, and system status data.

### 2. Proxmox RAID monitoring with Zabbix Agent 2
For Proxmox RAID monitoring, I used a Linux-side script executed through Zabbix Agent 2. The script collects SMART and RAID disk information from the host and exposes it to Zabbix using UserParameter definitions.

### 3. Unified dashboard logic
To make dashboards easier to understand visually, I mapped hardware states into normalized numeric values that can be rendered as tiles/honeycombs with colors:

- `0` = Unknown / not available
- `1` = OK / healthy / online
- `2` = Spare / standby
- `3` = Warning / degraded
- `4` = Rebuilding / in progress
- `5` = Failed / critical / offline

This allows a consistent dashboard style across different vendors, even when their raw SNMP values differ.

## Template development workflow

My workflow for creating templates was:

1. Identify the hardware platform and available MIBs
2. Walk the server MIB/SNMP tree
3. Find the OIDs relevant to health, RAID, disks, fans, PSU, memory, CPU, and sensors
4. Test the values manually
5. Build Zabbix items, discovery rules, triggers, and value mappings
6. Normalize state values for dashboard use
7. Use AI assistance to accelerate YAML template generation and cleanup
8. Validate the result by importing into Zabbix and testing on real systems

## Repository layout

```text
.
├── docker/                 # Zabbix deployment files
├── templates/              # Vendor/platform-specific template YAML files
├── scripts/                # Helper scripts for agent-based monitoring
├── zabbix-agent/           # Agent configs and UserParameter examples
├── examples/               # Sudoers examples and screenshots
└── docs/                   # Detailed documentation