# PHASE 1: MVP
## Months 1-3 — Core CLI Platform

**Goal:** Minimum viable CLI tool with event-driven agent system, supermemory, GitHub integration, and crash recovery. Each agent runs in its own terminal process. Zero cloud hosting costs.

---

## 1. Scope

### 1.1 In Scope
- CLI application with all agent management
- Agent creation and management (up to 10 agents)
- Each agent runs as its own terminal process
- Event-driven execution (agents sleep until triggered)
- Basic supermemory (vector store + conversation history)
- Simple roundtables (human-initiated, rendered in terminal)
- GitHub integration (PR, commit, branch — each agent on its own branch)
- Crash recovery via state checkpointing
- BYOK with Claude Sonnet/Haiku
- Agent config via YAML files

### 1.2 Out of Scope
- Web dashboard / frontend (deferred, maybe Phase 3)
- Auto-triggered roundtables
- Graph database relationships
- Advanced monitoring
- Mobile app
- Multi-project support
- Custom tool builder

---

## 2. Technical Architecture

```
┌─────────────────────────────────────────────────┐
│  CLI Interface (Commander.js / Ink)             │
│  - agents, roundtable, memory, status commands  │
└─────────────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────────────┐
│  Backend Core (Node.js/TypeScript)              │
│  - Agent Orchestrator                           │
│  - Task Queue (Bull/Redis)                      │
│  - GitHub Integration (Octokit)                 │
│  - Checkpoint Manager (crash recovery)          │
└─────────────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────────────┐
│  Agent Processes (each in own terminal)         │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │
│  │ CTO  │ │ BE-1 │ │ BE-2 │ │ FE-1 │ │  QA  │ │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ │
│  Event-driven: sleep until triggered            │
└─────────────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────────────┐
│  Data Layer (all free tier)                     │
│  - PostgreSQL/Supabase (agents, tasks, state)   │
│  - Pinecone (vector memory)                     │
│  - Redis/Upstash (cache, queue, pub/sub)        │
└─────────────────────────────────────────────────┘
```

---

## 3. Features

### 3.1 CLI Commands

```
agents login                    Auth via browser (like gh auth login)
agents init                     Initialize project, link GitHub repo
agents create <name> --role <role> --template <template>
agents list                     Show all agents with status table
agents status                   Live dashboard in terminal
agents start <name>             Start agent in new terminal process
agents stop <name>              Stop agent gracefully
agents assign <name> <task>     Assign task to agent
agents logs <name> --follow     Stream agent logs in real-time
agents config <name>            Edit agent YAML config

roundtable start <topic>        Start roundtable discussion
roundtable list                 Show active/past roundtables
roundtable watch <id>           Watch roundtable live in terminal
roundtable input <id> <message> Add human input to roundtable

memory search <query>           Search supermemory
memory show <id>                Show memory entry details
memory stats                    Show memory usage stats

project status                  Overall project status
project cost                    Show LLM API cost tracking
```

### 3.2 Agent System

**Agent Configuration (YAML):**
```yaml
# agents/backend-1.yaml
name: Backend-1
role: Backend Developer
template: senior-engineer
model: claude-sonnet-4
tools:
  - github
  - shell
  - file_system
confidence_threshold: 70
triggers:
  - on: task_assigned
  - on: roundtable_invited
reports_to: cto
branch_prefix: backend-1
```

**Agent Templates (5 built-in):**
1. **CTO/Architect** — Architecture decisions, roundtable leadership, PR merge approval
   - Prompt patterns from: Qoder Quest Design
2. **Senior Engineer** — Write production code, review PRs, solve complex problems
   - Prompt patterns from: Devin AI (convention mimicry, library verification)
3. **QA Engineer** — Run tests, validate PRs, report bugs
   - Prompt patterns from: Cursor (lint validation loops)
4. **Security Engineer** — Audit code, enforce security practices, scan for secrets
   - Prompt patterns from: Devin AI (secret detection, credential handling)
5. **Code Reviewer** — Review PRs, check style consistency, verify test coverage
   - Prompt patterns from: Cursor (todo-driven execution)

**Agent Actions (Event-Driven):**
- Sleep until triggered (task assigned, PR created, roundtable invited)
- Query supermemory before acting (like a standup)
- Use tools (GitHub, shell, file system)
- Return structured JSON output with confidence score
- Write results back to supermemory (log what was done)
- Checkpoint state after each step (crash recovery)

### 3.3 Event-Driven Execution

