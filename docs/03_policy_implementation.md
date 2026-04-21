# Task 3 — Policy Implementation

## Objective
Implement three security policies on FortiGate:
1. **IPv4 Firewall Policy** — Allow LAN-to-WAN internet traffic with NAT
2. **IPv4 DoS Policy** — Detect and block network flood attacks
3. **Web Filter Policy** — Block access to facebook.com

---

## A) IPv4 Firewall Policy

### Overview
A firewall policy evaluates each traffic flow against defined criteria (source, destination, service, schedule) and applies the specified action. This policy enables internal LAN clients to access the internet through the WAN interface.

### Configuration Steps

Navigate to: **Policy & Objects** → **Firewall Policy** → **Create New**

| Field | Value | Notes |
|-------|-------|-------|
| Name | `internet-access` | Descriptive identifier |
| Incoming Interface | `port1` | LAN interface |
| Outgoing Interface | `port2` | WAN interface |
| Source | `all` | All internal addresses |
| Destination | `all` | Any internet destination |
| Service | DNS, HTTP, HTTPS | Core internet services |
| Action | **ACCEPT** | Allow traffic |
| NAT | ✅ Enabled | Translate private IPs |
| IP Pool Configuration | Use Outgoing Interface Address | Source NAT |
| Inspection Mode | Flow-based | High performance |
| Log Allowed Traffic | All Sessions | Full visibility |

### Security Profile Settings

| Profile | Status |
|---------|--------|
| AntiVirus | Disabled (lab) |
| Web Filter | Not attached (this policy) |
| DNS Filter | Disabled |
| Application Control | Disabled |
| IPS | Disabled |

### Policy Sequence
This policy appears as entry **#1** in the policy table, evaluated before the web filter policy.

---

## B) IPv4 DoS Policy

### Overview
DoS policies are applied at the interface level *before* standard firewall policies. They analyze traffic patterns and drop traffic matching known attack signatures or exceeding defined rate thresholds.

### Configuration

Navigate to: **Policy & Objects** → **DoS Policy** → **Create New**

| Field | Value |
|-------|-------|
| Name | `Dos-protection-policy` |
| Incoming Interface | `port1` |
| Source Address | `all` |
| Destination Address | `all` |
| Service | `ALL` |

### L3 Anomaly Detection Rules

| Anomaly | Logging | Action | Threshold | Description |
|---------|---------|--------|-----------|-------------|
| `ip_src_session` | ✅ | **Block** | 5000 | Too many sessions from one source IP |
| `ip_dst_session` | ✅ | **Block** | 5000 | Too many sessions to one destination IP |

### L4 Anomaly Detection Rules

| Anomaly | Logging | Action | Threshold | Description |
|---------|---------|--------|-----------|-------------|
| `tcp_syn_flood` | ✅ | **Block** | 2000 | SYN packets per second |
| `tcp_port_scan` | ✅ | **Block** | 1000 | Port scan attempts per second |
| `tcp_src_session` | — | Disable | 5000 | TCP sessions per source |
| `tcp_dst_session` | — | Disable | 5000 | TCP sessions per destination |
| `udp_flood` | ✅ | **Block** | 2000 | UDP packets per second |
| `udp_scan` | ✅ | **Block** | 2000 | UDP scan attempts per second |
| `udp_src_session` | — | Disable | 5000 | UDP sessions per source |
| `udp_dst_session` | — | Disable | 5000 | UDP sessions per destination |
| `icmp_flood` | ✅ | **Block** | 250 | ICMP packets per second |
| `icmp_sweep` | — | Disable | 100 | ICMP ping sweep |
| `icmp_src_session` | — | Disable | 300 | ICMP sessions per source |
| `icmp_dst_session` | — | Disable | 1000 | ICMP sessions per destination |
| `sctp_flood` | — | Disable | 2000 | SCTP flood packets per second |
| `sctp_scan` | — | Disable | 1000 | SCTP port scan |
| `sctp_src_session` | — | Disable | 5000 | SCTP sessions per source |
| `sctp_dst_session` | — | Disable | 5000 | SCTP sessions per destination |

### DoS Test — SYN Flood Simulation

