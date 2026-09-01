# Evidence Summary

## Overview

This document maps the principal findings from the final security assessment to the technical evidence collected during the laboratory.

The purpose of this file is not to reproduce the complete raw output.

Instead, it provides a reference between:

```text
Security Finding
      ↓
Validation Evidence
      ↓
Raw Scan / Output
      ↓
Supporting Screenshot
```

where supporting screenshots exist.

Raw evidence should remain unchanged after collection.

---

# Evidence Principles

The assessment followed several evidence-handling principles.

## 1. Raw Output Is Primary Technical Evidence

Files stored under:

```text
scans/
```

preserve terminal, Nmap, Metasploit, HTTP, SMB, NFS, and other tool output.

Where a raw text file contains the complete result, it is generally considered stronger and more useful than a screenshot of the same information.

---

## 2. Screenshots Are Supporting Evidence

Screenshots stored under:

```text
evidence/
```

are used when they provide useful visual confirmation.

They are not required for every command.

A finding is not considered unsupported simply because no screenshot exists if the corresponding raw output was preserved.

---

## 3. Missing Evidence Is Not Recreated

Evidence that was not preserved during the original laboratory session should not be recreated solely to make the repository appear more complete.

The report only references evidence that actually exists in the project.

---

## 4. Failed Attempts Are Retained When Relevant

Failed exploitation or validation attempts may be retained when they provide useful technical context.

For example:

```text
DistCC Initial Payload
        ↓
Failed
        ↓
Target Error Preserved
        ↓
Payload Changed
        ↓
Successful Session
```

This demonstrates troubleshooting and preserves the actual laboratory workflow.

---

# F-01 — vsftpd 2.3.4 Backdoor

## Finding

```text
CVE-2011-2523
VSFTPD 2.3.4 Backdoor Command Execution
```

## Demonstrated Impact

```text
Remote Code Execution
Meterpreter Session
Native Shell
root
UID 0
GID 0
```

---

## Primary Evidence

```text
06-exploitation/ftp/scans/cve-2011-2523-validation.txt
```

### Evidence Contains

* Target identification.
* FTP service identification.
* CVE reference.
* Metasploit module.
* Successful backdoor activation.
* Meterpreter session establishment.
* `getuid`.
* `sysinfo`.
* `pwd`.
* Native shell creation.
* `whoami`.
* `id`.
* `hostname`.
* `uname -a`.

The most important validation results are:

```text
Server username: root
```

and:

```text
uid=0(root) gid=0(root)
```

---

## Supporting Evidence

Anonymous FTP validation:

```text
06-exploitation/ftp/scans/ftp-anonymous.txt
```

This demonstrates that anonymous authentication was independently available through the FTP service.

---

## Screenshot Evidence

If present:

```text
06-exploitation/ftp/evidence/
```

Only original screenshots should be retained.

---

# F-02 — Samba Username Map Script Command Execution

## Finding

```text
CVE-2007-2447
Samba Username Map Script Command Execution
```

## Demonstrated Impact

```text
Remote Command Execution
Command Shell
root
UID 0
GID 0
```

---

## Primary Evidence

```text
06-exploitation/smb/scans/cve-2007-2447-rce.txt
```

### Evidence Contains

* Target IP.
* Target port.
* CVE identification.
* Metasploit module.
* Successful command shell establishment.
* `whoami`.
* `id`.
* `hostname`.
* `uname -a`.
* `pwd`.

The principal privilege evidence is:

```text
whoami
root
```

and:

```text
uid=0(root) gid=0(root)
```

---

## Supporting Research Evidence

Configuration and protocol evidence collected during vulnerability research includes:

```text
05-vulnerability-research/scans/smb-config.txt
```

and additional SMB enumeration outputs.

The configuration evidence supports the applicability of the username map script vulnerability.

---

## Screenshot Evidence

If present:

```text
06-exploitation/smb/evidence/smb-rce.png
06-exploitation/smb/evidence/smb-rce-id.png
```

---

# F-03 — DistCC Remote Command Execution

