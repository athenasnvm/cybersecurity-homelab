# Privilege Escalation Results

## Summary

Privilege escalation testing was performed from the unprivileged `daemon` session previously obtained through exploitation of the distccd service.

Post-exploitation enumeration had identified `/usr/bin/nmap` as a root-owned SUID binary running the legacy Nmap 4.53 version.

The SUID configuration was successfully leveraged through Nmap interactive mode to execute a shell with an effective user ID of `0`.

The resulting shell was subsequently validated by accessing a security-sensitive file that had previously been inaccessible to the `daemon` account.

| Property              | Result       |
| --------------------- | ------------ |
| Initial Access Vector | distccd RCE  |
| Initial Account       | `daemon`     |
| Initial UID           | 1            |
| Initial Privilege     | Unprivileged |
| Escalation Vector     | SUID Nmap    |
| Nmap Version          | 4.53         |
| SUID Owner            | `root`       |
| Resulting EUID        | 0 (`root`)   |
| Sensitive File Access | Confirmed    |
| Privilege Escalation  | Confirmed    |
| Severity              | Critical     |

---

## 1. Initial Security Context

Privilege escalation testing began from the remote shell previously obtained through distccd.

The compromised session operated as:

```text
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

The account did not initially possess root privileges.

Previous post-exploitation testing had also established that the `daemon` account could not read security-sensitive resources such as:

```text
/etc/shadow
```

### Assessment

The `daemon` session represented an unprivileged initial foothold and provided a clear baseline against which successful privilege escalation could be validated.

**Status:** Confirmed — Unprivileged Initial Access

---

## 2. SUID Nmap Identification

Post-exploitation enumeration identified the following executable:

```text
/usr/bin/nmap
```

with root ownership and the SUID permission bit enabled:

```text
-rwsr-xr-x root root /usr/bin/nmap
```

The installed version was identified as:

```text
Nmap 4.53
```

### Assessment

The SUID permission causes the executable to run with the effective privileges of its owner.

Because Nmap was owned by `root`, its execution context was evaluated as a potential local privilege escalation vector.

The legacy Nmap version also supported interactive mode, making the configuration particularly security-sensitive.

**Status:** Confirmed — Privilege Escalation Candidate

---

## 3. Nmap Interactive Mode

The legacy Nmap installation supported interactive mode.

It was started using:

```bash
nmap --interactive
```

A system shell was then executed from the Nmap interactive prompt:

```text
nmap> !sh
```

### Assessment

Because the Nmap executable was configured with SUID root permissions, the shell launched through the privileged Nmap process inherited an effective root security context.

The resulting shell was then independently verified before considering the privilege escalation successful.

**Status:** Shell Execution Successful

---

## 4. Privilege Verification

The resulting shell was validated using:

```bash
whoami
id
```

The shell reported:

```text
root
```

and:

```text
uid=1(daemon) gid=1(daemon) euid=0(root) groups=1(daemon)
```

### Real UID vs Effective UID

The real user ID remained:

```text
uid=1(daemon)
```

while the effective user ID became:

```text
euid=0(root)
```

The effective UID determines the privileges used by the process for access-control decisions.

Therefore, despite retaining the original real UID associated with `daemon`, the shell operated with effective root privileges.

### Result

| Property        | Before Escalation | After Escalation |
| --------------- | ----------------- | ---------------- |
| Real UID        | 1 (`daemon`)      | 1 (`daemon`)     |
| Effective UID   | 1 (`daemon`)      | 0 (`root`)       |
| Privilege Level | Unprivileged      | Effective Root   |
| `whoami`        | `daemon`          | `root`           |

**Status:** Confirmed — Effective Root Privileges Obtained

---

## 5. Sensitive File Access Validation

The practical impact of the privilege escalation was validated using:

```text
/etc/shadow
```

Before privilege escalation, the original `daemon` session was unable to read this file.

After obtaining:

```text
euid=0(root)
```

the file became readable from the escalated shell.

The password hashes contained in `/etc/shadow` were intentionally excluded from the project documentation.

### Assessment

This test demonstrated that the privilege escalation was not limited to a change in reported identity.

The escalated process was able to bypass a filesystem access restriction that had previously prevented the `daemon` account from accessing the protected resource.

This provided practical confirmation of effective root privileges.

**Status:** Confirmed — Privileged File Access

---

## 6. Root Directory Access Validation

The root user's directory was inspected without modifying its contents.

The directory was identified as:

```text
drwxr-xr-x root root /root
```

Read access from the escalated shell was successfully confirmed.

Because the directory permissions themselves allowed read and execute access beyond the owner, this result was treated as supplementary context rather than the primary proof of privilege escalation.

The `/etc/shadow` access validation provided the stronger privilege-boundary test.

**Status:** Access Confirmed — Supporting Evidence

---

## 7. Privilege Escalation Attack Chain

The complete escalation path demonstrated during the laboratory was:

```text
distccd
CVE-2004-2687
        ↓
