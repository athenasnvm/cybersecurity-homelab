# Executive Summary

## Overview

A controlled security assessment was performed against a Metasploitable2 virtual machine deployed within an isolated laboratory environment.

The project was designed as a practical cybersecurity learning exercise covering the complete assessment workflow from initial host discovery to technical reporting.

The assessment included:

```text
Host Discovery
      ↓
Port Discovery
      ↓
Service Enumeration
      ↓
Technology Identification
      ↓
Vulnerability Research
      ↓
Exploitation
      ↓
Post-Exploitation
      ↓
Privilege Escalation
      ↓
Security Assessment
      ↓
Final Reporting
```

Testing identified multiple vulnerable services, insecure configurations, excessive privileges, and information-disclosure conditions.

Several weaknesses were successfully exploited and resulted in remote command execution, privileged filesystem access, and root-level system compromise.

---

# Assessment Scope

## Target

| Property        | Value                       |
| --------------- | --------------------------- |
| System          | Metasploitable2             |
| IP Address      | `192.168.56.107`            |
| Environment     | Isolated Virtual Laboratory |
| Assessment Host | Kali Linux                  |
| Kali IP         | `192.168.56.106`            |

The original laboratory design also included Ubuntu Server, Windows Server, and Windows Client systems.

Due to the amount of enumeration, validation, exploitation, evidence collection, and documentation required, the final assessment scope was deliberately reduced to Metasploitable2.

This allowed the project to prioritize depth of analysis over superficial testing of multiple targets.

---

# Overall Security Assessment

The Metasploitable2 target demonstrated a **critically insecure security posture within the tested laboratory environment**.

This conclusion is based on practical impact observed during testing rather than software versions alone.

The assessment successfully demonstrated:

```text
Remote Command Execution
        +
Direct Root-Level Remote Access
        +
Unprivileged Remote Shell Access
        +
Local Privilege Escalation
        +
Effective Root Execution
        +
Privileged Remote Filesystem Access
        +
Information Disclosure
        +
Administrative Interface Exposure
```

Most importantly, multiple independent attack paths could lead to significant system compromise.

Securing only one vulnerable service would therefore not have been sufficient to protect the target.

---

# Key Findings

## FTP — CVE-2011-2523

The FTP service was identified as:

```text
vsftpd 2.3.4
```

Controlled exploitation successfully triggered the backdoor associated with:

```text
CVE-2011-2523
```

A Meterpreter session was established and subsequently validated through a native command shell.

The resulting execution context was:

```text
User: root
UID: 0
GID: 0
```

### Demonstrated Impact

**Direct remote root-level system compromise.**

No additional privilege escalation was required.

---

# SMB — CVE-2007-2447

The target exposed:

```text
Samba 3.0.20-Debian
```

The assessment successfully exploited:

```text
CVE-2007-2447
Samba Username Map Script Command Execution
```

A remote command shell was obtained.

Validation returned:

```text
whoami
→ root

id
→ uid=0(root) gid=0(root)
```

### Demonstrated Impact

**Unauthenticated remote command execution with full root privileges.**

---

# DistCC — CVE-2004-2687

The exposed DistCC service was successfully exploited using:

```text
CVE-2004-2687
```

An initial reverse-shell payload failed because the target environment did not support the attempted `/dev/tcp` mechanism.

The payload was changed to:

```text
cmd/unix/reverse_perl
```

and a command shell was successfully established.

The resulting account was:

```text
daemon
UID 1
GID 1
```

### Demonstrated Impact

**Remote command execution with unprivileged system access.**

This foothold was later used for post-exploitation enumeration and privilege-escalation testing.

---

# NFS — Insecure Root Export

The NFS service exposed the target root filesystem using:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

This configuration combined:

```text
Root Filesystem Export
        +
Wildcard Client Access
        +
Read / Write Access
        +
no_root_squash
```

The filesystem could be mounted remotely from the Kali assessment host.

A temporary file created through the NFS mount as root on the client was stored remotely with:

```text
root:root
```

ownership.

### Demonstrated Impact

**Privileged remote filesystem access with UID 0 preservation.**

No remote shell was obtained through the NFS finding, and the report does not represent this condition as remote code execution.

---

# Local Privilege Escalation — SUID Nmap

Post-exploitation enumeration was performed from the unprivileged DistCC shell.

The initial security context was:

```text
daemon
UID 1
```

A root-owned SUID installation of:

```text
/usr/bin/nmap
```

was identified.

The installed version was:

```text
Nmap 4.53
```

The legacy interactive functionality was successfully used to execute a shell with:

```text
whoami
→ root
```

while `id` demonstrated:

```text
uid=1(daemon)
gid=1(daemon)
euid=0(root)
groups=1(daemon)
```

### Demonstrated Impact

**Local privilege escalation from an unprivileged remote shell to effective root privileges.**

The real UID remained `daemon`, while the effective UID became `root`.

---

# HTTP Security Findings

The HTTP service did not produce a remote command shell during the assessment.

However, multiple security-relevant exposures were confirmed.

Identified technologies and resources included:

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

