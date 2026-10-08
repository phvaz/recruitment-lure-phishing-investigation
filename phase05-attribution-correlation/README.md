# Phase 05 — Attribution & Pattern Correlation

## Objective

Place the operation in context: what technique set it uses, and what known pattern it fits. This phase is where the discipline of **separating fact, inference, attribution and intent** matters most, because the pull toward naming a culprit is strongest here and the evidence to support a name does not exist. The technique mapping uses **MITRE ATT&CK (Enterprise)**; the reasoning follows **ISO/IEC 27042**.

**Stated up front:** this investigation does **not** attribute the campaign to any named person, company or threat group. What follows is a **TTP-level** characterisation and a comparison to a publicly documented pattern. Those are different claims from "who did this", and the difference is kept explicit throughout.

---

## 5.1 — The lookalike logic, restated as technique

The operation's core deception is a domain that *reads* as Linea's careers site without *being* Linea's domain:

| Element | Genuine | This operation |
|---|---|---|
| Brand domain | `linea.build` | `lineatalent.xyz` — reuses "linea", adds "talent", cheap TLD |
| Trust signals | Established 2023, Cloudflare | Valid SPF/DKIM/DMARC + HTTPS on an 8-day-old domain |
| Channel | — | Unsolicited recruiter email via AWS SES/Resend |
| Hook | — | Target's **public GitHub / blockchain activity** |

The attacker does not need to break any security control. They **manufacture** every trust signal a non-expert checks (sender authenticated ✓, padlock ✓, brand-styled page ✓) on infrastructure they fully own. The only reliable tell is the one automated systems and casual inspection miss: the **domain is not the brand's real domain, and it is days old.**

---

## 5.2 — MITRE ATT&CK mapping

| Tactic | Technique | ID | Evidence in this case |
|---|---|---|---|
| Resource Development | Acquire Infrastructure: Domains | T1583.001 | `lineatalent.xyz` registered 2026-09-27 (Phase 03) |
| Resource Development | Acquire Infrastructure: Web Services | T1583.006 | AWS SES + Resend; ADVIN-AS hosting (Phases 02–04) |
| Resource Development | Establish Accounts: Email Accounts | T1585.002 | `lisabeck@hr.lineatalent.xyz` sending persona (Phase 02) |
| Reconnaissance | Gather Victim Identity Information | T1589 | Targeting keyed to the victim's **public GitHub/blockchain** activity (Phase 01) |
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Tailored recruitment email with link to `lineatalent.xyz` |
| Reconnaissance / Collection | Phishing for Information: Spearphishing Link | T1598.003 | "Reply with résumé/GitHub" + "verification code" funnel |
| Defense Evasion / Social Eng. | Impersonation | T1656 | Clone of "Linea \| Careers"; persona "Talent, Linea" |
| (Intended) Execution | User Execution: Malicious Link | T1204.001 | The CTA behind which the next stage would be delivered — **not exercised** |

Open-tracking (the hidden pixel, Phase 02) maps to campaign **measurement** rather than a single ATT&CK technique; it is noted as an operational capability indicating organisation.

---

## 5.3 — The pattern it fits

| Observed (facts, from earlier phases) | |
|---|---|
| Unsolicited **recruiter** approach | Persona "Lisa Beck, Talent, Linea" |
| Target selected for a **public blockchain / crypto / security profile** | "Your GitHub activity in blockchain repos stood out" |
| **Impersonation of a real crypto company** via lookalike domain | `lineatalent.xyz` vs `linea.build` |
| A **two-step funnel**: friendly email → attacker-controlled page / "assignment" | Link + "verification code" + "Accept invitation" |
| **Well-resourced** delivery (cloud ESP, tracking, HTTPS, brand clone) | Phases 02–04 |

> **Interpretation (pattern, not attribution).** This combination — fake job offers aimed at engineers and researchers in the cryptocurrency/blockchain space, using impersonated companies and a staged funnel — is a **publicly documented social-engineering pattern**. It has been reported under names such as **"Operation Dream Job"** and **"Contagious Interview"**, and is associated in open reporting with **DPRK-nexus activity (e.g. the TraderTraitor/Lazarus cluster)**. The present case **matches that pattern at the level of tactics and techniques.**

**What this does and does not mean:**

| Claim | Status | Confidence |
|---|---|---|
| The campaign uses a recruiter-lure TTP targeting crypto/blockchain professionals | **Fact** (from this case's evidence) | High |
| That TTP resembles the publicly documented "Dream Job"/"Contagious Interview" pattern | **Inference** (pattern comparison) | Medium–High |
| The campaign *is* the work of Lazarus/TraderTraitor or any other named group | **Not established** — the evidence here (one email, shared public infrastructure, a common TTP) **cannot** support a named-actor attribution | — |
| "Lisa Beck" is a real person | **Not established** — this is a sending **persona/account**, not an identified individual | — |

The TTP is widely enough used — including by financially-motivated criminals who are not state-linked — that resemblance to a famous campaign is **not** proof of the same author. Naming an actor would require evidence this investigation does not and cannot have from passive OSINT on a single message: infrastructure reuse tied to prior confirmed operations, malware samples, or privileged reporting.

---

## 5.4 — Intent (kept separate from attribution)

| Observed | Inferred intent | Confidence |
|---|---|---|
| Recruitment pretext + attacker-controlled "careers" page + verification-code funnel | To move the target off email and into a controlled environment where the next stage (credential capture, wallet connection, or a malicious "coding assignment"/download) can be delivered | Medium–High |
| Open-tracking beacon | To identify live, engaged targets for prioritised follow-up | High |
| Targeting via public blockchain profile | To reach people with access to crypto assets, keys, or developer environments worth compromising | Medium |

Intent is reasoned from the operation's structure; it is an **inference about purpose**, held separately from both the facts (what the evidence shows) and attribution (who did it, which remains unestablished).

---

## Phase 05 findings summary

| # | Finding | Type | Confidence |
|---|---|---|---|
| 1 | Operation maps cleanly onto ATT&CK resource-development, phishing and impersonation techniques | Fact | High |
| 2 | It is a recruiter-lure targeting a crypto/blockchain professional via a lookalike of a real company | Fact | High |
| 3 | The pattern **resembles** the documented "Dream Job"/"Contagious Interview" TTP (DPRK-nexus in open reporting) | Inference | Medium–High |
| 4 | **No named-actor attribution is made**; "Lisa Beck" is a persona, not an identified person | Boundary | — |
| 5 | Inferred intent: funnel the target into a controlled next stage for credential/asset/developer-environment compromise | Inference | Medium–High |

---

*Phase 05 — PHI-2026-001.*

**Next:** [Phase 06 — Report & Recommendations](../phase06-report-recommendations/README.md)
