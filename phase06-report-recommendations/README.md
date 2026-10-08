# Phase 06 — Report & Recommendations

## Objective

Consolidate the investigation into a decision-useful summary: what happened, what was established (and to what confidence), the indicators others can act on, and what should be done. Structure follows **NIST SP 800-61r2** (incident analysis and reporting). The formal, standalone report is in [`reports/PHI-2026-001-phishing-investigation-report.pdf`](../reports/PHI-2026-001-phishing-investigation-report.pdf).

---

## 6.1 — Executive summary

On 2026-10-05 a targeted recruitment email impersonating the blockchain company **Linea** was received. It offered an engineering role and drove the recipient toward **`lineatalent.xyz`** — a lookalike of Linea's real domain, `linea.build`, registered just **8 days earlier**. Passive analysis established that the message was sent through **Amazon SES (São Paulo) via the Resend ESP**, carried a **hidden open-tracking beacon**, and passed **SPF/DKIM/DMARC** — not because it was legitimate, but because the operator **controlled the lookalike domain**. The linked page was a **brand-cloned "Linea | Careers"** site on **separate hosting**, not yet flagged by reputation engines because of its newness. The operation's tactics are **consistent with recruiter-lure social engineering targeting crypto/blockchain professionals** — a documented pattern — but the evidence **does not** support attribution to any named actor.

The recipient did not engage: no reply, no link, no verification code, no download. The message was preserved and reported.

---

## 6.2 — Consolidated findings

| # | Finding | Type | Confidence |
|---|---|---|---|
| 1 | The email impersonates Linea using the lookalike domain `lineatalent.xyz` (real: `linea.build`) | Fact | High |
| 2 | `lineatalent.xyz` was registered 2026-09-27 — 8 days before the lure; `linea.build` dates to 2023 | Fact | High |
| 3 | SPF, DKIM and DMARC all pass **because the attacker controls the sending domain** — not a legitimacy signal | Fact | High |
| 4 | Real origin: Amazon SES `sa-east-1` via the Resend ESP — an organised, engineered pipeline | Fact | High |
| 5 | The message carries a per-recipient open-tracking beacon on `open.hr.lineatalent.xyz` (AWS-hosted; CT-confirmed) | Fact | High |
| 6 | The landing page is a brand-cloned "Linea | Careers" site on AS206216 (San Jose) — separate from the email leg | Fact | High |
| 7 | Reputation engines (urlscan, Google Safe Browsing) had not classified the domain — detection lag, not safety | Fact | High |
| 8 | Targeting was keyed to the recipient's public blockchain/GitHub activity | Inference | Medium–High |
| 9 | The TTP resembles the documented "Dream Job"/"Contagious Interview" recruiter-lure pattern | Inference | Medium–High |
| 10 | No named-actor attribution is possible from this evidence; "Lisa Beck" is a persona | Boundary | — |

---

## 6.3 — Indicators of compromise

Full machine-readable detail in [`indicators/iocs.md`](../indicators/iocs.md). Summary:

| Type | Indicator |
|---|---|
| Domain | `lineatalent.xyz` |
| Subdomain (sending/signing) | `hr.lineatalent.xyz`, `rsend.hr.lineatalent.xyz` |
| Subdomain (tracking) | `open.hr.lineatalent.xyz` |
| Sender address | `lisabeck@hr.lineatalent.xyz` |
| Sending IP (SES) | `23.249.215.51` |
| Hosting IP (page) | `193.160.32.10` (AS206216, ADVIN-AS) |
| URL | `https://lineatalent.xyz/` |
| Lure artifact | Verification code `Ref-4h31zzxmmutk76` |

> **Caveat:** the SES sending IP and AWS infrastructure are **shared, legitimate services**; they must **not** be blocklisted. Only the domain, subdomains, sender address, page IP and URL are safe to action.

---

## 6.4 — Recommendations

**For the individual target (done / to do)**
- ✅ Do not reply, click, or use the verification code. *(done)*
- ✅ Preserve the raw message and report it to the mailbox provider. *(done)*
- Report the domain to Namecheap abuse and to Linea's real security contact so the brand owner is aware.
- Submit the URL/domain to Google Safe Browsing and other reputation feeds to shorten the detection gap for the next target.

**For defenders / organisations**
- Treat **domain age + lookalike similarity** as a first-class signal. SPF/DKIM/DMARC passing is necessary but **not sufficient** — alert on newly-registered domains that are close matches to brands your staff interact with.
- Flag unsolicited recruiter emails that reference an employee's **public code or research** and link to a non-canonical domain.
- Educate staff that **a padlock and a clean "authenticated" banner mean nothing** on an attacker-owned domain. The check that works is: *is this the brand's real, established domain?*

**For the discipline (lessons this case teaches)**
- Examine suspected phishing **from raw source**, never by rendering it — rendering loads trackers and confirms you as a live target.
- Observe hostile pages **only through a sandbox**.
- Keep **authentication ≠ legitimacy** front of mind; it is the single most load-bearing misconception this case corrects.

---

## 6.5 — Limitations

- **Single message.** Findings describe one lure and its infrastructure; the full campaign's scale is unknown from this evidence.
- **Passive only.** No active enumeration; the landing page's post-CTA behaviour was characterised but not exercised.
- **Perishable data.** WHOIS/DNS/hosting reflect 2026-10-05→08 and may since have changed or been taken down; the preserved `.eml` and screenshots are the fixed record.
- **No attribution.** By design and by evidence, the operator is not named.

---

*Phase 06 — PHI-2026-001.*

**Report:** [`reports/PHI-2026-001-phishing-investigation-report.pdf`](../reports/PHI-2026-001-phishing-investigation-report.pdf) · **Back to** [README](../README.md)
