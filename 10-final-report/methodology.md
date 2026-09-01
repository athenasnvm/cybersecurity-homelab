# Assessment Methodology

## Overview

The security assessment followed a structured methodology designed to progress from initial target discovery to vulnerability validation, exploitation, post-exploitation analysis, privilege escalation, risk assessment, and final reporting.

The laboratory workflow was organized into sequential phases:

```text
01 — Network Discovery
        ↓
02 — Port Discovery
        ↓
03 — Service Enumeration
        ↓
04 — Service-Specific Enumeration
        ↓
05 — Vulnerability Research
        ↓
06 — Exploitation
        ↓
07 — Post-Exploitation
        ↓
08 — Privilege Escalation
        ↓
09 — Findings & Risk Assessment
        ↓
10 — Final Security Report
```

Each phase was intended to answer a different security question.

Rather than immediately attempting known Metasploitable2 exploits, the assessment attempted to reproduce a more structured workflow where exploitation decisions were based on information collected during previous phases.

---

# 1. Laboratory Environment

The assessment was conducted inside an isolated virtual laboratory.

## Assessment Host

```text
Operating System: Kali Linux
IP Address: 192.168.56.106
Role: Assessment / Attacker System
```

## Target

```text
System: Metasploitable2
IP Address: 192.168.56.107
Role: Vulnerable Target
```

Metasploitable2 is intentionally vulnerable and was used exclusively for cybersecurity training.

The environment was isolated from untrusted networks to prevent vulnerable services from being unintentionally exposed outside the laboratory.

---

# 2. Methodological Principles

Several principles were followed throughout the assessment.

## Evidence Before Conclusion

Findings were based on observed laboratory results rather than expected behavior.

The general process was:

```text
Observation
    ↓
Hypothesis
    ↓
Technical Test
    ↓
Collected Evidence
    ↓
Assessment
```

A vulnerability was not considered successfully exploited solely because the target software version appeared vulnerable.

---

## Enumeration Before Exploitation

Exploitation was not treated as the starting point of the assessment.

Instead:

```text
Discover
   ↓
Enumerate
   ↓
Research
   ↓
Validate
   ↓
Exploit
```

This allowed exploitation attempts to be connected to previously identified services and vulnerabilities.

---

## Separate Identification from Exploitation

The assessment distinguished between:

```text
Service Detected
      ↓
Potential Vulnerability
      ↓
Vulnerability Applicable
      ↓
Exploitation Attempt
      ↓
Successful Exploitation
      ↓
Observed Impact
```

This distinction was particularly important when evaluating HTTP vulnerabilities, where several potential weaknesses and legacy applications were identified but remote command execution was not demonstrated.

---

## Controlled Validation

Where possible, testing was limited to the actions required to demonstrate security impact.

For example, the NFS `no_root_squash` condition was validated using a temporary file rather than modifying an existing sensitive system file.

---

# 3. Phase 01 — Network Discovery

## Objective

Identify active hosts within the laboratory network and determine the IP address assigned to the target system.

At this stage, the assessment did not assume which services or vulnerabilities were present.

The objective was simply:

```text
Laboratory Network
        ↓
Active Hosts
        ↓
Target Identification
```

Network discovery established:

```text
Kali Linux
192.168.56.106

Metasploitable2
192.168.56.107
```

The identified Metasploitable2 address became the target used throughout the remaining assessment.

---

# 4. Phase 02 — Port Discovery

## Objective

Identify exposed TCP ports on the target.

After establishing that the host was reachable, port discovery was used to determine which network services could potentially be accessed.

The workflow became:

```text
192.168.56.107
       ↓
TCP Port Discovery
       ↓
Open Ports
       ↓
Potential Attack Surface
```

The target exposed numerous network services.

Rather than treating every open port as a vulnerability, the results were used as input for subsequent service enumeration.

This distinction is important:

```text
Open Port
    ≠
Vulnerability
```

An open port only demonstrates that a network service is reachable.

---

# 5. Phase 03 — Service Enumeration

## Objective

Identify the software and protocols associated with the discovered ports.

Service enumeration attempted to determine information such as:

```text
Protocol
Software
Version
Banner
Operating System Indicators
Service Behavior
```

