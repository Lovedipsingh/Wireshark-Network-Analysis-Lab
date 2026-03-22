# 📊 Technical Analysis Report
## Network Traffic Analysis — Metasploitable2 Lab
**Date:** March 22, 2026
**Analyst:** Lovedip Singh
**Environment:** Isolated VirtualBox Lab

---

## 1. Executive Summary

A network traffic analysis was conducted against a deliberately vulnerable target (Metasploitable2) using Wireshark and Nmap from a Kali Linux attack platform. The analysis identified **21 open ports**, **5 critical vulnerabilities**, and captured **2,626 packets** of network traffic demonstrating real-world attack patterns.

**Overall Risk Rating: CRITICAL**

---

## 2. Methodology

### 2.1 Network Discovery
```
Tool: Nmap 7.95
Command: sudo nmap -sn 10.0.2.0/24
Purpose: Identify live hosts on the network
Result: 3 hosts discovered
```

### 2.2 Service Detection
```
Tool: Nmap 7.95
Command: sudo nmap -sV 192.168.56.102
Purpose: Identify open ports and service versions
Result: 21 open ports identified
```

### 2.3 Traffic Capture
```
Tool: Wireshark 4.6
Interface: eth0
Duration: Full lab session
Packets Captured: 2,626
```

---

## 3. Findings

### 3.1 Port Scan Detection
During the Nmap service scan Wireshark captured the following patterns:

- **SYN packets** sent sequentially to all 1,000 ports
- **ICMP Destination Unreachable** responses for closed ports
- **TCP RST packets** from target for filtered ports
- **SYN-ACK responses** for open ports confirming services

This traffic pattern is a **textbook port scan signature** that any IDS/IPS would detect and alert on.

### 3.2 Critical Services Identified

#### vsftpd 2.3.4 (CVE-2011-2523)
- **Port:** 21/tcp
- **Risk:** CRITICAL
- **Description:** This FTP server version contains a backdoor. When a smiley face character `:)` is appended to the username during login, a root shell is spawned on port 6200.
- **CVSS Score:** 10.0

#### Metasploitable Root Shell
- **Port:** 1524/tcp
- **Risk:** CRITICAL
- **Description:** A bindshell running as root with no authentication. Any user connecting to this port receives immediate root access.
- **Remediation:** Immediately terminate this service and investigate how it was installed.

#### Unencrypted Remote Shell Protocols
- **Ports:** 512, 513, 514
- **Risk:** CRITICAL
- **Description:** rexec, rlogin, and rsh transmit all data in plaintext including credentials. Wireshark can capture everything transmitted over these protocols.

#### Samba Vulnerability
- **Ports:** 139, 445
- **Risk:** HIGH
- **Description:** Samba 3.X running on this system is vulnerable to multiple CVEs including remote code execution.

#### Exposed Database Services
- **Ports:** 3306 (MySQL), 5432 (PostgreSQL)
- **Risk:** HIGH
- **Description:** Both database services are exposed to the network with no firewall restrictions.

---

## 4. Wireshark Analysis

### 4.1 ARP Traffic
- Source: PCSSystemtec (Kali VM)
- Pattern: Normal ARP request/reply pairs
- Finding: No ARP spoofing detected — MAC addresses consistent

### 4.2 DNS Traffic
- Multiple PTR (reverse DNS) lookups during Nmap scan
- BIND 9.4.2 running on target — outdated version
- No DNS tunneling detected

### 4.3 TCP SYN Pattern
- Sequential SYN packets to ports 23, 25, 53, 80, 110, 111, 135, 139, 143, 443, 445, 993, 995, 1025, 1720, 1723, 3306, 3389, 5900, 8080
- Classic port scan signature
- ICMP unreachable responses confirm closed ports

---

## 5. Risk Summary

| Finding | Severity | CVE | Remediation |
|---|---|---|---|
| vsftpd 2.3.4 backdoor | CRITICAL | CVE-2011-2523 | Upgrade immediately |
| Root bindshell on 1524 | CRITICAL | N/A | Remove immediately |
| rexec/rlogin/rsh | CRITICAL | N/A | Disable, use SSH |
| Samba 3.X | HIGH | Multiple | Upgrade |
| MySQL exposed | HIGH | N/A | Firewall |
| PostgreSQL exposed | HIGH | N/A | Firewall |
| Telnet enabled | HIGH | N/A | Disable, use SSH |
| Apache 2.2.8 | HIGH | Multiple | Upgrade |
| UnrealIRCd | HIGH | CVE-2010-2075 | Remove |
| VNC exposed | HIGH | N/A | Restrict access |

---

## 6. Conclusion

The target system (Metasploitable2) demonstrates numerous critical security vulnerabilities that would allow an attacker to gain complete root access through multiple attack vectors. This analysis demonstrates the importance of:

1. Regular vulnerability scanning
2. Network traffic monitoring with tools like Wireshark
3. Patch management and service hardening
4. Network segmentation and firewall rules
5. Principle of least privilege

**All findings were identified in an isolated lab environment for educational purposes.**

---

*Report prepared by Lovedip Singh — Cybersecurity Analyst*
*CompTIA Security+ | Network+ | CySA+*
