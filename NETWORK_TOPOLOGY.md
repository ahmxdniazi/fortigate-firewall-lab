# Network Topology

## Lab Environment Overview

This document describes the complete network topology used in the FortiGate Firewall Security Lab.

---

## Physical / Virtual Host

| Component | Details |
|-----------|---------|
| Host OS | Windows 10 x64 |
| Hypervisor | VMware Workstation Pro |
| Host RAM | 16 GB+ |
| Host CPU | Intel/AMD 64-bit with VT-x/AMD-V |

---

## Network Diagram

```
                         PHYSICAL HOST MACHINE
    ╔══════════════════════════════════════════════════════════════╗
    ║                                                              ║
    ║   ┌──────────────────────────────────────────────────────┐  ║
    ║   │           VMware Virtual Network Editor               │  ║
    ║   │                                                       │  ║
    ║   │  VMnet0 ─── Bridged ──────── Physical NIC (WAN)      │  ║
    ║   │  VMnet1 ─── Host-only ─────── 192.168.139.0/24       │  ║
    ║   │  VMnet2 ─── Host-only ─────── 192.168.118.0/24 (WAN) │  ║
    ║   │  VMnet3 ─── Host-only ─────── 192.168.164.0/24       │  ║
    ║   │  VMnet4 ─── Host-only ─────── 192.168.224.0/24       │  ║
    ║   │  VMnet8 ─── NAT ───────────── 192.168.134.0/24 (Mgmt)│  ║
    ║   │  VMnet11 ── Host-only ─────── 192.168.205.0/24 (LAN) │  ║
    ║   │  VMnet12 ── Host-only ─────── 192.168.85.0/24        │  ║
    ║   └──────────────────────────────────────────────────────┘  ║
    ║                           │                                   ║
    ║            ┌──────────────┴──────────────────┐               ║
    ║            │                                 │               ║
    ║   ┌────────▼────────┐             ┌──────────▼───────┐       ║
    ║   │  FORTIGATE VM   │             │  KALI LINUX VM   │       ║
    ║   │                 │             │                  │       ║
    ║   │ port1: 134.10   │             │ eth0: 205.10     │       ║
    ║   │ port2: 118.2    │             │ (VMnet11 - LAN)  │       ║
    ║   │ port3: 205.2    │             └──────────────────┘       ║
    ║   │ (FW + Router)   │                                        ║
    ║   └────────┬────────┘             ┌──────────────────┐       ║
    ║            │                      │  WINDOWS 10 VM   │       ║
    ║            │ NAT (VMnet8)         │                  │       ║
    ║            │                      │ IP: 192.168.205.x│       ║
    ║        INTERNET                   │ (VMnet11 - LAN)  │       ║
    ║        8.8.8.8                    └──────────────────┘       ║
    ║        google.com                                             ║
    ╚══════════════════════════════════════════════════════════════╝
```

---

## Interface Mapping

### FortiGate VM Interfaces

| Port | VMnet | IP Address | Subnet | Role |
|------|-------|------------|--------|------|
| port1 | VMnet8 (NAT) | 192.168.134.10 | /24 | Management / WAN |
| port2 | VMnet2 | 192.168.118.2 | /24 | WAN |
| port3 | VMnet11 | 192.168.205.2 | /24 | LAN |
| port4 | VMnet3 | — | /24 | Reserved |
| port5 | VMnet4 | — | /24 | Reserved |
| port6 | VMnet12 | — | /24 | Reserved |

### Client VMs

| VM | Interface | VMnet | IP Address | Segment |
|----|-----------|-------|------------|---------|
| Kali Linux | eth0 | VMnet11 | 192.168.205.10 (DHCP) | LAN |
| Windows 10 | Ethernet | VMnet11 | 192.168.205.x (DHCP) | LAN |

---

## Traffic Flow Analysis

### 1. LAN to Internet (Normal)
```
Kali Linux (192.168.205.10)
    │
    │ [Raw packet]
    ▼
FortiGate port3 (LAN ingress)
    │
    │ [Policy: internet-access]
    │ [Action: ACCEPT]
    │ [NAT: Source → 192.168.134.10]
    ▼
FortiGate port1 (WAN egress)
    │
    ▼
Internet (8.8.8.8 / google.com)
```

### 2. LAN to Facebook (Blocked)
```
Kali Linux (192.168.205.10) → https://www.facebook.com
    │
    ▼
FortiGate port3 (LAN ingress)
    │
    │ [Policy: No-Facebook-Internet-Access]
    │ [Web Filter: default profile]
    │ [URL Check: facebook.com → BLOCK (wildcard)]
    │
    ▼
    ⊘ DENY (UTM Blocked) — packet dropped
    Browser receives: "Did Not Connect" error
```

### 3. SYN Flood Attack (Blocked by DoS Policy)
```
Kali Linux → hping3 --flood → 192.168.205.2
    │
    │ [DoS Policy: Dos-protection-policy]
    │ [Anomaly: tcp_syn_flood]
    │ [Rate: >2000 pps detected]
    │
    ▼
    ⊘ BLOCK — All flood packets dropped
    Result: 100% packet loss at attacker
```

---

## IP Address Summary

| Device | Interface | IP Address | Notes |
|--------|-----------|------------|-------|
| FortiGate | port1 (Mgmt) | 192.168.134.10 | GUI & CLI access |
| FortiGate | port2 (WAN) | 192.168.118.2 | Internet uplink |
| FortiGate | port3 (LAN) | 192.168.205.2 | LAN gateway |
| Kali Linux | eth0 | 192.168.205.10 | LAN client |
| Windows 10 | LAN | 192.168.205.x | DHCP from FortiGate |
| Google DNS | — | 8.8.8.8 | Connectivity test target |
| Facebook | — | 157.240.227.35 | Blocked by web filter |

---

## Port & Protocol Reference

| Service | Protocol | Port | Policy |
|---------|----------|------|--------|
| HTTP | TCP | 80 | Allowed (web filter applied) |
| HTTPS | TCP | 443 | Allowed (web filter applied, facebook blocked) |
| DNS | UDP/TCP | 53 | Allowed |
| PING (ICMP) | ICMP | — | Allowed (within threshold) |
| SSH | TCP | 22 | Allowed (management) |
| FortiGate HTTPS GUI | TCP | 443 | Allowed on port1 |

---

*Topology designed for educational lab use — NUCES Islamabad | Advanced Network Security*
