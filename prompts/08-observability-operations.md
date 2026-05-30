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

## Real-world incident anchors (use for C or D grades)

---

### Target data breach, 2013

**Headline.** In November 2013, attackers installed malware on Target's
point-of-sale systems and exfiltrated 40 million credit card numbers and
70 million customer records — one of the largest retail breaches in history
at the time.

**The design error.** Monitoring existed and worked. Target had recently
deployed a $1.6 million security monitoring system (FireEye) that detected
the intrusion and fired alerts exactly as designed. The alerts were ignored.
There was no defined process for who was responsible for acting on a given
alert, what response was required, or what threshold made an alert worth
escalating. The monitoring produced signal; the organization had no wiring
to convert that signal into action.

**Why vibe-coders make the same mistake.** AI-assisted builders instrument
their systems when they think to. What they almost never do is write down
— even in a single Markdown file — who looks at what alert, and what they
do when they see it. The tooling gets configured; the response loop does not.
Alerts accumulate. They become noise. They get turned off. The system goes
dark by neglect rather than by omission.

**What happened.** Attackers entered through a third-party HVAC vendor's
credentials and moved laterally to Target's payment systems. FireEye detected
the malware and sent alerts to Target's security team in Bangalore, who
escalated to Minneapolis. The alerts were not acted on. The exfiltration ran
for three weeks. The breach was ultimately reported to Target by the US
Department of Justice — not discovered internally. By the time it was
contained, the attackers had everything.

---

### Cloudflare global outage, 2019

**Headline.** In July 2019, a single bad regular expression deployed to
Cloudflare's WAF caused CPU utilization to spike to 100% on all of
Cloudflare's servers globally, taking down a significant portion of internet
traffic for 27 minutes.

**The design error.** Monitoring was not the failure — Cloudflare detected
the CPU spike immediately. The failure was in the deployment pipeline: a
configuration change could go to 100% of infrastructure simultaneously with
no staged rollout and no automatic rollback trigger wired to the metrics that
were already being monitored. The system could observe the anomaly but could
not act on it automatically, and manual rollback took 27 minutes because the
blast radius was total from the moment of deployment.

**Why vibe-coders make the same mistake.** When AI generates deployment
configuration, it produces a working pipeline — push code, it ships. What
it does not produce is a pipeline that is wired to its own metrics: deploy
to 1%, watch the error rate, then proceed. That wiring requires connecting
two systems (deployment and monitoring) in a deliberate feedback loop.
Builders running on momentum ship to everything at once because the default
is convenient and the failure mode is not visible until it happens.

**What happened.** An engineer deployed a WAF rule containing a backtracking
regular expression. The expression was syntactically valid and passed review.
When it hit production traffic, the regex engine consumed all available CPU
evaluating it. Every server in Cloudflare's network was affected at the same
moment. HTTP and HTTPS traffic failed globally. Cloudflare's own internal
tools — which ran on the same infrastructure — became unreachable, slowing
the response. The fix was a one-line rollback, but reaching the systems to
execute it took the better part of half an hour.

---

### Uber data breach, 2016

**Headline.** In October 2016, attackers accessed credentials stored in
source code, used them to exfiltrate the personal data of 57 million Uber
customers and 600,000 drivers, and were paid $100,000 to stay quiet — after
which Uber concealed the breach from regulators and users for a year.

**The design error.** No alert existed for a credential being used from an
unfamiliar location to access a sensitive data store. The credential was in
source code — a security failure — but the year-long window of undetected
access was an operations failure. There was no behavioral baseline for what
legitimate access to that S3 bucket looked like, and therefore nothing to
compare against when access became illegitimate.

**Why vibe-coders make the same mistake.** AI-generated infrastructure code
creates resources and wires them together. It does not create a model of
what normal looks like — access from which IPs, at which hours, at what
volume — and it does not configure alerts for deviations from that model.
The result is infrastructure that is functional and completely opaque to
anomalous use.

**What happened.** Attackers found Uber's AWS credentials in a GitHub
repository. They used those credentials to pull a database backup from S3
containing the personal data of 57 million riders and 600,000 drivers.
Uber's security systems did not detect the breach. The company discovered
it only when the attackers contacted Uber to demand payment. Uber paid
$100,000 through its bug bounty program and required the attackers to sign
a non-disclosure agreement, treating extortion as a legitimate disclosure.
The breach remained secret for a year. It was revealed publicly only after
new leadership took over.

---

## Questions to ask if scan is ambiguous

- "If someone is accessing your system right now in a way they shouldn't be,
  would you know — and how would you find out?"
- "When something goes wrong in production, do you typically find out before
  a user contacts you, or after?"
