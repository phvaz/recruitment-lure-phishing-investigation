# Phase 02 — Email-Header Analysis

## Objective

Read the raw message headers to establish three things the body cannot tell us: **whether the message was authenticated, where it was really sent from, and what machinery sent it.** Headers written by transit and receiving infrastructure (Google, Amazon) are far harder to forge than a `From:` line, which makes them the load-bearing evidence in any phishing case. The authentication mechanics are governed by **RFC 7208 (SPF)**, **RFC 6376 (DKIM)** and **RFC 7489 (DMARC)**; the message structure by **RFC 5322**.

Source: the preserved [`.eml`](../evidence/Engineering_opening_Linea.eml).

---

## 2.1 — Authentication results

The receiving server (Google) recorded this verdict:

```
Authentication-Results: mx.google.com;
   dkim=pass header.i=@hr.lineatalent.xyz header.s=resend header.b="DRSBvth/";
   dkim=pass header.i=@amazonses.com header.s=n3kzsla...rsk header.b="OrtxB8EV";
   spf=pass (google.com: domain of 010301...@rsend.hr.lineatalent.xyz
        designates 23.249.215.51 as permitted sender)
        smtp.mailfrom=010301...@rsend.hr.lineatalent.xyz;
   dmarc=pass (p=NONE sp=NONE dis=NONE) header.from=lineatalent.xyz
```

| Mechanism | Result | What it actually proves |
|---|---|---|
| **SPF** | `pass` | IP `23.249.215.51` is authorised to send for `rsend.hr.lineatalent.xyz`. That DNS record was published **by whoever controls `lineatalent.xyz`** — i.e. the attacker. |
| **DKIM** | `pass` (×2) | The message was cryptographically signed by `hr.lineatalent.xyz` (selector `resend`) **and** by `amazonses.com`. Both keys belong to the sending side. |
| **DMARC** | `pass` (`p=NONE`) | The authenticated domains align with the `From:` domain. Policy is `p=NONE` — "monitor only, do nothing on failure". |

### The central teaching point of this case

> **SPF, DKIM and DMARC all passing is *not* evidence that the message is legitimate.** These mechanisms answer one question only: *"did the message really come from the domain it claims, and was it unaltered in transit?"* They say **nothing** about whether that domain is trustworthy. An attacker who registers their own lookalike domain and publishes correct SPF/DKIM/DMARC records for it will pass all three — precisely because they control the domain. Authentication passing here is a direct consequence of the attacker **owning `lineatalent.xyz`**, not a sign of honesty.

| Observed | Interpretation | Confidence |
|---|---|---|
| All three authentication checks pass for `(hr.)lineatalent.xyz` | The sender legitimately controls the lookalike domain and configured it properly. Authentication confirms *control of the domain*, not *legitimacy of the sender*. | High |
| DMARC policy is `p=NONE` | The domain owner set no enforcement. Common for throwaway attack domains — the goal is deliverability, not protecting a brand from spoofing. | Medium–High |

---

## 2.2 — Real origin: Amazon SES, São Paulo region

The transit header — written by Google's receiving server, not the sender — records where the message actually entered Google:

```
Received: from d215-51.smtp-out.sa-east-1.amazonses.com
          (d215-51.smtp-out.sa-east-1.amazonses.com. [23.249.215.51])
          by mx.google.com with ESMTPS id ...
          for <phvaz2001@gmail.com>
          Mon, 05 Oct 2026 02:37:44 -0700 (PDT)
```

Four independent header elements converge on the same origin:

| Signal | Value |
|---|---|
| `Received` from-host | `d215-51.smtp-out.**sa-east-1**.amazonses.com` |
| `Message-ID` domain | `@**sa-east-1**.amazonses.com` |
| `X-SES-Outgoing` | `2026.10.05-23.249.215.51` |
| `Feedback-ID` | `…:1.**sa-east-1**.…:AmazonSES` |

