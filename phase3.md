# PHASE 3: SCALE
## Months 7-12 — Teams, Enterprise & Optional Web UI

**Goal:** Support teams, multiple projects, advanced governance, custom tool building, and an optional web UI for those who want it. Scale to 100+ agents, enterprise-ready.

---

## 1. Scope

### 1.1 In Scope
- Multi-project support (isolated agents + memory per project)
- Team collaboration (shared agents, RBAC, approval workflows)
- Enterprise SSO (SAML, OIDC) — for teams that need it
- Custom tool builder (YAML-based workflows, not visual)
- Advanced analytics (CLI + optional web dashboard)
- Optional web UI (thin frontend over existing API — for non-CLI users)
- 100+ agents per project
- Pricing tiers

### 1.2 Out of Scope
- Self-hosted deployment
- On-premise option
- Custom LLM hosting
- Advanced AI research features (self-improving agents)

---

## 2. New Features

### 2.1 Multi-Project Support

**Structure:**
```
~/.agents/
├── config.yaml              # Global config (auth, API keys)
├── projects/
│   ├── my-saas-app/
│   │   ├── project.yaml     # Project config
│   │   ├── agents/          # Agent configs (YAML)
│   │   └── .memory/         # Local memory cache
│   ├── mobile-app/
│   │   ├── project.yaml
│   │   ├── agents/
│   │   └── .memory/
│   └── client-website/
│       ├── project.yaml
│       ├── agents/
│       └── .memory/
```

**Isolation:**
- Each project has its own supermemory namespace (Pinecone + PostgreSQL)
- Agents belong to one project (no cross-contamination)
- Shared templates across projects (from marketplace or local)
- Separate cost tracking per project

**CLI:**
```
$ agents project list

  ┌────┬─────────────────┬──────────┬────────┬──────────┐
  │ #  │ Project         │ Agents   │ Status │ Cost/wk  │
  ├────┼─────────────────┼──────────┼────────┼──────────┤
  │ 1  │ my-saas-app     │ 8 agents │ active │ $12.34   │
  │ 2  │ mobile-app      │ 5 agents │ active │ $6.78    │
  │ 3  │ client-website  │ 3 agents │ paused │ $0.00    │
  └────┴─────────────────┴──────────┴────────┴──────────┘

$ agents project switch mobile-app
  ✓ Switched to mobile-app

$ agents project create new-project --repo github.com/user/repo
  ✓ Project created. Run: agents init
```

### 2.2 Team Collaboration

**Invite team members to projects:**
```
$ agents team invite marcus@email.com --role admin
$ agents team invite sarah@email.com --role member
$ agents team list

  ┌──────────────┬────────┬────────────┬─────────────┐
  │ Member       │ Role   │ Joined     │ Last Active │
  ├──────────────┼────────┼────────────┼─────────────┤
  │ you          │ owner  │ 2026-01-15 │ now         │
  │ marcus       │ admin  │ 2026-02-01 │ 2h ago      │
  │ sarah        │ member │ 2026-02-15 │ 1d ago      │
  └──────────────┴────────┴────────────┴─────────────┘
```

**Roles & Permissions:**

| Permission | Owner | Admin | Member | Viewer |
|------------|-------|-------|--------|--------|
| Create/delete agents | ✅ | ✅ | ✅ | ❌ |
| Start/stop agents | ✅ | ✅ | ✅ | ❌ |
| Merge PRs | ✅ | ✅ | ❌ | ❌ |
| Project settings | ✅ | ✅ | ❌ | ❌ |
| Invite members | ✅ | ✅ | ❌ | ❌ |
| View status/logs | ✅ | ✅ | ✅ | ✅ |
| Billing | ✅ | ❌ | ❌ | ❌ |

**Approval Workflows:**
```yaml
# project.yaml
approval_rules:
  pr_merge:
    required_approvals: 1        # At least 1 human approval
    allowed_roles: [owner, admin]
  agent_create:
    required_approvals: 0        # Anyone can create agents
  dangerous_actions:             # git force push, delete branch, etc.
    required_approvals: 1
    allowed_roles: [owner]
```

