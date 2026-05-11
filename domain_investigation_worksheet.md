# Domain Investigation Worksheet

| Field | Value |
|---|---|
| **Target Domain** |  |
| **Investigation Date** |  |
| **Analyst** |  |
| **Case / Ticket ID** |  |

**Investigation Objective:**


---

## Phase 01 — WHOIS & DNS

| Field | Value |
|---|---|
| **Registrar** |  |
| **Registration Date** |  |
| **Expiry Date** |  |
| **Nameservers** |  |
| **A / AAAA Records** |  |
| **MX Records** |  |
| **CNAME / CDN** |  |
| **Hosting Provider / ASN** |  |

**Registrant Info:**


**TXT Records (SPF / DKIM / DMARC):**


---

## Phase 02 — Passive DNS & CT Logs

| Field | Value |
|---|---|
| **Certificate Issuer** |  |
| **Cert Validity Dates** |  |

**pDNS Historical IPs:**


**Co-hosted Domains (same IP):**


**CT Log Subdomains:**


**SAN Entries of Interest:**


---

## Phase 03 — IP Investigation

| Field | Value |
|---|---|
| **Primary IP(s)** |  |
| **VirusTotal IP Score** |  |
| **AbuseIPDB Confidence** |  |
| **GreyNoise Classification** |  |
| **Geolocation** |  |

**Shodan Open Ports & Services:**


**Shodan Vulnerabilities:**


**ASN Reputation Notes:**


---

## Phase 04 — Web Content & Tech Stack

| Field | Value |
|---|---|
| **URLScan.io Scan URL** |  |
| **Page Title** |  |
| **Favicon MurmurHash3** |  |
| **robots.txt / sitemap.xml** |  |

**Screenshot / Visual Notes:**


**Redirect Chain:**


**Technology Stack:**


**Source Code Notes:**
> Kit watermarks, Telegram handles, obfuscation, anti-analysis...


**Form Action / Exfil Target:**


---

## Phase 05 — Threat Intel Correlation

| Field | Value |
|---|---|
| **VirusTotal Domain Score** |  |
| **Google Safe Browsing** |  |
| **PhishTank Status** |  |
| **Impersonated Brand** |  |
| **Impersonation Technique** |  |
| **Data Being Harvested** |  |

**VT Community Comments:**


**ThreatFox / URLhaus Hits:**


**AlienVault OTX Pulses:**


---

## Phase 06 — Infrastructure Clustering

| Field | Value |
|---|---|
| **JARM Hash** |  |
| **SSH Host Key Fingerprint** |  |
| **Google Analytics / AdSense IDs** |  |

**Related Domains (Pivots):**


**Pivot Method Used:**
> How each related domain was linked — same SSL SAN, same favicon hash, reverse WHOIS, etc.


**Domain Pattern / DGA Notes:**


**Campaign Timeline:**
> Key dates: registration, first pDNS, first VT submission, IP rotations...


---

## Phase 07 — OSINT & Attribution Leads

| Field | Value |
|---|---|
| **Suspected Threat Actor / Group** |  |
| **Attribution Confidence** | `— / Confirmed / High / Moderate / Low / Unattributed` |

**Email / Contact Pivots:**


**Social / Dark Web Mentions:**


**Wayback Machine Notes:**


---

## Phase 08 — IOC Summary & Verdict

| Field | Value |
|---|---|
| **Verdict** | `— / Confirmed Malicious / Likely Malicious / Suspicious / Inconclusive / Benign` |

**IOC — Domains:**

```
```

**IOC — IP Addresses:**

```
```

**IOC — URLs:**

```
```

**IOC — File Hashes:**

```
```

**IOC — Email Addresses:**

```
```

**IOC — Cert Fingerprints:**

```
```

**Verdict Summary:**


**Actions Taken:**
> Takedown requested, IOCs shared, detections deployed, blocked at firewall...
