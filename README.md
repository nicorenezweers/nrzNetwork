<div align="center">

<img src="static/icon.png" width="96" alt="nrzNetwork">

# nrzNetwork

**A self-hosted DNS server inspired by Technitium DNS Server, written in Rust.**

[Deutsch](docs/README.de.md) · [English](README.md)

</div>

nrzNetwork answers the DNS queries of your whole network from one small program. It is authoritative for your own zones (`home.arpa`, `example.com`, reverse zones), sends everything else to the upstream servers you choose — classic UDP/TCP or encrypted over TLS and HTTPS — caches the answers, and blocks ads, trackers and malware with blocklists and your own rules. Every query lands in a searchable query log, and the dashboard shows what your devices are asking. Settings, zones, lists and the log live in one SQLite file. The web interface is a Rust/WebAssembly app (Dioxus) in a calm gray theme and also works on phones.

## Features

| | |
|---|---|
| **Authoritative zones** | Primary zones with SOA and NS, records of type **A, AAAA, CNAME, MX, TXT, NS, SRV, PTR and CAA**, **wildcards** (`*`, `*.dev`), disable single records or whole zones, comments. Every change raises the SOA serial. **Import and export** zone files in BIND format (RFC 1035). |
| **Conditional forwarding** | Forwarder zones send a domain to its own servers, e.g. `fritz.box` or `corp.example` to the router or a company DNS. Your own records in such a zone still win. |
| **Forwarding** | Upstream servers over **UDP** (with TCP fallback for truncated answers), **TCP**, **DNS over TLS** and **DNS over HTTPS**. Strategies: **sequential**, **random** or **parallel** (fastest answer wins). Presets for Cloudflare, Quad9, Google, AdGuard DNS and Mullvad. Health and response time per upstream. |
| **Cache** | Configurable size, minimum, maximum and negative TTL, **serve-stale** (RFC 8767) when no upstream answers, **prefetch** of popular entries shortly before they expire. Browse, search and flush the cache, all or by name. |
| **Blocking** | Blocklists in **hosts**, **plain domain** and **Adblock** (`\|\|ads.example.com^`) format, **presets** (StevenBlack, HaGeZi, OISD, AdGuard DNS filter), **automatic updates** every N hours, **pause** for a few minutes. Blocked names get `0.0.0.0`/`::`, `NXDOMAIN`, `REFUSED` or an address of your choice. "Is this domain blocked, and why?" check. |
| **Own rules** | **Blocked domains** and **allowed domains** of your own; each rule covers the domain and all its subdomains, and an allowed domain beats every block. |
| **Query log** | Every query with client, protocol, name, type, response code, source of the answer (zone, cache, upstream, blocked, local, dropped, failed), answer, duration and upstream. **Filters** for client, name, type, response code, source and time, **CSV export**, retention in days. |
| **Dashboard** | Totals, a **chart** over the last hour, day, week or month, **top clients, top domains, top blocked domains** and query types, server status with uptime, queries per second, cache size and upstream health. |
| **DNS client** | Ask this server (with zones, blocking and cache) or any other server over UDP, TCP, DoT or DoH, for any record type — with flags, answer, authority and additional sections. |
| **MCP server** | **Model Context Protocol** at `/mcp` for [nrzDashboard](#mcp-server-nrzdashboard), Claude and other MCP clients, with any number of named tokens (each read-only or with changes): all figures for an overview in one call, statistics, query log, DNS lookups, reverse proxy, Docker, network map, certificates and the whole REST API — and, if the token may, turn filters on/off, pause them, switch blocklists, add rules, flush the cache. |
| **Reverse proxy** | Names instead of IP and port: `music.nrz.ch` → `http://100.78.107.162:8350`, `bench.nrz.ch` → `http://100.78.107.162:8550`. nrzNetwork answers the name itself with the address of the proxy, and the built-in HTTP proxy forwards by `Host` header — streaming, WebSockets, redirects rewritten. You don't need to own the domain; it works on every device that uses nrzNetwork. HTTP only. |
| **DNS records (Cloudflare)** | All zones of your Cloudflare account and **all DNS records** in one place: A, AAAA, CNAME, TXT, MX, NS, SRV, CAA — create, edit, delete, search, toggle the Cloudflare proxy (orange cloud), TTL and comments. |
| **NetBird** | Your whole NetBird network through the NetBird API (cloud or self-hosted): **devices** (rename, groups, SSH, login expiry, remove), **groups**, **access rules** (who may reach whom, protocols, ports), **networks** (resources and routing peers), **DNS** (nameserver groups, plus one click: *nrzNetwork as DNS for all NetBird devices*), **setup keys** (with ready-to-copy `netbird up` command) and **users** (invite, role, block). The Devices page shows NetBird names automatically. |
| **Requests (monitor)** | Who opens your websites behind the reverse proxy: requests, visitors, data, errors, timeline, per site, top pages, sources (Cloudflare with real client IP and country, NetBird, LAN, internet), browsers and bots, status codes, visitors with device/NetBird names and a live feed. Retention configurable. |
| **Cloudflare Tunnel & NetBird built in** | nrzNetwork downloads `cloudflared` and the NetBird client itself (official GitHub releases, SHA-256 verified), supervises and restarts them and installs updates daily at a chosen hour. Reverse proxy entries can be made public with one switch: nrzNetwork sets the tunnel ingress rule and the proxied CNAME (foreign rules stay). NetBird runs rootless in netstack mode; a setup key can be created through the NetBird API. NetBird's own reverse proxy ("Services") is manageable on the NetBird page. |
| **Dieter (AI assistant)** | An AI assistant for the whole network via **OpenRouter**, **Grok (xAI)** or the **Claude CLI** (downloaded and verified by nrzNetwork). He can use the entire nrzNetwork API (zones, Cloudflare DNS records, reverse proxy, blocking, NetBird, tunnel, components, Spy, requests) and DNS lookups — read only or, if allowed, also change things (accounts, credentials and his own settings excluded; every change is logged). Built-in monitoring checks DNS forwarding, SERVFAIL rate, tunnel/NetBird, proxy targets, 5xx rates, the certificate and Spy; on a new problem Dieter steps in automatically and reports what he found and fixed. He lives as a **chat window in the bottom-right corner of every page** (knows which page you are on, suggestions, Markdown answers, collapsible steps, badge for open problems); setup under Settings → Dieter. |
| **Network map** | One canvas with every connection as a line: the **internet** (public domains), entry points (**Cloudflare Tunnel**, **NetBird** reverse proxy, **router** with open ports from Spy, local DNS), nrzNetwork's **reverse proxy** entries, and the targets in the **Docker** network (containers with internal IPs, Umbrel app proxies → apps), **at home (LAN)** and in the **NetBird network** (peers online/offline). Hovering highlights a whole path (e.g. domain → tunnel → proxy → container), line width shows requests in the last 24 h, dashed lines = path currently down; click for details. On the dashboard and as its own page, with zoom and full screen. Cloudflare is read directly as well: every DNS record (proxied through Cloudflare or direct), **all tunnels** your records point to — also ones nrzNetwork doesn't manage — with status, connector IP, data centres and ingress targets (needs the token permission *Account → Cloudflare Tunnel → Read*), and `cloudflared` containers on the device matched to their tunnel. |
| **Streamer mode** | For streaming or screen sharing: press `b` or use the profile menu. IP addresses, domains, e-mails, UUIDs, MAC addresses and keys are blacked out everywhere, names, input fields, technical (monospace) details, logs and the network map are blurred, tooltips are hidden. Active before the app loads, remembered in the browser, a red “Streamer” badge shows it is on. |
| **Docker** | Read-only view of the whole Docker host like `docker ps`: every container with name, app, state, **internal IPs per network**, ports and image, all Docker networks with subnet/gateway and members, and the last log lines per container. Container names also show up in the request monitor, and Dieter can use it. Needs `/var/run/docker.sock` mounted read-only. |
| **Two-factor sign-in (2FA)** | TOTP with any authenticator app (QR code), 10 one-time recovery codes, codes can't be reused, failed codes count towards the login throttle; admins can reset 2FA for a user. |
| **Spy** | Only for your own domains (Cloudflare zones, HTTPS names): what attackers can learn from public sources — subdomains and emails in Certificate Transparency logs, registrar and contacts (RDAP/WHOIS), DNS records, name server and mail providers, SPF/DMARC/DKIM/CAA/DNSSEC, services revealed by TXT records, the hosting provider of every address, internal addresses in public DNS, records pointing at your home connection, dangling CNAMEs (subdomain takeover), Cloudflare errors — plus active checks: web servers (version leaks, HSTS, security headers, public admin panels, HTTP only) and **all 65,535 ports** of every directly reachable address and of your own connection. Every finding comes with step-by-step instructions and a link to the place in nrzNetwork to fix it. Scans run daily at a chosen hour; the history keeps 60 reports per domain with the changes since the previous scan. |
| **HTTPS for your own domains** | A real Let's Encrypt certificate (e.g. `zweers.ch` + `*.zweers.ch`) obtained through the DNS-01 challenge with a **Cloudflare API token** — nothing has to be reachable from outside, it works purely in the local network. Entries then open as `https://music.zweers.ch` without a warning; `http://` can redirect to `https://`. Renews automatically when a third of the lifetime is left. HTTPS for names without an own certificate (e.g. umbrelOS's own HTTPS) is passed through untouched by SNI. |
| **Block page** | Blocked domains can point at nrzNetwork itself (block mode *block page*, the default): `http://` sites show "This site was blocked by nrzNetwork" with buttons to **unlock it for a while** (15 min, 1 h, 4 h – configurable) and a form to **ask for permanent access**; the admin approves or rejects requests under *Allowed*. Unlocks end by themselves. |
| **Devices & network tools** | Every device that asks nrzNetwork, with queries, blocked share and last seen — give devices names. **Ping** (ICMP, TCP fallback), **port check**, **HTTP check** (status, time, redirects, headers) and a **subnet calculator**. |
| **DoH endpoint** | nrzNetwork itself answers **DNS over HTTPS** (RFC 8484) at `/dns-query`, `GET` and `POST`, and can be switched off. |
| **Protection** | **Rate limiting** per client (queries per second), private reverse lookups (`192.168.x.x`, `10.x.x.x`, `fc00::/7`, …) are not sent to the internet, optional empty answers for AAAA in networks without IPv6. |
| **Multi-user** | The **first account becomes the administrator**. **Admins** change everything; **members** see the dashboard, zones, lists, cache, log and DNS client but only change their own account. Registration can be opened or closed. |
| **Languages** | Deutsch and English |
| **One binary** | DNS engine, web server and web interface in one program (the interface is embedded at build time), one SQLite file, images and binaries for `linux/amd64` and `linux/arm64` |