## Finding

```text
CVE-2004-2687
DistCC Daemon Command Execution
```

## Demonstrated Impact

```text
Remote Command Execution
Command Shell
daemon
UID 1
GID 1
```

---

## Initial Failed Attempt

```text
06-exploitation/distccd/scans/cve-2004-2687-exploit-attempt.txt
```

### Evidence Contains

The first exploitation attempt reached the target but failed to create a session.

The target returned `/dev/tcp` related errors.

This evidence explains why the original payload was replaced.

---

## Primary Successful Evidence

```text
06-exploitation/distccd/scans/cve-2004-2687-rce.txt
```

### Evidence Contains

* Successful reverse connection.
* Command shell establishment.
* `whoami`.
* `id`.
* `hostname`.
* `uname -a`.
* `pwd`.

The resulting security context was:

```text
daemon
UID 1
GID 1
```

---

## Evidence Significance

Together, the two files demonstrate:

```text
Initial Exploit Attempt
        ↓
Payload Compatibility Failure
        ↓
Payload Adjustment
        ↓
Successful Remote Command Shell
```

This provides evidence of both exploitation and troubleshooting.

---

# F-04 — Insecure NFS Root Export

## Finding

```text
Root Filesystem Export
Wildcard Access
rw
no_root_squash
```

## Demonstrated Impact

```text
Remote Root Filesystem Access
Read Access
Write Access
Remote UID 0 Preservation
root:root File Creation
```

---

## Export Enumeration

```text
06-exploitation/nfs/scans/nfs-export-enum.txt
```

### Evidence Contains

```text
/ *
```

This demonstrates that the root filesystem is exported to wildcard clients.

---

## NFS Configuration

```text
06-exploitation/nfs/scans/nfs-config.txt
```

### Evidence Contains

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

This is the primary configuration evidence for the insecure NFS export.

---

## Root Identity Preservation

```text
06-exploitation/nfs/scans/nfs-no-root-squash.txt
```

### Evidence Contains

The temporary file created through the NFS mount was owned by:

```text
root root
```

This confirms preservation of remote UID `0`.

---

## Filesystem Exposure

```text
06-exploitation/nfs/scans/nfs-filesystem-enum.txt
```

This output demonstrates that the mounted export corresponds to the target root filesystem.

Visible directories include system locations such as:

```text
/etc
/home
/root
/usr
/var
```

---

## Sensitive Location Enumeration

```text
06-exploitation/nfs/scans/nfs-sensitive-file-enum.txt
```

This demonstrates visibility into security-relevant filesystem locations such as:

```text
/etc/passwd
/root
```

---

## RPC Evidence

```text
06-exploitation/nfs/scans/rpc-enum.txt
```

This provides supporting evidence regarding RPC and NFS service availability.

---

# F-05 — SUID Nmap Privilege Escalation

## Finding

```text
Root-Owned SUID Nmap
Nmap 4.53
```

## Initial Context

```text
daemon
UID 1
Unprivileged
```

## Demonstrated Impact

```text
Effective Root Privileges
euid=0(root)
```

---

## Supporting Enumeration

Relevant post-exploitation evidence was collected during:

```text
07-post-exploitation/
```

including SUID enumeration and direct inspection of:

```text
/usr/bin/nmap
```

The binary was identified as root-owned with SUID permissions.

The installed version was:

```text
Nmap 4.53
```

---

## Privilege Escalation Validation

Relevant evidence is preserved under:

```text
08-privilege-escalation/
```

The principal result was:

```text
whoami
root
```

combined with:

```text
uid=1(daemon) gid=1(daemon) euid=0(root) groups=1(daemon)
```

---

## Evidence Significance

The identity output demonstrates the distinction between:

```text
Real UID
→ daemon

Effective UID
→ root
```

This is the primary technical evidence supporting the privilege-escalation finding.

---

# F-06 — HTTP Technology and Version Disclosure

## Finding

```text
Apache / PHP Version Disclosure
Public PHP Information
```

