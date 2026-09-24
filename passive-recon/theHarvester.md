# theHarvester — Passive Reconnaissance

## 1. Overview

**theHarvester** is an OSINT tool used during reconnaissance to collect publicly available information related to a domain or organization.

It can collect information such as:

* Hosts and subdomains
* Email addresses
* IP addresses
* URLs
* ASN information
* Other information from supported sources

This lab focuses on understanding theHarvester's capabilities, different data sources, and limitations through practical testing.

> **Authorization:** Reconnaissance should only be performed against systems that you own or have explicit permission to assess.

---

# 2. Installation & Setup

I used **Kali Linux** as the operating system for this practical exercise.

## 2.1 Check Whether theHarvester Is Installed

First, verify whether theHarvester is available:

```bash
theHarvester --help
```

If the help menu is displayed, the tool is installed and accessible from the terminal.

## 2.2 Check the Version

Check the installed version:

```bash
theHarvester --version
```

My installed version:

```text
theHarvester 4.10.1
```

## 2.3 Install theHarvester on Kali Linux

If the tool is not already installed, update the package lists:

```bash
sudo apt update
```

Install theHarvester:

```bash
sudo apt install theharvester
```

Verify the installation:

```bash
theHarvester --version
```

Then check the available options:

```bash
theHarvester --help
```

## 2.4 Verify Configuration

TheHarvester reads its proxy configuration from:

```text
/etc/theHarvester/proxies.yaml
```

During my practical exercises, the tool displayed:

```text
Read proxies.yaml from /etc/theHarvester/proxies.yaml
```

This confirmed that the configuration file was being read successfully.

## 2.5 Verify Available Backends

Run:

```bash
theHarvester --help
```

The `-b` option displays the supported data sources/backends.

Examples available in my installation included:

```text
crtsh
urlscan
waybackarchive
github-code
virustotal
shodan
dnsdumpster
securityTrails
```

The exact availability and requirements of individual backends can depend on the installed version and whether API credentials are required.

## Installation Verification Summary

| Check         | Command                  | Result     |
| ------------- | ------------------------ | ---------- |
| Installation  | `theHarvester --help`    | Successful |
| Version       | `theHarvester --version` | 4.10.1     |
| Configuration | `proxies.yaml`           | Loaded     |
| Backend list  | `theHarvester --help`    | Available  |


## 2. Environment

| Item    | Details               |
| ------- | --------------------- |
| Tool    | theHarvester          |
| Version | 4.10.1                |
| OS      | Kali Linux            |
| Target  | `certifiedhacker.com` |

---

## 3. Basic Syntax

```bash
theHarvester -d <domain> -b <source>
```

Example:

```bash
theHarvester -d certifiedhacker.com -b crtsh
```

### Important Options Tested

| Option | Purpose                        |
| ------ | ------------------------------ |
| `-d`   | Specify the domain             |
| `-b`   | Select a data source/backend   |
| `-l`   | Limit search results           |
| `-S`   | Specify starting result number |
| `-r`   | Perform DNS resolution         |
| `-n`   | Enable DNS server lookup       |
| `-c`   | Perform DNS brute force        |
| `-f`   | Save results as XML and JSON   |

---

# 4. Activity 1 — Certificate Transparency

### Command

```bash
theHarvester -d certifiedhacker.com -b crtsh
```

### Result

```text
IPs found: 0
Emails found: 0
People found: 0
Hosts found: 26
```

The CRT.sh backend returned **26 hosts** associated with the target domain.

### Observation

Certificate Transparency data can reveal hostnames that have appeared in publicly logged TLS certificates.

Finding a hostname does not automatically mean that the host is currently active or vulnerable.

---

# 5. Activity 2 — DNS Resolution

### Command

```bash
theHarvester -d certifiedhacker.com -b crtsh -r
```

### Result

```text
IPs found: 2

162.241.216.11
66.235.200.145

Hosts found: 46
```

Examples:

```text
autoconfig.certifiedhacker.com:162.241.216.11
blog.certifiedhacker.com:162.241.216.11
ciphershield.certifiedhacker.com:66.235.200.145
mail.certifiedhacker.com:162.241.216.11
www.certifiedhacker.com:162.241.216.11
```

Some discovered hosts did not have an IP address shown.

### What I Learned

The `-r` option performs DNS resolution on discovered hosts.

This demonstrates:

```text
Hostname discovery
       ↓
DNS resolution
       ↓
IP association
```

---

# 6. Activity 3 — DNS Lookup

### Command

```bash
theHarvester -d certifiedhacker.com -b crtsh -n
```

### Result

The tool reported:

```text
Starting active queries for DNSLookup.
```

No additional hosts were reported after the reverse lookup.

### What I Learned

The `-n` option performs an **active DNS lookup phase**.

Therefore, it should not be considered purely passive reconnaissance.

---

# 7. Activity 4 — DNS Brute Force

### Command

```bash
theHarvester -d certifiedhacker.com -b crtsh -c
```

### Result

The tool attempted DNS brute forcing but returned:

```text
[Errno 2] No such file or directory:
'/usr/lib/python3/dist-packages/theHarvester/data/wordlists/dns-names.txt'
```

### What I Learned

The DNS brute-force functionality requires a wordlist.

The expected `dns-names.txt` file was missing from the installation path.

This demonstrated that installed tools can still have missing dependencies or supporting files.

---

# 8. Activity 5 — Result Limits

### Commands

```bash
theHarvester -d certifiedhacker.com -b crtsh -l 10 -S 0
```

```bash
theHarvester -d certifiedhacker.com -b crtsh -l 10 -S 10
```

### Observation

