# Template Mapping and Notes

## Dell PowerEdge R720
- SNMP-based template
- Focuses on controller, battery, physical disk, virtual disk, fan, PSU, and temperature monitoring
- Dashboard widgets are present in the exported template

## HPE ProLiant DL380 Unified
- Rebuilt into a unified style
- Preserves official-style vendor logic while adding normalized dashboard items
- Best example of the shared cross-template status model

## IBM IMM SNMPv3
- Unified IMM template
- Includes storage-oriented coverage such as RAID controllers, physical drives, storage pools, and virtual drives
- Useful for mixed hardware dashboards

## Lenovo XCC SNMPv3
- Unified “God Mode” style template
- Includes RAID physical/virtual disks and SSD wear handling
- Good reference template for normalization style

## Synology NAS SNMP
- Covers system, disks, RAID, temperature, fans, and DSM info
- Includes SSD wear and RAID health summaries

## Proxmox RAID SMART
- Agent-based rather than SNMP-based
- Uses smartctl against MegaRAID-backed disks
- Separates HDD health from SSD wear using template overrides