```bash
# Executed on Kali Linux
sudo hping3 -c 15000 -d 120 -S -w 64 -p 80 --flood --rand-source 192.168.205.2

# Parameters:
# -c 15000      Send 15,000 packets
# -d 120        Data payload 120 bytes
# -S            Set SYN flag
# -w 64         Window size 64
# -p 80         Target port 80
# --flood       Send packets as fast as possible
# --rand-source Randomize source IP address
```

**Result:**
```
960,735 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```
✅ All flood packets blocked by FortiGate DoS policy.

---

## C) Web Filter — Website Blocking

### Step 1: Enable Web Filter Feature

Navigate to: **System** → **Feature Visibility**

Toggle **Web Filter** to **Enabled** → Click **Apply**

### Step 2: Configure URL Filter in Default Profile

Navigate to: **Security Profiles** → **Web Filter**

Select the **default** profile → **Edit**

Under **Static URL Filter**:
- Enable **URL Filter** toggle
- Click **Create New**

| Field | Value |
|-------|-------|
| URL | `facebook.com` |
| Type | **Wildcard** |
| Action | **Block** |
| Status | **Enable** |

> **Wildcard matching** ensures both `www.facebook.com`, `m.facebook.com`, and all subdomains are blocked.

### Step 3: Create Web Filter Firewall Policy

Navigate to: **Policy & Objects** → **Firewall Policy** → **Create New**

| Field | Value |
|-------|-------|
| Name | `No-Facebook-Internet-Access` |
| Incoming Interface | `port1` |
| Outgoing Interface | `port2` |
| Source | `all` |
| Destination | `all` |
| Service | `ALL` |
| Action | **ACCEPT** |
| NAT | ✅ Enabled (Use Outgoing Interface Address) |
| **Web Filter Profile** | ✅ `default` |
| Inspection Mode | Flow-based |
| Log Allowed Traffic | All Sessions |

### Step 4: Verify Policy Sequence

Navigate to: **Policy & Objects** → **Firewall Policy** → View by **Sequence**

Policy order must be:
```
#1  internet-access (1)          — port1 → port2, DNS/HTTP/HTTPS
#2  No-Facebook-Internet-Access  — port1 → port2, ALL + Web Filter
    [Implicit Deny]              — any → any, ALL, DENY
```

> ⚠️ **Critical:** The web filter policy must appear in the correct sequence relative to general internet access policy. Since the web filter policy covers ALL services and the specific URL filter profile handles the blocking logic within an ACCEPT policy, ordering here ensures web filter is evaluated.

---

## Policy Summary Table

| # | Policy Name | From | To | Action | Special |
|---|-------------|------|----|--------|---------|
| 1 | internet-access | port1 | port2 | ACCEPT + NAT | DNS, HTTP, HTTPS |
| 2 | No-Facebook-Internet-Access | port1 | port2 | ACCEPT + NAT | Web Filter: default |
| — | Implicit Deny | any | any | DENY | All traffic |

---

## Screenshots Reference

| Screenshot | Description |
|-----------|-------------|
| `task3/ss30_firewall_policy_list.png` | Existing policy list |
| `task3/ss31_new_policy_form.png` | New policy configuration |
| `task3/ss32_logging_options.png` | Logging settings |
| `task3/ss33_dos_policy_l3.png` | DoS policy L3 anomalies |
| `task3/ss34_dos_l3_rules.png` | ip_src/dst_session rules |
| `task3/ss35_dos_l4_rules.png` | TCP/UDP flood rules |
| `task3/ss36_dos_complete.png` | Complete L4 anomaly table |
| `task3/ss37_hping3_test.png` | SYN flood test on Kali |
| `task3/ss38_feature_visibility.png` | Web filter feature toggle |
| `task3/ss39_web_filter_profiles.png` | Web filter profile list |
| `task3/ss40_url_filter_create.png` | URL filter creation |
| `task3/ss41_facebook_block_rule.png` | facebook.com block rule |
| `task3/ss42_webfilter_policy.png` | Web filter policy creation |
| `task3/ss43_security_profiles.png` | Security profiles attached |
| `task3/ss44_policy_complete.png` | Completed policy settings |
| `task3/ss45_policy_sequence.png` | Policy by sequence view |

✅ **Task 3 Complete** — All three security policies implemented and active.
