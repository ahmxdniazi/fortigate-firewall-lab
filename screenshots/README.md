# Screenshots

This folder contains all lab screenshots organized by task.

## Folder Structure

```
screenshots/
├── task1/    ← FortiGate VM Deployment (SS01 – SS23)
├── task2/    ← GUI Access Configuration (SS24 – SS29)
├── task3/    ← Policy Implementation (SS30 – SS45)
└── task4/    ← Demonstration & Results (SS46 – SS51)
```

## Naming Convention

Screenshots are named using the format:
```
ss{number}_{short_description}.png
```

Example: `ss01_fortinet_download.png`

## Task 1 — FortiGate VM Deployment

| File | Description |
|------|-------------|
| `task1/ss01_fortinet_download.png` | FortiGate VM download page on Fortinet support portal |
| `task1/ss02_extracted_files.png` | Extracted archive showing VM files |
| `task1/ss03_vnet_editor.png` | VMware Virtual Network Editor |
| `task1/ss04_admin_privileges.png` | Change Settings prompt for admin access |
| `task1/ss05_multiple_vnets.png` | Multiple host-only interfaces configured |
| `task1/ss06_ovf_import.png` | OVF import EULA dialog |
| `task1/ss07_vm_name.png` | VM naming and storage path |
| `task1/ss08_vm_settings.png` | VM settings panel showing hardware |
| `task1/ss09_nic_config.png` | Six NICs assigned to custom VMnets |
| `task1/ss10_cli_login.png` | Initial CLI login and password change |
| `task1/ss11_get_interface.png` | `get system interface` output |
| `task1/ss12_port1_config.png` | Port1 static IP CLI commands |
| `task1/ss13_show_interface.png` | `show system interface` confirmation |
| `task1/ss14_ping_management.png` | Successful ping to management IP |
| `task1/ss15_gui_login.png` | FortiGate web GUI login page |
| `task1/ss16_setup_wizard.png` | Initial setup wizard |
| `task1/ss17_interfaces_menu.png` | Network > Interfaces menu |
| `task1/ss18_wan_config.png` | WAN interface (port2) configuration |
| `task1/ss19_lan_config.png` | LAN interface (port3) configuration |
| `task1/ss20_dhcp_server.png` | DHCP server settings |
| `task1/ss21_static_routes.png` | Static routes table |
| `task1/ss22_default_route.png` | Default route creation form |
| `task1/ss23_internet_test.png` | Successful ping to 8.8.8.8 and google.com |

## Task 2 — GUI Access Configuration

| File | Description |
|------|-------------|
| `task2/ss24_port1_edit.png` | port1 Edit Interface dialog |
| `task2/ss25_admin_access.png` | Administrative Access checkboxes |
| `task2/ss26_system_menu.png` | System navigation menu |
| `task2/ss27_edit_admin.png` | Admin account edit dialog |
| `task2/ss28_login_page.png` | Login page after logout |
| `task2/ss29_dashboard.png` | Successful re-login to dashboard |

## Task 3 — Policy Implementation

| File | Description |
|------|-------------|
| `task3/ss30_firewall_policy_list.png` | Existing firewall policy list |
| `task3/ss31_new_policy_form.png` | New firewall policy form |
| `task3/ss32_logging_options.png` | Logging options section |
| `task3/ss33_dos_policy_form.png` | DoS policy creation form |
| `task3/ss34_dos_l3_rules.png` | L3 anomaly rules |
| `task3/ss35_dos_l4_rules.png` | L4 anomaly rules (TCP/UDP) |
| `task3/ss36_dos_complete.png` | Complete L4 anomaly table |
| `task3/ss37_hping3_test.png` | hping3 SYN flood test |
| `task3/ss38_feature_visibility.png` | Feature Visibility — Web Filter enabled |
| `task3/ss39_web_filter_profiles.png` | Web filter profile list |
| `task3/ss40_url_filter_create.png` | URL filter creation page |
| `task3/ss41_facebook_block_rule.png` | facebook.com block rule |
| `task3/ss42_webfilter_policy.png` | Web filter policy list |
| `task3/ss43_security_profiles.png` | New policy with security profiles |
| `task3/ss44_policy_complete.png` | Completed policy NAT + Web Filter settings |
| `task3/ss45_policy_sequence.png` | Policy sequence view |

## Task 4 — Demonstration

| File | Description |
|------|-------------|
| `task4/ss46_facebook_blocked_browser.png` | Browser blocked from facebook.com |
| `task4/ss47_traffic_log_denials.png` | Traffic log showing UTM denials |
| `task4/ss48_deny_policy_log.png` | Deny entries in traffic log |
| `task4/ss49_webfilter_log.png` | Web filter log — facebook blocked |
| `task4/ss50_dos_security_events.png` | Security events summary (98 events) |
| `task4/ss51_hping3_packet_loss.png` | hping3 result — 100% packet loss |

---

> **Note:** Add your actual lab screenshots to the corresponding subfolders following the naming convention above.
