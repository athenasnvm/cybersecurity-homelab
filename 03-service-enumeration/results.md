# Service Enumeration Results

## Summary

Service and version detection was performed against the open TCP ports previously identified on Metasploitable2.

The assessment confirmed a broad range of remotely accessible services, including file transfer, remote administration, web, database, file sharing, RPC, and application services.

**Target:** `192.168.56.107`
**Host:** Metasploitable2
**Open TCP Ports Assessed:** 30

|  Port | Service    | Identified Technology / Version     |
| ----: | ---------- | ----------------------------------- |
|    21 | FTP        | vsftpd 2.3.4                        |
|    22 | SSH        | OpenSSH 4.7p1 Debian 8ubuntu1       |
|    23 | Telnet     | Linux telnetd                       |
|    25 | SMTP       | Postfix smtpd                       |
|    53 | DNS        | ISC BIND 9.4.2                      |
|    80 | HTTP       | Apache httpd 2.2.8                  |
|   111 | RPC        | rpcbind 2                           |
|   139 | SMB        | Samba smbd 3.X–4.X                  |
|   445 | SMB        | Samba smbd 3.X–4.X                  |
|   512 | rexec      | netkit-rsh rexecd                   |
|   513 | rlogin     | Login service                       |
|   514 | rsh        | Netkit rshd                         |
|  1099 | Java RMI   | GNU Classpath grmiregistry          |
|  1524 | Bind Shell | Metasploitable root shell           |
|  2049 | NFS        | NFS v2–v4                           |
|  2121 | FTP        | ProFTPD 1.3.1                       |
|  3306 | MySQL      | MySQL 5.0.51a-3ubuntu5              |
|  3632 | distccd    | distccd v1 / GNU 4.2.4              |
|  5432 | PostgreSQL | PostgreSQL 8.3.0–8.3.7              |
|  5900 | VNC        | VNC protocol 3.3                    |
|  6000 | X11        | X11 — Access denied                 |
|  6667 | IRC        | UnrealIRCd                          |
|  6697 | IRC        | UnrealIRCd                          |
|  8009 | AJP13      | Apache Jserv Protocol 1.3           |
|  8180 | HTTP       | Apache Tomcat/Coyote JSP engine 1.1 |
|  8787 | DRb        | Ruby DRb RMI / Ruby 1.8             |
| 40677 | RPC Status | RPC #100024                         |
| 43497 | Java RMI   | GNU Classpath grmiregistry          |
| 43896 | mountd     | RPC mountd v1–v3                    |
| 60823 | nlockmgr   | RPC nlockmgr v1–v4                  |

---

## 1. File Transfer Services

Multiple file transfer services were identified.

### FTP — Port 21

```text
vsftpd 2.3.4
```

### FTP — Port 2121

```text
ProFTPD 1.3.1
```

Two separate FTP implementations were therefore exposed by the target.

At this stage, service enumeration only identified the technologies and versions. Authentication configuration and potential vulnerabilities required further investigation.

**Status:** Identified

---

## 2. Remote Access Services

Several services capable of providing remote system access were identified.

| Port | Service | Identification                |
| ---: | ------- | ----------------------------- |
|   22 | SSH     | OpenSSH 4.7p1 Debian 8ubuntu1 |
|   23 | Telnet  | Linux telnetd                 |
|  512 | rexec   | netkit-rsh rexecd             |
|  513 | rlogin  | Login service                 |
|  514 | rsh     | Netkit rshd                   |
| 5900 | VNC     | VNC protocol 3.3              |

The presence of multiple remotely accessible administration and login protocols increases the number of services requiring subsequent security analysis.

No authentication bypass or unauthorized access was established during this phase.

**Status:** Multiple Remote Access Services Identified

---

## 3. Web and Application Services

Several web and application-related services were identified.

### Apache HTTP Server

```text
80/tcp
Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```

### Apache JServ Protocol

```text
8009/tcp
Apache Jserv (Protocol v1.3)
```

### Apache Tomcat

```text
8180/tcp
Apache Tomcat/Coyote JSP engine 1.1
```

These results established multiple web-facing technologies for subsequent application and configuration enumeration.

**Status:** Identified

---

## 4. File Sharing and RPC Services

SMB, NFS and multiple RPC-related services were detected.

### SMB

```text
139/tcp
445/tcp
Samba smbd 3.X - 4.X
```

### NFS

```text
2049/tcp
NFS v2-v4
```

Supporting RPC services included:

* rpcbind
* mountd
* nlockmgr
* RPC status

The presence of these services justified additional enumeration of shares, exports, permissions, and remote accessibility.

**Status:** Identified — Further Enumeration Required

---

## 5. Database Services

Two database management systems were remotely accessible.

### MySQL

```text
3306/tcp
MySQL 5.0.51a-3ubuntu5
```

### PostgreSQL

```text
5432/tcp
PostgreSQL 8.3.0 - 8.3.7
```

No database authentication or data-access conclusions were made during this phase.

**Status:** Identified

---

## 6. distccd

The distributed compilation service was identified on TCP port `3632`.

```text
distccd v1
GNU 4.2.4
Ubuntu 4.2.4-1ubuntu4
```

The service was remotely reachable and its identification provided sufficient information for later vulnerability research.

No exploitation was performed during service enumeration.

**Status:** Identified — Further Research Required

---

## 7. Additional Network Services

Several additional services were detected:

|  Port | Service  | Identification             |
| ----: | -------- | -------------------------- |
|    25 | SMTP     | Postfix smtpd              |
|    53 | DNS      | ISC BIND 9.4.2             |
|  1099 | Java RMI | GNU Classpath grmiregistry |
|  6000 | X11      | Access denied              |
|  6667 | IRC      | UnrealIRCd                 |
|  6697 | IRC      | UnrealIRCd                 |
|  8787 | DRb      | Ruby DRb RMI / Ruby 1.8    |
| 43497 | Java RMI | GNU Classpath grmiregistry |

These services were recorded as part of the exposed network attack surface but were not individually assessed during this enumeration phase.

**Status:** Identified

---

## 8. Bind Shell Detection

Nmap identified TCP port `1524` as:

```text
Metasploitable root shell
```

This was recorded as a significant service identification result.

However, service detection alone was not treated as proof that unauthorized root access had been obtained through this port during this phase.

**Status:** Service Identified — Not Validated During This Phase

---

## 9. Service Enumeration Assessment

Metasploitable2 exposed a broad and diverse collection of remotely accessible services across all 30 TCP ports assessed.

The identified attack surface included:

* File transfer services
* Remote login and administration protocols
* Web and application servers
* SMB and NFS file sharing
* Database servers
* RPC infrastructure
* Distributed compilation
* IRC
* Java RMI
* Ruby DRb
* X11

The results provided the technical foundation required for subsequent technology identification, configuration analysis, and vulnerability research.

The presence of a service or legacy version was not considered proof of vulnerability during this phase.

### Final Result

| Property                           | Result            |
| ---------------------------------- | ----------------- |
| Target                             | Metasploitable2   |
| IP Address                         | `192.168.56.107`  |
| Open TCP Ports Assessed            | 30                |
| FTP Implementations                | 2                 |
| Database Services                  | MySQL, PostgreSQL |
| Primary Web Server                 | Apache 2.2.8      |
| SMB                                | Detected          |
| NFS                                | Detected          |
| distccd                            | Detected          |
| Remote Access Services             | Multiple          |
| Further Security Analysis Required | Yes               |

**Final Status: Service Enumeration Completed**