Agents don't run 24/7. They activate on events:

```
┌──────────────────┬──────────────────────────────────┐
│ Event            │ Agents Triggered                 │
├──────────────────┼──────────────────────────────────┤
│ Task assigned    │ Target agent                     │
│ PR created       │ QA + Security + Code Reviewer    │
│ PR reviewed      │ Original dev agent               │
│ Roundtable start │ Selected participants             │
│ Conflict detect  │ CTO + involved agents            │
│ Test failure     │ Original dev agent               │
└──────────────────┴──────────────────────────────────┘
```

### 3.4 Supermemory (Basic)

**Components:**
- Vector store (Pinecone) for semantic search
- PostgreSQL for conversation history + agent state
- Simple embedding model (text-embedding-3-large)

**How Agents Use It (Like a Standup):**
```
Agent picks up task
       │
       ▼
Queries supermemory:
  - "What are other agents working on?"
  - "Any decisions about this area?"
  - "What code exists for this feature?"
       │
       ▼
Agent does work
       │
       ▼
Writes to supermemory:
  - "I built X using Y approach"
  - "Created file Z"
  - "Decision: using JWT not sessions"
```

**Memory Query (CLI):**
```
$ agents memory search "auth implementation"

  ┌────┬──────────┬────────────────────────────────┬───────┐
  │ #  │ Agent    │ Content                        │ Score │
  ├────┼──────────┼────────────────────────────────┼───────┤
  │ 1  │ CTO      │ Decision: Use JWT with refresh │ 0.95  │
  │ 2  │ BE-1     │ Created auth.ts with jose lib  │ 0.89  │
  │ 3  │ Security │ Flagged: add rate limiting     │ 0.82  │
  └────┴──────────┴────────────────────────────────┴───────┘
```

### 3.5 Roundtables (Simple — Terminal UI)

**Flow:**
1. User runs `roundtable start "API Rate Limiting"`
2. Select participants (agents) via interactive prompt
3. Roundtable streams in terminal like a chat
4. Each agent provides input sequentially
5. User can add input anytime
6. User makes final decision
7. Decision stored in supermemory

**Terminal Rendering:**
```
$ roundtable watch rt-001

  ┌─ Roundtable: API Rate Limiting ─────────────────────────┐
  │ Status: Active │ Participants: 3 │ Round: 2             │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │ [CTO] (confidence: 90%)                                 │
  │ Use token bucket algorithm with Redis. It handles       │
  │ distributed rate limiting well.                         │
  │                                                          │
  │ [BE-1] (confidence: 75%)                                │
  │ Agreed. I'll use rate-limiter-flexible package.         │
  │ Concern: should we rate limit per-user or per-IP?       │
  │                                                          │
  │ [Security] (confidence: 85%)                            │
  │ Per-user for authenticated, per-IP for public.          │
  │ Also add sliding window for brute force protection.     │
  │                                                          │
  │ [You] Type your input or /decide to make a decision     │
  └──────────────────────────────────────────────────────────┘
```

**Limitations (Phase 1):**
- Manual initiation only (no auto-detection)
- Sequential agent input (no parallel discussion)
- Max 5 participants
- No consensus algorithm

### 3.6 GitHub Integration

**Each Agent Gets Its Own Branch:**
```
main (stable — nothing merges without approval)
  ├── backend-1/auth-api
  ├── backend-2/payment-service
  ├── frontend-1/login-page
  └── qa/test-suite
```

**Supported Operations:**
- Clone repository
- Create branch (format: `agent-name/task-description`)
- Commit with structured messages
- Open pull requests
- Comment on PRs
- Merge (only with user approval)

**PR Flow:**
```
Agent finishes work → Creates PR
       │
       ▼
QA + Security agents auto-triggered → Review PR
       │
       ▼
Results sent back to dev agent
       │
       ▼
User reviews on GitHub or via CLI:
  $ agents pr list
  $ agents pr review 42
       │
  ┌────┴────────┐
  │ Approve     │ → merges to main
  │ Changes     │ → dev agent fixes
  │ Reject      │ → branch deleted, main untouched
  └─────────────┘
```

