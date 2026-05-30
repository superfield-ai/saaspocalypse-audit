# Category 8: Observability & Operations

**What this assesses:** Whether the team knows what the system is doing in
production — and whether they find out about problems before users do. A
system with no observability is a system where the first alert is a customer
complaint or a newspaper story.

---

## What to look for in the scan

- Is there an error tracking service configured (Sentry, Datadog, Rollbar,
  or equivalent)?
- Is there application-level logging beyond framework defaults — structured
  logs that can be queried?
- Is there infrastructure monitoring — CPU, memory, disk, latency — with
  alerting thresholds?
- Is there an audit log — a tamper-evident record of who did what and when?
  (This is distinct from application logs; it is a security requirement.)
- Are there any alerting configurations, on-call schedules, or escalation
  paths defined?
- Is there a status page or uptime monitoring visible to customers?
- Is there a runbook — documented procedures for the most likely operational
  scenarios?

## Signals of absent observability

- No error tracking service — failures are invisible unless a user reports them
- Log statements are `console.log` or `print()` with no structure or retention
- No alerting configuration anywhere
- No audit log — if a record was accessed or modified, there is no trace
- No documentation of what "normal" looks like — no baseline to detect anomalies
- Uptime monitoring is "check if the demo works before a meeting"

---

## Grading

**A** — Errors surface to the team automatically and are tracked to resolution.
Logs are structured, retained, and queryable after an incident. Infrastructure
metrics are monitored with calibrated alerts — thresholds that reflect actual
failure conditions, not defaults. An audit log records security-relevant events
(logins, data access, permission changes) in a tamper-evident store. An
on-call procedure exists. Mean time to detection for a production failure is
under fifteen minutes.

**B** — Error tracking exists and is monitored. Logs exist and are retained.
Infrastructure monitoring exists with some alerting. An audit log exists for
some events but not all security-relevant ones. There is no formal on-call
process, but the team is responsive. Mean time to detection is under an hour
for obvious failures.

**C** — Observability is partial and unreliable. Errors may be logged but not
alerted. Infrastructure monitoring exists but alerting thresholds are defaults
that produce either too many alerts (ignored) or too few (gaps). No audit log.
The team typically learns about production problems from user reports rather
than internal monitoring. Mean time to detection for non-obvious failures is
measured in days.

**D** — The team has no meaningful visibility into what the system is doing in
production. Errors are not tracked. Logs either do not exist or are not
retained past a few hours. No infrastructure monitoring. No audit log. The
only indicator of a problem is a user telling you — or discovering it from
outside the company entirely.

---

## Real-world incident anchor (use for C or D grades)

**Uber data breach, 2016.**

In October 2016, attackers accessed an Uber GitHub repository, found
credentials in the code, and used them to access an Amazon S3 bucket containing
the personal data of 57 million Uber customers and 600,000 drivers — names,
email addresses, and phone numbers for customers; driver's license numbers for
drivers.

Uber's security systems did not detect the breach. The company discovered it
only when the attackers contacted Uber to demand payment. Uber paid $100,000
through its bug bounty program and asked the attackers to sign a non-disclosure
agreement, treating the extortion as a legitimate disclosure.

The breach remained secret for a year, during which time Uber disclosed it
to no one — not regulators, not affected users, not the drivers whose license
numbers had been taken. It was revealed publicly only after new leadership
took over.

The failure to detect the breach was an observability gap: no alert existed
for a credential being used from an unfamiliar location to access a sensitive
data store. The credential itself was in source code — a security failure —
but the year-long window of undetected access was an operations failure.

---

## Questions to ask if scan is ambiguous

- "If someone is accessing your system right now in a way they shouldn't be,
  would you know — and how would you find out?"
- "When something goes wrong in production, do you typically find out before
  a user contacts you, or after?"
