# Hardware Resource Monitoring

![Screenshot](images/image5.png)

A production-oriented collection of **Zabbix 7.4 hardware monitoring templates** for mixed infrastructure environments.

> **Core workflow:** Read vendor MIBs -> identify useful OIDs -> build Zabbix templates -> normalize statuses into a shared numeric model -> display them consistently in dashboards and honeycomb tiles.

---

## 📌 At a glance

This repository combines:

- **SNMP-based vendor templates** for Dell, HPE, IBM, Lenovo, and Synology
- **Agent-based Proxmox RAID monitoring** for MegaRAID-backed disks and arrays
- **Normalized value mapping** for consistent dashboard colors and meanings
- **Discovery-heavy monitoring** for disks, RAID, power, fans, memory, CPU, and other hardware components

### Supported monitoring paths

| Monitoring path | Source | Main purpose |
|---|---|---|
| Vendor hardware monitoring | SNMP / SNMPv3 | BMC, RAID, PSU, fan, memory, CPU, temp, sensors |
| Proxmox RAID disk monitoring | Zabbix Agent 2 + `smartctl` | SSD wear, HDD SMART health |
| Proxmox RAID array monitoring | Zabbix Agent 2 + `StorCLI` / `PercCLI` | Physical disk RAID state, virtual drive RAID state |

---

## 🎯 Why this repository exists

The main reason for building this setup was simple:

> **The company should not need to manually check every single hardware component every time it wants to confirm system health.**

Before this monitoring approach:

- hardware status checks were not centralized
- checks were repetitive and manual
- different vendors exposed information differently
- there was no single automated view for quick health validation

This repository solves that by creating a structured monitoring approach where Zabbix can:

- automatically collect hardware data
- discover components dynamically
- normalize vendor-specific states
- show clear dashboard colors and tiles
- reduce manual checking effort

So the value of this repository is not only in the templates themselves — it is in turning a **manual hardware validation process** into a more **automated, repeatable, and operationally useful monitoring system**.

---

## 🧭 What is a monitoring tool?

A monitoring tool is a platform that continuously checks the health, status, and behavior of infrastructure components such as:

- servers
- disks and RAID controllers
- fans, PSUs, and temperature sensors
- memory and CPU
- network interfaces
- storage appliances and NAS devices

Instead of engineers manually logging into each device and checking hardware one by one, a monitoring tool collects the data centrally, evaluates it, and highlights what is healthy, degraded, or failed.

In this repository, **Zabbix** is the monitoring platform used to collect hardware data through **SNMP** and **Zabbix Agent 2**.

---

## 🚨 Why monitoring is needed

In the current environment, hardware checks often have to be done **manually** across multiple devices and vendors.

That creates several operational problems:

- it is **time-consuming**
- it is **error-prone**
- it depends too much on **manual effort**
- it is difficult to maintain a **consistent checking process**
- hardware issues can be **missed or noticed late**

As the environment grows, repeated manual checking becomes harder to sustain. This repository was built to replace that with a more structured, automated approach.

---

## 🏗️ Project start and deployment approach

When this project started, the first design decision was to run **Zabbix inside a container on a virtual machine**.

That decision was made to make the setup:

- easier to **replicate**
- easier to **rebuild**
- easier to **move between environments**
- easier to **document in a clean and repeatable way**

Using a containerized Zabbix deployment on a VM gave a practical balance between:

- VM-level isolation and infrastructure control
- container-level portability
- simpler documentation and repeatability for future deployments

This repository should therefore be understood not only as a template collection, but also as the documented result of building a **replicable monitoring environment**.

---

## 🛠️ Template development journey

One of the main practical challenges during this project was hardware template availability.

In the data center, one of the most common server models is the **Lenovo SR650**. Zabbix did not provide a built-in template that matched the monitoring needs for this hardware, so the initial plan of simply importing an official template was not possible.

### What happened first

