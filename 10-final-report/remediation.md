# Remediation Recommendations

## Overview

This document provides remediation and hardening recommendations based on the security findings identified during the Metasploitable2 assessment.

The recommendations are prioritized according to demonstrated impact.

Particular attention is given to weaknesses that resulted in:

```text
Remote Code Execution
        ↓
Root-Level Access
        ↓
Privilege Escalation
        ↓
Privileged Filesystem Access
```

The recommended remediation strategy is:

```text
Remove Direct Compromise Paths
        ↓
Reduce Exposed Services
        ↓
Correct Unsafe Configurations
        ↓
Remove Excessive Privileges
        ↓
Restrict Administrative Interfaces
        ↓
Reduce Information Disclosure
        ↓
Validate Remediation
```

---

# Remediation Priority

| Priority  | Description                                                        |
| --------- | ------------------------------------------------------------------ |
| Immediate | Weakness allows direct or near-direct system compromise            |
| High      | Weakness provides significant unauthorized access or privilege     |
| Medium    | Weakness increases attack surface or exposes sensitive information |
| Low       | Hardening improvement with limited demonstrated direct impact      |

---

# R-01 — Remove Vulnerable vsftpd Service

## Related Finding

```text
F-01 — vsftpd 2.3.4 Backdoor
CVE-2011-2523
```

## Priority

**Immediate**

---

## Risk

The FTP service was successfully exploited and provided direct remote access as:

```text
root
UID 0
```

This represents complete system compromise through a network-accessible service.

---

## Recommended Actions

The vulnerable vsftpd `2.3.4` installation should not remain in operation.

Recommended actions include:

1. Remove the compromised or legacy vsftpd installation.
2. Replace it with a current, supported FTP/SFTP solution if file transfer is required.
3. Prefer SSH-based file transfer such as SFTP where operationally appropriate.
4. Restrict access to the service using network-level controls.
5. Disable anonymous FTP unless there is a documented business requirement.
6. Verify the integrity and origin of installed software packages.

---

## Additional Hardening

If FTP must remain available:

```text
Disable Anonymous Access
        +
Restrict Source Networks
        +
Use Supported Software
        +
Avoid Plaintext Credentials
```

Traditional FTP transmits authentication and data without strong transport protection.

Encrypted alternatives should therefore be preferred.

---

## Verification

After remediation:

```bash
nmap -sV -p 21 <TARGET_IP>
```

The expected result should be either:

```text
21/tcp closed
```

or a supported replacement service.

Anonymous access should also be tested and rejected where not explicitly required.

---

# R-02 — Patch or Replace Vulnerable Samba

## Related Finding

```text
F-02 — Samba Username Map Script Command Execution
CVE-2007-2447
```

## Priority

**Immediate**

---

## Risk

The Samba vulnerability was successfully exploited and resulted in a remote command shell running as:

```text
root
UID 0
```

This provides direct full-system compromise.

---

## Recommended Actions

1. Upgrade Samba to a supported version that is not affected by CVE-2007-2447.
2. Remove the vulnerable `username map script` configuration.
3. Review the Samba configuration for unnecessary legacy options.
4. Restrict SMB access to trusted systems and networks.
5. Disable anonymous and guest access unless explicitly required.
6. Review all SMB shares for excessive read/write permissions.
7. Disable SMBv1.
8. Require SMB message signing where supported and operationally appropriate.

---

## Configuration Review

The assessment identified:

```text
username map script = /etc/samba/scripts/mapusers.sh
```

This configuration should be removed unless a secure and supported equivalent is explicitly required.

Any username mapping functionality should be reviewed to ensure that external user-controlled input cannot influence command execution.

---

## Verification

After remediation:

```bash
nmap -sV -p 139,445 <TARGET_IP>
```

and:

```bash
nmap --script smb-protocols -p 139,445 <TARGET_IP>
```

The system should no longer expose vulnerable Samba behavior or SMBv1.

Anonymous enumeration should also be tested:

```bash
smbclient -L //<TARGET_IP> -N
```

Unauthenticated access should fail unless explicitly intended.

---

# R-03 — Remove or Restrict DistCC

## Related Finding

```text
F-03 — DistCC Remote Command Execution
CVE-2004-2687
```

## Priority

**Immediate**

---

## Risk

