# 04 — Technology Identification Commands

## Objective

Perform technology-specific enumeration of selected services identified during the service enumeration phase.

The purpose of this phase is to move beyond basic service and version detection and analyze the configuration, exposed resources, authentication behavior, protocol characteristics, and accessible functionality of selected technologies.

No vulnerability exploitation is performed during this phase.

---

## Target

| Property       | Value                       |
| -------------- | --------------------------- |
| Host           | Metasploitable2             |
| Target IP      | `192.168.56.107`            |
| Attacker       | Kali Linux                  |
| Attacker IP    | `192.168.56.106`            |
| Environment    | Isolated Virtual Laboratory |
| Previous Phase | Service Enumeration         |

---

# 1. Methodology

Phase 03 identified the services and software versions exposed by the target.

Phase 04 performs deeper enumeration of selected technologies:

```text
Service Enumeration
        ↓
Technology Selection
        ↓
Technology-Specific Enumeration
        ↓
Configuration Analysis
        ↓
Access Validation
        ↓
Security-Relevant Findings
```

The technologies investigated during this phase were:

```text
FTP
HTTP
SMB
NFS
distccd
```

The results obtained here are later used as input for vulnerability research.

---

# 2. FTP Enumeration

**Port:** `21/tcp`

## Initial Enumeration

```bash
nmap -sV -sC -p 21 192.168.56.107 -oN scans/ftp-enumeration.txt
```

### Purpose

Performs service/version detection and executes Nmap's default scripts against the FTP service.

The scan is used to identify:

* FTP server implementation
* Software version
* Authentication behavior
* Anonymous access
* Additional service information exposed by the server

---

## Anonymous FTP Validation

The FTP service was subsequently accessed using the standard FTP client.

```bash
ftp 192.168.56.107
```

When prompted for a username:

```text
anonymous
```

An anonymous password is supplied if requested by the FTP client.

After authentication:

```text
pwd
ls
dir
```

### Purpose

Validates whether the anonymous access detected during enumeration can be reproduced manually.

This test establishes authentication behavior only.

No exploitation of the FTP service is performed during this phase.

---

## Exit FTP

```text
bye
```

---

# 3. HTTP Enumeration

**Port:** `80/tcp`

HTTP enumeration was performed in multiple stages because the web service exposes both server-level information and multiple web applications/resources.

---

## 3.1 HTTP Service Identification

```bash
nmap -sV -sC -p 80 192.168.56.107 -oN scans/http-enumeration.txt
```

### Purpose

Identifies the HTTP server implementation and performs Nmap's default HTTP-related enumeration.

The scan provides an initial server fingerprint before deeper resource enumeration.

---

## 3.2 HTTP Resource Enumeration

```bash
nmap --script http-enum -p 80 192.168.56.107 -oN scans/http-enum.txt
```

### Purpose

Searches for known or potentially interesting web resources exposed by the HTTP server.

This can identify:

* Web applications
* Administrative interfaces
* Information pages
* Test resources
* Directory listings
* Other exposed paths

The resulting resources are investigated individually where relevant.

---

## 3.3 HTTP Response Inspection

A raw HTTP response from the target was preserved as:

```text
scans/http-response.txt
```

The response contains server and application information exposed through HTTP headers and page content.

Relevant HTTP response fields include:

```text
Server
X-Powered-By
Content-Type
HTTP status
```

### Purpose

Provides an additional technology fingerprint independent of the Nmap version-detection output.

It also preserves the web application's original response for later analysis.

---

## 3.4 Web Application Identification

Resources identified through HTTP enumeration were reviewed to determine which technologies and applications were exposed.

The enumeration identified resources associated with technologies such as:

```text
TikiWiki
phpMyAdmin
PHP information
WebDAV
Test resources
Additional web applications
```

The purpose at this stage is identification and enumeration.

Potential vulnerabilities associated with these technologies are researched separately during phase 05.

---

# 4. SMB Enumeration

**Ports:** `139/tcp`, `445/tcp`

SMB was investigated using both Nmap NSE scripts and the `smbclient` utility.

This provides both protocol-level information and practical validation of accessible shares.

---

## 4.1 Initial SMB Enumeration

```bash
nmap -sV -sC -p 139,445 192.168.56.107 -oN scans/smb-enumeration.txt
```

### Purpose

Performs service/version detection and default SMB-related script enumeration.

The scan is used to identify:

* SMB/Samba implementation
* Software version
* Authentication characteristics
* Security mode
* Message-signing configuration
* Operating system information when available

---

## 4.2 SMB Protocol and Host Details

```bash
nmap --script smb-os-discovery,smb-protocols -p 139,445 192.168.56.107 -oN scans/smb-details.txt
```

### Purpose

Performs more focused SMB enumeration.

### `smb-os-discovery`

