# <Project> Server Factbook (template)

**Server:** `<host>` (`<ip>`), SSH alias `<alias>`
**Last full collection:** YYYY-MM-DD HH:MM
**Authority:** Single source of truth for all docker/django/SSH-related tech-specs. Validators (skeptic, architect) MUST read this file before validating tech-spec claims about server state.

## Per-section TTL

| Section | TTL | Reason |
|---|---|---|
| Docker stacks/services | 7 days | Changes after every deploy |
| Application models/apps | 30 days | Schema changes on epics |
| Sudoers, cron, nginx, env names | 30 days | Low churn |
| System tools versions | 90 days | OS-level, rare |

If a section is older than its TTL when validators read it — skeptic blocks the tech-spec with `status: stale, refresh required`.

## Factbook-first rule

Before drafting any tech-spec with docker service names, application model names, sudoers commands, SSH commands, nginx paths, cron jobs, PHP-FPM, Apache, or host-level services — **read this factbook first** and quote exact values from it. Do not write tech-spec claims "from memory" — that's how hallucination-class findings are born.

## Service enumeration first (the source of factbook scope)

Before populating sections below, enumerate ALL listening services on the host. This is the source of factbook scope, not "what I remember exists":

```bash
ssh <server> "ss -tlnp 2>/dev/null"
ssh <server> "systemctl list-units --type=service --state=running --no-pager"
```

From this enumeration → derive section list → collect each section.

## Section template (one per service class)

```markdown
## N. <Service class>

**Last verified:** YYYY-MM-DD HH:MM
**TTL:** <7|30|90> days

<facts — names, versions, ports, paths, env vars (names only, redacted values), configurations>
```

## Mandatory sections (adapt to your stack)

1. Docker Stacks (services with replicas, images)
2. Compose Files (what's in each)
3. Application Apps Inventory (Django apps / Rails apps / etc)
4. Active Session / Active User Domain (for active-session gate in deploy)
5. Existing CI/CD Workflows
6. Git Checkouts on Server (remotes, branches, HEAD)
7. Sudoers + system tools
8. Nginx / Apache vhosts
9. Cron jobs (root + cron.d)
10. Env Variables (NAMES ONLY, redact values)
11. System Tools (versions + paths)
12. External Integrations (payment gateways, video, OAuth, CRM)
13. Other host-level services (PHP-FPM, Apache, host-nginx, etc.)
14. Appendix — raw collection outputs (for audit)

## Findings format

Append findings discovered during collection as a table at the end:

| ID | Severity | Finding |
|---|---|---|
| <PROJECT>-Fxxx | P0/P1/P2 | <description> |
