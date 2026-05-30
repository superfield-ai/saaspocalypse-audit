# Category 7: Test Soundness

**What this assesses:** Whether the tests in this system actually protect
against the failures that matter — or whether they create the appearance of
safety without the substance. Vibe-coded systems frequently have tests that
were generated alongside the code, assert that things ran rather than that
they worked correctly, and test only the happy path.

---

## What to look for in the scan

- Does a test directory exist at all?
- What is the ratio of unit tests to integration tests to end-to-end tests?
  (A healthy pyramid has many unit tests, fewer integration tests, few E2E
  tests. An inverted pyramid — or only one type — is a signal.)
- Do the tests mock the database, external services, and the network — or do
  at least some tests run against real (or realistic test-double) systems?
- Do tests cover failure cases: what happens when the database is unavailable,
  when an external API returns an error, when input is malformed or adversarial?
- Are AI-generated tests verifying behavior or just verifying that code ran
  without throwing an exception?
- Is there any load or performance testing — or has the system never been run
  with more than a handful of concurrent users?
- Is there any security regression testing — do the tests verify that
  previously fixed vulnerabilities have not returned?

## Signals of unsound tests

- Test files with assertions like `expect(result).toBeDefined()` or
  `assert result is not None` — confirming existence, not correctness
- Test files that mock every external dependency including the system under
  test itself
- Only happy-path tests — no test that deliberately provides bad, missing,
  or adversarial input
- E2E or integration test suite is the only test suite
- Test files that were clearly generated in the same session as the code,
  covering the same scenarios in the same order, with the same assumptions
- No test that simulates the failure of an external dependency

---

## Grading

**A** — A real test pyramid exists: many unit tests covering individual
components, integration tests that run against a real database and real network
calls (or high-fidelity doubles), and a small number of E2E tests covering
critical user paths. Tests cover failure cases and adversarial input. At least
some tests would catch a regression in a security control. The test suite is
maintained — failures block deployment.

**B** — Tests exist and are run in CI. The coverage is skewed toward one layer
(usually unit or E2E) at the expense of others. Failure cases are partially
covered. The tests would catch most obvious regressions but not subtle behavior
changes. Tests were reviewed and are not simply auto-generated assertions of
existence.

**C** — Tests exist but they provide false confidence. Either they mock so
much that they test only the test logic itself, or they cover only the happy
path, or they were AI-generated and assert that code ran rather than that it
worked. The test suite passes consistently but would not catch the classes of
failure most likely to affect a real user.

**D** — No tests exist, or the tests that exist are structurally incapable of
catching the failures most likely to occur. This includes: a test suite where
every test passes a mocked input through mocked logic and asserts a mocked
result; a system where the only "testing" was clicking through the interface
during development; or a suite that has never been run in CI and is not
confirmed to be passing.

---

## Real-world incident anchors (use for C or D grades)

---

### Incident 1: Heartbleed — OpenSSL, April 2014

**Headline.** A two-line bounds-checking omission in OpenSSL sat undetected
for two years, exposing private keys and session tokens from an estimated
500,000 servers and roughly 17 percent of HTTPS-protected sites on the
internet.

**The design error.** The test suite covered what the heartbeat extension was
intended to do — confirm a live connection. It did not test what happened when
a heartbeat request claimed a payload larger than what was actually sent. No
test probed a boundary condition with a mismatched length field. The adversarial
input space was never exercised.

**Why vibe-coders make the same mistake.** AI coding tools generate tests
optimistically: they test the code path they just wrote, against inputs that
match what the code expects. They rarely ask "what happens if the input is
wrong." Bounds checks, length validations, and rejection of malformed payloads
are exactly the cases that AI-generated tests omit — and exactly the cases
that attackers probe first.

**What happened.** OpenSSL implemented a TLS feature called the heartbeat
extension — a lightweight ping that lets one end of a connection ask the other
to echo back a small block of data, confirming it is still alive. The request
included a field stating the payload length. The implementation trusted that
field without verifying it against the actual bytes sent. An attacker could
send a request claiming a payload of 64 KB when they had sent a few bytes.
The server would dutifully read 64 KB out of process memory and send it back
— including whatever happened to live in that memory: private keys, session
tokens, passwords, other users' plaintext data. The fix was two lines of code.
The omission had been in production for two years. The test suite had passed
throughout.

---

### Incident 2: TSB Bank migration failure — May 2018

**Headline.** TSB Bank's weekend migration of 5.4 million customer accounts
to a new platform left 1.9 million customers locked out for days, allowed some
users to see other people's account balances, and ultimately cost the bank
over £330 million in remediation, compensation, and regulatory fines.