| Observed | Interpretation | Confidence |
|---|---|---|
| The message transited **Amazon SES in the `sa-east-1` (São Paulo) region** | The operator used a legitimate, high-reputation cloud mail service to achieve deliverability into Gmail. The reverse-DNS host name and the `X-SES-Outgoing` / `Feedback-ID` headers are set by **Amazon**, not the sender, so this origin is reliable. | High |
| Region is `sa-east-1` (Brazil) | Consistent with either a Brazil-based operator or simply the region chosen to best reach a Brazilian target. Region choice alone is **not** attribution — it is an infrastructure fact, not a location of a person. | Low (as a locating signal) |

---

## 2.3 — The ESP layer: Resend on top of SES

Two DKIM signatures, not one, tell us how the pipeline was built:

```
DKIM-Signature: ... s=resend;  d=hr.lineatalent.xyz; ...
DKIM-Signature: ... s=n3kzsla...rsk; d=amazonses.com; ...
Return-Path: <010301a10b6d1feb-...@rsend.hr.lineatalent.xyz>
```

- The `s=resend` selector and the `rsend.hr.lineatalent.xyz` Return-Path are the signature of **Resend**, a developer email-API/ESP that delivers through Amazon SES.
- The second DKIM signature from `amazonses.com` is SES's own.

| Observed | Interpretation | Confidence |
|---|---|---|
| Message sent via **Resend → Amazon SES**, with a dedicated `rsend.hr.*` sending subdomain and a signing subdomain `hr.lineatalent.xyz` | The operator stood up a **proper transactional-email pipeline**: a developer ESP, a verified sending domain, DKIM keys. This is deliberate engineering effort, not a hijacked account or a spam cannon. It signals an **organised operation**. | High |

---

## 2.4 — Open-tracking: the hidden pixel

The HTML body ends with a hidden image:

```html
<img alt="" src="https://open.hr.lineatalent.xyz/CI0/010301a10b6d1feb-...=258"
     style="display: none; width: 1px; height: 1px;">
```

| Observed | Interpretation | Confidence |
|---|---|---|
| A 1×1, `display:none` image hosted on the attacker's own subdomain `open.hr.lineatalent.xyz`, carrying a per-recipient token | An **open-tracking beacon**. If loaded, it tells the operator the exact moment this specific recipient opened the mail — confirming a live, engaged human and enabling follow-up. The per-message token individualises the target. | High |
| The tracking subdomain is `open.hr.*`, matching the ESP's tracking convention | Consistent with Resend/SES open-tracking being switched on — again, infrastructure configured for **campaign measurement**. | High |

**Operational handling:** because the beacon was identified in the raw source, the HTML was never rendered in a live client, so the open event was never sent. This is why Phase 00's "examine from raw source" rule mattered in practice.

---

## 2.5 — Content tells

Minor but consistent with the triage read:

| Observed | Interpretation | Confidence |
|---|---|---|
| Greeting "Hi **phvaz**" = the email local-part | Automated personalisation from the address; the operator does not know the recipient's actual name. | High |
| Subject uses a correctly encoded em-dash (`=?UTF-8?Q?…=E2=80=94…?=`) | Competent, templated composition — not a sign of a low-effort actor. | Low |
| `Reply-To` = `From` (both `lisabeck@hr.lineatalent.xyz`) | Replies route back to the attacker-controlled domain, opening a human conversation channel. | Medium |

---

## Phase 02 findings summary

| # | Finding | Confidence |
|---|---|---|
| 1 | The message is cryptographically authenticated (SPF/DKIM/DMARC pass) **for the attacker-controlled lookalike domain** — authentication proves domain control, not legitimacy | High |
| 2 | It really originated from **Amazon SES, `sa-east-1`** (verified via Amazon-set headers) | High |
| 3 | It was sent through a **Resend → SES** transactional pipeline with a purpose-built sending subdomain — an organised operation | High |
| 4 | It carries a **per-recipient open-tracking beacon** on `open.hr.lineatalent.xyz` | High |
| 5 | Personalisation is **automated** (local-part greeting); the operator does not know the target personally | High |

The headers establish that this is a deliberately engineered campaign using reputable infrastructure to reach the inbox. **What** the campaign points at — the domain and the page — is the subject of Phases 03 and 04.

---

*Phase 02 — PHI-2026-001.*

**Next:** [Phase 03 — Domain & Infrastructure](../phase03-domain-infrastructure/README.md)
