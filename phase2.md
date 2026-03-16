# PHASE 2: ENHANCED
## Months 4-6 — Intelligence & Automation

**Goal:** Add auto-roundtables, agent marketplace, code sandbox, graph memory, and Slack/Discord notifications — all CLI-first. Agents scale to 50+ per project.

---

## 1. Scope

### 1.1 In Scope
- Auto-triggered roundtables (confidence-based)
- Agent marketplace (community templates, CLI-based)
- Code execution sandbox (Docker, local)
- Graph memory (Neo4j for entity relationships)
- Advanced monitoring (CLI dashboards, cost analytics)
- Slack/Discord notifications (webhook-based, not full bots)
- Support for 50+ agents per project
- Performance optimization (<10s agent response)

### 1.2 Out of Scope
- Web dashboard / frontend
- Multi-project support
- Team collaboration features
- Enterprise SSO
- Mobile app
- Self-improving agents

---

## 2. New Features

### 2.1 Auto-Roundtables

**Automatic Triggers (no human initiation needed):**

| Trigger | Condition | Example |
|---------|-----------|---------|
| Low confidence | Agent score < 70% | "I'm only 60% sure about this API design" |
| Conflict | Two agents disagree | BE-1 says REST, BE-2 says GraphQL |
| Missing info | Required context unavailable | "Can't find database schema" |
| Same file | Two agents touching same file | BE-1 and FE-1 both editing `config.ts` |

**Auto-Participant Selection:**
```
Trigger detected (e.g., BE-1 confidence 55% on auth design)
       │
       ▼
Analyze topic via embeddings
       │
       ▼
Find relevant expert agents:
  - CTO (always included for architecture)
  - Security (auth-related topic)
  - BE-2 (same domain expertise)
       │
       ▼
Auto-start roundtable in background
       │
       ▼
Notify user in terminal:
  ⚠ Auto-roundtable started: "Auth Token Strategy"
  Participants: CTO, Security, BE-1, BE-2
  Run: roundtable watch rt-042
```

**CLI:**
```
$ agents config auto-roundtable --threshold 70 --notify terminal
$ roundtable list --auto       # Show auto-triggered roundtables
$ roundtable watch rt-042      # Watch auto-roundtable live
```

### 2.2 Agent Marketplace

**Template Registry (CLI-based, no web UI):**
```
$ agents marketplace search "react developer"

  ┌────┬─────────────────────┬──────────┬───────┬──────────┐
  │ #  │ Template            │ Author   │ ⭐    │ Downloads│
  ├────┼─────────────────────┼──────────┼───────┼──────────┤
  │ 1  │ React Frontend Dev  │ @sarah   │ 4.8   │ 342      │
  │ 2  │ Next.js Fullstack   │ @marcus  │ 4.5   │ 218      │
  │ 3  │ React Native Mobile │ @dev-co  │ 4.2   │ 156      │
  └────┴─────────────────────┴──────────┴───────┴──────────┘

$ agents marketplace install react-frontend-dev
  ✓ Template installed. Create agent: agents create fe-1 --template react-frontend-dev

$ agents marketplace publish ./agents/my-custom-agent.yaml
  ✓ Template submitted for review.
```

**Template Categories:**
- Software Development (React, Node, Python, Go, Rust)
- DevOps (Docker, K8s, CI/CD, Terraform)
- Data Engineering (SQL, ETL, Analytics)
- Content (Technical Writer, Documentation, SEO)
- Business (PM, Marketing, Sales)

**Template Format:**
```yaml
# marketplace template
name: React Frontend Developer
description: Specializes in React/Next.js with TailwindCSS
author: "@sarah"
version: 1.2.0
tags: [react, frontend, nextjs, tailwind]
model: claude-sonnet-4
confidence_threshold: 70
tools: [github, shell, file_system]
system_prompt: |
  You are a senior React frontend developer...
  CONVENTIONS:
  - Use functional components with hooks
  - TailwindCSS for styling
  - React Query for data fetching
  ...
```

### 2.3 Code Execution Sandbox

**Purpose:** Agents test code locally before committing. No cloud needed.

**Architecture (Docker-based, runs locally):**
```
Agent writes code
       │
       ▼
┌─────────────────┐
│ Sandbox Manager  │  (manages Docker containers)
└─────────────────┘
       │
       ▼
┌─────────────────┐
│ Docker Container │  (isolated, time-limited)
│ - Mount code     │
│ - Run tests      │
│ - Capture output │
└─────────────────┘
       │
       ▼
Results back to agent:
  ✓ Tests passed (12/12)
  ✗ 2 lint errors found
  → Agent fixes and retries
```

**CLI:**
```
$ agents sandbox run be-1 --cmd "npm test"
  Running in isolated container...
  ✓ 12/12 tests passed
  Duration: 4.2s

$ agents sandbox status
  Active: 1 container (BE-1)
  Idle: 0
  Limit: 5 minutes per execution
```

