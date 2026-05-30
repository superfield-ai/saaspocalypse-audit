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

## Real-world incident anchors (use for C or D grades)

---

**Parler, January 2021. 70 terabytes of private user data — including deleted
posts and GPS coordinates from photos — downloaded in days by automated
scrapers.**

The design error: the architecture had no concept of access control on
resources. Authentication was implemented at login, but the concept of "who is
allowed to retrieve this specific piece of content" did not exist anywhere in
the design. Any URL that worked for a logged-in user worked for anyone.

Why vibe-coders make the same mistake: AI-assisted builders typically generate
working login flows immediately — the app asks for a password, the password
check works, and the builder moves on. What gets skipped is the separate,
harder question: after someone is logged in, does the system check whether they
are allowed to access each specific thing they request? Those are two different
problems. The first one looks finished. The second one is invisible until it
fails.

What happened: Parler was a social network that had grown to 15 million users.
When its infrastructure providers announced they were taking it offline, a
researcher noticed that the media endpoints — the URLs that served photos,
videos, and location data — required no credentials at all. She wrote a simple
script to request content in sequence. Over the next several days, before the
shutdown completed, 70 terabytes of data were downloaded, including posts users
believed they had deleted and GPS coordinates stripped from uploaded photos.
There was no sophisticated attack. The researcher simply asked for things and
the system gave them.

---

**Optus, October 2022. 9.8 million customer records — driver's license and
passport numbers included — retrieved by an attacker who incremented a number
in a web address.**

The design error: the API had no authorization model. Every customer record had
a sequential integer ID, and any request for a record — authenticated or not —
was fulfilled. The architecture assumed that reaching the login page was the
only gate that mattered. There was no design-level concept of "this customer's
record may only be accessed by this customer."

Why vibe-coders make the same mistake: when an AI tool generates an API, it
typically creates endpoints that accept an ID and return the matching record.
That is the correct shape of a working feature. What the generator does not add
— because the developer did not ask for it — is the check that confirms the
person making the request is the person whose record is being requested. The
feature works in testing because the developer always tests with their own data.
The gap only becomes visible when someone else uses a different ID.

What happened: Optus is one of Australia's largest telecommunications companies.
A customer's account record in their system had an ID — just a number, assigned
in order. An attacker discovered that they could request a record by its ID
through an API endpoint, and the system returned it without checking whether the
requester had any right to that record. By incrementing the ID, they moved from
one customer's record to the next. The attacker downloaded 9.8 million records
before anyone noticed. The exposed data included names, dates of birth, phone
numbers, email addresses, and government identity document numbers.

---

**Facebook / Meta, April 2019. 540 million Facebook user records found sitting
in publicly accessible storage on Amazon's servers — placed there not by
Facebook, but by third-party developers Facebook had trusted with user data.**

The design error: the architecture had no data governance model for what
happened to user data after it left Facebook's systems. Trust and access control
were defined at the API boundary — getting data out required credentials — but
the design made no provision for what partners were allowed to do with the data
once they had it. The system treated "left our servers" as "no longer our
problem."

Why vibe-coders make the same mistake: builders using AI tools think about their
own system. They add authentication, they protect their own database, and they
move on. The question "what happens to this data after a third party receives
it from our API?" rarely comes up during AI-assisted development because it is
not a code question — it is an architectural question about trust boundaries and
where your responsibility ends. AI tools do not prompt builders to ask it.

What happened: Facebook allowed third-party app developers to access user data
through its API to build games, quizzes, and other applications. Two of those
developers stored the data they received in Amazon S3 buckets — essentially
cloud storage folders — and misconfigured those folders so that anyone on the
internet could read them. Security researchers found 540 million Facebook user
records — account names, IDs, comments, and in one dataset, plaintext passwords
— exposed with no access restrictions. Facebook's own servers were never
breached. The failure was that once data crossed the API boundary, the
architecture gave Facebook no visibility into or control over what partners did
with it.

---

## Questions to ask if scan is ambiguous

- "Is there any document — even a diagram on a whiteboard that was photographed
  — that describes how the different parts of the system connect?"
- "If one customer's data accidentally appeared in another customer's account,
  would the system have any way of detecting that?"
