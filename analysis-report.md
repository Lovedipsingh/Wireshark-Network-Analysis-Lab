# 📊 Technical Analysis Report
## Network Traffic Analysis — Metasploitable2 Lab

| Field | Details |
|---|---|
| **Date** | March 22, 2026 |
| **Analyst** | Lovedip Singh |
| **Environment** | VirtualBox lab with host-only target network and NAT connectivity for the Kali VM |
| **Classification** | Educational — NOT Production |

---

## 1. Executive Summary

A network traffic analysis was conducted against a deliberately vulnerable target (Metasploitable2) using Wireshark and Nmap from a Kali Linux attack platform. The assessment identified **21 open ports**, **3 critical findings**, **7 high-severity findings**, and captured **2,626 packets** of network traffic demonstrating common attack patterns in a controlled lab environment.

**Overall Risk Rating: CRITICAL**

| Metric | Value |
|---|---|
| Total Packets Captured | 2,626 |
| Open Ports Discovered | 21 |
| Critical Findings | 3 |
| High Severity Findings | 7 |
| Overall Risk Rating | CRITICAL |

**Top 5 Most Important Findings:**
1. vsftpd 2.3.4 backdoor — CVE-2011-2523
2. Unauthenticated root shell service on port 1524
3. Unencrypted remote shell protocols (rexec/rlogin/rsh)
4. Network-accessible database services (MySQL, PostgreSQL)
5. Telnet enabled — credentials transmitted in plaintext

---

## 2. Scope

| Field | Details |
|---|---|
| **Target** | Metasploitable2 (192.168.56.102) |
| **Scope** | Single host — all ports and services |
| **Authorization** | Lab environment — no third-party systems involved |
| **Excluded** | Exploitation, credential access, post-exploitation |
| **Test Type** | Black-box service enumeration and traffic analysis |

---

## 3. Methodology

### 3.1 Network Discovery
| Field | Details |
|---|---|
| Tool | Nmap 7.95 |
| Command | `sudo nmap -sn 192.168.56.0/24` |
| Purpose | Identify live hosts on the host-only lab network |
| Result | Live hosts discovered including Metasploitable2 at 192.168.56.102 |

### 3.2 Service Detection
| Field | Details |
|---|---|
| Tool | Nmap 7.95 |
| Command | `sudo nmap -sV 192.168.56.102` |
| Purpose | Identify open ports and service versions on the target |
| Result | 21 open ports identified |

### 3.3 Traffic Capture
| Field | Details |
|---|---|
| Tool | Wireshark 4.6 |
| Interface | eth0 (NAT-facing) |
| Duration | Full lab session |
| Packets Captured | 2,626 |

> **Note:** The Kali VM used two interfaces — `eth0` (NAT, `10.0.2.0/24`) and `eth1` (host-only, `192.168.56.0/24`). Wireshark capture was conducted on `eth0` and provides supplementary packet-analysis context. Definitive target findings for `192.168.56.102` are derived from Nmap service detection output.

---

## 4. Findings

### 4.1 Port Scan Detection

During Nmap service enumeration, scan-related TCP SYN activity was observed during the lab session. In standard TCP service detection, open ports typically respond with **SYN/ACK** packets while closed ports typically respond with **TCP RST** packets. DNS PTR reverse lookup queries were also generated during enumeration.

This is a common port-scan pattern that many IDS/IPS rulesets are designed to detect and alert on.

### 4.2 Critical Services Identified

#### vsftpd 2.3.4 (CVE-2011-2523)
- **Port:** 21/tcp
- **Risk:** CRITICAL
- **Observed:** FTP service identified as vsftpd 2.3.4 on port 21
- **Description:** This version is historically associated with CVE-2011-2523 — a backdoor introduced when the vsftpd download server was compromised. Sending a smiley face character in the username triggers a root shell on port 6200.
- **Validation status:** Version-based identification only — exploit was not executed in this lab.

