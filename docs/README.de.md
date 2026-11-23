<div align="center">

<img src="../static/icon.png" width="96" alt="nrzNetwork">

# nrzNetwork

**Ein selbst gehosteter DNS-Server nach dem Vorbild von Technitium DNS Server, in Rust.**

[English](../README.md) · [Deutsch](README.de.md)

</div>

nrzNetwork beantwortet die DNS-Anfragen deines ganzen Netzes mit einem kleinen Programm. Für deine eigenen Zonen (`home.arpa`, `example.com`, Rückwärtszonen) antwortet es verbindlich, alles andere fragt es bei vorgelagerten Servern deiner Wahl nach – klassisch über UDP/TCP oder verschlüsselt über TLS und HTTPS –, speichert die Antworten im Cache und sperrt Werbung, Tracker und Schadsoftware mit Sperrlisten und eigenen Regeln. Jede Anfrage landet in einem durchsuchbaren Abfrageprotokoll, und das Dashboard zeigt, was deine Geräte fragen. Einstellungen, Zonen, Listen und Protokoll liegen in einer SQLite-Datei. Die Oberfläche ist eine Rust/WebAssembly-App (Dioxus) in ruhigem Grau und funktioniert auch am Handy.

## Funktionen

| | |
|---|---|
| **Eigene Zonen** | Primärzonen mit SOA und NS, Einträge vom Typ **A, AAAA, CNAME, MX, TXT, NS, SRV, PTR und CAA**, **Platzhalter** (`*`, `*.dev`), einzelne Einträge oder ganze Zonen abschalten, Kommentare. Änderungen erhöhen die SOA-Seriennummer. Zonendateien im BIND-Format (RFC 1035) **importieren und exportieren**. |
| **Bedingte Weiterleitung** | Weiterleitungszonen schicken eine Domain an eigene Server, z. B. `fritz.box` oder `firma.example` an den Router oder den Firmen-DNS. Eigene Einträge in so einer Zone haben trotzdem Vorrang. |
| **Weiterleitung** | Vorgelagerte Server über **UDP** (mit TCP-Rückfall bei abgeschnittenen Antworten), **TCP**, **DNS über TLS** und **DNS über HTTPS**. Strategien: **der Reihe nach**, **zufällig** oder **parallel** (die schnellste Antwort gewinnt). Vorlagen für Cloudflare, Quad9, Google, AdGuard DNS und Mullvad. Zustand und Antwortzeit je Server. |
| **Cache** | Grösse, minimale, maximale und negative TTL einstellbar, **abgelaufene Einträge ausliefern** (RFC 8767), wenn kein vorgelagerter Server antwortet, **Prefetch** häufig gefragter Einträge kurz vor Ablauf. Cache durchsehen, durchsuchen und leeren, ganz oder je Name. |
| **Sperren** | Sperrlisten im **hosts**-, **Domain**- und **Adblock**-Format (`\|\|ads.example.com^`), **Vorlagen** (StevenBlack, HaGeZi, OISD, AdGuard DNS Filter), **automatisches Neuladen** alle N Stunden, **Pause** für ein paar Minuten. Gesperrte Namen bekommen `0.0.0.0`/`::`, `NXDOMAIN`, `REFUSED` oder eine Adresse deiner Wahl. Prüfung „Wird diese Domain gesperrt, und warum?“. |
| **Eigene Regeln** | **Eigene Sperren** und **Ausnahmen**; jede Regel gilt für die Domain und alle Subdomains, eine Ausnahme schlägt jede Sperre. |
| **Abfrageprotokoll** | Jede Anfrage mit Client, Protokoll, Name, Typ, Antwortcode, Herkunft der Antwort (Zone, Cache, Weiterleitung, gesperrt, lokal, verworfen, fehlgeschlagen), Antwort, Dauer und vorgelagertem Server. **Filter** nach Client, Name, Typ, Antwortcode, Herkunft und Zeitraum, **CSV-Export**, Aufbewahrung in Tagen. |
| **Dashboard** | Kennzahlen, **Verlaufsdiagramm** über die letzte Stunde, den letzten Tag, die letzte Woche oder den letzten Monat, **Top-Clients, Top-Domains, meistgesperrte Domains** und Anfragetypen, Serverzustand mit Laufzeit, Anfragen je Sekunde, Cache-Grösse und Zustand der Weiterleitungen. |
| **DNS-Client** | Diesen Server (mit Zonen, Sperren und Cache) oder jeden anderen über UDP, TCP, DoT oder DoH fragen, für jeden Eintragstyp – mit Flags, Antwort-, Authority- und Additional-Abschnitt. |
| **MCP-Server** | **Model Context Protocol** unter `/mcp` für [nrzDashboard](#mcp-server-nrzdashboard), Claude und andere MCP-Clients, mit beliebig vielen benannten Tokens (jedes nur lesend oder mit Änderungen): alle Kennzahlen für eine Übersicht in einem Aufruf, Statistiken, Abfrageprotokoll, DNS-Abfragen, Reverse Proxy, Docker, Netzwerkkarte, Zertifikate und die ganze REST-API – und, wenn das Token darf, Filter ein-/ausschalten, pausieren, Sperrlisten umschalten, Regeln anlegen, Cache leeren. |
| **Reverse Proxy** | Namen statt IP und Port: `music.nrz.ch` → `http://100.78.107.162:8350`, `bench.nrz.ch` → `http://100.78.107.162:8550`. nrzNetwork beantwortet den Namen selbst mit der Adresse des Proxys, der eingebaute HTTP-Proxy reicht nach dem `Host`-Header weiter – mit Streaming, WebSockets und umgeschriebenen Weiterleitungen. Die Domain muss dir nicht gehören; es klappt auf allen Geräten, die nrzNetwork benutzen. Nur HTTP. |
| **DNS-Einträge (Cloudflare)** | Alle Zonen deines Cloudflare-Kontos und **alle DNS-Einträge** an einem Ort: A, AAAA, CNAME, TXT, MX, NS, SRV, CAA – anlegen, ändern, löschen, suchen, Cloudflare-Proxy (orange Wolke) ein/aus, TTL und Kommentare. |
| **NetBird** | Dein ganzes NetBird-Netzwerk über die NetBird-API (Cloud oder selbst gehostet): **Geräte** (umbenennen, Gruppen, SSH, Ablauf der Anmeldung, entfernen), **Gruppen**, **Zugriffsregeln** (wer wen erreicht, Protokolle, Ports), **Netzwerke** (Ressourcen und Router), **DNS** (Nameserver, dazu ein Klick: *nrzNetwork als DNS für alle NetBird-Geräte*), **Setup-Keys** (mit fertigem `netbird up`-Befehl) und **Benutzer** (einladen, Rolle, sperren). Die Geräte-Seite zeigt NetBird-Namen automatisch. |
| **Zugriffe (Monitor)** | Wer deine Websites hinter dem Reverse Proxy aufruft: Aufrufe, Besucher, Daten, Fehler, Verlauf, je Website, Top-Seiten, Herkunft (Cloudflare mit echter Adresse und Land, NetBird, LAN, Internet), Browser und Bots, Statuscodes, Besucher mit Geräte-/NetBird-Namen und Live-Ansicht. Aufbewahrung einstellbar. |
| **Cloudflare Tunnel & NetBird eingebaut** | nrzNetwork lädt `cloudflared` und den NetBird-Client selbst (offizielle GitHub-Releases, SHA-256 geprüft), betreut und startet sie neu und spielt täglich zur gewählten Stunde Updates ein. Einträge des Reverse Proxys werden mit einem Schalter öffentlich: nrzNetwork setzt Tunnel-Regel und CNAME (fremde Regeln bleiben). NetBird läuft rootless im Netstack-Modus; ein Setup-Key lässt sich über die NetBird-API erzeugen. NetBirds eigener Reverse Proxy („Freigaben“) ist auf der NetBird-Seite verwaltbar. |
| **Dieter (KI-Assistent)** | Ein KI-Assistent für das ganze Netz über **OpenRouter**, **Grok (xAI)** oder die **Claude CLI** (von nrzNetwork geladen und geprüft). Er nutzt die ganze API von nrzNetwork (Zonen, DNS-Einträge bei Cloudflare, Reverse Proxy, Sperren, NetBird, Tunnel, Komponenten, Spy, Zugriffe) und DNS-Abfragen – nur lesend oder, wenn erlaubt, auch ändernd (Konten, Zugangsdaten und seine eigenen Einstellungen ausgenommen; jede Änderung im Protokoll). Die Überwachung prüft DNS-Weiterleitung, SERVFAIL-Anteil, Tunnel/NetBird, Proxy-Ziele, Serverfehler, Zertifikat und Spy; bei einem neuen Problem springt Dieter automatisch ein und berichtet, was er gefunden und behoben hat. Er sitzt als **Chat-Fenster unten rechts auf jeder Seite** (kennt die aktuelle Seite, Vorschläge, Antworten mit Markdown, aufklappbare Schritte, Abzeichen für offene Probleme); eingerichtet wird er unter Einstellungen → Dieter. |
| **Netzwerkkarte** | Ein Canvas mit allen Verbindungen als Linien: das **Internet** (öffentliche Domains), die Zugänge (**Cloudflare Tunnel**, **NetBird**-Reverse-Proxy, **Router** mit offenen Ports aus dem Spy, lokales DNS), die **Reverse-Proxy**-Einträge von nrzNetwork und die Ziele im **Docker**-Netz (Container mit internen IPs, Umbrel-App-Proxys → Apps), **zu Hause (LAN)** und im **NetBird-Netz** (Geräte online/offline). Beim Darüberfahren leuchtet der ganze Weg auf (z. B. Domain → Tunnel → Proxy → Container), die Linienstärke zeigt die Zugriffe der letzten 24 h, gestrichelt = Weg gerade unterbrochen; Klick zeigt Details. Oben im Dashboard und als eigene Seite, mit Zoom und Vollbild. Cloudflare wird direkt gelesen: alle DNS-Einträge (über Cloudflare oder direkt), **alle Tunnel**, auf die Einträge zeigen – auch solche, die nrzNetwork nicht verwaltet – mit Status, Connector-IP, Rechenzentren und Ingress-Zielen (braucht beim Token das Recht *Konto → Cloudflare Tunnel → Lesen*), und `cloudflared`-Container auf dem Gerät mit ihrem Tunnel. |
| **Streamer-Modus** | Zum Streamen oder Bildschirm teilen: Taste `b` oder im Profilmenü. IP-Adressen, Domains, E-Mails, UUIDs, MAC-Adressen und Schlüssel werden überall geschwärzt, Namen, Eingabefelder, technische (Monospace-)Angaben, Logs und die Netzwerkkarte unscharf, Tooltips ausgeblendet. Schon vor dem Laden aktiv, im Browser gespeichert, ein rotes „Streamer“-Abzeichen zeigt ihn an. |
| **Docker** | Lesende Sicht auf den ganzen Docker-Host wie `docker ps`: jeder Container mit Name, App, Zustand, **internen IPs je Netz**, Ports und Image, alle Docker-Netze mit Subnetz/Gateway und Mitgliedern, und die letzten Log-Zeilen je Container. Container-Namen erscheinen auch im Zugriffs-Monitor, und Dieter kann alles nutzen. Braucht `/var/run/docker.sock` (nur lesend eingebunden). |
| **Zwei-Faktor-Anmeldung (2FA)** | TOTP mit jeder Authenticator-App (QR-Code), 10 einmalige Wiederherstellungscodes, Codes nicht wiederverwendbar, falsche Codes zählen zur Anmeldesperre; Admins können 2FA eines Benutzers zurücksetzen. |
| **Spy** | Nur für eigene Domains (Cloudflare-Zonen, HTTPS-Namen): was Angreifer aus öffentlichen Quellen erfahren – Subdomains und E-Mails in den Certificate-Transparency-Logs, Registrar und Kontakte (RDAP/WHOIS), DNS-Einträge, Nameserver- und Mail-Anbieter, SPF/DMARC/DKIM/CAA/DNSSEC, über TXT verratene Dienste, der Hoster jeder Adresse, interne Adressen im öffentlichen DNS, Einträge auf den eigenen Anschluss, verwaiste CNAMEs (Übernahme von Subdomains), Cloudflare-Fehler – dazu aktiv Webserver (verratene Versionen, HSTS, Sicherheits-Kopfzeilen, öffentliche Verwaltungsseiten, nur HTTP) und **alle 65 535 Ports** jeder direkt erreichbaren Adresse und des eigenen Anschlusses. Jede Erkenntnis mit Schritt-für-Schritt-Anleitung und Sprung zur passenden Stelle in nrzNetwork. Täglicher Scan zur gewählten Stunde; der Verlauf hält 60 Berichte je Domain mit den Änderungen zum vorigen Scan. |
| **HTTPS für eigene Domains** | Echtes Let's-Encrypt-Zertifikat (z. B. `zweers.ch` + `*.zweers.ch`) über die DNS-Prüfung mit einem **Cloudflare-API-Token** – von aussen muss nichts erreichbar sein, es klappt rein im lokalen Netz. Einträge öffnen sich dann als `https://music.zweers.ch` ohne Warnung, `http://` kann auf `https://` umleiten. Erneuert sich von selbst, wenn noch ein Drittel der Laufzeit übrig ist. HTTPS für Namen ohne eigenes Zertifikat (z. B. das eigene HTTPS von umbrelOS) wird anhand des Namens (SNI) unverändert durchgereicht. |
| **Sperrseite** | Gesperrte Domains können auf nrzNetwork selbst zeigen (Sperrmodus *Sperrseite*, die Vorgabe): `http://`-Seiten zeigen „Diese Seite wurde durch nrzNetwork gesperrt“ mit Knöpfen zum **Freischalten auf Zeit** (15 Min., 1 Std., 4 Std. – einstellbar) und einem Formular, um eine **dauerhafte Freigabe zu beantragen**; der Admin genehmigt oder lehnt unter *Ausnahmen* ab. Freischaltungen enden von selbst. |
| **Geräte & Netzwerk-Tools** | Jedes Gerät, das nrzNetwork fragt, mit Anfragen, Anteil gesperrt und zuletzt gesehen – Geräte bekommen Namen. **Ping** (ICMP, sonst TCP), **Port-Check**, **HTTP-Check** (Status, Zeit, Weiterleitungen, Header) und **Subnetz-Rechner**. |
| **DoH-Endpunkt** | nrzNetwork beantwortet selbst **DNS über HTTPS** (RFC 8484) unter `/dns-query`, `GET` und `POST`, abschaltbar. |
| **Schutz** | **Rate-Limit** je Client (Anfragen pro Sekunde), Rückwärtsanfragen für private Netze (`192.168.x.x`, `10.x.x.x`, `fc00::/7` …) gehen nicht ins Internet, auf Wunsch leere AAAA-Antworten für Netze ohne IPv6. |
| **Mehrere Konten** | Das **erste Konto wird Administrator**. **Admins** ändern alles; **Mitglieder** sehen Dashboard, Zonen, Listen, Cache, Protokoll und DNS-Client, ändern aber nur ihr eigenes Konto. Die Registrierung lässt sich öffnen und schliessen. |
| **Sprachen** | Deutsch und English |
| **Ein Programm** | DNS-Engine, Webserver und Oberfläche in einem Programm (die Oberfläche wird beim Bauen eingebettet), eine SQLite-Datei, Images und Programme für `linux/amd64` und `linux/arm64` |

## Schnellstart

Du brauchst Docker. Das Image enthält Programm und Oberfläche.

```bash
docker run -d --name nrzdns \
  -p 53:53/udp -p 53:53/tcp -p 8380:8380 \
  -v nrzdns-data:/data \
  ghcr.io/nicorenezweers/nrz-dns:1.15.0
```

Das Image ist für mehrere Architekturen gebaut (`linux/amd64`, `linux/arm64`) und erscheint mit jedem [Release](https://github.com/nicorenezweers/nrzNetwork/releases). Selbst bauen (Docker mit buildx und `cargo`, das nur das Dockerfile erzeugt – übersetzt wird alles in Docker):

```bash
git clone https://github.com/nicorenezweers/nrzNetwork.git
cd nrzNetwork
cargo xtask docker -t nrzdns
```

<http://localhost:8380> öffnen. Das erste Konto, das du anlegst, wird Administrator. Danach ist die Registrierung geschlossen, bis du sie in den Einstellungen öffnest; dort legst du auch Konten für andere an.

> **Port 53 muss frei sein.** Nur ein DNS-Server kann darauf lauschen. Technitium, Pi-hole, AdGuard Home oder dnsmasq vorher beenden. Auf vielen Linux-Rechnern belegt `systemd-resolved` die Adresse `127.0.0.53:53`; das stört nicht, solange niemand auf `0.0.0.0:53` lauscht. Ist der Port belegt, startet das Programm selbst trotzdem und zeigt den Fehler im Dashboard; in Docker (und auf Umbrel) scheitert schon das Veröffentlichen des Ports, der Container startet dann nicht.

### Docker Compose

```yaml
services:
  nrzdns:
    image: ghcr.io/nicorenezweers/nrz-dns:1.15.0   # oder: build: .
    container_name: nrzdns
    restart: unless-stopped
    ports:
      - "53:53/udp"
      - "53:53/tcp"
      - "8380:8380"
    volumes:
      - nrzdns-data:/data          # nrzdns.db: alles in einer Datei
    environment:
      TZ: Europe/Zurich
      # NRZDNS_COOKIE_SECURE: "true"   # wenn die Oberfläche über HTTPS läuft
    sysctls:
      net.ipv4.ip_unprivileged_port_start: 0   # Benutzer 1000 darf Port 53 öffnen
    cap_drop: [ALL]
    security_opt: [no-new-privileges:true]

volumes:
  nrzdns-data:
```

```bash
docker compose up -d
```

Der Container läuft als Benutzer `1000` ohne zusätzliche Rechte. Docker (ab 20.10) erlaubt im Container ohnehin jedem Benutzer die niedrigen Ports; die Zeile `sysctls` macht das ausdrücklich.

> **`/data` sichern.** Dort liegt `nrzdns.db` mit allem drin (während der Server läuft, zusätzlich `nrzdns.db-wal` und `nrzdns.db-shm`). Den ganzen Ordner kopieren, am besten bei gestopptem Container.

### Ohne Docker

Jedes [Release](https://github.com/nicorenezweers/nrzNetwork/releases) enthält das Programm für Linux (`amd64`, `arm64`) als einzelne Datei, dazu `SHA256SUMS`. Die Wurzelzertifikate für DoT, DoH und das Laden der Sperrlisten bringt es selbst mit.

```bash
curl -LO https://github.com/nicorenezweers/nrzNetwork/releases/download/v1.15.0/nrzdns-1.15.0-linux-amd64
chmod +x nrzdns-1.15.0-linux-amd64
sudo setcap 'cap_net_bind_service=+ep' nrzdns-1.15.0-linux-amd64   # Port 53 ohne root
NRZDNS_ADDRESS=0.0.0.0 NRZDNS_PORT=8380 NRZDNS_DATA_DIR=./data ./nrzdns-1.15.0-linux-amd64
```

Zum Ausprobieren, ohne Port 53 anzufassen, auf einer anderen Adresse lauschen lassen, z. B. `NRZDNS_DNS_LISTEN=127.0.0.1:5353`, und mit `dig @127.0.0.1 -p 5353 example.com` fragen.

### Beide Architekturen

```bash
cargo xtask docker --platform linux/amd64,linux/arm64 -t <registry>/nrz-dns:1.15.0 --push
```

Rust wird quer-kompiliert, nicht emuliert – der arm64-Build ist auf einem x86-Rechner also schnell und umgekehrt. Nur das Programm, gzip-komprimiert je Architektur (für die Umbrel-App): `cargo xtask store-binaries <ordner>`. Das Dockerfile als Datei (z. B. für `docker compose build`): `cargo xtask dockerfile > Dockerfile`.

## Erste Schritte

1. **Konto anlegen**: das erste wird Administrator.
2. **Vorgelagerte Server wählen** (*Einstellungen → Weiterleitung*): die Vorgabe (Cloudflare und Quad9 über UDP) behalten oder eine Vorlage wählen, z. B. Quad9 über DoH, dazu eine Strategie.
3. **Sperren einschalten** (*Sperren*): eine oder zwei Vorlagen hinzufügen, z. B. *HaGeZi Multi Normal*. Sie laden sofort und werden alle 24 Stunden neu geladen.
4. **Eigene Namen eintragen** (*Zonen*): eine Zone wie `home.arpa` anlegen und `nas`, `drucker` usw. als A-Einträge hinzufügen – oder eine vorhandene BIND-Zonendatei importieren. Für Namen, die dein Router kennt (z. B. `fritz.box`), eine Weiterleitungszone auf den Router anlegen.
5. **Das Netz auf nrzNetwork umstellen** – siehe unten.
6. **Zuschauen**: Dashboard und Abfrageprotokoll zeigen jede Anfrage; der DNS-Client beantwortet „Wohin zeigt dieser Name?“.

## nrzNetwork im Netz benutzen

- **Ganzes Netz**: In den DHCP-Einstellungen des Routers als DNS-Server für die Geräte die IP-Adresse des Rechners mit nrzNetwork eintragen (z. B. `192.168.1.10`). Die Geräte übernehmen sie, sobald sie ihre Adresse erneuern (WLAN neu verbinden oder neu starten). Dem Rechner vorher eine feste IP-Adresse geben.
- **Einzelnes Gerät**: In den Netzwerkeinstellungen des Geräts diese IP-Adresse als DNS-Server eintragen.
- **Prüfen**: `nslookup example.com 192.168.1.10` oder `dig @192.168.1.10 example.com`; die Anfrage erscheint im Protokoll.
- **DNS über HTTPS**: `https://<dein-host>/dns-query` funktioniert für Browser und Handys, sobald die Oberfläche über HTTPS erreichbar ist (Reverse Proxy mit Zertifikat); über einfaches HTTP taugt es nur zum Testen.
- Keinen zweiten DNS-Server in den DHCP-Einstellungen lassen (oder dort ein zweites nrzNetwork eintragen) – sonst umgehen die Geräte Sperren und eigene Namen, wann sie wollen.

## Konfiguration

Einstellungen für den Start kommen aus Umgebungsvariablen mit dem Präfix `NRZDNS_` (oder aus `Rocket.toml`). Alles rund um DNS selbst stellst du in der Oberfläche ein; es gilt sofort, ohne Neustart.

| Variable | Vorgabe | Bedeutung |
|---|---|---|
| `NRZDNS_DATA_DIR` | `./data` (`/data` im Image) | Ordner für `nrzdns.db` |
| `NRZDNS_PORT` / `NRZDNS_ADDRESS` | `8380` / `0.0.0.0` | Wo Oberfläche, API und `/dns-query` lauschen |
| `NRZDNS_DNS_LISTEN` | `0.0.0.0:53` | DNS-Adressen (jeweils UDP und TCP), durch Kommas getrennt, z. B. `0.0.0.0:53,[::]:53` |
| `NRZDNS_HOST_GATEWAY` | – | Name, unter dem der Container das Gerät selbst erreicht (`host.docker.internal`); Ziele auf einer Adresse dieses Geräts verbindet der Proxy darüber |
| `NRZDNS_PROXY_FALLBACK` | – | Wohin Namen ohne Eintrag, IP-Adressen und `.local`-Namen gehen, z. B. `http://host.docker.internal:80` (umbrelOS) |
| `NRZDNS_PROXY_TLS_LISTEN` | `0.0.0.0:443` | HTTPS-Eingang des Proxys (leer = keiner) |
| `NRZDNS_TLS_FALLBACK` | – | Wohin HTTPS für Namen ohne eigenes Zertifikat unverändert geht, z. B. `host.docker.internal:443` (umbrelOS) |
| `NRZDNS_PROXY_LISTEN` | `0.0.0.0:80` | Adressen des Reverse Proxys (HTTP), durch Kommas getrennt; leer = aus. Auf einem anderen Port brauchen die Namen ihn: `http://music.nrz.ch:8480/` |
| `NRZDNS_CONFIG` | `Rocket.toml` (`/app/Rocket.toml` im Image) | Pfad der Konfigurationsdatei |
| `NRZDNS_COOKIE_SECURE` | `false` | Auf `true` setzen, wenn die Oberfläche über HTTPS läuft |
| `NRZDNS_IP_HEADER` | `X-Real-IP` | Kopfzeile, in der ein Reverse Proxy die Adresse des Aufrufers mitschickt (Bremse gegen Passwort-Raten); `false`, wenn nrzNetwork direkt erreichbar ist |
| `NRZDNS_MCP_TOKEN` | – | Zusätzliches festes Token für den MCP-Server (mindestens 16 Zeichen); Tokens sonst unter *Einstellungen → MCP* erzeugen |
| `NRZDNS_LOG` | `info` | Log-Filter, z. B. `debug` oder `info,nrzdns_engine=debug` |

### DNS-Einstellungen (Oberfläche, Admins)

| Bereich | Vorgaben |
|---|---|
| Allgemein | Servername `ns.nrzdns.local` (SOA/NS neuer Zonen), Standard-TTL 3600 s, DoH an, kein Rate-Limit, private Rückwärtsanfragen lokal beantwortet, AAAA-Antworten an |
| Weiterleitung | Cloudflare `1.1.1.1` und Quad9 `9.9.9.9` über UDP, der Reihe nach, 2000 ms Zeitlimit je Server |
| Cache | An, 50 000 Einträge, TTL zwischen 10 s und 1 Tag, negative Antworten 300 s, abgelaufene Einträge bis 1 Tag ausliefern, Prefetch an |
| Sperren | An, Sperrseite (Adresse dieses Geräts; `0.0.0.0`, solange keine bekannt ist), TTL 60 s, Listen alle 24 Stunden neu laden |
| Abfrageprotokoll | An, 30 Tage aufbewahren |

## Reverse Proxy

*Reverse Proxy* in der Seitenleiste (Admins): Namen und Ziel eintragen, z. B. `music.nrz.ch` → `http://100.78.107.162:8350` und `bench.nrz.ch` → `http://100.78.107.162:8550`. Ab dann:

1. nrzNetwork beantwortet `music.nrz.ch` selbst (A-Eintrag, TTL 60 s) mit der Adresse des Geräts, auf dem es läuft – der Adresse unter *DNS-Antwort* oder, wenn die leer ist, der IP aus dem Ziel.
2. Der Browser verbindet sich mit Port 80 dieses Geräts; der eingebaute Proxy wählt das Ziel nach dem `Host`-Header und reicht die Anfrage weiter (`Host` bleibt, dazu `X-Forwarded-For`, `X-Real-IP`, `X-Forwarded-Host`, `X-Forwarded-Proto`; Inhalte gestreamt; WebSockets durchgereicht; `Location: http://100.78.107.162:8350/…` wird zu `http://music.nrz.ch/…`).

Die Domain muss dir nicht gehören – die Namen gibt es nur für Geräte, die nrzNetwork fragen. Darum auch nur HTTP: Für eine fremde Domain gibt es kein öffentliches Zertifikat. Eine vorhandene Zone wie `nrz.ch` läuft weiter; der Proxy-Name gewinnt, weil er genauer ist. Unbekannte Namen bekommen vom Proxy eine kurze 404-Seite, nicht erreichbare Ziele eine 502-Seite.

## MCP-Server (nrzDashboard)

nrzNetwork spricht das [Model Context Protocol](https://modelcontextprotocol.io) unter `POST /mcp` (Streamable HTTP, JSON-RPC 2.0, zustandslos, Antworten als `application/json`). Darüber holt sich nrzDashboard die Werte für seine Übersicht – jeder andere MCP-Client geht genauso.

1. *Einstellungen → MCP* (Admins): Namen vergeben (z. B. `Claude Code`), **Darf ändern** wählen oder nicht, **Token erzeugen** und kopieren (es wird nur einmal angezeigt, mit fertiger `claude mcp add …`-Zeile und JSON-Konfiguration). Beliebig viele Tokens – eins je Client –, einzeln umbenennen, Rechte ändern oder löschen. `NRZDNS_MCP_TOKEN` gilt zusätzlich.
2. Im Client: URL `http://<host>:8380/mcp`, Kopfzeile `Authorization: Bearer <Token>` (oder `X-API-Key: <Token>`; Clients ohne eigene Kopfzeilen nehmen `http://<host>:8380/mcp?token=<Token>`).

```bash
claude mcp add --transport http nrznetwork http://<host>:8380/mcp --header "Authorization: Bearer <Token>"
```

CORS ist offen (der Zugang hängt nur am Token, nie an Cookies) – auch Clients im Browser wie der MCP Inspector gehen; `/.well-known/oauth-*` antwortet 404, damit Clients nicht auf OAuth warten.

| Werkzeug | Liefert / macht |
|---|---|
| `get_overview` `{range?}` | Alles für eine Übersichtskachel in einem Aufruf: Server (Version, Laufzeit, Anfragen/s, DNS-Ports), Sperren (`enabled`, `paused`, `active`, gesperrte Domains, Listen), Summen im Zeitraum (`queries`, `blocked`, `blockedPercent`, `cacheHitPercent`, `failed`, `clients`, `avgDurationMs` …), Weiterleitungen gesund/gesamt, Top 5 Clients, Domains und gesperrte Domains |
| `get_statistics` `{range?, includeSeries?}` | Das ganze Dashboard: Summen, Zeitreihe, Top-10-Listen, Abfragetypen, Serverzustand |
| `get_server_status`, `get_blocking_status` | Laufender Zustand; Sperren mit allen Listen |
| `get_query_log` `{client?, qname?, qtype?, kind?, rcode?, from?, to?, limit?, page?}` | Letzte Anfragen, neueste zuerst |
| `check_domain` `{domain}` | Würde gesperrt – und warum? |
| `list_zones`, `list_rules` `{list}` | Eigene Zonen; eigene Sperren oder Ausnahmen |
| `set_blocking` `{enabled}` ✎ | Filter ein- oder ausschalten (Einschalten beendet auch eine Pause) |
| `pause_blocking` `{minutes}` ✎ | Filter pausieren, `0` setzt fort |
| `set_blocklist` `{id \| name, enabled}` ✎ | Eine Sperrliste umschalten |
| `update_blocklists` ✎ | Alle aktiven Listen jetzt neu laden |
| `add_rule` / `remove_rule` `{list, domain, comment?}` ✎ | Eigene Sperren (`blocked`) oder Ausnahmen (`allowed`) |
| `flush_cache` `{name?}` ✎ | Cache leeren, ganz oder für einen Namen |
| `dns_lookup` `{name, type?, server?}` | DNS-Abfrage wie dig – leerer `server` = nrzNetwork selbst, sonst z. B. `1.1.1.1` |
| `get_zone` `{zone}` | Eine Zone (Name oder id) mit allen Einträgen |
| `get_cache` `{q?, limit?}` | Einträge im Cache |
| `get_proxy`, `get_tls` | Reverse-Proxy-Einträge, Ports, Zugriffsschutz; HTTPS-Zertifikat |
| `get_docker`, `get_docker_logs` `{container, tail?}` | Container mit internen IPs, Netzen, Ports; Logs eines Containers |
| `get_network_map` | Die ganze Netzwerkkarte (Internet, Tunnel, Proxy, Docker, LAN, NetBird) |
| `get_devices` `{days?}`, `get_access_log` `{host?, limit?}`, `get_access_stats` `{period?, host?}` | Clients; Zugriffs-Monitor des Reverse Proxy |
| `get_components` | Eingebauter Cloudflare Tunnel und NetBird-Client |
| `list_api_endpoints` | Alle REST-Endpunkte (für `api_request`) |
| `api_request` `{method?, path, query?, body?}` | Beliebiger REST-Aufruf (Pfad ohne `/api/v1`, z. B. `GET /cloudflare/zones`); lesende Tokens nur `GET` |

✎ = nur für Tokens, die ändern dürfen; bei den anderen fehlen diese Werkzeuge schon in der Liste. `range` ist `hour`, `day` (Vorgabe), `week` oder `month`. Jedes Ergebnis kommt als JSON-Text und als `structuredContent`.

```bash
curl -s http://localhost:8380/mcp -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_overview","arguments":{"range":"day"}}}'
```

## Umbrel

nrzNetwork gibt es im Community-App-Store **nrzStore** (App `nrzNetwork`, ID `nrz-dns`). Die Oberfläche öffnet sich auf Port `8400`; nrzNetwork hat eine eigene Anmeldung, die Umbrel-Anmeldung davor ist ausgeschaltet. DNS läuft auf Port `53` (UDP und TCP) des Umbrel-Geräts, darum:

- **Port 53 muss auf dem Gerät frei sein** – nicht gleichzeitig Technitium, Pi-hole oder AdGuard Home betreiben. Sonst schlägt die Installation mit `app-script install … Exit-Code 1` fehl; den anderen DNS-Server zuerst stoppen oder deinstallieren.
- **Umzug von Technitium:** in Technitium jede Zone exportieren (*Zones → Export Zone*) und in nrzNetwork unter *Zonen → Importieren* einlesen; Sperrlisten, eigene Sperren und Ausnahmen neu eintragen. Zeigt der Router schon auf den Umbrel, ändert sich dort nichts – nrzNetwork übernimmt dieselbe Adresse.
- In den DHCP-Einstellungen des Routers (oder an einzelnen Geräten) die **IP-Adresse deines Umbrel** als DNS-Server eintragen.
- **Namen ohne Port:** Port 80 gehört umbrelOS. Ein kleiner Wächter, den die App auf dem Gerät startet (`hooks/nrznetwork-gateway`), leitet Port 80 der LAN- und VPN-Adressen an den Proxy (Port `81`), solange der Proxy antwortet; alles ohne Eintrag – auch die umbrelOS-Oberfläche – reicht der Proxy unverändert an umbrelOS weiter. Antwortet der Proxy nicht, bekommt umbrelOS Port 80 innerhalb von Sekunden zurück. Abschalten unter *Reverse Proxy → Gerät und Sperrseite*.
- Ziele auf dem Umbrel selbst (z. B. `http://100.78.107.162:8400`) erreicht der Proxy über `host.docker.internal`; die Adressen des Geräts (LAN, NetBird/Tailscale) werden erkannt, jeder Client bekommt die passende.

Die Daten liegen unter `app-data/nrz-dns/data`. Die App bringt das Programm für `amd64` und `arm64` im Store mit und startet es im offiziellen Image `debian:trixie-slim` als Benutzer `1000`, ohne zusätzliche Rechte.

## Entwicklung

```bash
# Backend (Rust 1.85+): Server, DNS-Engine, gemeinsame Typen
NRZDNS_DNS_LISTEN=127.0.0.1:5353 cargo run   # Oberfläche auf http://localhost:8380, DNS auf Port 5353
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings

# Oberfläche mit Hot Reload (dx = dioxus-cli 0.7)
rustup target add wasm32-unknown-unknown
cargo install dioxus-cli --version 0.7.10 --locked
cargo xtask serve                 # http://localhost:8080, leitet /api und /static an :8380 weiter

# Oberfläche ins Programm einbetten
cargo xtask ui && cargo build --release
```

Der Workspace hat vier Crates: `nrzdns` (der Rocket-Server, `src/`), `nrzdns-engine` (der DNS-Server selbst, `engine/`), `nrzdns-types` (alle Typen der API, `types/`, auch von der Oberfläche benutzt) und `xtask` (Build-Aufgaben, `cargo xtask`). Die Oberfläche in `ui/` ist ein eigenes Projekt für `wasm32-unknown-unknown`. `cargo xtask ui` baut sie nach `ui/dist/public`; `build.rs` bettet sie (gzip) zusammen mit `locales/` und `static/` ein. Ohne sie läuft der Server trotzdem und zeigt einen Hinweis statt der Oberfläche. Übersetzungen liegen in `locales/<sprache>.json` (deutscher Ausgangstext → Übersetzung).

Der Code ist zu 100 % Rust: Die Tailwind-Eingabe steht in `ui/tailwind.rs` (`cargo xtask` schreibt daraus `ui/tailwind.css`, bevor `dx` läuft), das Dockerfile in `xtask/src/docker.rs` und die Datenbank-Migrationen in `src/db/migrations.rs`; Build-Aufgaben sind `cargo xtask` statt Shell-Skripte.

Die JSON-API liegt unter `/api/v1`; die Tabelle aller Endpunkte steht im Doc-Kommentar von [`types/src/lib.rs`](../types/src/lib.rs). Aufbau im Detail: [ARCHITECTURE.md](../ARCHITECTURE.md).

## Lizenz

AGPL-3.0-or-later. nrzNetwork ist ein eigenständiges Projekt nach dem Vorbild von [Technitium DNS Server](https://technitium.com/dns/) und teilt keinen Code damit. Die Komponenten der Oberfläche basieren auf [rust-ui](https://github.com/rust-ui/ui) (MIT), die Schrift ist [Inter](https://rsms.me/inter/) (OFL).
