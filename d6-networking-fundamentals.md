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

## Task: HTTP Status Codes

Same approach here — worth exploring by actually triggering some of these against real requests (a tool like `curl -I <url>` shows the status code a server returns) rather than only memorizing the categories:

- **1xx — Informational** — the request was received, processing continues
- **2xx — Success** — the request was received, understood, and accepted (`200 OK` being the one you'll see constantly)
- **3xx — Redirection** — further action is needed to complete the request, usually because the resource has moved (`301 Moved Permanently`, `302 Found`)
- **4xx — Client Error** — the request itself was faulty in some way (`404 Not Found`, `403 Forbidden`, `401 Unauthorized`)
- **5xx — Server Error** — the request was valid, but the server failed to fulfill it (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`)

The categorization pattern itself (which digit starts the code) is worth knowing cold, since it lets you immediately judge whether an error is "something's wrong with what I sent" (4xx) versus "something's wrong on the server's end" (5xx) — a distinction that comes up constantly once you're debugging real deployments later in this course.