Attempts to identify host information exposed through SMB, including:

* Operating system
* Computer name
* Domain
* Fully qualified domain name

### `smb-protocols`

Enumerates supported SMB protocol dialects.

This helps identify legacy protocol support that may require additional security review.

---

## 4.3 SMB Share Enumeration

```bash
nmap --script smb-enum-shares -p 139,445 192.168.56.107 -oN scans/smb-shares.txt
```

### Purpose

Enumerates SMB shares exposed by the target and attempts to determine their accessibility.

The scan provides information including:

* Share name
* Share type
* Filesystem path when exposed
* Anonymous access permissions

---

## 4.4 smbclient Share Enumeration

```bash
smbclient -L //192.168.56.107 -N
```

### Purpose

Requests the list of SMB shares without supplying authentication credentials.

### Option Breakdown

#### `-L`

Requests a list of available shares.

#### `-N`

Prevents `smbclient` from requesting a password.

This provides independent manual validation of anonymous SMB enumeration.

The resulting output was preserved as:

```text
scans/smbclient-shares.txt
```

---

## 4.5 Anonymous `tmp` Share Access

The identified `tmp` share was accessed without supplying credentials.

```bash
smbclient //192.168.56.107/tmp -N
```

After connecting:

```text
ls
```

### Purpose

Validates whether the anonymous access identified during share enumeration provides practical access to the contents of the share.

The resulting directory listing was preserved as:

```text
scans/smb-tmp.txt
```

This remains an access validation test rather than exploitation.

---

# 5. NFS Enumeration

**Ports:** `111/tcp`, `2049/tcp`

NFS enumeration was performed in several stages:

```text
Service Enumeration
       ↓
RPC Enumeration
       ↓
Export Discovery
       ↓
Filesystem Inspection
       ↓
Mount Validation
       ↓
Configuration Inspection
```

---

## 5.1 Initial NFS Enumeration

```bash
nmap -sV -sC -p 111,2049 192.168.56.107 -oN scans/nfs-enumeration.txt
```

### Purpose

Identifies NFS and RPC-related services and executes relevant default scripts.

The scan provides information about:

* rpcbind
* NFS
* Supported NFS versions
* mountd
* nlockmgr
* RPC status services
* Dynamically assigned RPC ports

---

## 5.2 Export Discovery

```bash
nmap --script nfs-showmount -p 111,2049 192.168.56.107 -oN scans/nfs-showmount.txt
```

### Purpose

Queries the target for exported NFS filesystems.

This determines which filesystem paths are being offered to NFS clients and which hosts are permitted to access them.

---

## 5.3 NFS Filesystem Inspection

```bash
nmap --script nfs-ls,nfs-statfs -p 111,2049 192.168.56.107 -oN scans/nfs-details.txt
```

### Purpose

Performs additional enumeration of the exposed NFS filesystem.

### `nfs-ls`

Attempts to list files and directories available through the NFS export.

### `nfs-statfs`

Retrieves filesystem statistics associated with the export.

This helps determine the practical extent of filesystem exposure before mounting the export manually.

---

# 6. NFS Mount Validation

After identifying the exported filesystem, a local mount point was prepared on Kali.

## Create Mount Point

```bash
mkdir -p /home/kali/nfs-mount
```

---

## Mount Export

```bash
sudo mount -t nfs 192.168.56.107:/ /home/kali/nfs-mount
```

### Purpose

Mounts the exported root filesystem locally to validate that the NFS export can actually be accessed from the assessment host.

---

## Verify Mount

```bash
mount | grep nfs-mount
```

The resulting mount information was preserved as:

```text
scans/nfs-mount.txt
```

### Purpose

Confirms that the remote filesystem is mounted and identifies relevant mount characteristics.

---

## Inspect Mounted Filesystem

```bash
ls -lah /home/kali/nfs-mount
```

### Purpose

Confirms practical access to the mounted filesystem and allows inspection of the exposed directory structure.

---

# 7. NFS Write Access Validation

A temporary file was created to determine whether the mounted filesystem permitted write operations.

```bash
touch /home/kali/nfs-mount/tmp/nfs_test
```

## Verify Temporary File

```bash
ls -lah /home/kali/nfs-mount/tmp/nfs_test
```

### Purpose

Confirms whether the current Kali user can create files through the mounted NFS export.

This validates practical write access without modifying an existing target file.

---

## Cleanup

```bash
rm /home/kali/nfs-mount/tmp/nfs_test
```

### Purpose

Removes the temporary validation artifact immediately after the test.

---

# 8. NFS Export Configuration Inspection

Because the target root filesystem was mounted through NFS, the export configuration file was inspected directly.

```bash
cat /home/kali/nfs-mount/etc/exports
```

### Purpose

Reviews the server-side NFS export configuration exposed through the mounted filesystem.