Both commands returned the same 26 hosts from the CRT.sh backend.

The tool documents `-l` as the result limit and `-S` as the starting result number. However, this particular backend did not produce visibly different output when `-S` was changed.

### Lesson

Backend behavior can affect how command-line options behave in practice.

---

# 9. Activity 6 — Output Export

### Command

```bash
theHarvester -d certifiedhacker.com -b crtsh -f results
```

### Result

```text
XML File saved.
JSON File saved.
```

### What I Learned

The `-f` option allows reconnaissance results to be saved in structured XML and JSON formats.

A useful workflow is:

```text
Collect
   ↓
Save
   ↓
Analyze
   ↓
Document
   ↓
Report
```

---

# 10. Activity 7 — Wayback Archive

### Command

```bash
theHarvester -d certifiedhacker.com -b waybackarchive
```

### Result

No useful information was returned for the target during this test.

### Observation

An OSINT source may not always provide results for a particular target.

A lack of results does not prove that the target has no historical information.

### What I Learned

Different OSINT sources have different datasets and coverage. A reconnaissance workflow should therefore use multiple appropriate sources rather than relying on a single source.

---

# 11. Activity 8 — URLScan

### Command

```bash
theHarvester -d certifiedhacker.com -b urlscan
```

### Results

```text
ASNs found: 2
AS31898
AS46606

Interesting URLs found: 5

http://certifiedhacker.com/
http://www.certifiedhacker.com/
https://certifiedhacker.com/
https://demo.certifiedhacker.com/
https://www.certifiedhacker.com/

IPs found: 1

162.241.216.11

Hosts found: 1

demo.certifiedhacker.com

Emails found: 0
People found: 0
```

### Findings Summary

| Category         | Result |
| ---------------- | -----: |
| ASNs             |      2 |
| Interesting URLs |      5 |
| IP addresses     |      1 |
| Hosts            |      1 |
| Emails           |      0 |
| People           |      0 |

### What I Learned

The URLScan backend provided information different from the CRT.sh backend.

CRT.sh primarily helped discover certificate-associated hosts, while URLScan returned publicly observed URLs, an IP address, host information, and ASN information.

This demonstrates the value of using multiple OSINT sources.

---

# 12. Comparing the Sources

| Backend          | Main Information Observed            |
| ---------------- | ------------------------------------ |
| `crtsh`          | Certificate-associated hosts         |
| `waybackarchive` | Historical web information           |
| `urlscan`        | URLs, hosts, IPs and ASN information |

### Key Lesson

No single OSINT source necessarily provides a complete picture.

A better reconnaissance process is:

```text
Source 1
   ↓
Source 2
   ↓
Source 3
   ↓
Correlate findings
   ↓
Validate
   ↓
Document
```

---

# 13. Overall Practical Findings

| Test                    | Main Result                                  |
| ----------------------- | -------------------------------------------- |
| CRT.sh                  | 26 hosts                                     |
| CRT.sh + DNS resolution | 2 IPs and 46 host entries                    |
| DNS lookup              | Active DNS lookup performed                  |
| DNS brute force         | Missing required wordlist                    |
| Result pagination       | No visible change with `-S` for this backend |
| XML/JSON export         | Successful                                   |
| Wayback Archive         | No useful result                             |
| URLScan                 | 2 ASNs, 5 URLs, 1 IP, 1 host                 |

---

# 14. Important Reconnaissance Lessons

### Discovery ≠ Vulnerability

Finding a hostname, IP address, or URL does not automatically indicate a vulnerability.

### Different Sources Give Different Information

CRT.sh and URLScan produced different types of reconnaissance data.

### DNS Adds Context

DNS resolution can associate discovered hostnames with IP addresses.

### Active and Passive Techniques Differ

Some theHarvester options, such as `-n`, perform active DNS queries.

### Tool Errors Are Useful Findings

The DNS brute-force test demonstrated that a feature can depend on additional files such as wordlists.

### Results Need Validation

OSINT information may be incomplete, outdated, or dependent on the source's coverage.

---

# 15. Troubleshooting

### DNS Brute Force Error

```text
dns-names.txt: No such file or directory
```

The expected wordlist was not present at the path reported by the tool.

Instead of treating this as a failed learning exercise, it was documented as a troubleshooting observation.

---

# 16. What I Learned

Through this practical exercise I learned:

1. How to use theHarvester 4.10.1.
2. How to select different OSINT backends.
3. How CRT.sh can reveal certificate-associated hosts.
4. How DNS resolution can associate hostnames with IP addresses.
5. How active DNS lookup differs from passive collection.
6. Why DNS brute forcing requires a wordlist.
7. How result limits and starting offsets work.
8. How to export results to XML and JSON.
9. How historical sources may return no results.
10. How URLScan can provide URLs, hosts, IPs and ASN information.
11. Why multiple OSINT sources should be correlated.
12. Why reconnaissance findings require validation.

---

# 17. Ethical Considerations

Reconnaissance should only be performed against:

* Systems you own
* Authorized penetration-testing targets
* CTF environments
* Security training laboratories

Do not use collected information for unauthorized access, exploitation, credential attacks, or social engineering.

---

# 18. Conclusion

This practical exercise demonstrated that theHarvester can combine information from different OSINT sources to build a broader picture of a target's public digital footprint.

The most important lesson was not simply learning individual commands, but understanding the reconnaissance process:

```text
Discover
   ↓
Collect
   ↓
Correlate
   ↓
Validate
   ↓
Document
```

Different sources may produce different or incomplete results, so findings should be analyzed in context rather than treated as automatically accurate or indicative of vulnerabilities.
