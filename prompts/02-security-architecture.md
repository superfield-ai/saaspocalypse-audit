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

## Real-world incident anchor (use for C or D grades)

**Colonial Pipeline, May 2021.**

Colonial Pipeline carries 45 percent of the fuel used on the East Coast of the
United States. Attackers gained access through a single VPN account — one that
used a reused password and had no second factor of authentication required. The
architectural security design of the network did not separate the business IT
systems from the operational systems that control the physical pipeline. Once
inside one, attackers could reach the other.

The company paid $4.4 million in ransom and shut down pipeline operations for
six days. Gas prices spiked across the Eastern United States. Lines formed at
fuel stations in multiple states.

The entry point was a single missing architectural decision: requiring a second
factor of authentication on remote access. The blast radius was defined by a
second missing decision: network segmentation between IT and operational
systems. Neither required sophisticated code. Both required someone to ask
the design question first.

---

## Questions to ask if scan is ambiguous

- "If an employee leaves the company, is there a process to remove their access
  to all systems — or would you have to remember to do it manually for each
  tool?"
- "Are your database credentials stored in the same place as your source code,
  or somewhere separate that requires special access to read?"
