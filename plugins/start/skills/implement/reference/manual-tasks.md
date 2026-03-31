# Manual Tasks Protocol

How to detect, document, and gate on tasks that require human action outside the codebase.

---

## What Counts as a Manual Task

A manual task is any action that:
- **Cannot be performed by the agent** — requires human access, credentials, or a browser session
- **Is required for the feature to work** — not optional or nice-to-have
- **Happens outside the codebase** — in a dashboard, admin panel, DNS provider, etc.

### Common Categories

| Category | Examples |
|----------|----------|
| **API Keys & Secrets** | Create Stripe API key, generate OAuth client ID, provision AWS access key |
| **Third-Party Config** | Configure webhook URL in provider dashboard, set OAuth redirect URIs, enable API features |
| **Infrastructure** | Set up DNS records, create S3 bucket, provision database, configure CDN |
| **Service Accounts** | Create service account in GCP/AWS, set up bot user in Slack/Discord |
| **Permissions** | Grant team member access, configure RBAC roles, approve app in marketplace |
| **External Verification** | Verify domain ownership, submit app for review, complete KYC |
| **Environment Config** | Add secrets to CI/CD, configure environment variables in hosting platform |

### NOT Manual Tasks

- Installing npm packages (agent can do this)
- Writing config files (agent can do this)
- Running migrations (agent can do this)
- Creating `.env.example` templates (agent can do this)

---

## Detection

Agents should flag a manual task when they encounter:

| Signal | Where Found | Example |
|--------|-------------|---------|
| Placeholder secrets in code | `.env.example`, config files | `STRIPE_SECRET_KEY=sk_test_...` |
| SDK initialization requiring keys | Implementation code | `new Stripe(process.env.STRIPE_SECRET_KEY)` |
| OAuth/auth provider setup | SDD, implementation | `AUTH0_DOMAIN`, `GOOGLE_CLIENT_ID` |
| Webhook registration | SDD, implementation | "Register webhook at provider dashboard" |
| DNS/domain requirements | SDD, deployment config | CNAME records, TXT verification |
| External service dependencies | PRD, SDD | "Integrate with Twilio for SMS" |
| CI/CD secret references | Workflow files | `secrets.DEPLOY_KEY` in GitHub Actions |
| Infrastructure provisioning | SDD | "Requires Redis instance", "S3 bucket for uploads" |

### When to Detect

1. **During plan creation** (`specify-plan`) — read SDD for external dependencies, flag known manual tasks
2. **During implementation** (`implement`) — as agents write code that references external services, flag new manual tasks
3. **During testing** (`test`) — when tests fail due to missing credentials or external service config

---

## MANUAL_TASKS.md Format

Location: `.start/specs/[NNN]-[name]/MANUAL_TASKS.md`

```markdown
---
spec: "[NNN]-[name]"
status: incomplete | complete
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Manual Tasks

Tasks that require human action before this feature can be committed and deployed.

**Status**: {N} of {M} complete

## Tasks

### 1. [Short title]

- **Category**: API Keys & Secrets | Third-Party Config | Infrastructure | ...
- **Required by**: Phase {N}, Task {T.id} | Deployment | Testing
- **Status**: [ ] Pending | [x] Complete
- **Blocking**: Yes — cannot commit without this

**What to do:**
1. Step-by-step instructions with specific URLs, dashboard paths, button names
2. Each step should be concrete and actionable
3. Include screenshots paths or links if helpful

**Where to put the result:**
- Add `STRIPE_SECRET_KEY` to `.env` (see `.env.example` for format)
- Or: Add to CI/CD secrets in GitHub Settings → Secrets → Actions

**How to verify it worked:**
- Run `curl -s https://api.stripe.com/v1/charges -u $STRIPE_SECRET_KEY:` — should return `{"data":[]}`
- Or: Run `npm test -- --grep "stripe"` — should pass

**Research notes:**
Agent's research on this task — links to docs, gotchas, pricing tiers, etc.

---

### 2. [Next task...]

...
```

### Key Principles

1. **Research everything** — don't just say "create an API key". Link to the exact dashboard URL, describe which plan/tier is needed, note any gotchas.
2. **Verification steps** — every task must have a "how to verify" section so the human knows it worked.
3. **Where to put it** — tell the human exactly where the output (key, config value, etc.) needs to go.
4. **Blocking flag** — all tasks are blocking by default. Non-blocking tasks don't belong in this file.

---

## Commit Gate

### How It Works

Before any commit during implementation:

1. Check if `.start/specs/[NNN]-[name]/MANUAL_TASKS.md` exists.
2. Parse the task list — count `- [ ]` (pending) and `- [x]` (complete) items.
3. If any tasks are pending:

```
match (pending tasks) {
  > 0 => BLOCK COMMIT
         Present pending tasks to user.
         AskUserQuestion:
           "I've completed them — mark as done" => walk through each, mark [x], re-check
           "Show me the steps" => display the detailed steps for pending tasks
           "Defer this task (remove gate)" => requires explicit justification, logged in file
}
```

4. If all tasks complete: proceed with commit.

### Defer Protocol

If a human defers a manual task, it must be logged:

```markdown
### 3. [Deferred task title]

- **Status**: [~] Deferred
- **Deferred by**: [user] on YYYY-MM-DD
- **Reason**: "Will configure in staging, not needed for local dev"
- **Blocking**: No (deferred)
```

Deferred tasks don't block the commit but remain visible. The `status` frontmatter stays `incomplete` until all tasks are complete or deferred.

### Gate Check Points

The manual task gate runs at:
1. **Phase completion** — before marking a phase as complete in implement
2. **Final commit** — before the implement skill offers to commit/PR
3. **Test suite** — test skill surfaces pending manual tasks in the final report

---

## Integration with Spec Workflow

### specify-plan

When creating the implementation plan, scan the SDD for external dependencies. For each one:
- Create a preliminary entry in MANUAL_TASKS.md with category and "Required by" reference
- Mark as `[ ] Pending` with placeholder steps
- Note: "Detailed steps will be researched during implementation"

### implement

During phase execution:
1. **Before phase starts** — check for pending manual tasks that block this phase
2. **During task execution** — when agents write code referencing external services, research and add/update MANUAL_TASKS.md entries with full step-by-step instructions
3. **After phase completes** — run gate check before marking phase complete

### test

During test execution:
- If tests fail with credential/connection errors that map to pending manual tasks, surface the connection:
  "Test `stripe.test.ts:23` failed — this requires Manual Task #1 (Create Stripe API key) to be completed first."

### validate

During validation:
- Check for completeness: every external dependency in the SDD should have a corresponding manual task entry
- Check for staleness: tasks created during planning but never updated with research during implementation
