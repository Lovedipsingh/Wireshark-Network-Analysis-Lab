# 🔍 Wireshark Network Analysis Lab

![Wireshark](https://img.shields.io/badge/Wireshark-4.6-blue?style=flat-square&logo=wireshark)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-2025-red?style=flat-square&logo=kalilinux)
![Nmap](https://img.shields.io/badge/Nmap-7.95-green?style=flat-square)
![Security](https://img.shields.io/badge/Security-SOC%20Lab-orange?style=flat-square)

A hands-on network traffic analysis lab using Wireshark and Nmap against a deliberately vulnerable target (Metasploitable2) in an isolated VirtualBox environment. Demonstrates real SOC analyst workflows including port scan detection, vulnerability identification, and traffic analysis.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| **Attacker** | Kali Linux 2025 (VirtualBox VM) |
| **Target** | Metasploitable2 — 192.168.56.102 |
| **Tools** | Wireshark 4.6, Nmap 7.95 |
| **Network** | Isolated VirtualBox host-only network |

---

## 🎯 Objectives

- Capture and analyze real network traffic using Wireshark
- Perform service version detection with Nmap
- Identify open ports and vulnerable services
- Detect port scan patterns in packet captures
- Document findings in a professional security report

---

## 📋 Lab Exercises

### Exercise 1 — Network Discovery
**Command used:**
```bash
sudo nmap -sn 10.0.2.0/24
```
**Result:** Identified 3 live hosts on the network including the target Metasploitable2 VM.

---

### Exercise 2 — Service Version Detection
**Command used:**
```bash
sudo nmap -sV 192.168.56.102
```

**Open Ports & Services Discovered:**

| Port | State | Service | Version | Risk |
|---|---|---|---|---|
| 21/tcp | open | FTP | vsftpd 2.3.4 | 🔴 CRITICAL — backdoor vulnerability |
| 22/tcp | open | SSH | OpenSSH 4.7p1 | 🟡 MEDIUM |
| 23/tcp | open | Telnet | Linux telnetd | 🔴 HIGH — unencrypted |
| 25/tcp | open | SMTP | Postfix smtpd | 🟡 MEDIUM |
| 53/tcp | open | DNS | ISC BIND 9.4.2 | 🟡 MEDIUM |
| 80/tcp | open | HTTP | Apache 2.2.8 | 🔴 HIGH — outdated |
| 139/tcp | open | NetBIOS | Samba 3.X-4.X | 🔴 HIGH — vulnerable |
| 445/tcp | open | SMB | Samba 3.X-4.X | 🔴 HIGH — vulnerable |
| 512/tcp | open | exec | netkit-rsh rexecd | 🔴 CRITICAL |
| 513/tcp | open | login | rlogind | 🔴 CRITICAL |
| 514/tcp | open | shell | rshd | 🔴 CRITICAL |
| 1099/tcp | open | Java-RMI | GNU Classpath | 🔴 HIGH |
| 1524/tcp | open | bindshell | Metasploitable root shell | 🔴 CRITICAL — backdoor |
| 2049/tcp | open | NFS | 2-4 (RPC) | 🟡 MEDIUM |
| 3306/tcp | open | MySQL | 5.0.51a | 🔴 HIGH — exposed |
| 5432/tcp | open | PostgreSQL | 8.3.0-8.3.7 | 🟡 MEDIUM |
| 5900/tcp | open | VNC | Protocol 3.3 | 🔴 HIGH — exposed |
| 6000/tcp | open | X11 | access denied | 🟡 MEDIUM |
| 6667/tcp | open | IRC | UnrealIRCd | 🔴 HIGH — C2 channel |
| 8009/tcp | open | AJP13 | Apache Jserv | 🟡 MEDIUM |
| 8180/tcp | open | HTTP | Apache Tomcat | 🔴 HIGH |

---

### Exercise 3 — Wireshark Traffic Analysis

#### ARP Analysis
- No ARP spoofing detected
- Normal router/device MAC-to-IP mappings
- Consistent ARP replies — no IP claiming multiple MACs

#### Port Scan Detection
- Captured SYN packets from Nmap scan in real time
- Wireshark clearly shows sequential port probing pattern
- Multiple ICMP "Destination Unreachable" responses visible — closed ports responding

#### Protocol Analysis
- ICMPv6 multicast traffic — normal network discovery
- DNS reverse lookup queries during Nmap scan
- TCP SYN packets to multiple ports — classic port scan signature

---

## 🚨 Critical Findings

### Finding 1 — vsftpd 2.3.4 Backdoor (CVE-2011-2523)
**Severity: CRITICAL**
Port 21 is running vsftpd 2.3.4 which contains a backdoor introduced by an attacker who compromised the vsftpd download server. Sending a smiley face in the username triggers a root shell on port 6200.

### Finding 2 — Metasploitable Root Shell (Port 1524)
**Severity: CRITICAL**
Port 1524 is running a bindshell that provides immediate root access with no authentication required.

### Finding 3 — Unencrypted Remote Access Protocols
**Severity: CRITICAL**
Ports 512, 513, 514 (rexec, rlogin, rsh) transmit all data including credentials in plaintext. These protocols were deprecated decades ago and should never be exposed.

### Finding 4 — Exposed Database Services
**Severity: HIGH**
MySQL (3306) and PostgreSQL (5432) are exposed with no firewall protection. Default credentials may allow unauthorized database access.

### Finding 5 — Telnet Enabled (Port 23)
**Severity: HIGH**
Telnet transmits all data including usernames and passwords in plaintext. Any network observer can capture credentials in Wireshark.

---

## 🛡️ Recommendations

1. **Immediately disable** vsftpd 2.3.4 and upgrade to a patched version
2. **Remove** bindshell on port 1524
3. **Disable** rexec, rlogin, rsh — replace with SSH
4. **Disable** Telnet — replace with SSH
5. **Firewall** database ports 3306 and 5432 — restrict to localhost only
6. **Update** all services to current patched versions
7. **Implement** network segmentation to limit attack surface

---

## 📁 Repository Structure

```
Wireshark-Network-Analysis-Lab/
├── README.md                    # This file — full lab documentation
├── analysis-report.md           # Detailed technical findings
├── findings-summary.md          # Executive summary
├── screenshots/
│   ├── nmap-scan-results.png    # Full Nmap service scan output
│   ├── wireshark-capture.png    # Live packet capture during scan
│   ├── port-scan-detection.png  # SYN packets captured in Wireshark
│   ├── arp-analysis.png         # ARP traffic analysis
│   ├── dns-analysis.png         # DNS query analysis
│   └── https-traffic.png        # TLS/HTTPS traffic analysis
└── captures/
    └── kali-lab-capture.pcapng  # Raw packet capture file
```

---

## 🛠️ Tools Used

- **Wireshark 4.6** — packet capture and protocol analysis
- **Nmap 7.95** — network discovery and service detection
- **Kali Linux 2025** — attack platform
- **Metasploitable2** — intentionally vulnerable target
- **VirtualBox** — isolated lab environment

---

## ⚠️ Disclaimer

This lab was conducted in an **isolated VirtualBox environment** against a **deliberately vulnerable** target (Metasploitable2). All activities were performed for **educational purposes only**. Never perform these activities against systems you do not own or have explicit written permission to test.

---

## 👨‍💻 Author

**Lovedip Singh** — U.S. Army Veteran | Cybersecurity Analyst | Security+ | Network+ | CySA+

- GitHub: [github.com/Lovedipsingh](https://github.com/Lovedipsingh)
- LinkedIn: [linkedin.com/in/lovedip-singh-76802a1a3](https://linkedin.com/in/lovedip-singh-76802a1a3)
- Email: lovedip590@outlook.com

---

## 📄 License

MIT License — for educational use only.
