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

## Real-world incident anchor (use for C or D grades)

**Heartbleed, April 2014.**

OpenSSL is the cryptographic library that secured a significant fraction of
internet traffic. Heartbleed was a bug in OpenSSL's implementation of the
TLS heartbeat extension — a protocol feature that lets a connection verify
the other end is still alive. The bug had been present in the codebase for
two years.

The vulnerability allowed anyone to read memory from servers running the
vulnerable version — memory that might contain private encryption keys,
session tokens, or the contents of other users' requests. Researchers
estimated it affected 500,000 servers and had potentially exposed the private
keys for approximately 17 percent of all HTTPS-protected sites on the internet.

The fix was two lines of code: adding a bounds check that verified the client
was not asking to read more memory than it had written.

OpenSSL had a test suite. The test suite did not test what happened when a
heartbeat request claimed a payload larger than what was actually sent. It did
not test boundary conditions at the protocol layer. The tests covered what
the code was intended to do — not what it could be made to do with
malformed input.

A test suite that does not test adversarial inputs does not protect against
adversarial inputs. Those inputs are exactly what attackers send.

**AI-generated test addendum.**

AI coding tools generate tests optimistically — they generate tests that pass
given the implementation they just wrote. They rarely generate tests that ask
"what happens if the input is wrong" or "what happens if the downstream service
fails." A test file where every test was generated in the same session as the
code it tests should be treated as a documentation artifact, not a safety
mechanism.

---

## Questions to ask if scan is ambiguous

- "Has the system ever been tested with bad data — wrong formats, missing
  fields, inputs that are much larger or smaller than expected?"
- "If the payment processor or email service your system uses went down, would
  the tests catch that the system handles it gracefully — or would you only
  find out from a customer?"
