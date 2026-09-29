# Service Enumeration – FTP & HTTP

## Overview

Service enumeration is the process of gathering detailed information about
services running on a target.

Today's practice was performed against an authorized Metasploitable 2 lab.

**Target:** `192.168.127.132`
**Platform:** Kali Linux → Metasploitable 2

Topics covered:

* FTP NSE enumeration
* Anonymous FTP
* FTP service information
* HTTP title enumeration
* HTTP headers
* HTTP methods
* HTTP content enumeration
* `phpinfo.php`
* HTTP `OPTIONS`
* Directory listing
* Apache documentation enumeration
* Compressed `.gz` files
* Apache changelog analysis

---

# 1. FTP Enumeration

## 1.1 Finding FTP NSE Scripts

First, available FTP-related NSE scripts were reviewed.

### Command

```bash
nmap --script-help "ftp-*"
```

### Important scripts identified

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

### Observation

Nmap provides different FTP scripts for authentication checks,
information gathering, vulnerability checks, and brute-force testing.

For this exercise, safe enumeration was performed instead of brute-force
or exploit testing.

---

# 2. Anonymous FTP Enumeration

The `ftp-anon` script was used to determine whether anonymous FTP login
was enabled.

### Command

```bash
nmap --script ftp-anon -p 21 192.168.127.132
```

### Result

```text
Anonymous FTP login allowed (FTP code 230)
```

### Finding

Anonymous FTP authentication is enabled on port 21.

### Security Impact

Anonymous access may allow unauthenticated users to access files depending
on the FTP server configuration.

The result confirms anonymous login but does not automatically prove that
sensitive files can be read or modified.

---

# 3. FTP System Information

The `ftp-syst` NSE script was used to gather information from the FTP
service.

### Command

```bash
nmap --script ftp-syst -p21 192.168.127.132
```

### Important output

```text
Connected to 192.168.127.128
Logged in as ftp
TYPE ASCII
No session bandwidth limit
Session timeout 300 sec
Control connection plain text
Data connections plain text
vsFTPd 2.3.4
```

### Observations

The FTP service disclosed:

* `vsFTPd 2.3.4`
* Session information
* Session timeout
* Plain-text control connection
* Plain-text data connection

### Security Note

Traditional FTP does not provide encryption by default. Credentials and
transferred data can therefore be exposed to network interception on an
untrusted network.

---

# 4. HTTP Title Enumeration

The `http-title` NSE script was used to identify web page titles.

### Command

```bash
nmap --script http-title -p 80,8180 192.168.127.132
```

### Results

```text
80/tcp     Metasploitable2 - Linux
8180/tcp   Apache Tomcat/5.5
```

### Observation

Two HTTP services were identified:

* Port `80` → Apache HTTP Server
* Port `8180` → Apache Tomcat

---

# 5. HTTP Header Enumeration

The `http-headers` NSE script was used to inspect HTTP response headers.

### Command

```bash
nmap --script http-headers 192.168.127.132
```

### Port 80

```text
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Connection: close
Content-Type: text/html
```

### Port 8180

```text
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=ISO-8859-1
Connection: close
```

### Observation

The response headers disclosed server and technology information.

This information is useful for service and technology fingerprinting.

---

# 6. HTTP Method Enumeration

The `http-methods` NSE script was used to identify supported HTTP methods.

### Command

```bash
nmap --script http-methods 192.168.127.132
```

### Observed methods

```text
GET
HEAD
POST
OPTIONS
```

### Observation

The HTTP services reported the above methods.

A supported HTTP method is not automatically a vulnerability. It is an
enumeration result that requires further investigation.

---

# 7. HTTP Content Enumeration

The `http-enum` NSE script was used to identify potentially interesting
web paths.

### Command

```bash
nmap --script http-enum -p80 192.168.127.132
```

### Discovered paths

```text
/tikiwiki/
/test/
/phpinfo.php
/phpMyAdmin/
/doc/
/icons/
/index/
```

### Observation

These paths were identified as potentially interesting resources.

