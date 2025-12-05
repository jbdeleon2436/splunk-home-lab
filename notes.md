Splunk Home Lab – Notes
1. Overview of What I Did

Set up a Splunk home lab using Oracle VirtualBox.

Created two virtual machines:

Windows 11 VM (Splunk Enterprise + log generator)

Kali Linux VM (attacker simulation: SSH attacks, failed sudo, Nmap scans)

Installed Splunk Enterprise on Windows.

Configured and installed the Splunk Universal Forwarder on Windows to send logs.

Simulated multiple Windows and Linux security events for SIEM analysis.

📸 Screenshot Placeholder:
Screenshot of the overall lab setup diagram
![Lab Overview Diagram](screenshots/lab-overview.png)

2. Initial Setup & Problems Encountered
IP Conflict Issue

Both VMs initially received the same IP from DHCP.
Fix: Changed their network adapters to Bridged Mode.

📸 Screenshot Placeholder:
Screenshot of VirtualBox Network Adapter set to “Bridged Adapter”
![Bridged Adapter](screenshots/vbox-bridged.png)

Windows 11 Performance Issues

Windows VM kept lagging or freezing.

Fixes applied:

Increased CPU and RAM allocation

Enabled 3D Acceleration

Switched Graphics Controller → VBoxSVGA

Installed VirtualBox Guest Additions

📸 Screenshot Placeholder:
Screenshot of VirtualBox Display settings for Windows VM
![Windows Display Settings](screenshots/windows-display-settings.png)

3. Virtual Machine Configuration
Kali Linux VM

RAM: 4096 MB

CPU: 2

Boot Order: Optical, Hard Disk

Acceleration: Nested Paging, PAE/NX, KVM

Display: 128 MB, VMSVGA

Storage: 25 GB

Network: Bridged

📸 Screenshot Placeholder:
![Kali VM Settings](screenshots/kali-settings.png)

Windows 11 VM

RAM: 4096 MB

CPU: 4

Boot Order: Floppy, Optical, Hard Disk

Acceleration: Nested Paging, PAE/NX, Hyper-V

Display: 128 MB, VBoxSVGA

Storage: 300 GB

Network: Bridged

📸 Screenshot Placeholder:
![Windows VM Settings](screenshots/windows-settings.png)

4. Windows Event Generation
Failed Login Attempts

To generate Event ID 4625 (failed logon):

runas /user:wronguser cmd


📸 Screenshot Placeholder:
![Failed Login Attempt CMD](screenshots/windows-failed-login-cmd.png)

Checking Windows Security Logs

Event Viewer → Windows Logs → Security
Check for:

4625 – Failed logon

4624 – Successful logon

📸 Screenshot Placeholder:
![Event Viewer Security Logs](screenshots/event-viewer-security.png)

Exporting Windows Logs

Saved logs as:

Security.evtx

.xml format (better display in Splunk)

📸 Screenshot Placeholder:
![Exported Security Logs](screenshots/windows-export-logs.png)

5. Kali Linux Log Generation
Viewing Authentication Logs

Commands:

sudo journalctl -u ssh
sudo journalctl -f
sudo journalctl -xe | grep auth
sudo journalctl -u ssh -f


📸 Screenshot Placeholder:
![Kali Journalctl Output](screenshots/kali-auth-logs.png)

Triggering Failed Sudo Attempts
sudo su


Enter wrong password multiple times.

Check logs:

sudo journalctl -u sudo -f


📸 Screenshot Placeholder:
![Failed Sudo Attempts](screenshots/kali-sudo-fail.png)

SSH Attack Simulation

Start SSH service:

sudo systemctl start ssh


From Windows:

ssh root@<kali-ip>


Enter wrong passwords → generates failed login attempts.

Monitor live logs:

sudo journalctl -u ssh -f


📸 Screenshot Placeholder:
![SSH Failed Attempts](screenshots/ssh-failed.png)

Nmap Reconnaissance Scan

From Kali → Windows:

nmap -sS <Windows-IP>


Generates logs on Windows, including:

Event ID 5152, 5156, 5985, 26001, and other firewall blocks

📸 Screenshot Placeholder:
![Nmap Scan Output](screenshots/nmap-scan.png)

6. Exporting Logs for Splunk
Exporting Linux Logs
sudo journalctl --unit=ssh > ssh_logs.txt
sudo journalctl | grep sudo > sudo_logs.txt


📸 Screenshot Placeholder:
![Linux Logs Export](screenshots/kali-export-logs.png)

Exporting Windows Logs

Already exported as:

.evtx

.xml

📸 Screenshot Placeholder:
![Windows EVTX Export](screenshots/windows-evtx-export.png)

7. Next Steps

To expand the project:

Build Splunk Dashboards

Authentication failures

SSH attempts

Nmap recon events

Windows firewall activity

📸 Screenshot Placeholder:
![Splunk Dashboard](screenshots/splunk-dashboard.png)

Create Alerts

Multiple failed logins

Sudden Nmap scan

Repeated sudo failures

Install Sysmon

Adds richer logs (process creation, network connections, registry changes).

📸 Screenshot Placeholder:
![Sysmon Installed](screenshots/sysmon-installed.png)

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
