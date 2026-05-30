# Category 1: Architecture

**What this assesses:** Whether the system was designed or just assembled. An
architectural audit looks for evidence of deliberate decisions about structure,
boundaries, and data — not whether the code is clean.

---

## What to look for in the scan

- Is there any architecture document, ADR, or design doc?
- Does the directory structure suggest intentional domain separation, or is
  everything in one flat layer?
- Is there a data model document or schema file?
- Does the README describe how the system is structured, not just what it does?
- Is there evidence of multi-tenancy thinking (separate schemas, tenant IDs,
  row-level security configuration)?
- Are there explicit boundaries between user-facing code, business logic, and
  data access — or is everything mixed?

## Signals of absent architecture

- Single large file or directory with everything in it
- No schema or data model documented anywhere
- API routes defined in the same file as database queries
- No mention of how the system scales or what happens under load
- README describes features only, never structure

---

## Grading

**A** — Explicit architecture documentation exists. Trust boundaries are named
and enforced at the right layers. Data model is documented and has integrity
constraints. Multi-tenancy (if applicable) is an explicit design decision with
a documented isolation strategy. Decisions are recorded as ADRs or equivalent.

**B** — The system has discernible structure but it is implicit — you can infer
the architecture by reading the code, but it was never written down. Trust
boundaries exist but are enforced inconsistently. The data model is functional
but undocumented.

**C** — The system has no discernible architecture. Business logic, data access,
and API handling are interleaved. There is no evidence that anyone asked "what
happens if two tenants' data mixes" or "what changes when we add a second
service." The structure reflects the order things were built, not a design.

**D** — The system has critical structural omissions that are not implementation
details. Access control is not a concept at the architectural layer — any
authenticated (or unauthenticated) actor can reach any resource. Or: no
isolation exists between tenants, environments, or privilege levels anywhere
in the design.

---

## Real-world incident anchor (use for C or D grades)

**Parler, January 2021.**

Parler was a social network that grew to 15 million users. When it was taken
offline by its infrastructure providers, a researcher discovered that its API
had no authentication on the endpoints that served media — photos, videos,
and location metadata embedded in posts.

The architecture simply did not include the concept of access control on
resources. It wasn't that the implementation was wrong; the design never
asked the question. In the days before shutdown, 70 terabytes of content —
including posts users had deleted, and GPS coordinates from photos — were
downloaded by automated scrapers.

The damage was not from a sophisticated attack. It was from an architectural
omission that no one had noticed because the system had never been reviewed
by anyone asking "who is allowed to access what, and where is that enforced?"

---

## Questions to ask if scan is ambiguous

- "Is there any document — even a diagram on a whiteboard that was photographed
  — that describes how the different parts of the system connect?"
- "If one customer's data accidentally appeared in another customer's account,
  would the system have any way of detecting that?"
