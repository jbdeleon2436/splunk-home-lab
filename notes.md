# Splunk Home Lab Documentation

This documentation explains the setup, configuration, event generation, and log collection for a Splunk Home Lab built using Oracle VirtualBox, Windows 11, and Kali Linux.

---

## Step 1: Home Lab Setup

### Virtual Machines Created
- **Windows 11 VM** — Splunk Enterprise + Universal Forwarder  
- **Kali Linux VM** — Attacker machine (failed logins, sudo failures, SSH brute force, nmap scans)

### Platform
- **Oracle VirtualBox**

### Objective
Simulate security-related events on both systems and ingest the logs into Splunk for analysis.

---

### VM Networking
Initially both VMs received the **same DHCP IP**, causing conflicts.  
**Fix:** Switched both VMs to **Bridged Adapter** mode.

---

### Windows 11 VM Configuration

Windows initially lagged; performance was improved by adjusting VM resource allocation and enabling graphics optimizations.

**Settings Applied:**
- Base Memory: **4096 MB**
- Processors: **4**
- Boot Order: Floppy → Optical → Hard Disk
- Acceleration: Nested Paging, PAE/NX, Hyper-V Paravirtualization
- **Display:**  
  - 128 MB Video Memory  
  - VBoxSVGA Graphics Controller  
  - 3D Acceleration Enabled
- **Storage:** 300 GB
- **Network:** Bridged Adapter
- **Guest Additions Installed** for better performance

**Screenshot Placeholder:**  
```
![Windows VM Settings](screenshots/windows-vm-settings.png)
```

---

### Kali Linux VM Configuration

Kali Linux operated smoothly with no configuration issues.

**Settings Applied:**
- Base Memory: **4096 MB**
- Processors: **2**
- Boot Order: Optical → Hard Disk
- Acceleration: Nested Paging, PAE/NX, KVM Paravirtualization
- **Display:**  
  - 128 MB Video Memory  
  - VMSVGA Graphics Controller
- **Storage:** 25 GB
- **Network:** Bridged Adapter

**Screenshot Placeholder:**  
```
![Kali VM Settings](screenshots/kali-vm-settings.png)
```

---

## Step 2: Install and Configure Splunk

### Splunk on Windows 11
Actions:
- Installed **Splunk Enterprise**
- Installed **Splunk Universal Forwarder**
- Enabled Windows Security logs for collection
- Configured forwarding to Splunk Enterprise
- Confirmed ingestion via Splunk Web

**Screenshot Placeholder:**
```
![Splunk Login](screenshots/splunk-login.png)
```

---

## Step 3: Generate Windows Security Events

### 3.1 Failed Login Attempts  
Trigger failed logons:

```
runas /user:wronguser cmd
```

Generated Windows Event IDs:
- **4625** — Failed logon
- **4624** — Successful logon

Logs exported as:
- `Security.evtx`
- `Security.xml`

**Screenshot Placeholders:**
```
![Runas Failed Logon](screenshots/runas-failed-logon.png)
![Event Viewer 4625](screenshots/event-4625.png)
```

---

## Step 4: Generate Kali Linux Security Events

### 4.1 View Linux Authentication Logs

Commands used:

```
sudo journalctl -u ssh
sudo journalctl -f
sudo journalctl -xe | grep auth
sudo journalctl -u ssh -f
```

---

### 4.2 Failed Sudo Attempts

Trigger events:

```
sudo su
```

Enter incorrect passwords multiple times.

View logs:

```
sudo journalctl -u sudo -f
```

**Screenshot Placeholder:**
```
![Kali Failed Sudo Logs](screenshots/kali-sudo-fail.png)
```

---

### 4.3 SSH Brute Force Attempts

Start SSH service:

```
sudo systemctl start ssh
```

From Windows VM:

```
ssh root@<kali-vm-ip>
```

Enter wrong passwords to generate:
- Failed password
- Invalid user
- Authentication failure