## Quick start

You need Docker. The image contains the program and the web interface.

```bash
docker run -d --name nrzdns \
  -p 53:53/udp -p 53:53/tcp -p 8380:8380 \
  -v nrzdns-data:/data \
  ghcr.io/nicorenezweers/nrz-dns:1.14.0
```

The image is multi-arch (`linux/amd64`, `linux/arm64`) and is published for every [release](https://github.com/nicorenezweers/nrzNetwork/releases). To build it yourself instead (Docker with buildx and `cargo`, which only generates the Dockerfile – everything is compiled inside Docker):

```bash
git clone https://github.com/nicorenezweers/nrzNetwork.git
cd nrzNetwork
cargo xtask docker -t nrzdns
```

Open <http://localhost:8380>. The first account you create becomes the administrator. After that, registration is closed until you open it in the settings, where you can also create accounts for others.

> **Port 53 must be free.** Only one DNS server can listen on it. Stop Technitium, Pi-hole, AdGuard Home or dnsmasq first. On many Linux desktops `systemd-resolved` holds `127.0.0.53:53`; that is fine as long as nothing listens on `0.0.0.0:53`. If the port is taken, the program itself still starts and shows the error on the dashboard; in Docker (and on Umbrel) publishing the port already fails, so the container does not start.

### Docker Compose

```yaml
services:
  nrzdns:
    image: ghcr.io/nicorenezweers/nrz-dns:1.14.0   # or: build: .
    container_name: nrzdns
    restart: unless-stopped
    ports:
      - "53:53/udp"
      - "53:53/tcp"
      - "8380:8380"
    volumes:
      - nrzdns-data:/data          # nrzdns.db: everything in one file
    environment:
      TZ: Europe/Zurich
      # NRZDNS_COOKIE_SECURE: "true"   # when the web interface is served via HTTPS
    sysctls:
      net.ipv4.ip_unprivileged_port_start: 0   # user 1000 may bind port 53
    cap_drop: [ALL]
    security_opt: [no-new-privileges:true]

volumes:
  nrzdns-data:
```

```bash
docker compose up -d
```

The container runs as user `1000` without any capabilities. Docker (20.10 and later) lets unprivileged users bind low ports inside a container anyway; the `sysctls` line makes that explicit.

> **Back up `/data`.** It holds `nrzdns.db` with everything in it (plus `nrzdns.db-wal` and `nrzdns.db-shm` while the server runs). Copy the whole folder, ideally while the container is stopped.

### Without Docker

Every [release](https://github.com/nicorenezweers/nrzNetwork/releases) has the program for Linux (`amd64`, `arm64`) as a single file, with `SHA256SUMS`. It brings its own root certificates for DoT, DoH and blocklist downloads.

```bash
curl -LO https://github.com/nicorenezweers/nrzNetwork/releases/download/v1.14.0/nrzdns-1.14.0-linux-amd64
chmod +x nrzdns-1.14.0-linux-amd64
sudo setcap 'cap_net_bind_service=+ep' nrzdns-1.14.0-linux-amd64   # allow port 53 without root
NRZDNS_ADDRESS=0.0.0.0 NRZDNS_PORT=8380 NRZDNS_DATA_DIR=./data ./nrzdns-1.14.0-linux-amd64
```

To try it without touching port 53, let it listen elsewhere, e.g. `NRZDNS_DNS_LISTEN=127.0.0.1:5353`, and ask it with `dig @127.0.0.1 -p 5353 example.com`.

### Both architectures

```bash
cargo xtask docker --platform linux/amd64,linux/arm64 -t <registry>/nrz-dns:1.14.0 --push
```

Rust is cross-compiled, not emulated, so the arm64 build is fast on an x86 machine and vice versa. Only the program, gzipped per architecture (for the Umbrel app): `cargo xtask store-binaries <folder>`. Need the Dockerfile as a file (e.g. for `docker compose build`)? `cargo xtask dockerfile > Dockerfile`.

## First steps

1. **Create your account**: the first one becomes the administrator.
2. **Choose upstream servers** (*Settings → Forwarding*): keep the default (Cloudflare and Quad9 over UDP) or pick a preset, e.g. Quad9 over DoH, and a strategy.
3. **Turn on blocking** (*Blocking*): add one or two preset lists, e.g. *HaGeZi Multi Normal*. They load immediately and update every 24 hours.
4. **Add your own names** (*Zones*): create a zone such as `home.arpa` and add `nas`, `printer` and friends as A records — or import an existing BIND zone file. For names your router knows (e.g. `fritz.box`), add a forwarder zone that points at the router.
5. **Point your network at nrzNetwork** — see below.
6. **Watch it work**: the dashboard and the query log show every query; the DNS client answers "what does this name resolve to?".

## Using nrzNetwork in your network

- **Whole network**: in your router's DHCP settings, set the DNS server handed to clients to the IP address of the machine running nrzNetwork (e.g. `192.168.1.10`). Devices pick it up the next time they renew their lease (reconnect Wi-Fi or restart). Give that machine a fixed IP address first.
- **Single device**: set the DNS server in the device's network settings to that IP address.
- **Check**: `nslookup example.com 192.168.1.10` or `dig @192.168.1.10 example.com`; the query appears in the log.
- **DNS over HTTPS**: `https://<your-host>/dns-query` works for browsers and phones once the web interface is reachable via HTTPS (reverse proxy with a certificate); over plain HTTP it is useful for testing only.
- Keep a second DNS server out of the DHCP settings (or point it at another nrzNetwork), otherwise devices bypass blocking and your local names whenever they like.

## Configuration

Start-up settings come from environment variables with the prefix `NRZDNS_` (or `Rocket.toml`). Everything about DNS itself is set in the web interface and takes effect immediately, without a restart.

| Variable | Default | Meaning |
|---|---|---|
| `NRZDNS_DATA_DIR` | `./data` (`/data` in the image) | Folder for `nrzdns.db` |
| `NRZDNS_PORT` / `NRZDNS_ADDRESS` | `8380` / `0.0.0.0` | Where the web interface, the API and `/dns-query` listen |
| `NRZDNS_DNS_LISTEN` | `0.0.0.0:53` | DNS socket addresses (UDP and TCP each), comma-separated, e.g. `0.0.0.0:53,[::]:53` |
| `NRZDNS_HOST_GATEWAY` | – | Name under which the container reaches the device itself (`host.docker.internal`); targets on an address of this device are connected through it |
| `NRZDNS_PROXY_FALLBACK` | – | Where names without an entry, IP addresses and `.local` names go, e.g. `http://host.docker.internal:80` (umbrelOS) |
| `NRZDNS_PROXY_TLS_LISTEN` | `0.0.0.0:443` | HTTPS entry of the proxy (empty = none) |
| `NRZDNS_TLS_FALLBACK` | – | Where HTTPS for names without an own certificate goes unchanged, e.g. `host.docker.internal:443` (umbrelOS) |
| `NRZDNS_PROXY_LISTEN` | `0.0.0.0:80` | Addresses of the reverse proxy (HTTP), comma-separated; empty = off. On another port names need it: `http://music.nrz.ch:8480/` |
| `NRZDNS_CONFIG` | `Rocket.toml` (`/app/Rocket.toml` in the image) | Path of the configuration file |
| `NRZDNS_COOKIE_SECURE` | `false` | Set to `true` when the web interface is served via HTTPS |
| `NRZDNS_IP_HEADER` | `X-Real-IP` | Header in which a reverse proxy passes the client address (login throttling); `false` when nrzNetwork is reachable directly |
| `NRZDNS_MCP_TOKEN` | – | Additional fixed token for the MCP server (at least 16 characters); tokens are normally created under *Settings → MCP* |
| `NRZDNS_LOG` | `info` | Log filter, e.g. `debug` or `info,nrzdns_engine=debug` |

### DNS settings (web interface, admins)

| Area | Defaults |
|---|---|
| General | Server name `ns.nrzdns.local` (SOA/NS of new zones), default TTL 3600 s, DoH on, rate limit off, private reverse lookups answered locally, AAAA answers on |
| Forwarding | Cloudflare `1.1.1.1` and Quad9 `9.9.9.9` over UDP, sequential, 2000 ms timeout per upstream |
| Cache | On, 50 000 entries, TTL between 10 s and 1 day, negative answers 300 s, serve-stale up to 1 day, prefetch on |
| Blocking | On, block page (address of this device; `0.0.0.0` while none is known), TTL 60 s, lists updated every 24 hours |
| Query log | On, kept for 30 days |

## Reverse proxy

*Reverse Proxy* in the sidebar (admins): add a name and a target, e.g. `music.nrz.ch` → `http://100.78.107.162:8350` and `bench.nrz.ch` → `http://100.78.107.162:8550`. From then on:

1. nrzNetwork answers `music.nrz.ch` itself (A record, TTL 60 s) with the address of the device it runs on — the address under *DNS answer*, or, if that is empty, the IP from the target.
2. The browser connects to port 80 of that device; the built-in proxy picks the target by the `Host` header and forwards the request (keeps `Host`, adds `X-Forwarded-For`, `X-Real-IP`, `X-Forwarded-Host`, `X-Forwarded-Proto`; streams bodies; passes WebSockets; rewrites `Location: http://100.78.107.162:8350/…` to `http://music.nrz.ch/…`).

You don't have to own the domain — the names only exist for devices that ask nrzNetwork. That is also why it is HTTP only: no public certificate can be issued for a name you don't own. An existing zone such as `nrz.ch` keeps working; the proxy name wins because it is more specific. Unknown names get a short 404 page from the proxy, unreachable targets a 502 page.

## MCP server (nrzDashboard)

nrzNetwork speaks the [Model Context Protocol](https://modelcontextprotocol.io) at `POST /mcp` (Streamable HTTP, JSON-RPC 2.0, stateless, answers as `application/json`). That is how nrzDashboard gets the values for its overview — and any other MCP client works too.

1. *Settings → MCP* (admins): give the token a name (e.g. `Claude Code`), choose **May make changes** or not, **Create token** and copy it (it is shown only once, together with a ready `claude mcp add …` line and a JSON config). Create as many tokens as you like — one per client — rename them, change their rights or delete them one by one. `NRZDNS_MCP_TOKEN` works in addition.
2. In the client: URL `http://<host>:8380/mcp`, header `Authorization: Bearer <token>` (or `X-API-Key: <token>`; clients that cannot send headers use `http://<host>:8380/mcp?token=<token>`).

```bash
claude mcp add --transport http nrznetwork http://<host>:8380/mcp --header "Authorization: Bearer <token>"
```

CORS is open (access depends only on the token, never on cookies), so browser clients such as the MCP Inspector work too; `/.well-known/oauth-*` answers 404 so clients don't wait for OAuth.

| Tool | What it returns / does |
|---|---|
| `get_overview` `{range?}` | Everything for an overview tile in one call: server (version, uptime, queries/s, DNS ports), blocking (`enabled`, `paused`, `active`, blocked domains, lists), totals for the range (`queries`, `blocked`, `blockedPercent`, `cacheHitPercent`, `failed`, `clients`, `avgDurationMs` …), upstreams healthy/total, top 5 clients, domains and blocked domains |
| `get_statistics` `{range?, includeSeries?}` | The full dashboard: totals, time series, top 10 lists, query types, server state |
| `get_server_status`, `get_blocking_status` | Running state; blocking with every list |
| `get_query_log` `{client?, qname?, qtype?, kind?, rcode?, from?, to?, limit?, page?}` | Latest queries, newest first |
| `check_domain` `{domain}` | Would it be blocked, and why? |
| `list_zones`, `list_rules` `{list}` | Own zones; own blocked or allowed domains |
| `set_blocking` `{enabled}` ✎ | Turn the filters on or off (on also ends a pause) |
| `pause_blocking` `{minutes}` ✎ | Pause filtering, `0` resumes |
| `set_blocklist` `{id \| name, enabled}` ✎ | Switch a single blocklist |
| `update_blocklists` ✎ | Reload all active lists now |
| `add_rule` / `remove_rule` `{list, domain, comment?}` ✎ | Own blocked (`blocked`) or allowed (`allowed`) domains |
| `flush_cache` `{name?}` ✎ | Flush the cache, all or one name |
| `dns_lookup` `{name, type?, server?}` | DNS query like dig — empty `server` = nrzNetwork itself, otherwise e.g. `1.1.1.1` |
| `get_zone` `{zone}` | A zone (name or id) with all records |
| `get_cache` `{q?, limit?}` | Cache entries |
| `get_proxy`, `get_tls` | Reverse proxy hosts, ports, access protection; HTTPS certificate |
| `get_docker`, `get_docker_logs` `{container, tail?}` | Containers with internal IPs, networks, ports; container logs |
| `get_network_map` | The whole network map (internet, tunnels, proxy, Docker, LAN, NetBird) |
| `get_devices` `{days?}`, `get_access_log` `{host?, limit?}`, `get_access_stats` `{period?, host?}` | Clients; access monitor of the reverse proxy |
| `get_components` | Built-in Cloudflare Tunnel and NetBird client |
| `list_api_endpoints` | Every REST endpoint (for `api_request`) |
| `api_request` `{method?, path, query?, body?}` | Any REST call (path without `/api/v1`, e.g. `GET /cloudflare/zones`); read-only tokens may only `GET` |

✎ = only for tokens that may make changes; for the others these tools are not even listed. `range` is `hour`, `day` (default), `week` or `month`. Each result comes as JSON text and as `structuredContent`.

```bash
curl -s http://localhost:8380/mcp -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_overview","arguments":{"range":"day"}}}'
```

## Umbrel

nrzNetwork is available in the community app store **nrzStore** (app `nrzNetwork`, ID `nrz-dns`). The web interface opens on port `8400`; nrzNetwork has its own login, so the Umbrel login in front of it is switched off. DNS is published on port `53` (UDP and TCP) of the Umbrel device, so:

- **Port 53 has to be free** on the device — don't run Technitium, Pi-hole or AdGuard Home at the same time. Otherwise the installation fails with `app-script install … exit code 1`; stop or uninstall the other DNS server first.
- **Coming from Technitium:** export each zone in Technitium (*Zones → Export Zone*) and import it in nrzNetwork under *Zonen → Importieren*; add your blocklist URLs, blocked and allowed domains again. If your router already points at the Umbrel, nothing changes there — nrzNetwork takes over the same address.
- Point your router's DHCP DNS setting (or single devices) at the **IP address of your Umbrel**.
- **Names without a port:** port 80 belongs to umbrelOS. A small watchdog that the app starts on the device (`hooks/nrznetwork-gateway`) points port 80 of the LAN and VPN addresses at the proxy (port `81`) as long as the proxy answers; everything without an entry — including the umbrelOS interface — is passed on to umbrelOS unchanged. If the proxy stops answering, umbrelOS gets port 80 back within seconds. Switch it off under *Reverse proxy → Device and block page*.
- Targets on the Umbrel itself (e.g. `http://100.78.107.162:8400`) are reached through `host.docker.internal`; the addresses of the device (LAN, NetBird/Tailscale) are detected automatically, and every client gets the matching one.

Data lives in `app-data/nrz-dns/data`. The app ships the program for `amd64` and `arm64` inside the store and runs it in the official `debian:trixie-slim` image as user `1000`, without capabilities.

## Development

```bash
# backend (Rust 1.85+): server, DNS engine, shared types
NRZDNS_DNS_LISTEN=127.0.0.1:5353 cargo run   # web on http://localhost:8380, DNS on port 5353
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings

# web interface with hot reload (dx = dioxus-cli 0.7)
rustup target add wasm32-unknown-unknown
cargo install dioxus-cli --version 0.7.10 --locked
cargo xtask serve                 # http://localhost:8080, proxies /api and /static to :8380

# embed the interface into the program
cargo xtask ui && cargo build --release
```

The workspace has four crates: `nrzdns` (the Rocket server, `src/`), `nrzdns-engine` (the DNS server itself, `engine/`), `nrzdns-types` (every API type, `types/`, also used by the UI) and `xtask` (build tasks, `cargo xtask`). The interface in `ui/` is its own project for `wasm32-unknown-unknown`. `cargo xtask ui` builds it into `ui/dist/public`; `build.rs` embeds it (gzip) together with `locales/` and `static/`. Without it the server still runs and shows a notice instead of the interface. Translations live in `locales/<language>.json` (German source text → translation).

The code is 100 % Rust: the Tailwind input lives in `ui/tailwind.rs` (`cargo xtask` writes `ui/tailwind.css` from it before `dx` runs), the Dockerfile in `xtask/src/docker.rs` and the database migrations in `src/db/migrations.rs`; build tasks are `cargo xtask` instead of shell scripts.

The JSON API lives under `/api/v1`; the endpoint table is the doc comment of [`types/src/lib.rs`](types/src/lib.rs). Architecture details: [ARCHITECTURE.md](ARCHITECTURE.md).

## License

AGPL-3.0-or-later. nrzNetwork is an independent project inspired by [Technitium DNS Server](https://technitium.com/dns/) and shares no code with it. The UI components are based on [rust-ui](https://github.com/rust-ui/ui) (MIT), the font is [Inter](https://rsms.me/inter/) (OFL).