**The design error.** The migration was tested against a subset of accounts in
a test environment that did not reflect the volume, variety, or edge cases of
the real customer base. Performance under real load had never been verified.
Data edge cases — unusual account types, dormant accounts, accounts with
historical anomalies — were not represented in the test dataset. The test
environment passed. The production environment, with five million real accounts
and peak weekend login traffic, did not.

**Why vibe-coders make the same mistake.** AI-assisted builders routinely test
against small, well-formed datasets — the happy-path data that was easy to
generate or describe. They verify that the logic works for representative
examples, not that it holds under volume, variety, and real-world messiness.
Load testing and data-variety testing require deliberate effort that is easy to
skip when everything passes locally.

**What happened.** TSB had been operating on infrastructure shared with its
former parent, Lloyds Banking Group, since its separation in 2013. A dedicated
weekend was scheduled to migrate all customer accounts to TSB's own platform.
The test runs in the preceding months used a representative but incomplete
subset of accounts in an environment that could not replicate production load.
When the cutover happened on a Friday night, the new platform buckled under
real traffic. Customers logging in on Saturday morning found their accounts
inaccessible. Some could see account information belonging to other customers
— a data isolation failure that had not appeared in testing because the test
environment never ran concurrent sessions at scale. The outage lasted weeks
for some customers. The CEO resigned. The FCA fined TSB £48.6 million.

---

### Incident 3: Knight Capital zombie code — August 2012

**Headline.** Knight Capital Group, a major US equity trading firm, lost $440
million in 45 minutes when a software deployment reactivated a dormant code
path that had not been tested in a production-like environment and was
invisible to the existing test suite.

**The design error.** A feature flag used years earlier to control an
experimental order-routing strategy called Power Peg had been repurposed to
activate a new component. The old Power Peg code was never deleted; it was
simply dormant. No test verified what happened when that flag was set in a
production-like environment — because the tests only exercised the current
intended behavior. The zombie code path existed outside the boundary of the
test suite entirely.

**Why vibe-coders make the same mistake.** AI-assisted development produces
code that tests what the new feature does. It does not produce tests that ask
"is there any other code path that this deployment could activate?" Dead code,
legacy flags, and dormant branches are invisible to tests that only verify
intended behavior. When old code and new deployments interact in an untested
combination, the test suite offers no warning.

**What happened.** Knight Capital was deploying a new automated order-routing
system. The deployment reused a configuration flag that had previously
controlled Power Peg, an experimental strategy that had been decommissioned
years earlier but whose code remained in the codebase. The new deployment was
rolled out to seven of eight production servers; one server was missed. When
markets opened on August 1, 2012, that server began executing Power Peg logic:
buying and immediately selling stocks, accumulating a massive unintended
position. The automated system executed millions of orders in 45 minutes.
By the time engineers identified the source and shut it down, the firm had
lost $440 million — more than its entire net income for the previous year.
Knight Capital survived only through an emergency equity injection. A test
that verified what happened when the Power Peg flag was set would have
surfaced the zombie code before it reached production.

---

**AI-generated test addendum.**

AI coding tools generate tests optimistically — they generate tests that pass
given the implementation they just wrote, against inputs that match what the
code expects. They rarely generate tests that ask "what happens if the input
is wrong," "what happens if the downstream service fails," or "what does this
code path do under load?" Three specific gaps appear repeatedly in AI-generated
test suites:

- **Adversarial input is not tested.** The tool writes the code and writes
  tests for the code. Both share the same optimistic assumptions about what
  inputs look like. Malformed data, boundary values, and attacker-crafted
  payloads are not in the mental model that produced either artifact.

- **Load and data variety are not tested.** AI tools generate tests against
  small, well-formed example data. They do not generate tests that ask whether
  the system holds under five million real accounts, high concurrency, or a
  dataset with historical edge cases.

- **Dead code paths are not tested.** Tests verify intended behavior. Dormant
  branches, legacy flags, and interaction effects between an old codebase and
  a new deployment are outside the boundary of what an AI tool will
  spontaneously test.

A test file where every test was generated in the same session as the code it
tests should be treated as a documentation artifact — evidence of what the
author intended the code to do — not a safety mechanism. Safety mechanisms
test what can go wrong, not what is supposed to go right.

---

## Questions to ask if scan is ambiguous

- "Has the system ever been tested with bad data — wrong formats, missing
  fields, inputs that are much larger or smaller than expected?"
- "If the payment processor or email service your system uses went down, would
  the tests catch that the system handles it gracefully — or would you only
  find out from a customer?"
- "Has the system ever been run under anything close to its expected peak load,
  or has it only ever been used by a handful of people at once?"
- "Is there any code in this codebase that is no longer called by the current
  application — old feature flags, legacy endpoints, deprecated functions?
  Have you verified what happens if any of it were to be reactivated?"