This phase transformed the port-discovery results into a more useful representation of the target's attack surface.

The workflow progressed from:

```text
Port
  ↓
Service
  ↓
Software
  ↓
Version
```

This information later became important when researching known vulnerabilities.

---

# 6. Phase 04 — Service-Specific Enumeration

## Objective

Perform deeper analysis of individual services.

Generic service detection was not sufficient to understand the security state of the target.

Selected services were therefore examined using protocol-specific enumeration techniques.

The assessment investigated services including:

```text
FTP
HTTP
SMB
NFS / RPC
DistCC
```

Service-specific enumeration attempted to identify characteristics such as:

```text
Anonymous Access
Shared Resources
Exported Filesystems
HTTP Applications
Web Directories
Service Configuration
Protocol Capabilities
Authentication Requirements
```

This phase provided the technical context required for meaningful vulnerability research.

---

# 7. Phase 05 — Vulnerability Research

## Objective

Correlate the technologies and configurations discovered during enumeration with known vulnerabilities and security weaknesses.

Research was based on the actual characteristics identified on the target.

The general process was:

```text
Enumerated Service
        ↓
Software / Version / Configuration
        ↓
Vulnerability Research
        ↓
Potential CVE or Misconfiguration
        ↓
Applicability Assessment
```

Examples of vulnerability candidates identified during this process included:

```text
DistCC
→ CVE-2004-2687

vsftpd 2.3.4
→ CVE-2011-2523

Samba
→ CVE-2007-2447

NFS
→ Insecure no_root_squash Configuration

HTTP
→ Legacy Applications and Security Exposures
```

---

## Vulnerability Classification

Potential vulnerabilities were not automatically classified as confirmed exploitation.

The research phase distinguished between conditions such as:

```text
Candidate
Applicable by Version
Appears Vulnerable
Configuration Confirmed
Not Reproduced
```

Practical exploitation was reserved for the following phase.

---

# 8. Phase 06 — Exploitation and Security Validation

## Objective

Determine whether identified vulnerabilities and security weaknesses could produce practical security impact.

Controlled exploitation was performed against selected attack surfaces.

The methodology followed:

```text
Research Finding
      ↓
Exploit / Validation Technique
      ↓
Observed Response
      ↓
Session or Access
      ↓
Identity Validation
      ↓
Impact Assessment
```

---

## DistCC

CVE-2004-2687 was tested using the corresponding Metasploit module.

An initial reverse-shell mechanism failed because the target environment did not support the attempted `/dev/tcp` mechanism.

The payload was adjusted and a Perl-based reverse shell successfully established a command session.

The resulting execution context was:

```text
daemon
UID 1
GID 1
```

This demonstrated remote command execution without direct root privileges.

---

## FTP

The vsftpd `2.3.4` service was tested for CVE-2011-2523.

The backdoor was successfully triggered and a remote session was established.

The resulting identity was:

```text
root
UID 0
GID 0
```

This demonstrated direct root-level remote compromise.

---

## SMB

CVE-2007-2447 was tested against the Samba service.

Successful exploitation established a command shell.

The resulting identity was:

```text
root
UID 0
GID 0
```

This provided another independent path to full system compromise.

---

## NFS

NFS required a different validation methodology because the weakness was a configuration issue rather than remote command execution.

The target exported:

```text
/
```

using:

```text
rw
no_root_squash
```

and wildcard client access.

The filesystem was mounted remotely and a temporary test file was created as root from the assessment host.

The remote file was created with:

```text
root:root
```

ownership.

This demonstrated preservation of remote UID `0` and privileged filesystem access.

---

## HTTP

HTTP validation focused on determining the practical impact of the applications and resources discovered during enumeration.

Testing confirmed conditions including:

```text
Version Disclosure
WebDAV Exposure
Public phpinfo()
Public phpMyAdmin Interface
TikiWiki 1.9.5
Exposed TikiWiki Installer
Database Information Disclosure
Filesystem Path Disclosure
Test Directory Exposure
Directory Listing
```

However:

```text
Remote Code Execution: Not Demonstrated
Command Shell: Not Obtained
```

The HTTP results were therefore classified according to the demonstrated exposure rather than represented as successful system compromise.

