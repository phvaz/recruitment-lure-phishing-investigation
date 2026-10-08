# Indicators of Compromise — PHI-2026-001

Indicators extracted from the investigation of the Linea recruitment-lure phishing campaign (`lineatalent.xyz`). All indicators are drawn from the preserved `.eml` and passive public sources; see [`collection-log.md`](../collection-log.md) for provenance.

> **Read the caveat first.** Shared, legitimate cloud infrastructure (Amazon SES, Resend) appears in this campaign only because the operator rented it. **Do not blocklist AWS/SES/Resend infrastructure or the SES sending IP.** Only the attacker-owned domain, subdomains, sender address, landing-page IP and URL are safe to action.

---

## Network indicators

| Type | Indicator | Context | Actionable? |
|---|---|---|---|
| Domain | `lineatalent.xyz` | Lookalike of `linea.build`; registered 2026-09-27 | ✅ Yes — block / monitor |
| Domain (subdomain) | `hr.lineatalent.xyz` | DKIM signing + sender domain | ✅ Yes |
| Domain (subdomain) | `rsend.hr.lineatalent.xyz` | Return-Path / SPF (Resend envelope) | ✅ Yes |
| Domain (subdomain) | `open.hr.lineatalent.xyz` | Open-tracking beacon host (AWS-hosted; CT-confirmed) | ✅ Yes |
| URL | `https://lineatalent.xyz/` | Phishing landing page ("Linea \| Careers" clone) | ✅ Yes |
| URL | `https://open.hr.lineatalent.xyz/CI0/010301a10b6d1feb-89530428-678e-4998-a38c-849eb49c88c9-000000/...=258` | Per-recipient tracking pixel | ✅ Yes |
| IPv4 | `193.160.32.10` | Landing-page host — AS206216 ADVIN-AS, San Jose US | ✅ Yes (host-specific) |
| IPv4 | `23.249.215.51` | SES sending IP (rDNS `d215-51.smtp-out.sa-east-1.amazonses.com`) | ⚠️ No — shared Amazon SES, do not block |

---

## Email indicators

| Type | Indicator |
|---|---|
| Sender (From / Reply-To) | `Lisa Beck <lisabeck@hr.lineatalent.xyz>` |
| Envelope / Return-Path | `010301a10b6d1feb-89530428-678e-4998-a38c-849eb49c88c9-000000@rsend.hr.lineatalent.xyz` |
| Subject | `Engineering opening — Linea` |
| Message-ID | `<010301a10b6d1feb-89530428-678e-4998-a38c-849eb49c88c9-000000@sa-east-1.amazonses.com>` |
| Sending pipeline | Resend ESP (DKIM `s=resend`) → Amazon SES `sa-east-1` |
| Lure string | Verification code `Ref-4h31zzxmmutk76` |
| Lure hook | "Your GitHub activity in blockchain repos stood out" |
| Open-tracking | 1×1 `display:none` image on `open.hr.lineatalent.xyz` |

---

## Host / certificate indicators

| Attribute | Value |
|---|---|
| Registrar | NAMECHEAP INC |
| Domain created | 2026-09-27 (WHOIS, urlscan `00:30:34 UTC`, CT all agree) |
| Nameservers | `dns1.registrar-servers.com`, `dns2.registrar-servers.com` |
| Tracking-subdomain cert | `open.hr.lineatalent.xyz` — issuer **Amazon ECDSA 256 M01**, Not Before 2026-09-27, Not After 2027-04-12 (crt.sh IDs 29635893668, 29601496893) |
| Landing-page cert | Issuer **"YE2"**, issued 2026-09-29, 3-month validity (urlscan) |
| Hosting ASN | AS206216 — ADVIN-AS, Advin Services LLC (San Jose, US) |

---

## MITRE ATT&CK (Enterprise) mapping

| Tactic | Technique | ID |
|---|---|---|
| Resource Development | Acquire Infrastructure: Domains | T1583.001 |
| Resource Development | Acquire Infrastructure: Web Services | T1583.006 |
| Resource Development | Establish Accounts: Email Accounts | T1585.002 |
| Reconnaissance | Gather Victim Identity Information | T1589 |
| Initial Access | Phishing: Spearphishing Link | T1566.002 |
| Reconnaissance | Phishing for Information: Spearphishing Link | T1598.003 |
| Defense Evasion / Social Engineering | Impersonation | T1656 |
| Execution (intended, not exercised) | User Execution: Malicious Link | T1204.001 |

---

## Pattern reference (non-attribution)

The TTP set is **consistent with** publicly documented recruiter-lure campaigns targeting cryptocurrency/blockchain professionals — reported as **"Operation Dream Job"** and **"Contagious Interview"**, associated in open reporting with DPRK-nexus activity (TraderTraitor/Lazarus cluster). This is a **pattern resemblance at the TTP level only**. **No named-actor attribution is asserted** for this campaign; see [Phase 05](../phase05-attribution-correlation/README.md).

---

*IOCs — PHI-2026-001 — Recruitment-Lure Phishing — Linea Lookalike.*
