# Security Assessment

## Executive Summary

A security assessment was performed against the Metasploitable2 system within an isolated laboratory environment.

The assessment followed a structured methodology consisting of:

1. Host Discovery
2. Port Discovery
3. Service Enumeration
4. Technology Identification
5. Vulnerability Research
6. Exploitation
7. Post-Exploitation
8. Privilege Escalation

Initial network enumeration identified Metasploitable2 as the system with the largest exposed TCP attack surface, with **30 open TCP ports**.

Further enumeration identified multiple legacy and remotely accessible services, including FTP, SMB, NFS, HTTP, distccd, database services, remote administration protocols, and additional application services.

Controlled exploitation confirmed multiple independent security weaknesses capable of providing unauthorized access or remote command execution.

The most significant findings included:

* Samba remote command execution through CVE-2007-2447.
* distccd remote command execution through CVE-2004-2687.
* vsftpd 2.3.4 backdoor command execution associated with CVE-2011-2523.
* NFS root filesystem exposure with read/write permissions and `no_root_squash`.
* Anonymous FTP access.
* HTTP information disclosure through TikiWiki.
* Local privilege escalation through a root-owned SUID installation of Nmap 4.53.

A complete attack chain was successfully demonstrated using the distccd vulnerability to obtain an initial unprivileged `daemon` shell, followed by local enumeration and exploitation of the SUID Nmap configuration to obtain an effective UID of `0` (`root`).

The assessment therefore demonstrated that the target could be compromised through multiple independent attack paths and that an initially restricted remote foothold could be escalated to effective root privileges.

**Overall Security Risk: Critical**

---

# 1. Attack Surface Assessment

Port discovery identified **30 open TCP ports** on Metasploitable2.

The exposed attack surface included:

* FTP
* SSH
* Telnet
* SMTP
* DNS
* HTTP
* RPC
* SMB
* rexec
* rlogin
* rsh
* Java RMI
* NFS
* MySQL
* PostgreSQL
* distccd
* VNC
* X11
* IRC
* Apache JServ Protocol
* Apache Tomcat
* Ruby DRb

Multiple additional RPC-related services were also exposed.

### Assessment

The large number and diversity of remotely accessible services significantly increased the number of potential entry points available to an attacker.

Several exposed services were legacy implementations or were configured in ways that subsequently resulted in confirmed security findings.

**Risk:** Critical

---

# 2. Confirmed Security Findings

## Finding 01 — Samba Remote Command Execution

**Service:** SMB
**Technology:** Samba 3.0.20-Debian
**Vulnerability:** CVE-2007-2447
**Severity:** Critical
**Status:** Confirmed

Controlled exploitation successfully produced remote command execution with root privileges.

### Impact

The vulnerability provides a direct remote path to privileged system compromise.

Because root privileges were obtained directly, successful exploitation can bypass the need for a separate local privilege escalation stage.

### CIA Impact

| Security Property | Impact   |
| ----------------- | -------- |
| Confidentiality   | Critical |
| Integrity         | Critical |
| Availability      | Critical |

---

## Finding 02 — distccd Remote Command Execution

**Service:** distccd
**Port:** `3632/tcp`
**Vulnerability:** CVE-2004-2687
**Severity:** Critical
**Status:** Confirmed

Controlled exploitation successfully produced a remote shell operating as:

```text
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

### Impact

The vulnerability provides unauthenticated remote command execution and establishes an initial foothold on the target.

Although the resulting session was initially unprivileged, it provided sufficient local access to perform post-exploitation enumeration.

This foothold was subsequently chained with a local privilege escalation weakness to obtain effective root privileges.

### CIA Impact

| Security Property | Impact |
| ----------------- | ------ |
| Confidentiality   | High   |
| Integrity         | High   |
| Availability      | High   |

---

## Finding 03 — vsftpd Backdoor Command Execution

**Service:** FTP
**Technology:** vsftpd 2.3.4
**Vulnerability:** CVE-2011-2523
**Severity:** Critical
**Status:** Confirmed

The backdoor command execution vulnerability associated with the deployed vsftpd version was successfully reproduced.

### Impact

Successful exploitation provided an additional remote command execution vector against the target.

This represents an independent compromise path separate from Samba and distccd.

### CIA Impact

| Security Property | Impact |
| ----------------- | ------ |
| Confidentiality   | High   |
| Integrity         | High   |
| Availability      | High   |

---

## Finding 04 — NFS Root Filesystem Exposure

**Service:** NFS
**Port:** `2049/tcp`
**Severity:** Critical
**Status:** Confirmed

The target exported its root filesystem using:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

The export was successfully mounted remotely.

Read and write access were confirmed, and a controlled root-level test demonstrated that files created through the mounted export using client-side root privileges retained:

```text
root:root
```

ownership.

### Impact

The configuration exposes the target's root filesystem to remote clients with read/write access.

The use of:

```text
no_root_squash
```

allows root identity from the NFS client to be preserved when accessing the exported filesystem.

This represents a critical trust and access-control failure.

### CIA Impact

| Security Property | Impact   |
| ----------------- | -------- |
| Confidentiality   | Critical |
| Integrity         | Critical |
| Availability      | Critical |

---

## Finding 05 — SUID Nmap Privilege Escalation

**Component:** `/usr/bin/nmap`
**Version:** Nmap 4.53
**Configuration:** SUID root
**Severity:** Critical
**Status:** Confirmed

Post-exploitation enumeration identified:

```text
-rwsr-xr-x root root /usr/bin/nmap
```

The installed Nmap version supported interactive mode.

A shell executed through the SUID Nmap process resulted in:

```text
uid=1(daemon) gid=1(daemon) euid=0(root) groups=1(daemon)
```

### Impact

The unsafe SUID configuration allowed the initially unprivileged `daemon` account to obtain effective root privileges.

The practical impact was independently demonstrated by successfully accessing `/etc/shadow`, which had been inaccessible to the original `daemon` session.

### CIA Impact

| Security Property | Impact   |
| ----------------- | -------- |
| Confidentiality   | Critical |
| Integrity         | Critical |
| Availability      | Critical |

---

## Finding 06 — Anonymous FTP Access

**Service:** FTP
**Technology:** vsftpd 2.3.4
**Severity:** High
**Status:** Confirmed

The FTP service accepted anonymous authentication.

### Impact

Unauthenticated users were able to establish FTP sessions without requiring a uniquely authenticated account.

This reduces access-control restrictions on the exposed FTP service.

### CIA Impact

| Security Property | Impact           |
| ----------------- | ---------------- |
| Confidentiality   | High             |
| Integrity         | Not Demonstrated |
| Availability      | Not Demonstrated |

---

## Finding 07 — Anonymous SMB Access

**Service:** SMB
**Technology:** Samba 3.0.20-Debian
**Severity:** High
**Status:** Confirmed

Anonymous SMB authentication and share enumeration were successfully performed.

The `tmp` share was accessible without authenticated user credentials.

### Impact

Unauthenticated users could enumerate SMB resources and access the exposed `tmp` share.

### CIA Impact

| Security Property | Impact           |
| ----------------- | ---------------- |
| Confidentiality   | High             |
| Integrity         | Not Demonstrated |
| Availability      | Not Demonstrated |

---

## Finding 08 — SMBv1 Enabled

**Service:** SMB
**Severity:** High
**Status:** Confirmed

SMBv1 support was identified on the target.

### Impact

The system exposes a legacy SMB protocol, increasing the security risk associated with the SMB attack surface.

This finding is separate from the successfully exploited Samba remote command execution vulnerability.

### CIA Impact

No direct confidentiality, integrity, or availability impact was independently demonstrated from SMBv1 support alone during this assessment.

---

## Finding 09 — SMB Message Signing Disabled

**Service:** SMB
**Severity:** Medium
**Status:** Confirmed

SMB message signing was identified as disabled.

### Impact

The configuration represents a weakening of SMB communication protections.

No independent exploitation of this configuration was performed during the assessment.

---

## Finding 10 — TikiWiki Information Disclosure

**Service:** HTTP
**Application:** TikiWiki 1.9.5
**Severity:** Medium
**Status:** Confirmed

Testing produced database-related errors and exposed configuration-related information.

### Impact

Internal application and database-related information was disclosed through application error behavior.

No remote command execution was obtained through this finding.

### CIA Impact

| Security Property | Impact            |
| ----------------- | ----------------- |
| Confidentiality   | Medium            |
| Integrity         | None Demonstrated |
| Availability      | None Demonstrated |

---

## Finding 11 — PHP Information Disclosure

**Service:** HTTP
**Resource:** `/phpinfo.php`
**Severity:** Medium
**Status:** Confirmed

A publicly accessible PHP information page exposed PHP and server configuration details.

### Impact

The disclosed information can assist reconnaissance by revealing internal configuration and technology details.

No direct system compromise was demonstrated through this finding.

### CIA Impact

| Security Property | Impact            |
| ----------------- | ----------------- |
| Confidentiality   | Medium            |
| Integrity         | None Demonstrated |
| Availability      | None Demonstrated |

---

# 3. Additional Attack Surface Findings

Additional HTTP resources and technologies were identified during enumeration, including:

* Apache HTTP Server 2.2.8
* PHP 5.2.4
* WebDAV functionality
* `/dav/`
* `/phpMyAdmin/`
* `/test/`
* TikiWiki 1.9.5

These findings increased the exposed application attack surface and provided additional reconnaissance information.

However, the presence of a legacy version or exposed resource was not automatically classified as a successfully exploited vulnerability.

Historical vulnerabilities associated with TikiWiki were investigated but were not successfully reproduced during the assessment.

**Status:** Identified / Not Reproduced where applicable

---

# 4. Primary Attack Chain

One complete attack chain was demonstrated from remote initial access to effective root privileges.

```text
Remote Attacker
      │
      ▼