---

# 9. Session Validation

Obtaining a shell or session was not considered sufficient by itself.

Where remote command execution was achieved, the resulting execution context was validated using system information such as:

```text
whoami
id
hostname
uname -a
pwd
```

Depending on the session type, equivalent Meterpreter commands were also used.

The purpose was to confirm:

```text
Who am I?
      ↓
What privileges do I have?
      ↓
Which machine am I controlling?
      ↓
What operating system is running?
      ↓
Where am I executing?
```

This prevented a successful connection message from being treated as the only evidence of compromise.

---

# 10. Phase 07 — Post-Exploitation

## Objective

Enumerate the compromised system after obtaining initial remote access.

The DistCC shell was particularly useful for this phase because it provided an unprivileged execution context.

Initial access was:

```text
User: daemon
UID: 1
Privilege: Unprivileged
```

Post-exploitation enumeration examined areas including:

```text
System Identity
Users and Groups
Operating System
Processes
Services
Filesystem Permissions
Sensitive Files
Scheduled Tasks
SUID / SGID Binaries
Potential Privilege-Escalation Paths
```

The purpose was not simply to collect system information.

Instead, the phase attempted to answer:

```text
Initial Access
      ↓
What can this account see?
      ↓
What can this account execute?
      ↓
What security boundaries exist?
      ↓
Is privilege escalation possible?
```

---

# 11. Phase 08 — Privilege Escalation

## Objective

Determine whether the unprivileged DistCC shell could be elevated to root-level execution.

Local enumeration identified multiple SUID binaries.

One particularly relevant binary was:

```text
/usr/bin/nmap
```

The binary was owned by root and had the SUID permission enabled.

The installed version was:

```text
Nmap 4.53
```

The legacy version supported functionality that could be used to execute commands while retaining the effective privileges provided by the SUID configuration.

---

## Privilege-Escalation Validation

The identified Nmap condition was tested from the compromised `daemon` session.

Successful validation resulted in:

```text
whoami
→ root
```

and:

```text
uid=1(daemon)
gid=1(daemon)
euid=0(root)
groups=1(daemon)
```

This distinction was preserved in the documentation.

The real user remained:

```text
daemon
```

while the effective user became:

```text
root
```

Therefore, the demonstrated result was effective root-level execution rather than a replacement of the original account identity.

---

## Attack Chain

The complete escalation path was:

```text
DistCC
   ↓
CVE-2004-2687
   ↓
Remote Command Shell
   ↓
daemon (UID 1)
   ↓
Post-Exploitation Enumeration
   ↓
SUID Nmap 4.53
   ↓
Effective UID 0
   ↓
Root-Level Execution
```

This demonstrated how an initially limited remote compromise could be combined with a local configuration weakness to increase impact.

---

# 12. Phase 09 — Findings and Risk Assessment

## Objective

Transform technical observations into structured security findings.

The assessment results were reviewed to distinguish between:

```text
Observation
Vulnerability
Misconfiguration
Successful Exploitation
Security Impact
```

The risk-assessment phase considered both technical severity and demonstrated impact.

Particular importance was given to weaknesses that resulted in:

```text
Remote Code Execution
Root-Level Execution
Privilege Escalation
Privileged Filesystem Access
Information Disclosure
```

The purpose was to avoid evaluating vulnerabilities solely according to their names or CVE identifiers.

Instead, the practical consequences observed in the laboratory were incorporated into the assessment.

---

# 13. Phase 10 — Final Reporting

## Objective

Consolidate the complete laboratory assessment into a professional and understandable security report.

The final reporting phase separates information into several audiences and purposes:

```text
README
→ Assessment Overview

Executive Summary
→ Major Security Outcomes

Methodology
→ Assessment Process

Findings
→ Technical Security Issues

Remediation
→ Recommended Corrective Actions

Evidence Summary
→ Technical Evidence Mapping
```

The final report does not replace the detailed documentation contained in phases 01–09.

Instead, it provides a consolidated interpretation of that technical work.

---

# 14. Evidence Collection

Raw technical outputs were retained throughout the assessment.

Evidence was generally separated into:

```text
commands.md / command.md
→ Commands and testing procedures

results.md
→ Interpretation of results

scans/
→ Raw command and tool output

evidence/
→ Supporting screenshots where available
```