**Features:**
- Docker containers (no Firecracker complexity — save that for Phase 3)
- Pre-built images: Node 20, Python 3.11, Go 1.21, Rust 1.75
- 5-minute execution limit
- No outbound network (security)
- Auto-cleanup after execution
- Agents auto-run sandbox before creating PR

**Supported Languages:**
- JavaScript/TypeScript (Node 20)
- Python (3.11)
- Go (1.21)
- Rust (1.75)
- Java (17)

### 2.4 Graph Memory (Neo4j)

**Why:** Supermemory vector search finds similar content, but graph memory tracks *relationships* — which agent modified which file, which decision affects which code, which files depend on each other.

**Schema:**
```cypher
// Nodes
(Agent {agent_id, name, role})
(File {path, type, content_hash})
(Decision {decision_id, topic, outcome, roundtable_id})
(Task {task_id, type, status})

// Relationships
(Agent)-[:CREATED]->(Decision)
(Agent)-[:WORKED_ON]->(Task)
(Task)-[:MODIFIES]->(File)
(Decision)-[:AFFECTS]->(File)
(File)-[:DEPENDS_ON]->(File)
(Agent)-[:REVIEWED]->(Task)
(Decision)-[:SUPERSEDES]->(Decision)
```

**CLI:**
```
$ agents memory graph "auth.ts"

  auth.ts
  ├── Modified by: BE-1 (3 times), Security (1 fix)
  ├── Depends on: jwt-utils.ts, config.ts
  ├── Depended on by: middleware.ts, routes/login.ts
  ├── Decisions affecting:
  │   └── RT-005: "Use JWT with refresh tokens"
  └── Related tasks: AUTH-001, AUTH-003, AUTH-007

$ agents memory graph --agent be-1

  BE-1
  ├── Files modified: auth.ts, user.ts, db.ts (12 total)
  ├── Decisions participated: 5 roundtables
  ├── Tasks completed: 23
  └── Collaborates most with: CTO, Security
```

**Benefits:**
- "What files are affected if I change this?" → Graph traversal
- "Who worked on auth?" → Agent-file relationships
- "What decisions led to this code?" → Decision-code tracing
- Smarter context injection for agents

### 2.5 Advanced Monitoring (CLI Dashboard)

**Live Status Dashboard:**
```
$ agents dashboard

  ╔══════════════════════════════════════════════════════════╗
  ║  PROJECT: my-saas-app          Cost Today: $2.34        ║
  ╠══════════════════════════════════════════════════════════╣
  ║                                                          ║
  ║  AGENTS                                                  ║
  ║  ┌────────┬──────────────┬──────────┬───────────────┐    ║
  ║  │ Agent  │ Role         │ Status   │ Current Task  │    ║
  ║  ├────────┼──────────────┼──────────┼───────────────┤    ║
  ║  │ CTO    │ Architect    │ 💤 idle  │ —             │    ║
  ║  │ BE-1   │ Backend Dev  │ ▶ active │ Auth API      │    ║
  ║  │ BE-2   │ Backend Dev  │ ▶ active │ Payment svc   │    ║
  ║  │ FE-1   │ Frontend Dev │ 💤 idle  │ Waiting on API│    ║
  ║  │ QA     │ Tester       │ 💤 idle  │ Waiting PR    │    ║
  ║  │ SEC    │ Security     │ 💤 idle  │ Waiting PR    │    ║
  ║  └────────┴──────────────┴──────────┴───────────────┘    ║
  ║                                                          ║
  ║  ROUNDTABLES                                             ║
  ║  🔵 RT-042: Auth Token Strategy (auto, 3 participants)  ║
  ║                                                          ║
  ║  TODAY'S STATS                                           ║
  ║  Tasks: 12 done, 3 pending │ PRs: 4 open, 2 merged     ║
  ║  LLM calls: 87 │ Tokens: 124K │ Avg confidence: 82%    ║
  ╚══════════════════════════════════════════════════════════╝
```

**Cost Analytics:**
```
$ agents cost --period week

  ┌──────────┬──────────┬──────────┬──────────┐
  │ Agent    │ Calls    │ Tokens   │ Cost     │
  ├──────────┼──────────┼──────────┼──────────┤
  │ BE-1     │ 145      │ 312K     │ $4.12    │
  │ CTO      │ 23       │ 89K      │ $1.45    │
  │ QA       │ 67       │ 156K     │ $2.01    │
  │ Security │ 34       │ 78K      │ $0.98    │
  │ FE-1     │ 112      │ 245K     │ $3.22    │
  ├──────────┼──────────┼──────────┼──────────┤
  │ TOTAL    │ 381      │ 880K     │ $11.78   │
  └──────────┴──────────┴──────────┴──────────┘

  Budget: $50/week │ Used: 23.6% │ On track ✓
```

**Alerting (terminal + Slack/Discord):**
```
$ agents alerts set --cost-threshold 80% --notify slack
$ agents alerts set --agent-error-rate 10% --notify terminal
$ agents alerts set --roundtable-deadlock 1h --notify all
```

