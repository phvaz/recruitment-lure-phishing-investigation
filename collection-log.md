# Collection Log — Provenance Record (PHI-2026-001)

This log records **how and when each data point was obtained**, so that any third party can reproduce the collection and arrive at the same raw data.

In a media-based forensic investigation this role is served by a chain-of-custody document. Open-source intelligence has no seized artifact to take custody of — the equivalent discipline is **provenance**: recording the source queried, the exact query issued, the timestamp of collection, and whether the source is authoritative or a third-party relay. Without this, a finding cannot be independently verified, and OSINT data is perishable (WHOIS entries, DNS records and certificate logs all change over time).

The one artifact that *is* held in custody here is the raw message itself (`Engineering_opening_Linea.eml`), preserved before any analysis and fixed by hash.

---

## Evidence register

| ID | Item | Source | Acquired (UTC) | Integrity |
|---|---|---|---|---|
| E1 | `Engineering_opening_Linea.eml` — raw message with full headers | Exported from the recipient mailbox (`phvaz2001@gmail.com`) via "Download message" | 2026-10-05 | SHA-256 fixed at staging (see manifest below) |
| E2 | `01_email_gmail_view.png` — rendered message as received | Screenshot, recipient mailbox web UI | 2026-10-05 | SHA-256 |
| E3 | `02_whois_lineatalent_xyz.png` — WHOIS of the lookalike domain | who.is | 2026-10-06 | SHA-256 |
| E4 | `03_whois_linea_build.png` — WHOIS of the genuine Linea domain | who.is | 2026-10-06 | SHA-256 |
| E5 | `04_urlscan_lineatalent_xyz.png` — sandboxed page scan | urlscan.io | 2026-10-05 (scan date shown in report) | SHA-256 |
| E6 | `05_crtsh_lineatalent_xyz.png` — Certificate Transparency records | crt.sh | 2026-10-08 | SHA-256 |

> Dates reflect when the examiner captured each item. Where a third-party service embeds its own timestamp (urlscan's scan time, crt.sh's "Logged At", WHOIS "Updated On"), that service-side value is treated as authoritative for the event it records and is cited in the relevant phase.

---

## Collection sessions

| # | Date (UTC) | Source | Query / action | Data obtained | Phase |
|---|---|---|---|---|---|
| 1 | 2026-10-05 | Recipient mailbox | "Download message" → raw `.eml`; screenshot of rendered view | Full message headers + body; visual rendering | 01, 02 |
| 2 | 2026-10-05 | urlscan.io | Existing/submitted scan of `http://lineatalent.xyz/` | IP, ASN, geolocation, TLS issuer, page title, request graph, verdicts | 04 |
| 3 | 2026-10-06 | who.is | WHOIS lookup `lineatalent.xyz` | Registrar, creation/expiry/update dates, nameservers, domain status | 03 |
| 4 | 2026-10-06 | who.is | WHOIS lookup `linea.build` | Registrar, creation/expiry/update dates, nameservers, status (baseline for comparison) | 03 |
| 5 | 2026-10-08 | crt.sh | Identity search `lineatalent.xyz` (ILIKE) | Certificate Transparency records for `open.hr.lineatalent.xyz` — issuer, validity window | 03 |

---

## Source authority classification

A distinction that materially affects how much weight each finding can carry:

| Source | Authority | Note |
|---|---|---|
| Preserved `.eml` (E1) | **Primary artifact** | The message itself, captured before analysis. The `Received`, `Authentication-Results`, DKIM and SES headers are written by the *receiving* and *transit* infrastructure (Google, Amazon), not freely forgeable by the sender — which is what makes them reliable. |
| crt.sh | **Authoritative (relay)** | Certificate Transparency logs are cryptographically verifiable and append-only by design; crt.sh relays them but the underlying data cannot be silently altered. |
| urlscan.io | **Analysis platform / relay** | Performs the page retrieval in a sandbox on the analyst's behalf and records what it observed. Authoritative for *what the sandbox saw at scan time*; page content can change after the fact. |
| who.is | **Third-party relay** | Presents WHOIS/RDAP data sourced from the registry/registrar. The *underlying* registry records are authoritative; who.is is a convenience layer. For a formal product, queries should be re-run directly against the registrar's RDAP endpoint. |
| Google Safe Browsing (via urlscan) | **Reputation signal** | A classification verdict, not primary evidence. A "No classification" result means *not yet catalogued*, not *safe*. |

---

## Reproducibility notes

- **Data is time-bound.** All lookups reflect the state of public infrastructure on 2026-10-05 → 2026-10-08. WHOIS, DNS and hosting may change; the domain may be taken down. A re-run later is expected to differ (or return nothing) while the preserved `.eml` and the screenshots remain fixed.
- **No active engagement.** No port scans, service probes, or connection attempts were made against `lineatalent.xyz`, its subdomains, or the hosting IP. The landing page was observed only through urlscan.io's sandbox — never from the examiner's own browser or environment.
- **The link and sender were never engaged.** The URL was not opened directly, no reply was sent, no "verification code" was used, and nothing from the sender was downloaded or executed.

---

## SHA-256 manifest

Generated at evidence staging. Re-run `sha256sum *` inside `evidence/` to verify.

```
a4e93da7ab97f44b3c4998f42f1cbe1ff69fff534de751255b90bcc1d17fcc00  01_email_gmail_view.png
83115b6ca4d9dd467405d89aa3064550dffb9b6424fc1f12b1a3cbe54082b276  02_whois_lineatalent_xyz.png
7f9a03cb7a9168015efba0c060304d97a15b0b33158dc8d2b8b0d6782d2375f9  03_whois_linea_build.png
9822e0d935ade6f014da22fc2a2586e7e87dfe0a4d572e8c2bae1537e244ee18  04_urlscan_lineatalent_xyz.png
0fbec2055e2e53a3f94b34b4555b8f41dcf772a38a911eb88e809b46d78c633d  05_crtsh_lineatalent_xyz.png
6c31f23cbb398ba1db3435cfa26ccd967f0b5d606b9245ed0abe43e79730fa8e  Engineering_opening_Linea.eml
```

---

*Collection Log — PHI-2026-001 — Recruitment-Lure Phishing — Linea Lookalike.*
