# Host Discovery Results

## Summary

Host discovery was performed on the isolated laboratory network to identify active systems and verify the availability of the expected virtual machines.

A total of six additional IP addresses were detected alongside the VirtualBox host interface.

| IP Address       | Host                 | Expected | Detected | In Scope |
| ---------------- | -------------------- | :------: | :------: | :------: |
| `192.168.56.1`   | VirtualBox Interface |    Yes   |    Yes   |    No    |
| `192.168.56.100` | Unknown              |    No    |    Yes   |  Unknown |
| `192.168.56.103` | Ubuntu Server        |    Yes   |    Yes   |    Yes   |
| `192.168.56.106` | Kali Linux           |    Yes   |    Yes   |    Yes   |
| `192.168.56.107` | Metasploitable2      |    Yes   |    Yes   |    Yes   |
| `192.168.56.108` | Windows Server       |    Yes   |    Yes   |    Yes   |
| `192.168.56.109` | Windows Client       |    Yes   |    Yes   |    Yes   |

---

## 1. Expected Hosts

The expected laboratory systems were successfully detected on the network:

* Kali Linux — `192.168.56.106`
* Metasploitable2 — `192.168.56.107`
* Ubuntu Server — `192.168.56.103`
* Windows Server — `192.168.56.108`
* Windows Client — `192.168.56.109`

This confirmed that the primary systems required for the laboratory were reachable during the discovery phase.

**Status:** Confirmed

---

## 2. VirtualBox Interface

The address `192.168.56.1` was identified as the VirtualBox interface.

This address was expected as part of the virtualized laboratory environment and was not included as an assessment target.

**Status:** Detected — Out of Scope

---

## 3. Unidentified Host

An additional system was detected at:

```text
192.168.56.100
```

This address was not part of the expected host inventory and its role was not identified during this phase.

No assumptions were made regarding the operating system, purpose, or ownership of this host.

**Status:** Detected — Unidentified

---

## 4. Discovery Assessment

Host discovery successfully identified all expected laboratory systems.

The results established the active host inventory used for subsequent network enumeration and assessment phases.

At this stage, discovery only confirmed host availability. No conclusions regarding exposed services, vulnerabilities, or security posture were made from these results alone.

### Final Result

| Property                    | Result   |
| --------------------------- | -------- |
| Expected Lab Systems        | 5        |
| Expected Systems Detected   | 5        |
| Additional Unknown Host     | 1        |
| VirtualBox Interface        | Detected |
| Primary Lab Hosts Reachable | Yes      |

**Final Status: Host Discovery Completed**