### 2.6 Slack/Discord Notifications

**Not full bots — lightweight webhook notifications:**

```
$ agents integrations add slack --webhook-url https://hooks.slack.com/...
$ agents integrations add discord --webhook-url https://discord.com/api/webhooks/...
```

**What Gets Notified:**
- Auto-roundtable started (needs attention)
- Agent error (stuck or failed)
- PR ready for review
- Daily digest (summary of agent work)
- Cost threshold reached
- Roundtable deadlocked (needs human input)

**Daily Digest Example (sent to Slack):**
```
📊 Daily Agent Report — my-saas-app
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Tasks completed: 8
📝 PRs created: 3, merged: 2
💬 Roundtables: 2 (1 auto-triggered)
💰 Cost: $4.23
⚠️ Issues: 1 security flag on PR #45
```

---

## 3. Technical Improvements

### 3.1 Performance

**Targets:**
- Agent response: < 10s (p95) — down from 15s
- Memory query: < 200ms
- CLI command: < 500ms
- Sandbox execution: < 30s for tests

**Optimizations:**
- Redis caching for hot memory queries
- Connection pooling for all DB connections
- Parallel tool execution (agent can run GitHub + memory queries concurrently)
- Batch embedding upserts to Pinecone
- LLM response streaming in terminal

### 3.2 Scalability

- Support 50+ agents per project
- Queue-based task distribution (no agent overload)
- Memory namespace per project (Pinecone namespaces)
- Agent process pooling (reuse idle processes)

### 3.3 Security

- Audit logging (all agent actions to PostgreSQL)
- Secret scanning in agent-generated code
- Sandbox network isolation
- API key rotation reminders

---

## 4. Migration from Phase 1

### 4.1 Database Migration
- Add graph memory tables (or Neo4j connection)
- Add marketplace tables (templates, ratings)
- Add audit log table
- Add alert configuration table
- Backfill existing memories into graph

### 4.2 Config Changes
```yaml
# ~/.agents/config.yaml — new fields
auto_roundtable:
  enabled: true
  confidence_threshold: 70
  conflict_detection: true

sandbox:
  enabled: true
  docker_path: /usr/local/bin/docker
  timeout: 300  # seconds
  images: [node:20, python:3.11, golang:1.21]

notifications:
  slack_webhook: https://hooks.slack.com/...
  discord_webhook: null
  daily_digest: true
  digest_time: "09:00"

neo4j:
  uri: bolt://localhost:7687
  user: neo4j
  password: ...

cost:
  budget_weekly: 50
  alert_threshold: 80  # percent
```

### 4.3 Feature Flags
- Auto-roundtables: opt-in via config
- Sandbox: requires Docker installed
- Graph memory: optional (falls back to vector-only)
- Notifications: opt-in per channel

---

## 5. Timeline

### Week 1-3: Auto-Roundtables
- Confidence-based trigger detection
- Conflict detection (same file, disagreeing outputs)
- Auto-participant selection algorithm
- Background roundtable execution
- Terminal notification system

### Week 4-5: Marketplace
- Template registry (hosted on Supabase or simple API)
- CLI commands: search, install, publish
- Rating system
- Template validation

### Week 6-7: Code Sandbox
- Docker integration
- Pre-built language images
- Sandbox manager (spawn, execute, capture, cleanup)
- Agent integration (auto-sandbox before PR)
- Security hardening (no network, time limits)

### Week 8-9: Graph Memory
- Neo4j setup (local or cloud free tier)
- Schema design and migration
- Graph write on agent actions
- Graph query CLI commands
- Context injection from graph

### Week 10-11: Monitoring + Notifications
- CLI dashboard (live refresh)
- Cost analytics
- Alert system
- Slack/Discord webhook integration
- Daily digest generation

### Week 12: Polish
- Performance optimization
- Bug fixes
- Documentation updates
- npm version bump

---

## 6. Success Criteria

| Metric | Phase 1 | Phase 2 Target |
|--------|---------|----------------|
| Agents per project | 10 | 50+ |
| Auto-roundtables/week | 0 | 20+ |
| Marketplace templates | 5 built-in | 50+ community |
| Sandbox executions/week | 0 | 500+ |
| Agent response (p95) | 15s | 10s |
| Crash recovery rate | 95% | 99% |
| Cost tracking accuracy | N/A | ±5% |

---

## 7. Risks & Mitigation

| Risk | Mitigation |
|------|------------|
| Docker not installed on user machine | Graceful fallback: skip sandbox, warn user |
| Neo4j complexity | Optional — falls back to vector-only supermemory |
| Auto-roundtable noise (too many triggers) | Configurable thresholds, cooldown period |
| Marketplace spam/low-quality templates | Validation + rating system + verified badges |
| Slack/Discord webhook limits | Queue notifications, batch digest |

---

**Phase 2 Goal: Platform becomes intelligent — agents auto-detect uncertainty, discuss autonomously, test code in sandboxes, and notify you only when needed. Still $0 hosting, still CLI-first.**
