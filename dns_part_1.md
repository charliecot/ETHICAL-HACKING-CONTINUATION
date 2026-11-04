# DNS HIERARCHY — A COMPLETE BEGINNER-TO-ADVANCED GUIDE

**Author:** Nzegge Charles Ngong

---

# 1. Introduction to DNS

DNS stands for **Domain Name System**.

DNS is a distributed naming system that translates domain names into information computers can use to communicate, especially IP addresses.

For example:

```text
www.example.com
```

A computer needs an IP address to communicate with the server hosting that domain.

DNS helps discover that address.

## Why does DNS have a hierarchy?

Imagine that every DNS server in the world stored information about every domain name.

That would create problems:

- Maintaining all domain records in one place would be difficult.
- DNS queries would create enormous workloads.
- Updating domain information would become complicated.
- There would be no practical way to distribute responsibility.

DNS solves this problem using a hierarchical and distributed structure.

Instead of one central server containing every domain's records, different DNS servers manage different parts of the namespace.

The main hierarchy is:

```text
                  ROOT (.)
                     |
          +----------+----------+
          |          |          |
         .com       .org       .net
          |          |          |
       example     wikipedia   example
          |
      example.com
          |
    www.example.com
```

The DNS hierarchy is read from right to left in a domain name.

For example:

```text
www.example.com.
```

- `.` is the DNS root.
- `com` is the Top-Level Domain.
- `example` is the Second-Level Domain.
- `www` is a subdomain.

The final dot represents the root of the DNS namespace. It is often omitted when writing domain names.

---

# 2. The Four Main Components of DNS Resolution

Before studying the hierarchy, understand these four components.

| Component | Responsibility |
|---|---|
| Stub resolver | The DNS client on your computer that sends queries to a resolver |
| Recursive DNS resolver | Finds the answer on behalf of the client |
| Root DNS server | Directs the resolver towards the appropriate TLD |
| TLD DNS server | Directs the resolver towards the authoritative DNS server |
| Authoritative DNS server | Provides the official DNS information for its zone |

Important distinction:

The recursive resolver is not the same thing as the root, TLD, or authoritative server.

The recursive resolver performs the lookup on behalf of your computer. The other servers provide information about the DNS namespace.

---

# 3. Understanding the DNS Root (.)

The DNS root is at the very top of the DNS hierarchy.

It is represented by:

```text
.
```

The root is responsible for directing DNS resolvers towards the appropriate Top-Level Domain (TLD) servers.

Examples of TLDs include:

```text
.com
.org
.net
.edu
.gov
.cm
.uk
```

The root does not normally contain the final IP addresses for websites.

Instead, it provides referrals to the relevant TLD name servers.

## Root DNS servers

There are 13 named root server identities.

They are identified using letters A through M:

```text
a.root-servers.net
b.root-servers.net
c.root-servers.net
d.root-servers.net
e.root-servers.net
f.root-servers.net
g.root-servers.net
h.root-servers.net
i.root-servers.net
j.root-servers.net
k.root-servers.net
l.root-servers.net
m.root-servers.net
```

Important:

There are 13 named identities, not merely 13 physical machines. Each identity is served by multiple instances distributed across the world using anycast.

This makes the root DNS infrastructure more resilient and geographically distributed.

## What does the root server do?

Suppose you want to resolve:

```text
www.example.com
```

Your recursive resolver does not have the answer cached.

It asks a root server:

```text
Where can I find information about www.example.com?
```

The root server identifies `.com` as the relevant TLD and provides a referral to `.com` name servers.

The root server does not need to know the final IP address.

It only needs to direct the resolver to the next level.

---

# 4. Understanding TLD DNS Servers

TLD means **Top-Level Domain**.

A TLD is the highest-level domain immediately below the DNS root.

Examples:

| TLD | Category or common use |
|---|---|
| `.com` | Generic top-level domain |
| `.org` | Generic top-level domain |
| `.net` | Generic top-level domain |
| `.edu` | Education-related |
| `.gov` | Government-related |
| `.cm` | Cameroon country-code TLD |
| `.uk` | United Kingdom country-code TLD |
| `.de` | Germany country-code TLD |
| `.io` | Country-code TLD also widely used in technology |

There are different categories of TLDs:

### A. Generic Top-Level Domains (gTLDs)

Examples:

```text
.com
.org
.net
.info
```

### B. Country-Code Top-Level Domains (ccTLDs)

These are associated with countries or territories.

Examples:

```text
.cm    Cameroon
.uk    United Kingdom
.us    United States
.de    Germany
.ng    Nigeria
```

### C. Sponsored and other specialized TLDs