**Revert (User doesn't like something):**
```
$ agents revert frontend-1/login-page
  ✓ Branch deleted. Main untouched. No other agents affected.
```

### 3.7 Crash Recovery

**State Checkpointing After Each Step:**
```
Agent starts task    → checkpoint: {status: "started", task: "auth-api"}
Agent creates file   → checkpoint: {status: "in_progress", files: ["auth.ts"]}
Agent runs test      → checkpoint: {status: "testing", test_result: "pass"}
Agent creates PR     → checkpoint: {status: "pr_created", pr: 42}
          [CRASH — power cut, internet loss, etc.]
Agent restarts       → reads last checkpoint
                     → "I was on auth-api, created auth.ts, tests passed, PR 42 created"
                     → continues from next step
```

**Stored in:** PostgreSQL (persists across restarts)

### 3.8 Auth

**CLI Auth Flow (like `gh auth login`):**
```
$ agents login
  → Opens browser for OAuth login
  → User authenticates
  → Token saved locally (~/.agents/config.yaml)
  → Done. No web server needed.
```

---

## 4. Data Models

### Agent
```sql
CREATE TABLE agents (
  agent_id UUID PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  role TEXT NOT NULL,
  tools JSONB NOT NULL,
  model VARCHAR(50) NOT NULL,
  system_prompt TEXT NOT NULL,
  config JSONB NOT NULL,        -- full YAML config as JSON
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  status VARCHAR(20) DEFAULT 'idle'  -- idle, active, error, stopped
);
```

### Task
```sql
CREATE TABLE tasks (
  task_id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(agent_id),
  type VARCHAR(50) NOT NULL,
  payload JSONB NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  result JSONB,
  confidence_score INTEGER,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);
```

### Checkpoint (Crash Recovery)
```sql
CREATE TABLE checkpoints (
  checkpoint_id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(agent_id),
  task_id UUID REFERENCES tasks(task_id),
  state JSONB NOT NULL,         -- full agent state at this point
  step_number INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Roundtable
```sql
CREATE TABLE roundtables (
  roundtable_id UUID PRIMARY KEY,
  topic TEXT NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  initiator_id UUID REFERENCES users(id),
  participants JSONB NOT NULL,
  discussion JSONB DEFAULT '[]',
  outcome JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  concluded_at TIMESTAMP
);
```

### Memory
```sql
CREATE TABLE memories (
  memory_id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(agent_id),
  task_id UUID REFERENCES tasks(task_id),
  type VARCHAR(50) NOT NULL,    -- decision, code, conversation, fact
  content TEXT NOT NULL,
  summary TEXT NOT NULL,
  tags JSONB DEFAULT '[]',
  pinecone_id VARCHAR(100),     -- reference to vector store
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 5. Prompt Engineering (Borrowed Patterns)

### 5.1 Orchestration Pattern (from Claude Code 2.0)
- Parent orchestrator spawns stateless child agents
- Each agent gets detailed prompt with full context
- Results collected and summarized

### 5.2 CTO Agent (from Qoder Quest Design)
- Takes feature idea → produces technical design doc
- Analyzes repo structure before deciding
- Includes architecture, components, data flows, testing strategy

### 5.3 Developer Guardrails (from Devin AI)
- Mimic existing code conventions, don't invent new patterns
- Verify libraries exist before using them
- Never commit secrets or keys
- Run tests before submitting changes

### 5.4 Task Management (from Cursor)
- Atomic tasks with status tracking (pending → in_progress → completed)
- Reconciliation before new work
- Self-correction: detect skipped steps, auto-fix

### 5.5 Context Injection (from Manus)
- Planner/Knowledge/Datasource modules per agent
- Each agent gets role-specific context stack
- Event stream as single source of truth

### 5.6 Pipeline Flow (from Kiro)
- Requirements → Design → Task list with approval gates
- User sign-off between phases

---

## 6. User Flows

### 6.1 First-Time Setup
```
$ agents login              # Opens browser, saves token
$ agents init               # Links GitHub repo, creates project config
$ agents create cto --template cto    # Create CTO agent
$ agents create be-1 --template senior-engineer  # Create backend agent
$ agents create qa --template qa-engineer         # Create QA agent
$ agents start cto          # Start CTO in terminal 1
$ agents start be-1         # Start BE-1 in terminal 2
$ agents assign be-1 "Build user authentication with JWT"
```

### 6.2 Daily Workflow
```
$ agents status             # See what all agents are doing
$ agents logs be-1 --follow # Watch backend agent work
$ agents pr list            # Review pending PRs
$ agents pr review 42       # Review specific PR
$ roundtable start "Database schema for payments"
$ agents memory search "architecture decisions"
```

### 6.3 Agent Workflow (Internal)
```
1. Agent wakes up (event trigger)
2. Reads last checkpoint (if resuming from crash)
3. Queries supermemory: "What should I know?"
4. Executes task with tools
5. Checkpoints state after each step
6. Writes results to supermemory
7. Creates PR if code was written
8. Goes back to sleep
```

---

## 7. Infrastructure (All Free Tier)

| Service | Purpose | Cost |
|---------|---------|------|
| Supabase | PostgreSQL (agents, tasks, checkpoints, memory) | Free |
| Pinecone | Vector store (semantic search) | Free tier |
| Upstash | Redis (task queue, pub/sub, cache) | Free tier |
| GitHub | Repos, branches, PRs | Free |
| LLM API | Agent intelligence | BYOK (user pays) |

**No hosting costs.** CLI runs locally. Data stored on free-tier cloud services.

### Environment Variables
```
# ~/.agents/config.yaml (created on `agents login`)
database_url: postgres://...
redis_url: redis://...
pinecone_api_key: pk-...
pinecone_index: agents-memory
github_token: ghp_...
anthropic_api_key: sk-ant-...  # BYOK
```

---

## 8. Testing

### 8.1 Unit Tests
- Agent prompt generation
- Memory query/building
- GitHub API wrappers (Octokit)
- Checkpoint save/restore
- YAML config parsing

**Target: 80% coverage**

### 8.2 Integration Tests
- Agent task execution (assign → execute → complete)
- Supermemory read/write cycle
- GitHub operations (branch → commit → PR)
- Roundtable flow (start → discuss → decide)
- Crash recovery (checkpoint → kill → resume)

**Target: All critical paths covered**

### 8.3 E2E Tests
- Complete workflow: Create agent → Assign task → PR created → Review → Merge
- Roundtable: Start → Discussion → Decision stored in memory
- Multi-agent: Two agents working, checking supermemory before acting
- Crash recovery: Agent mid-task → kill process → restart → resumes

**Target: 5 core user journeys**

---

## 9. Timeline

### Week 1-2: Foundation
- Project structure (Node.js/TypeScript monorepo)
- CLI framework setup (Commander.js or oclif)
- Database schema + Supabase setup
- Auth flow (browser OAuth → token storage)
- YAML config parser for agents

### Week 3-4: Agent System
- Agent CRUD (create, list, start, stop)
- Agent process manager (spawn terminal processes)
- Task queue (Bull + Redis)
- Event-driven trigger system
- Basic agent execution loop (receive task → query memory → LLM call → return result)
- Prompt templates for 5 built-in roles

### Week 5-6: Supermemory
- Pinecone integration (embeddings, upsert, query)
- Memory indexing (code repo, conversations, decisions)
- Context retrieval before agent execution
- Memory write-back after agent completes
- `memory search` CLI command

### Week 7-8: GitHub Integration + Crash Recovery
- Octokit integration (clone, branch, commit, PR, review, merge)
- Per-agent branch strategy
- PR creation and review flow
- Checkpoint system (save/restore agent state)
- Resume from last checkpoint on restart

### Week 9-10: Roundtables + Terminal UI
- Roundtable CLI commands
- Terminal rendering (colored, formatted chat view)
- Sequential agent discussion flow
- Human input during roundtable
- Decision recording to supermemory

### Week 11-12: Polish & Launch
- End-to-end testing
- Bug fixes
- `agents status` live dashboard (terminal table)
- Cost tracking (`agents cost`)
- Documentation (README, getting started guide)
- npm publish

---

## 10. Success Criteria

| Metric | Target |
|--------|--------|
| Agents created per user | > 3 |
| Tasks completed/week | > 10 |
| Crash recovery success rate | > 95% |
| Agent response time (p95) | < 15s |
| Supermemory query time | < 500ms |
| CLI command response | < 1s |
| npm installs (first month) | > 100 |

---

## 11. Risks & Mitigation

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| LLM API costs high | Medium | Caching, use Haiku for simple tasks, BYOK |
| Pinecone free tier limits | Low | Monitor usage, implement cleanup |
| GitHub rate limits | Medium | Backoff + queue operations |
| Agent hallucinations | Medium | Strict prompting (Devin patterns), validation, confidence scoring |
| Terminal UI complexity | Medium | Start simple (tables + text), add TUI later |
| Cross-platform CLI issues | Medium | Test on Windows, macOS, Linux |

---

**Phase 1 Goal: Working CLI tool where developers can create AI agent teams that collaborate via supermemory, discuss decisions in terminal roundtables, work on isolated Git branches, and recover from crashes — all at $0 hosting cost.**
