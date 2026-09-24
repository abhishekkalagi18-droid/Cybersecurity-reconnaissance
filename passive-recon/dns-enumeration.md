# DNS Enumeration

## 1. What is DNS Enumeration?

DNS enumeration is the process of collecting DNS information about a domain.

The main goal is to understand how a domain's DNS infrastructure is configured and identify records such as:

* IPv4 addresses
* IPv6 addresses
* Mail servers
* Name servers
* Aliases
* Zone information
* TXT records

> Only perform DNS enumeration against domains you own or have explicit permission to investigate.

---

## 2. DNS Record Types

| Record | Purpose                               |
| ------ | ------------------------------------- |
| A      | Maps a domain to an IPv4 address      |
| AAAA   | Maps a domain to an IPv6 address      |
| MX     | Identifies mail servers               |
| NS     | Identifies authoritative name servers |
| CNAME  | Creates an alias for another hostname |
| TXT    | Stores text/configuration information |
| SOA    | Provides zone/authority information   |

---

## 3. Tools Used

During this exercise I used:

* `dig`
* `host`
* `nslookup`

---

# 4. Using `dig`

## A Record

Command:

```bash
dig goodshopping.com A
```

Result:

```text
goodshopping.com → 15.197.148.33
goodshopping.com → 3.33.130.190
```

This shows that the domain has two IPv4 addresses.

---

## AAAA Record

Command:

```bash
dig goodshopping.com AAAA
```

Result:

```text
ANSWER: 0
```

No AAAA record was returned in this query.

---

## MX Record

Command:

```bash
dig goodshopping.com MX
```

Result:

```text
10 mailstore1.secureserver.net.
0 smtp.secureserver.net.
```

The number represents the MX priority. A lower value has higher preference.

---

## NS Record

Command:

```bash
dig goodshopping.com NS
```

Result:

```text
ns13.domaincontrol.com.
ns14.domaincontrol.com.
```

These are the domain's name servers.

---

## TXT Record

Command:

```bash
dig goodshopping.com TXT
```

Result:

```text
ANSWER: 0
```

No TXT record was returned in this query.

---

## SOA Record

Command:

```bash
dig @1.1.1.1 goodshopping.com SOA
```

The SOA response showed:

```text
Primary NS: ns13.domaincontrol.com.
Responsible mailbox: dns.jomax.net.
Serial: 2024072902
Refresh: 28800
Retry: 7200
Expire: 604800
Minimum: 3600
```

The initial SOA query through my local DNS configuration timed out, so I tested the query using Cloudflare's DNS resolver.

---

## CNAME Record

Command:

```bash
dig www.goodshopping.com CNAME
```

Result:

```text
www.goodshopping.com. CNAME goodshopping.com.
```

This means `www.goodshopping.com` is an alias for `goodshopping.com`.

---

# 5. Following CNAME Resolution

Command:

```bash
dig www.goodshopping.com
```

The response showed:

```text
www.goodshopping.com. CNAME goodshopping.com.
goodshopping.com. A 3.33.130.190
goodshopping.com. A 15.197.148.33
```

The resolution chain can be represented as:

```text
www.goodshopping.com
        |
        | CNAME
        v
goodshopping.com
        |
        +---- A → 3.33.130.190
        |
        +---- A → 15.197.148.33
```

---

# 6. Using `host`

Command:

```bash
host goodshopping.com
```

Output showed:

```text
goodshopping.com has address 3.33.130.190
goodshopping.com has address 15.197.148.33
```

It also displayed the MX records.

For a specific record:

```bash
host -t MX goodshopping.com
```

This returned:

```text
goodshopping.com mail is handled by 0 smtp.secureserver.net.
goodshopping.com mail is handled by 10 mailstore1.secureserver.net.
```

`host` provides a simpler and more human-readable DNS lookup than `dig`.

---

# 7. Using `nslookup`

Command:

```bash
nslookup goodshopping.com
```

Returned:

```text
15.197.148.33
3.33.130.190
```

To query name servers:

```bash
nslookup -type=NS goodshopping.com
```

Returned:

```text
ns13.domaincontrol.com.
ns14.domaincontrol.com.
```

`nslookup` is useful for simple DNS queries and troubleshooting.

---

# 8. DNS Trace

Command:

```bash
dig +trace goodshopping.com
```

The trace demonstrated the DNS hierarchy:

```text
Root (.)
   ↓
.com TLD
   ↓
Authoritative DNS
   ↓
goodshopping.com
   ↓
A records
```

The trace ultimately returned:

```text
goodshopping.com. A 15.197.148.33
goodshopping.com. A 3.33.130.190
```

I also observed an IPv6 network connectivity error during the trace:

```text
network unreachable
```

The trace nevertheless successfully returned DNS information over IPv4.

---

# 9. Comparing DNS Tools

| Tool         | Main Use                                |
| ------------ | --------------------------------------- |
| `dig`        | Detailed DNS investigation              |
| `host`       | Quick DNS lookup                        |
| `nslookup`   | Simple DNS queries and troubleshooting  |
| `dig +trace` | Understanding DNS delegation/resolution |

---

# 10. DNS Enumeration Workflow

A basic workflow I practiced:

```text
Domain
   ↓
A / AAAA
   ↓
NS
   ↓
MX
   ↓
TXT
   ↓
SOA
   ↓
CNAME
   ↓
DNS Trace
   ↓
Correlate Results
```

---

# 11. What I Learned

Through this exercise I learned:

1. How to identify IPv4 addresses using A records.
2. How to check for IPv6 using AAAA records.
3. How to identify mail servers using MX records.
4. How to identify name servers using NS records.
5. How CNAME records create aliases.
6. How to inspect SOA information.
7. How to use `dig`, `host`, and `nslookup`.
8. How `dig +trace` demonstrates the DNS hierarchy.
9. How local DNS resolver problems can affect queries.
10. Why different DNS tools can provide different levels of detail.

---

# 12. Important Security Learning

DNS enumeration is primarily an information-gathering activity.

DNS information can help security professionals understand an organization's external infrastructure during an authorized assessment.

However:

* DNS records do not automatically indicate vulnerabilities.
* A discovered IP address does not mean a system should be attacked.
* A hostname does not prove that a service is active.
* DNS data should be correlated with other authorized reconnaissance results.

---

# 13. Troubleshooting

### Local DNS resolver problem

If a DNS query fails, check:

```bash
cat /etc/resolv.conf
```

Check routing:

```bash
ip route
```

A query can also be tested against a specific DNS resolver:

```bash
dig @1.1.1.1 example.com
```

This can help determine whether the problem is with the local resolver path or the DNS information itself.

---

# 14. Conclusion

This exercise provided practical experience with DNS enumeration using standard Linux DNS utilities.

The main lesson was that DNS enumeration is not just about finding an IP address. By examining different DNS record types and following the DNS hierarchy, it is possible to build a clearer understanding of how a domain's external DNS infrastructure is organized.