---

## TikiWiki

The publicly exposed TikiWiki installation disclosed:

```text
Version: 1.9.5
Database Technology: MySQL
Database User: root
Database Host: localhost
Internal Filesystem Paths
```

The application installation interface was also accessible without prior authentication.

Historical vulnerabilities affecting TikiWiki 1.9.5 were researched but were not independently reproduced because the application failed during database initialization.

### Demonstrated Impact

**Backend information disclosure and insecure installation-interface exposure.**

---

## phpMyAdmin

The phpMyAdmin administration interface was publicly reachable.

A login form was exposed to unauthenticated network clients.

The assessment did not demonstrate:

```text
Authentication Bypass
Database Access
Administrative Access
Remote Code Execution
```

### Demonstrated Impact

**Administrative interface exposure and increased attack surface.**

---

## WebDAV

WebDAV functionality was confirmed at:

```text
/dav/
```

The endpoint advertised extended methods including:

```text
DELETE
PROPFIND
PROPPATCH
COPY
MOVE
LOCK
UNLOCK
```

However, unauthorized write, modification, or deletion operations were not demonstrated.

### Demonstrated Impact

**Expanded HTTP attack surface.**

---

# Attack Path Summary

The primary attack paths demonstrated during the assessment were:

```text
FTP
 ↓
CVE-2011-2523
 ↓
root
```

```text
SMB
 ↓
CVE-2007-2447
 ↓
root
```

```text
DistCC
 ↓
CVE-2004-2687
 ↓
daemon
 ↓
SUID Nmap
 ↓
euid=0(root)
```

```text
NFS
 ↓
Root Filesystem Export
 ↓
rw + no_root_squash
 ↓
Remote UID 0 Preservation
```

These paths demonstrate that administrative compromise was possible through several independent weaknesses.

---

# Security Themes

The assessment identified several recurring security problems.

## Legacy Software

Multiple services and applications were significantly outdated and associated with known security vulnerabilities.

---

## Excessive Service Exposure

The target exposed a large number of network services and web applications simultaneously.

Each additional service increased the available attack surface.

---

## Unsafe Configuration

Examples included:

```text
NFS Root Export
no_root_squash
Anonymous FTP
Anonymous SMB
SMBv1
SMB Message Signing Disabled
SUID Nmap
```

---

## Excessive Privilege

Several attack paths resulted directly in root-level execution.

Local configuration also allowed an unprivileged shell to obtain effective root privileges.

---

## Information Disclosure

HTTP applications exposed:

```text
Technology Versions
Database Information
Internal Filesystem Paths
Installation Interfaces
Diagnostic Resources
Directory Listings
```

---

# Risk Interpretation

The most important security issue was not a single vulnerability.

Instead, the target contained multiple weaknesses that could be chained or exploited independently.

The security posture can therefore be represented as:

```text
Multiple Vulnerable Services
        +
Weak Configuration
        +
Excessive Privileges
        +
Information Disclosure
        ↓
Multiple Paths to System Compromise
```

This significantly increases the probability and potential impact of successful exploitation.

---

# Recommended Remediation Priorities

The highest remediation priorities are:

```text
1. Remove or patch vulnerable vsftpd
2. Upgrade or replace vulnerable Samba
3. Remove or restrict DistCC
4. Correct the NFS root export
5. Remove SUID privileges from Nmap
6. Disable anonymous FTP and SMB access
7. Disable SMBv1
8. Restrict administrative web interfaces
9. Remove exposed installers and diagnostic resources
10. Upgrade legacy HTTP applications
11. Reduce information disclosure
12. Reduce the overall network attack surface
```

Detailed corrective actions are documented in:

```text
10-final-report/remediation.md
```

---

# Evidence

The conclusions presented in this summary are supported by raw technical evidence preserved throughout the repository.

Primary exploitation and validation evidence is available under:

```text
06-exploitation/
```

Post-exploitation evidence is documented under:

```text
07-post-exploitation/
```

Privilege-escalation validation is documented under:

```text
08-privilege-escalation/
```

The consolidated mapping between findings and supporting files is documented in:

```text
10-final-report/evidence-summary.md
```

---

# Assessment Documentation

The complete project structure is:

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

Each stage documents a different part of the assessment process.

---

# Conclusion

The security assessment demonstrated multiple practical paths toward administrative compromise of the Metasploitable2 target.

FTP and SMB independently resulted in direct root-level remote access.

DistCC provided an unprivileged remote shell that was successfully chained with a local SUID Nmap weakness to obtain effective root privileges.

The insecure NFS configuration provided privileged remote filesystem access while preserving client UID `0`.

HTTP testing identified extensive application exposure and information disclosure, although no HTTP-based remote command execution was demonstrated.

The overall result demonstrates a central security principle:

```text
A system is not secured by fixing
only its most obvious vulnerability.

The complete attack surface,
privilege boundaries,
service configuration,
and interaction between weaknesses
must be considered together.
```

## Final Assessment

**Multiple independent paths to full or privileged system compromise were successfully demonstrated in the isolated laboratory environment.**
