# Task 1 — FortiGate Firewall VM Deployment

## Objective
Deploy FortiGate VM on VMware Workstation, configure virtual network interfaces, and establish management access via CLI and GUI.

---

## Step 1 — Download FortiGate VM

Obtain the FortiGate VM image from the Fortinet support portal:
```
https://support.fortinet.com/support/#/downloads/vm
```

**Files in the extracted archive:**
| File | Type | Purpose |
|------|------|---------|
| `FortiGate-VM64.ovf` | OVF Template | Primary import file |
| `FortiGate-VM64.vmx` | VM Config | VMware machine config |
| `fortios.vmdk` | Disk Image | FortiOS system disk (~120 GB) |
| `datadrive.vmdk` | Disk Image | Data storage disk |

> **Note:** Extract the `.zip` archive before importing. The OVF and VMDK files must remain in the same directory.

---

## Step 2 — Configure Virtual Network Interfaces

Open VMware Workstation → **Edit** → **Virtual Network Editor**

> Administrator privileges required. Click **Change Settings** to proceed.

### Network Adapter Assignments

| VMnet | Type | Subnet | Purpose |
|-------|------|--------|---------|
| VMnet0 | Bridged | Auto | Physical network bridge |
| VMnet1 | Host-only | 192.168.139.0/24 | Original host network |
| VMnet2 | Host-only | 192.168.118.0/24 | WAN interface |
| VMnet3 | Host-only | 192.168.164.0/24 | Additional zone |
| VMnet4 | Host-only | 192.168.224.0/24 | Additional zone |
| VMnet8 | NAT | 192.168.134.0/24 | NAT (Management) |
| VMnet11 | Host-only | 192.168.205.0/24 | LAN interface |
| VMnet12 | Host-only | 192.168.85.0/24 | DMZ interface |

**Key Setting:** All custom VMnets must be **Host-only** with unique, non-overlapping subnets.

---

## Step 3 — Import FortiGate VM

1. In VMware Workstation: **File** → **Open** (`Ctrl+O`)
2. Navigate to extracted folder and select `FortiGate-VM64.ovf`
3. Accept the End User License Agreement
4. Provide VM name: `FortiGate-VM64`
5. Set storage path
6. Click **Import**

---

## Step 4 — Configure VM Hardware Resources

After import, click **Edit Virtual Machine Settings**:

| Resource | Value |
|----------|-------|
| Memory | **2 GB RAM** |
| Processors | **1 vCPU** |
| Hard Disk (SCSI) | **2 GB** (system) |
| Hard Disk 2 (SCSI) | **30 GB** (data) |
| Network Adapter 1 | Bridged (port1 — management) |
| Network Adapter 2 | Custom VMnet2 (port2 — WAN) |
| Network Adapter 3 | Custom VMnet11 (port3 — LAN) |
| Network Adapter 4 | Custom VMnet3 (port4) |
| Network Adapter 5 | Custom VMnet4 (port5) |
| Network Adapter 6 | Custom VMnet12 (port6) |

---

## Step 5 — Configure Management Interface via CLI

**Default credentials on first boot:**
```
Username: admin
Password: (press Enter — blank)
```

> You will be forced to set a new password meeting the policy:
> minimum 12 characters, 1 uppercase, 1 lowercase, 1 number, 1 non-alphanumeric

### Inspect Current Interfaces
```bash
get system interface
```

### Assign Static IP to Port1 (Management)
```bash
config system interface
    edit port1
        set mode static
        set ip 192.168.139.131 255.255.255.0
        set allowaccess http https telnet ssh ping
    end
```

### Verify Configuration
```bash
show system interface
```

Expected output for port1:
```
edit "port1"
    set vdom "root"
    set ip 192.168.139.131 255.255.255.0
    set allowaccess ping https ssh http telnet
    set type physical
    set snmp-index 1
next
```

---

## Step 6 — Configure WAN and LAN Interfaces (GUI)

Access GUI: `https://192.168.134.10`

### WAN Interface (port2)
- **Role:** WAN
- **Addressing Mode:** Manual
- **IP/Netmask:** 192.168.118.2/255.255.255.0
- **Administrative Access:** PING, HTTPS, SSH, SNMP

### LAN Interface (port3)
- **Role:** LAN
- **Addressing Mode:** Manual
- **IP/Netmask:** 192.168.205.2/255.255.255.0
- **Enable DHCP Server:** Yes
  - Address Range: 192.168.205.10 – 192.168.205.100
  - Netmask: 255.255.255.0
  - Default Gateway: Same as Interface IP
  - Lease Time: 604800 seconds

---

## Step 7 — Add Default Route

**Navigate to:** Network → Static Routes → Create New

| Field | Value |
|-------|-------|
| Destination | 0.0.0.0 / 0.0.0.0 |
| Gateway Address | 192.168.118.1 (ISP gateway) |
| Interface | port2 (WAN) |
| Administrative Distance | 10 |
| Status | Enabled |

---

## Step 8 — Verify Internet Connectivity

```bash
# From FortiGate CLI
execute ping 8.8.8.8
execute ping google.com
```

**Expected Result:**
```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=119 time=43.1 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=60.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=43.4 ms
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
```

✅ **Task 1 Complete** — FortiGate VM deployed and internet routing verified.

---

## Screenshots Reference

| Screenshot | Description |
|-----------|-------------|
| `task1/ss01_fortinet_download.png` | FortiGate VM download portal |
| `task1/ss02_extracted_files.png` | Extracted archive contents |
| `task1/ss03_vnet_editor.png` | VMware Virtual Network Editor |
| `task1/ss04_admin_privileges.png` | Change Settings prompt |
| `task1/ss05_multiple_vnets.png` | Multiple host-only interfaces |
| `task1/ss06_ovf_import.png` | OVF file import dialog |
| `task1/ss07_vm_name.png` | VM naming and storage path |
| `task1/ss08_vm_settings.png` | VM hardware settings panel |
| `task1/ss09_nic_config.png` | Six NICs assigned to VMnets |
| `task1/ss10_cli_login.png` | Initial CLI login |
| `task1/ss11_get_interface.png` | get system interface output |
| `task1/ss12_port1_config.png` | Port1 IP assignment commands |
| `task1/ss13_show_interface.png` | show system interface output |
| `task1/ss14_ping_management.png` | Ping management IP |
| `task1/ss15_gui_login.png` | FortiGate GUI login page |
| `task1/ss16_setup_wizard.png` | Initial setup wizard |
| `task1/ss17_interfaces_menu.png` | Network > Interfaces menu |
| `task1/ss18_wan_config.png` | WAN interface configuration |
| `task1/ss19_lan_config.png` | LAN interface configuration |
| `task1/ss20_dhcp_server.png` | DHCP server settings |
| `task1/ss21_static_routes.png` | Static routes table |
| `task1/ss22_default_route.png` | Default route creation |
| `task1/ss23_internet_test.png` | Internet connectivity test |
