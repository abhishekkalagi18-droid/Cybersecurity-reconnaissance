# Nmap Basics

## 1. What is Nmap?

Nmap (Network Mapper) is a network scanning and reconnaissance tool used to discover hosts, open ports, services, service versions, and operating-system information.

In this lab, I used Nmap against an authorized **Metasploitable** virtual machine to understand active reconnaissance.

> Only perform Nmap scanning against systems you own or have explicit permission to test.

---

# 2. Basic Nmap Scan

Command:

```bash
nmap 192.168.127.132
```

The basic scan identified that the host was up and discovered open TCP ports.

Example findings included:

```text
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
```

The scan also reported that many other scanned TCP ports were closed.

### What I learned

A basic Nmap scan provides an initial view of the network services exposed by a host.

---

# 3. Service and Version Detection

Command:

```bash
nmap -sV 192.168.127.132
```

The `-sV` option attempts to identify the service and version running on an open port.

Examples from the lab:

```text
21/tcp   open  ftp       vsftpd 2.3.4
22/tcp   open  ssh       OpenSSH 4.7p1
80/tcp   open  http      Apache httpd 2.2.8
3306/tcp open  mysql     MySQL 5.0.51a
5432/tcp open  postgresql PostgreSQL 8.3.x
8180/tcp open  http      Apache Tomcat/Coyote
```

### Why version detection matters

Knowing the service and version allows a security tester to research whether that specific software version has known security issues.

However:

> A detected version does not automatically mean the service is vulnerable.

Configuration and the actual software installation must also be considered.

---

# 4. Default NSE Scripts

Command:

```bash
nmap -sC 192.168.127.132
```

The `-sC` option runs Nmap's default NSE scripts.

The scan provided additional information about several services.

Examples:

```text
FTP:
Anonymous FTP login allowed

SMTP:
SMTP commands and SSL information

DNS:
BIND version information

RPC:
RPC services and ports

SMB:
Operating system and security configuration information

MySQL:
Database protocol and version information

VNC:
VNC protocol information

IRC:
IRC server information

HTTP:
Web page title
```

### What I learned

NSE scripts provide information beyond simply identifying whether a port is open.

---

# 5. Targeted Port Scanning

Command:

```bash
nmap -p 21,22,80,445,3306,8180 192.168.127.132
```

The `-p` option allows specific ports to be selected.

In this exercise I focused on:

```text
21    FTP
22    SSH
80    HTTP
445   SMB
3306  MySQL
8180  Tomcat/HTTP
```

### Why targeted scanning is useful

After discovering services, a security tester can focus subsequent enumeration on specific ports instead of repeatedly scanning every port.

---

# 6. Full TCP Port Scan

Command:

```bash
nmap -p- 192.168.127.132
```

The `-p-` option scans TCP ports from:

```text
1 → 65535
```

The full scan discovered additional ports that were not present in the initial default scan.

Additional ports included:

```text
3632/tcp  distccd
6697/tcp  ircs-u
8787/tcp  msgsrvr
42224/tcp unknown
47631/tcp unknown
50971/tcp unknown
54560/tcp unknown
```

This demonstrated that services can run on less commonly scanned ports.

---

# 7. Identifying Newly Discovered Services

After discovering additional ports, I used:

```bash
nmap -sV -p 3632,6697,8787,42224,47631,50971,54560 192.168.127.132
```

The services were identified as:

```text
3632/tcp  distccd     distccd v1
6697/tcp  irc         UnrealIRCd
8787/tcp  drb         Ruby DRb RMI
42224/tcp java-rmi    GNU Classpath grmiregistry
47631/tcp nlockmgr    RPC
50971/tcp mountd      RPC
54560/tcp status      RPC
```

### Reconnaissance workflow

```text
Full port scan
      ↓
Discover additional ports
      ↓
Service/version detection
      ↓
Identify the services
      ↓
Investigate the services
```

---

# 8. Operating System Detection

Command:

```bash
nmap -O 192.168.127.132
```

The result identified:

```text
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
```

### What I learned

The `-O` option attempts to fingerprint the operating system using network characteristics.

OS detection should be treated as a fingerprint or estimate rather than absolute proof.

---

# 9. Combining Nmap Options

Command:

```bash
nmap -sC -sV -O 192.168.127.132
```

This combines:

```text
-sC → Default NSE scripts
-sV → Service/version detection
-O  → OS detection
```

Combining options provides a broader reconnaissance view of the target.

---

# 10. NSE — Nmap Scripting Engine

NSE allows Nmap to run scripts designed for specific reconnaissance, discovery, authentication, and security-testing tasks.

To view FTP-related scripts:

```bash
nmap --script-help "ftp-*"
```

The Kali installation contained scripts including:

```text
ftp-anon
ftp-bounce
ftp-brute
ftp-libopie
ftp-proftpd-backdoor
ftp-syst
ftp-vsftpd-backdoor
ftp-vuln-cve2010-4221
```

NSE scripts have different categories, including:

```text
default
safe
discovery
auth
vuln
intrusive
exploit
brute
```

Not every NSE script has the same safety characteristics.

---

# 11. HTTP NSE Enumeration

Command:

```bash
nmap --script http-title -p 80 192.168.127.132
```

Result:

```text
80/tcp open http
|_http-title: Metasploitable2 - Linux
```

The same scan also identified the HTTP title on port 8180:

```text
8180/tcp open
|_http-title: Apache Tomcat/5.5
```

### Why `ftp-title` does not exist

The `http-title` script is designed to retrieve the HTML page title from an HTTP response.

FTP does not use HTML page titles, so there is no corresponding `ftp-title` script.

Instead, FTP has protocol-specific scripts such as:

```text
ftp-anon
ftp-syst
ftp-bounce
```

---

# 12. Anonymous FTP Enumeration

Command:

```bash
nmap --script ftp-anon -p 21 192.168.127.132
```

Result:

```text
21/tcp open ftp
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

### Finding

**Finding:** Anonymous FTP login enabled

**Port:** 21/TCP

**Service:** FTP

**Evidence:** `ftp-anon` NSE script

**Result:** Anonymous login was allowed.

### Security significance

Anonymous FTP can allow users without normal credentials to access files exposed through the FTP service.

The scan confirms anonymous login, but this result alone does not establish what files are accessible or whether they are writable.

---

# 13. How Nmap Information Is Used in VAPT

A typical reconnaissance workflow is:

```text
Target
  ↓
Port discovery
  ↓
Service identification
  ↓
Version detection
  ↓
OS fingerprinting
  ↓
NSE enumeration
  ↓
Research applicable weaknesses
  ↓
Validate findings in an authorized environment
  ↓
Document evidence and remediation
```

For example:

```text
21/tcp
   ↓
FTP
   ↓
vsftpd 2.3.4
   ↓
Anonymous login confirmed
   ↓
Further authorized assessment
```

The important lesson is that **Nmap findings are evidence for further investigation, not automatic proof of compromise or vulnerability**.

---

# 14. Nmap Options Learned

| Option          | Purpose                        |
| --------------- | ------------------------------ |
| `nmap TARGET`   | Basic port scan                |
| `-sV`           | Service/version detection      |
| `-sC`           | Default NSE scripts            |
| `-p`            | Scan specific ports            |
| `-p-`           | Scan all TCP ports             |
| `-O`            | OS detection                   |
| `--script`      | Run a specific NSE script      |
| `--script-help` | Display NSE script information |

---

# 15. Practical Reconnaissance Workflow

The commands practiced in this lab can be organized as:

```bash
# 1. Basic scan
nmap TARGET

# 2. Service/version detection
nmap -sV TARGET

# 3. Default NSE enumeration
nmap -sC TARGET

# 4. Specific ports
nmap -p 21,22,80 TARGET

# 5. Full TCP scan
nmap -p- TARGET

# 6. Identify newly discovered services
nmap -sV -p PORTS TARGET

# 7. OS detection
nmap -O TARGET

# 8. Specific NSE script
nmap --script SCRIPT -p PORT TARGET
```

---

# 16. What I Learned

Through this Nmap lab, I learned:

1. How to perform a basic TCP port scan.
2. How to identify services and versions with `-sV`.
3. How default NSE scripts provide additional information.
4. How to scan specific ports using `-p`.
5. How to scan all TCP ports using `-p-`.
6. Why uncommon ports can be important during reconnaissance.
7. How to identify the operating system using `-O`.
8. How to use individual NSE scripts.
9. How to inspect available NSE scripts with `--script-help`.
10. How to confirm anonymous FTP access using `ftp-anon`.
11. How reconnaissance findings become inputs for further authorized security assessment.

---

# 17. Ethical Considerations

Nmap is a powerful active-reconnaissance tool.

Only scan:

* Your own systems
* Your own virtual machines
* Authorized penetration-testing environments
* Systems where you have explicit permission

A port being open does not mean it should be attacked.

The goal of reconnaissance is to understand the exposed attack surface and provide evidence that can be used for authorized security testing and remediation.

---

# 18. Conclusion

This lab demonstrated how Nmap can progressively reveal information about a target:

```text
Open ports
    ↓
Services
    ↓
Versions
    ↓
Operating system
    ↓
NSE information
    ↓
Security findings
```

The main lesson is that effective reconnaissance is **systematic**. Instead of immediately attempting exploitation, a security tester first builds an accurate picture of the target's exposed services and configuration.