Not every phase required every evidence type.

For example, a raw text scan containing complete exploitation output may provide stronger technical evidence than an additional screenshot containing the same information.

---

# 15. Evidence Integrity

A key reporting principle was:

```text
Claim
  ↓
Must Be Supported
  ↓
By Observed Evidence
```

When expected results differed from observed results, the actual result was documented.

Examples included:

* The initial DistCC payload failed before a compatible payload was selected.
* HTTP vulnerability research did not automatically result in confirmed exploitation.
* A WebDAV PROPFIND request returned `403 Forbidden` under the tested conditions.
* phpMyAdmin exposure did not demonstrate authentication bypass.
* NFS root identity preservation did not automatically imply that a remote root shell had been obtained.

This approach reduces the risk of overstating the results of the assessment.

---

# 16. Use of Automation and Security Tools

Multiple tools were used throughout the assessment for different purposes.

Examples include:

| Tool / Technology    | Primary Purpose                                   |
| -------------------- | ------------------------------------------------- |
| Nmap                 | Host, port, service, and script-based enumeration |
| Metasploit Framework | Controlled exploitation                           |
| curl                 | HTTP request and response validation              |
| FTP Client           | FTP authentication and resource validation        |
| NFS Utilities        | Export enumeration and filesystem mounting        |
| Linux Utilities      | Local enumeration and privilege validation        |
| Markdown             | Technical documentation and reporting             |

Tool output was interpreted in the context of the assessment rather than treated as a conclusion by itself.

---

# 17. AI-Assisted Methodology

Artificial intelligence was used as an educational and productivity aid throughout the project.

AI assistance supported activities such as:

```text
Concept Explanation
Troubleshooting
Command Selection
Output Interpretation
Documentation Structure
Technical Writing Review
```

However, AI-generated statements were not treated as technical evidence.

The methodology required suggested techniques to be tested against the actual laboratory environment.

Therefore:

```text
AI Guidance
     ↓
Manual Laboratory Execution
     ↓
Observed Target Response
     ↓
Raw Evidence
     ↓
Technical Interpretation
```

When the laboratory output contradicted an expected result, the observed result was considered authoritative for the purposes of this report.

---

# 18. Scope Limitations

The original laboratory design considered additional systems, including:

```text
Ubuntu Server
Windows Server
Windows Client
```

These systems were ultimately excluded from the final assessment scope.

The decision was made to prioritize depth of analysis and documentation against Metasploitable2 rather than performing a broader but less detailed assessment across several targets.

As a result, the methodology documented in this report applies specifically to:

```text
192.168.56.107
Metasploitable2
```

No conclusions should be inferred regarding the security posture of systems that were not included in the final testing scope.

---

# 19. Methodology Summary

The complete assessment methodology can be summarized as:

```text
Discover
   ↓
Enumerate
   ↓
Understand
   ↓
Research
   ↓
Validate
   ↓
Exploit
   ↓
Enumerate Locally
   ↓
Escalate Privileges
   ↓
Assess Impact
   ↓
Document Evidence
   ↓
Recommend Remediation
```

This structure allowed the project to move beyond simply executing known exploits against an intentionally vulnerable machine.

The assessment instead focused on understanding how information gathered at each stage influences the next stage of a penetration-testing workflow.

---

# Conclusion

The methodology used throughout this laboratory emphasized structured testing, evidence collection, and practical validation.

Network discovery established the target.

Port and service enumeration defined the attack surface.

Service-specific analysis provided deeper technical context.

Vulnerability research identified potential weaknesses.

Controlled exploitation demonstrated which weaknesses could produce practical access.

Post-exploitation enumeration identified additional local security conditions.

Privilege-escalation testing demonstrated how an unprivileged DistCC compromise could be elevated to effective root-level execution.

Risk assessment transformed technical results into security findings.

Finally, the reporting phase consolidated the complete workflow into a documented security assessment.

The resulting methodology can be summarized by one central principle:

```text
Do Not Assume Impact
       ↓
Test It
       ↓
Observe It
       ↓
Preserve the Evidence
       ↓
Report Only What Was Demonstrated
```