distccd :3632
CVE-2004-2687
      │
      ▼
Remote Command Execution
      │
      ▼
daemon
UID 1
      │
      ▼
Post-Exploitation Enumeration
      │
      ▼
SUID Binary Discovery
      │
      ▼
/usr/bin/nmap
Nmap 4.53
SUID root
      │
      ▼
Interactive Mode
      │
      ▼
!sh
      │
      ▼
uid=1(daemon)
euid=0(root)
      │
      ▼
Effective Root Privileges
```

## Attack Chain Analysis

The chain demonstrates the relationship between remote exploitation and local privilege escalation.

CVE-2004-2687 did not initially provide root privileges. Instead, it provided an unprivileged `daemon` shell.

Local post-exploitation enumeration subsequently revealed the unsafe Nmap SUID configuration.

Exploitation of that configuration converted the restricted foothold into effective root access.

This demonstrates that vulnerabilities should not always be evaluated independently.

A vulnerability providing limited initial access can become significantly more severe when combined with local security weaknesses.

**Attack Chain Result:** Effective Root Compromise

---

# 5. Independent Critical Attack Paths

The assessment identified multiple high-impact paths rather than relying on a single vulnerability.

### Path 1 — SMB

```text
Remote Attacker
      ↓
Samba
      ↓
CVE-2007-2447
      ↓
Remote Command Execution
      ↓
Root
```

**Result:** Direct root compromise

---

### Path 2 — distccd + SUID Nmap

```text
Remote Attacker
      ↓
CVE-2004-2687
      ↓
daemon
      ↓
SUID Nmap 4.53
      ↓
euid=0(root)
```

**Result:** Chained root compromise

---

### Path 3 — NFS

```text
Remote Client
      ↓
