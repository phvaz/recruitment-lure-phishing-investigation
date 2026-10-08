# PHI-2026-001 — Recruitment-Lure Phishing Targeting a Blockchain Researcher

**Threat-intelligence investigation of a live phishing lure impersonating Linea (`lineatalent.xyz`), received and analysed first-hand.**

| Field | Value |
|---|---|
| Case reference | PHI-2026-001 |
| Case name | Recruitment-Lure Phishing — Linea Lookalike (`lineatalent.xyz`) |
| Examiner | Paulo Vaz |
| Investigation type | Passive OSINT / email-header forensics / threat intelligence |
| Target of the lure | The examiner's own personal inbox (`phvaz2001@gmail.com`) |
| Lure received | 2026-10-05 09:37:43 UTC |
| Investigation window | 2026-10-05 → 2026-10-08 |
| Disposition | Reported to the mailbox provider; not engaged. Investigation conducted on the preserved `.eml` and public data only. |

---

## What this case is

On 2026-10-05 the examiner received an unsolicited "engineering opening" email from *Lisa Beck — Talent, Linea*. The hook was specific: *"Your GitHub activity in blockchain repos stood out."* The examiner had, days earlier, published blockchain-investigation work publicly. The message pointed to `lineatalent.xyz` — not Linea's real domain, `linea.build`.

Rather than reply or open the link, the examiner preserved the raw message and investigated it as a threat-intelligence case. This repository documents that investigation end to end: how the message was authenticated, where it was really sent from, who registered the lookalike domain and when, what the landing page was, and what pattern the operation fits.

**This is a defensive, first-hand case.** Every finding here was produced by the examiner from material the examiner personally received, using only passive public sources. No attacker infrastructure was contacted, no link was opened outside a sandbox, and no file from the sender was executed.

---

## Headline findings

1. **The lookalike domain was registered 8 days before the email** — `lineatalent.xyz` on 2026-09-27; Linea's genuine domain `linea.build` dates to 2023-03-14. A ~3.5-year age gap.
2. **The email authenticated cleanly (SPF/DKIM/DMARC all `pass`) — because the attacker controlled the sending domain.** Passing authentication is *not* a signal of legitimacy; it only proves the sender owns the domain they sent from. This is the central teaching point of the case.
3. **The operation runs on professional mass-mail infrastructure** — Amazon SES in the `sa-east-1` (São Paulo) region, fronted by the Resend ESP, with hidden open-tracking. This is an organised campaign, not casual spam.
4. **The infrastructure is compartmentalised** — email and tracking on AWS/SES; the phishing landing page on a separate host (`193.160.32.10`, AS206216 ADVIN-AS, San Jose) behind a different certificate authority.
5. **Reputation engines had not yet caught it** — at analysis time both urlscan.io and Google Safe Browsing returned *No classification*, because the domain was only days old. Freshness is the attacker's window.
6. **The targeting pattern is consistent with recruiter-lure social engineering aimed at people with public crypto/blockchain profiles** — a TTP associated with, among others, DPRK-nexus "Dream Job"/TraderTraitor activity. This is a **pattern** observation at the TTP level; **no actor attribution is asserted.**

---

## Repository structure

```
PHI-2026-001/
├── README.md                          ← you are here
├── collection-log.md                  ← OSINT provenance (the chain-of-custody equivalent)
├── evidence/                          ← preserved raw evidence + SHA-256 manifest
│   ├── Engineering_opening_Linea.eml
│   ├── 01_email_gmail_view.png
│   ├── 02_whois_lineatalent_xyz.png
│   ├── 03_whois_linea_build.png
│   ├── 04_urlscan_lineatalent_xyz.png
│   └── 05_crtsh_lineatalent_xyz.png
├── indicators/
│   └── iocs.md                        ← indicators of compromise + MITRE ATT&CK mapping
├── phase00-scope-methodology/         ← scope, rules of engagement, method
├── phase01-initial-triage/            ← what arrived and why it was suspicious
├── phase02-header-analysis/           ← SPF/DKIM/DMARC, SES, Resend, tracking pixel
├── phase03-domain-infrastructure/     ← WHOIS comparison, crt.sh, IP/ASN
├── phase04-page-analysis/             ← safe (sandboxed) landing-page analysis
├── phase05-attribution-correlation/   ← lookalike logic, TTP pattern, non-attribution
├── phase06-report-recommendations/    ← consolidated findings, IOCs, recommendations
└── reports/
    └── PHI-2026-001-phishing-investigation-report.pdf
```

Read the phases in order — each builds on the previous one. The PDF in `reports/` is the consolidated formal report.

---

## Method and discipline

- **Passive OSINT only.** All data came from public lookup services (WHOIS/RDAP, Certificate Transparency, urlscan.io, Safe Browsing) and from the preserved message itself. No port scans, no connection attempts against attacker infrastructure, no interaction with the sender.
- **Provenance over custody.** OSINT has no seized artifact to take custody of; the equivalent discipline is recording *what source was queried, the exact query, the time (UTC), and whether the source is authoritative or a relay*. See [`collection-log.md`](collection-log.md).
- **Calibrated language.** Findings are separated into **Observed → Interpretation → Confidence**, and the report keeps **fact, inference, attribution, and intent** distinct. Where a conclusion is probabilistic it says "consistent with", not "proves". An email persona ("Lisa Beck") is treated as an account, never as an identified person.

---

## Standards referenced

| Standard | Role in this case |
|---|---|
| ISO/IEC 27037 | Identification, collection and preservation of digital evidence (adapted to OSINT provenance) |
| ISO/IEC 27041 | Assurance that the investigative methods are suitable and the results can be relied upon |
| ISO/IEC 27042 | Analysis and interpretation of digital evidence |
| ISO/IEC 27043 | Incident-investigation principles and process |
| NIST SP 800-61r2 | Computer security incident handling (detection, analysis, reporting) |
| RFC 5322 / 7208 / 6376 / 7489 | Internet Message Format / SPF / DKIM / DMARC — the email-authentication basis |
| RFC 3227 | Guidelines for evidence collection and archiving (provenance discipline) |
| LGPD (Lei 13.709/2018) | The targeted data subject's own personal data governs handling of the message |
| MITRE ATT&CK (Enterprise) | Technique mapping for the observed behaviour |

---

*PHI-2026-001 — Recruitment-Lure Phishing — Linea Lookalike. Investigation and documentation by Paulo Vaz.*