**How It Works:**
```
Agent BE-1 wants to merge PR #42
       │
       ▼
Check approval_rules → pr_merge requires 1 human approval
       │
       ▼
Notify team members (terminal + Slack):
  "🔔 PR #42 ready for merge. Approve: agents pr approve 42"
       │
       ▼
Admin Marcus runs: agents pr approve 42
       │
       ▼
PR merged ✓
```

**Shared Agent Pools:**
- Team members can start/use any project agent from their own terminal
- Agents are project-scoped, not user-scoped
- Multiple team members can watch same roundtable

### 2.3 Enterprise SSO

**For teams that need corporate auth:**
```
$ agents auth setup-sso --provider okta --domain company.com
$ agents auth setup-sso --provider google-workspace
$ agents auth setup-sso --provider azure-ad
```

**Supported:**
- SAML 2.0 (Okta, Azure AD, OneLogin)
- OIDC (Google Workspace, Auth0)
- SCIM provisioning (auto-add/remove users)

**Features:**
- Auto-provision users when they join org
- Group/role mapping (Okta groups → project roles)
- Session management
- Force SSO (disable password login)
- Audit logging of all auth events

### 2.4 Custom Tool Builder

**YAML-based workflows (no visual editor needed in CLI):**

```yaml
# tools/jira-ticket-from-pr.yaml
name: Create Jira Ticket from PR
description: Auto-creates Jira ticket when agent opens PR
trigger:
  event: pr_created
  conditions:
    - branch_prefix: "feature/*"

steps:
  - name: Extract PR info
    action: extract
    fields:
      title: "{{ pr.title }}"
      description: "{{ pr.body }}"
      agent: "{{ pr.author }}"

  - name: Create Jira ticket
    action: http_post
    url: "https://company.atlassian.net/rest/api/3/issue"
    headers:
      Authorization: "Basic {{ secrets.JIRA_TOKEN }}"
    body:
      fields:
        project: { key: "DEV" }
        summary: "{{ steps.extract.title }}"
        description: "{{ steps.extract.description }}"
        issuetype: { name: "Task" }

  - name: Comment on PR
    action: github_comment
    pr: "{{ pr.number }}"
    body: "Jira ticket created: {{ steps.create_jira.response.key }}"
```

**CLI:**
```
$ agents tools create ./tools/jira-ticket-from-pr.yaml
  ✓ Tool registered: jira-ticket-from-pr

$ agents tools list
  ┌────┬──────────────────────┬──────────┬─────────┐
  │ #  │ Tool                 │ Trigger  │ Status  │
  ├────┼──────────────────────┼──────────┼─────────┤
  │ 1  │ jira-ticket-from-pr  │ pr_created│ active │
  │ 2  │ slack-deploy-notify  │ pr_merged │ active │
  │ 3  │ db-migration-runner  │ manual   │ active │
  └────┴──────────────────────┴──────────┴─────────┘

$ agents tools run db-migration-runner
  Running...
  ✓ Migration completed. 2 tables created.

$ agents tools publish jira-ticket-from-pr
  ✓ Published to marketplace.
```

**Tool Components:**
- **Triggers:** pr_created, pr_merged, task_completed, roundtable_concluded, manual, schedule (cron)
- **Actions:** http_post/get, github_comment, shell_exec, memory_write, agent_notify, slack_webhook
- **Variables:** Template syntax `{{ }}` with access to event data, secrets, previous step outputs

### 2.5 Advanced Analytics