The exposed DistCC daemon allowed unauthenticated remote command execution.

Although the resulting shell initially executed as:

```text
daemon
UID 1
```

the access later provided a foothold for privilege escalation.

---

## Recommended Actions

1. Remove DistCC if distributed compilation is not required.
2. Upgrade to a supported implementation if it must remain in use.
3. Restrict DistCC access to explicitly trusted build systems.
4. Do not expose the service to general user networks.
5. Apply host firewall restrictions to port `3632/tcp`.
6. Run the service using a dedicated low-privilege account.
7. Review service configuration for permissive client-access rules.

---

## Network Restriction

DistCC should not accept requests from arbitrary systems.

Access should follow:

```text
Trusted Build Hosts
        ↓
DistCC

All Other Hosts
        ↓
Blocked
```

---

## Verification

After remediation:

```bash
nmap -sV -p 3632 <TARGET_IP>
```

If DistCC is not required, the desired state is:

```text
3632/tcp closed
```

If required, access should only succeed from authorized build systems.

---

# R-04 — Correct NFS Root Export Configuration

## Related Finding

```text
F-04 — Insecure NFS Root Export
```

## Priority

**Immediate**

---

## Risk

The NFS server exported:

```text
/
```

using:

```text
*(rw,sync,no_root_squash,no_subtree_check)
```

The assessment demonstrated that remote UID `0` was preserved and could create files owned by:

```text
root:root
```

on the target filesystem.

---

## Recommended Actions

The current configuration should be removed.

Do not export:

```text
/
```

through NFS.

Instead:

1. Export only the minimum directory required.
2. Restrict access to specific trusted clients.
3. Replace wildcard access (`*`) with explicit hosts or networks.
4. Use `root_squash`.
5. Prefer read-only (`ro`) exports where write access is unnecessary.
6. Review filesystem permissions independently of NFS export permissions.
7. Restrict NFS using firewall rules.
8. Review all existing exports for unnecessary exposure.

---

## Example Safer Concept

Instead of:

```text
/ *(rw,sync,no_root_squash,no_subtree_check)
```

a safer design would follow the principle:

```text
Specific Directory
        +
Specific Client
        +
Minimum Required Permissions
        +
root_squash
```

The exact production configuration should be determined according to operational requirements.

---

## Verification

After remediation:

```bash
showmount -e <TARGET_IP>
```

The root filesystem should no longer appear as an unrestricted export.

A root client should also be unable to preserve UID `0` through the export.

---

# R-05 — Remove SUID Permission from Nmap

## Related Finding

```text
F-05 — SUID Nmap Privilege Escalation
```

## Priority

**Immediate**

---

## Risk

The legacy Nmap binary:

```text
/usr/bin/nmap
```

was owned by root and configured with SUID permissions.

This allowed an unprivileged `daemon` session to obtain:

```text
euid=0(root)
```

through Nmap's interactive command-execution functionality.

---

## Recommended Actions

1. Remove the SUID bit from `/usr/bin/nmap`.
2. Upgrade Nmap to a supported version.
3. Review why Nmap was configured as SUID in the first place.
4. Use dedicated privilege mechanisms instead of SUID where elevated network functionality is required.
5. Perform a complete review of all SUID and SGID executables.

---

## Permission Correction

The desired security principle is:

```text
Network Utility
      ≠
Arbitrary Root Command Execution
```

Nmap should not generally require unrestricted SUID root execution.

---

## Verification

Check permissions using:

```bash
ls -lah /usr/bin/nmap
```

The SUID bit should no longer be present.

A broader validation should also be performed:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Each remaining SUID binary should have a documented requirement.

---

# R-06 — Reduce HTTP Technology Disclosure

## Related Finding

```text
F-06 — HTTP Technology & Version Disclosure
```

## Priority

**Medium**

---

## Risk

The web server disclosed specific versions including:

```text
Apache/2.2.8
PHP/5.2.4
```

This makes technology fingerprinting easier and assists version-specific vulnerability research.

---

## Recommended Actions

1. Upgrade Apache and PHP to supported versions.
2. Reduce unnecessary server banner disclosure.
3. Disable unnecessary `X-Powered-By` information.
4. Review error handling to avoid exposing stack or runtime details.
5. Avoid relying on banner suppression as a substitute for patching.

---

## Important Note

