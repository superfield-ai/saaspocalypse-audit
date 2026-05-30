# Category 2: Security Architecture

**What this assesses:** Whether security is a structural property of the system
or an afterthought applied to a system that was designed without it. A system
can have login screens and still have no security architecture.

---

## What to look for in the scan

- Is there a threat model, security doc, or any document that names adversaries
  and what they might do?
- Is authentication and authorization handled in a single, consistent layer —
  or scattered across individual routes?
- Is there evidence of defense in depth (multiple layers that each enforce
  something) or a single perimeter?
- Are secrets (API keys, database passwords, signing keys) stored in a dedicated
  secrets manager, or in environment variables, config files, or source code?
- Is there network segmentation — can the database be reached directly from the
  internet, or only from the application layer?
- If AI is present: is there an explicit boundary around what the LLM is
  permitted to do, and how much it is trusted?

## Signals of absent security architecture

- `.env` files with real credentials present in the repo (or committed history)
- Auth middleware applied inconsistently — some routes protected, others not
- No mention of secrets rotation anywhere
- Database connection string visible in application config with no secret store
- Admin and user functionality served from the same authentication context
- No distinction between authentication (who are you) and authorization (what
  can you do)

---

## Grading

**A** — A threat model exists naming at least the primary adversary scenarios.
Authentication and authorization are enforced at a consistent architectural
layer. Secrets live in a dedicated store (Vault, AWS Secrets Manager, similar)
with rotation. Network topology separates public-facing from private services.
Defense in depth is intentional: multiple layers each enforce something
independently.

**B** — Auth exists and works. Secrets are externalized from code. But the
security design is implicit — there is no threat model, and the authorization
model is ad hoc rather than systematic. A determined attacker would find gaps,
but opportunistic attacks would fail.

**C** — Auth exists for the happy path but has structural gaps: some routes are
unprotected, admin functionality is reachable by regular users in some paths,
or secrets are stored in ways that make rotation impossible without a redeploy.
There is no concept of defense in depth — one layer failing means everything
fails.

**D** — The system has a login screen but no security architecture. Any
authenticated user can reach any resource. Or: credentials are stored in source
code or version history. Or: the database is reachable from the public internet
with only an application-layer password as protection.

---

## Real-world incident anchors (use for C or D grades)

---

**Colonial Pipeline, May 2021.** A six-day fuel shutdown across the Eastern
United States, $4.4 million ransom paid, gas lines in multiple states.

**The design error:** No second factor required on remote access, and no
network segmentation between business IT systems and the operational systems
that control the physical pipeline.

**Why vibe-coders make the same mistake:** AI-assisted builders add a login
screen and consider auth "done." Nobody asks what happens after login —
whether internal systems are separated from each other, or whether a single
compromised account can reach everything. The perimeter gets built; the
interior never gets divided.

**What happened:** Attackers found a reused password for a VPN account that
had no MFA requirement. They got in through that one account. Once inside the
business IT network, there was no wall between it and the operational systems
running the pipeline. Colonial shut down the pipeline themselves, preemptively,
because they could not tell whether the attackers had reached the operational
side. They had no way to know, because the architecture had not separated the
two. The ransom bought a decryption key. The shutdown and the gas lines came
from the architecture, not the ransomware.

---

**Equifax, July 2017.** 147 million Social Security numbers stolen — the
largest breach of financial identity data on record at the time.

**The design error:** No interior network segmentation. The web server tier
could talk directly to the database tier. Once inside the web application,
attackers had a straight line to the data. There was a perimeter. There were
no internal boundaries.

**Why vibe-coders make the same mistake:** AI tools generate database
connection strings and query logic in the same layer as the web application
code. Nothing in the scaffolding separates the tier that handles HTTP requests
from the tier that holds sensitive data. The application works correctly, so
nobody questions the topology. Segmentation is never part of the "make it
work" phase, and there is no "make it safe" phase that follows.

**What happened:** Apache Struts, the web framework Equifax used, had a
publicly disclosed vulnerability. A patch had been available for two months.
Equifax had not applied it. Attackers exploited the vulnerability to run
commands on the web server. From the web server, they could query the
databases directly — no additional credentials, no additional barrier. They
spent 76 days moving through the network, copying data in small batches to
avoid detection. The vulnerability was the entry point. The missing network
segmentation was why 147 million records left instead of one server's worth.

---

**Uber, September 2022.** An 18-year-old attacker reached Uber's internal
systems, admin dashboards, source code repositories, and internal Slack
channels in a single session.

**The design error:** Secrets hardcoded in internal scripts rather than a
secrets store, and no separation between read and write access on internal
tools. One compromised employee account could reach everything without
privilege escalation.

**Why vibe-coders make the same mistake:** When building internal tooling and
automation scripts, AI assistants put credentials directly in the script
because that is the fastest path to a working result. Internal tools get built
quickly and never revisited. There is no moment where someone asks "what
happens if someone gets access to this script?" because the script is internal
and the focus was on making it work.

**What happened:** The attacker used MFA fatigue — sending repeated
authentication push notifications to an Uber employee's phone until the
employee approved one, probably assuming it was a system glitch. That gave the
attacker access to the employee's account on the corporate VPN. From there,
they found an internal network share containing PowerShell scripts. Those
scripts had admin credentials hardcoded in them — real credentials, not
references to a secrets store. With those credentials, the attacker reached
Uber's internal admin panels, its AWS and GCP environments, its HackerOne
vulnerability reports, and its Slack workspace. The MFA fatigue was the entry
point. The hardcoded credentials were why everything followed from it.

---

## Questions to ask if scan is ambiguous

- "If an employee leaves the company, is there a process to remove their access
  to all systems — or would you have to remember to do it manually for each
  tool?"
- "Are your database credentials stored in the same place as your source code,
  or somewhere separate that requires special access to read?"