**CLI Reports:**
```
$ agents analytics --period month

  ══════════════════════════════════════════════════
  MONTHLY REPORT — my-saas-app — March 2026
  ══════════════════════════════════════════════════

  EXECUTIVE SUMMARY
  ├── Tasks completed:     156
  ├── PRs merged:          34
  ├── Roundtables held:    12 (7 auto, 5 manual)
  ├── Avg confidence:      84%
  ├── Total cost:          $47.23
  └── Est. time saved:     ~120 hours

  AGENT PERFORMANCE
  ┌──────────┬───────┬────────────┬──────────┬──────────┐
  │ Agent    │ Tasks │ Confidence │ Errors   │ Cost     │
  ├──────────┼───────┼────────────┼──────────┼──────────┤
  │ CTO      │ 15    │ 91%        │ 0        │ $5.12    │
  │ BE-1     │ 42    │ 83%        │ 2        │ $12.34   │
  │ BE-2     │ 38    │ 81%        │ 1        │ $10.98   │
  │ FE-1     │ 35    │ 85%        │ 1        │ $9.45    │
  │ QA       │ 22    │ 88%        │ 0        │ $6.12    │
  │ Security │ 4     │ 92%        │ 0        │ $3.22    │
  └──────────┴───────┴────────────┴──────────┴──────────┘

  QUALITY METRICS
  ├── Hallucination rate:  0.8%
  ├── Bug detection rate:  94%
  ├── Security issues caught: 7
  └── Code review accuracy: 91%

$ agents analytics export --format csv --output report.csv
$ agents analytics export --format json --output report.json
```

### 2.6 Optional Web UI

**For users who want a visual interface — thin layer over existing CLI API:**

This is NOT a full web rewrite. The backend API already exists from Phase 1-2. This is a lightweight Next.js frontend that calls the same API.

**What the web UI provides:**
- Visual agent status dashboard
- Roundtable viewer (easier to read than terminal)
- Memory browser (search and explore supermemory)
- Analytics charts
- Team management UI
- PR review interface

**What stays CLI-only:**
- Agent execution (still local terminal processes)
- Sandbox execution (Docker, local)
- Git operations

**Architecture:**
```
┌──────────────────────────────────┐
│  Optional Web UI (Next.js)       │  ← NEW: thin frontend
│  - Dashboard, roundtables, etc.  │
└──────────────┬───────────────────┘
               │ Same API
┌──────────────┴───────────────────┐
│  Backend API (already exists)    │  ← From Phase 1-2
│  Agent orchestration, memory,    │
│  tasks, GitHub, etc.             │
└──────────────────────────────────┘
               │
┌──────────────┴───────────────────┐
│  Local Terminal Agents           │  ← Still local
└──────────────────────────────────┘
```

**Deployment:** Self-hosted (user runs `agents web start` — spins up local Next.js on localhost:3000) or hosted option for teams.

---

## 3. Infrastructure Scaling

### 3.1 Architecture (Still Mostly Local + Free Tier)

```
Local Machine
├── CLI Tool (agents)
├── Agent Processes (terminal)
├── Docker Sandbox
└── Optional Web UI (localhost:3000)

Cloud (Free/Cheap Tier)
├── Supabase PostgreSQL (agents, tasks, checkpoints, teams)
├── Pinecone (vector memory, namespaces per project)
├── Upstash Redis (queue, cache, pub/sub)
├── Neo4j Aura Free (graph memory)
└── Optional: Vercel (web UI hosting for teams)
```

### 3.2 Performance Targets

| Metric | Target |
|--------|--------|
| Agents per project | 100+ |
| Agent response (p95) | < 10s |
| Memory query (p95) | < 200ms |
| CLI command | < 500ms |
| Concurrent team members | 20+ |
| Sandbox execution | < 30s |

### 3.3 Disaster Recovery (for team/enterprise use)

- Database: Supabase point-in-time recovery
- Memory: Pinecone managed backups
- Agent state: Checkpoint-based recovery (from Phase 1)
- Git: GitHub is the source of truth for all code

---

## 4. Governance & Compliance

### 4.1 Audit Logging

**Every action logged:**
```
$ agents audit log --period week

  ┌─────────────────────┬──────────┬───────────────────────────┐
  │ Timestamp           │ Actor    │ Action                    │
  ├─────────────────────┼──────────┼───────────────────────────┤
  │ 2026-03-16 10:30    │ BE-1     │ Created PR #42            │
  │ 2026-03-16 10:28    │ marcus   │ Approved PR #41           │
  │ 2026-03-16 10:25    │ CTO      │ Roundtable decision: JWT  │
  │ 2026-03-16 10:20    │ sarah    │ Created agent FE-2        │
  │ 2026-03-16 10:15    │ Security │ Flagged secret in PR #40  │
  └─────────────────────┴──────────┴───────────────────────────┘

$ agents audit export --format csv --from 2026-03-01
  ✓ Exported to audit-2026-03.csv
```

