# Domain Investigation Run Sheet

A step-by-step manual investigation methodology for CTI and OSINT domain analysis. Each phase builds on the previous — work sequentially, pivot on findings, and document everything.

---

## Phase 1: Initial Domain Triage

**Objective:** Establish baseline facts about the domain before deeper analysis.

### 1.1 WHOIS Lookup

- Query WHOIS for the target domain (use `whois` CLI, DomainTools, or whois.domaintools.com)
- Record:
  - Registrar and registration date
  - Expiry date
  - Nameservers
  - Registrant org/name/email (if not redacted)
  - WHOIS privacy service in use (e.g. Withheld for Privacy, Domains By Proxy)
- **Pivot point:** Registrant email, registrant org, or nameservers can link to other domains

### 1.2 DNS Record Enumeration

Query all record types and document the results:

- **A / AAAA** — Hosting IP(s), IPv6 presence
- **MX** — Mail infrastructure, third-party mail providers (Google Workspace, Microsoft 365, Zoho)
- **NS** — Authoritative nameservers, hosting provider clues
- **TXT** — SPF, DKIM, DMARC, domain verification tokens (Google, Facebook, etc.)
- **CNAME** — Aliases, CDN usage (Cloudflare, Fastly, AWS CloudFront)
- **SOA** — Primary nameserver, admin email

Tools: `dig`, `nslookup`, `host`, DNSDumpster, SecurityTrails

### 1.3 Hosting & Infrastructure Fingerprint

- Resolve A record IP(s) and identify the hosting provider via IP WHOIS/ASN lookup
- Check for CDN/reverse proxy (Cloudflare, Akamai, AWS CloudFront) — if present, note that the origin IP is masked
- Identify the ASN and netblock owner
- Tools: `whois <IP>`, ipinfo.io, bgp.he.net, Hurricane Electric BGP Toolkit

---

## Phase 2: Passive DNS & Certificate Transparency

**Objective:** Map the domain's historical infrastructure and discover related assets.

### 2.1 Passive DNS (pDNS)

- Query pDNS for all historical resolutions of the domain
- Look for IP address changes over time (infrastructure migration, bulletproof hosting rotation)
- Reverse-resolve each IP — what other domains have pointed to the same IPs?
- Sources: SecurityTrails, Farsight DNSDB, VirusTotal (Passive DNS tab), RiskIQ/PassiveTotal, Mnemonic pDNS

### 2.2 Certificate Transparency (CT) Logs