Removing version information does not fix underlying vulnerabilities.

The remediation order should be:

```text
Patch / Upgrade
      ↓
Secure Configuration
      ↓
Reduce Version Disclosure
```

---

## Verification

Inspect response headers:

```bash
curl -I http://<TARGET_IP>/
```

The response should reveal only the minimum information required.

---

# R-07 — Reduce Exposed HTTP Resources

## Related Finding

```text
F-07 — Excessive HTTP Resource Exposure
```

## Priority

**High**

---

## Risk

The server exposed multiple applications and directories, including diagnostic, administrative, testing, and legacy resources.

This substantially increases the web attack surface.

---

## Recommended Actions

Perform an application inventory and classify each resource as:

```text
Required
Not Required
Administrative
Diagnostic
Development / Testing
Legacy
```

Remove or restrict anything not required.

Particular attention should be given to:

```text
/phpinfo.php
/phpMyAdmin/
/tikiwiki/
/test/
/doc/
/index/
```

---

## Verification

Repeat resource enumeration after remediation.

Only approved application paths should remain accessible.

---

# R-08 — Disable or Restrict WebDAV

## Related Finding

```text
F-08 — Exposed WebDAV Functionality
```

## Priority

**High**

---

## Risk

The `/dav/` endpoint exposed WebDAV functionality and advertised methods beyond standard HTTP retrieval.

Although unauthorized write operations were not demonstrated, the additional functionality increases attack surface.

---

## Recommended Actions

1. Disable WebDAV if it is not required.
2. Restrict `/dav/` to authenticated users if WebDAV is required.
3. Limit allowed HTTP methods to those operationally necessary.
4. Restrict access by network location where possible.
5. Review filesystem permissions behind the WebDAV endpoint.
6. Enable appropriate authentication and authorization controls.
7. Review logging for DAV modification operations.

---

## Verification

Test:

```bash
curl -i -X OPTIONS http://<TARGET_IP>/dav/
```

The endpoint should either be unavailable or expose only explicitly required methods to authorized clients.

Unauthorized DAV operations should be rejected.

---

# R-09 — Restrict phpMyAdmin

## Related Finding

```text
F-09 — Public phpMyAdmin Interface
```

## Priority

**High**

---

## Risk

A database administration interface was reachable by unauthenticated network clients.

Although no authentication bypass was demonstrated, exposing the application creates an unnecessary high-value target.

---

## Recommended Actions

1. Remove phpMyAdmin if it is not required.
2. Restrict access to trusted administrative networks.
3. Require strong authentication.
4. Use additional access controls such as VPN or reverse-proxy authentication.
5. Keep phpMyAdmin updated.
6. Apply least privilege to database accounts.
7. Do not expose database administrative interfaces directly to untrusted networks.

---

## Preferred Access Model

```text
Administrator
     ↓
Trusted Management Network / VPN
     ↓
phpMyAdmin

General Network
     ↓
Blocked
```

---

## Verification

Access from a non-administrative client should return a denial response rather than the phpMyAdmin login interface.

---

# R-10 — Correct TikiWiki Information Disclosure

## Related Finding

```text
F-10 — TikiWiki Information Disclosure
```

## Priority

**High**

---

## Risk

TikiWiki exposed backend information including:

```text
Database User: root
Database Host: localhost
Internal Filesystem Paths
Database Driver Details
Application Version
```

These disclosures provide useful reconnaissance information to an attacker.

---

## Recommended Actions

1. Upgrade or remove the legacy TikiWiki installation.
2. Correct the database configuration.
3. Disable verbose PHP/application error output in production.
4. Log detailed errors server-side instead of returning them to clients.
5. Avoid using the database `root` account for web applications.
6. Create a dedicated database account with minimum required permissions.
7. Review application configuration files for sensitive information exposure.

---

## Database Privilege Recommendation

The web application should follow:

```text
Application
     ↓
Dedicated Database Account
     ↓
Only Required Database Privileges
```

and not:

```text
Application
     ↓
Database root
```

---

## Verification

Unauthenticated requests should not reveal:

```text
Database Username
Database Host
Internal Filesystem Paths
Source File Names
Source Line Numbers
```

Application errors should instead present generic user-facing messages.

---

# R-11 — Remove TikiWiki Installation Interface

## Related Finding

