# 🔍 Wireshark Network Analysis Lab

![Wireshark](https://img.shields.io/badge/Wireshark-4.6-blue?style=flat-square)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-2025-red?style=flat-square&logo=kalilinux)
![Nmap](https://img.shields.io/badge/Nmap-7.95-green?style=flat-square)
![Security](https://img.shields.io/badge/Security-SOC%20Lab-orange?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

A hands-on network traffic analysis lab using Wireshark and Nmap against a deliberately vulnerable target (Metasploitable2) in an isolated VirtualBox environment. Demonstrates real SOC analyst workflows including port scan detection, vulnerability identification, and traffic analysis.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| **Attacker** | Kali Linux 2025 (VirtualBox VM) |
| **Target** | Metasploitable2 — 192.168.56.102 |
| **Tools** | Wireshark 4.6, Nmap 7.95 |
| **Network** | VirtualBox lab with host-only target network and NAT connectivity for the Kali VM |

### 🔧 Network Configuration Note

The Kali VM used two virtual interfaces during this lab:
- `eth0` — VirtualBox NAT network (`10.0.2.0/24`) for general VM connectivity
- `eth1` — Host-only lab network (`192.168.56.0/24`) for communication with Metasploitable2

Target-specific findings are based on Nmap service enumeration of `192.168.56.102`. Supplementary Wireshark screenshots were captured on the NAT-facing interface and are provided as examples of packet-capture workflow and background ARP/DNS activity during the lab session.

---

## 🔵 SOC Analyst Use Case

Wireshark packet analysis and vulnerability scanning are core SOC analyst skills. This lab replicates real-world analyst workflows — capturing traffic, identifying attack patterns, and documenting findings in a structured security report.

**Typical SOC network investigation workflow:**

1. Anomalous traffic detected in SIEM → analyst opens packet capture tool
2. Wireshark filter applied → isolate relevant protocol (DNS, ARP, TCP)
3. SYN packet pattern identified → port scan investigation opened
4. ARP table reviewed → check for MAC-to-IP inconsistencies (MITM indicator)
5. Nmap service scan run → identify exposed services and vulnerable versions
6. CVEs cross-referenced → assess exploitability and business impact
7. Findings documented → risk ratings assigned, remediation steps drafted
8. Report submitted → attached to incident ticket as evidence

**What this lab demonstrates:**

| Skill | SOC Relevance |
|---|---|
| Wireshark packet filtering | Daily analyst tool for traffic investigation |
| Port scan detection via SYN patterns | Recognizing reconnaissance activity |
| ARP analysis | Detecting Man-in-the-Middle attack setup |
| Service version detection | Identifying vulnerable software in environment |
| CVE documentation | Assessing and communicating risk to stakeholders |
| Remediation reporting | Core deliverable for security analysts |

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
sudo nmap -sn 192.168.56.0/24
```
**Result:** Identified live hosts on the host-only network including the target Metasploitable2 VM at 192.168.56.102.

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
| 6667/tcp | open | IRC | UnrealIRCd | 🔴 HIGH — unauthorized communication risk |
| 8009/tcp | open | AJP13 | Apache Jserv | 🟡 MEDIUM |
| 8180/tcp | open | HTTP | Apache Tomcat | 🔴 HIGH |

---

### Exercise 3 — Wireshark Traffic Analysis

> **Note:** Wireshark screenshots in this lab are supplementary examples of packet-capture workflow captured on the NAT-facing interface. Definitive target findings for `192.168.56.102` are derived from Nmap service detection output.

#### ARP Analysis
- ARP request/reply activity observed on the local VM network
- No ARP spoofing detected — MAC-to-IP mappings remained consistent throughout the session

#### Port Scan Detection
- Scan-related TCP SYN traffic was observed during the lab session
- Open TCP services identified by Nmap would normally respond with SYN/ACK packets
- Closed TCP ports typically respond with TCP RST packets
- DNS reverse lookups were generated during service enumeration

#### Wireshark Filters Used

| Filter | Purpose |
|---|---|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Isolate SYN packets — identifies port scan activity |
| `tcp.flags.reset == 1` | Show TCP RST packets — closed port responses |
| `arp` | ARP traffic — check for spoofing indicators |
| `dns` | DNS queries — detect tunneling or suspicious domains |
| `ip.addr == 192.168.56.102` | Filter all traffic to/from target |

#### Additional Observations
- ICMPv6 multicast traffic observed as part of normal local network discovery behavior
- DNS PTR lookups were generated during Nmap service enumeration
- Multiple TCP SYN packets to destination ports formed a classic port scan signature

---

## 🚨 Critical Findings

### Finding 1 — vsftpd 2.3.4 Backdoor (CVE-2011-2523)
**Severity: CRITICAL**
**Observed:** FTP service identified as vsftpd 2.3.4 on port 21.
**Risk:** This version is historically associated with CVE-2011-2523 — a backdoor introduced when the vsftpd download server was compromised. Sending a smiley face character in the username triggers a root shell on port 6200.
**Validation status:** Version-based identification only — exploit was not executed in this lab.

### Finding 2 — Metasploitable Root Shell (Port 1524)
**Severity: CRITICAL**
Nmap identified port 1524 as the Metasploitable root shell backdoor service. In Metasploitable2, this service is intended to provide unauthenticated root shell access. Exploit validation was not performed in this lab.

### Finding 3 — Unencrypted Remote Access Protocols
**Severity: CRITICAL**
Ports 512, 513, 514 (rexec, rlogin, rsh) transmit all data including credentials in plaintext. These protocols were deprecated decades ago and should never be exposed.

### Finding 4 — Network-Accessible Database Services
**Severity: HIGH**
MySQL (3306) and PostgreSQL (5432) were reachable from the scanning host. Exposing database services increases attack surface and may allow unauthorized access if weak credentials or legacy configurations exist. Database services should be restricted to trusted hosts only.

### Finding 5 — Telnet Enabled (Port 23)
**Severity: HIGH**
Telnet transmits all data including usernames and passwords in plaintext. Any network observer can capture credentials in Wireshark.

---

## 🛡️ Recommendations

1. Immediately disable vsftpd 2.3.4 and upgrade to a patched version
2. Remove bindshell on port 1524
3. Disable rexec, rlogin, rsh — replace with SSH
4. Disable Telnet — replace with SSH
5. Restrict database ports 3306 and 5432 to trusted hosts or required application servers only
6. Update all services to current patched versions
7. Implement network segmentation to limit attack surface

---

## 📁 Repository Structure

```
Wireshark-Network-Analysis-Lab/
├── README.md                           # This file — full lab documentation
├── analysis-report.md                  # Detailed technical findings
├── findings-summary.md                 # Executive summary
├── screenshots/
│   ├── nmap-scan-results.webp          # Figure 1 — Nmap service enumeration against Metasploitable2
│   ├── nmap-wireshark-sidebyside.webp  # Figure 2 — Side-by-side lab workflow view
│   └── wireshark-arp-dns.webp          # Figure 3 — Supplementary ARP and DNS traffic on NAT interface
└── .gitignore
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

**Lovedip Singh** — U.S. Army Veteran | Cybersecurity Analyst | Security+ | Network+

- GitHub: [github.com/Lovedipsingh](https://github.com/Lovedipsingh)
- LinkedIn: [linkedin.com/in/lovedip-singh-76802a1a3](https://linkedin.com/in/lovedip-singh-76802a1a3)
- Email: lovedip590@outlook.com

---

## 📄 License

MIT License
