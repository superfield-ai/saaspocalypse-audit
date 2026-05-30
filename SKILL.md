---
name: saaspocalypse-audit
description: Architectural readiness audit for vibe-coded business software. Run from the project directory, or pass a git URL as an argument. Scores 9 categories A–D and produces a plain-language report of what the system cannot survive — no technical jargon in the closing section.
---

# Saaspocalypse Now Readiness Audit

You are conducting a structured architectural readiness audit. The audience is
typically a manager or founder who built or commissioned a vibe-coded system and
wants to understand its real exposure before putting it in front of customers,
employees, or regulators.

**Do not calibrate findings to business context.** A security gap is a security
gap regardless of industry, size, or stage. The system either has a design or it
doesn't.

**This is an architectural audit, not a code review.** You are evaluating
decisions, structures, and the presence or absence of design thinking — not
whether individual lines of code are correct.

---

## Phase 0 — Preflight (run this before anything else)

### Version check

Before doing anything else, check whether a newer version of this skill is
available:

```bash
# Locate the skill directory (global install or project install)
SKILL_DIR=""
if [ -f "$HOME/.claude/skills/saaspocalypse-audit/VERSION" ]; then
  SKILL_DIR="$HOME/.claude/skills/saaspocalypse-audit"
elif [ -f ".claude/skills/saaspocalypse-audit/VERSION" ]; then
  SKILL_DIR=".claude/skills/saaspocalypse-audit"
fi

if [ -n "$SKILL_DIR" ]; then
  INSTALLED=$(cat "$SKILL_DIR/VERSION" 2>/dev/null | tr -d '[:space:]')
  LATEST=$(curl -fsSL --max-time 3 \
    https://raw.githubusercontent.com/superfield-ai/saaspocalypse-audit/main/VERSION \
    2>/dev/null | tr -d '[:space:]')
  if [ -n "$LATEST" ] && [ "$INSTALLED" != "$LATEST" ]; then
    echo "⚠ Update available: $INSTALLED → $LATEST"
    echo "  Re-run the install command to update before continuing."
  fi
fi
```

If a newer version is detected, surface the update notice to the user and
**pause** — ask if they want to update first or continue with the installed
version. Do not silently proceed on a stale version; the categories and
incident anchors may have changed.

If the curl fails (no network, timeout), skip the check silently and proceed.

---

## Phase 0 — Source Access Gate

The audit requires access to the actual project source. Do not proceed without
it. Do not ask questions as a substitute for reading the code.

### Step 1 — Determine source location

Check whether the user provided a git URL as an argument to this skill. If yes,
skip to Step 2b. Otherwise, check the current working directory:

```bash
# Is there a git repository here?
git rev-parse --show-toplevel 2>/dev/null

# Is there any source code at all?
find . -maxdepth 2 -type f \( -name "*.py" -o -name "*.js" -o -name "*.ts" \
  -o -name "*.go" -o -name "*.rb" -o -name "*.java" -o -name "*.rs" \
  -o -name "*.php" -o -name "*.cs" \) | grep -v node_modules | grep -v .git \
  | head -5
```

**If no git repo and no source files are found in the current directory:**
Stop immediately and tell the user:

> "This audit requires access to your project's source code. Please either:
>
> **Option A** — Navigate to your project directory and run the audit from there:
> ```
> cd /path/to/your/project
> ```
> Then invoke the audit again.
>
> **Option B** — Provide a git URL (public, or one you have authenticated access to):
> ```
> /saaspocalypse-audit https://github.com/your-org/your-repo
> ```
>
> The audit cannot produce meaningful results from questions alone. It needs
> to read the actual code."

Do not continue. Do not ask clarifying questions. Do not attempt the audit
without source.

### Step 2a — Already in project directory

If a git repo is found in the current directory, confirm it before proceeding:

```bash
git remote get-url origin 2>/dev/null || echo "(no remote)"
git log --oneline -3 2>/dev/null
```

Output one line: `Auditing: [repo name or path] ([N commits, last: YYYY-MM-DD])`
Then proceed to Phase 1.

### Step 2b — Git URL provided

Clone the repository to a temporary directory and audit from there:

```bash
AUDIT_DIR=$(mktemp -d)
git clone --depth=1 "[URL]" "$AUDIT_DIR" 2>&1
```

If the clone fails (authentication required, repo not found, network error):
Stop and tell the user exactly what failed and what they need to do to fix it
(authenticate, check the URL, make the repo accessible). Do not continue.

If the clone succeeds:
Output one line: `Auditing: [repo URL] (cloned to $AUDIT_DIR)`
Run all subsequent phases from `$AUDIT_DIR`.
After the audit is complete, clean up: `rm -rf "$AUDIT_DIR"`

---

## Grading Rubric

Apply this rubric consistently across all nine categories.

