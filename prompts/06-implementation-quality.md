# Category 6: Implementation Quality

**What this assesses:** Whether a second person — human or AI — can understand,
change, and reason about the code without introducing new failures. Vibe-coded
systems often have working features alongside implementation patterns that make
the system progressively harder to change safely over time.

This is still an architectural lens: not whether individual functions are
optimal, but whether the code's organization reflects a coherent model of what
the system does.

---

## What to look for in the scan

- Is the code organized by domain or by technical layer — and is that choice
  consistent throughout?
- Is there evidence of dead code — features that appear to still be configured
  or connected but serve no current purpose?
- Are error conditions handled consistently — or does the system silently
  swallow failures in some paths and crash in others?
- Is there a logging strategy — structured, queryable logs — or ad hoc print
  statements?
- Is configuration (URLs, limits, feature flags) centralized or scattered
  through code as magic strings?
- Are there multiple implementations of the same concept (two auth libraries,
  two ORM patterns, three ways to call the same API) suggesting accumulated
  AI-generated code that was never reconciled?

## Signals of poor implementation quality (architectural view)

- Multiple files doing the same thing differently — sign of unreviewed AI
  output
- Feature flags or configuration values in comments or unused variables
- Error handling absent from async operations — unhandled promise rejections,
  uncaught exceptions
- Log statements that reveal internal state (stack traces, SQL queries) in
  production paths
- Hardcoded values that change between environments scattered through business
  logic
- No centralized configuration — changing a URL requires finding every place
  it appears

---

## Grading

**A** — The code has a coherent, consistent model. Someone unfamiliar with the
project can locate where a given behavior is implemented by following the
organizational logic. Errors surface to the right layer. Logs are structured
and queryable. Configuration is centralized. Dead code is absent or explicitly
marked for removal. AI-generated code has been reviewed and reconciled into the
existing model.

**B** — The code is generally navigable but has accumulated inconsistencies.
Some paths handle errors differently from others. Logs exist but are
inconsistent in structure. Configuration is mostly centralized with some
scattered exceptions. An experienced engineer would be productive but would
need to map some of the inconsistencies before making changes safely.

**C** — The codebase is a collage. Different sections were written (or generated)
independently and never reconciled. Error handling is inconsistent enough that
failures in some paths are invisible. There is no logging strategy — some things
are logged, others are not, with no reasoning behind the distinction. Dead code
is present and indistinguishable from live code without tracing execution paths.

**D** — The code contains at least one class of implementation problem that will
produce unpredictable behavior under real conditions: zombie features that
activate under specific conditions no one is tracking, errors that are silently
swallowed in the main execution path, or configuration values that produce
different behavior than expected because they were copied from an AI session
that had a different environment in mind.

---

## Real-world incident anchors (use for C or D grades)

---

### Knight Capital Group, August 1, 2012 — $440 million in 45 minutes

**The design error:** Dead code left in the codebase, still wired to a live
feature flag, with no indication that the flag was still meaningful.

**Why vibe-coders make the same mistake:** AI-assisted development produces
working features quickly, but rarely produces the cleanup work that follows —
removing old code, auditing what flags still do, verifying that nothing
disabled is still connected. The codebase looks functional because the new
feature works. The old code is invisible until it activates.

**What happened:** Knight Capital was one of the largest market makers in US
equities, handling roughly 11 percent of US stock trading volume. On August 1,
2012, they deployed a software update to their trading system.

The update repurposed a feature flag that had previously activated a function
called "Power Peg" — a trading strategy that had been disabled and was no
longer used. No one had removed the dead code. No one knew the flag was still
wired to it.

On deployment, the flag was inadvertently set to the active state on seven of
eight production servers. The eighth server did not receive the update.

For forty-five minutes, the system executed four million unintended stock trades
— buying high, selling low, repeatedly, across 154 different stocks. By the
time anyone understood what was happening and could shut it down, Knight Capital
had lost $440 million.

The company was acquired within days. It did not survive as an independent firm.

