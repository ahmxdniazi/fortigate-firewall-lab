# Technical Report — FortiGate Firewall Security Lab

> **Assignment No. 2 — Advanced Network Security**  
> **Student:** Muhammad Ahmad | **Roll #:** 25I-7705  
> **Program:** MS Cybersecurity  
> **Instructor:** Dr. Zafar Iqbal  
> **Institution:** NUCES (FAST) — Islamabad  

---

## Executive Summary

This report documents the successful deployment and configuration of a **FortiGate Next-Generation Firewall (NGFW)** in a virtualized environment using VMware Workstation. The lab encompassed four key tasks: virtual machine deployment, GUI access configuration, security policy implementation (IPv4 firewall, DoS protection, and web filtering), and live demonstration of all security controls. All tests passed with firewall policies correctly blocking unauthorized traffic while permitting legitimate internet access.

---

## Task 1 — FortiGate Firewall Deployment

### 1.1 VM Image Acquisition
The FortiGate VM image (v7.6.6 build3652) was downloaded from the official Fortinet support portal at `https://support.fortinet.com/support/#/downloads/vm`. The package was extracted to reveal the OVF template and associated VMDK disk images.

### 1.2 Virtual Network Pre-configuration
Prior to VM import, VMware's Virtual Network Editor was used to create and configure multiple host-only virtual network adapters, each assigned a unique non-overlapping subnet. This step is critical as it defines the virtual network segments that FortiGate will later manage and segment.

Eight virtual networks were configured: VMnet0 (Bridged), VMnet1 (192.168.139.0/24), VMnet2 (192.168.118.0/24), VMnet3 (192.168.164.0/24), VMnet4 (192.168.224.0/24), VMnet8 NAT (192.168.134.0/24), VMnet11 (192.168.205.0/24), and VMnet12 (192.168.85.0/24).

### 1.3 VM Import and Hardware Configuration
The FortiGate-VM64.ovf file was imported via VMware Workstation's File > Open dialog. After accepting the EULA, the VM was provisioned with the following hardware specifications:

- **RAM:** 2 GB
- **vCPU:** 1 processor
- **Storage:** 2 GB (system SCSI) + 30 GB (data SCSI)
- **Network Adapters:** 6 virtual NICs mapped to VMnet2, VMnet3, VMnet4, VMnet11, VMnet11, VMnet12

### 1.4 CLI-Based Management Interface Configuration
On first boot, the FortiGate prompted for a new administrator password (enforcing a complexity policy requiring minimum 12 characters with mixed case, numbers, and special characters). The management interface (port1) was then configured via CLI:

```
config system interface
    edit port1
        set mode static
        set ip 192.168.139.131 255.255.255.0
        set allowaccess http https telnet ssh ping
    end
```

The `show system interface` command confirmed the configuration was applied correctly.

### 1.5 WAN and LAN Interface Configuration (GUI)
Accessed via `https://192.168.134.10`, the FortiGate GUI was used to configure:

- **port2 (WAN):** Manual IP 192.168.118.2/24, role WAN
- **port3 (LAN):** Manual IP 192.168.205.2/24, role LAN, DHCP server enabled (pool: 192.168.205.10–100)

A default static route (0.0.0.0/0) was added pointing to the ISP gateway (192.168.118.1) via port2.

### 1.6 Connectivity Verification
Internet connectivity was confirmed via FortiGate CLI:
```
execute ping 8.8.8.8  → 5/5 packets, 0% loss
execute ping google.com → 5/5 packets, 0% loss
```

---

## Task 2 — GUI Access Configuration

### 2.1 HTTPS Access Verification
HTTPS access on port1 was verified through the GUI at Network > Interfaces > port1. The Administrative Access section confirmed HTTPS, HTTP, SSH, PING, and TELNET were enabled.

### 2.2 Administrator Credential Management
The default admin account was accessed via System > Administrators. The account was edited to update the password, maintaining local user authentication type.

### 2.3 Login Verification
A logout/login cycle confirmed the updated credentials functioned correctly. The FortiGate Dashboard displayed system information including hostname (FGVMEVTXU0VWOE55), firmware version (v7.6.6 build3652), and resource utilization.

---

## Task 3 — Policy Implementation

### 3.1 IPv4 Firewall Policy (internet-access)

An IPv4 firewall policy was created to permit LAN-to-WAN internet traffic:

| Parameter | Value |
|-----------|-------|
| Name | internet-access |
| Incoming Interface | port1 (LAN) |
| Outgoing Interface | port2 (WAN) |
| Source/Destination | all |
| Service | DNS, HTTP, HTTPS |
| Action | ACCEPT |
| NAT | Enabled (Use Outgoing Interface Address) |
| Logging | All Sessions |

### 3.2 IPv4 DoS Policy (Dos-protection-policy)

A DoS policy was created on port1 to detect and block flood attacks before they reach firewall policy evaluation:

**L3 Anomalies configured:**
- `ip_src_session`: Block at threshold 5000
- `ip_dst_session`: Block at threshold 5000

**L4 Anomalies configured:**
- `tcp_syn_flood`: Block at 2000 pps
- `tcp_port_scan`: Block at 1000 pps
- `udp_flood`: Block at 2000 pps
- `udp_scan`: Block at 2000 pps
- `icmp_flood`: Block at 250 pps
- `icmp_sweep`: Block at 100 pps
- Multiple SCTP anomalies: Disabled (not applicable to this topology)

### 3.3 Web Filter Policy (No-Facebook-Internet-Access)

**Web Filter Profile Configuration:**
- Feature enabled via System > Feature Visibility
- Default web filter profile edited
- URL filter created: `facebook.com`, Type: Wildcard, Action: Block

**Firewall Policy Configuration:**
- Incoming: port1, Outgoing: port2
- Action: ACCEPT with NAT
- Security Profile: Web Filter → default
- Policy placed at sequence position #2

---

## Task 4 — Demonstration Results

### 4.1 Website Blocking Test
Accessing `https://www.facebook.com` from Kali Linux (192.168.205.10) resulted in:
- Browser error: "Did Not Connect: Potential Security Issue"
- FortiGate log: 103 UTM Block denials recorded
- Traffic log showed consistent `Deny (Deny: UTM Blocked)` by `No-Facebook-Internet-Access` policy

### 4.2 DoS Attack Simulation
```
Command: sudo hping3 -c 15000 -d 120 -S -w 64 -p 80 --flood --rand-source 192.168.205.2
Result:  960,735 packets transmitted, 0 received, 100% packet loss
```
FortiGate security event logs showed 98 events in the summary, with the DoS policy successfully blocking all flood packets.

### 4.3 Legitimate Traffic Validation
Concurrent to security testing, internet connectivity remained fully functional, confirming that security policies did not inadvertently block legitimate traffic.

---

## Conclusion

All four lab tasks were completed successfully. The FortiGate NGFW was deployed in a virtualized environment and configured with three distinct security control layers: stateful IPv4 inspection with NAT, rate-based DoS anomaly protection, and application-layer URL filtering. Live testing confirmed that facebook.com was blocked, SYN flood attacks were mitigated with 100% packet drop, and legitimate internet traffic remained unaffected. This lab demonstrates the viability of multi-layered perimeter security architecture using enterprise-grade NGFW technology.

---

*Submitted in partial fulfillment of the requirements for Advanced Network Security*  
*NUCES (FAST) — Islamabad | MS Cybersecurity Program*