The first step was to search online for an existing Lenovo community template. Only **one community template** could be found that was somewhat relevant.

### Why that was not enough

Although that template was helpful as a starting point, it was **not suitable enough for the environment and monitoring requirements**. In practice, that meant:

- the available items were incomplete
- the structure was not aligned with the desired dashboard style
- the extracted values were not consistent enough for unified visual monitoring
- it did not fully solve the problem of cross-vendor status consistency

### What had to be done instead

Because of that, the template work had to move from simple reuse to actual template engineering.

### Practical workflow

1. Inspect the vendor **MIB tree**
2. Browse the structure using the **Observium MIB browser / database**
3. Identify the useful OIDs required for monitoring
4. Test and validate which values are operationally meaningful
5. Build Zabbix items, discovery rules, value maps, and dashboards from those OIDs

### AI-assisted template building

AI was used as an accelerator for:

- generating template YAML structure faster
- building discovery prototypes more quickly
- producing initial value maps
- iterating on vendor-specific monitoring logic
- helping maintain **value consistency across templates** for unified dashboards

AI made the process faster, but the monitoring logic still depended on:

- understanding vendor MIBs
- validating OIDs manually
- checking extracted values against actual device behavior
- aligning value maps so dashboards stay visually consistent

> This project was not simply “generate a template and import it.”  
> It was a structured process of finding the correct MIB path, selecting useful OIDs, validating outputs, and shaping the result into dashboard-friendly templates.

---

## 🔍 Monitoring approach

### 1. SNMP templates for vendor hardware

These templates monitor out-of-band or appliance hardware through SNMP:

- Dell PowerEdge R720 / iDRAC
- Dell PowerEdge R740
- HPE ProLiant DL380
- IBM IMM
- Lenovo XCC SR650
- Synology NAS

### 2. Agent-based monitoring for Proxmox RAID disks and arrays

For Proxmox RAID-backed disks, SNMP alone is not always enough. In this setup:

- a custom shell script runs on the Linux host
- the script uses **`smartctl`** for per-disk SMART data and SSD wear information
- the script uses **StorCLI / PercCLI** for RAID controller, physical-disk state, and virtual-drive state
- `zabbix-agent2` exposes the data through `UserParameter`
- Zabbix discovers disks and RAID virtual drives and shows:
  - **HDD health**
  - **SSD wear**
  - **physical disk RAID state**
  - **virtual drive RAID state**

### 3. Shared dashboard semantics

Where possible, templates normalize component states into a common dashboard model:

| Value | Meaning |
|---|---|
| `0` | Unknown / not available |
| `1` | OK / normal / online |
| `2` | Spare / standby / unused where applicable |
| `3` | Warning / degraded |
| `4` | Rebuilding / in progress |
| `5` | Failed / critical / offline |

This shared model makes it much easier to build **consistent tiles and honeycomb views** across vendors.

---

## 🧱 Repository structure

```text
zabbix-hardware-monitoring/
├── README.md
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── deployment-guide.md
│   └── template-mapping.md
├── templates/
│   ├── dell/
│   │   ├── DELL PowerEdge R720.yaml
│   │   └── DELL PowerEdge R740.yaml
│   ├── hpe/
│   │   └── HPE ProLiant DL380 SNMP.yaml
│   ├── ibm/
│   │   └── IBM IMM SNMPv3.yaml
│   ├── lenovo/
│   │   └── Lenovo XCC SNMPv3.yaml
│   ├── proxmox/
│   │   └── Proxmox RAID.yaml
│   └── synology/
│       └── Synology NAS SNMP.yaml
├── scripts/
│   └── proxmox/
│       └── proxmox_raid_pd_attr.sh
├── configs/
│   └── zabbix-agent2/
│       ├── proxmox-raid-smart.conf
│       └── zabbix_agent2.conf
└── sudoers/
    └── zabbix-smart-raid
```

---

## 📦 Included templates