```text
F-11 — Exposed TikiWiki Installation Interface
```

## Priority

**High**

---

## Risk

The publicly accessible installer exposed application version and database configuration fields.

Although unauthorized configuration modification was not demonstrated, installation functionality should not remain publicly accessible after deployment.

---

## Recommended Actions

1. Remove or disable `tiki-install.php` after installation.
2. Restrict installation and configuration interfaces to administrators.
3. Require authentication before configuration operations.
4. Restrict administrative endpoints to trusted management networks.
5. Review whether installation scripts or setup files remain accessible elsewhere.

---

## Verification

Request:

```text
/tikiwiki/tiki-install.php
```

after remediation.

The endpoint should return an appropriate denial or not-found response for unauthorized users.

---

# R-12 — Remove Public Test Resources

## Related Finding

```text
F-12 — Public Test Resources & Directory Listing
```

## Priority

**Medium**

---

## Risk

The publicly accessible `/test/` resource exposed directory indexing and internal testing content.

Test directories may unintentionally expose development files, debug output, configuration data, or future sensitive content.

---

## Recommended Actions

1. Remove test and development resources from production-facing web roots.
2. Disable unnecessary directory indexing.
3. Review web directories for temporary or backup files.
4. Separate development/testing environments from production services.
5. Restrict any required test resources to authorized users.

---

## Verification

Requests to:

```text
/test/
```

should no longer return an open directory listing to unauthenticated users.

---

# Additional Remediation — Anonymous FTP

## Related Security Condition

Anonymous FTP authentication was successfully validated during the assessment.

## Priority

**High**

## Recommendation

Disable anonymous FTP unless there is a documented requirement.

If anonymous access is intentionally required:

* Limit accessible directories.
* Prefer read-only access.
* Prevent uploads.
* Monitor anonymous sessions.
* Do not expose sensitive content.
* Restrict service availability where possible.

---

# Additional Remediation — Anonymous SMB Access

## Related Security Condition

Anonymous SMB enumeration and share access were identified during the assessment.

## Priority

**High**

## Recommendation

1. Disable guest and anonymous SMB access unless required.
2. Require authenticated accounts.
3. Apply least privilege to every share.
4. Remove anonymous read/write permissions.
5. Review all exposed shares.
6. Restrict SMB to trusted network segments.

---

# Additional Remediation — SMBv1

## Related Security Condition

The target supported legacy SMBv1 functionality.

## Priority

**High**

## Recommendation

Disable SMBv1 and require modern supported SMB protocol versions.

Legacy protocol support should only remain where a documented compatibility requirement exists and compensating controls are implemented.

---

# Additional Remediation — SMB Message Signing

## Related Security Condition

SMB message signing was identified as disabled.

## Priority

**High**

## Recommendation

Require SMB signing where supported by the environment.

This should be combined with:

```text
Modern SMB Protocol
        +
Authenticated Access
        +
Least Privilege Shares
        +
Network Segmentation
```

---

# System-Wide Hardening Recommendations

The assessment demonstrates that individual vulnerability remediation alone would not be sufficient.

Multiple weaknesses existed simultaneously.

A broader hardening program should therefore be applied.

---

# 1. Reduce Network Attack Surface

Review every listening network service.

For each service ask:

```text
Is It Required?
      ↓
Yes → Restrict and Harden
No  → Disable or Remove
```

Services should not remain reachable solely because they are installed.

---

# 2. Apply Network Segmentation

Administrative, file-sharing, database, and development services should not be exposed equally to all hosts.

Separate:

```text
User Network
Management Network
Server Network
Development Network
```

using firewall and segmentation controls.

---

# 3. Patch Management

Establish a process for:

```text
Asset Inventory
      ↓
Software Version Tracking
      ↓
Security Advisory Review
      ↓
Testing
      ↓
Patch Deployment
      ↓
Verification
```

Legacy software should be upgraded or replaced rather than indefinitely retained.

---

# 4. Apply Least Privilege

Services and applications should execute with only the privileges required for their function.

Examples from this assessment include:

```text
Web Applications
→ Should not use database root

Nmap
→ Should not execute SUID root

NFS Clients
→ Should not retain remote UID 0 unnecessarily
```

---

# 5. Review SUID / SGID Permissions

Regularly audit:

```bash
find / -perm -4000 -type f 2>/dev/null
```

