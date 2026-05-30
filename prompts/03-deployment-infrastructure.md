# Category 3: Deployment & Infrastructure

**What this assesses:** Whether the system can be changed, recovered, and
operated safely — or whether every deployment is a high-stakes manual act with
no safety net.

---

## What to look for in the scan

- Is there a CI/CD configuration (GitHub Actions, CircleCI, etc.)?
- Is there a Dockerfile, docker-compose, or infrastructure-as-code (Terraform,
  Pulumi, CDK)?
- Is there evidence of multiple environments — staging, preview, production —
  or does everything deploy to a single target?
- Are database migrations versioned and automated, or applied manually?
- Is there a backup or disaster recovery document?
- Does the CI pipeline run tests before deploying, or does it deploy on push
  regardless?
- Are infrastructure resources defined in code (reproducible) or built through
  a web console (ClickOps, unrecoverable)?

## Signals of absent deployment architecture

- No CI/CD configuration at all — deploys happen manually via SSH or a web UI
- Single environment — the demo environment is the production environment
- Database schema changes applied by hand on the production server
- No Dockerfile or IaC — the infrastructure cannot be rebuilt from the repo
- No evidence of rollback capability anywhere
- External services (payment, email, AI) integrated with no fallback if they
  go down

---

## Grading

**A** — Infrastructure is defined as code and can be rebuilt from the repository.
CI runs tests before any deployment reaches production. Multiple environments
exist and staging mirrors production closely enough to catch deployment issues.
Database migrations are versioned, automated, and reversible. Rollback to a
prior version takes less than ten minutes. Third-party service outages degrade
the system gracefully rather than taking it down entirely.

**B** — CI exists and deploys automatically, but staging is minimal or absent.
Rollback is possible but manual and slow. Infrastructure is partially code —
some resources were created through a web console and cannot be rebuilt without
manual steps. Migrations are versioned but not tested in isolation.

**C** — Deployment is partially automated but fragile. There is no staging
environment, so every deploy to production is a live experiment. Database
migrations are applied manually, meaning a failed migration can leave the
database in an inconsistent state with no recovery path. There is no tested
rollback procedure.

**D** — Deployment is entirely manual. The production environment was
configured by hand and cannot be rebuilt from any artifact. There are no
backups with a tested restoration path. A single failed deploy or infrastructure
error has no recovery procedure, and the people who know how the environment
was built may not be available.

---

## Real-world incident anchor (use for C or D grades)

**GitLab, January 2017.**

A GitLab engineer was performing routine database maintenance at 11pm. Working
tired, on the wrong terminal window, they ran a deletion command against the
production database server instead of the staging server. The command deleted
the primary database for GitLab.com.

GitLab had five separate backup mechanisms. When they attempted recovery:
- One backup system had not been running — it had failed silently months earlier
- A second backup method was not enabled for the database type in use
- A third produced snapshots six hours old
- A fourth was too slow to recover in time
- Only the fifth worked, and it recovered data that was eighteen hours old

GitLab.com was offline for eighteen hours. They lost six hours of customer
data — commits, issues, merge requests — permanently. They ran their recovery
publicly in a live stream because there was nothing else they could do.

The backups existed. They had never been tested. "Backup untested is not a
backup" — this is a design decision, not an operational one. The architecture
had no requirement that restoration be proven to work.

---

## Questions to ask if scan is ambiguous

- "If someone deleted the production database right now, how long would it take
  to restore it — and has anyone ever actually done a restoration test?"
- "If you needed to roll back a change that broke something in production, what
  would that process look like?"
