# Splunk Home Lab Documentation

This documentation explains the setup, configuration, event generation, and log collection for a Splunk Home Lab using Oracle VirtualBox, Windows 11, and Kali Linux. It includes personal observations, troubleshooting steps, and lessons learned during the process.

---

## Step 1: Home Lab Setup

### Virtual Machines Created
- **Windows 11 VM** — Splunk Enterprise + Universal Forwarder  
- **Kali Linux VM** — Attacker machine (failed logins, sudo failures, SSH brute force, nmap scans)

**Narrative / Challenges:**
- Initially, both VMs received the same IP from DHCP, causing connectivity issues. Fixed by switching both VMs to **Bridged Adapter** mode.  
- Windows 11 VM was lagging and crashing due to limited resources. Fixed by increasing CPU, RAM, enabling **3D acceleration**, setting graphics controller to **VBoxSVGA**, and installing **VirtualBox Guest Additions** for better video and memory performance.  
- Kali Linux VM ran smoothly without issues.

---

### Windows 11 VM Settings

**Hardware**
- Base Memory: 4096 MB  
- Processors: 4  
- Boot Order: Floppy → Optical → Hard Disk  
- Acceleration: Nested Paging, PAE/NX, Hyper-V Paravirtualization  

**Display**
- 128 MB Video Memory  
- VBoxSVGA Graphics Controller  
- 3D Acceleration Enabled  

**Storage**
- 300 GB  

**Network**
- Bridged Adapter  

**Screenshot Placeholder:**  
```
![Windows VM Settings](screenshots/windows-vm-settings.png)
```

---

### Kali Linux VM Settings

**Hardware**
- Base Memory: 4096 MB  
- Processors: 2  
- Boot Order: Optical → Hard Disk  
- Acceleration: Nested Paging, PAE/NX, KVM Paravirtualization  

**Display**
- 128 MB Video Memory  
- VMSVGA Graphics Controller  

**Storage**
- 25 GB  

**Network**
- Bridged Adapter  

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
- Enabled Security Event Logs in Windows  
- Configured log forwarding to Splunk Enterprise  

**Narrative / Notes:**
- Initially had trouble with indexing, forwarder setup, and connecting Splunk Universal Forwarder. Solved by confirming forwarder status, checking ports, and verifying hostnames.  
- Logs could be uploaded directly via Splunk Web UI as `.evtx` files or forwarded via Universal Forwarder for near real-time ingestion.

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
- **4625** — Failed login  
- **4624** — Successful login  

Logs exported as:
- `Security.evtx`  
- `Security.xml` (for readability)  

**Observations / Notes:**
- Confirmed event IDs appeared in Event Viewer before exporting.  
- These logs were later ingested into Splunk to validate dashboard displays.

**Screenshot Placeholders:**
```
![Runas Failed Logon](screenshots/runas-failed-logon.png)
![Event Viewer 4625](screenshots/event-4625.png)
```

---

## Step 4: Generate Kali Linux Security Events

### 4.1 Monitoring Linux Authentication Logs

Commands:

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

Enter wrong password multiple times.

Check logs:

```
sudo journalctl -u sudo -f
```

**Observations / Notes:**
- Failed sudo attempts generated authentication failure logs, which can be used to simulate privilege abuse detection in Splunk.

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
- Invalid user logs  
- Failed password logs  
- Authentication failure logs  

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

Generates Windows firewall events. Check Event Viewer for:
- **5152**  
- **5156**  
- **5985**  
- **26001**  
- Firewall blocks  

**Observations / Notes:**
- Nmap SYN scans trigger firewall alerts on Windows.  
- Useful to simulate network reconnaissance detection in Splunk.

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

- Upload Windows `.evtx` files and Linux `.txt` logs into Splunk.  
- Confirm indexing using SPL queries:  

```
index=home_lab host="<hostname>" | stats count by sourcetype
index=home_lab sourcetype=WinEventLog:Security | stats count by EventCode
```

**Observations / Notes:**
- Initially had issues with WinEventLog formatting, but exporting as XML solved it.  
- Linux logs indexed successfully after saving as `.txt`.

**Screenshot Placeholder:**
```
![Splunk Data Upload](screenshots/splunk-upload.png)
```

---

## Step 7: Next Steps & Improvements

- Build dashboards for:
  - Failed logins  
  - Sudo abuse  
  - SSH brute-force attempts  
  - Recon / nmap detection  
- Create alerts for suspicious activity  
- Install **Sysmon** on Windows for advanced logging  
- Add more attacker scenarios for richer dataset  
- Automate log generation scripts for continuous testing  

---

## Screenshot Summary (Placeholders for Later)

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