| Vendor / Platform | Method | Main coverage |
|---|---|---|
| Dell PowerEdge R720 | SNMP | System health, controllers, virtual disks, physical disks, fan / temperature / power |
| Dell PowerEdge R740 | SNMP | System health, controllers, virtual disks, physical disks, fan / temperature / power |
| HPE ProLiant DL380 | SNMP | Unified health model, array cache, controllers, disks, network adapters, fans |
| IBM IMM | SNMPv3 | System health, storage pools, RAID PD/VD, PSU, temperature, fan, SSD wear |
| Lenovo XCC | SNMPv3 | Hardware, PSU, fan, memory, CPU, firmware, RAID PD/VD, SSD wear |
| Synology NAS | SNMP | System, disks, RAID health, temperature, fans, DSM info, SSD wear |
| Proxmox RAID SMART | Agent2 + `smartctl` + `StorCLI` | MegaRAID disk discovery, VD discovery, HDD health, SSD wear, PD state, VD state, dynamic status |

---

## 🚀 Get Zabbix up and running

```bash
git clone https://github.com/Rafi-Siddiki/zabbix-infrastructure-monitoring.git
cd zabbix-infrastructure-monitoring/docker/
docker compose up -d
```

### Default login

| Field | Value |
|---|---|
| Username | `Admin` |
| Password | `zabbix` |

---

## ✅ Key design decisions

### 1. Value normalization for dashboard colors

Different vendors report different states. This repository maps them into predictable dashboard values so tiles can use a common visual language:

- 🟩 **Green** -> OK / normal / online
- 🟦 **Blue** -> hot-spare / unconfigured-good / standby / healthy non-active role
- 🟨 **Yellow / orange** -> warning / degraded / rebuilding
- 🟥 **Red** -> failed / critical / offline
- ⬜ **Gray** -> unknown / unavailable

### 2. Discovery-first design

Low-level discovery is heavily used for:

- physical disks
- virtual disks
- RAID arrays / controllers
- fans
- PSUs
- memory modules
- network adapters
- firmware blocks

### 3. Separation of vendor logic and dashboard logic

Vendor-specific logic stays inside each template, while normalized items make dashboards easier to share and maintain.

### 4. Consistent value mapping for unified dashboards

One important design goal in this project was to keep dashboard semantics as consistent as possible across vendors.

That means the templates were adjusted so that, where reasonable:

- the same numeric status means the same visual severity
- honeycomb tiles can use the same threshold logic
- cross-vendor dashboards remain easier to understand operationally

This is especially important in mixed environments where Dell, HPE, Lenovo, IBM, Synology, and Proxmox systems may all appear in the same monitoring platform.

---

# 🧠 Value mapping model: the most important concept

This is the most important section for understanding how the dashboards work.

## 1. Raw vendor mapping vs unified mapping

Many templates contain **raw vendor items** and **normalized dashboard items**.

### Raw vendor item
A raw vendor item shows the vendor’s original meaning exactly as reported by SNMP or the controller CLI.

Examples:

- Dell virtual disk state from iDRAC
- HPE logical drive status from `cpqDaLogDrvStatus`
- StorCLI raw physical disk state like `Onln` or `Rbld`

Raw vendor values are important for:

- troubleshooting
- confirming the original source data
- understanding device-native terminology

### Unified dashboard item
A unified dashboard item translates the raw vendor meaning into a shared numeric model that can be used consistently across dashboards.

Examples:

- `1 = Online`
- `3 = Degraded`
- `5 = Failed`

These normalized items are what should be used in shared dashboard widgets.

> **Important rule**  
> Do not assume every raw vendor value already follows the shared dashboard model.  
> In several templates, the shared model is implemented through a dependent item or preprocessing step.

---

## 2. Physical disk state vs virtual drive state vs severity

These three concepts are different and should not be mixed.

### A. Physical disk state
This describes the state or role of **one member disk** behind a RAID controller.

