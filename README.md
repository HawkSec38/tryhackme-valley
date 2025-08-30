
---

# CTF Write-Up: Valley

**Platform:** TryHackMe  
**Machine:** Valley  
**Difficulty:** Medium  
**Author:** [Goodluck Temilolu Oyebisi]  
**Date:** [30/8/2025]  

## Overview
Valley is a medium-difficulty CTF machine that involves web exploitation, forensics analysis, hash cracking, and privilege escalation via Python library hijacking. The goal is to find two flags: `user.txt` and `root.txt`.

## Tools Used
- **Nmap** - Network enumeration
- **Gobuster** - Directory bruteforcing
- **cURL** - Web requests and testing
- **Wireshark/tshark** - PCAP analysis
- **Hashcat/Crackstation** - Hash cracking
- **SCP** - File transfer
- **Strings** - Binary analysis
- **Python3** - HTTP server and exploitation

## Step 1: Reconnaissance
### Port Scanning
```bash
nmap -A 10.10.99.141
```
**Open Ports:**
- `22/tcp` - OpenSSH 8.2p1 Ubuntu
- `80/tcp` - Apache httpd 2.4.41

### Web Enumeration
**Gobuster Scan:**
```bash
gobuster dir -u http://10.10.99.141 -w /usr/share/wordlists/dirb/common.txt
```
**Discovered Directories:**
- `/gallery` (301)
- `/pricing` (301)
- `/static` (301)

## Step 2: Web Exploitation
### Initial Findings
- The main page (`index.html`) contained links to `/gallery` and `/pricing`.
- The gallery page (`gallery.html`) displayed images sourced from `/static/1` to `/static/18`.

### Discovering Hidden Notes
A note was found at `/pricing/note.txt`:
```
J,
Please stop leaving notes randomly on the website
-RP
```

### SSRF and Developer Login
- SSRF was tested via `<iframe>` tags in Markdown, leading to the discovery of a hidden developer login page at `/dev1243224123123/`.
- The login page contained a form and linked JavaScript files (`dev.js` and `button.js`).

### Analyzing JavaScript
The `dev.js` file contained hardcoded credentials:
- Username: `siemDev`
- Password: `california`
- Redirect URL on success: `/dev1243224123123/devNotes37370.txt`

### Accessing Developer Notes
The `devNotes37370.txt` file revealed:
```
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

## Step 3: FTP Access
### Port Discovery
A full port scan revealed FTP running on port `37370`.

### FTP Login
Credentials `siemDev:california` were reused to log in:
```bash
ftp 10.10.99.141 37370
```

### Retrieving PCAP Files
Three PCAP files were downloaded:
- `siemFTP.pcapng`
- `siemHTTP1.pcapng`
- `siemHTTP2.pcapng`

## Step 4: Forensics Analysis
### Analyzing PCAPs
- **siemHTTP2.pcapng** contained an HTTP POST request with credentials:
  - Username: `valleyDev`
  - Password: `ph0t0s1234`

### SSH Access
These credentials allowed SSH access:
```bash
ssh valleyDev@10.10.99.141
```

## Step 5: Privilege Escalation to Valley
### Binary Analysis
A binary (`/home/valleyAuthenticator`) was found in `/home`. Analysis with `strings` revealed a hash:
```
e6722920bab2326f8217e4
```

### Hash Cracking
The hash was cracked to `liberty123` using Crackstation.

### User Switching
The password `liberty123` allowed switching to the `valley` user:
```bash
su valley
Password: liberty123
```

## Step 6: Privilege Escalation to Root
### Python Module Hijacking
The user `valley` was a member of the `valleyAdmin` group, which had write permissions to `/usr/lib/python3.8/base64.py`.

### Malicious Code Injection
The file was edited to include:
```python
import os; os.system("chmod u+s /bin/bash")
```

### Cron Job Exploitation
A root cron job was found:
```
1 * * * * root python3 /photos/script/photosEncrypt.py
```
This job executed every minute and imported the `base64` module, triggering the malicious code.

### Root Shell
After the cron job ran, the SUID bit was set on `/bin/bash`:
```bash
/bin/bash -p
```

## Flags
### User Flag
**Location:** `/home/valley/user.txt`  
**Flag:** `THM{...}`

### Root Flag
**Location:** `/root/root.txt`  
**Flag:** `THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}`

## Conclusion
The Valley machine involved multiple techniques:
- **Web Enumeration** leading to hidden notes
- **SSRF** for internal service discovery
- **PCAP Analysis** for credential harvesting
- **Hash Cracking** for privilege escalation
- **Python Library Hijacking** for root access

This challenge highlighted the importance of secure credential management and the risks of reusable passwords.

---