Examples include:

```text
.edu
.gov
.mil
```

Their registration rules and administration differ.

## What does a TLD server do?

Consider:

```text
www.example.com
```

The root server has already directed the resolver to the `.com` TLD servers.

The resolver asks a `.com` server:

```text
Where can I find the authoritative DNS information for example.com?
```

The TLD server returns a referral to the authoritative name servers for `example.com`.

For example, an illustrative response might identify:

```text
ns1.example-dns.com
ns2.example-dns.com
```

These names are examples, not actual DNS servers for every domain.

The TLD server generally does not provide the final IP address of `www.example.com`.

It tells the resolver which authoritative servers are responsible for the domain.

---

# 5. Understanding Authoritative DNS Servers

An authoritative DNS server stores or serves the official DNS records for a DNS zone.

For example, the authoritative server for `example.com` may contain records such as:

```text
example.com.       A       192.0.2.10
www.example.com.   A       192.0.2.10
mail.example.com.  A       192.0.2.20
example.com.       MX      10 mail.example.com.
```

These are illustrative addresses and records.

## What is a DNS zone?

A DNS zone is an administratively managed portion of the DNS namespace.

For example:

```text
example.com
```

may be managed as one DNS zone.

Its authoritative DNS servers maintain records for that zone.

A domain and a DNS zone are related, but they are not always identical.

For example:

```text
example.com
```

can delegate a subdomain:

```text
sales.example.com
```

The `sales.example.com` subdomain can be managed by separate authoritative name servers.

That means the parent zone delegates responsibility for that subdomain to another DNS zone.

## Primary and secondary authoritative servers

A DNS zone may be served by multiple authoritative servers.

Two common roles are:

**Primary (master) server:**

- Holds the primary editable copy of the zone data.
- Zone changes are made or published through the primary DNS management process.

**Secondary (slave) server:**

- Obtains zone data from another authoritative server.
- Provides authoritative answers.
- Improves availability and redundancy.

Secondary servers commonly obtain updates using DNS zone transfers.

The two major transfer mechanisms are:

```text
AXFR    Full zone transfer
IXFR    Incremental zone transfer
```

Zone transfers commonly use TCP port 53.

---

# 6. Complete DNS Resolution Process

Now connect everything.

Imagine opening your browser and entering:

```text
https://www.example.com
```

Your computer needs to resolve the hostname before connecting to the web server.

Assume the answer is not already cached.

## Step 1: Your computer checks its DNS information

Your operating system's DNS resolver checks its available cache.

Other layers, such as the browser, may also maintain their own DNS cache.

If a valid cached answer exists, the lookup may finish without contacting the DNS hierarchy.

If no suitable answer exists, your computer sends a DNS query to its configured recursive resolver.

## Step 2: The recursive resolver contacts a root server

The resolver asks a root server about:

```text
www.example.com
```

The root server identifies `.com` as the relevant TLD.

It returns a referral containing information about `.com` name servers.

## Step 3: The resolver contacts a .com TLD server

The resolver asks the `.com` server:

```text
Where are the authoritative name servers for example.com?
```

The TLD server returns a referral identifying the authoritative name servers.

## Step 4: The resolver contacts the authoritative DNS server

The resolver asks the authoritative server for the address record of:

```text
www.example.com
```

The authoritative server responds with the appropriate DNS record, if one exists.

For example:

```text
www.example.com.    A    192.0.2.10
```

## Step 5: The recursive resolver returns the answer

The recursive resolver returns the DNS answer to your computer.

It may also cache the result according to its TTL.

## Step 6: Your computer connects to the destination

Your browser can now use the resolved IP address to connect to the web server.

For HTTPS, the browser also performs the necessary TLS handshake.

The DNS lookup itself does not download the website.

---

# 7. Visualizing the Full DNS Hierarchy

```text
                    YOUR COMPUTER
                          |
                          |
                  DNS QUERY REQUEST
                          |
                          v
                RECURSIVE DNS RESOLVER
                          |
                          |
                 Query Root DNS Server
                          |
                          v
                    ROOT SERVER
                          |
                Referral to .com servers
                          |
                          v
                   .COM TLD SERVER
                          |
          Referral to example.com name servers
                          |
                          v
              AUTHORITATIVE DNS SERVER
                          |
                 A record response
                          |
                          v
                192.0.2.10 (Example)
                          |
                          v
               RECURSIVE DNS RESOLVER
                          |
                          v
                    YOUR COMPUTER
                          |
                          v
                 WEB SERVER CONNECTION
```

Remember:

The arrows between the recursive resolver and the DNS hierarchy represent DNS queries and responses.

