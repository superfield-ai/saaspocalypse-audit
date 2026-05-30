# Saaspocalypse Audit — Agent Bootstrap

You are installing the `saaspocalypse-audit` skill for Claude Code.

## Step 1 — Determine install location

**Global install** (available in all projects on this machine):
```bash
mkdir -p ~/.claude/skills/saaspocalypse-audit
```

**Project install** (scoped to the current project only):
```bash
mkdir -p .claude/skills/saaspocalypse-audit
```

Default to **global** unless the user has specified project-only.

## Step 2 — Download and install

```bash
curl -fsSL https://github.com/superfield-ai/saaspocalypse-audit/archive/refs/heads/main.tar.gz \
  | tar -xz --strip-components=1 -C ~/.claude/skills/saaspocalypse-audit
```

For project install, replace `~/.claude/skills/` with `.claude/skills/`.

## Step 3 — Confirm

```bash
cat ~/.claude/skills/saaspocalypse-audit/VERSION
```

Tell the user: "saaspocalypse-audit vX.X.X installed. Run `/saaspocalypse-audit` from any project directory, or `/saaspocalypse-audit <git-url>` to audit a remote repo."
