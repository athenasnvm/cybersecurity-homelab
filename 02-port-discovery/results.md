# Port Discovery Results

## Summary

TCP port discovery was performed against the in-scope laboratory hosts to identify exposed network services and compare their externally reachable attack surface.

Metasploitable2 presented significantly more open TCP ports than the other assessed systems and was therefore selected as the primary target for further service enumeration.

| Host            | IP Address       | Open TCP Ports | Initial Assessment             |
| --------------- | ---------------- | -------------: | ------------------------------ |
| Metasploitable2 | `192.168.56.107` |             30 | High exposed attack surface    |
| Ubuntu Server   | `192.168.56.103` |              0 | No open TCP ports identified   |
| Windows Server  | `192.168.56.108` |              1 | Limited exposed attack surface |
| Windows Client  | `192.168.56.109` |              0 | No open TCP ports identified   |

> The assessment in this phase is based only on the TCP ports identified during port discovery. The presence or absence of open ports does not by itself confirm whether a host is vulnerable.

---

## 1. Metasploitable2

**Target:** `192.168.56.107`

A total of **30 open TCP ports** were identified on the Metasploitable2 host.

Compared with the other systems assessed during this phase, Metasploitable2 exposed the largest number of reachable TCP services.

The number of exposed ports indicated a broad network attack surface and justified prioritizing the host for subsequent service and version identification.

**Status:** 30 Open TCP Ports Identified — Prioritized for Further Enumeration

---

## 2. Ubuntu Server

**Target:** `192.168.56.103`

No open TCP ports were identified during this discovery phase.

This result indicates that the scan did not identify externally reachable TCP services within the tested port range.

The result does not establish that the system has no running services or vulnerabilities; it only reflects the network exposure observed during this scan.

**Status:** No Open TCP Ports Identified

---

## 3. Windows Server

**Target:** `192.168.56.108`

One open TCP port was identified during port discovery.

Compared with Metasploitable2, the host presented a substantially smaller externally reachable TCP attack surface during this phase.

Further enumeration would be required to identify the service associated with the open port and determine its security relevance.

**Status:** 1 Open TCP Port Identified

---

## 4. Windows Client

**Target:** `192.168.56.109`

No open TCP ports were identified during this discovery phase.

As with the Ubuntu Server, this result only represents the externally reachable TCP services identified during the scan and does not constitute a vulnerability assessment of the system.

**Status:** No Open TCP Ports Identified

---

## 5. Port Discovery Assessment

The port discovery phase identified a clear difference in network exposure between the laboratory systems.

Metasploitable2 exposed **30 TCP ports**, substantially more than any other assessed host. Based on network exposure alone, it represented the most appropriate target for deeper service enumeration.

At this stage, no vulnerability conclusions were made. Open ports only indicate reachable network services and require additional identification and security analysis.

### Final Result

| Property                                   | Result          |
| ------------------------------------------ | --------------- |
| Hosts Assessed                             | 4               |
| Host with Largest TCP Exposure             | Metasploitable2 |
| Metasploitable2 Open TCP Ports             | 30              |
| Ubuntu Server Open TCP Ports               | 0               |
| Windows Server Open TCP Ports              | 1               |
| Windows Client Open TCP Ports              | 0               |
| Target Prioritized for Service Enumeration | Metasploitable2 |

**Final Status: Port Discovery Completed**