The discovery of a path alone does not prove that the resource is
vulnerable.

---

# 8. phpinfo.php Investigation

One of the paths discovered by `http-enum` was:

```text
/phpinfo.php
```

Full URL:

```text
http://192.168.127.132/phpinfo.php
```

## 8.1 Check Response Headers

### Command

```bash
curl -I http://192.168.127.132/phpinfo.php
```

### Result

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html
```

### Observation

The `phpinfo.php` endpoint was publicly accessible and returned
HTTP `200 OK`.

---

# 9. phpinfo Information Disclosure

The full `phpinfo()` page was investigated.

Information exposed included:

* PHP version
* Linux system information
* Configuration paths
* PHP configuration
* PHP modules
* Database/library information
* Session configuration

### Finding

**Publicly Accessible phpinfo() Endpoint**

### Impact

A public `phpinfo()` page can disclose detailed information about the
server and PHP environment.

This information can assist technology fingerprinting and further
reconnaissance.

The presence of `phpinfo()` alone does not prove remote code execution or
complete system compromise.

---

# 10. OPTIONS Request

An HTTP `OPTIONS` request was sent directly to `phpinfo.php`.

### Command

```bash
curl -I -X OPTIONS http://192.168.127.132/phpinfo.php
```

### Result

```text
HTTP/1.1 200 OK
Date: Mon, 28 Sep 2026 16:19:28 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Length: 48014
Content-Type: text/html
```

### Observation

The endpoint successfully processed the `OPTIONS` request.

No `Allow:` header was returned in the observed response, so the complete
list of allowed methods could not be determined from this response alone.

---

# 11. /doc/ Directory Investigation

The `/doc/` path discovered using `http-enum` was investigated.

## 11.1 Check Directory

### Command

```bash
curl -I http://192.168.127.132/doc/
```

### Result

```text
HTTP/1.1 200 OK
Date: Mon, 28 Sep 2026 16:37:10 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2
Content-Type: text/html;charset=UTF-8
```

The directory content was then retrieved.

### Command

```bash
curl http://192.168.127.132/doc/
```

The page displayed:

```text
Index of /doc/
```

### Finding

Directory listing was enabled on `/doc/`.

---

# 12. Extracting Links from /doc/

Instead of manually reading the complete HTML page, links were extracted
using `grep`.

### Command

```bash
curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30
```

### Results

```text
href="?C=N;O=D"
href="?C=M;O=A"
href="?C=S;O=A"
href="?C=D;O=A"
href="/"
href="acl/"
href="adduser/"
href="ant/"
href="antlr/"
href="apache2-mpm-prefork/"
href="apache2-utils/"
href="apache2.2-common/"
href="apache2/"
href="apparmor-utils/"
href="apparmor/"
href="apt-utils/"
href="apt/"
href="aptitude/"
href="at/"
href="attr/"
href="autoconf/"
href="autoconf2.59/"
href="base-files/"
href="base-passwd/"
href="bash-completion/"
href="bash/"
href="belocs-locales-bin/"
href="bind9-host/"
href="bind9/"
href="binutils/"
```

### Observation

The directory listing exposed numerous package documentation directories.

Examples:

```text
apache2/
apache2-utils/
apache2.2-common/
apparmor/
apt/
bash/
bind9/
binutils/
```

This provides additional software/package information for fingerprinting.

---

# 13. Understanding the Link Extraction Command

Command:

```bash
curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30
```

## curl

```bash
curl -s http://192.168.127.132/doc/
```

* `curl` retrieves the HTTP response.
* `-s` enables silent mode.

## Pipe

```text
|
```

The pipe sends the output of one command to the next command.

## grep

```bash
grep -Eo 'href="[^"]+"'
```

* `-E` → Extended Regular Expressions
* `-o` → Print only matching text

The pattern:

```text
href="[^"]+"
```

extracts `href` attributes from the HTML.

Example:

```html
<a href="apache2/">apache2</a>
```

becomes:

```text
href="apache2/"
```

## head

```bash
head -30
```

Displays only the first 30 results.

---

# 14. Relative URL Investigation

One extracted link was:

```text
href="apache2/"
```

Because this link appeared inside `/doc/`, the resolved path is:

```text
/doc/apache2/
```

An attempt was made to access:

### Command

```bash
curl -s http://192.168.127.132/apache2/ | head -30
```

### Result

```text
404 Not Found
```

The server reported:

```text
The requested URL /apache2/ was not found on this server.
```

### Observation

The path was incorrect.

The link `apache2/` was relative to `/doc/`, therefore the correct path
is:

```text
/doc/apache2/
```

This demonstrated the importance of understanding relative URLs during
web enumeration.

---

# 15. Apache Documentation Directory

The correct Apache documentation path was accessed.

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/ | head -30
```

