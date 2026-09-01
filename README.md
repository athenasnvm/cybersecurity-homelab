# Metasploitable2 Security Lab

## Cybersecurity Portfolio Project

This repository documents a complete cybersecurity laboratory assessment performed against **Metasploitable2** inside an isolated virtual environment.

The project was developed as a practical learning exercise to gain hands-on experience with:

* Network reconnaissance
* Port scanning
* Service enumeration
* Technology identification
* Vulnerability research
* Controlled exploitation
* Post-exploitation
* Privilege escalation
* Evidence collection
* Risk assessment
* Technical reporting

The project follows a structured workflow designed to resemble a simplified penetration-testing process.

---

# About Me

**Simón Atenas G.**

Cybersecurity student at **AIEP**, based in Santiago, Chile.

I am currently developing practical skills in cybersecurity through hands-on laboratories, technical documentation, networking exercises, vulnerability analysis, and security tooling.

## Technical Interests

* Vulnerability assessment
* Network security
* Linux security
* Penetration testing
* Security monitoring
* Cloud security
* Cybersecurity automation

## Technologies Used

```text
Kali Linux
Metasploitable2
Nmap
Metasploit Framework
SMB / Samba
NFS / RPC
FTP
HTTP / WebDAV
Linux CLI
Markdown
VirtualBox
```

Additional technologies and platforms used during my cybersecurity studies include Windows, Ubuntu, JavaScript, AWS, Packet Tracer, and related security tools.

---

# Project Objective

The primary objective of this laboratory was to practice the complete process of assessing a vulnerable system.

Rather than beginning directly with known Metasploitable2 exploits, the project followed the sequence:

```text
Discovery
   ↓
Enumeration
   ↓
Technology Identification
   ↓
Vulnerability Research
   ↓
Validation
   ↓
Exploitation
   ↓
Post-Exploitation
   ↓
Privilege Escalation
   ↓
Risk Assessment
   ↓
Reporting
```

A major goal of the project was to understand **why an attack works**, not only how to execute a predefined exploit.

---

# Laboratory Environment

The final laboratory environment focused on:

| Role            | System          | IP Address       |
| --------------- | --------------- | ---------------- |
| Assessment Host | Kali Linux      | `192.168.56.106` |
| Target          | Metasploitable2 | `192.168.56.107` |

The systems were deployed in an isolated virtual laboratory.

Metasploitable2 is intentionally vulnerable and was used exclusively for cybersecurity training.

---

# Original Laboratory Scope

The original project was designed to include several machines:

```text
Kali Linux
     ↓
Metasploitable2
Ubuntu Server
Windows Server
Windows Client
```

The intention was to build a heterogeneous laboratory containing both Linux and Windows systems.

As the assessment progressed, the amount of enumeration, research, testing, evidence collection, troubleshooting, and documentation required for each system became significantly larger than expected.

Because of the available time and my current learning stage, I decided to reduce the final scope and focus primarily on:

```text
Metasploitable2
```

This allowed me to perform a deeper assessment instead of producing a superficial analysis of several machines.

Ubuntu Server, Windows Server, and Windows Client may be incorporated into future projects or additional laboratory exercises.

---

# Project Structure

The repository is organized according to the assessment workflow.

```text
01-host-discovery/
02-port-discovery/
03-service-enumeration/
04-technology-identification/
05-vulnerability-research/
06-exploitation/
07-post-exploitation/
08-privilege-escalation/
09-assessment/
10-final-report/
```

Each phase contains documentation and, where appropriate:

```text
command.md / commands.md
results.md
scans/
evidence/
research/
```

---

# 01 — Host Discovery

The first phase focused on identifying active systems within the laboratory network.

The objective was to determine which hosts were reachable before performing any deeper assessment.

Primary activities included:

```text
Network Discovery
Host Identification
Target Selection
```

---

# 02 — Port Discovery

After identifying the target, TCP port discovery was performed to determine the exposed network attack surface.

Metasploitable2 exposed numerous open TCP ports and network services.

The results of this phase became the input for service enumeration.

---

# 03 — Service Enumeration

Service and version detection was performed against the previously identified open ports.

This phase attempted to determine:

```text
Port
 ↓
Protocol
 ↓
Service
 ↓
Software
 ↓
Version
```

The resulting information was later used for vulnerability research.

---

# 04 — Technology Identification

Selected services were investigated in greater depth.

The primary technologies analyzed included:

```text
FTP
HTTP
SMB
NFS
DistCC
```

This phase focused on authentication behavior, protocol configuration, web resources, shares, filesystem exports, and other technology-specific characteristics.