#### Recommended shared model

| Value | Meaning |
|---|---|
| `0` | Unknown |
| `1` | Online |
| `2` | Hot spare |
| `3` | Unconfigured good |
| `4` | Rebuilding |
| `5` | Offline / Failed / Missing |

This model is appropriate because a single physical disk can be:

- online
- spare
- good but unused
- rebuilding
- failed

### B. Virtual drive state
This describes the state of the **whole RAID logical volume**.

#### Recommended shared model

| Value | Meaning |
|---|---|
| `0` | Unknown |
| `1` | Online / Optimal |
| `3` | Degraded |
| `4` | Rebuilding / recovery / operation in progress |
| `5` | Failed / Offline |

This model is different from physical disk state because a virtual drive cannot meaningfully be a hot-spare or unconfigured-good member role.

### C. Severity / health status
This is the dashboard-friendly summary status used for health and dynamic items.

#### Recommended shared model

| Value | Meaning |
|---|---|
| `0` | Unknown |
| `1` | OK |
| `3` | nonCritical / Warning |
| `4` | Rebuilding / in progress *(optional, only if the item family uses it)* |
| `5` | Critical / Failed |

This type is suitable for:

- SMART health
- dynamic health summaries
- controller-derived warning / critical items

---

## 3. Why physical disk and virtual drive states must stay separate

A single disk can fail while the whole array is still only degraded.

### Example

- one physical disk fails -> **physical disk state = `5`**
- the RAID5 virtual drive is still accessible but degraded -> **virtual drive state = `3`**

That is correct and expected.

If those two layers are mixed into one value map, dashboards become misleading.

### Correct design

- **Physical disk items** -> use the physical disk state map
- **Virtual drive items** -> use the virtual drive state map
- **Health / dynamic items** -> use the severity map

---

## 4. How Proxmox fits into this model

The Proxmox RAID template uses the shared model in the following way.

### SMART / wear side
Collected from:

- `smartctl`

Used for:

- SSD wear remaining
- HDD SMART health
- SMART-based dynamic status

### RAID controller side
Collected from:

- `StorCLI` / `PercCLI`

Used for:

- physical disk RAID state
- virtual drive RAID state
- controller-derived RAID dynamic status

That means the Proxmox template is intentionally split into:

- a **disk health / wear** branch
- a **RAID state** branch

This is not duplication. It is a deliberate design decision.

---

## 5. Architecture explanation in one sentence

> The repository preserves vendor-native source items for troubleshooting, but it builds normalized items for shared dashboards. Physical disk state, virtual drive state, and severity are treated as separate monitoring concepts because they represent different layers of RAID behavior.

---

# 💽 Proxmox RAID SMART setup

The Proxmox part of this repository expects a custom script and `zabbix-agent2` configuration on the Linux host.

## Dependency summary

| Dependency | Why it is needed |
|---|---|
| `zabbix-agent2` | exposes custom monitoring items |
| `smartmontools` | provides `smartctl` for SMART data and SSD wear |
| `sudo` | allows the Zabbix user to run the script safely |
| `python3` | used by the script for JSON parsing / escaping |
| `StorCLI` or `PercCLI` | provides RAID controller, PD state, and VD state data |

---

## Why both `smartctl` and `StorCLI` are required

This setup uses **two different data sources** on purpose.

### `smartctl`
Used for:

- per-disk SMART data
- SSD wear / endurance information
- HDD SMART health
- vendor-specific disk-level attributes

### `StorCLI` / `PercCLI`
Used for:

- RAID controller visibility
- physical disk RAID state
- virtual drive RAID state
- fast controller-driven discovery of arrays and member states

> **Important**  
> `smartctl` alone is not enough for reliable RAID virtual-drive state.  
> `StorCLI` alone is not enough for vendor-specific SMART wear interpretation.  
> Both are required to provide the full monitoring model.

---