### Result

The server returned:

```text
Index of /doc/apache2
```

The directory contained:

```text
NEWS.Debian.gz
changelog.Debian.gz
copyright
```

The listing also showed file sizes:

```text
NEWS.Debian.gz       1.0K
changelog.Debian.gz  30K
copyright            31K
```

### Observation

Apache package documentation was publicly accessible through the directory
listing.

---

# 16. Extracting Apache Documentation Links

The available links were extracted with:

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/ | grep -Eo 'href="[^"]+"' | tail -10
```

### Result

```text
href="?C=N;O=D"
href="?C=M;O=A"
href="?C=S;O=A"
href="?C=D;O=A"
href="/doc/"
href="NEWS.Debian.gz"
href="changelog.Debian.gz"
href="copyright"
```

### Observation

Three documentation files were confirmed:

```text
NEWS.Debian.gz
changelog.Debian.gz
copyright
```

---

# 17. Reading the Apache Copyright File

The plain-text `copyright` file was retrieved.

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/copyright | tail -30
```

### Observation

The output contained:

* Software licensing information
* Copyright notices
* Apache-related documentation
* OpenDocument icon licensing information

This is not a vulnerability by itself.

The important observation is that package documentation was publicly
readable through the web server.

---

# 18. Compressed Changelog Investigation

The `changelog.Debian.gz` file was identified as a gzip-compressed file.

## 18.1 Initial Attempt

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | head
```

### Result

The terminal displayed unreadable characters.

### Explanation

The `.gz` file contains compressed binary data.

The data must be decompressed before it can be read as text.

---

# 19. Decompressing the Changelog

The compressed changelog was decompressed directly through a pipeline.

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | gzip -dc | head -30
```

### Result

The beginning of the changelog showed:

```text
apache2 (2.2.8-1) unstable; urgency=low
```

The changelog also contained historical security-related entries including:

```text
CVE-2007-5000
CVE-2007-6388
CVE-2007-6421
CVE-2007-6422
CVE-2008-0005
```

### Important Observation

These CVEs appeared as historical entries in the Apache package changelog.

Their presence in the changelog does **not** prove that the target is
currently vulnerable to those CVEs.

---

# 20. Extracting Apache Package Versions

The Apache package entries were extracted from the decompressed changelog.

### Command

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | gzip -dc | grep -m 5 "^apache2"
```

### Result

```text
apache2 (2.2.8-1) unstable; urgency=low
apache2 (2.2.6-3) unstable; urgency=low
apache2 (2.2.6-2) unstable; urgency=low
apache2 (2.2.6-1) unstable; urgency=low
apache2 (2.2.4-3) unstable; urgency=low
```

### Observation

The latest Apache package version listed in the changelog was:

```text
2.2.8-1
```

This is consistent with the HTTP service information previously observed:

```text
Apache/2.2.8 (Ubuntu)
```

---

# 21. Understanding the Changelog Command

Command:

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | gzip -dc | grep -m 5 "^apache2"
```

### `curl -s`

Downloads the compressed changelog silently.

### `gzip -dc`

Decompresses the gzip data and sends the result to standard output.

### `grep`

Searches the decompressed text.

### `-m 5`

Stops after five matching lines.

### `^apache2`

