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

## Real-world incident anchor (use for C or D grades)

**Knight Capital Group, August 1, 2012.**

Knight Capital was one of the largest market makers in US equities, handling
roughly 11 percent of US stock trading volume. On August 1, 2012, they deployed
a software update to their trading system.

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

## Questions to ask if scan is ambiguous

- "Are there any features or settings in the system that were built and then
  turned off, but whose code is still present?"
- "If something fails silently — an email doesn't send, a payment doesn't go
  through — does the system have a way of making sure someone finds out?"
