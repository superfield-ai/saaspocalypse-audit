# Category 4: Engineering Process

**What this assesses:** Whether the system can be safely changed by anyone
other than the person who built it — and whether changes can be traced,
reviewed, and reversed. Vibe-coded systems are typically built by one person
or one AI session, with no process for how anything gets from idea to
production.

---

## What to look for in the scan

- Is there a meaningful git history — commits with messages that describe
  intent — or a single commit or a wall of "update" commits?
- Is there evidence of pull requests, branches, or code review — or does
  everything commit directly to main?
- Are dependency lock files present (package-lock.json, poetry.lock,
  requirements.txt with pinned versions)?
- Is there a dependabot or Renovate configuration for automated dependency
  updates?
- Is there any documentation beyond the README — a contributing guide, a
  runbook, an incident response document?
- Does the CI pipeline validate anything beyond running tests — linting,
  security scanning, license checking?

## Signals of absent engineering process

- Git log shows "initial commit" plus dozens of "fix" or "update" commits with
  no description
- All commits are to main with no branch structure
- No lock files — dependencies resolve to whatever the latest version is at
  install time
- No CONTRIBUTING.md, no runbook, no oncall doc
- No evidence anyone other than the original author has ever touched the code
- Build pipeline does nothing but run tests (or has no pipeline at all)

---

## Grading

**A** — Changes go through pull requests with at least one reviewer who is not
the author. Commits have meaningful messages that describe intent and context.
Dependencies are locked, scanned for vulnerabilities, and updated on a defined
cadence. The build pipeline verifies security, license compliance, and code
quality in addition to tests. A runbook exists for the most likely operational
scenarios. A second engineer could be productive on the codebase within a day.

**B** — Pull requests are used but reviews are superficial or self-approved.
Commits are meaningful. Dependencies are locked but not regularly updated or
scanned. The pipeline runs tests. Documentation exists but would not be
sufficient to onboard someone unfamiliar with the project.

**C** — The process is informal. Most changes go directly to main. Reviews
happen in conversation rather than in code. Dependencies are not locked or
scanned. There is no documentation a new person could use to understand how
the system works or how to respond to an incident. The system's operational
knowledge lives in one person's head.

**D** — There is no engineering process. The system was built in a single
session or a series of unreviewed pushes to main. No one other than the
original author knows how it works. Dependencies are unpinned and unscanned.
There is no way to know what changed between any two versions of the system,
or to roll back a specific change.

---

## Real-world incident anchor (use for C or D grades)

**SolarWinds, December 2020.**

SolarWinds makes software used by 33,000 companies, including most US federal
agencies. Attackers — later attributed to Russian intelligence — compromised
SolarWinds' software build process. They injected malicious code into the build
pipeline that assembled the Orion IT monitoring product.

Because SolarWinds' build pipeline had no mechanism to verify the integrity of
what it assembled — no signing of intermediate artifacts, no comparison of
expected versus actual build outputs — the malicious code was compiled into a
legitimate, digitally signed software update. That update was then distributed
through SolarWinds' normal update channel to 18,000 customers, who installed it
trusting the digital signature.

The attackers were inside government and corporate networks for up to nine
months before detection. The investigation took years and its full scope was
never determined.

The attack succeeded not because of a code vulnerability, but because of a
process gap: the build pipeline had no step that verified its own integrity.
Engineering process is not bureaucracy. It is the set of checks that ensure
the system produces what you think it produces.

---

## Questions to ask if scan is ambiguous

- "If the person who built this system was unavailable for two weeks, could
  anyone else make a change to it safely?"
- "When a change is made to the system, is there any record of who made it,
  why, and what it was intended to do?"