View in real time:

```
sudo journalctl -u ssh -f
```

**Screenshot Placeholder:**
```
![Kali SSH Logs](screenshots/kali-ssh-logs.png)
```

---

### 4.4 Nmap Reconnaissance Scan

From Kali:

```
nmap -sS <Windows-IP>
```

This generates Windows firewall events.

Check Windows Event Viewer for:
- 5152  
- 5156  
- 5985  
- 26001  
- Firewall blocks  

**Screenshot Placeholder:**
```
![Windows Firewall Events](screenshots/windows-firewall-events.png)
```

---

## Step 5: Export Linux Logs for Splunk

SSH logs:

```
sudo journalctl --unit=ssh > ssh_logs.txt
```

Sudo logs:

```
sudo journalctl | grep sudo > sudo_logs.txt
```

Saved in:
- `logs/linux/ssh_logs.txt`
- `logs/linux/sudo_logs.txt`

**Screenshot Placeholder:**
```
![Linux Logs Exported](screenshots/linux-logs-export.png)
```

---

## Step 6: Import Logs into Splunk

- Uploaded Windows `.evtx` files
- Uploaded Linux `.txt` logs
- Verified indexing in Splunk

**Screenshot Placeholder:**
```
![Splunk Data Upload](screenshots/splunk-upload.png)
```

---

## Step 7: Next Steps (Future Work)

- Build custom dashboards  
- Create detection alerts  
- Install **Sysmon** on Windows for advanced logging  
- Add additional attacker scenarios  
- Automate log generation scripts  
- Create Splunk correlations for:
  - SSH brute-force  
  - Sudo abuse  
  - Nmap scan detection  

---

## Screenshot Summary (Fill These In Later)

```
![Windows VM Settings](screenshots/windows-vm-settings.png)
![Kali VM Settings](screenshots/kali-vm-settings.png)
![Splunk Login](screenshots/splunk-login.png)
![Runas Failed Logon](screenshots/runas-failed-logon.png)
![Event Viewer 4625](screenshots/event-4625.png)
![Kali Failed Sudo Logs](screenshots/kali-sudo-fail.png)
![Kali SSH Logs](screenshots/kali-ssh-logs.png)
![Windows Firewall Events](screenshots/windows-firewall-events.png)
![Linux Logs Exported](screenshots/linux-logs-export.png)
![Splunk Data Upload](screenshots/splunk-upload.png)
```

---


================================================

## Step 1: Install Splunk Enterprise
- OS: Windows 11 VM
- Downloaded Splunk Enterprise from official website
- Installed with default settings
- Accessed web UI at http://localhost:8000
- Created admin account

![Splunk Login](screenshots/splunk-login.png)

## Step 2: Windows Security Logs
Actions:
- Generated failed logins using lock-screen and runas.
- Observed Event IDs: 4625 (failed), 4624 (success).
- Security event logs saved as `Security.evtx`

Files saved:
- logs/windows/Security.evtx

![Runas Generate Failed Logons](screenshots/runas-failed-logon.png)
![Events Viewer 4625](screenshots/event-viewer-4625.png)

## Step 3 - Kali Linux Auth Logs
Actions:
- Started SSH service and generated failed SSH login attempts from Windows.
- Generated failed sudo attempts locally.
- Auth logs monitored: `/var/log/auth.log`
- Generated events:
  - `sudo` commands
  - nmap scan to Windows VM
- Logs saved in `/logs/linux/`

Files saved:
- logs/linux/ssh_logs.txt
- logs/linux/sudo_logs.txt

Commands used:
- sudo systemctl start ssh
- sudo journalctl -u ssh -f
- nmap -sS <windows-ip>

![](screenshots/kali-failed-sudo.png)
![](screenshots/windows-failed-ssh.png)

- Kali SSH Logs

![](screenshots/kali-ssh-logs.png)
