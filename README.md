# 🔍 Wireshark Network Analysis Lab

![Wireshark](https://img.shields.io/badge/Wireshark-4.6-blue?style=flat-square)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-2025-red?style=flat-square&logo=kalilinux)
![Nmap](https://img.shields.io/badge/Nmap-7.95-green?style=flat-square)
![Security](https://img.shields.io/badge/Security-SOC%20Lab-orange?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

A hands-on network traffic analysis lab using Wireshark and Nmap against a deliberately vulnerable target (Metasploitable2) in an isolated VirtualBox environment. Demonstrates real SOC analyst workflows: port scan detection, service enumeration, vulnerability identification, and structured security reporting.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| **Attacker** | Kali Linux 2025 (VirtualBox VM) |
| **Target** | Metasploitable2 — 192.168.56.102 |
| **Tools** | Wireshark 4.6, Nmap 7.95 |
| **Network** | Isolated VirtualBox host-only network (192.168.56.0/24) |

**Network configuration:** The Kali VM used two interfaces — `eth0` (NAT, 10.0.2.0/24) for general VM connectivity and `eth1` (host-only, 192.168.56.0/24) for communication with Metasploitable2. All target findings are derived from Nmap enumeration of `192.168.56.102`. Wireshark captures demonstrate packet analysis workflow and protocol-level visibility.

---

## 🎯 Objectives

- Capture and analyze real network traffic using Wireshark in a lab environment
- Perform service version detection with Nmap against a vulnerable target
- Identify exposed services and map them to known vulnerabilities
- Detect port scan patterns at the packet level
- Document findings in a structured security report with risk ratings and remediation recommendations

---

## 🔵 SOC Analyst Workflow

**Typical network investigation scenario this lab replicates:**

1. Anomalous traffic detected in SIEM → analyst opens packet capture
2. Wireshark filter applied → isolate relevant protocol (TCP SYN, ARP, DNS)
3. SYN packet pattern identified → port scan investigation opened
4. ARP table reviewed → check for MAC-to-IP inconsistencies (MITM indicator)
5. Nmap service scan run → identify exposed services and software versions
6. CVEs cross-referenced → assess exploitability and business impact
7. Risk ratings assigned → findings documented with remediation steps
8. Report submitted → attached to incident ticket as evidence

---

## 📋 Lab Exercises

### Exercise 1 — Network Discovery

```bash
sudo nmap -sn 192.168.56.0/24
```

Identified live hosts on the host-only network including Metasploitable2 at `192.168.56.102`.

---

### Exercise 2 — Service Version Detection

```bash
sudo nmap -sV 192.168.56.102
```

**Open Ports and Services Discovered:**

| Port | Service | Version | Risk |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 | 🔴 CRITICAL |
| 22/tcp | SSH | OpenSSH 4.7p1 | 🟡 MEDIUM |
| 23/tcp | Telnet | Linux telnetd | 🔴 HIGH |
| 25/tcp | SMTP | Postfix smtpd | 🟡 MEDIUM |
| 53/tcp | DNS | ISC BIND 9.4.2 | 🟡 MEDIUM |
| 80/tcp | HTTP | Apache 2.2.8 | 🔴 HIGH |
| 139/tcp | NetBIOS | Samba 3.X-4.X | 🔴 HIGH |
| 445/tcp | SMB | Samba 3.X-4.X | 🔴 HIGH |
| 512/tcp | rexec | netkit-rsh rexecd | 🔴 CRITICAL |
| 513/tcp | rlogin | rlogind | 🔴 CRITICAL |
| 514/tcp | rsh | rshd | 🔴 CRITICAL |
| 1099/tcp | Java-RMI | GNU Classpath | 🔴 HIGH |
| 1524/tcp | bindshell | Metasploitable root shell | 🔴 CRITICAL |
| 2049/tcp | NFS | 2-4 (RPC) | 🟡 MEDIUM |
| 3306/tcp | MySQL | 5.0.51a | 🔴 HIGH |
| 5432/tcp | PostgreSQL | 8.3.0-8.3.7 | 🔴 HIGH |
| 5900/tcp | VNC | Protocol 3.3 | 🔴 HIGH |
| 6000/tcp | X11 | access denied | 🟡 MEDIUM |
| 6667/tcp | IRC | UnrealIRCd | 🔴 HIGH |
| 8009/tcp | AJP13 | Apache Jserv | 🟡 MEDIUM |
| 8180/tcp | HTTP | Apache Tomcat | 🔴 HIGH |

---

### Exercise 3 — Wireshark Traffic Analysis

#### Wireshark Filters Applied

| Filter | Purpose |
|---|---|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Isolate SYN packets — identifies port scan activity |
| `tcp.flags.reset == 1` | TCP RST packets — closed port responses |
| `arp` | ARP traffic — check for cache poisoning indicators |
| `dns` | DNS queries — detect tunneling or suspicious lookups |
| `ip.addr == 192.168.56.102` | Filter target-specific traffic |

#### Observations
- TCP SYN packets to sequential destination ports confirmed classic Nmap scan signature in packet capture
- SYN/ACK responses visible on open ports; TCP RST responses on closed ports — consistent with expected scan behavior
- ARP request/reply activity observed; MAC-to-IP mappings remained consistent throughout — no spoofing detected
- DNS PTR lookups generated during Nmap service enumeration — normal scan behavior
- ICMPv6 multicast traffic observed as standard local network discovery behavior

---

## 🚨 Critical Findings

### Finding 1 — vsftpd 2.3.4 (CVE-2011-2523)
**Severity:** CRITICAL  
**Port:** 21/tcp  
**Detail:** vsftpd 2.3.4 is the version distributed after the official vsftpd download server was compromised in 2011. The attacker injected a backdoor that triggers an unauthenticated root shell on port 6200 when a smiley face string is sent in the FTP username field. This is a supply-chain compromise baked into the binary itself, not a misconfiguration.  
**Validation:** Version-based identification only — exploitation not performed in this lab.  
**Remediation:** Remove vsftpd 2.3.4 immediately. Replace with a current patched version or disable FTP entirely and use SFTP over SSH.

---

### Finding 2 — Unauthenticated Root Shell (Port 1524)
**Severity:** CRITICAL  
**Port:** 1524/tcp  
**Detail:** Metasploitable2 intentionally exposes a bindshell providing unauthenticated root access. Any host that can reach this port has immediate full system access.  
**Validation:** Version-based identification only — exploitation not performed in this lab.  
**Remediation:** Disable immediately. This service has no legitimate production use case.

---

### Finding 3 — Legacy Plaintext Remote Access Protocols
**Severity:** CRITICAL  
**Ports:** 512 (rexec), 513 (rlogin), 514 (rsh)  
**Detail:** These protocols transmit all data — including credentials — in cleartext. Any network observer with Wireshark can capture usernames and passwords in real time. These protocols were deprecated in favor of SSH in the 1990s.  
**Remediation:** Disable rexec, rlogin, and rsh entirely. Enforce SSH for all remote access.

---

### Finding 4 — Telnet Enabled (Port 23)
**Severity:** HIGH  
**Detail:** Telnet transmits all session data including credentials in plaintext. Demonstrable with a single Wireshark capture — credentials appear in clear text in TCP stream.  
**Remediation:** Disable Telnet. Replace with SSH.

---

### Finding 5 — Network-Accessible Database Services
**Severity:** HIGH  
**Ports:** 3306 (MySQL), 5432 (PostgreSQL)  
**Detail:** Both database services are reachable from the scanning host with no apparent network-level restriction. Database services should never be exposed beyond the application tier.  
**Remediation:** Bind both services to localhost (127.0.0.1) or restrict access via firewall rules to trusted application servers only.

---

## 🗺️ MITRE ATT&CK Mapping

| Technique | ID | Lab Relevance |
|---|---|---|
| Network Service Discovery | T1046 | Nmap port and service enumeration |
| Exploit Public-Facing Application | T1190 | vsftpd CVE-2011-2523, Apache 2.2.8 |
| Valid Accounts | T1078 | Plaintext credential exposure via Telnet/rsh |
| Network Sniffing | T1040 | Wireshark packet capture demonstration |
| Exploitation of Remote Services | T1210 | Metasploitable root shell, rexec/rlogin/rsh |

---

## 🛡️ Remediation Summary

| Finding | Priority | Action |
|---|---|---|
| vsftpd 2.3.4 backdoor | Immediate | Remove and replace or disable FTP |
| Root shell port 1524 | Immediate | Disable service |
| rexec/rlogin/rsh | Immediate | Disable — replace with SSH |
| Telnet | High | Disable — replace with SSH |
| Exposed databases | High | Bind to localhost or restrict by firewall |
| Legacy Apache/Tomcat | Medium | Upgrade to supported versions |
| VNC exposed to network | Medium | Restrict access or require VPN |

---

## 📁 Repository Structure

```
Wireshark-Network-Analysis-Lab/
├── README.md                           # Full lab documentation
├── analysis-report.md                  # Detailed technical findings
├── findings-summary.md                 # Executive summary
├── screenshots/
│   ├── nmap-scan-results.webp          # Nmap service enumeration output
│   ├── nmap-wireshark-sidebyside.webp  # Side-by-side lab workflow view
│   └── wireshark-arp-dns.webp          # ARP and DNS traffic capture
└── .gitignore
```

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Wireshark | 4.6 | Packet capture and protocol analysis |
| Nmap | 7.95 | Network discovery and service version detection |
| Kali Linux | 2025 | Attack platform |
| Metasploitable2 | — | Intentionally vulnerable target |
| VirtualBox | — | Isolated lab environment |

---

## ⚠️ Disclaimer

This lab was conducted in an **isolated VirtualBox environment** against a **deliberately vulnerable** target (Metasploitable2). All activities were performed for educational purposes only. Never perform these activities against systems you do not own or have explicit written permission to test.

---

*Built by [Lovedip Singh](https://github.com/Lovedipsingh) — SOC analyst portfolio project.*  
*[LinkedIn](https://linkedin.com/in/lovedip-singh-76802a1a3) | [GitHub](https://github.com/Lovedipsingh)*
