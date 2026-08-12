# Networking Fundamentals — OSI Model, Connection Testing, and DNS Lookups

## The OSI Model, Revisited

The OSI (Open Systems Interconnection) model is a conceptual framework describing how data actually travels from one machine to another, broken into seven distinct layers. Nothing on a network works by magic — every request, every page load, every SSH session is really this stack of layers working together, each one handing off to the next.

| Layer | Name | What it's responsible for | Rough example |
| --- | --- | --- | --- |
| 7 | Application | The actual protocols applications use to communicate | HTTP, DNS, SSH |
| 6 | Presentation | Data formatting, encryption, compression | TLS/SSL, character encoding |
| 5 | Session | Establishing, managing, and tearing down connections between two hosts | Session tokens, login sessions |
| 4 | Transport | Reliable (or unreliable) delivery of data between hosts | TCP, UDP |
| 3 | Network | Addressing and routing data across different networks | IP, routers |
| 2 | Data Link | Getting data across a single physical link reliably | Ethernet, MAC addresses, switches |
| 1 | Physical | The actual physical transmission of raw bits | Cables, radio signals, voltage |

The layer that matters most for the commands below is the **Transport layer** — specifically TCP, since testing whether a port is open and reachable is fundamentally a Transport-layer question, not an Application-layer one. You can successfully "connect" at the TCP level to a port that has no working application behind it at all, which is exactly the distinction `telnet` is useful for testing.

---

## Testing Raw Connectivity: `telnet`

`telnet` predates almost everything else on this list — it was originally built as a remote login protocol, but its actual login functionality has long since been replaced by SSH for anything security-conscious. What survives is its usefulness as a blunt, no-frills tool for testing whether a TCP connection to a specific host and port can be established at all.

```bash
telnet <ip-or-hostname> <port>
```

Running this attempts to open a raw TCP connection to that IP address on that specific port. What happens next tells you something concrete:

- **Connection succeeds** — something is actively listening on that port and accepting TCP connections. This doesn't necessarily mean the *application* behind it is working correctly, only that the port itself is open and reachable.
- **Connection refused** — the host is reachable, but nothing is listening on that specific port.
- **Connection times out / hangs** — likely a firewall silently dropping the traffic, or the host itself being unreachable.

This is genuinely useful in real troubleshooting: if a web server isn't loading in a browser, `telnet <server-ip> 80` tells you immediately whether the problem is at the TCP level (port not open, firewall blocking it) or further up the stack (the web application itself misbehaving despite the port being open).

**Getting out of a telnet session** isn't obvious the first time — `Ctrl+]` (Control and the right square bracket together) is the escape sequence that drops you into telnet's own command prompt, from which typing `close` (or `quit`) actually ends the session. A plain `Ctrl+C` doesn't reliably work the way it does for most other terminal programs.

---

## ICMP and `ping`

Where TCP and UDP (Transport layer) carry actual application data, **ICMP** (Internet Control Message Protocol) exists at the Network layer specifically for diagnostic and error-reporting messages between machines — it's not used to transfer application data at all, only to ask "are you there?" and report network-level problems.

`ping` is ICMP's most familiar use: it sends a small ICMP "echo request" packet to a target and waits for an "echo reply" back, measuring whether the host responds and how long the round trip takes.

```bash
ping google.com
```

A successful ping confirms basic network reachability — the target exists and is responding — but says nothing about whether any particular *application* on that machine (a web server, an API) is actually working. That's exactly the gap `telnet` (testing a specific port) fills in, which is why the two get used together: `ping` answers "is the machine there at all," `telnet <host> <port>` answers "is this specific service on it reachable."

Both `ping` and `telnet` may need installing first on a fresh server:

```bash
sudo apt install iputils-ping
sudo apt install curl
```

---

## `netcat` (`nc`)

`nc`, short for **netcat**, is often described as a more capable, general-purpose relative of `telnet`. Where `telnet` is really only built for testing TCP connections, `nc` can work with both **TCP and UDP**, making it a broader tool specifically for checking network connectivity across either protocol, not just TCP.

```bash
nc -zv <host> <port>
```

(`-z` scans without sending data, just checking if the port is open; `-v` adds verbose output showing the result clearly.) The underlying idea is the same as `telnet`'s connectivity test — confirming whether something is reachable on a given port — just with wider protocol support and more options built in.

---

## `curl` — Talking to Servers from the Command Line

`curl` is, in effect, a command-line web browser — it lets you send HTTP(S) requests and see exactly what comes back, without needing an actual browser window at all.

```bash
curl google.com
```

This sends a request to `google.com` and prints the raw HTML response directly to the terminal — the same content a browser would render visually, just shown as unrendered source instead.

```bash
curl -L google.com
```

`-L` tells `curl` to **follow redirects**. Many sites respond to a plain request with a redirect (a 3xx status code, covered further down) pointing somewhere else — to `https://` instead of `http://`, for instance — and without `-L`, `curl` would just show you that redirect response itself rather than following it through to the final page.