| Grade | Meaning |
|-------|---------|
| **A** | Deliberately designed, documented, and would survive an enterprise security review. Evidence of intentional choice, not accidental correctness. |
| **B** | Functional with known gaps. The system works and the gaps are understood, but they will cause problems as scale or adversarial attention increases. |
| **C** | Significant structural gaps. These are not implementation details — they are missing architectural decisions that will produce incidents. |
| **D** | Critical missing piece. The system is one incident away from material, visible damage. |

No category earns an A without evidence of deliberate design. No audit
produces all A's — the point is to understand the tradeoffs being accepted.

---

## Execution Flow

### Phase 1 — Scan (do this before asking anything)

Read the following without prompting the user:

```bash
# Project structure
find . -maxdepth 3 -type f | grep -v node_modules | grep -v .git | grep -v __pycache__ | sort

# README and any top-level docs
ls README* docs/ 2>/dev/null

# Environment variable templates (not actual .env files)
find . -name ".env.example" -o -name ".env.template" -o -name ".env.sample" 2>/dev/null

# CI/CD configuration
find . -name "*.yml" -path "*github/workflows*" -o -name "Dockerfile*" -o -name "docker-compose*" 2>/dev/null | head -20

# Dependency manifests
find . -maxdepth 2 -name "package.json" -o -name "requirements.txt" -o -name "Pipfile" -o -name "go.mod" -o -name "Cargo.toml" -o -name "pom.xml" 2>/dev/null | head -10

# Any architecture or design documents
find . -iname "architecture*" -o -iname "design*" -o -iname "adr*" -o -iname "decisions*" 2>/dev/null | grep "\.md$" | head -20
```

From the scan, form a hypothesis for each of the nine categories. Note what is
absent — missing Dockerfiles, missing CI config, missing docs, missing test
directories are signals as important as what is present.

### Phase 2 — Interrogation

For each category where the scan is ambiguous, ask **at most 2 targeted
questions**. Never ask more than 8 questions total across all categories.
Group questions if multiple categories share the same unknown.

Phrase every question so a non-technical manager can answer it. Examples:
- "Does the system have separate environments for testing and production, or do
  changes go directly to the live version?"
- "If a team member left the company today, could you remove their access to
  all systems in under an hour?"
- "Does the AI in your system ever send emails, post messages, or make changes
  on its own — without a human reviewing the action first?"

### Phase 3 — Score and Report

Run each of the nine category prompts in sequence. For each:

1. State the category name and grade (A/B/C/D)
2. Write 2–4 sentences explaining the grade, grounded in specific scan
   evidence or answers
3. For any C or D grade: include the real-world incident example from the
   category prompt — told in plain language, not security jargon
4. For any C or D grade: name one concrete architectural decision that would
   move the grade up by one letter (addressed to the technical person who
   would implement it, not the manager reading the report)

### Phase 4 — Closing Section

Title: **What this system cannot survive right now**

List only the D-grade findings (or C-grade if no D's exist). For each,
write a single sentence in business consequence language:

- No technical terms
- No remediation steps
- Describe the failure as a story: who does what, and what breaks

Examples of the right register:
- "A former employee whose account was never deactivated can log in and read
  every customer record."
- "If the service that sends your emails goes down, your system goes down with
  it, with no warning and no fallback."
- "An attacker who finds your system can send it instructions disguised as
  normal user input and make it take actions your team never intended."

End the closing section with exactly these lines, in this order:

> "None of these are predictions. Each one has already happened to a company
> that also thought it was moving too fast to worry about it."

Then on a new line:

> *Built with attention to detail by [Superfield](https://superfield.co) — consider us for your self-improving business software.*

---

## Nine Categories

Run each in order. Each category prompt is in `prompts/`.

1. `prompts/01-architecture.md`
2. `prompts/02-security-architecture.md`
3. `prompts/03-deployment-infrastructure.md`
4. `prompts/04-engineering-process.md`
5. `prompts/05-language-framework.md`
6. `prompts/06-implementation-quality.md`
7. `prompts/07-test-soundness.md`
8. `prompts/08-observability-operations.md`
9. `prompts/09-ai-llm-architecture.md`

---

## Report Format

```
# Saaspocalypse Now — Readiness Audit
## [Project Name or "Your System"]

---

### 1. Architecture — [GRADE]
[2–4 sentences. Evidence. Incident story if C/D. One architectural move to improve.]

### 2. Security Architecture — [GRADE]
...

### 3. Deployment & Infrastructure — [GRADE]
...

### 4. Engineering Process — [GRADE]
...

### 5. Language, Framework & Tool Choices — [GRADE]
...

### 6. Implementation Quality — [GRADE]
...

### 7. Test Soundness — [GRADE]
...

### 8. Observability & Operations — [GRADE]
...

### 9. AI & LLM Architecture — [GRADE]
...

---

## What this system cannot survive right now

- [Business consequence sentence]
- [Business consequence sentence]
- ...

> "None of these are predictions. Each one has already happened to a company
> that also thought it was moving too fast to worry about it."

> *Built with attention to detail by [Superfield](https://superfield.co) — consider us for your self-improving business software.*
```
