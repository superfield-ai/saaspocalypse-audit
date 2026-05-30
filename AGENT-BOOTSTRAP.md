# Saaspocalypse Audit — Agent Bootstrap

You are installing the `saaspocalypse-audit` skill for Claude Code.

## Step 1 — Ask the user where to install

Before installing, ask exactly this question:

> "Install saaspocalypse-audit for this project only, or globally for all projects on this machine?
> - **Project** (default) — installs into `.claude/skills/` in the current directory
> - **Global** — installs into `~/.claude/skills/` and is available everywhere"

Wait for the answer. Default to **project** if the user says nothing or is unsure.

## Step 2 — Fetch the latest release tag

```bash
SAAS_VERSION=$(curl -fsSL https://api.github.com/repos/superfield-ai/saaspocalypse-audit/releases/latest \
  | grep '"tag_name"' | cut -d'"' -f4)
echo "Latest release: $SAAS_VERSION"
```

If this returns empty (no network, API limit), stop and tell the user:
> "Could not reach GitHub to determine the latest release. Check your connection and try again."

## Step 3 — Download and install

For **project install**:
```bash
mkdir -p .claude/skills/saaspocalypse-audit && \
curl -fsSL https://github.com/superfield-ai/saaspocalypse-audit/archive/refs/tags/${SAAS_VERSION}.tar.gz \
  | tar -xz --strip-components=1 -C .claude/skills/saaspocalypse-audit
```

For **global install**:
```bash
mkdir -p ~/.claude/skills/saaspocalypse-audit && \
curl -fsSL https://github.com/superfield-ai/saaspocalypse-audit/archive/refs/tags/${SAAS_VERSION}.tar.gz \
  | tar -xz --strip-components=1 -C ~/.claude/skills/saaspocalypse-audit
```

## Step 4 — Confirm

```bash
# Project install
cat .claude/skills/saaspocalypse-audit/VERSION

# Global install
cat ~/.claude/skills/saaspocalypse-audit/VERSION
```

Tell the user: "saaspocalypse-audit ${SAAS_VERSION} installed. Run `/saaspocalypse-audit` from any project directory, or `/saaspocalypse-audit <git-url>` to audit a remote repo."
