# HTTP Response Analysis

## Problem Statement

During web reconnaissance, identifying accessible resources is not enough. We also need to understand how the web server responds to different paths.

This exercise analyzes HTTP response **status codes, content types, and response sizes** for several resources on an authorized Metasploitable 2 laboratory machine.

---

## Objective

* Understand HTTP status codes
* Compare existing and nonexistent resources
* Identify publicly accessible web resources
* Learn basic response analysis using `curl`

---

## Target

```text
Target: 192.168.127.132
Environment: Metasploitable 2 Lab
Service: HTTP
Port: 80
```

> Testing was performed against an intentionally vulnerable local laboratory machine.

---

# Command Used

Because the Kali `PATH` environment variable was temporarily misconfigured, the full path to `curl` was used.

```bash
for path in / /phpinfo.php /doc/ /phpMyAdmin/ /doesnotexist/; do
    echo "===== $path ====="
    /usr/bin/curl -s -o /dev/null -w "HTTP Status: %{http_code}\nContent-Type: %{content_type}\nSize: %{size_download}\n" "http://192.168.127.132$path"
done
```

### How the Command Works

The `for` loop tests multiple web paths automatically.

For each path:

1. Prints the path being tested.
2. Sends an HTTP request using `curl`.
3. Discards the response body.
4. Displays the HTTP status code.
5. Displays the response `Content-Type`.
6. Displays the downloaded response size.

---

## Important `curl` Options

| Option             | Purpose                                |
| ------------------ | -------------------------------------- |
| `-s`               | Silent mode                            |
| `-o /dev/null`     | Discards the response body             |
| `-w`               | Displays selected response information |
| `%{http_code}`     | HTTP status code                       |
| `%{content_type}`  | Response Content-Type                  |
| `%{size_download}` | Downloaded response size               |

---

# Results

```text
===== / =====
HTTP Status: 200
Content-Type: text/html
Size: 891

===== /phpinfo.php =====
HTTP Status: 200
Content-Type: text/html
Size: 48002

===== /doc/ =====
HTTP Status: 200
Content-Type: text/html;charset=UTF-8
Size: 114160

===== /phpMyAdmin/ =====
HTTP Status: 200
Content-Type: text/html; charset=utf-8
Size: 4145

===== /doesnotexist/ =====
HTTP Status: 404
Content-Type: text/html; charset=iso-8859-1
Size: 297
```

---

# Response Analysis

## 1. `/`

```text
HTTP Status: 200
Content-Type: text/html
Size: 891
```

The main web page exists and the server successfully returned an HTML response.

---

## 2. `/phpinfo.php`

```text
HTTP Status: 200
Content-Type: text/html
Size: 48002
```

The PHP information page is publicly accessible.

This is significant because `phpinfo()` can expose detailed PHP and server configuration information.

---

## 3. `/doc/`

```text
HTTP Status: 200
Content-Type: text/html;charset=UTF-8
Size: 114160
```

The documentation directory is accessible and returns a large HTML directory listing.

---

## 4. `/phpMyAdmin/`

```text
HTTP Status: 200
Content-Type: text/html; charset=utf-8
Size: 4145
```

The phpMyAdmin web interface is accessible from the target.

A `200` response only confirms that the resource is accessible; it does **not** by itself prove that the application is vulnerable.

---

## 5. `/doesnotexist/`

```text
HTTP Status: 404
Content-Type: text/html; charset=iso-8859-1
Size: 297
```

The server correctly returned `404 Not Found` for a resource that does not exist.

This provides a useful baseline for comparing discovered resources against nonexistent paths.

---

# HTTP Status Codes

The most important status codes observed in this exercise were:

| Status | Meaning   | Observation                                  |
| ------ | --------- | -------------------------------------------- |
| `200`  | OK        | Requested resource was successfully returned |
| `404`  | Not Found | Requested resource was not found             |

### Important Distinction

```text
200 OK ≠ Vulnerable
404 Not Found ≠ Secure
```

A `200` response tells us that the requested resource was successfully returned. Further enumeration and validation are required before identifying a security vulnerability.

---

# Key Learning

HTTP response analysis helps reconnaissance by showing:

1. Whether a resource exists
2. What type of content the server returns
3. The approximate response size
4. How the server handles invalid paths

Response size can also help distinguish different resources during automated enumeration.

---

# Skills Practiced

* HTTP reconnaissance
* `curl`
* HTTP status-code analysis
* Content-Type identification
* Response-size analysis
* Web resource enumeration
* Basic security observation
* Bash `for` loops
* Basic command-line automation

---

# Ethical Note

All testing in this exercise was performed against the authorized **Metasploitable 2 laboratory VM**.

**Target:** `192.168.127.132`
**Purpose:** Security learning and authorized laboratory practice.
