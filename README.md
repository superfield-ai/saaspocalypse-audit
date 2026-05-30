# Saaspocalypse Now — Readiness Audit

A structured architectural readiness audit for vibe-coded business software.

Built for founders, managers, and teams who have assembled a working system
with AI assistance and want to understand what it can and cannot survive before
putting it in front of real users, real customers, or regulators.

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

## How to run it

### Option A — From your project directory

Install [Claude Code](https://claude.ai/code), navigate to your project, and
run:

```
/saaspocalypse-audit
```

### Option B — Pass a git URL

```
/saaspocalypse-audit https://github.com/your-org/your-repo
```

The audit clones the repo, runs the full analysis, and removes the clone when
finished. Works with public repos or any repo your git credentials can reach.

### Installing the skill

Copy `SKILL.md` and the `prompts/` directory into your Claude Code global
skills directory:

```bash
git clone https://github.com/superfield-ai/saaspocalypse-audit
cp -r saaspocalypse-audit ~/.claude/skills/saaspocalypse-audit
```

## What the audit will not do

- Ask you to fill out a form instead of reading your code
- Calibrate findings to your industry, company size, or stage
- Tell you what to fix — it tells you what you cannot survive
- Produce a passing grade

## Real-world incidents referenced

Each failing category is anchored to a real incident:

- **Architecture** — Parler (2021): 70TB scraped because the API design had no concept of access control on resources
- **Security Architecture** — Colonial Pipeline (2021): $4.4M ransom, 6-day fuel shutdown from a single VPN account with no second factor
- **Deployment** — GitLab (2017): 18 hours of downtime, 6 hours of permanent data loss from untested backups
- **Engineering Process** — SolarWinds (2020): 18,000 customers installed malware because the build pipeline had no integrity verification
- **Language & Frameworks** — Log4Shell (2021): 500M+ devices exposed by a transitive dependency most teams didn't know they had
- **Implementation Quality** — Knight Capital (2012): $440M lost in 45 minutes from dead code no one knew was still connected
- **Test Soundness** — Heartbleed (2014): 2 years in production, 17% of HTTPS sites exposed, caught by a 2-line fix no test had ever checked for
- **Observability** — Uber (2016): 57M records stolen, undetected for a year, discovered when the attacker asked for money
- **AI & LLM Architecture** — Bing Chat (2023): indirect prompt injection via external web pages; Samsung (2023): proprietary source code sent to an AI provider with no data agreement

---

Built by [Superfield](https://superfield.ai).
