# Phase 00 — Scope, Methodology and Rules of Engagement

## Why this phase exists

Before touching the evidence, the investigation fixes **what it is allowed to do, what question it is answering, and how it will record its work**. In a phishing case that discipline is not bureaucratic overhead — the thing under investigation is live, hostile infrastructure built to be interacted with. A single careless click (opening the link from a real browser, replying to "confirm", submitting the verification code) would both compromise the examiner and contaminate the evidence. The rules of engagement below are the safety boundary that made the rest of the case possible.

This framing follows **ISO/IEC 27043**, which treats defining scope, authorisation and process as the first stage of any sound investigation, and **ISO/IEC 27041**, which requires that the methods chosen be demonstrably suitable for the question asked.

---

## The question

> A targeted recruitment email impersonating Linea was received on 2026-10-05. **Is it legitimate, and if not, what can be established about the operation behind it — its infrastructure, its freshness, and the pattern it fits — using only passive, public sources?**

The investigation is **not** trying to unmask a named individual or company behind the campaign. OSINT of this kind can characterise *infrastructure and behaviour*; it cannot, on this evidence, identify a person. That boundary is set here deliberately and respected through to the attribution phase.

---

## Scope

**In scope**
- The preserved raw message (`.eml`) — headers and body.
- The sending and tracking infrastructure as declared in the headers (SES region, ESP, tracking subdomain).
- The lookalike domain `lineatalent.xyz` and, for comparison, the genuine `linea.build`: registration, nameservers, certificates, hosting.
- The landing page, observed **only** through a third-party sandbox.
- The behavioural pattern the operation fits, at the level of tactics and techniques.

**Out of scope**
- Any interaction with the sender, the link, or the "verification code".
- Any active scanning or probing of attacker infrastructure.
- Identification of a natural person or legal entity behind the campaign.
- Remediation of third parties (reporting to the abused providers is noted as a recommendation, not performed as part of the forensic analysis).

---

## Rules of engagement (safety boundary)

These are non-negotiable and apply to every subsequent phase:

1. **Passive only.** Public lookup services and the preserved message. No port scans, no service probes, no direct connections to `lineatalent.xyz` or its subdomains.
2. **The landing page is observed through a sandbox, never directly.** urlscan.io retrieves the page in its own isolated environment; the examiner's browser and network never touch it.
3. **No engagement with the sender.** No reply, no forwarding to the sender, no use of the verification code, no download or execution of anything the sender offers.
4. **Treat the sender's content as hostile until proven otherwise** — including any file or link, and including the message body's own embedded resources (the tracking pixel was identified in the raw source and never loaded).
5. **Preserve before analyse.** The `.eml` is exported and hashed before any interpretation begins; all analysis is performed on the preserved copy.

> **Note on the tracking pixel.** The message contains a hidden 1×1 open-tracking image hosted on the attacker's own subdomain. Rendering the HTML in a normal mail client would silently fetch it and signal "this human opened the mail" back to the operator. The message was therefore examined from its raw source, where the pixel is inert text, not a loaded resource. This is treated in detail in [Phase 02](../phase02-header-analysis/README.md).

---

## Method

The investigation proceeds in ordered phases, each building on the last:

| Phase | Focus | Primary standard |
|---|---|---|
| 00 | Scope, rules of engagement, method *(this document)* | ISO/IEC 27043, 27041 |
| 01 | Initial triage — what arrived, why it was suspicious | ISO/IEC 27037 |
| 02 | Email-header analysis — authentication, origin, tracking | RFC 5322 / 7208 / 6376 / 7489 |
| 03 | Domain & infrastructure — WHOIS comparison, CT logs, IP/ASN | ISO/IEC 27042 |
| 04 | Landing-page analysis — sandboxed observation | ISO/IEC 27042 |
| 05 | Attribution & pattern correlation — lookalike logic, TTP mapping | MITRE ATT&CK; ISO/IEC 27042 |
| 06 | Report, IOCs, recommendations | NIST SP 800-61r2 |

**Recording discipline.** Because OSINT data is perishable and has no seized artifact, every collection action is logged with its source, exact query, UTC timestamp, and source-authority classification (authoritative vs relay). This provenance record — the OSINT analogue of chain of custody — is maintained in [`collection-log.md`](../collection-log.md).

**Language discipline.** Every finding is expressed as **Observed → Interpretation → Confidence**. The report keeps **fact** (what the evidence directly shows), **inference** (what can be reasoned from it), **attribution** (who is responsible) and **intent** (what they meant to achieve) as separate claims with separate confidence. A sending persona is an *account*, not an identified individual.

---

## A note on authorisation and the data subject

The message was addressed to the examiner's own inbox; the examiner is both the investigator and the targeted data subject. There is therefore no third-party authorisation question for the analysis itself. The examiner's own personal data in the message (name fragment, email address, inferred professional activity) is handled under **LGPD** principles — minimised in this public documentation and never republished beyond what is necessary to explain the finding.

---

*Phase 00 — PHI-2026-001.*

**Next:** [Phase 01 — Initial Triage](../phase01-initial-triage/README.md)
