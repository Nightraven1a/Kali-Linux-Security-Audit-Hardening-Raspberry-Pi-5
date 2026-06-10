# 🔒 Kali Linux Security Audit & Hardening — Raspberry Pi 5

![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%205-red)
![OS](https://img.shields.io/badge/OS-Kali%20Linux-blue)
![Status](https://img.shields.io/badge/Status-Hardened-brightgreen)
![Rootkits](https://img.shields.io/badge/Rootkits-0%20Found-brightgreen)

A full security audit and system hardening performed on a Kali Linux installation running on a Raspberry Pi 5. This project covers threat detection, service hardening, firewall configuration, and automated intrusion prevention.

---

## 🎯 Objective

Perform a comprehensive security audit to detect unauthorized access, malware, or misconfigurations — then remediate every identified vulnerability and establish a clean, verified security baseline.

---

## 🖥️ System Environment

| Component | Details |
|-----------|---------|
| Hardware | Raspberry Pi 5 |
| OS | Kali Linux (kali-rolling) |
| Architecture | ARM64 (aarch64) |
| Kernel | Linux |
| User | Non-root user (`pi`) with sudo access |

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| `rkhunter 1.4.6` | Rootkit, backdoor, and malware detection |
| `chkrootkit` | Secondary rootkit scanner |
| `debsums` | Verify integrity of installed Debian packages |
| `ufw` | Firewall configuration and management |
| `fail2ban` | Automated SSH brute-force protection |
| `ss` / `netstat` | Network socket and port analysis |
| `systemctl` | Service management and hardening |

---

## 🔍 Audit Findings

### Network Exposure (Pre-Hardening)

| Port | Protocol | Service | Risk | Action Taken |
|------|----------|---------|------|-------------|
| 80 | TCP | Apache2 | Medium — exposed to internet | Disabled |
| 500 | UDP | StrongSwan IPSec | Low — unnecessary service | Disabled |
| 4500 | UDP | StrongSwan IPSec | Low — unnecessary service | Disabled |
| 22 | TCP | SSH | Medium — password auth enabled | Hardened |
| 3306 | TCP | MySQL | Low — localhost only | No action needed |
| 53 | TCP/UDP | DNS (libvirt) | Low — internal only | No action needed |

### Rootkit / Malware Scan Results

| Scanner | Rootkits Checked | Result |
|---------|-----------------|--------|
| rkhunter | 500 signatures | ✅ None found |
| chkrootkit | All modules | ✅ Not infected |

### Authentication & Login Audit

| Check | Result |
|-------|--------|
| Unauthorized SSH logins | ✅ None |
| Failed login attempts | ✅ None |
| Root equivalent accounts | ✅ None found |
| Passwordless accounts | ✅ None found |
| Active login sessions | Only local user `pi` |

### PAM Integrity Check

```
sudo debsums -c libpam-runtime
# Output: (empty — all files verified clean)
```

All PAM authentication files verified against Debian package checksums. No tampering detected.

---

## 🛡️ Remediation Steps

### 1. Disabled Apache2 (Port 80)
Apache was running and listening on all interfaces with no active use case.

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

### 2. Disabled StrongSwan IPSec (Ports 500 / 4500)
StrongSwan was installed as a Kali default but not needed. Removed unnecessary attack surface.

```bash
sudo systemctl stop strongswan-starter
sudo systemctl disable strongswan-starter
```

### 3. Hardened SSH Configuration
Disabled password-based authentication to prevent brute-force attacks. Key-based auth only.

```bash
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

**Verified:**
```bash
sudo grep "PasswordAuthentication" /etc/ssh/sshd_config
# Output: PasswordAuthentication no
```

### 4. Deployed UFW Firewall
Configured default-deny inbound policy, allowing only SSH.

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
```

**Result:**
```
Status: active
To                    Action      From
--                    ------      ----
22/tcp                ALLOW       Anywhere
22/tcp (v6)           ALLOW       Anywhere (v6)
```

### 5. Configured Fail2Ban (SSH Brute-Force Protection)
Installed and enabled fail2ban to automatically ban IPs with repeated failed SSH attempts.

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**Verified:**
```bash
sudo fail2ban-client status sshd
# Jail: active | Failed: 0 | Banned: 0
```

### 6. Silenced Known False Positives in rkhunter
Whitelisted `lwp-request` (a known Kali false positive) in `/etc/rkhunter.conf`:

```
SCRIPTWHITELIST=/usr/bin/lwp-request
```

### 7. Updated rkhunter Baseline
After all changes, updated the file property database to establish a clean baseline.

```bash
sudo rkhunter --propupd
# Output: File updated: searched for 182 files, found 143
```

---

## ✅ Post-Hardening Scan Results

```
System checks summary
=====================
File properties checks...
    Files checked: 143
    Suspect files: 0          ← Down from 1

Rootkit checks...
    Rootkits checked : 500
    Possible rootkits: 2      ← Known false positives (legit Debian packages)

Applications checks...
    All checks skipped
```

All remaining warnings traced to legitimate Debian packages (llvm, openjdk, dbeaver) — confirmed false positives, not threats.

---

## 🔄 Ongoing Maintenance

Run weekly:

```bash
# Update system packages
sudo apt update && sudo apt upgrade

# Run rootkit scan
sudo rkhunter --check

# Check fail2ban status
sudo fail2ban-client status sshd
```

---

## 📋 Final Security Posture

| Layer | Tool | Status |
|-------|------|--------|
| Firewall | ufw (default deny) | ✅ Active |
| Brute-force protection | fail2ban | ✅ Active |
| Rootkit detection | rkhunter + chkrootkit | ✅ Clean |
| SSH | Password auth disabled | ✅ Hardened |
| Unnecessary services | Apache2 + StrongSwan | ✅ Disabled |
| Package integrity | debsums | ✅ Verified |
| System updates | apt | ✅ Current |
| Scan baseline | rkhunter propupd | ✅ Updated |

---

## 👤 Author

**Johnathan Royster**  
IT Professional | Google IT Support | Google IT Automation with Python | Google Cybersecurity  
GitHub: [github.com/Nightraven1a](https://github.com/Nightraven1a)

---

> *"Security is a process, not a product."* — Bruce Schneier