## Demonstrated Impact

```text
Technology Fingerprinting
Information Disclosure
```

---

## Primary Evidence

```text
06-exploitation/http/scans/http-headers.txt
```

### Evidence Contains

HTTP headers identifying:

```text
Apache/2.2.8 (Ubuntu) DAV/2
PHP/5.2.4-2ubuntu5.10
```

---

## Supporting Enumeration

```text
06-exploitation/http/scans/http-enum.txt
```

This demonstrates the existence of multiple exposed HTTP resources.

---

# F-07 — Excessive HTTP Resource Exposure

## Finding

Multiple publicly reachable HTTP resources.

## Primary Evidence

```text
06-exploitation/http/scans/http-enum.txt
```

### Identified Resources Include

```text
/tikiwiki/
/test/
/phpinfo.php
/phpMyAdmin/
/doc/
/icons/
/index/
```

---

## Demonstrated Impact

```text
Increased Attack Surface
Diagnostic Resource Exposure
Administrative Interface Exposure
Legacy Application Exposure
```

---

# F-08 — WebDAV Exposure

## Finding

```text
WebDAV Enabled
Extended DAV Methods
```

## Primary Evidence

```text
06-exploitation/http/scans/webdav-options.txt
```

### Evidence Contains

```text
DAV: 1,2
```

and advertised methods including:

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

## PROPFIND Evidence

```text
06-exploitation/http/scans/webdav-propfind.txt
```

The tested request returned:

```text
403 Forbidden
```

because the request used an unsupported depth condition.

This output is important because it prevents the report from incorrectly claiming unrestricted WebDAV enumeration.

---

## Nmap Supporting Evidence

```text
06-exploitation/http/scans/webdav-enumeration.txt
```

No additional exploitable WebDAV capability was demonstrated by this scan.

---

# F-09 — Public phpMyAdmin Interface

## Finding

```text
phpMyAdmin Administration Interface Exposed
```

## Primary Evidence

```text
06-exploitation/http/scans/phpmyadmin.txt
```

The returned page contains the phpMyAdmin login interface.

---

## Supporting Evidence

```text
06-exploitation/http/scans/phpmyadmin-headers.txt
06-exploitation/http/scans/phpmyadmin-version.txt
```

---

## Demonstrated Impact

The evidence confirms:

```text
phpMyAdmin Installed
Interface Publicly Reachable
Authentication Interface Exposed
```

The evidence does not demonstrate:

```text
Authentication Bypass
Database Access
Administrative Access
```

---

# F-10 — TikiWiki Information Disclosure

## Finding

```text
TikiWiki 1.9.5
Database / Configuration Information Disclosure
```

## Primary Evidence

```text
06-exploitation/http/scans/tiki-install-version.txt
```

### Evidence Contains

The exposed installer identifies:

```text
Tiki installer v1.9.5
```

and exposes a database connection error involving:

```text
root@localhost
```

---

## Application Response Evidence

```text
06-exploitation/http/scans/tikiwiki-html.txt
```

This demonstrates that the application is improperly configured and cannot establish its backend database connection.

---

## Headers

```text
06-exploitation/http/scans/tikiwiki-headers.txt
```

Provides supporting HTTP response information.

---

## Demonstrated Information

The combined evidence supports disclosure of:

```text
Application Version
Database Technology
Database Username
Database Host
Internal Application Path
Database Driver Information
```

---

# F-11 — Exposed TikiWiki Installation Interface

## Finding

```text
/tikiwiki/tiki-install.php
Publicly Accessible
```

## Primary Evidence

```text
06-exploitation/http/scans/tiki-install.txt
```

---

## Header Evidence

```text
06-exploitation/http/scans/tiki-install-headers.txt
```

Confirms that the installation endpoint returned an HTTP response successfully.

---

## Version Evidence

```text
06-exploitation/http/scans/tiki-install-version.txt
```

Confirms:

```text
Tiki installer v1.9.5
```

---

## Installation Form Evidence

```text
06-exploitation/http/scans/tiki-install-form.txt
```