NFS Root Export
      ↓
rw
      +
no_root_squash
      ↓
Root-Level Filesystem Impact
```

**Result:** Critical remote filesystem exposure

---

### Path 4 — FTP

```text
Remote Attacker
      ↓
vsftpd 2.3.4
      ↓
CVE-2011-2523
      ↓
Remote Command Execution
```

**Result:** Independent remote command execution vector

---

# 6. Confidentiality Impact

Multiple findings affected the confidentiality of the target.

The strongest demonstrated examples were:

* Effective root access through SUID Nmap.
* Successful access to `/etc/shadow` after privilege escalation.
* Remote access to the NFS root filesystem.
* Anonymous FTP access.
* Anonymous SMB access.
* TikiWiki information disclosure.
* PHP configuration disclosure.

### Assessment

The assessment demonstrated that unauthorized users could obtain access ranging from service-level information disclosure to protected root-level resources.

**Overall Confidentiality Impact: Critical**

---

# 7. Integrity Impact

Multiple confirmed findings provided the capability to execute commands or modify system resources.

The strongest demonstrated examples were:

* Root remote command execution through Samba.
* Remote command execution through distccd.
* Remote command execution through vsftpd.
* NFS root filesystem write access.
* Effective root privileges through SUID Nmap.

The NFS assessment specifically confirmed write access and preservation of root ownership through `no_root_squash`.

### Assessment

Successful exploitation could provide extensive control over system resources and therefore presents a critical integrity risk.

**Overall Integrity Impact: Critical**

---

# 8. Availability Impact

The assessment did not intentionally disrupt services or perform destructive actions.

However, multiple findings resulted in root-level or command-execution capabilities.

Such privileges provide the technical capability to affect system services, processes, files, and overall operation.

### Assessment

Availability impact was not directly exercised during the laboratory, but the demonstrated level of privilege indicates significant potential availability impact.

**Overall Availability Risk: Critical**

---

# 9. Risk Classification

The confirmed findings can be grouped by severity as follows:

| Finding                            | Severity | Status         |
| ---------------------------------- | :------: | -------------- |
| Samba CVE-2007-2447                | Critical | Confirmed      |
| distccd CVE-2004-2687              | Critical | Confirmed      |
| vsftpd CVE-2011-2523               | Critical | Confirmed      |
| NFS Root Export + `no_root_squash` | Critical | Confirmed      |
| SUID Nmap 4.53                     | Critical | Confirmed      |
| Anonymous FTP                      |   High   | Confirmed      |
| Anonymous SMB Access               |   High   | Confirmed      |
| SMBv1 Enabled                      |   High   | Confirmed      |
| SMB Signing Disabled               |  Medium  | Confirmed      |
| TikiWiki Information Disclosure    |  Medium  | Confirmed      |
| PHP Information Disclosure         |  Medium  | Confirmed      |
| Historical TikiWiki CVEs           |   High   | Not Reproduced |

### Assessment

The presence of multiple independently exploitable critical findings substantially increases overall risk.

Compromise does not depend on a single vulnerability or attack path.

An attacker could potentially select among several exposed services to obtain initial access or privileged impact.

**Overall Risk Rating: Critical**

---

# 10. Remediation Priorities

Remediation should prioritize findings according to demonstrated impact and their role in the confirmed attack chains.

## Priority 1 — Remove Direct Remote Compromise Paths

Immediately address the services associated with confirmed remote command execution:

* Samba affected by CVE-2007-2447.
* distccd affected by CVE-2004-2687.
* vsftpd 2.3.4 associated with CVE-2011-2523.

Legacy vulnerable software should be removed, disabled, isolated, or upgraded to supported secure versions.

**Priority:** Critical

---

## Priority 2 — Correct NFS Configuration

The root filesystem should not be exported broadly to remote clients.

The configuration:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

should be removed or redesigned.

In particular:

* Avoid exporting `/`.
* Restrict exports to explicitly authorized hosts.
* Avoid unnecessary write permissions.
* Remove `no_root_squash` unless a specific trusted requirement exists.
* Apply least-privilege filesystem access.

**Priority:** Critical

---

## Priority 3 — Remove SUID from Nmap

Nmap should not require SUID root privileges for normal use.

The SUID permission on:

```text
/usr/bin/nmap
```

should be removed.

Legacy Nmap installations should also be replaced with a supported version where appropriate.

**Priority:** Critical

---

## Priority 4 — Restrict Anonymous Access

Anonymous access should be reviewed and disabled where it is not explicitly required.

This applies to:

* FTP
* SMB

Access should require appropriate authentication and authorization.

**Priority:** High

---

## Priority 5 — Harden SMB

SMB configuration should be reviewed to:

* Disable unnecessary anonymous access.
* Remove SMBv1 support.
* Enable appropriate SMB signing protections.
* Restrict exposed shares.
* Upgrade legacy Samba installations.

**Priority:** High

---

## Priority 6 — Reduce HTTP Information Exposure

Unnecessary diagnostic, administrative, test, and legacy application resources should not be publicly exposed.

Relevant resources include:

```text
/phpinfo.php
/phpMyAdmin/
/dav/
/test/
/tikiwiki/
```

Server and application configuration should minimize unnecessary information disclosure.

Legacy web technologies should be upgraded or removed when no longer required.

**Priority:** High

---

## Priority 7 — Reduce Network Attack Surface

Services that are not required should be disabled or restricted.

Metasploitable2 exposed 30 TCP ports, including multiple legacy remote administration and application protocols.

Network access controls should restrict management, database, file-sharing, RPC, and other sensitive services to authorized systems only.

**Priority:** High

---

# 11. Overall Security Posture

The assessed Metasploitable2 system presented a critically insecure security posture.

The conclusion is based on demonstrated results rather than software age alone.

The assessment confirmed:

* Multiple remote command execution vulnerabilities.
* Direct remote root compromise through SMB.
* An unprivileged remote foothold through distccd.
* Successful local privilege escalation from `daemon` to effective root.
* Remote exposure of the root filesystem through NFS.
* Preservation of remote root identity through `no_root_squash`.
* Anonymous service access.
* Legacy protocol exposure.
* Application and configuration information disclosure.

Most importantly, several findings could be combined into complete attack chains.

The demonstrated distccd-to-Nmap chain showed that an attacker obtaining only limited initial privileges could subsequently achieve effective root access through local misconfiguration.

At the same time, Samba demonstrated that alternative attack paths could bypass this intermediate stage and provide root-level access directly.

This combination of broad network exposure, independently exploitable vulnerabilities, weak access controls, legacy technologies, and local privilege escalation opportunities means that compromise of the target does not depend on a single point of failure.

---

# Final Assessment

| Category                        | Assessment  |
| ------------------------------- | ----------- |
| Network Attack Surface          | Critical    |
| Remote Code Execution Risk      | Critical    |
| Authentication / Access Control | High        |
| Filesystem Security             | Critical    |
| Local Privilege Escalation      | Critical    |
| Information Disclosure          | Medium–High |
| Confidentiality Impact          | Critical    |
| Integrity Impact                | Critical    |
| Availability Risk               | Critical    |
| Overall Security Posture        | Critical    |

The laboratory successfully demonstrated the progression from network reconnaissance to vulnerability identification, exploitation, post-exploitation, and privilege escalation.

The most significant demonstrated attack chain was:

```text
Remote Access
     ↓
distccd CVE-2004-2687
     ↓
daemon
     ↓
Post-Exploitation Enumeration
     ↓
SUID Nmap 4.53
     ↓
euid=0(root)
     ↓
Effective Root Compromise
```

Additional independent critical attack paths were confirmed through Samba, vsftpd, and NFS.

**Final Status: Assessment Completed**

**Overall Risk: Critical**