`curl` is genuinely one of the most-used tools in real DevOps troubleshooting, alongside `telnet` — it's the fastest way to check whether a web service is actually responding correctly, what status code it's returning, and what it's sending back, all without leaving the terminal.

---

## Seeing What's Listening: `netstat`

`netstat` reports network connections, listening ports, and related statistics for the local machine. The flags combine to control exactly what gets shown and how:

| Flag | Meaning |
| --- | --- |
| `-t` | Show TCP connections |
| `-u` | Show UDP connections |
| `-p` | Show the process (program name and PID) that owns each socket |
| `-l` | Show only sockets that are actively listening for incoming connections |
| `-a` | Show all sockets, listening and already-established connections alike |
| `-n` | Show numeric addresses and port numbers instead of resolving them to hostnames/service names — faster, and avoids DNS lookups slowing the command down |
| `-e` | Show extended information |

Putting flags together:

```bash
netstat -tuplen
```

TCP and UDP sockets, only the ones actively listening, showing the owning process, extended details, and numeric output. This is close to the classic "what's currently listening on this machine, and what program put it there" command.

```bash
netstat -plante
```

Same flags, different order (order doesn't change the result) — but note this includes `-a` rather than relying on `-l` alone, so it shows *all* sockets (both listening and already-connected), not just the listening ones, alongside the process, numeric addressing, TCP, and extended info.

**Worth knowing going forward:** `netstat` itself is considered a legacy tool at this point — it's been effectively deprecated in favor of a newer, faster command that reads directly from the kernel instead of parsing through `/proc`.

---

## The Modern Replacement: `ss`

```bash
ss -plante
```

Same flag meanings as `netstat` above, but `ss` ("socket statistics") is the actively maintained successor, part of the `iproute2` package. It's meaningfully faster on systems with a large number of open connections, since it talks to the kernel directly through netlink rather than reading and parsing text files the way `netstat` does under the hood. In practice, most of what you'd reach for `netstat` to do, `ss` does the same way, with the same flags, just faster and with ongoing maintenance behind it — worth defaulting to `ss` going forward rather than treating the two as interchangeable by habit.

### Making Sense of the Output

Running either command produces a **State** column (or similar) alongside each entry — `LISTEN` being the one that matters most day to day. A `LISTEN` state means the local machine has a process actively waiting for incoming connections on that port — this is literally how you answer "which ports is this machine listening on right now."

The address column itself is worth being able to read at a glance:

- **`0.0.0.0:8880`** — the `0.0.0.0` means "any IP address on this machine." A service listening here accepts connections arriving on *any* of the machine's network interfaces, not just one specific one.
- **`127.0.0.1:6444`** — `127.0.0.1` is the **loopback address**, meaning the machine talking to itself. A service listening only here is reachable exclusively from the same machine, and refuses connections coming from anywhere else on the network — a deliberate security choice for services that should never be exposed externally.

Both of the examples above are IPv4. The IPv6 equivalent looks noticeably different in format, wrapped in brackets with a colon-separated address:

```text
[::ffff:127.0.0.1]:8377
```

Same underlying concept — an address and a port — just IPv6's longer notation instead of IPv4's dotted format.

---

## Querying DNS Directly: `dig`

`dig` ("domain information groper") queries DNS servers directly and shows exactly what they return — genuinely useful for troubleshooting DNS issues that a browser or `ping` would only show you the symptoms of, not the cause.

```bash
dig -t A fb.com
```

The `-t` flag specifies which **record type** to query. `A` records map a domain name to an IPv4 address — this asks specifically "what IPv4 address does fb.com point to," and returns just that, rather than every piece of DNS information about the domain at once.

```bash
dig -t CNAME fb.com
```

Queries for a **CNAME** record instead — an alias that points one domain name to another domain name, rather than directly to an IP address. If `www.fb.com` were configured as a CNAME pointing to `fb.com`, this is the record type that relationship would live in.

The general pattern is always the same: `dig -t <record-type> <domain>` — swapping the record type changes exactly what piece of DNS information comes back, without having to sift through an entire unfiltered DNS response to find it.

### Reading `dig`'s Output: the ANSWER Section

A real `dig` response is broken into several sections, but the one that actually matters for most everyday use is the **ANSWER SECTION** — this is where the actual record you asked for shows up, e.g. confirming that `nabilbank.com` resolves to a specific IP address like `103.71.243.24`. The rest of the output (query time, server used, message size) is metadata about the lookup itself, useful for deeper debugging but not the answer you're usually after.

### Two More Record Types Worth Knowing

```bash
dig -t NS nabilbank.com
```

**NS** records specify which name servers are authoritative for a domain — in other words, which servers are actually responsible for answering DNS questions about it.

```bash
dig -t PTR nabilbank.com
```

**PTR** records work in reverse compared to everything else so far — instead of "what IP does this domain point to," a PTR lookup asks "what domain does this IP belong to." This is sometimes described as **two-way verification**: an A record confirms domain → IP, and a PTR record confirms the same relationship IP → domain, which is exactly why mismatched or missing PTR records are a common flag in spam/security filtering — legitimate mail servers are generally expected to have both directions line up.

### Querying Multiple Types at Once

```bash
dig -t A CNAME NS nabilbank.com
```

`dig` can be pointed at several record types in the same query, returning whatever combination of A, CNAME, and NS records exist for that domain in one pass, rather than running three separate lookups.

---

## DMZ — Demilitarized Zone

A **DMZ** is a network security concept, not a Linux command — but it came up alongside these tools and is worth capturing properly.

A DMZ is a separate, isolated network zone sitting between the public internet and an organization's internal/private network — a deliberately "safe" middle ground where servers that genuinely need to be reachable from the internet get placed.

The reasoning: a company's web server needs to be accessible from the outside internet by definition — that's the whole point of a public website. But the company's internal systems (employee computers, internal databases, internal tools) absolutely should not be directly reachable from the internet. Placing the web server in a DMZ means it's exposed to the internet as intended, while staying network-isolated from the internal systems behind it. If an attacker manages to compromise the web server itself, the DMZ's isolation is what stops that breach from spreading straight into the internal network and its databases — the attacker gets the public-facing box, not the crown jewels behind it.

---

## Task: DNS Records

Rather than a full reference dump here, this is worth working through hands-on with `dig` itself, since that's the actual point of the exercise — querying real record types against real domains and seeing what comes back, rather than just reading a table of definitions.

Worth researching and testing directly with `dig -t <type> <domain>` for each of these:

- **A** — maps a domain to an IPv4 address (already covered above)
- **AAAA** — the IPv6 equivalent of an A record
- **CNAME** — an alias pointing one domain to another (already covered above)
- **MX** — specifies which mail servers handle email for a domain
- **TXT** — arbitrary text data attached to a domain, commonly used for domain verification and email security (SPF, DKIM records)
- **NS** — specifies which name servers are authoritative for a domain
- **SOA** — the "start of authority" record, containing administrative info about a DNS zone
- **PTR** — the reverse of an A record; maps an IP address back to a domain name

Running `dig -t <type> <a-real-domain>` for each of these against a few different real domains (not just `fb.com`) is the actual assignment — the table above is a starting map, not a substitute for seeing the real output.

## HTTP Status Codes and CRUD Operations

### CRUD and HTTP Methods

Before the status codes themselves, it's worth connecting them to *what kind of request* produces them. **CRUD** — Create, Read, Update, Delete — describes the four basic operations any system that manages data needs to support, and HTTP methods map onto them directly:

| CRUD Operation | HTTP Method |
| --- | --- |
| Create | `POST` |
| Read | `GET` |
| Update | `PUT` / `PATCH` |
| Delete | `DELETE` |

A **POST API**, for instance, is specifically an endpoint whose job is creating a new resource — submitting a new user, a new order, a new record. `PATCH` and `PUT` both handle updates, though with a difference worth knowing: `PUT` conventionally replaces a resource entirely, while `PATCH` applies a partial update to just the fields being changed.

### HTTP Status Codes

HTTP (HyperText Transfer Protocol) is what a browser (or `curl`) and a web server actually communicate over. Every response the server sends back includes a **3-digit status code** — a compact way of telling the client exactly what happened to its request, before any actual content even matters.

**The five categories:**

| Code Range | Meaning | Example |
| --- | --- | --- |
| 1xx | Informational | 100 Continue |
| 2xx | Success | 200 OK |
| 3xx | Redirection | 301 Moved Permanently |
| 4xx | Client Error | 404 Not Found |
| 5xx | Server Error | 500 Internal Server Error |

The categorization pattern itself (which digit starts the code) is worth knowing cold, since it lets you immediately judge whether an error is "something's wrong with what I sent" (4xx) versus "something's wrong on the server's end" (5xx) — a distinction that comes up constantly once you're debugging real deployments later in this course.

**2xx — Success:**

- `200 OK` — request successful
- `201 Created` — a new resource was created (the typical response to a successful `POST`)
- `204 No Content` — successful, but there's no content to return

**3xx — Redirection:**

- `301 Moved Permanently` — the resource has permanently moved to a new location
- `302 Found` — a temporary redirect
- `304 Not Modified` — the client's cached version is still valid and can be used as-is

**4xx — Client Error:**

- `400 Bad Request` — the request itself was malformed or invalid
- `401 Unauthorized` — authentication is required, or the provided credentials failed
- `403 Forbidden` — the server understood the request but refuses to fulfill it
- `404 Not Found` — the requested resource doesn't exist
- `405 Method Not Allowed` — the HTTP method used isn't permitted on this resource

**5xx — Server Error:**

- `500 Internal Server Error` — a general, unspecified server-side failure
- `502 Bad Gateway` — a gateway or proxy received an invalid response from an upstream server
- `503 Service Unavailable` — the server is temporarily unable to handle the request
- `504 Gateway Timeout` — a gateway or proxy didn't receive a response from the upstream server in time

A fast way to see these in action directly from the terminal, tying back to `curl` from earlier:

```bash
curl -I <url>
```

`-I` fetches only the response headers (not the full page body), with the status code right at the top — genuinely the quickest way to check what a server is actually returning without opening a browser at all.
