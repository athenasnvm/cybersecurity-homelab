# Technical Security Findings

## Overview

This document consolidates the principal security findings identified and validated during the Metasploitable2 security assessment.

The findings are based on observed laboratory evidence collected throughout the assessment.

A distinction is maintained between:

```text
Identified
    ↓
Condition or potential weakness discovered

Validated
    ↓
Technical testing confirmed the condition

Exploited
    ↓
The weakness was successfully used

Impact Confirmed
    ↓
The resulting security consequence was directly observed
```

Software versions and historical vulnerability applicability alone are not considered proof of successful exploitation.

---

# Findings Summary

| ID   | Finding                                     | Attack Surface | Validation Status | Demonstrated Impact                 |
| ---- | ------------------------------------------- | -------------- | ----------------- | ----------------------------------- |
| F-01 | vsftpd 2.3.4 Backdoor                       | FTP            | Exploited         | Root-level remote access            |
| F-02 | Samba Username Map Script Command Execution | SMB            | Exploited         | Root command shell                  |
| F-03 | DistCC Remote Command Execution             | DistCC         | Exploited         | Shell as `daemon`                   |
| F-04 | Insecure NFS Root Export                    | NFS            | Validated         | Privileged remote filesystem access |
| F-05 | SUID Nmap Privilege Escalation              | Local System   | Exploited         | Effective root execution            |
| F-06 | HTTP Technology & Version Disclosure        | HTTP           | Validated         | Information disclosure              |
| F-07 | Exposed HTTP Resources                      | HTTP           | Validated         | Increased attack surface            |
| F-08 | Exposed WebDAV Functionality                | HTTP / WebDAV  | Validated         | Extended HTTP attack surface        |
| F-09 | Public phpMyAdmin Interface                 | HTTP           | Validated         | Administrative interface exposure   |
| F-10 | TikiWiki Information Disclosure             | HTTP           | Validated         | Backend information disclosure      |
| F-11 | Exposed TikiWiki Installation Interface     | HTTP           | Validated         | Configuration interface exposure    |
| F-12 | Public Test Resources & Directory Listing   | HTTP           | Validated         | Information / resource exposure     |

---

# F-01 — vsftpd 2.3.4 Backdoor

## Category

```text
Remote Code Execution
```

## Affected Service

```text
FTP
21/tcp
vsftpd 2.3.4
```

## Vulnerability

```text
CVE-2011-2523
```

## Status

**Confirmed — Successfully Exploited**

---

## Description

The target FTP service was identified as running vsftpd version `2.3.4`.

During vulnerability research, CVE-2011-2523 was identified as a potential attack path.

Controlled exploitation was subsequently performed using:

```text
exploit/unix/ftp/vsftpd_234_backdoor
```

The backdoor was successfully triggered.

Metasploit reported that a remote session had been established.

---

## Validation

The resulting Meterpreter session reported:

```text
Server username: root
```

A native shell was subsequently opened and the execution context was independently validated.

```text
whoami
→ root
```

```text
id
→ uid=0(root) gid=0(root)
```

---

## Demonstrated Impact

Successful exploitation provided direct root-level remote access.

The attack chain was:

```text
FTP
 ↓
vsftpd 2.3.4
 ↓
CVE-2011-2523
 ↓
Backdoor Triggered
 ↓
Remote Session
 ↓
root
 ↓
UID 0
```

No separate privilege-escalation step was required.

### Impact

**Full system compromise.**

---

## Evidence

Primary exploitation evidence:

```text
06-exploitation/ftp/scans/cve-2011-2523-validation.txt
```

Supporting anonymous FTP evidence:

```text
06-exploitation/ftp/scans/ftp-anonymous.txt
```

---

# F-02 — Samba Username Map Script Command Execution

## Category

```text
Remote Code Execution
```

## Affected Service

```text
SMB
139/tcp
445/tcp
Samba 3.0.20-Debian
```

## Vulnerability

```text
CVE-2007-2447
```

## Status

**Confirmed — Successfully Exploited**

---

## Description

The Samba service exposed by the target was identified as vulnerable to username map script command execution.

The relevant exploitation path was tested using:

```text
exploit/multi/samba/usermap_script
```

against the SMB service.

The exploitation attempt successfully established a remote command shell.

---

## Validation

The resulting shell returned:

```text
whoami
→ root
```

and:

```text
id
→ uid=0(root) gid=0(root)
```

Target identity was further validated using:

```text
hostname
uname -a
pwd
```

The shell was confirmed to be executing on the intended Metasploitable2 target.

---

## Demonstrated Impact

The vulnerability provided unauthenticated remote command execution with root privileges.

The demonstrated attack chain was:

```text
SMB
 ↓
Samba 3.0.20-Debian
 ↓
CVE-2007-2447
 ↓
Remote Command Execution
 ↓
Command Shell
 ↓
root
 ↓
UID 0
```

### Impact

**Full system compromise.**

---

## Evidence

```text
06-exploitation/smb/scans/cve-2007-2447-rce.txt
```

---

# F-03 — DistCC Remote Command Execution

## Category

```text
Remote Code Execution
```

## Affected Service

```text
DistCC
3632/tcp
```

## Vulnerability

```text
CVE-2004-2687
```

## Status

**Confirmed — Successfully Exploited**

---

## Description

The exposed DistCC daemon was identified as vulnerable to remote command execution.

Controlled exploitation was performed using:

```text
exploit/unix/misc/distcc_exec
```

An initial payload attempted to establish a reverse shell using `/dev/tcp`.

The target environment returned errors and no session was created.

The failed attempt was retained as part of the assessment evidence rather than being represented as successful exploitation.

A compatible Perl-based reverse-shell payload was subsequently selected.

```text
cmd/unix/reverse_perl
```

The second attempt successfully established a remote command shell.

---

## Validation

The resulting session returned:

```text
whoami
→ daemon
```

and:

```text
id
→ uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

Additional validation identified:

```text
Hostname: metasploitable
Kernel: Linux 2.6.24-16-server
Working Directory: /tmp
```

---

## Demonstrated Impact

The vulnerability allowed unauthenticated remote command execution.

Unlike the FTP and SMB attack paths, the resulting shell was not initially privileged.

```text
DistCC
   ↓
CVE-2004-2687
   ↓
Remote Command Execution
   ↓
daemon
   ↓
UID 1
```

### Impact

**Remote command execution with unprivileged system access.**

The resulting access later served as the starting point for post-exploitation enumeration and local privilege escalation.

---

## Evidence

Initial failed attempt:

```text
06-exploitation/distccd/scans/cve-2004-2687-exploit-attempt.txt
```

Successful exploitation:

```text
06-exploitation/distccd/scans/cve-2004-2687-rce.txt
```

---

# F-04 — Insecure NFS Root Export

## Category

```text
Security Misconfiguration
Privileged Remote Filesystem Access
```

## Affected Service

```text
NFS
```

## CVE

```text
Not Applicable
```

## Status

**Confirmed — Security Impact Validated**

---

## Description

The target exposed its root filesystem through NFS.

The observed configuration was:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

This configuration combines several security-sensitive conditions.

### Root Filesystem Export

```text
/
```

The entire target filesystem was exported.

### Wildcard Client Access

```text
*
```

The export was available to unrestricted client addresses within the reachable network context.

### Read / Write Access

```text
rw
```

Clients were permitted to perform write operations.

### Root Squashing Disabled

```text
no_root_squash
```

UID `0` from the remote NFS client was not remapped to an unprivileged anonymous identity.

---

## Validation

The target export was mounted from the assessment host.

A temporary file was created through the mounted filesystem as root on the client.

The resulting remote file was:

```text
-rw-r--r-- 1 root root ... /tmp/root_test
```

The ownership:

```text
root:root
```

confirmed that remote UID `0` was preserved.

The temporary file was removed after validation.

---

## Demonstrated Impact

The configuration provides privileged remote filesystem access to the target's root filesystem.

The demonstrated chain was:

```text
NFS
 ↓
Export /
 ↓
Wildcard Access
 ↓
Read / Write
 ↓
no_root_squash
 ↓
Remote UID 0 Preserved
 ↓
