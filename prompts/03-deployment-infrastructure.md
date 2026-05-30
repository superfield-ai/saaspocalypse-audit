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

## Real-world incident anchors (use for C or D grades)

---

### GitLab, January 2017 — Eighteen hours of downtime and six hours of permanent data loss because five backup systems had never been tested.

**The design error:** No architectural requirement that backup restoration be
proven to work. The system had five separate backup mechanisms and the team
believed they were covered. Not one mechanism had a tested, documented
restoration path.

**Why vibe-coders make the same mistake:** AI-assisted builders follow prompts
like "set up automated backups" and treat the presence of a backup job as the
feature being done. Testing restoration is a separate, unglamorous task that
doesn't show up in a demo and never makes it into the MVP scope.

**What happened:** A GitLab engineer was performing routine database maintenance
late at night. Working tired, on the wrong terminal window, they ran a deletion
command against the production database instead of the staging server. GitLab.com
went down. When the team attempted recovery, one backup system had silently
failed months earlier and no one had noticed. A second method was not enabled
for the database type in use. A third produced snapshots that were six hours
old. A fourth was too slow to be usable. Only the fifth worked, and it had data
that was eighteen hours old. GitLab.com was offline for eighteen hours. Six
hours of customer commits, issues, and merge requests were gone permanently.
The engineers ran the recovery as a public live stream because there was nothing
else to do.

---

### Capital One, July 2019 — A misconfigured firewall exposed AWS credentials with excessive permissions, leading to the exfiltration of 100 million customer records.

**The design error:** IAM roles were granted far broader permissions than the
workload required. The web application server needed to do a narrow set of
things; its IAM role gave it the ability to list and download from S3 buckets
across the account. When the server was compromised, that over-permission became
the attacker's entire toolkit.

**Why vibe-coders make the same mistake:** When getting cloud infrastructure
working, the path of least resistance is to grant broad permissions — often
AdministratorAccess or a wildcard S3 policy — to stop getting access-denied
errors during development. AI code assistants frequently suggest permissive IAM
policies because they produce working code faster. Tightening permissions to
the minimum required is a separate hardening pass that never gets scheduled.

**What happened:** An attacker exploited a misconfigured web application
firewall to send a server-side request forgery (SSRF) request — essentially
tricking the server into making an internal HTTP request to the AWS metadata
service. That service returned the temporary IAM credentials attached to the
server's role. Because those credentials had been granted the ability to list
and download from S3 buckets across the AWS account, the attacker spent hours
downloading over 100 million customer records including names, addresses, credit
scores, and Social Security numbers. A proper least-privilege policy — one
scoped to exactly the buckets and actions the web server legitimately needed —
would have made those credentials useless.

---

### Cloudflare, November 2023 — Credentials that survived a known breach were used to access internal source code repositories weeks later.

**The design error:** No credential rotation policy. An access token issued at
one point in time was assumed valid indefinitely. When the upstream identity
provider (Okta) was compromised and the compromise became public, there was no
mechanism — and no process — to systematically invalidate tokens that had been
issued through it.

**Why vibe-coders make the same mistake:** Rotating credentials and API tokens
is maintenance work that delivers no new features. AI-assisted developers set
up integrations once, use long-lived tokens because they are simpler to manage,
and move on. The token is never mentioned in the codebase again until something
goes wrong. There is also no natural moment in a build cycle that prompts the
question "what tokens have we issued that might be compromised upstream?"

**What happened:** Attackers had previously compromised Okta, the identity
provider used by many companies including Cloudflare. Cloudflare was aware of
the Okta breach. However, an access token that had been issued via Okta was
never rotated after the breach was disclosed. Weeks later, on Thanksgiving, the
attackers used that token to authenticate into Cloudflare's internal Atlassian
environment and access source code repositories. The entry point was not a
sophisticated attack on Cloudflare's systems — it was a key that should have
been revoked and was not.

---

## Questions to ask if scan is ambiguous

- "If someone deleted the production database right now, how long would it take
  to restore it — and has anyone ever actually done a restoration test?"
- "If you needed to roll back a change that broke something in production, what
  would that process look like?"
