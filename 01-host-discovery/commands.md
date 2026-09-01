# 01 — Host Discovery Commands

## Objective

Identify active hosts within the isolated laboratory network before performing port and service enumeration.

The purpose of this phase is to establish which systems are reachable from the Kali Linux assessment host and define the targets available for subsequent reconnaissance.

---

## 1. Network Host Discovery

### Tool

Nmap

### Command

```bash
nmap -sn 192.168.56.0/24 -oN scans/host-discovery.txt
```

### Purpose

Performs host discovery across the laboratory subnet:

```text
192.168.56.0/24
```

The `-sn` option disables port scanning and limits the operation to determining which hosts are active.

This makes the command appropriate for the initial discovery phase because the objective is to identify reachable systems rather than enumerate their exposed services.

---

## 2. Command Breakdown

### `nmap`

Runs the Nmap network scanning tool.

### `-sn`

Performs host discovery without conducting a port scan.

The objective is to determine which IP addresses correspond to active hosts.

### `192.168.56.0/24`

Defines the laboratory subnet to be scanned.

The `/24` network includes the address range associated with the isolated VirtualBox laboratory network.

### `-oN scans/host-discovery.txt`

Stores the Nmap output in normal text format at:

```text
scans/host-discovery.txt
```

Saving the raw scan output preserves the original discovery evidence independently from the interpreted findings documented in `results.md`.

---

## 3. Expected Result

The scan is expected to identify the virtual machines and network infrastructure currently reachable within the laboratory subnet.

No assumption is made that every detected IP address necessarily represents an in-scope assessment target.

Unexpected hosts should be identified before being included in subsequent testing.

---

## 4. Scope Validation

After host discovery, detected systems should be compared against the known laboratory inventory.

Only confirmed laboratory systems should proceed to subsequent active testing.

Unknown or unexpected hosts should remain outside the testing scope until their identity and ownership are established.

---

## 5. Output

Raw Nmap output:

```text
scans/host-discovery.txt
```

The interpreted host discovery findings are documented separately in:

```text
results.md
```

---

## Next Step

Use the confirmed in-scope hosts identified during this phase as targets for TCP port discovery.

**Next Phase: 02 — Port Discovery**