---

# 05 — Vulnerability Research

The technologies and versions identified during enumeration were researched for known vulnerabilities and insecure configurations.

Relevant findings included:

```text
CVE-2011-2523
CVE-2007-2447
CVE-2004-2687
Insecure NFS Configuration
HTTP Information Disclosure
Legacy Web Applications
```

The project deliberately distinguishes between:

```text
Version Match
     ≠
Confirmed Vulnerability
     ≠
Successful Exploitation
```

Potential vulnerabilities were validated before being represented as successfully exploited.

---

# 06 — Exploitation

Controlled exploitation and security validation were performed against selected findings.

## FTP

```text
vsftpd 2.3.4
      ↓
CVE-2011-2523
      ↓
Remote Session
      ↓
root
UID 0
```

The FTP attack path resulted directly in root-level remote access.

---

## SMB

```text
Samba
  ↓
CVE-2007-2447
  ↓
Command Shell
  ↓
root
UID 0
```

The SMB attack path also resulted directly in a root command shell.

---

## DistCC

```text
DistCC
  ↓
CVE-2004-2687
  ↓
Command Shell
  ↓
daemon
UID 1
```

The DistCC attack path produced an unprivileged remote shell.

---

## NFS

```text
Root Filesystem Export
        ↓
rw
        ↓
no_root_squash
        ↓
Remote UID 0 Preserved
```

The insecure NFS configuration allowed privileged remote filesystem operations.

---

## HTTP

HTTP testing confirmed multiple exposures and information-disclosure conditions.

No HTTP-based remote command shell was obtained.

This distinction is intentionally preserved in the documentation.

---

# 07 — Post-Exploitation

The DistCC shell was used as an unprivileged foothold for local system enumeration.

The assessment examined:

```text
Users
Groups
Processes
Services
Sensitive Files
Network Configuration
Scheduled Tasks
SUID Binaries
SGID Binaries
Potential Privilege Escalation Paths
```

This phase identified the local privilege-escalation candidate used in the next stage.

---

# 08 — Privilege Escalation

A root-owned SUID installation of:

```text
/usr/bin/nmap
```

was identified.

The installed version was:

```text
Nmap 4.53
```

The legacy interactive functionality was successfully used to obtain:

```text
uid=1(daemon)
euid=0(root)
```

This demonstrated escalation from an unprivileged DistCC shell to effective root privileges.

---

# 09 — Security Assessment

The technical results from previous phases were consolidated into structured security findings.

The assessment distinguishes between:

```text
Observation
Vulnerability
Misconfiguration
Successful Exploitation
Security Impact
```

This phase focused on interpreting what the technical results actually meant from a security perspective.

---

# 10 — Final Report

The final report consolidates the project into professional assessment documentation.

```text
10-final-report/
├── README.md
├── executive-summary.md
├── methodology.md
├── findings.md
├── remediation.md
└── evidence-summary.md
```

## Final Report Documents

### Executive Summary

Provides a high-level overview of the most important security outcomes.

### Methodology

Explains how the assessment progressed from discovery to reporting.

### Findings

Contains the consolidated technical security findings.

### Remediation

Provides recommended corrective actions and validation steps.

### Evidence Summary

Maps major findings to supporting raw evidence.

---

# Key Validated Findings

The most significant attack paths demonstrated during the project were:

## FTP

```text
CVE-2011-2523
        ↓
Remote Code Execution
        ↓
root
```

## SMB

```text
CVE-2007-2447
        ↓
Remote Command Execution
        ↓
root
```

## DistCC

```text
CVE-2004-2687
        ↓
Remote Command Execution
        ↓
daemon
```

## DistCC + Local Privilege Escalation

```text
daemon
  ↓
SUID Nmap
  ↓
euid=0(root)
```

## NFS

```text
/
+
rw
+
*
+
no_root_squash
        ↓
Privileged Remote Filesystem Access
```

---

# HTTP Findings

HTTP testing identified several exposed technologies and applications, including:

```text
Apache 2.2.8
PHP 5.2.4
WebDAV
phpinfo.php
phpMyAdmin
TikiWiki 1.9.5
TikiWiki Installer
Test Resources
Directory Listings
```

TikiWiki exposed backend information such as:

```text
Database Technology: MySQL
Database User: root
Database Host: localhost
Internal Filesystem Paths
```

No HTTP-based system shell or database compromise was demonstrated.

---

# Skills Practiced

This laboratory provided practical experience with:

