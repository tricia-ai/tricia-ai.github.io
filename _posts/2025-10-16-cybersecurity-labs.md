---
title: "Cybersecurity Labs!"
categories:
  - blog
tags:
  - Labs
---

Completed Labs on TryHackme and HackTheBox to understand various aspects of cybersecurity.

🔗 [View Repository](https://github.com/tricia-ai/cybersecurity_labs.git)

# Labs & TryHackMe Reports

Below are two detailed write-ups from TryHackMe labs: **RootMe** and **Basic Pentesting: Exploiting SMB & SSH**.  

---

# RootMe — TryHackMe

### 🧠 Problem Statement
Gain initial access to a vulnerable web server and escalate privileges to root. The lab exercises web exploitation (file upload / reverse shell) and Linux privilege escalation using SUID binaries.

---

### ⚙️ Approach

1. **Reconnaissance**
   - Performed an Nmap scan to identify open ports and services.

2. **Exploitation**
   - Prepared a PHP reverse shell, renamed the file to bypass filters (e.g. `.phtml`) and uploaded it to the `/panel/` upload functionality.
   - Set up a Netcat listener locally:
     ```bash
     nc -lvnp 1234
     ```
   - Triggered the uploaded reverse shell, resulting in a shell as `www-data`.

3. **Privilege Escalation**
   - Enumerated SUID binaries:
     ```bash
     find / -perm -u=s -type f 2>/dev/null
     ```
   - Noted `/usr/bin/python` had the SUID bit set.
   - Used a GTFOBins technique to spawn a root shell:
     ```bash
     python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
     ```
   - Retrieved `user.txt` and `root.txt` flags.

---

### 🧰 Tools Used
- **Nmap** — Port & service enumeration  
- **Dirb** — Directory enumeration  
- **Netcat (nc)** — Reverse shell listener  
- **GTFOBins** — Privilege escalation reference for SUID binaries  
- **Kali Linux** — Attack environment

---

### 🧩 Key Lessons Learned
- Directory enumeration is essential — uploads and hidden directories reveal real attack vectors.  
- Upload filters can often be bypassed with alternative extensions or small tweaks.  
- SUID binaries are high-value targets for local privilege escalation.  
- The end-to-end flow from discovery → exploitation → escalation is critical to practice and document.

---

# Basic Pentesting: Exploiting SMB & SSH — TryHackMe

### 🧠 Problem Statement
Enumerate and exploit SMB and SSH weaknesses to gain access. The lab covers SMB share enumeration, discovery of user artifacts, brute-force SSH, and cracking encrypted private keys to escalate privileges.

---

### ⚙️ Approach

1. **Reconnaissance**
   - Ran Nmap to discover open services (SSH, HTTP, SMB ports 139/445, proxy/HTTP on other ports).
     ```bash
     nmap -sV -sC 10.10.228.159
     ```
   - Ran `dirb` against web ports to find directories (e.g. `/development/`).

2. **SMB Enumeration**
   - Enumerated SMB shares and access:
     ```bash
     nmap -p 139,445 --script smb-enum-shares 10.10.228.159
     smbclient \\10.10.228.159\Anonymous
     ```
   - Downloaded `staff.txt` from the anonymous share; it referenced users (e.g., Jan, Kay).

3. **User & Key Discovery**
   - Used `enum4linux` to gather SMB user and group information:
     ```bash
     enum4linux -a 10.10.228.159
     ```
   - Located a private SSH key in a user directory (e.g., `/home/kay/.ssh/id_rsa`) that was encrypted.

4. **Cracking Private Key & SSH Access**
   - Converted the SSH private key for John the Ripper:
     ```bash
     /usr/share/john/ssh2john.py id_rsa > id_rsa.txt
     ```
   - Cracked the passphrase with John:
     ```bash
     sudo john id_rsa.txt --wordlist=/usr/share/wordlists/rockyou.txt
     ```
   - Used the cracked passphrase to SSH into the account:
     ```bash
     ssh -i id_rsa kay@<target-ip>
     ```
   - Retrieved sensitive files (e.g., `pass.bak`) and acquired further credentials.

---

### 🧰 Tools Used
- **Nmap** — Port & service scanning, SMB scripting  
- **Dirb** — Web directory enumeration  
- **SMBClient** — Accessing SMB shares  
- **Enum4Linux** — SMB user & share enumeration  
- **Hydra** — SSH bruteforce (where applicable)  
- **John the Ripper** — Convert and crack SSH private key passphrases  
- **Kali Linux** — Attack environment

---

### 🧩 Key Lessons Learned
- Misconfigured SMB shares with anonymous read/write can leak sensitive files and user hints.  
- Proper password hygiene is critical; weak or guessable passwords lead to full account compromise.  
- Encrypted private keys still present a risk if the passphrase is weak — cracking workflows (ssh2john → John) are effective.  
- Comprehensive enumeration (Nmap, enum4linux, smbclient) yields the critical clues needed to progress.
