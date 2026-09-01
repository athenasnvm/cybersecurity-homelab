# 02 — Port Discovery Commands

## Objective

Identify open TCP ports on the confirmed in-scope laboratory hosts discovered during the previous host discovery phase.

The purpose of this phase is to determine which systems expose network services and establish the initial attack surface for subsequent service enumeration.

---

## 1. Methodology

TCP port discovery was performed from the Kali Linux assessment host against the confirmed laboratory systems.

Two scanning approaches were used:

* Standard Nmap TCP scanning against Ubuntu Server, Windows Server, and Windows Client.
* Full TCP port scanning against Metasploitable2.

Metasploitable2 received a full TCP port scan because it was the primary intentionally vulnerable target of the laboratory.

Raw Nmap output was preserved under the `scans/` directory for later analysis.

---

## 2. Metasploitable2

**Target:** `192.168.56.107`

### Full TCP Port Scan

```bash
nmap -p- 192.168.56.107 -oN scans/port-discovery-meta.txt
```

### Purpose

Performs a full TCP port scan against Metasploitable2.

Unlike the default Nmap scan, which examines the 1,000 most commonly scanned TCP ports, `-p-` instructs Nmap to scan the complete TCP port range.

This provides broader visibility into the attack surface of the primary vulnerable target.

### Command Breakdown

#### `-p-`

Scans all TCP ports instead of limiting discovery to Nmap's default port selection.

#### `192.168.56.107`

Specifies Metasploitable2 as the target.

#### `-oN scans/port-discovery-meta.txt`

Stores the raw Nmap output in normal text format at:

```text
scans/port-discovery-meta.txt
```

### Expected Result

The scan is expected to identify TCP ports accepting connections on Metasploitable2.

Because the system is intentionally configured with multiple vulnerable and legacy services, a comparatively large attack surface is expected.

---

## 3. Ubuntu Server

**Target:** `192.168.56.103`

### Standard TCP Port Scan

```bash
nmap -p- 192.168.56.103 -oN scans/port-discovery-ubuntuserver.txt
```

### Purpose

Performs Nmap's standard TCP port scan against the Ubuntu Server.

This scan examines Nmap's default selection of commonly scanned TCP ports.

### Output

```text
scans/port-discovery-ubuntuserver.txt
```

### Expected Result

The scan determines whether commonly exposed TCP services are reachable on the Ubuntu Server.

No assumption is made that the host must expose an open TCP port.

---

## 4. Windows Server

**Target:** `192.168.56.108`

### Standard TCP Port Scan

```bash
nmap 192.168.56.108 -oN scans/port-discovery-windowsserver.txt
```

### Purpose

Performs Nmap's standard TCP port scan against the Windows Server.

The objective is to identify TCP services exposed through commonly scanned ports.

### Output

```text
scans/port-discovery-windowsserver.txt
```

### Expected Result

The scan determines which commonly scanned TCP ports, if any, are exposed by the Windows Server.

---

## 5. Windows Client

**Target:** `192.168.56.109`

### Standard TCP Port Scan

```bash
nmap 192.168.56.109 -oN scans/port-discovery-windowsclient.txt
```

### Purpose

Performs Nmap's standard TCP port scan against the Windows Client.

The objective is to identify TCP services exposed through commonly scanned ports.

### Output

```text
scans/port-discovery-windowsclient.txt
```

### Expected Result

The scan determines whether the Windows Client exposes TCP services among Nmap's default port selection.

---

## 6. Scan Coverage

The port discovery methodology used during this phase can be summarized as follows:

| Target          | Scan Type          | Coverage          |
| --------------- | ------------------ | ----------------- |
| Metasploitable2 | Full TCP scan      | All TCP ports     |
| Ubuntu Server   | **Full TCP scan**  | **All TCP ports** |
| Windows Server  | Standard Nmap scan | Default TCP ports |
| Windows Client  | Standard Nmap scan | Default TCP ports |


### Important Limitation

A standard Nmap scan does not test every possible TCP port.

Therefore, the absence of open ports during a standard scan should only be interpreted as:

> No open TCP ports were identified within the ports tested by the scan.

It should not be interpreted as proof that the host has no TCP services listening anywhere in the complete port range.

---

## 7. Output

Raw scan results are stored under:

```text
scans/
```

Specifically:

```text
scans/port-discovery-meta.txt
scans/port-discovery-ubuntuserver.txt
scans/port-discovery-windowsserver.txt
scans/port-discovery-windowsclient.txt
```

The discovered ports, host-specific findings, attack-surface comparison, and interpretation of the results are documented separately in:

```text
results.md
```

---

## Next Step

Use the ports identified on the primary target to perform service and version enumeration.

The next phase focuses on determining what technologies are actually listening behind the discovered TCP ports rather than relying solely on Nmap's initial port-based service labels.

**Next Phase: 03 — Service Enumeration**
