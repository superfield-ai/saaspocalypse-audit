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

## Real-world incident anchors (use for C or D grades)

---

**SolarWinds, December 2020.** A nation-state attacker compromised the build
pipeline of a major IT monitoring vendor and distributed malicious code to
18,000 customers, including most US federal agencies, via a legitimate signed
software update. The attacker had undetected access to government and corporate
networks for up to nine months.

**The design error:** The build pipeline had no mechanism to verify its own
integrity. There was no signing of intermediate artifacts, no comparison of
expected versus actual build outputs. The pipeline assembled and signed whatever
it was given.

**Why vibe-coders make the same mistake:** AI-generated build pipelines are
oriented around delivery — get it compiled, get it shipped. Integrity checks on
the pipeline itself are a second-order concern that never makes it into the first
draft. If you didn't design the pipeline to catch tampering, it won't.

**What happened:** Attackers inserted a small malicious module into the source
tree that SolarWinds' build system compiled into its Orion product. The output
was code-signed by SolarWinds in the normal way, so every customer who applied
the update received a backdoor that appeared to be a legitimate patch. The full
scope of the breach was never determined. The investigation ran for years.

---

**Codecov, April 2021.** Attackers modified a bash script hosted by code
coverage tool Codecov that was fetched and executed inside the CI pipelines of
thousands of companies, including Twilio, Atlassian, and Rapid7. The modified
script silently exfiltrated environment variables — including secrets, API keys,
and tokens — from every pipeline that ran it. The compromise went undetected for
two months.

**The design error:** Codecov's CI uploader was distributed as a script fetched
from a remote URL at runtime, with no integrity check. No hash verification, no
signature validation. Any modification to the hosted file was automatically
trusted by every downstream pipeline.

**Why vibe-coders make the same mistake:** Fetching a script from a URL and
piping it into bash is a common pattern in developer tooling and setup guides.
AI-assisted pipelines reproduce this pattern without adding integrity checks
because the pattern appears to work — until the remote file changes in a way
you didn't intend. Pinning and verifying external artifacts is a discipline that
requires knowing why it matters before you know to do it.

**What happened:** Attackers gained access to Codecov's Docker image build
process and modified the bash uploader script stored there. When customers' CI
pipelines ran the Codecov step, they fetched the tampered script and executed
it. The script collected all environment variables from the running pipeline —
credentials, access tokens, signing keys — and sent them to an attacker-
controlled server. Companies discovered the breach only after Codecov's own
investigation surfaced it two months later.

---

**npm event-stream, November 2018.** A malicious maintainer took ownership of
event-stream, an npm package with two million weekly downloads, and added a
dependency that stole private keys from users of the Copay Bitcoin wallet.
Hundreds of thousands of npm installs pulled in the malicious code before
detection.

**The design error:** npm's package ownership model allowed any package to be
transferred to a new maintainer with no review process, regardless of how
critical the package had become to the ecosystem. Ownership of critical
infrastructure was transferable on the same terms as ownership of a personal
project.

**Why vibe-coders make the same mistake:** Dependency management in AI-assisted
projects is typically about making something work, not about understanding who
controls what you're pulling in. Lock files pin versions but they don't answer
the question: who owns this package now, and do I trust them? That question
requires a process that most teams — and essentially all AI-generated setups —
never establish.

**What happened:** The original author of event-stream transferred the package
to a new contributor who had submitted a helpful-looking pull request. The new
maintainer added a dependency called flatmap-stream that contained obfuscated
malicious code targeting the Copay Bitcoin wallet. The malicious code was
executed only when the Copay wallet app was running; all other users were
unaffected, which delayed detection. By the time the attack was discovered,
the malicious version had been installed hundreds of thousands of times.

---

## Questions to ask if scan is ambiguous

- "If the person who built this system was unavailable for two weeks, could
  anyone else make a change to it safely?"
- "When a change is made to the system, is there any record of who made it,
  why, and what it was intended to do?"
