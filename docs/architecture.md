# Architecture

## Overview

This repository contains two monitoring patterns:

1. **SNMP-based hardware monitoring** for vendor appliances and server management controllers
2. **Agent-based monitoring** for Proxmox RAID physical disks using `smartctl`

## Flow

```text
Vendor MIBs / smartctl
        |
        v
  Useful OIDs / attributes identified
        |
        v
  Zabbix items + LLD discovery rules
        |
        v
  Value maps + preprocessing
        |
        v
  Normalized status items
        |
        v
  Honeycomb / overview dashboards
```

## Design principles

- Keep vendor-specific raw values available where helpful
- Add normalized items for shared dashboard behavior
- Use discovery for scalable hardware monitoring
- Separate platform-specific collection methods from Zabbix presentation logic

## Status normalization philosophy

Many vendors expose incompatible values. This repo tries to present a simpler mental model:

- `0` unknown
- `1` healthy
- `3` warning / degraded
- `4` rebuilding / in progress
- `5` failed / critical

Not every vendor uses every state, but dashboards remain predictable.