Remote Command Execution
        ↓
daemon
UID 1
        ↓
Post-Exploitation Enumeration
        ↓
SUID Binary Discovery
        ↓
/usr/bin/nmap
SUID root
        ↓
Nmap 4.53
Interactive Mode
        ↓
!sh
        ↓
uid=1(daemon)
euid=0(root)
        ↓
Effective Root Privileges
```

### Assessment

The attack chain demonstrates how an initial remote command execution vulnerability providing only an unprivileged foothold can be combined with a local security misconfiguration to achieve effective root privileges.

The distccd vulnerability provided initial access, while the unsafe SUID configuration of the legacy Nmap binary enabled the local privilege escalation.

---

## 8. Security Impact

Successful exploitation resulted in effective root privileges on the target.

The practical impact was demonstrated through access to `/etc/shadow`, which had been inaccessible before privilege escalation.

The demonstrated impact affects the primary security properties of the system:

### Confidentiality

Effective root privileges allow access to protected information.

This impact was directly demonstrated through successful access to `/etc/shadow`.

**Impact:** Critical

### Integrity

Effective root privileges provide the security context required to modify root-protected system resources.

No destructive modification was required to demonstrate the privilege escalation.

**Impact:** Critical

### Availability

Effective root privileges provide administrative-level control over system resources and processes.

No availability-impacting actions were performed during the laboratory.

**Impact:** Critical

---

## 9. Privilege Escalation Assessment

The privilege escalation phase successfully demonstrated a complete transition from an unprivileged remote foothold to effective root privileges.

The initial distccd exploitation produced:

```text
uid=1(daemon)
```

Post-exploitation enumeration subsequently identified:

```text
/usr/bin/nmap
```

as a root-owned SUID executable running Nmap 4.53.

Using the interactive functionality available in this legacy Nmap version, a shell was executed that obtained:

```text
euid=0(root)
```

The practical impact was independently validated by demonstrating access to `/etc/shadow`, which the original `daemon` session could not read.

No persistence mechanisms, password modifications, additional user accounts, destructive filesystem changes, or service disruptions were required.

---

## Final Result

| Property                               | Result                 |
| -------------------------------------- | ---------------------- |
| Initial Access                         | distccd RCE            |
| Initial Account                        | `daemon`               |
| Initial UID                            | 1                      |
| Initial Privilege                      | Unprivileged           |
| Escalation Candidate                   | `/usr/bin/nmap`        |
| Nmap Version                           | 4.53                   |
| SUID Owner                             | `root`                 |
| Escalation Technique                   | Nmap Interactive Shell |
| Real UID After Escalation              | 1 (`daemon`)           |
| Effective UID After Escalation         | 0 (`root`)             |
| `/etc/shadow` Access Before Escalation | Denied                 |
| `/etc/shadow` Access After Escalation  | Confirmed              |
| Effective Root Access                  | Confirmed              |
| Severity                               | Critical               |

**Final Status: Privilege Escalation Completed — Effective Root Access Confirmed**