The relevant configuration identified during the laboratory is documented in `results.md`.

This configuration becomes particularly important during the exploitation phase because it determines how remote client identities are handled by the NFS server.

No root-level write test is performed during this phase.

That validation belongs to phase 06.

---

# 9. NFS Unmounting

When NFS enumeration is complete, the mounted filesystem can be detached using:

```bash
sudo umount /home/kali/nfs-mount
```

### Purpose

Cleanly removes the remote filesystem from the Kali mount point after testing.

---

# 10. distccd Enumeration

**Port:** `3632/tcp`

## Service Enumeration

```bash
nmap -sV -sC -p 3632 192.168.56.107 -oN scans/distccd-enumeration.txt
```

### Purpose

Performs service and version detection against the exposed distccd service.

The resulting version information provides the basis for later vulnerability research.

No remote command execution is attempted during this phase.

---

# 11. Evidence Handling

The original technical outputs from this phase are preserved under:

```text
scans/
```

The expected scan structure is:

```text
scans/
├── ftp-enumeration.txt
├── distccd-enumeration.txt
│
├── http-enumeration.txt
├── http-enum.txt
├── http-response.txt
│
├── smb-enumeration.txt
├── smb-details.txt
├── smb-shares.txt
├── smbclient-shares.txt
├── smb-tmp.txt
│
├── nfs-enumeration.txt
├── nfs-showmount.txt
├── nfs-details.txt
└── nfs-mount.txt
```

Raw outputs should remain unchanged after being preserved.

The filename may be normalized for repository organization, but the original content should not be retrospectively edited.

---

# 12. Evidence Screenshots

Separate screenshots are not required when the corresponding raw terminal or Nmap output has already been preserved.

If screenshots from the original laboratory session exist, useful candidates would include:

```text
Anonymous FTP access
Anonymous SMB share access
Mounted NFS filesystem
HTTP application/resource discovery
```

Screenshots should only be included when they add meaningful visual evidence.

Missing screenshots should not be recreated retrospectively solely for documentation purposes.

---

# 13. Command Reference

## FTP

```bash
nmap -sV -sC -p 21 192.168.56.107 -oN scans/ftp-enumeration.txt
ftp 192.168.56.107
```

---

## HTTP

```bash
nmap -sV -sC -p 80 192.168.56.107 -oN scans/http-enumeration.txt

nmap --script http-enum -p 80 192.168.56.107 -oN scans/http-enum.txt
```

---

## SMB

```bash
nmap -sV -sC -p 139,445 192.168.56.107 -oN scans/smb-enumeration.txt

nmap --script smb-os-discovery,smb-protocols -p 139,445 192.168.56.107 -oN scans/smb-details.txt

nmap --script smb-enum-shares -p 139,445 192.168.56.107 -oN scans/smb-shares.txt

smbclient -L //192.168.56.107 -N

smbclient //192.168.56.107/tmp -N
```

---

## NFS

```bash
nmap -sV -sC -p 111,2049 192.168.56.107 -oN scans/nfs-enumeration.txt

nmap --script nfs-showmount -p 111,2049 192.168.56.107 -oN scans/nfs-showmount.txt

nmap --script nfs-ls,nfs-statfs -p 111,2049 192.168.56.107 -oN scans/nfs-details.txt

mkdir -p /home/kali/nfs-mount

sudo mount -t nfs 192.168.56.107:/ /home/kali/nfs-mount

mount | grep nfs-mount

ls -lah /home/kali/nfs-mount

touch /home/kali/nfs-mount/tmp/nfs_test

ls -lah /home/kali/nfs-mount/tmp/nfs_test

rm /home/kali/nfs-mount/tmp/nfs_test

cat /home/kali/nfs-mount/etc/exports

sudo umount /home/kali/nfs-mount
```

---

## distccd

```bash
nmap -sV -sC -p 3632 192.168.56.107 -oN scans/distccd-enumeration.txt
```

---

# 14. Phase Boundary

Technology identification establishes what is exposed and how those technologies are configured.

It does not determine vulnerability solely from software version information.

The workflow becomes:

```text
03 — Service Enumeration
          ↓
Services and Versions
          ↓
04 — Technology Identification
          ↓
Configuration / Access / Exposure
          ↓
05 — Vulnerability Research
```

Examples of information carried forward include:

```text
Software versions
Anonymous authentication
Legacy protocol support
Accessible SMB shares
Exposed web applications
NFS exports
NFS permissions
Service configuration
```

These observations provide the technical basis for researching vulnerabilities and prioritizing exploitation candidates.

---

## Next Step

Research vulnerabilities and security weaknesses associated with the technologies and configurations identified during this phase.

Potential findings must be validated before being classified as exploitable.

**Next Phase: 05 — Vulnerability Research**