### Evidence Contains

Configuration fields for:

```text
Database Type
Database Host
Database User
Database Password
Database Name
```

---

## Demonstrated Impact

The installer and its configuration form are exposed.

The evidence does not demonstrate that an unauthorized configuration change was successfully performed.

---

# F-12 — Public Test Directory

## Finding

```text
/test/
Public Directory Listing
```

## Primary Evidence

```text
06-exploitation/http/scans/http-test.txt
```

### Evidence Contains

```text
HTTP/1.1 200 OK
```

and directory indexing exposing:

```text
testoutput/
```

---

## Demonstrated Impact

```text
Test Resource Exposure
Directory Listing
```

No sensitive file disclosure was demonstrated from the tested directory.

---

# Attack Path Evidence Mapping

The most important attack chains can be mapped directly to their supporting files.

---

## Attack Path A — FTP to Root

```text
CVE-2011-2523
        ↓
06-exploitation/ftp/
scans/cve-2011-2523-validation.txt
        ↓
Meterpreter Session
        ↓
getuid → root
        ↓
Native Shell
        ↓
uid=0(root)
```

---

# Attack Path B — SMB to Root

```text
CVE-2007-2447
        ↓
06-exploitation/smb/
scans/cve-2007-2447-rce.txt
        ↓
Command Shell
        ↓
whoami → root
        ↓
uid=0(root)
```

---

# Attack Path C — DistCC to Privilege Escalation

```text
CVE-2004-2687
        ↓
06-exploitation/distccd/
scans/cve-2004-2687-rce.txt
        ↓
daemon
UID 1
        ↓
07-post-exploitation/
SUID Enumeration
        ↓
/usr/bin/nmap
Nmap 4.53
        ↓
08-privilege-escalation/
        ↓
euid=0(root)
```

---

# Attack Path D — NFS Privileged Filesystem Access

```text
06-exploitation/nfs/
scans/nfs-export-enum.txt
        ↓
/ *
        ↓
scans/nfs-config.txt
        ↓
rw + no_root_squash
        ↓
scans/nfs-no-root-squash.txt
        ↓
root:root
```

---

# Evidence by Assessment Phase

## 01 — Network Discovery

Primary evidence:

```text
01-network-discovery/scans/host-discovery.txt
```

Purpose:

```text
Active Host Identification
Target Discovery
Laboratory Network Mapping
```

---

# 02 — Port Discovery

Primary evidence includes:

```text
02-port-discovery/scans/port-discovery-meta.txt
02-port-discovery/scans/port-discovery-ubuntuserver.txt
02-port-discovery/scans/port-discovery-windowsserver.txt
02-port-discovery/scans/port-discovery-windowsclient.txt
```

The Metasploitable2 scan established the network attack surface used during later phases.

---

# 03 — Service Enumeration

Primary evidence:

```text
03-service-enumeration/scans/service-enumeration.txt
```

Purpose:

```text
Service Detection
Version Detection
Technology Identification
```

---

# 04 — Technology Identification

Evidence includes service-specific enumeration for:

```text
FTP
HTTP
SMB
NFS
DistCC
```

under:

```text
04-technology-identification/scans/
```

These outputs supported subsequent vulnerability research.

---

# 05 — Vulnerability Research

Research documentation:

```text
05-vulnerability-research/research/ftp.md
05-vulnerability-research/research/http.md
05-vulnerability-research/research/smb.md
05-vulnerability-research/research/nfs.md
05-vulnerability-research/research/distccd.md
```

Raw technical evidence is stored under:

```text
05-vulnerability-research/scans/
```

---

# 06 — Exploitation / Security Validation

Primary evidence is organized by service:

```text
06-exploitation/
├── distccd/
├── ftp/
├── http/
├── nfs/
└── smb/
```

This phase contains the strongest direct evidence for remote exploitation and practical security impact.

---

# 07 — Post-Exploitation

Evidence from this phase supports:

```text
User Enumeration
System Enumeration
Process Enumeration
Service Enumeration
Filesystem Permission Analysis
SUID / SGID Enumeration
Cron Enumeration
Privilege Escalation Candidate Identification
```

The primary purpose of this evidence is to establish the transition from the unprivileged DistCC shell to the privilege-escalation candidate.

---

# 08 — Privilege Escalation

Evidence from this phase demonstrates:

```text
SUID Nmap Execution
        ↓
Effective UID 0
```

The key technical distinction is:

```text
uid=1(daemon)
euid=0(root)
```

---

# 09 — Findings and Risk Assessment

This phase contains interpreted findings rather than primary raw exploitation evidence.

Its purpose is to translate technical observations into:

```text
Finding
Impact
Severity
Risk
Assessment
```

---

# Evidence Quality Matrix

| Finding              | Raw Output | Independent Validation | Screenshot Required |
| -------------------- | :--------: | :--------------------: | :-----------------: |
| FTP RCE              |      ✓     |            ✓           |          No         |
| SMB RCE              |      ✓     |            ✓           |          No         |
| DistCC RCE           |      ✓     |            ✓           |          No         |
| NFS `no_root_squash` |      ✓     |            ✓           |          No         |
| SUID Nmap            |      ✓     |            ✓           |          No         |
| HTTP Disclosure      |      ✓     |            ✓           |          No         |
| WebDAV               |      ✓     |            ✓           |          No         |
| phpMyAdmin Exposure  |      ✓     |            ✓           |          No         |
| TikiWiki Disclosure  |      ✓     |            ✓           |          No         |
| Test Directory       |      ✓     |            ✓           |          No         |

Screenshots may improve presentation but are not required where complete raw output exists.

---

# Recommended Evidence Retention

The following evidence should receive the highest retention priority.

## Critical Exploitation Evidence

```text
cve-2011-2523-validation.txt
cve-2007-2447-rce.txt
cve-2004-2687-rce.txt
nfs-no-root-squash.txt
```

---

## Critical Configuration Evidence

```text
nfs-config.txt
nfs-export-enum.txt
smb-config.txt
```

---

## Important HTTP Evidence

```text
http-enum.txt
webdav-options.txt
phpmyadmin.txt
tiki-install-version.txt
tiki-install-form.txt
http-test.txt
```

---

## Supporting Discovery Evidence

```text
host-discovery.txt
port-discovery-meta.txt
service-enumeration.txt
```

---

# Evidence Naming Guidelines

Evidence filenames should be descriptive and avoid generic names such as:

```text
results.txt
scan.txt
output.txt
test.txt
```

Preferred naming follows:

```text
<service>-<purpose>.txt
```

or:

```text
<cve>-<validation>.txt
```

Examples:

```text
service-enumeration.txt
nfs-export-enum.txt
nfs-no-root-squash.txt
cve-2007-2447-rce.txt
cve-2011-2523-validation.txt
```

Raw contents should not be edited simply because the filename was normalized.

---

# Evidence Integrity Recommendation

Once the project is complete, raw files under:

```text
scans/
```

should ideally remain unchanged.

If further work is performed against the laboratory later, new outputs should be stored separately rather than overwriting original evidence.

A useful approach is:

```text
Original Evidence
      ↓
Preserved

New Assessment
      ↓
New File / New Phase
```

This preserves the historical sequence of the laboratory.

---

# Final Evidence Assessment

The project contains sufficient raw technical evidence to support the principal conclusions documented in the final report.

The strongest findings are supported by direct exploitation or validation output rather than by vulnerability descriptions alone.

The evidence demonstrates:

```text
FTP
→ Root Remote Compromise

SMB
→ Root Remote Compromise

DistCC
→ Remote Shell as daemon

DistCC + SUID Nmap
→ Effective Root Privileges

NFS
→ Privileged Remote Filesystem Access

HTTP
→ Confirmed Information and Application Exposure
```

This evidence structure allows a reviewer to move from the final report back to the original technical output used to support each conclusion.

## Final Status

**Evidence Mapping Complete**
