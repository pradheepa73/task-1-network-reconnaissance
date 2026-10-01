# 🛡️ Task 1 — Local Network Port Scanning & Exposure Assessment

![Nmap](https://img.shields.io/badge/Tool-Nmap-blue?logo=nmap)
![Wireshark](https://img.shields.io/badge/Tool-Wireshark-1679A7?logo=wireshark)
![Kali](https://img.shields.io/badge/OS-Kali%20Linux-557C94?logo=kalilinux)
![VirtualBox](https://img.shields.io/badge/Platform-VirtualBox-183A61?logo=virtualbox)
![Security](https://img.shields.io/badge/Domain-Network%20Security-red)
![Status](https://img.shields.io/badge/Status-Completed-success)
![License](https://img.shields.io/badge/License-MIT-yellow)

> A professional network reconnaissance report demonstrating host discovery, TCP SYN scanning, service enumeration, packet-level validation, risk assessment, and remediation planning.

---

## 📌 Executive Summary

This assessment discovered live hosts and open ports within a controlled VirtualBox network using **Nmap** and validated scan behaviour with **Wireshark**. The environment consisted of a **Kali Linux VM (192.168.56.10)** running on a Windows host **(192.168.56.1)**.

**Key metrics:**
- Network range scanned: `192.168.56.0/24`
- Live hosts discovered: `2` (Kali VM, Windows host)
- Open TCP ports identified: `2` (on Kali VM)
- Most critical exposure: `tor-socks (9050)` and `http (8080)` on the Kali VM
- Windows host: all probes returned `filtered` → effective firewall posture

> [!IMPORTANT]
> This scan was performed only on a network I own or have explicit permission to test. Unauthorized port scanning may be illegal and unethical.

---

## 🎯 Objectives

- Discover live devices on the local network.
- Identify open TCP ports and running services.
- Perform TCP SYN scanning with Nmap.
- Capture and analyse packets with Wireshark.
- Research common services and associated risks.
- Produce a professional remediation and hardening plan.

---

## 🧰 Tools & Environment

| Tool | Version | Purpose |
|---|---|---|
| Kali Linux | 2026.2 | Testing platform |
| VirtualBox | 7.x | Virtualisation |
| Windows Host | Windows 10/11 | Host OS |
| Nmap | 7.99 | Host discovery, port scanning, service detection |
| Wireshark | 4.6.6 | Packet capture and SYN scan validation |
| Network Mode | Host-Only | Lab network |

---

## 💻 Scan Execution

### 1. Identify local IP range
```bash
ip a
ip route
```

### 2. Live host discovery
```bash
sudo nmap -sn 192.168.56.0/24 -oN scans/live_hosts.txt
```

### 3. TCP SYN scan with service detection
```bash
sudo nmap -sS -sV --open -T4 192.168.56.0/24 -oA scans/network_scan
```

### 4. Targeted localhost scan
```bash
sudo nmap -sS -sV -p 9050,8080 localhost -oN scans/localhost_scan.txt
```

---

## 📊 Results

### Live Hosts Discovered

| # | IP Address | Role |
|---|---|---|
| 1 | `192.168.56.10` | Kali Linux VM (scanner) |
| 2 | `192.168.56.1` | Windows Host (VirtualBox gateway) |

### Open Ports & Services

| Host | Port | Protocol | Service | Version | Risk | Recommendation |
|---|---|---|---|---|---|---|
| 192.168.56.10 (Kali) | 9050 | TCP | tor-socks | Tor SOCKS proxy | Medium | Bind to 127.0.0.1 only; disable if unused |
| 192.168.56.10 (Kali) | 8080 | TCP | http | Python SimpleHTTPServer 0.6 | Low | Stop test server when not in use |
| 192.168.56.1 (Windows) | all | TCP | filtered | — | Low | Firewall correctly blocking |

### Key Findings

1. **Tor SOCKS proxy listening on 9050** — unknown exposure if bound to external interface; verify it is localhost-only.
2. **Python SimpleHTTPServer running on 8080** — a test service; ensure it is not left running in production.
3. **Windows host fully firewalled** — all probes to `192.168.56.1` returned `filtered`; this is the correct secure baseline.
4. **Isolated Host-Only network** — no bridged exposure to the physical LAN, which limits attack surface.

---

## 🕵️ Wireshark Packet Analysis

Wireshark confirmed Nmap's TCP SYN scan behaviour.

**Filter used:**
```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Observed behaviour:**
- Nmap sends SYN packets to target ports.
- Open ports respond with SYN/ACK.
- Closed ports respond with RST.
- The full TCP handshake is not completed for `-sS` (half-open scan).

**Evidence:** `screenshots/05-wireshark-syn.png` and `scans/syn_scan_capture.pcapng`

---

## 🧠 Common Services & Security Implications

| Port | Service | Common Risk |
|---|---|---|
| 21 | FTP | Plaintext credentials |
| 22 | SSH | Brute-force, weak passwords |
| 23 | Telnet | Plaintext remote access |
| 25 | SMTP | Open relay, spam abuse |
| 53 | DNS | Amplification, cache poisoning |
| 80 | HTTP | Unencrypted traffic |
| 443 | HTTPS | Weak TLS, certificate issues |
| 445 | SMB | Ransomware, lateral movement |
| 3306 | MySQL | Weak credentials, data exposure |
| 3389 | RDP | Brute-force, BlueKeep-style risks |
| 9050 | Tor SOCKS | Anonymity abuse, policy violations |

---

## 📊 Risk Assessment Matrix

| Finding | Risk Level | Impact | Likelihood |
|---------|------------|--------|------------|
| Tor SOCKS on 9050 | 🟠 Medium | Anonymity abuse / policy violation | Medium |
| Python HTTP on 8080 | 🟡 Low | Information disclosure | Low |
| Windows host filtered | 🟢 Secure | N/A – correctly configured | N/A |
| Host-Only network | 🟢 Secure | Limited attack surface | N/A |

---

## 🛡️ Remediation & Hardening Plan

| Priority | Action | Owner | Status |
|---|---|---|---|
| 🔴 High | Bind Tor SOCKS to `127.0.0.1` or disable if unused | Admin | Pending |
| 🔴 High | Stop Python HTTP test server when not in use | Admin | Pending |
| 🔴 High | Keep Windows Firewall enabled (already blocking) | Admin | ✅ Done |
| 🟠 Medium | Patch all network-facing services | Admin | Pending |
| 🟠 Medium | Enforce strong passwords and MFA | Admin | Pending |
| 🟡 Low | Enable IDS/IPS and log monitoring | Admin | Pending |
| 🟡 Low | Segment IoT and guest devices | Admin | Pending |

---

## 🎤 Interview Questions & Answers

### 1. What is an open port?
An open port is a TCP or UDP port that is actively listening and accepting connections for a service.

### 2. How does Nmap perform a TCP SYN scan?
Nmap sends a SYN packet. If the target replies SYN/ACK, the port is open. If it replies RST, the port is closed. Nmap does not complete the handshake — called a half-open scan.

### 3. What risks are associated with open ports?
Exposure of vulnerable services, unauthorized access, data leakage, brute-force attacks, malware infection, and lateral movement.

### 4. Explain the difference between TCP and UDP scanning.
TCP is connection-oriented and uses handshake responses. UDP is connectionless; closed ports may return ICMP port unreachable, while no response can mean open or filtered. UDP scanning is slower and less reliable.

### 5. How can open ports be secured?
Close unnecessary ports, use firewalls, patch services, enforce strong authentication, use VPNs, segment networks, and monitor traffic.

### 6. What is a firewall's role regarding ports?
A firewall filters traffic by IP, port, and protocol. It can allow or block connections to reduce exposure.

### 7. What is a port scan and why do attackers perform it?
A port scan probes a host to find open, closed, or filtered ports. Attackers use it for reconnaissance to identify entry points and vulnerable services.

### 8. How does Wireshark complement port scanning?
Wireshark shows the actual packets, confirms scan behaviour, helps troubleshoot filtered ports, and provides forensic evidence of network traffic.

---

## 📁 Repository Structure

```text
task-1-network-port-scanning/
├── README.md
├── 📁 scans/
│   ├── live_hosts.txt
│   ├── network_scan.nmap
│   ├── network_scan.xml
│   ├── network_scan.gnmap
│   ├── localhost_scan.txt
│   └── syn_scan_capture.pcapng
└── 📁 screenshots/
    ├── 01-virtualbox-network.png
    ├── 02-kali-ip.png
    ├── 03-live-hosts.png
    ├── 04-nmap-syn-scan.png
    ├── 05-wireshark-syn.png
    └── 06-files-saved.png
```

---

## 🧾 Lessons Learned

- Port scanning is a foundational reconnaissance technique.
- TCP SYN scans are fast and stealthy compared to full connect scans.
- Service version detection maps exposure to real risks.
- Wireshark provides packet-level validation of scan behaviour.
- Open ports must be justified, monitored, and hardened.
- A correctly configured firewall makes a host appear "down" to scan probes.

---

## 📚 References

- [Nmap Official Documentation](https://nmap.org/book/man.html)
- [Nmap Port Scanning Techniques](https://nmap.org/book/man-port-scanning-techniques.html)
- [Wireshark User Guide](https://www.wireshark.org/docs/)
- [OWASP Network Security](https://owasp.org/www-community/Network_Security)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)

---

## 👤 Author

**Pradheepa M**  
B.Sc. Computer Science with Cybersecurity  
📧 pradheepa378@gmail.com  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/pradheepa-m-051728372)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/pradheepa73)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:pradheepa378@gmail.com)

---

<div align="center">

**"Empowering The Digital Defenders"**  
*Network Security Assessment — Task 1*

</div>

---

> **Prepared by:** Pradheepa.M  
> **Date:** 2026-10-01  
> **Repository:** https://github.com/pradheepa73/task-1-network-port-scanning
```