Your computer usually communicates with the recursive resolver, not directly with every server in the hierarchy.

---

# 8. Recursive Query vs Iterative Query

These two terms are very important.

## A. Recursive query

A recursive query means the client asks a DNS resolver to obtain the final answer or return an error.

Example:

```text
Computer → Recursive Resolver
```

Your computer asks:

```text
What is the IP address of www.example.com?
```

It expects the resolver to do the necessary work.

The resolver either returns the answer or an error.

## B. Iterative query

An iterative query allows a DNS server to respond with the best information it has, including a referral to another DNS server.

For example:

```text
Resolver → Root Server
Resolver → TLD Server
Resolver → Authoritative Server
```

The resolver follows referrals until it finds the answer.

## Comparison

| Recursive query | Iterative query |
|---|---|
| Client expects a final answer | Server may return a referral |
| Resolver does the lookup work | Requester follows the referral |
| Commonly used between computer and recursive resolver | Commonly used between recursive resolver and DNS hierarchy servers |

Important: A recursive resolver may perform iterative queries to root, TLD, and authoritative servers while satisfying your computer's recursive request.

---

# 9. Understanding DNS Caching and TTL

TTL means **Time To Live**.

DNS records have TTL values that determine how long a DNS answer may be cached.

Example:

```text
www.example.com.  3600  IN  A  192.0.2.10
```

Here:

```text
3600 seconds = 1 hour
```

The resolver can generally cache this answer for up to one hour.

Why is caching important?

Without caching, DNS resolvers would repeatedly contact the DNS hierarchy for the same information.

Caching helps:

- Reduce DNS traffic.
- Improve lookup speed.
- Reduce workload on authoritative servers.
- Improve efficiency.

Important:

TTL is not a guarantee that the domain will remain available for that duration. It is a caching instruction for the DNS record.

---

# 10. Understanding DNS Records

Different DNS records provide different types of information.

| Record | Function |
|---|---|
| A | Maps a hostname to IPv4 |
| AAAA | Maps a hostname to IPv6 |
| CNAME | Defines an alias |
| MX | Identifies mail servers |
| NS | Identifies authoritative name servers |
| TXT | Stores text information |
| PTR | Supports reverse DNS |
| SOA | Provides DNS zone authority information |
| SRV | Identifies services and their connection details |
| CAA | Specifies permitted certificate authorities |

Example:

```text
example.com.      A       192.0.2.10
example.com.      MX      10 mail.example.com.
example.com.      NS      ns1.example.com.
www.example.com.  CNAME   example.com.
```

A DNS server does not only resolve website IP addresses. It also provides information for email, service discovery, domain verification, and other network services.

---

# 11. DNS Port Numbers and Transport Protocols

DNS commonly uses port 53.

| Protocol | Port | Usage |
|---|---:|---|
| UDP | 53 | Most ordinary DNS queries and responses |
| TCP | 53 | Zone transfers and DNS responses that require TCP |

Other encrypted DNS protocols include:

| Technology | Common transport and port |
|---|---|
| DNS over TLS (DoT) | TCP 853 |
| DNS over HTTPS (DoH) | HTTPS, commonly TCP 443 |
| DNS over QUIC (DoQ) | UDP 853 |

Traditional DNS is not encrypted by default.

DNSSEC authenticates DNS data but does not encrypt the DNS query.

---

# 12. Practical DNS Commands on Windows

You can practise DNS hierarchy concepts using PowerShell or CMD.

## A. nslookup

Resolve a domain:

```powershell
nslookup example.com
```

Query an A record:

```powershell
nslookup -type=A example.com
```

Query an AAAA record:

```powershell
nslookup -type=AAAA example.com
```

Query MX records:

```powershell
nslookup -type=MX example.com
```

Query NS records:

```powershell
nslookup -type=NS example.com
```

Query TXT records:

```powershell
nslookup -type=TXT example.com
```

## B. Specify the DNS resolver

Use Google's DNS resolver:

```powershell
nslookup example.com 8.8.8.8
```

Use Cloudflare's DNS resolver:

```powershell
nslookup example.com 1.1.1.1
```

This lets you compare responses from different recursive resolvers.

## C. Resolve-DnsName

PowerShell also provides:

```powershell
Resolve-DnsName example.com
```

Specific record:

```powershell
Resolve-DnsName example.com -Type A
```

Name servers:

```powershell
Resolve-DnsName example.com -Type NS
```

Mail servers:

```powershell
Resolve-DnsName example.com -Type MX
```

## D. Display your DNS cache

```powershell
ipconfig /displaydns
```

Clear your Windows DNS cache:

```powershell
ipconfig /flushdns
```

---

# 13. Understanding DNS with dig

`dig` is a DNS lookup utility commonly used on Linux and other Unix-like systems.

Basic lookup:

```bash
dig example.com
```

Request A record:

```bash
dig example.com A
```

Request MX record:

```bash
dig example.com MX
```

Request NS record:

```bash
dig example.com NS
```

Request TXT record:

```bash
dig example.com TXT
```

## Trace the DNS hierarchy

One particularly useful command is:

```bash
dig +trace example.com
```

This attempts to show the DNS resolution path from the root towards the authoritative servers.

You can observe:

1. Root server referrals.
2. TLD server referrals.
3. Authoritative server responses.

This is one of the best practical commands for understanding the DNS hierarchy.

---

# 14. DNS and Cybersecurity

DNS is an important part of network security.

Understanding its hierarchy helps you understand several security issues.

## A. DNS spoofing

An attacker attempts to provide false DNS information.

For example, a victim requests a legitimate domain but is given an attacker-controlled IP address.

This can redirect the victim to a malicious destination.

## B. DNS cache poisoning

An attacker attempts to insert forged DNS information into a resolver's cache.

If successful, other clients using that resolver may receive the forged answer.

## C. DNS tunnelling

DNS traffic can be abused to transfer information through DNS queries and responses.

It can be used for command-and-control communication or data exfiltration.

## D. DNS enumeration

DNS enumeration involves discovering publicly available DNS records and infrastructure information.

Tools include:

```text
nslookup
dig
host
whois
```

## E. DNSSEC

DNSSEC stands for Domain Name System Security Extensions.

It uses digital signatures to help validate the authenticity and integrity of DNS data.

DNSSEC does not encrypt DNS traffic.

---

# 15. Important Terms to Memorize

| Term | Meaning |
|---|---|
| DNS | Domain Name System |
| Root | Top of the DNS hierarchy |
| TLD | Top-Level Domain |
| gTLD | Generic Top-Level Domain |
| ccTLD | Country-Code Top-Level Domain |
| Resolver | DNS component that performs name resolution |
| Recursive resolver | Obtains the answer for a client |
| Stub resolver | DNS client on a device |
| Authoritative server | Provides official DNS data for a zone |
| DNS zone | Administratively managed DNS namespace |
| Referral | Information directing a resolver to another DNS server |
| TTL | Time To Live |
| FQDN | Fully Qualified Domain Name |
| AXFR | Full DNS zone transfer |
| IXFR | Incremental DNS zone transfer |
| DNSSEC | DNS Security Extensions |

---

# 16. Exercises

Try answering these questions without looking at the explanations.

1. What does DNS stand for?
2. Why do computers need DNS?
3. What is the purpose of the DNS root?
4. How many named root server identities exist?
5. What is a TLD?
6. What is the difference between `.com` and `example.com`?
7. What is the responsibility of an authoritative DNS server?
8. What is the difference between a recursive resolver and an authoritative server?
9. What is the difference between recursive and iterative queries?
10. What does TTL mean?
11. Why does DNS use caching?
12. Which port does DNS commonly use?
13. What is the difference between UDP and TCP in DNS?
14. What is DNSSEC?
15. What does `dig +trace example.com` do?

## Practical exercises

Run these commands and examine their output:

```powershell
nslookup google.com
nslookup -type=A google.com
nslookup -type=AAAA google.com
nslookup -type=MX google.com
nslookup -type=NS google.com
nslookup google.com 8.8.8.8
ipconfig /displaydns
```

If you have `dig` installed:

```bash
dig google.com
dig google.com NS
dig google.com MX
dig +trace google.com
```

---

# 17. Final Summary

The DNS hierarchy is a distributed system that divides responsibility for domain names among different levels.

Remember this order:

```text
ROOT
  |
  v
TLD
  |
  v
AUTHORITATIVE DNS SERVER
  |
  v
DNS RECORD / ANSWER
```

And remember the role of the recursive resolver:

```text
YOUR COMPUTER
      |
      v
RECURSIVE DNS RESOLVER
      |
      +----> ROOT
      |
      +----> TLD
      |
      +----> AUTHORITATIVE SERVER
      |
      v
FINAL DNS ANSWER
      |
      v
YOUR COMPUTER
```

The most important concept is that **DNS is hierarchical and distributed**.

No single DNS server needs to maintain every domain's information. Each level helps the resolver discover the next level until the appropriate authoritative information is found.

Once you understand this hierarchy, learning DNS enumeration, DNS troubleshooting, DNS packet analysis, and DNS security becomes much easier.