root:root File Creation
```

### Impact

**Privileged remote filesystem access.**

The validation did not establish a remote command shell through NFS, and the finding is therefore not represented as NFS-based remote code execution.

---

## Evidence

Export enumeration:

```text
06-exploitation/nfs/scans/nfs-export-enum.txt
```

Configuration:

```text
06-exploitation/nfs/scans/nfs-config.txt
```

Primary validation:

```text
06-exploitation/nfs/scans/nfs-no-root-squash.txt
```

Supporting filesystem evidence:

```text
06-exploitation/nfs/scans/nfs-filesystem-enum.txt
06-exploitation/nfs/scans/nfs-sensitive-file-enum.txt
06-exploitation/nfs/scans/rpc-enum.txt
```

---

# F-05 — SUID Nmap Privilege Escalation

## Category

```text
Local Privilege Escalation
Unsafe SUID Configuration
```

## Affected Binary

```text
/usr/bin/nmap
```

## Version

```text
Nmap 4.53
```

## Status

**Confirmed — Successfully Exploited**

---

## Description

Post-exploitation enumeration performed from the unprivileged DistCC shell identified multiple SUID-enabled binaries.

Among them:

```text
/usr/bin/nmap
```

was configured with SUID permissions and owned by root.

The installed version was:

```text
Nmap 4.53
```

The legacy Nmap version supported interactive functionality capable of executing commands.

Because the binary was SUID root, the resulting commands could execute with an effective UID of `0`.

---

## Initial Access Context

The privilege-escalation test began from:

```text
User: daemon
UID: 1
GID: 1
Privilege: Unprivileged
```

This access had previously been obtained through CVE-2004-2687.

---

## Validation

After using the identified Nmap execution path:

```text
whoami
→ root
```

The identity information returned:

```text
uid=1(daemon) gid=1(daemon) euid=0(root) groups=1(daemon)
```

This demonstrates an important distinction:

```text
Real UID:      1 (daemon)
Effective UID: 0 (root)
```

The original account identity remained `daemon`, but commands could execute with effective root privileges.

---

## Demonstrated Impact

The condition allowed an unprivileged compromised account to obtain root-level execution.

The complete chain was:

```text
CVE-2004-2687
      ↓
daemon
UID 1
      ↓
Local Enumeration
      ↓
SUID Nmap 4.53
      ↓
Effective UID 0
      ↓
Root-Level Execution
```

### Impact

**Local privilege escalation to effective root privileges.**

---

# F-06 — HTTP Technology and Version Disclosure

## Category

```text
Information Disclosure
```

## Affected Service

```text
HTTP
80/tcp
```

## Status

**Confirmed**

---

## Description

HTTP responses disclosed detailed information about the server and runtime environment.

Observed information included:

```text
Apache/2.2.8 (Ubuntu) DAV/2
PHP/5.2.4-2ubuntu5.10
```

A publicly accessible PHP information resource was also identified.

---

## Demonstrated Impact

The information allows unauthenticated clients to fingerprint the server environment and identify specific technologies and versions.

This can reduce the effort required for vulnerability research.

### Impact

**Technology fingerprinting and information disclosure.**

No remote command execution was demonstrated from this finding.

---

# F-07 — Excessive HTTP Resource Exposure

## Category

```text
Attack Surface Exposure
```

## Status

**Confirmed**

---

## Description

HTTP enumeration identified multiple publicly reachable resources, including:

```text
/tikiwiki/
/test/
/phpinfo.php
/phpMyAdmin/
/doc/
/icons/
/index/
```

The target therefore exposed diagnostic functionality, administrative interfaces, legacy applications, test resources, and directory listings through a single HTTP service.

---

## Demonstrated Impact

The number and type of publicly reachable resources significantly increase the available attack surface.

Individual high-value resources were assessed separately in subsequent findings.

### Impact

**Increased HTTP attack surface.**

---

# F-08 — Exposed WebDAV Functionality

## Category

```text
Service / Functionality Exposure
```

## Endpoint

```text
/dav/
```

## Status

**Confirmed**

---

## Description

The `/dav/` endpoint advertised WebDAV support.

Observed headers included:

```text
DAV: 1,2
MS-Author-Via: DAV
```

The endpoint advertised methods including:

```text
DELETE
PROPFIND
PROPPATCH
COPY
MOVE
LOCK
UNLOCK
```

---

## PROPFIND Validation

A tested PROPFIND request returned:

```text
HTTP/1.1 403 Forbidden
```

with the explanation that requests using:

```text
Depth: infinity
```

were not allowed for `/dav/`.

---

## Demonstrated Impact

The endpoint exposes functionality beyond standard HTTP retrieval and therefore increases the available attack surface.

However:

```text
Advertised Method
       ≠