**Logged Events:**
- All agent actions (tasks, PRs, commits, reviews)
- All roundtable decisions
- All human actions (approvals, config changes, member changes)
- All auth events (login, SSO, key rotation)

**Retention:**
- Active storage: 90 days
- Archive: 1 year (exportable)
- Enterprise: custom retention

### 4.2 Data Governance

- Data classification (agents can be restricted from accessing certain repos/files)
- PII detection in agent outputs (flag, don't store)
- Data export (GDPR: export all data for a user)
- Data deletion (GDPR: delete all data for a user)
- Project-level data isolation

### 4.3 Compliance

- SOC 2 readiness (audit logging, RBAC, encryption)
- GDPR compliant (export, deletion, consent)
- HIPAA considerations (for healthcare teams — BAA available on Enterprise)

---

## 5. Pricing Tiers

### Free (Solo Developer)
- 1 project
- 5 agents
- Basic roundtables (manual only)
- Community marketplace
- 50K tokens/day (on platform LLM) or unlimited BYOK
- Community support

### Pro ($29/month — Individual)
- 3 projects
- 20 agents per project
- Auto-roundtables
- Sandbox
- Graph memory
- Slack/Discord notifications
- Cost analytics
- Email support

### Team ($49/user/month)
- Unlimited projects
- 50 agents per project
- Team collaboration (RBAC, approvals)
- Custom tools
- Advanced analytics
- Shared agent pools
- Priority support

### Enterprise (Custom)
- Unlimited everything
- 100+ agents per project
- SSO (SAML, OIDC, SCIM)
- Audit logs (custom retention)
- SLA (99.9% uptime)
- Dedicated support
- Custom contracts
- Compliance certifications

---

## 6. Timeline

### Week 1-4: Multi-Project Support
- Project isolation (separate namespaces, configs)
- Project CRUD CLI commands
- Per-project cost tracking
- Migration of existing single-project to multi-project structure

### Week 5-8: Team Collaboration
- User accounts + team membership (Supabase Auth)
- RBAC system
- Approval workflows
- Shared agent access
- Team CLI commands

### Week 9-12: Enterprise SSO
- SAML 2.0 integration
- OIDC integration
- SCIM provisioning
- Session management

### Week 13-16: Custom Tools + Analytics
- YAML-based tool builder
- Tool execution engine
- Trigger system (event-based + cron)
- Analytics data collection
- CLI analytics reports
- Export functionality

### Week 17-20: Optional Web UI
- Next.js setup (thin frontend)
- Dashboard page
- Roundtable viewer
- Memory browser
- Team management
- `agents web start` command

### Week 21-24: Compliance, Polish & Launch
- Audit logging system
- Data governance features
- Documentation (full docs site)
- Pricing/billing integration (Stripe)
- Enterprise launch
- Marketing

---

## 7. Success Criteria

| Metric | Target |
|--------|--------|
| Teams using platform | 100+ |
| Paid conversions | 15% |
| Avg team size | 5 members |
| Projects per team | 3+ |
| Enterprise customers | 10+ |
| NPS | > 60 |
| Monthly churn | < 5% |
| Uptime (for hosted services) | 99.9% |

---

## 8. Future Considerations (Phase 4+)

- Self-improving agents (learn from roundtable outcomes)
- Predictive task assignment (CTO agent auto-breaks down features)
- Cross-project knowledge sharing (learn patterns from all projects)
- AI agent marketplace (paid agents, revenue sharing)
- Industry-specific templates (fintech, healthcare, e-commerce)
- Self-hosted enterprise option
- Custom LLM support (local models, Ollama)
- IDE plugins (VS Code, JetBrains)

---

**Phase 3 Goal: Platform ready for teams and enterprises — multi-project, RBAC, approval workflows, audit logging, custom tools — with an optional web UI for those who want it. Still CLI-first at its core.**
