# Project Decisions Log

## Platform: Multi-Agent AI Development Platform
**Date:** 2026-03-16

---

## Decision 1: Agent Execution Model — Event-Driven, Not Always-On

**Context:** Agents don't all run simultaneously. They work like real company employees.

**Decision:** Agents are event-driven. They sleep until triggered.

**Details:**
- Small project: ~5-6 agents (CTO, 2 backend, 2 frontend, 1 tester, 1 security)
- Big project: multiple agents per role, scaling naturally
- Tester + Security agents wake up on PR creation
- CTO activates only for architecture decisions/roundtables
- Dev agents work when tasks are assigned
- Results flow between agents (e.g., tester results → dev agents)

**Impact:** Low LLM API cost (~$5-30/month for small projects), no heavy orchestration needed.

---

## Decision 2: Supermemory as Shared Brain

**Context:** Multiple agents (especially multiple agents per role in big projects) need coordination.

**Decision:** Supermemory (Pinecone + PostgreSQL) is the single source of truth for all agents.

**Details:**
- Every agent reads supermemory before starting work (like a standup meeting)
- Every agent writes back what it did after completing work
- Prevents duplication — agents know what others are working on
- Conflicts (two agents touching same file) auto-trigger roundtables
- Stores all decisions, context, file changes, architecture choices

**Analogy:**
| Real Company | Platform Equivalent |
|---|---|
| Standup meeting | Supermemory query before work |
| Jira board | Task queue |
| Slack channels | Roundtables |
| Code review | Tester + Security agents on PR |
| Tech lead decision | CTO agent in roundtable |
| Wiki/Docs | Supermemory stores decisions |

---

## Decision 3: CLI-First Architecture — No Web/Cloud

**Context:** Debated between web cloud, hybrid (web + CLI), and CLI-only.

**Decision:** Pure CLI terminal application. No web frontend.

**Reasons:**
- Zero hosting costs (no Vercel, no frontend infra)
- Faster to build (backend + CLI only, no React/Next.js)
- Developers live in terminals — natural UX
- Terminal already streams real-time output (no websockets needed)
- Build time cut roughly in half
- Can always add web later — the API layer will exist

**Architecture:**
```
Backend API + Queue (core logic)
       │
       ▼
   CLI Interface
       │
  ┌────┴────┬────────┬────────┬────────┐
  │ T1:CTO  │ T2:BE  │ T3:FE  │ T4:QA  │
  └─────────┴────────┴────────┴────────┘
  Each agent = its own terminal process
```

**Auth:** CLI opens browser once for login (like `gh auth login`), saves token locally.

---

## Decision 4: Each Agent Runs in Its Own Terminal

**Context:** How to isolate and visualize agent work.

**Decision:** Each agent gets its own terminal process.

**Benefits:**
- Visibility — watch each agent work in real-time (like watching an employee's screen)
- Isolation — agents can't interfere with each other
- Intuitive — "this terminal is my backend agent"
- Cost-effective — runs locally, no cloud VMs per agent

---

## Decision 5: Git Branches for Agent Isolation & Revert

**Context:** User concerned about reverting one agent's work without affecting others.

**Decision:** Each agent works on its own Git branch. Nothing merges without approval.

**Flow:**
```
main (stable)
  ├── backend-agent/auth-api
  ├── frontend-agent/login-page
  ├── frontend-agent/dashboard
  └── tester-agent/test-suite
```

**Key Points:**
- Agent finishes → creates PR (proposal, not merged)
- User reviews → approve / request changes / reject
- Reject = branch deleted, main untouched
- Revert merged code = one command, creates undo commit
- Cherry-pick = keep only the changes you like
- Full history of which agent wrote what and why

---

## Decision 6: Crash Recovery via Checkpointing

**Context:** What happens if terminal dies (power cut, internet loss, etc.)?

**Decision:** Agents checkpoint state to supermemory + database after each step. Resume from last checkpoint on restart.

**Flow:**
```
Agent starts task → saves state
Agent creates file → saves state
Agent runs test   → saves state
     [crash]
Agent restarts    → reads last state → continues from next step
```

---

## Decision 7: Prompt Engineering — Borrow from Proven Systems

**Context:** Reviewed github.com/x1xhlol/system-prompts-and-models-of-ai-tools (131k stars) for useful patterns.

**Decision:** Combine best patterns from existing AI tools:

| Component | Source | What We Take |
|---|---|---|
| Agent orchestration | Claude Code 2.0 | Parent spawns stateless child agents, collects results |
| CTO/Architect agent | Qoder Quest Design | Feature idea → technical design doc with architecture |
| Dev agent guardrails | Devin AI | Convention mimicry, library verification, no secrets |
| Task management | Cursor | Todo system with atomic tasks, status tracking |
| Requirements pipeline | Kiro | Requirements → design → task list with approval gates |
| Context injection | Manus | Planner/Knowledge/Datasource modules per agent |

**Key Patterns to Implement:**
- Tool parallelization (independent calls run concurrently)
- Single-action iteration (one tool per cycle, prevents runaway)
- Event stream as state (chronological log = source of truth)
- Approval gates (user sign-off between phases)
- Self-correction loops (detect skipped steps, auto-fix)

---

## Decision 8: Infrastructure — Free Tier Stack

**Decision:** Minimize costs using free tiers.

| Service | Purpose | Cost |
|---|---|---|
| GitHub | Repos, branches, PRs, collaboration | Free |
| Pinecone | Supermemory vector store | Free tier |
| Supabase/PostgreSQL | Agent state, conversation history | Free tier |
| Redis (Upstash) | Task queue | Free tier |
| LLM API | Agent intelligence | BYOK (user's own keys) |

**Total platform cost: $0 to start.**

---

## Decision 9: Phased Build Approach

**Build order (revised for CLI-first):**
1. Phase 1 (2-3 months): Core CLI — agents, supermemory, GitHub integration, basic roundtables
2. Phase 2 (3-4 months): Auto-roundtables, marketplace, code sandbox, graph memory
3. Phase 3 (4-6 months): Teams, RBAC, enterprise features, optional web UI

**Web frontend is deferred** — may be added in Phase 3 if needed, since the API layer exists.

---

## Open Questions
- Exact CLI framework choice (e.g., Commander.js, Ink, oclif)
- Agent config format (YAML vs JSON)
- How roundtable UI renders in terminal (streaming text vs TUI framework)
- Detailed prompt templates per agent role
