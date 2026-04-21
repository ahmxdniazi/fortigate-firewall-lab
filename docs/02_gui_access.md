# Task 2 — GUI Access Configuration

## Objective
Enable and verify secure HTTPS-based browser access to the FortiGate management GUI, and configure administrator credentials.

---

## Step 1 — Enable HTTPS GUI Access

HTTPS access was initially configured via CLI in Task 1 using:
```bash
set allowaccess http https telnet ssh ping
```

### Verification via GUI

Navigate to: **Network** → **Interfaces** → **port1** → **Edit**

Under **Administrative Access (IPv4)**, confirm the following are checked:

| Protocol | Status | Purpose |
|----------|--------|---------|
| ✅ HTTPS | Enabled | Secure browser-based management |
| ✅ HTTP | Enabled | Plaintext browser access |
| ✅ SSH | Enabled | Secure CLI access |
| ✅ PING | Enabled | Connectivity testing |
| ✅ TELNET | Enabled | Legacy CLI access |
| ☐ FMG-Access | Disabled | FortiManager integration |
| ☐ SNMP | Disabled | Network monitoring |
| ☐ FTM | Disabled | FortiToken Mobile |

> **Security Best Practice:** In production, disable HTTP, Telnet, and restrict management to a dedicated OOB interface.

---

## Step 2 — Configure Administrator Account

Navigate to: **System** → **Administrators**

Available options under **System** menu:
- Administrators
- Admin Profiles
- Firmware & Registration
- Settings
- HA (High Availability)
- SNMP
- Replacement Messages
- FortiGuard
- Feature Visibility

### Edit Admin Account

Click **Edit** on the `admin` account:

| Field | Value |
|-------|-------|
| Username | `admin` |
| Type | Local User |
| Two-Factor Authentication | Disabled (lab environment) |
| Restrict login to trusted hosts | Disabled (lab environment) |

> **Production Recommendation:** Enable Two-Factor Authentication and restrict login to specific management subnets using "Restrict login to trusted hosts."

### Change Password

Click **Change Password** and set a strong password meeting FortiGate policy:
- Minimum 12 characters
- At least 1 uppercase letter
- At least 1 lowercase letter
- At least 1 number
- At least 1 non-alphanumeric character

---

## Step 3 — Verify Login

1. Click the admin menu (top-right) → **Log Out**
2. Navigate to: `https://192.168.134.10/login`
3. Enter updated credentials
4. Confirm successful authentication to the FortiGate Dashboard

### Dashboard Information Verified

| Field | Value |
|-------|-------|
| Hostname | FGVMEVTXU0VWOE55 |
| Serial Number | FGVMEVTXU0VWOE55 |
| Firmware | v7.6.6 build3652 (Mature) |
| Mode | NAT |
| System Time | 2026/03/07 02:04:03 |
| Public IP | Unknown (NAT behind host) |
| Allocated vCPUs | 1/1 (100%) |
| Allocated RAM | 2 GiB / 2 GiB (97%) |

---

## Security Notes

```
⚠️  Lab vs Production Differences
──────────────────────────────────────────────────────────
Lab Environment:
  - HTTP and Telnet enabled for convenience
  - Single admin account with no 2FA
  - Management on shared network interface

Production Best Practices:
  - Disable HTTP and Telnet completely
  - Enable HTTPS only with valid certificate
  - Use dedicated out-of-band management interface
  - Enable Two-Factor Authentication (FortiToken)
  - Restrict admin access to specific source IPs
  - Create role-based admin profiles (read-only, auditor, etc.)
  - Implement admin login timeout (idle session)
```

---

## Screenshots Reference

| Screenshot | Description |
|-----------|-------------|
| `task2/ss24_port1_edit.png` | port1 edit interface dialog |
| `task2/ss25_admin_access.png` | Administrative Access checkboxes |
| `task2/ss26_system_menu.png` | System navigation menu |
| `task2/ss27_edit_admin.png` | Admin account edit dialog |
| `task2/ss28_login_page.png` | Login page after logout |
| `task2/ss29_dashboard.png` | Successful re-login to dashboard |

✅ **Task 2 Complete** — HTTPS GUI access confirmed and admin credentials updated.