- Search CT logs for all certificates issued for the domain and its subdomains
- This often reveals subdomains not visible via brute-force enumeration (staging, admin, api, mail, cpanel, webmail, dev, test)
- Record: issuer (Let's Encrypt is common for phishing), validity dates, SANs (Subject Alternative Names — other domains on the same cert)
- **Pivot point:** SANs can link seemingly unrelated domains to the same operator
- Tools: crt.sh (`%.example.com`), Censys, Google CT search

### 2.3 Subdomain Enumeration

- Combine CT log results with passive enumeration
- Cross-reference with DNS brute-forcing if scope permits (but note this is active, not passive)
- Tools: Subfinder, Amass (passive mode), crt.sh, SecurityTrails subdomain API

---

## Phase 3: IP Address Investigation

**Objective:** Investigate all IPs associated with the domain (current and historical).

### 3.1 IP Reputation & Enrichment

For each IP, query the following:

- **VirusTotal** — Community score, communicating files, detected URLs, passive DNS
- **AbuseIPDB** — Abuse reports, confidence score, ISP, usage type
- **GreyNoise** — Is it a known scanner/benign service, or targeted malicious activity?
- **Shodan** — Open ports, banners, services, SSL cert details, technologies, vulnerabilities
- **Censys** — Similar to Shodan; services, certificates, host metadata
- **ThreatFox (abuse.ch)** — IOC associations, malware family links

### 3.2 Geolocation & ASN Context

- Geolocate each IP (MaxMind, ipinfo.io)
- Identify the ASN and check its reputation — is it a known bulletproof hosting provider?
- Known suspicious ASNs to flag: certain providers in RU, NL, LV, BZ, SC frequently used for phishing/C2
- Tools: bgp.he.net, ipinfo.io, Shodan ASN search

### 3.3 Port & Service Analysis (Passive)

- Review Shodan/Censys data for each IP:
  - Web servers (Apache, Nginx, LiteSpeed, IIS) — version and config clues
  - Open admin panels (cPanel, Plesk, DirectAdmin, phpMyAdmin)
  - SSH banners and versions
  - Unusual open ports (e.g. 8443, 8080, 2083, 2087, 4443)
  - TLS certificate details on each port
- **Pivot point:** Self-signed or shared certificates across IPs can cluster infrastructure

---

## Phase 4: Web Content & Technology Analysis

**Objective:** Examine what the domain is actually serving and how it's built.

### 4.1 Live Site Inspection (Sandboxed)

- Load the site in a sandboxed browser or use URLScan.io for a safe render
- Capture: screenshot, page title, favicon, HTTP headers, response codes
- Check for redirects (HTTP 301/302 chains, JavaScript redirects, meta refreshes)
- Note any gating (geofencing, user-agent filtering, CAPTCHA, cloaking)
- Tools: URLScan.io, any.run, Browserling, hybrid-analysis.com

### 4.2 Technology Stack Fingerprinting

- Identify CMS (WordPress, Joomla, custom), frameworks, JavaScript libraries
- Check HTTP response headers: Server, X-Powered-By, Content-Security-Policy
- Look for generator meta tags, common file paths (/wp-login.php, /administrator)
- Tools: Wappalyzer, BuiltWith, WhatWeb, manual header inspection

### 4.3 Source Code Review

- View page source — look for:
  - Hardcoded credentials or API keys (common in phishing kits)
  - Comments left by kit authors (watermarks, signatures, Telegram handles)
  - Obfuscated JavaScript (Base64, eval(), document.write)
  - Form action URLs — where are credentials being exfiltrated to? (Telegram bot API, external PHP endpoints, email via mail())
  - Hidden iframes loading external content
  - Anti-analysis code (devtools detection, right-click disabling, VM detection)
- Check `robots.txt` and `sitemap.xml` for directory structure hints
- Look for exposed directories (directory listing enabled)

### 4.4 Favicon Hash Analysis

- Download the favicon and compute its MurmurHash3
- Search Shodan using `http.favicon.hash:<hash>` to find other servers using the same favicon
- This is highly effective for clustering phishing infrastructure using the same kit
- Tools: Shodan favicon hash search, custom script

---

## Phase 5: Threat Intelligence Correlation

**Objective:** Cross-reference findings with threat intelligence platforms.

### 5.1 Multi-Engine Scanning

- **VirusTotal** — URL and domain scan; check detections, community comments, relations graph
- **URLScan.io** — Rendered page, DOM snapshot, outbound requests, linked domains
- **Hybrid Analysis / any.run** — Dynamic analysis if downloadable payloads are present
- **Google Safe Browsing** — Check if flagged (Transparency Report)
- **PhishTank** — Community-reported phish; check if the domain/URL is listed

### 5.2 Threat Feed & IOC Checks

- Check the domain and IPs against:
  - abuse.ch ThreatFox, URLhaus, MalwareBazaar
  - AlienVault OTX
  - MISP feeds
  - OpenCTI / your local MISP instance if available
  - Emerging Threats / ET Intelligence (Proofpoint)
- Record any associated malware families, campaigns, or threat actor attributions

### 5.3 Brand Impersonation Analysis

- If the domain appears to target a specific brand:
  - Compare the domain name to the legitimate brand domain (typosquatting, homoglyph, combosquatting)
  - Compare page content/design to the legitimate site
  - Check if the brand's trademarks or logos are used
  - Note the credential fields — what is being harvested? (username/password, MFA tokens, credit cards, PII)

---

## Phase 6: Infrastructure Clustering & Pivoting

**Objective:** Identify related domains, shared infrastructure, and the broader campaign.

### 6.1 Pivoting on Shared Infrastructure

Use the data collected to pivot and find related domains:

- **Same IP** — Reverse DNS, Shodan/Censys host search, VirusTotal passive DNS
- **Same nameservers** — Query all domains using those NS records
- **Same registrant** — Reverse WHOIS on email, org, phone (DomainTools, WhoisXML API)
- **Same SSL certificate** — Search by cert serial, SHA-256 fingerprint, or SAN entries (Censys, crt.sh)
- **Same favicon hash** — Shodan `http.favicon.hash`
- **Same Google Analytics / AdSense ID** — SpyOnWeb, BuiltWith, publicwww.com
- **Same SSH host key** — Shodan `ssh.fingerprint`
- **Same JARM hash** — TLS server fingerprint clustering (Shodan `ssl.jarm`)

### 6.2 Domain Generation Pattern Analysis

- Check if the domain fits a pattern (DGA-like, sequential registration, keyword combos)
- Search for similar domains registered around the same time
- Tools: DNSTwist, URLCrazy, dnstwister.report, manual regex on CT logs

### 6.3 Timeline Correlation

- Build a timeline:
  - Domain registration date
  - First seen in pDNS / CT logs
  - First seen in VT / URLScan
  - First abuse report
  - Infrastructure changes (IP rotations, nameserver changes)
- Correlate with known campaign timelines or threat actor activity windows

---

## Phase 7: OSINT & Attribution Leads

**Objective:** Gather any open-source intelligence that could support attribution.

### 7.1 Email & Contact Pivoting

- If registrant email is visible, search for it across:
  - Breach databases (Have I Been Pwned — presence only, not contents)
  - Social media, forums, paste sites
  - Other domain registrations (reverse WHOIS)
- Check abuse contact email for the hosting provider

### 7.2 Social Media & Dark Web Mentions

- Search for the domain on:
  - Twitter/X, Telegram, Reddit, Hacker Forums
  - Paste sites (Pastebin, Ghostbin, rentry.co)
  - Dark web search engines (Ahmia, if in scope)
- Look for the domain being advertised, discussed, or sold
- Check Telegram channels relevant to phishing kit distribution

### 7.3 Wayback Machine / Web Archives

- Check archive.org for historical snapshots of the domain
- Compare content changes over time — was it a legitimate site that was compromised, or purpose-built?
- Look for earlier versions that may have had less obfuscation or exposed more infrastructure
- Tools: web.archive.org, Wayback CDX API

---

## Phase 8: Documentation & Reporting

**Objective:** Produce actionable output from the investigation.

### 8.1 IOC Collection

Compile all confirmed IOCs in a structured format:

- Domains and subdomains
- IP addresses (with first/last seen dates)
- URLs (phishing pages, kit download locations, exfil endpoints)
- Email addresses
- File hashes (if payloads were identified)
- SSL certificate fingerprints
- JARM hashes
- Favicon hashes

### 8.2 Investigation Notes

Document for each finding:

- What was observed
- Which tool/source provided the data
- Timestamp of the observation
- Confidence level (confirmed malicious, suspicious, inconclusive, benign)
- Relationship to other findings (pivots)

### 8.3 Deliverables

Depending on context, produce:

- **Structured IOC export** — JSON, STIX 2.1, CSV, or MISP event
- **Investigation report** — Narrative walkthrough with evidence screenshots
- **Obsidian note** — Zettelkasten-formatted note with backlinks to related investigations
- **Detection content** — Splunk queries, YARA rules, Sigma rules, or firewall block rules
- **Takedown request** — Report to registrar, hosting provider, or brand abuse team with evidence

---

## Quick Reference: Tool Cheat Sheet

| Purpose | Tools |
|---|---|
| WHOIS | `whois`, DomainTools, WhoisXML API |
| DNS | `dig`, DNSDumpster, SecurityTrails |
| Passive DNS | SecurityTrails, Farsight DNSDB, VirusTotal, PassiveTotal |
| CT Logs | crt.sh, Censys |
| IP Reputation | VirusTotal, AbuseIPDB, GreyNoise, Shodan, Censys |
| Threat Intel | ThreatFox, URLhaus, AlienVault OTX, PhishTank |
| Web Analysis | URLScan.io, any.run, Hybrid Analysis |
| Tech Fingerprint | Wappalyzer, BuiltWith, WhatWeb |
| Pivoting | Shodan, Censys, SpyOnWeb, crt.sh, reverse WHOIS |
| Archives | Wayback Machine, Wayback CDX API |
| Domain Variants | DNSTwist, URLCrazy, dnstwister.report |

---

## Investigation Principles

1. **Passive first, always.** Exhaust passive sources before any active interaction with the target. Active scanning tips off the operator and may be out of scope.
2. **Pivot relentlessly.** Every data point is a potential pivot. One shared IP, one reused certificate, one registrant email can crack open an entire campaign.
3. **Timestamp everything.** Infrastructure is ephemeral. Record when you observed each finding — it may be gone tomorrow.
4. **Assume cloaking.** Phishing operators use geofencing, user-agent filtering, and referrer checks. If a page looks blank or redirects to Google, try different request parameters or use URLScan from a different region.
5. **Cluster before you attribute.** Build the infrastructure map first. Attribution is the hardest part and should come last, supported by evidence, not assumptions.
6. **Automate the repeatable parts.** Once your methodology is solid, script the mechanical lookups and focus your manual effort on analysis and pivoting.