Unauthorized Operation Confirmed
```

The assessment did not demonstrate unauthorized file creation, modification, movement, or deletion through WebDAV.

### Impact

**Extended HTTP/WebDAV attack surface.**

---

## Evidence

```text
06-exploitation/http/scans/webdav-options.txt
06-exploitation/http/scans/webdav-propfind.txt
06-exploitation/http/scans/webdav-enumeration.txt
```

---

# F-09 — Public phpMyAdmin Interface

## Category

```text
Administrative Interface Exposure
```

## Endpoint

```text
/phpMyAdmin/
```

## Status

**Confirmed**

---

## Description

The phpMyAdmin database administration interface was publicly reachable through the HTTP service.

The endpoint returned a valid application response and displayed an authentication interface containing:

```text
Username
Password
```

fields.

---

## Demonstrated Impact

The finding confirms that a database administration application is exposed to unauthenticated network clients.

However, the assessment did not demonstrate:

```text
Authentication Bypass
Database Authentication
Database Access
Administrative Access
Remote Code Execution
```

### Impact

**Administrative interface exposure and increased attack surface.**

---

## Evidence

```text
06-exploitation/http/scans/phpmyadmin.txt
06-exploitation/http/scans/phpmyadmin-headers.txt
06-exploitation/http/scans/phpmyadmin-version.txt
```

---

# F-10 — TikiWiki Information Disclosure

## Category

```text
Information Disclosure
Legacy Application Exposure
```

## Application

```text
TikiWiki 1.9.5
```

## Status

**Confirmed**

---

## Description

The TikiWiki application exposed detailed backend error information to unauthenticated HTTP clients.

Observed information included:

```text
Database Technology: MySQL
Database Host: localhost
Database User: root
```

The application additionally disclosed an internal filesystem path:

```text
/var/www/tikiwiki/lib/adodb/drivers/adodb-mysql.inc.php
```

and associated source-code information.

The application version was independently exposed through the installation interface as:

```text
Tiki installer v1.9.5
```

---

## Demonstrated Impact

The application reveals information about:

```text
Database Technology
Database Host
Database Username
Application Version
Installation Path
Directory Structure
Internal Source Files
```

This information could assist further attack planning.

The database password itself was not disclosed.

Database access was not obtained.

### Impact

**Detailed backend information disclosure.**

---

## Evidence

Relevant evidence includes:

```text
06-exploitation/http/scans/tikiwiki-html.txt
06-exploitation/http/scans/tikiwiki-version.txt
06-exploitation/http/scans/tiki-install-version.txt
```

---

# F-11 — Exposed TikiWiki Installation Interface

## Category

```text
Configuration Interface Exposure
```

## Endpoint

```text
/tikiwiki/tiki-install.php
```

## Status

**Confirmed**

---

## Description

The TikiWiki installation interface remained publicly accessible.

The endpoint returned:

```text
HTTP/1.1 200 OK
```

and identified itself as:

```text
Tiki installer v1.9.5
```

The exposed form contained configuration fields for:

```text
Database Type
Database Host
Database User
Database Password
Database Name
```

---

## Demonstrated Impact

The exposure makes application installation and database configuration functionality reachable by unauthenticated HTTP clients.

However, the assessment did not submit database credentials or demonstrate a successful unauthorized configuration change.

Therefore:

```text
Installer Exposure: Confirmed
Configuration Form Exposure: Confirmed

Unauthorized Configuration Modification:
Not Demonstrated

Application Takeover:
Not Demonstrated
```

### Impact

**Sensitive application configuration interface exposure.**

---

## Evidence

```text
06-exploitation/http/scans/tiki-install.txt
06-exploitation/http/scans/tiki-install-headers.txt
06-exploitation/http/scans/tiki-install-version.txt
06-exploitation/http/scans/tiki-install-form.txt
```

---

# F-12 — Public Test Resources and Directory Listing

## Category

```text
Information / Resource Exposure
```

## Endpoint

```text
/test/
```

## Status

**Confirmed**

---

## Description

The target exposed a public test directory through the HTTP service.

The endpoint returned:

```text
HTTP/1.1 200 OK
```

with directory indexing enabled.

The listing exposed:

```text
/test/testoutput/
```

---

## Demonstrated Impact

Public test and development resources can expose information or functionality that was not intended for production access.

The tested directory confirmed directory indexing and resource exposure.

No sensitive file disclosure was demonstrated from this directory during the assessment.

### Impact

**Test resource exposure and directory listing.**

---

## Evidence

```text
06-exploitation/http/scans/http-test.txt
```

---

# Attack Path Analysis

Individual findings become more significant when their relationships are considered.

---

## Attack Path 1 — FTP to Root

```text
Network Access
     ↓
21/tcp
     ↓
vsftpd 2.3.4
     ↓
