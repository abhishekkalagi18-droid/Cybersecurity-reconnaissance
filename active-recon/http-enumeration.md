# Service Enumeration – FTP & HTTP

## Overview

Today I performed authorized service enumeration on a **Metasploitable 2** lab machine using **Kali Linux**.

### Target

* **IP:** `192.168.127.132`
* **Environment:** Kali Linux → Metasploitable 2

### Topics Covered

* FTP enumeration
* HTTP enumeration
* Nmap NSE scripts
* HTTP headers and methods
* Web path enumeration
* `phpinfo()` information disclosure
* Directory listing
* HTML link extraction
* Relative URL handling

---

# 1. FTP Enumeration

## 1.1 Finding FTP NSE Scripts

```bash
nmap --script-help "ftp-*"
```

Important scripts identified:

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

For this lab, safe enumeration scripts such as `ftp-anon` and `ftp-syst` were used.

Brute-force and exploit scripts were not executed.

---

## 1.2 Anonymous FTP Enumeration

The `ftp-anon` NSE script checks whether anonymous FTP authentication is enabled.

```bash
nmap --script ftp-anon -p 21 192.168.127.132
```

### Result

```text
Anonymous FTP login allowed (FTP code 230)
```

### Finding

Anonymous FTP authentication is enabled on port `21`.

### Security Impact

Anonymous access may allow unauthenticated users to access files depending on the FTP server configuration.

---

## 1.3 FTP System Information

```bash
nmap --script ftp-syst -p21 192.168.127.132
```

Important information obtained:

```text
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
* FTP session information
* Session timeout
* Plain-text control connection
* Plain-text data connection

---

# 2. HTTP Enumeration

## 2.1 HTTP Title Enumeration

```bash
nmap --script http-title -p 80,8180 192.168.127.132
```

### Results

```text
80/tcp     Metasploitable2 - Linux
8180/tcp   Apache Tomcat/5.5
```

### Observation

Two web services were identified:

* Port `80` → Apache HTTP Server
* Port `8180` → Apache Tomcat

---

## 2.2 HTTP Headers

```bash
nmap --script http-headers 192.168.127.132
```

### Port 80

```text
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html
```

### Port 8180

```text
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=ISO-8859-1
```

### Observation

HTTP response headers disclosed server and technology information useful for service fingerprinting.

---

## 2.3 HTTP Methods

```bash
nmap --script http-methods 192.168.127.132
```

Observed methods:

```text
GET
HEAD
POST
OPTIONS
```

> A supported HTTP method is not automatically a vulnerability. It is an enumeration result that requires further testing.

---

# 3. HTTP Content Enumeration

The `http-enum` NSE script was used to discover interesting web paths.

```bash
nmap --script http-enum -p80 192.168.127.132
```

### Discovered Paths

```text
/tikiwiki/
/test/
/phpinfo.php
/phpMyAdmin/
/doc/
/icons/
/index/
```

These paths were investigated as part of the enumeration process.

---

# 4. phpinfo.php Investigation

One discovered path was:

```text
/phpinfo.php
```

Full URL:

```text
http://192.168.127.132/phpinfo.php
```

## 4.1 HTTP Response

```bash
curl -I http://192.168.127.132/phpinfo.php
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html
```

### Finding

The `phpinfo.php` page is publicly accessible.

## 4.2 Information Disclosure

The page exposed information including:

* PHP version
* Linux system information
* Configuration paths
* PHP configuration
* PHP modules
* Database/library information
* Session configuration

### Finding

**Publicly accessible `phpinfo()` page**

### Impact

This represents an information-disclosure issue because detailed server and PHP configuration information is exposed.

It does not by itself prove system compromise.

---

# 5. OPTIONS Request

An `OPTIONS` request was sent to `phpinfo.php`.

```bash
curl -I -X OPTIONS http://192.168.127.132/phpinfo.php
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Length: 48014
Content-Type: text/html
```

### Observation

The endpoint returned `200 OK` to the `OPTIONS` request.

No `Allow:` header was returned, so the complete list of allowed methods could not be determined from this response.

---

# 6. Directory Listing

The `/doc/` path discovered through `http-enum` was investigated.

## 6.1 Check `/doc/`

```bash
curl -I http://192.168.127.132/doc/
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
Content-Type: text/html;charset=UTF-8
```

The directory was then retrieved:

```bash
curl http://192.168.127.132/doc/
```

The response displayed:

```text
Index of /doc/
```

### Finding

Directory listing is enabled on `/doc/`.

---

# 7. Extracting Links from Directory Listing

Instead of manually reading the complete HTML page, `href` values were extracted.

```bash
curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30
```

Example results:

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

This can provide additional information for software fingerprinting.

---

# 8. Understanding the Link Extraction Command

Command:

```bash
curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30
```

### `curl -s`

```bash
curl -s http://192.168.127.132/doc/
```

Retrieves the webpage.

`-s` means silent mode.

### Pipe `|`

```text
|
```

Passes the output of one command to the next command.

### `grep -Eo`

```bash
grep -Eo 'href="[^"]+"'
```

* `-E` → Extended Regular Expression
* `-o` → Print only the matching part

The pattern:

```text
href="[^"]+"
```

extracts links such as:

```text
href="apache2/"
```

### `head -30`

```bash
head -30
```

Displays only the first 30 results.

---

# 9. Relative URL Investigation

One extracted link was:

```text
href="apache2/"
```

Because the link appeared inside `/doc/`, its relative path resolves to:

```text
/doc/apache2/
```

I first tested:

```bash
curl -s http://192.168.127.132/apache2/ | head -30
```

Result:

```text
HTTP/1.1 404 Not Found
```

The server returned:

```text
The requested URL /apache2/ was not found on this server.
```

### Explanation

The issue was the path.

The extracted link:

```text
apache2/
```

was relative to:

```text
/doc/
```

Therefore, the correct resolved path is:

```text
/doc/apache2/
```

This demonstrated the importance of understanding relative URLs during web enumeration.

---

# 10. Findings Summary

## FTP

### Anonymous FTP

```text
Anonymous FTP login allowed
```

### FTP Information Disclosure

```text
vsFTPd 2.3.4
Plain-text FTP connections
```

## HTTP

### Server Information Disclosure

```text
Apache/2.2.8 (Ubuntu)
PHP/5.2.4-2ubuntu5.10
Apache-Coyote/1.1
```

### phpinfo Exposure

```text
/phpinfo.php
```

The endpoint exposes detailed PHP/server configuration information.

### Directory Listing

```text
/doc/
```

Directory indexing is enabled and package documentation directories are visible.

---

# 11. Commands Used

## FTP

```bash
nmap --script-help "ftp-*"

nmap --script ftp-anon -p 21 192.168.127.132

nmap --script ftp-syst -p21 192.168.127.132
```

## HTTP

```bash
nmap --script http-title -p 80,8180 192.168.127.132

nmap --script http-headers 192.168.127.132

nmap --script http-methods 192.168.127.132

nmap --script http-enum -p80 192.168.127.132

curl -I http://192.168.127.132/phpinfo.php

curl -I -X OPTIONS http://192.168.127.132/phpinfo.php

curl -I http://192.168.127.132/doc/

curl http://192.168.127.132/doc/

curl -s http://192.168.127.132/doc/ | grep -Eo 'href="[^"]+"' | head -30

curl -s http://192.168.127.132/apache2/ | head -30
```

---

# 12. Key Learning

Today's practice covered:

* FTP NSE enumeration
* Anonymous FTP detection
* FTP service fingerprinting
* HTTP service enumeration
* Nmap HTTP NSE scripts
* HTTP headers
* HTTP methods
* Web path discovery
* `phpinfo()` information disclosure
* Directory listing
* HTML link extraction
* `grep` regular expressions
* Linux pipes
* Relative URL handling
* HTTP `200 OK` and `404 Not Found`

---

## Lab Environment

All testing documented here was performed against the authorized **Metasploitable 2** lab environment.

**Target:** `192.168.127.132`
**Attacker:** Kali Linux
