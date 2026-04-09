# Deployment Guide

## 1. Import vendor templates

Import the matching YAML files from the `templates/` directory into Zabbix.

## 2. Assign templates to the correct host types

- Dell template → Dell iDRAC / supported R720 target
- HPE template → HPE ProLiant DL380 target
- IBM template → IBM IMM target
- Lenovo template → Lenovo XCC target
- Synology template → Synology NAS target
- Proxmox template → Linux host running `zabbix-agent2` and the MegaRAID SMART script

## 3. Configure SNMP correctly

Ensure:

- SNMP version matches the template and host settings
- ACL / firewall permits polling from Zabbix server
- correct community string or SNMPv3 credentials are used
- management interface is reachable from Zabbix server/proxy

## 4. Configure Proxmox host for agent-based collection

Install:

```bash
sudo apt update
sudo apt install zabbix-agent2 smartmontools sudo -y
```

Create the script, UserParameter config, and sudoers rule.

## 5. Validate data collection

Check:

- latest data appears
- discoveries create entities
- triggers behave as expected
- dashboards show normalized colors

## 6. Tune thresholds if needed

Especially review:

- SSD wear thresholds
- warning vs critical trigger levels
- device-specific not-installed / absent states