#### Metasploitable Root Shell
- **Port:** 1524/tcp
- **Risk:** CRITICAL
- **Observed:** Nmap identified port 1524 as the Metasploitable root shell service.
- **Description:** In Metasploitable2, this service is intended to provide unauthenticated root shell access.
- **Validation status:** Exploit validation was not performed in this lab.

#### Unencrypted Remote Shell Protocols
- **Ports:** 512, 513, 514 (rexec, rlogin, rsh)
- **Risk:** CRITICAL
- **Description:** These protocols transmit all data including credentials in plaintext. They were deprecated decades ago and should never be exposed on any network.

#### Samba Vulnerability
- **Ports:** 139, 445
- **Risk:** HIGH
- **Description:** Samba 3.X is a legacy version associated with multiple CVEs. Requires version-specific validation to confirm exploitability.

#### Network-Accessible Database Services
- **Ports:** 3306 (MySQL), 5432 (PostgreSQL)
- **Risk:** HIGH
- **Description:** Both database services were network-accessible from the scanning host. Exposing database services increases attack surface and may allow unauthorized access if weak credentials or legacy configurations exist.

---

## 5. Wireshark Analysis

> Wireshark captures in this lab were collected on the NAT-facing interface and are presented as supplementary packet-analysis context. Target-host exposure findings are supported primarily by Nmap scan results.

### 5.1 ARP Traffic
- ARP request/reply activity observed on the local VM network
- No ARP spoofing detected — MAC addresses remained consistent throughout the session

### 5.2 DNS Traffic
- Multiple PTR (reverse DNS) lookups observed during Nmap service enumeration
- Service enumeration separately identified the target's DNS service as BIND 9.4.2, an outdated version
- No evidence of DNS tunneling identified in the packet capture

### 5.3 TCP SYN Pattern
- Multiple TCP SYN probes observed across destination ports during service enumeration
- Pattern is consistent with standard Nmap service detection behavior
- This is a common port-scan signature that IDS/IPS systems are designed to detect

---

## 6. Risk Summary

| Finding | Severity | CVE | Remediation |
|---|---|---|---|
| vsftpd 2.3.4 backdoor | CRITICAL | CVE-2011-2523 | Upgrade immediately |
| Root shell on port 1524 | CRITICAL | N/A | Remove immediately |
| rexec/rlogin/rsh | CRITICAL | N/A | Disable — use SSH |
| Samba 3.X | HIGH | Multiple | Upgrade |
| MySQL exposed | HIGH | N/A | Restrict to trusted hosts |
| PostgreSQL exposed | HIGH | N/A | Restrict to trusted hosts |
| Telnet enabled | HIGH | N/A | Disable — use SSH |
| Apache 2.2.8 | HIGH | Multiple | Upgrade |
| UnrealIRCd | HIGH | CVE-2010-2075 | Remove or upgrade |
| VNC exposed | HIGH | N/A | Restrict access |

---

## 7. Limitations

| Limitation | Impact |
|---|---|
| Version-based identification only | Some findings rely on banner data — exploit validation was not performed |
| Wireshark captured on NAT interface | Packet evidence is supplementary — does not directly show target host traffic |
| Single scan session | Dynamic or intermittent services may not have been captured |
| Lab environment | Findings apply to Metasploitable2 specifically — not a production system |

---

## 8. Conclusion

The target system (Metasploitable2) demonstrates numerous legacy and insecure services that are historically associated with severe compromise risk, including potential unauthenticated root access and plaintext credential exposure. In a real environment, systems with similar weaknesses would represent a critical security risk and require immediate remediation. This analysis demonstrates the importance of:

1. Regular vulnerability scanning and patch management
2. Network traffic monitoring with tools like Wireshark
3. Service hardening and removal of legacy protocols
4. Network segmentation and firewall rules
5. Principle of least privilege

**All findings were identified in an isolated lab environment for educational purposes only.**

---

*Report prepared by Lovedip Singh — Cybersecurity Analyst*
*CompTIA Security+ | Network+*