```text
Network Reconnaissance
TCP Port Scanning
Service Fingerprinting
Protocol Enumeration
Vulnerability Research
CVE Analysis
Metasploit Framework
Remote Code Execution
Linux Enumeration
Privilege Escalation
SUID Analysis
NFS Security
SMB Security
HTTP Security
Evidence Collection
Technical Documentation
Risk Analysis
Remediation Planning
```

---

# Evidence-Driven Documentation

One of the main principles followed during the project was:

```text
Do Not Assume Impact
       ↓
Test It
       ↓
Observe It
       ↓
Save the Evidence
       ↓
Document the Actual Result
```

Examples include:

* A failed DistCC payload was documented before switching to a compatible payload.
* HTTP vulnerability candidates were not marked as exploited without supporting evidence.
* phpMyAdmin exposure was not represented as authentication bypass.
* WebDAV method exposure was not represented as unauthorized file modification.
* NFS UID `0` preservation was not represented as a remote shell.
* The SUID Nmap result was documented as `euid=0(root)` rather than incorrectly claiming that the real UID changed to 0.

---

# AI-Assisted Learning

Artificial intelligence tools were used extensively during this project as an educational and productivity aid.

At the beginning of the laboratory, I did not yet have enough practical experience to independently design, execute, troubleshoot, interpret, and document every stage of a complete security assessment within the time available.

AI assistance was therefore used to support areas such as:

```text
Concept Explanation
Command Selection
Troubleshooting
Output Interpretation
Vulnerability Research Guidance
Documentation Structure
Markdown Improvement
Technical Writing Review
```

The goal was to use AI as a learning assistant rather than treat AI-generated output as evidence.

The workflow followed during the project was:

```text
AI Guidance
     ↓
Manual Command Execution
     ↓
Real Laboratory Output
     ↓
Evidence Collection
     ↓
Result Interpretation
     ↓
Documentation
```

Commands were executed against the local laboratory environment and conclusions were based on the observed outputs.

When AI guidance or an expected result did not match the laboratory response, the actual technical result was preserved.

This repository therefore contains both successful and unsuccessful testing where relevant.

---

# Learning Context

This is an educational cybersecurity project and not a professional client penetration test.

The project represents my current learning process and is intended to demonstrate practical progress rather than expert-level offensive security experience.

Some sections of the methodology and documentation were refined iteratively as I learned more about the tools, vulnerabilities, and assessment process.

Future versions of this laboratory may expand the scope to additional operating systems and attack scenarios.

---

# Tools

Tools and technologies used throughout the project include:

| Tool                 | Purpose                                            |
| -------------------- | -------------------------------------------------- |
| Nmap                 | Host discovery, port scanning, service enumeration |
| Metasploit Framework | Controlled exploitation                            |
| curl                 | HTTP validation                                    |
| smbclient            | SMB enumeration                                    |
| NFS Utilities        | Export and filesystem analysis                     |
| FTP Client           | FTP validation                                     |
| Linux CLI            | System and privilege enumeration                   |
| VirtualBox           | Laboratory virtualization                          |
| Markdown             | Documentation                                      |

---

# Final Assessment

The assessment demonstrated several independent paths toward significant compromise of the target.

```text
FTP ─────────────→ root

SMB ─────────────→ root

DistCC → daemon
           ↓
       SUID Nmap
           ↓
        euid=0

NFS ─────────────→ privileged filesystem access

HTTP ────────────→ information and application exposure
```

The most important lesson from the laboratory was that security cannot be evaluated only by looking at individual vulnerabilities.

The interaction between:

```text
Exposed Services
       +
Vulnerable Software
       +
Weak Configuration
       +
Privilege Boundaries
```

determines the actual attack paths available against a system.

---

# Final Report

For the complete consolidated assessment, see:

```text
10-final-report/
```

Recommended reading order:

```text
10-final-report/README.md
        ↓
10-final-report/executive-summary.md
        ↓
10-final-report/findings.md
        ↓
10-final-report/remediation.md
        ↓
10-final-report/evidence-summary.md
```

For the detailed technical workflow, review phases:

```text
01-host-discovery/
        ↓
09-assessment/
```

---

# Disclaimer

This project was performed exclusively in an isolated laboratory using intentionally vulnerable systems designed for cybersecurity training.

The techniques documented in this repository should only be used against systems that you own or have explicit authorization to assess.

---

# Author

**Simón Atenas G.**

Cybersecurity Student — AIEP
Santiago, Chile

GitHub:

```text
https://github.com/athenasnvm
```

Email:

```text
simon.atenas7@gmail.com
```

Linkedin:

```text
https://www.linkedin.com/in/sim%C3%B3n-atenas-194170259/
```