The failure was not a bug in the trading logic. The failure was dead code that
no one knew was still connected, activated by a flag no one knew was still
meaningful. Implementation quality is not about elegance — it is about whether
the system does only what its operators believe it does.

---

### CrowdStrike, July 19, 2024 — 8.5 million machines, simultaneous global outage

**The design error:** A content configuration file was applied to all production
endpoints simultaneously, with no schema validation and no staged rollout. The
file contained an out-of-bounds read that the sensor's update logic did not
catch before execution.

**Why vibe-coders make the same mistake:** When AI generates a data-loading or
configuration-parsing function, it typically handles the happy path and trusts
the input. Schema validation, bounds checking, and canary deployment are
operational disciplines that require deliberate design decisions — they don't
emerge from "make this work." Vibe-coded systems routinely parse external files
and apply configuration without validating structure, because the files the
developer tested with were always valid.

**What happened:** CrowdStrike's Falcon sensor is endpoint security software
running on millions of Windows machines worldwide. It is updated frequently
with "content configuration" files — not code updates in the traditional sense,
but data files that tell the sensor what to look for.

On the morning of July 19, 2024, CrowdStrike pushed a routine content update.
The file contained a structural error that caused the sensor to attempt an
out-of-bounds memory read when processing it. Windows does not allow that
operation from kernel-level code. Every machine that applied the file
immediately crashed with a blue screen of death and could not restart cleanly.

The update went to all enrolled machines simultaneously. There was no canary
deployment, no staged rollout, no circuit breaker that would have stopped the
push after the first wave of crashes was detected.

Airlines grounded flights. Hospitals diverted emergency patients. Broadcasters
went dark. Banks took their systems offline. The recovery required physical
access to each affected machine to delete the file manually — it could not be
done remotely because the machines could not boot. Estimated global economic
damage exceeded $5 billion.

The sensor had been reading content files for years without incident. The
validation logic trusted the format. It took one malformed file, delivered
everywhere at once, to take down critical infrastructure on five continents.

---

### Boeing 737 MAX MCAS, October 2018 and March 2019 — 346 people killed

**The design error:** A safety-critical automated system relied on input from a
single sensor with no cross-validation against a redundant sensor. When that
sensor malfunctioned, the system acted on false data with no alerting to the
operators and no manual override that pilots were trained to use.

**Why vibe-coders make the same mistake:** Single-source input without
validation is the default pattern in AI-generated code. When an AI writes a
function that reads a value and acts on it, it does not ask "what if this value
is wrong?" Redundancy, cross-validation, and operator alerting require the
developer to explicitly design for the failure case. Vibe-coded systems almost
never do — the sensor works in every test, so there is no test that surfaces
the absent safeguard.

**What happened:** The 737 MAX had a new engine configuration that changed the
aircraft's handling characteristics. Boeing added the Maneuvering Characteristics
Augmentation System — MCAS — to automatically push the nose down under certain
conditions to compensate. Pilots were not told the system existed. It was not in
the flight manual. There was no training for it.

MCAS read the angle-of-attack from a single sensor on the nose of the aircraft.
It did not cross-check against the second sensor that all 737 MAXs carry. When
that single sensor malfunctioned and reported a false high angle, MCAS activated
and repeatedly pushed the nose down. Each time pilots pulled up to counteract
it, the system overrode them and pushed down again.

On Lion Air Flight 610 in October 2018, and Ethiopian Airlines Flight 302 in
March 2019, pilots fought the system for several minutes before the aircraft
became unrecoverable. 346 people died.

The engineering failure was not a malfunctioning sensor — sensors fail, and
aircraft are designed to tolerate that. The failure was a single point of
failure in an automated control system, with no redundancy, no cross-validation,
no alerting to the crew that the system had activated, and no documented
override. The system was confident in its single input, and it was wrong.

---

## Questions to ask if scan is ambiguous

- "Are there any features or settings in the system that were built and then
  turned off, but whose code is still present?"
- "If something fails silently — an email doesn't send, a payment doesn't go
  through — does the system have a way of making sure someone finds out?"
