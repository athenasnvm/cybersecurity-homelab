# 03 — Service Enumeration Commands

## Objective

Identify the services, technologies, and software versions running on the open TCP ports discovered on Metasploitable2 during the previous port discovery phase.

The purpose of this phase is to transform the previously identified open ports into a more detailed representation of the target's exposed network services.

---

## Target

| Property       | Value            |
| -------------- | ---------------- |
| Host           | Metasploitable2  |
| IP Address     | `192.168.56.107` |
| Open TCP Ports | 30               |
| Previous Phase | Port Discovery   |
| Tool           | Nmap             |

---

# 1. Methodology

The previous port discovery phase identified 30 open TCP ports on Metasploitable2.

Rather than performing another complete port scan, service and version detection was performed specifically against those known open ports.

This approach follows the workflow:

```text
Port Discovery
      ↓
30 Open TCP Ports Identified
      ↓
Targeted Service Enumeration
      ↓
Service Identification
      ↓
Version Identification
      ↓
Technology-Specific Enumeration
```

The service enumeration phase therefore builds directly on the results obtained during phase 02.

---

# 2. Targeted Service and Version Detection

## Command

```bash
nmap -sV -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,40677,43497,43896,60823 192.168.56.107 -oN scans/service-enumeration.txt
```

## Purpose

Performs service and version detection against all 30 TCP ports previously identified as open on Metasploitable2.

Instead of relying only on Nmap's initial port-based service labels, `-sV` actively probes the selected ports to determine which applications and software versions are responding.

---

# 3. Command Breakdown

## `nmap`

Runs the Nmap network scanner.

---

## `-sV`

Enables service and version detection.

Nmap sends service-specific probes to the selected ports and analyzes the responses in an attempt to identify:

* Network protocol
* Service implementation
* Software product
* Software version
* Additional service information when available

The accuracy of version detection depends on the information exposed by the remote service.

---

## `-p`

Restricts the scan to explicitly selected TCP ports.

The following ports were carried forward from the previous port discovery phase:

```text
21
22
23
25
53
80
111
139
445
512
513
514
1099
1524
2049
2121
3306
3632
5432
5900
6000
6667
6697
8009
8180
8787
40677
43497
43896
60823
```

This prevents unnecessary rescanning of the complete TCP port range during service enumeration.

---

## `192.168.56.107`

Specifies Metasploitable2 as the target.

---

## `-oN scans/service-enumeration.txt`

Stores the raw Nmap output in normal text format.

The resulting file is preserved under:

```text
scans/service-enumeration.txt
```

This file provides the original technical record used to support the interpreted findings documented in `results.md`.

---

# 4. Enumeration Coverage

The targeted scan covered all TCP ports previously identified as open.

For documentation purposes, they can be grouped by service category.

## Remote Access and Administration

```text
21/tcp
22/tcp
23/tcp
512/tcp
513/tcp
514/tcp
1524/tcp
5900/tcp
```

These ports were evaluated to identify remote access, file transfer, remote shell, and remote administration technologies.

---

## Web and Application Services

```text
80/tcp
8009/tcp
8180/tcp
```

These ports were evaluated for HTTP and web application infrastructure.

Detailed HTTP-specific enumeration is performed during the technology identification phase.

---

## File Sharing and Network Filesystems

```text
111/tcp
139/tcp
445/tcp
2049/tcp
```

These ports were evaluated for RPC, SMB/Samba, and NFS-related services.

Detailed SMB and NFS enumeration is performed separately during phase 04.

---

## Database Services

```text
3306/tcp
5432/tcp
```

These ports were evaluated to identify exposed database technologies.

---

## Development and Distributed Services

```text
1099/tcp
3632/tcp
8787/tcp
43497/tcp
```

These ports were evaluated for distributed computing, remote method invocation, and related application services.

---

## Messaging and Communication Services

```text
25/tcp
6667/tcp
6697/tcp
```

These ports were evaluated for mail and IRC-related technologies.

---

## Additional Network Services

```text
53/tcp
2121/tcp
6000/tcp
40677/tcp
43896/tcp
60823/tcp
```

These ports were included because they were previously confirmed as open and therefore remained part of the exposed attack surface.

---

# 5. Service Enumeration Strategy

The scan was designed to answer three primary questions for each previously discovered port.

### What service is listening?

The port number alone does not guarantee which application is actually running behind it.

Service probing provides stronger identification.

### Which implementation is being used?

Different products can implement the same protocol.

For example, identifying a port as FTP is less useful than identifying the specific FTP server implementation.

### Which version is exposed?

Version information provides an important input for subsequent vulnerability research.

However:

```text
Detected Version ≠ Confirmed Vulnerability
```

A software version must be researched and validated independently before being classified as vulnerable.

---

# 6. Relationship With Port Discovery

Phase 02 established the attack surface:

```text
192.168.56.107
        ↓
30 Open TCP Ports
```

Phase 03 adds service context:

```text
Open Port
    ↓
Service
    ↓
Implementation
    ↓
Version
```

This prevents later vulnerability research from relying only on assumptions derived from standard port assignments.

---

# 7. Relationship With Technology Identification

Service enumeration provides broad identification across the target.

The next phase performs deeper investigation of selected technologies.

For example:

```text
FTP
 ↓
Authentication behavior
 ↓
Anonymous access testing
```

```text
HTTP
 ↓
Headers
Resources
Methods
Web applications
```

```text
SMB
 ↓
Shares
Protocols
Authentication
Security configuration
```

```text
NFS
 ↓
Exports
RPC services
Filesystem access
Permissions
```

This separation keeps basic service/version detection distinct from deeper service-specific enumeration.

---

# 8. Scope and Limitations

The service enumeration scan identifies technologies based on network responses.

The output should therefore be treated as reconnaissance evidence rather than proof of vulnerability.

A detected software version may:

* Be correctly identified
* Be partially identified
* Be reported as a version range
* Expose insufficient information for exact identification

For this reason, subsequent phases perform additional validation where necessary.

No exploitation is performed during this phase.

---

# 9. Documentation

## Raw Scan

The original Nmap output is preserved as:

```text
scans/service-enumeration.txt
```

The scan should remain unchanged after being added to the repository.

Even if the original internal Nmap header references a different output filename, the raw contents should not be edited retrospectively.

---

## Results

The interpreted findings from the service enumeration are documented in:

```text
results.md
```

`results.md` should contain the meaningful identified technologies and versions rather than reproducing the complete Nmap output.

---

## Evidence

A separate screenshot is not required for this phase when the original Nmap output has already been preserved.

The raw `.txt` scan provides a complete, searchable, and reproducible technical record of the service enumeration.

---

# 10. Command Reference

The service enumeration performed during this phase can be reproduced using:

```bash
nmap -sV -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,40677,43497,43896,60823 192.168.56.107 -oN scans/service-enumeration.txt
```

This single scan performs version detection against the complete set of TCP ports identified as open during phase 02.

---

## Next Step

Use the identified services and versions to select technologies that warrant deeper service-specific enumeration.

The next phase focuses on authentication behavior, exposed resources, protocol configuration, shares, filesystem exports, HTTP functionality, and other technology-specific characteristics.

**Next Phase: 04 — Technology Identification**