CVE-2011-2523
     ↓
Remote Session
     ↓
root
```

### Result

**Direct full system compromise.**

---

## Attack Path 2 — SMB to Root

```text
Network Access
     ↓
139/tcp
     ↓
Samba
     ↓
CVE-2007-2447
     ↓
Remote Command Shell
     ↓
root
```

### Result

**Direct full system compromise.**

---

## Attack Path 3 — DistCC to Local Privilege Escalation

```text
Network Access
     ↓
3632/tcp
     ↓
DistCC
     ↓
CVE-2004-2687
     ↓
daemon
     ↓
Local Enumeration
     ↓
SUID Nmap 4.53
     ↓
euid=0(root)
```

### Result

**Unprivileged remote compromise chained with local privilege escalation to effective root.**

---

## Attack Path 4 — NFS Privileged Filesystem Access

```text
Network Access
     ↓
NFS
     ↓
Export /
     ↓
rw
     ↓
no_root_squash
     ↓
Remote UID 0 Preserved
     ↓
Privileged Filesystem Operations
```

### Result

**Remote privileged access to the target filesystem.**

---

# Combined Security Impact

The findings demonstrate that the target did not contain a single isolated weakness.

Instead:

```text
FTP ───────────────→ root

SMB ───────────────→ root

DistCC → daemon
           ↓
       SUID Nmap
           ↓
        euid=0

NFS ───────────────→ privileged filesystem access

HTTP ──────────────→ information and application exposure
```

This creates substantial redundancy from an attacker's perspective.

Even if one compromise path were removed, other high-impact paths would remain available.

---

# Validation Matrix

| Finding                 | Identified | Validated |     Exploited    | Root-Level Impact |
| ----------------------- | :--------: | :-------: | :--------------: | :---------------: |
| CVE-2011-2523           |      ✓     |     ✓     |         ✓        |         ✓         |
| CVE-2007-2447           |      ✓     |     ✓     |         ✓        |         ✓         |
| CVE-2004-2687           |      ✓     |     ✓     |         ✓        |         —         |
| NFS `no_root_squash`    |      ✓     |     ✓     |        N/A       |  Filesystem UID 0 |
| SUID Nmap               |      ✓     |     ✓     |         ✓        |  Effective UID 0  |
| HTTP Version Disclosure |      ✓     |     ✓     |        N/A       |         —         |
| WebDAV Exposure         |      ✓     |     ✓     | Not demonstrated |         —         |
| phpMyAdmin Exposure     |      ✓     |     ✓     | Not demonstrated |         —         |
| TikiWiki Disclosure     |      ✓     |     ✓     | Not demonstrated |         —         |
| TikiWiki Installer      |      ✓     |     ✓     | Not demonstrated |         —         |
| Test Directory          |      ✓     |     ✓     |        N/A       |         —         |

---

# Key Conclusions

The assessment established four particularly important technical conclusions.

## 1. Remote root compromise was possible

Both FTP and SMB independently provided remote paths to UID `0`.

```text
FTP → root
SMB → root
```

---

## 2. Unprivileged access could be escalated

DistCC initially provided only:

```text
daemon
UID 1
```

but local enumeration identified a separate privilege-escalation path:

```text
SUID Nmap
→ euid=0(root)
```

This demonstrates the importance of evaluating the complete attack chain rather than only the privilege level of the initial exploit.

---

## 3. Root-level impact did not always require a shell

NFS demonstrated that severe security impact can exist without obtaining an interactive shell.

The combination of:

```text
/
+
rw
+
*
+
no_root_squash
```

allowed privileged remote filesystem operations.

---

## 4. Exposure was not represented as exploitation

HTTP testing identified numerous security weaknesses, but no HTTP-based shell was obtained.

Therefore:

```text
HTTP Exposure
      ≠
HTTP Remote Code Execution
```

This distinction is intentionally maintained throughout the final report.

---

# Final Technical Assessment

The target contains multiple independent weaknesses capable of producing severe security consequences.

Confirmed outcomes include:

```text
Remote Code Execution
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

The most significant issue is not any individual CVE.

It is the coexistence of multiple remotely accessible vulnerable services and insecure local configurations.

The target therefore presents multiple independent paths toward administrative compromise.

Remediation should prioritize removal of direct remote root-compromise paths while simultaneously reducing exposed services and correcting local privilege and filesystem configuration weaknesses.

Detailed corrective actions are documented in:

```text
remediation.md
```
