# Nginx, SSL/TLS, and DNS Resolution

## Installing and Running Nginx

```bash
sudo apt install nginx
```

Once installed, Nginx runs as a **systemd service**, managed the same way any other background service is:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

These two commands do genuinely different things, and it's worth keeping them straight:

- **`start`** turns the service on *right now*, for this session.
- **`enable`** configures the service to start automatically on every future boot.

They're independent of each other. A service can be started without being enabled (it'll run now, but not survive a reboot), or enabled without being started (it'll be dormant now, but come alive automatically the next time the machine restarts). In practice you almost always want both — `start` for immediate use, `enable` so the web server doesn't silently vanish after a server reboot.

Confirming it's actually working:

```bash
curl localhost:80
```

This sends a request to the local machine's own port 80 and should return Nginx's default welcome page HTML — a quick sanity check that the service is genuinely listening and responding, without needing a browser.

---

## Nginx Configuration

```bash
cd /etc/nginx
vim nginx.conf
```

(Running `vim .` opens Vim's built-in file browser on the current directory, from which a specific file like `nginx.conf` can be selected and opened — a quick way to browse before committing to a specific file.)

### The Shape of `nginx.conf`

Nginx's configuration is organized into nested blocks, each one scoping settings to a particular level:

- **Main context** (top of the file, outside any block) — global settings that apply to the whole server process, including things like which user Nginx runs as.
- **`events { }`** — settings governing how Nginx handles connections at a low level (worker connection limits, and similar).
- **`http { }`** — HTTP-wide settings: logging formats, MIME types, and, nested inside it, one or more `server { }` blocks.
- **`server { }`** — represents a single virtual host: which port it listens on, which domain name(s) it responds to, and where its content lives.
- **`location { }`** — nested inside a `server` block, matching specific URL paths and controlling how requests to those paths are handled.

### Setting Custom HTML Content

Nginx's default setup serves whatever's in `/var/www/html/` as the site's content. Serving custom content means either of two approaches:

- Replace the files inside `/var/www/html/` directly with your own `index.html` and assets, leaving Nginx's default configuration untouched, or
- Point Nginx at a different directory entirely, by changing the `root` directive inside the relevant `server` block to a different path, and setting `index` to whichever file should be served by default when a directory is requested (`index index.html;`).

The second approach is generally the better habit — it keeps custom content clearly separated from Nginx's own default files, rather than overwriting them in place.

### Logging Directives

```text
error_log /var/log/nginx/error.log;
access_log /var/log/nginx/access.log;
```

- **`error_log`** — where Nginx records its own errors and warnings: configuration problems, upstream failures, worker process issues. This is the first place to look when something is actually broken.
- **`access_log`** — a record of every single request the server processes: client IP, timestamp, the exact request made, the status code returned, bytes sent, and (depending on the configured log format) the referrer and user agent too. This is less about diagnosing breakage and more about visibility — traffic patterns, which resources are actually being hit, and by whom.

### The `user` Directive

```text
user www-data;
```

This specifies which system user Nginx's **worker processes** actually run as — deliberately not root. `www-data` is the conventional, low-privilege account used for web server processes on Debian-based systems specifically so that if the web server itself is ever compromised, the attacker inherits only `www-data`'s limited permissions rather than full root access to the machine. (Nginx's master process typically starts as root to bind to privileged ports like 80/443, but immediately drops privilege for the actual worker processes that handle requests.)

---

## Nginx vs Apache

Both are mature, widely used web servers, but they're built around genuinely different architectures:

| | Nginx | Apache |
| --- | --- | --- |
| **Connection model** | Event-driven, asynchronous — a small number of worker processes, each handling many connections concurrently through a non-blocking event loop | Traditionally process- or thread-per-connection — each connection typically gets its own dedicated worker |
| **Resource usage under load** | Lower memory overhead as concurrent connections scale up, since workers aren't tied 1:1 to connections | Memory usage climbs more steeply under high concurrency, since more processes/threads are spun up |
| **Configuration style** | Centralized configuration files; no per-directory override mechanism | Supports `.htaccess` files for per-directory configuration overrides, which Nginx has no direct equivalent for |
| **Typical strengths** | Serving static content at high concurrency, acting as a reverse proxy or load balancer in front of application servers | Flexible per-directory configuration, a long-standing and extensive module ecosystem, tight `mod_php`-style integration for dynamic content |

Neither is strictly "better" — Nginx tends to win in high-traffic, high-concurrency scenarios and as the front door of a modern infrastructure stack (reverse proxying to application servers), while Apache remains common where per-directory `.htaccess` flexibility genuinely matters, such as certain shared hosting environments.

---

## SSL/TLS Configuration

```text
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
```

This directive lists which versions of TLS (Transport Layer Security — the actual modern protocol behind "SSL," even though the name "SSL" has stuck around informally) the server is willing to negotiate with a connecting client.

| Version | Status |
| --- | --- |
| SSLv2 / SSLv3 | Fully broken, never listed or enabled anywhere — vulnerable to well-known attacks (e.g. POODLE for SSLv3) with no fix possible |
| TLSv1.0 (1999) | Deprecated — officially dropped by major browsers and PCI-DSS compliance standards around 2020; vulnerable to attacks like BEAST |
| TLSv1.1 (2006) | Also deprecated alongside TLSv1.0, for similar reasons — considered obsolete by the same 2020 industry-wide cutoff |
| TLSv1.2 (2008) | Still considered secure and is the current practical baseline — widely supported, strong when configured with modern cipher suites |
| TLSv1.3 (2018) | The current standard — faster handshake, and the protocol itself removes support for older, weaker ciphers entirely rather than just discouraging them |

The line above technically still allows the two deprecated versions (`TLSv1` and `TLSv1.1`) to be negotiated if a client requests them — which is a real security tradeoff, not just a formality. A security-conscious configuration today would typically drop those two entirely and list only `TLSv1.2 TLSv1.3`, unless there's a specific, known legacy client that genuinely still requires the older versions.

```text
ssl_prefer_server_ciphers on;
```

Normally, during a TLS handshake, the client offers a list of ciphers it supports, in its own preferred order. This directive tells Nginx to override that — instead of just picking the client's top choice, the **server's own configured cipher order** takes priority, and the server picks the strongest mutually supported option from its own list. This matters because a client's preference order isn't necessarily trustworthy or well-configured (it could be an old, misconfigured, or even malicious client offering weak ciphers first) — letting the server dictate the choice keeps the connection's actual security in the hands of the side that's presumably better configured. (Note: this directive is largely a non-issue under TLS 1.3 specifically, since that version changed how cipher negotiation works — it mainly matters for TLS 1.2 connections.)

**On the ciphers themselves:** a TLS connection actually combines several different cryptographic algorithms working together, not just one:

- **Key exchange** (e.g. `ECDHE`) — how the two sides agree on a shared secret without ever transmitting it directly. `ECDHE` specifically provides **forward secrecy**, meaning even if a server's private key were compromised in the future, past captured traffic still couldn't be decrypted retroactively.
- **Bulk encryption** (e.g. `AES-256-GCM`, `ChaCha20-Poly1305`) — the actual algorithm encrypting the data itself once the connection is established.
- **Authentication/integrity** (e.g. `SHA-256`) — ensuring data hasn't been tampered with in transit.

A "cipher suite" name like `ECDHE-RSA-AES256-GCM-SHA384` is really naming all of these pieces at once — key exchange method, authentication method, bulk cipher, and integrity check, strung together.

---

## `sites-available` vs `sites-enabled`

Nginx (following a convention originally from Apache) typically splits virtual host configuration into two directories:

- **`/etc/nginx/sites-available/`** — holds the actual configuration file for every site that's been defined, whether or not it's currently active. Think of this as the full catalog.
- **`/etc/nginx/sites-enabled/`** — holds **symlinks** pointing to specific files inside `sites-available` that should actually be loaded. Nginx's main configuration includes everything found in `sites-enabled`, not `sites-available` directly.

Enabling a site means creating a symlink:

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

Disabling it just means removing that symlink — the actual configuration file stays untouched in `sites-available`, ready to be re-enabled later without having to rewrite it.

```bash
cd sites-enabled
ls -la
```

Running this shows the symlinks themselves, with an arrow notation like `mysite -> /etc/nginx/sites-available/mysite`, confirming exactly which sites are actually live versus merely defined.

This split exists specifically so configurations can be kept around, tested, or temporarily disabled without deleting any actual work — flipping a site on or off becomes a one-line symlink operation rather than an edit.

---

## SSL Chaining

A server's SSL/TLS certificate (the "leaf" or "end-entity" certificate) is virtually never signed directly by a root Certificate Authority. Instead, it's signed by an **intermediate CA**, which is itself signed by the root. Browsers and operating systems only come pre-loaded with trust for a relatively small set of root CAs — they don't inherently know about or trust the intermediate on their own.

**SSL chaining** (or the "certificate chain") is the practice of bundling the leaf certificate together with the intermediate certificate(s) that connect it back to a trusted root, so that a connecting client receives the *entire* path — leaf → intermediate → root — and can verify trust all the way up, rather than getting handed an isolated certificate it has no way to independently verify.

In Nginx, this means the `ssl_certificate` directive should point to a **fullchain** file — the leaf certificate concatenated together with the necessary intermediate certificate(s) — rather than just the leaf certificate on its own. Serving only the leaf certificate without its intermediates is a genuinely common misconfiguration: it can appear to work fine in some browsers (which may cache intermediates from other sites and patch the gap silently) while failing outright in others, or in tools like `curl`, that don't have that same cached fallback — producing confusing, inconsistent SSL trust errors that seem to work "sometimes."

---

## What a DNS Record Actually Is

A DNS record is a single entry in a domain's DNS configuration, mapping that domain (or part of it) to some piece of information — most commonly an IP address, but not always. Every time a domain needs to communicate something about itself to the rest of the internet — where it lives, which mail servers handle its email, which other services it delegates to — that information lives as a specific type of DNS record, queryable by anyone.

## Types of DNS Records

| Record Type | What it maps |
| --- | --- |
| **A** | A domain name to an IPv4 address |
| **AAAA** | A domain name to an IPv6 address |
| **CNAME** | One domain name to another domain name (an alias, rather than directly to an IP) |
| **MX** | A domain to the mail servers responsible for handling its email |
| **TXT** | A domain to arbitrary text data — commonly used for domain ownership verification and email security standards like SPF and DKIM |
| **NS** | A domain to the name servers that are authoritative for it |
| **SOA** | Administrative metadata about a DNS zone — the primary name server, an administrative contact, and timing values controlling how the zone gets refreshed and cached |
| **PTR** | An IP address back to a domain name — the reverse of an A record |
| **SRV** | A specific service (and port) to the host that provides it, used by protocols that need to advertise more than just an IP — e.g. locating a specific service within a domain |
| **CAA** | A domain to which Certificate Authorities are actually permitted to issue SSL certificates for it — a security control against unauthorized certificate issuance |

## DNS Resolution — The Actual Process

Looking up a domain name isn't a single request to a single server — it's a chain of lookups, most of which are invisible unless something goes wrong. Roughly, in order:

1. **Browser cache** — the browser checks whether it's already resolved this domain recently and cached the result.
2. **OS-level cache / `/etc/hosts`** — if the browser doesn't have it cached, the operating system checks its own resolver cache, and any manual overrides in `/etc/hosts`.
3. **Recursive resolver** — if still unresolved, the query goes out to a configured recursive DNS resolver (commonly the ISP's own resolver, or a public one like `8.8.8.8`). This resolver checks its own cache first.
4. If the resolver doesn't have it cached either, it starts the actual recursive lookup chain:
   - It asks a **root name server**: "who's responsible for `.com`?" (or whichever top-level domain the query falls under).
   - The root server doesn't know the final answer, but it knows who does — it replies with a referral to the appropriate **TLD (top-level domain) name server**.
   - The resolver then asks that TLD server: "who's authoritative for `example.com` specifically?"
   - The TLD server replies with a referral to the domain's actual **authoritative name server**.
   - The resolver finally asks that authoritative server directly, and gets back the real record — the actual IP address, or whatever record type was requested.
5. The resolver caches this result (for however long the record's TTL specifies) and returns the answer back up the chain to the OS, then the browser.
6. Only now does the browser actually open a connection to the resolved IP address.

The entire multi-step chain typically happens in a fraction of a second, and is skipped almost entirely most of the time thanks to caching at every layer — the full root-to-authoritative walk only happens when nothing along the chain already has the answer cached.

## Root Name Servers

Root name servers sit at the very top of the DNS hierarchy — the starting point every resolution eventually falls back to if nothing further down is cached. There are **13 logical root server addresses**, labeled `a` through `m`, operated collectively by 12 different organizations. In practice, each of those 13 logical addresses is served by many physical machines distributed globally, using a routing technique called **anycast** — the same IP address is announced from multiple physical locations, and a query automatically reaches whichever instance is geographically or topologically closest, rather than one single physical server worldwide.

Root servers themselves don't know the final answer to any query — their entire job is knowing which TLD server to point a resolver toward next. That single, narrow responsibility, replicated globally, is what keeps the entire DNS system functioning even under enormous global query volume.

---

## Inspecting HTTP Traffic with `curl -v`

```bash
curl -vvL google.com
```

`-v` (verbose) makes `curl` print out exactly what's happening during the request, not just the final response body. Worth knowing: `curl` doesn't actually have distinct stacked verbosity levels the way some other tools do (`ssh -vv`, for instance, genuinely gets more detailed with each extra `v`) — `curl`'s verbosity is effectively just on or off with `-v`; writing `-vv` doesn't unlock any additional detail beyond what a single `-v` already shows, though it's a common habit carried over from tools where it does matter.

Reading verbose output, the prefix on each line tells you what kind of information it is:

- **`*`** — connection-level details: DNS resolution, which IP it's connecting to, the TLS handshake (including which cipher was negotiated and certificate details).
- **`>`** — the actual request headers `curl` sent out.
- **`<`** — the actual response headers received back from the server.
- Everything after that is the response body itself.

```bash
curl -vvL nabilbank.com | head -n 50
```

Piping into `head -n 50` caps the output at the first 50 lines. Verbose output combined with a full HTML response body can run extremely long — this keeps it to a readable chunk in the terminal rather than scrolling past everything.

---

## Reading a Browser's Network Tab

Opening a browser's developer tools and switching to the **Network** tab shows every request the page makes, in real time. A few things worth knowing how to read:

**The request list itself** typically shows, per request: the resource name, its status code, its type (document, script, stylesheet, image, XHR/fetch, etc.), what triggered it, its size, and how long it took.

**Clicking into a single request** breaks it down further, usually across several sections:

- **Headers** — the actual request and response headers exchanged (covered below), plus general info like the full request URL, method, and status code.
- **Preview** — a rendered, readable preview of the response (formatted JSON, a rendered image, etc.).
- **Response** — the raw, unformatted response body.
- **Timing** — a breakdown of exactly where time was spent on this specific request: DNS lookup, initial connection, TLS negotiation, waiting for the server's first byte, and downloading the content.

### User Agents

A **user agent** is a string sent by the client as part of every request, identifying what's actually making the request — browser name and version, rendering engine, operating system. A typical one looks something like:

```text
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
```

(The string looks messy and historically inconsistent because it's accumulated decades of backward-compatibility tokens from the browser wars — modern browsers still claim to be "Mozilla" and "like Gecko" for legacy compatibility reasons, not because that's literally what they are.) Servers can use this to serve different content to different browsers/devices, or simply to log what's actually hitting them.

### Headers

Headers are key-value metadata attached to every HTTP request and response, separate from the actual body content.

- **Request headers** describe the client and what it's asking for — `Host` (which domain it's targeting), `User-Agent`, `Accept` (what content types the client can handle), `Authorization` (credentials/tokens), `Cookie`.
- **Response headers** describe what's being sent back — `Content-Type`, `Content-Length`, `Set-Cookie`, `Cache-Control`, `Server`.

### Status Codes

Covered in full detail in the networking notes on HTTP status codes — the short version worth repeating here in the context of the Network tab specifically: every single request listed carries its own status code, and scanning down the list for anything outside the 2xx/3xx range is usually the fastest way to spot what's actually broken on a page, before digging into any individual request's details.