## Installing dependencies on Debian / Ubuntu / Proxmox

### Install packages from the OS repository

```bash
sudo apt update
sudo apt install zabbix-agent2 smartmontools sudo python3 unzip -y
```

### Install StorCLI

#### Step 1: download the official Linux StorCLI package
Download the Linux StorCLI package from Broadcom for your controller family.

#### Step 2: extract the archive

```bash
unzip <Broadcom_StorCLI_package>.zip
cd <extracted_directory>
```

#### Step 3: find the Debian package

```bash
find . -type f \( -name "*.deb" -o -name "storcli64" \)
```

#### Step 4: install the `.deb` package

```bash
sudo dpkg -i ./<storcli_package>.deb
```

If dependency resolution is needed:

```bash
sudo apt -f install -y
```

#### Step 5: verify installation

```bash
/opt/MegaRAID/storcli/storcli64 show
```

If the package places the binary somewhere else, adjust the path accordingly.

---

## Installing dependencies on RHEL / CentOS / AlmaLinux / Rocky

### Install packages from the OS repository

```bash
sudo yum install -y zabbix-agent2 smartmontools sudo python3 unzip
```

For newer distributions that use `dnf`:

```bash
sudo dnf install -y zabbix-agent2 smartmontools sudo python3 unzip
```

### Install StorCLI from the official RPM package

#### Step 1: download the official Linux StorCLI package
Download the Linux StorCLI package from Broadcom for your controller family.

#### Step 2: extract the archive

```bash
unzip <Broadcom_StorCLI_package>.zip
cd <extracted_directory>
```

#### Step 3: install the RPM package

```bash
sudo rpm -ivh <StorCLI-*.rpm>
```

#### Step 4: verify installation

```bash
/opt/MegaRAID/storcli/storcli64 show
```

---

## Place the custom script

```bash
sudo nano /usr/local/bin/proxmox_raid_pd_attr.sh
sudo chown root:root /usr/local/bin/proxmox_raid_pd_attr.sh
sudo chmod 755 /usr/local/bin/proxmox_raid_pd_attr.sh
```

---

## Allow Zabbix to run the script

```bash
sudo visudo -f /etc/sudoers.d/zabbix-smart-raid
```

Expected content:

```sudoers
zabbix ALL=(ALL) NOPASSWD: /usr/local/bin/proxmox_raid_pd_attr.sh
```

Then set the correct permission:

```bash
sudo chmod 440 /etc/sudoers.d/zabbix-smart-raid
```

---

## Configure Zabbix Agent 2

```bash
sudo cp /etc/zabbix/zabbix_agent2.conf /etc/zabbix/zabbix_agent2.conf.bak
sudo nano /etc/zabbix/zabbix_agent2.conf
sudo nano /etc/zabbix/zabbix_agent2.d/proxmox-raid-smart.conf
```

### Notes for the agent configuration

- make sure the `AllowKey=` entries match the item keys used by the imported template
- use only one custom include file for the RAID keys to avoid duplicate `UserParameter` definitions
- confirm `Timeout` stays within the valid range supported by your Zabbix agent version
- if the agent fails to start, test with:

```bash
sudo /usr/sbin/zabbix_agent2 -f -c /etc/zabbix/zabbix_agent2.conf
```

---

## Test manually

```bash
/usr/local/bin/proxmox_raid_pd_attr.sh discover /dev/sda
/usr/local/bin/proxmox_raid_pd_attr.sh vddiscover /dev/sda
/usr/local/bin/proxmox_raid_pd_attr.sh wearpct /dev/sda 0
/usr/local/bin/proxmox_raid_pd_attr.sh vdstate /dev/sda 0/0
/usr/local/bin/proxmox_raid_pd_attr.sh pdstate /dev/sda 0
```

Then test through Zabbix Agent 2:

```bash
sudo zabbix_agent2 -c /etc/zabbix/zabbix_agent2.conf -t 'proxmox.raid.pd.discovery'
sudo zabbix_agent2 -c /etc/zabbix/zabbix_agent2.conf -t 'proxmox.raid.vd.discovery'
sudo zabbix_agent2 -c /etc/zabbix/zabbix_agent2.conf -t 'proxmox.raid.pd.wear.pct[0]'
sudo zabbix_agent2 -c /etc/zabbix/zabbix_agent2.conf -t 'proxmox.raid.vd.state[0/0]'
```

---

## Restart and verify

```bash
sudo systemctl enable zabbix-agent2
sudo systemctl restart zabbix-agent2
sudo systemctl status zabbix-agent2 --no-pager
sudo ss -tulpn | grep :10050
```

---

## Importing templates into Zabbix

1. Open **Zabbix -> Data collection -> Templates**
2. Click **Import**
3. Select the YAML file from the relevant vendor folder under `templates/`
4. Review value maps, dashboards, items, and discovery rules
5. Import the template
6. Link the template to the matching host
7. Verify latest data, discovery, and dashboard tiles

---

## Adding a device into Zabbix

Before adding a device in the Zabbix UI, the most important prerequisite is that the device must already be prepared for monitoring.

### Step 0: create an SNMP monitoring account on the device

For SNMP-based monitoring, you first need to create or enable an **SNMP monitoring account / SNMP configuration** on the server, appliance, or management controller.

This part is vendor-specific.

Different devices have different:

- web interfaces
- menu paths
- authentication methods
- SNMP version support
- security settings
- permission models

So in practice, this means you often need to do some **device-specific research** to find the correct method for that particular hardware.

Examples:

- Dell iDRAC SNMP configuration is different from Lenovo XCC
- Lenovo XCC setup is different from IBM IMM
- Synology DSM SNMP setup is different from server BMC interfaces
- HPE iLO / SNMP process differs again

### Recommended preparation checklist

Before creating the host in Zabbix, make sure you know:

- device IP / DNS name
- vendor and model
- whether it uses **SNMPv2c** or **SNMPv3**
- community string or SNMPv3 credentials
- the correct template to apply
- whether firewall rules allow Zabbix to reach UDP/161

---

## Host onboarding in the Zabbix UI

### 1. Navigate to **Data collection -> Hosts**

![Screenshot](images/image1.png)

### 2. Click **Create host**

![Screenshot](images/image2.png)

### 3. Fill in the basic host information

Provide the following carefully:

1. **Hostname** — must be unique for every device
2. **Visible name** — the display name shown in Zabbix
3. **Template** — select the correct vendor template from this repository
4. **Host group** — choose a group that makes filtering easier later
5. **Interface type** — choose the required interface type, most commonly **SNMP**

![Screenshot](images/image3.png)

### 4. Fill in the SNMP interface details

Enter the required SNMP fields according to the SNMP version used by the device:

- for **SNMPv2c**, configure the community string
- for **SNMPv3**, configure the username, authentication, privacy method, and related credentials

Make sure the SNMP settings entered in Zabbix match exactly what was configured on the device.

![Screenshot](images/image4.png)

### 5. Validate after adding the host

After saving the host:

- confirm the template linked successfully
- check **Latest data**
- confirm discovery rules start creating items
- verify that dashboard tiles and honeycomb widgets show expected values
- test whether the value mapping looks correct for that device type

---

## 📝 Maintainer note

This repository reflects a practical hardware monitoring implementation where templates were developed by:

- understanding the operational monitoring need
- containerizing Zabbix on a VM for easier replication
- reading vendor MIBs
- identifying useful OIDs
- checking MIB trees through the Observium MIB browser/database
- generating and refining template logic with AI assistance
- validating extracted values against actual hardware behavior
- mapping hardware states into consistent dashboard values

In other words, this repository is the result of both **monitoring design** and **template engineering**, built to reduce manual hardware checking and provide clearer, more unified infrastructure visibility.
