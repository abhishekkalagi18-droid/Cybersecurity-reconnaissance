# OSINT Basics

## 1. What is Reconnaissance?

Reconnaissance is the information-gathering phase of a cybersecurity assessment. It involves collecting and analyzing information about a target's digital presence, infrastructure, technologies, domains, and publicly available information before performing further security testing.

The main objective is to understand the target's external attack surface and identify information that may be relevant to a security assessment.

### Objectives of Reconnaissance

* Identify domains and subdomains
* Discover IP addresses and DNS information
* Identify technologies and services
* Collect publicly available organizational information
* Identify potential attack surfaces
* Build an initial understanding of the target environment

---

## 2. Passive vs Active Reconnaissance

Reconnaissance can generally be divided into two approaches.

| Feature           | Passive Reconnaissance                                      | Active Reconnaissance              |
| ----------------- | ----------------------------------------------------------- | ---------------------------------- |
| Interaction       | Uses publicly available information and third-party sources | Directly interacts with the target |
| Target visibility | Usually lower, but not necessarily zero                     | Generally higher                   |
| Information       | Broad contextual information                                | Detailed technical information     |
| Examples          | OSINT, WHOIS, public DNS information                        | Port scanning, service enumeration |
| Common tools      | Search engines, WHOIS, theHarvester                         | Nmap, Nessus                       |

### Passive Reconnaissance

Passive reconnaissance collects information without directly probing the target's infrastructure.

Examples:

* Search engine research
* Public DNS records
* WHOIS/RDAP information
* Public documents
* Job postings
* Public technology information
* theHarvester

### Active Reconnaissance

Active reconnaissance involves directly communicating with or probing systems belonging to the target.

Examples:

* Port scanning
* Service enumeration
* Banner grabbing
* Network discovery
* Vulnerability scanning

> **Important:** Active reconnaissance should only be performed against systems that you own or have explicit authorization to test.

---

## 3. What is OSINT?

**Open-Source Intelligence (OSINT)** is the process of collecting, analyzing, and correlating information from publicly available sources.

In cybersecurity, OSINT can help security professionals understand an organization's public digital footprint before performing technical security testing.

### Common OSINT Sources

* Search engines
* Public DNS records
* WHOIS/RDAP databases
* Public code repositories
* Public documents
* Job postings
* Social media
* Certificate transparency logs
* Public security databases

---

## 4. Domains and Subdomains

A domain represents an organization's presence on the internet.

For example:

```text
example.com
```

Subdomains can represent different services or applications:

```text
www.example.com
mail.example.com
vpn.example.com
dev.example.com
```

Discovering publicly exposed subdomains can help security professionals understand the organization's external attack surface.

### Common Tools

* Amass
* SecurityTrails
* Certificate Transparency logs
* Search engines

---

## 5. DNS Records

The **Domain Name System (DNS)** translates domain names into information used to locate internet services.

Common DNS record types include:

| Record | Purpose                                                            |
| ------ | ------------------------------------------------------------------ |
| A      | Maps a domain to an IPv4 address                                   |
| AAAA   | Maps a domain to an IPv6 address                                   |
| MX     | Identifies mail servers                                            |
| NS     | Identifies authoritative name servers                              |
| TXT    | Stores text-based information such as SPF and verification records |
| CNAME  | Creates an alias for another domain                                |

### Useful Commands

```bash
dig example.com
```

```bash
dig example.com MX
```

```bash
dig example.com TXT
```

```bash
host example.com
```

Only use these against domains where such reconnaissance is permitted.

---

## 6. WHOIS / RDAP

WHOIS and the newer RDAP system can provide publicly available domain registration information.

Depending on the registrar and privacy settings, information may include:

* Registrar
* Domain status
* Registration dates
* Expiration information
* Name servers
* Registration organization information

Example:

```bash
whois example.com
```

> Modern domain registrations frequently use privacy protection, so personal registration information may not be publicly available.

---

## 7. Search Engine Reconnaissance

Search engines can be used to identify publicly indexed information about an organization.

Advanced search operators can help narrow results.

Examples:

```text
site:example.com
```

```text
site:example.com filetype:pdf
```

```text
site:example.com login
```

These techniques can help identify publicly indexed pages and documents.

> Search-engine indexing does not necessarily mean that information is confidential or improperly exposed. Always verify findings before treating them as a security issue.

---

## 8. Email and Domain Discovery

Public sources can sometimes reveal organizational email addresses and naming patterns.

Tools such as **theHarvester** can automate collection of publicly available information from supported sources.

Typical information may include:

* Email addresses
* Subdomains
* Hostnames
* IP addresses
* Publicly indexed sources

Example:

```bash
theHarvester -d example.com -b crtsh
```

Use only authorized domains or security-training targets.

---

## 9. Technology Identification

Technology identification involves determining publicly observable technologies used by a website or service.

Examples include:

* Web servers
* Frameworks
* JavaScript libraries
* CMS platforms
* Cloud services

### Passive Methods

Tools/services such as:

* Wappalyzer
* BuiltWith
* Browser developer tools
* Public source-code inspection

can provide technology information.

### Active Methods

Active techniques may involve directly communicating with the target, such as examining HTTP responses or service banners.

For example:

```bash
curl -I https://example.com
```

This can display HTTP response headers that may reveal information about the web server or application.

---

## 10. Reconnaissance Workflow

A basic reconnaissance workflow can be represented as:

```text
Target
   ↓
Domain Information
   ↓
DNS Enumeration
   ↓
Subdomain Discovery
   ↓
Public Information / OSINT
   ↓
Technology Identification
   ↓
Organize Findings
   ↓
Create Reconnaissance Report
```

---

## 11. What I Learned

Through this topic, I learned:

1. The purpose of reconnaissance in cybersecurity.
2. The difference between passive and active reconnaissance.
3. The fundamentals of OSINT.
4. Common DNS record types.
5. The purpose of WHOIS/RDAP.
6. How search engines can support reconnaissance.
7. The basics of subdomain and email discovery.
8. The difference between passive and active technology identification.
9. How reconnaissance information can be organized into a workflow.

---

## 12. Ethical Considerations

Reconnaissance should be performed responsibly.

* Test only systems you own or have explicit permission to assess.
* Do not attempt to access private information.
* Do not exploit discovered vulnerabilities without authorization.
* Do not use collected information for unauthorized attacks.
* Document the scope and authorization of security testing.

---

## References

* OWASP Amass — https://github.com/owasp-amass/amass
* theHarvester — https://github.com/laramies/theHarvester
* OSINT Framework — https://osintframework.com/
* Maltego — https://www.maltego.com/
* Wappalyzer — https://www.wappalyzer.com/
* BuiltWith — https://builtwith.com/