and:

```bash
find / -perm -2000 -type f 2>/dev/null
```

Every privileged executable should have a documented operational requirement.

Unexpected SUID or SGID permissions should be investigated and removed.

---

# 6. Secure Error Handling

Production applications should not expose:

```text
Internal Paths
Database Users
Database Hosts
Source Files
Line Numbers
Stack Information
Detailed Runtime Errors
```

Detailed errors should be written to protected logs.

Users should receive generic error responses.

---

# 7. Remove Installation and Diagnostic Interfaces

Deployment procedures should include removal or restriction of resources such as:

```text
Installer Scripts
phpinfo()
Test Directories
Debug Interfaces
Administrative Setup Pages
```

These resources should not remain publicly accessible after deployment.

---

# 8. Authentication Hardening

Anonymous or guest authentication should be disabled by default.

All administrative services should use:

```text
Unique Accounts
Strong Authentication
Least Privilege
Restricted Network Access
```

---

# 9. Logging and Monitoring

Enable monitoring for security-relevant activity including:

```text
Failed Authentication
Anonymous Access
Administrative Interface Access
Unexpected NFS Mounts
SMB Share Access
Privilege Escalation Attempts
Unusual Service Requests
```

Logs should be centrally retained where possible.

---

# 10. Remediation Validation

Corrective action should always be followed by security validation.

The workflow should be:

```text
Finding
   ↓
Remediation
   ↓
Configuration Review
   ↓
Retest
   ↓
Verify Finding Is No Longer Reproducible
```

A finding should not be considered resolved solely because a configuration change was made.

---

# Recommended Remediation Order

Based on the demonstrated attack paths, remediation should proceed in the following order:

## Immediate

```text
1. Remove / patch vulnerable vsftpd
2. Patch / replace vulnerable Samba
3. Remove / restrict DistCC
4. Correct NFS root export
5. Remove SUID from Nmap
```

These conditions directly enabled root compromise, remote command execution, or privileged access.

---

## High

```text
6. Disable anonymous FTP
7. Disable anonymous SMB
8. Disable SMBv1
9. Enable SMB signing
10. Restrict WebDAV
11. Restrict phpMyAdmin
12. Remove TikiWiki installer
13. Correct TikiWiki database/error configuration
```

---

## Medium

```text
14. Reduce HTTP version disclosure
15. Remove public phpinfo()
16. Remove test resources
17. Disable directory listing
18. Reduce unnecessary HTTP resources
```

---

# Remediation Validation Matrix

| Finding | Primary Corrective Action              | Verification                    |
| ------- | -------------------------------------- | ------------------------------- |
| F-01    | Remove/upgrade vsftpd                  | Retest port 21 and exploit path |
| F-02    | Upgrade Samba/remove vulnerable config | Retest SMB and CVE path         |
| F-03    | Remove/restrict DistCC                 | Verify port 3632 access         |
| F-04    | Remove root export/use `root_squash`   | Retest export and UID mapping   |
| F-05    | Remove Nmap SUID                       | Verify binary permissions       |
| F-06    | Upgrade and reduce banners             | Inspect HTTP headers            |
| F-07    | Remove unnecessary resources           | Repeat HTTP enumeration         |
| F-08    | Disable/restrict WebDAV                | Retest OPTIONS/DAV access       |
| F-09    | Restrict phpMyAdmin                    | Test unauthenticated access     |
| F-10    | Disable verbose errors                 | Retest TikiWiki error responses |
| F-11    | Remove installer                       | Request installer endpoint      |
| F-12    | Remove `/test/`/indexing               | Request test resource           |

---

# Final Recommendation

The target requires comprehensive remediation rather than isolated patching.

The assessment demonstrated multiple independent compromise paths:

```text
FTP → root
SMB → root
DistCC → daemon → privilege escalation
NFS → privileged filesystem access
```

Therefore, remediation should address both:

```text
Individual Vulnerabilities
        +
Underlying Security Architecture
```

The most effective long-term improvements are:

```text
Reduce Services
      +
Use Supported Software
      +
Apply Least Privilege
      +
Restrict Network Access
      +
Secure Administrative Interfaces
      +
Patch Consistently
      +
Validate Remediation
```

The system should be retested after remediation to confirm that previously demonstrated attack paths are no longer reproducible.
