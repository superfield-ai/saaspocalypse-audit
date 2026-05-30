# Saaspocalypse Now — Readiness Audit

A structured architectural readiness audit for vibe-coded business software.

Built for founders, managers, and teams who have assembled a working system
with AI assistance and want to understand what it can and cannot survive before
putting it in front of real users, real customers, or regulators.

---

## Quick start

Paste this into your coding agent:

```
Agent, install the saaspocalypse-audit skill. Follow the instructions at: https://raw.githubusercontent.com/superfield-ai/saaspocalypse-audit/main/agent-bootstrap.md
```

Then run the audit from your project directory:

```
/saaspocalypse-audit
```

Or audit a remote repo:

```
/saaspocalypse-audit https://github.com/your-org/your-repo
```

Requires [Claude Code](https://claude.ai/code).

---

## What this is

An audit that reads your actual source code and scores nine categories of
architectural readiness on an A–D scale. Each category includes:

- What the audit looks for (architectural signals, not line-by-line code)
- What A, B, C, and D look like
- A real-world incident — told in plain language — that shows what happens
  when this category is neglected
- Plain-language closing: **what this system cannot survive right now**

No perfect score exists. The goal is to understand the tradeoffs your system
is currently making, not to pass a checklist.

## The nine categories

| # | Category |
|---|----------|
| 1 | Architecture |
| 2 | Security Architecture |
| 3 | Deployment & Infrastructure |
| 4 | Engineering Process |
| 5 | Language, Framework & Tool Choices |
| 6 | Implementation Quality |
| 7 | Test Soundness |
| 8 | Observability & Operations |
| 9 | AI & LLM Architecture |

## What the audit will not do

- Ask you to fill out a form instead of reading your code
- Calibrate findings to your industry, company size, or stage
- Tell you what to fix — it tells you what you cannot survive
- Produce a passing grade

## Real-world incidents referenced

Each failing category is anchored to multiple real incidents, chosen for how clearly they reveal a specific design error — not just what went wrong, but what architectural decision was missing.

**Architecture**
- Parler (2021) — 70TB scraped; the API had no concept of per-resource access control
- Optus (2022) — 9.8M records retrieved by incrementing an integer in a URL; no authorization model on the API
- Facebook/Meta (2019) — 540M records exposed by partners; trust boundaries stopped at the API, not at the data

**Security Architecture**
- Colonial Pipeline (2021) — $4.4M ransom, 6-day fuel shutdown; single VPN credential, no MFA, no network segmentation
- Equifax (2017) — 147M Social Security numbers; unpatched framework + no interior network segmentation
- Uber (2022) — full internal access from one compromised account; hardcoded credentials in internal scripts

**Deployment & Infrastructure**
- GitLab (2017) — 18 hours down, 6 hours of permanent data loss; five backup mechanisms, none tested
- Capital One (2019) — 100M records; over-permissioned IAM role made a stolen credential a master key
- Cloudflare (2023) — source code accessed via a token that survived a known upstream breach; no rotation policy

**Engineering Process**
- SolarWinds (2020) — 18,000 customers installed malware; no integrity verification in the build pipeline
- Codecov (2021) — CI secrets exfiltrated from thousands of companies; remote script fetched with no hash check
- npm event-stream (2018) — malicious code in a 2M-download package; no vetting of maintainer transfers

**Language, Framework & Tool Choices**
- Equifax/Apache Struts (2017) — 147M records; known patch available 66 days before breach, no framework inventory
- left-pad (2016) — worldwide build failures from an 11-line package unpublished in a dispute
- Log4Shell (2021) — RCE in a transitive logging dependency present in hundreds of millions of applications

**Implementation Quality**
- Knight Capital (2012) — $440M in 45 minutes; dead code reactivated by a flag no one knew was still live
- CrowdStrike (2024) — 8.5M machines crashed simultaneously; config file applied with no validation and no staged rollout
- Boeing 737 MAX MCAS — 346 deaths; single-sensor input with no cross-validation, no operator alert, no known override

**Test Soundness**
- Heartbleed (2014) — 17% of HTTPS sites exposed; no test ever sent a heartbeat with a mismatched length field
- TSB Bank (2018) — £330M in losses; migration tested on clean data, never on real volume or real data variety
- Knight Capital (2012) — zombie code invisible to tests that only verified intended behavior

**Observability & Operations**
- Target (2013) — 40M cards stolen; FireEye fired correct alerts, no process existed to act on them
- Cloudflare (2019) — 27-minute global outage; monitoring detected the spike, deployment had no automatic rollback wired to it
- Uber (2016) — 57M records, undetected for a year; no behavioral baseline, no alert on credential misuse

**AI & LLM Architecture**
- Bing Chat (2023) — indirect prompt injection via web pages; no boundary between trusted instructions and untrusted content
- Samsung (2023) — proprietary source code sent to ChatGPT; no data classification layer, no DPA
- Air Canada chatbot (2024) — tribunal-enforced liability for a fabricated refund policy; no authority limits on what the AI could commit to
- Slack AI (2024) — private channel content exfiltrated via prompt injection; AI context crossed access control boundaries

---

Built by [Superfield](https://superfield.co).