The `^` character means the line must begin with:

```text
apache2
```

---

# 22. Enumeration Chain

The investigation followed this progression:

```text
FTP
 ↓
Anonymous FTP
 ↓
FTP service information
 ↓
HTTP
 ↓
HTTP titles
 ↓
HTTP headers
 ↓
HTTP methods
 ↓
HTTP path enumeration
 ↓
/phpinfo.php
 ↓
/doc/
 ↓
Directory listing
 ↓
/doc/apache2/
 ↓
Apache documentation
 ↓
changelog.Debian.gz
 ↓
gzip decompression
 ↓
Apache package history
```

---

# 23. Findings Summary

| Service | Finding / Observation            | Evidence                   |
| ------- | -------------------------------- | -------------------------- |
| FTP     | Anonymous login enabled          | `ftp-anon` → FTP code 230  |
| FTP     | Service information exposed      | `vsFTPd 2.3.4`             |
| FTP     | Plain-text FTP connections       | `ftp-syst` output          |
| HTTP    | Apache service                   | Apache/2.2.8               |
| HTTP    | Tomcat service                   | Apache Tomcat/5.5          |
| HTTP    | Technology disclosure            | HTTP response headers      |
| HTTP    | `phpinfo()` exposed              | `/phpinfo.php` → HTTP 200  |
| HTTP    | Directory listing enabled        | `/doc/` → `Index of /doc/` |
| HTTP    | Package documentation exposed    | `/doc/apache2/`            |
| HTTP    | Apache changelog accessible      | `changelog.Debian.gz`      |
| HTTP    | Apache package history disclosed | `2.2.8-1` listed           |

---

# 24. Complete Commands Used

## FTP

```bash
nmap --script-help "ftp-*"
```

```bash
nmap --script ftp-anon -p 21 192.168.127.132
```

```bash
nmap --script ftp-syst -p21 192.168.127.132
```

## HTTP

```bash
nmap --script http-title -p 80,8180 192.168.127.132
```

```bash
nmap --script http-headers 192.168.127.132
```

```bash
nmap --script http-methods 192.168.127.132
```

```bash
nmap --script http-enum -p80 192.168.127.132
```

```bash
curl -I http://192.168.127.132/phpinfo.php
```

```bash
curl -I -X OPTIONS http://192.168.127.132/phpinfo.php
```

```bash
curl -I http://192.168.127.132/doc/
```

```bash
curl http://192.168.127.132/doc/
```

```bash
curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30
```

```bash
curl -s http://192.168.127.132/apache2/ | head -30
```

```bash
curl -s http://192.168.127.132/doc/apache2/ | head -30
```

```bash
curl -s http://192.168.127.132/doc/apache2/ | grep -Eo 'href="[^"]+"' | tail -10
```

```bash
curl -s http://192.168.127.132/doc/apache2/copyright | tail -30
```

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | head
```

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | gzip -dc | head -30
```

```bash
curl -s http://192.168.127.132/doc/apache2/changelog.Debian.gz | gzip -dc | grep -m 5 "^apache2"
```

---

# 25. Key Learning Outcomes

Today's practice covered:

* FTP NSE script discovery
* Anonymous FTP enumeration
* FTP service fingerprinting
* HTTP service enumeration
* HTTP response headers
* HTTP methods
* Web content enumeration
* `phpinfo()` information disclosure
* HTTP `OPTIONS` requests
* Apache directory indexing
* HTML link extraction
* `grep` regular expressions
* Linux pipes
* Relative URLs
* HTTP redirects
* HTTP `200 OK` and `404 Not Found`
* Nested web-directory enumeration
* Gzip-compressed files
* `gzip -dc`
* Apache package changelogs
* Extracting version information with `grep`
* Distinguishing information disclosure from confirmed vulnerability

---

## Lab Scope

All testing documented here was performed against the authorized
Metasploitable 2 laboratory target:

```text
192.168.127.132
```

The purpose was cybersecurity learning, service enumeration, and
reconnaissance practice.
