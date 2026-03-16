# CUSTOMIZABLE MULTI-AGENT PLATFORM
## Product Requirements Document v3.0 — CLI-First Architecture
> User-Defined Agents. Shared Supermemory. Collective Intelligence. Zero Hallucinations. Zero Hosting Costs.

**Version:** 3.0 | **Status:** Updated Specification (CLI-First) | **Classification:** Internal
**Date:** 2026-03-16 | **Author:** Product Team

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Vision](#2-product-vision)
3. [User Personas](#3-user-personas)
4. [Core Concepts](#4-core-concepts)
5. [System Architecture](#5-system-architecture)
6. [Agent System](#6-agent-system)
7. [Supermemory System](#7-supermemory-system)
8. [Roundtable Protocol](#8-roundtable-protocol)
9. [Hallucination Prevention](#9-hallucination-prevention-framework)
10. [GitHub Integration](#10-github-integration)
11. [CLI Interface](#11-cli-interface)
12. [API Specifications](#12-api-specifications)
13. [Security & BYOK](#13-security--byok)
14. [Monitoring & Observability](#14-monitoring--observability)
15. [Performance Requirements](#15-performance-requirements)
16. [Error Handling & Crash Recovery](#16-error-handling--crash-recovery)
17. [Testing Strategy](#17-testing-strategy)
18. [Deployment & Infrastructure](#18-deployment--infrastructure)
19. [Success Metrics](#19-success-metrics)
20. [Future Roadmap](#20-future-roadmap)

---

## 1. Executive Summary

### 1.1 Problem Statement
Current AI agent platforms either:
- Force fixed agent structures (inflexible)
- Allow customization but agents work in isolation (no shared context)
- Fail silently when uncertain (hallucinations)
- Cannot escalate complex decisions collaboratively
- Require expensive cloud hosting
- Are web-only, alienating developer workflows

### 1.2 Solution
A **CLI-first platform** where users build custom AI teams that:
- **Run locally** — each agent in its own terminal process, zero hosting costs
- **Collaborate** via shared supermemory (agents read before working, write after)
- **Work like a real company** — event-driven, agents sleep until triggered
- **Escalate** uncertainty through structured roundtables (rendered in terminal)
- **Never hallucinate** via confidence scoring and strict output validation
- **Use Git branches** for isolation — each agent's work is reversible without affecting others
- **Recover from crashes** via state checkpointing
- **Scale** from 1 agent to 100+ without architectural changes

### 1.3 Key Differentiators

| Feature | Our Platform | Competitors |
|---------|-------------|-------------|
| Architecture | CLI-first, runs locally | Web/cloud hosted |
| Hosting Cost | $0 (free tier services) | $50-500+/month |
| Agent Count | User-defined (1-100+) | Fixed or limited |
| Agent Model | Event-driven (sleep until triggered) | Always-on or manual |
| Memory | Shared supermemory (vector + graph) | Isolated per agent |
| Collaboration | Terminal roundtables | None or basic chat |
| Uncertainty | Confidence scoring + auto-roundtable | Silent failure |
| Code Isolation | Per-agent Git branches | Single branch |
| Crash Recovery | Checkpoint-based resume | Start over |
| Hallucination | Confidence scoring + validation | None or basic |

### 1.4 Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Interface | CLI-first (web optional in Phase 3) | $0 hosting, developer-native, faster to build |
| Agent execution | Each agent in own terminal process | Isolation, visibility, local cost |
| Agent model | Event-driven (sleep until triggered) | Like real employees — work only when needed |
| Coordination | Supermemory as shared brain | Agents read before work, write after (like standup) |
| Code isolation | Per-agent Git branches | Revert one agent's work without affecting others |
| Crash handling | State checkpointing after each step | Resume from last checkpoint on restart |
| Cost model | BYOK (Bring Your Own Key) | User controls LLM spend |
| Infrastructure | All free-tier cloud services | $0 platform cost |
| Prompt patterns | Borrowed from proven systems | Qoder, Devin, Cursor, Kiro, Manus |

---

## 2. Product Vision

### 2.1 Vision Statement
"Every founder, developer, and team can build their perfect AI workforce — agents that remember, collaborate, and know when to ask for help — all from the terminal."

### 2.2 Target Users
- **Solo Founders**: Build entire dev teams with AI agents, $0 infra cost
- **Small Teams**: Scale execution without hiring, share agents across team
- **Enterprise**: Standardize AI workflows with governance, RBAC, audit trails
- **Developers**: Automate code review, testing, documentation from the terminal

### 2.3 Use Cases

#### Use Case 1: Solo Founder Building SaaS
```
$ agents create cto --template cto
$ agents create be-1 --template senior-engineer
$ agents create be-2 --template senior-engineer
$ agents create fe-1 --template senior-engineer
$ agents create qa --template qa-engineer
$ agents create sec --template security-engineer

# Assign work
$ agents assign be-1 "Build user authentication with JWT"
$ agents assign fe-1 "Build login page with React"

# BE-1 works → creates PR → QA + Security auto-triggered → review
# Conflict? → Auto-roundtable
# User reviews in terminal → approve/reject
```

#### Use Case 2: Big Project (Multiple Agents Per Role)
```
Backend Team: BE-1, BE-2, BE-3, BE-4
Frontend Team: FE-1, FE-2, FE-3
Shared: CTO, QA Lead, Security, DevOps

Each agent checks supermemory before working:
  BE-2 picks up "Build Payment API"
  → Queries: "What's BE-1 doing?" → Auth API, using JWT
  → Queries: "Schema decisions?" → PostgreSQL, REST not GraphQL
  → Now BE-2 knows the conventions and won't duplicate work
```

#### Use Case 3: Team Collaboration
```
$ agents team invite marcus@email.com --role admin
$ agents team invite sarah@email.com --role member

# Marcus and Sarah can both:
- Start/watch agents from their own terminals
- View roundtable discussions
- Approve/reject PRs
- Share the same agents and memory
```

---

## 3. User Personas

### 3.1 Persona: Sarah (Solo Founder)
- **Background**: Technical founder, building SaaS product alone
- **Pain Points**: Cannot afford to hire full team, context switching kills productivity
- **Goals**: Ship MVP fast, maintain code quality, stay within budget ($0 infra)
- **Usage Pattern**: Creates 5-6 agents, monitors via `agents status`, intervenes on major decisions via roundtables

### 3.2 Persona: Marcus (Engineering Lead)
- **Background**: Leads 10-person engineering team at mid-size company
- **Pain Points**: Code review bottlenecks, inconsistent documentation, security gaps
- **Goals**: Automate reviews, catch security issues early, improve code quality
- **Usage Pattern**: Team of 3 humans + 8 agents, uses approval workflows, reviews PRs in terminal

### 3.3 Persona: Enterprise DevOps Team
- **Background**: Large organization with compliance requirements
- **Pain Points**: Need audit trails, approval gates, budget controls, SSO
- **Goals**: Deploy AI agents with governance, track all decisions
- **Usage Pattern**: Full agent suite with RBAC, SSO, audit logging, custom tools

---

## 4. Core Concepts

### 4.1 Agent
An AI entity that **runs in its own terminal process** with:
- **Identity**: Name, role, description (defined in YAML config)
- **Capabilities**: Tools it can use (GitHub, shell, file system)
- **Lifecycle**: Event-driven — sleeps until triggered, works, goes back to sleep
- **Memory Access**: Read/write permissions to supermemory
- **Isolation**: Works on its own Git branch
- **Recovery**: Checkpoints state after each step

### 4.2 Supermemory
Centralized knowledge system — the **shared brain** of all agents:
- **Vector Store (Pinecone)**: Semantic search across all project knowledge
- **Graph Memory (Neo4j, Phase 2)**: Relationship tracking between entities
- **Relational (PostgreSQL)**: Structured data — agents, tasks, checkpoints
- **How it works**: Every agent reads before working (standup) and writes after (log)

### 4.3 Roundtable
Structured discussion protocol **rendered in the terminal**:
- **Trigger**: Low confidence, conflicting outputs, same file, or explicit request
- **Participants**: Uncertain agent + domain experts + optional human
- **Process**: Sequential input → Discussion → Consensus or escalation
- **Output**: Decision recorded in supermemory, action authorized

### 4.4 Confidence Scoring
Every agent output includes:
- **Numerical Score**: 0-100%
- **Uncertainty Flags**: Specific areas of doubt
- **Evidence**: Sources consulted, reasoning provided
- **Recommendation**: Proceed (≥70%), review, or roundtable (<70%)

### 4.5 Crash Recovery
Every agent checkpoints state after each step:
- **On crash**: Agent restarts, reads last checkpoint, continues from next step
- **Stored in**: PostgreSQL (persists across restarts)
- **No work lost**: Completed steps are preserved

---

## 5. System Architecture

### 5.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLI INTERFACE                                │
│  agents <command>  │  roundtable <command>  │  memory <command>  │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Agent        │  │ Roundtable   │  │ Task         │          │
│  │ Registry     │  │ Manager      │  │ Queue        │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│  ┌──────────────┐  ┌──────────────┐                             │
│  │ Checkpoint   │  │ Event        │                             │
│  │ Manager      │  │ Bus          │                             │
│  └──────────────┘  └──────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                 AGENT PROCESSES (Terminal)                       │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐       │
│  │  CTO   │ │  BE-1  │ │  BE-2  │ │  FE-1  │ │   QA   │       │
│  │ (idle) │ │(active)│ │(active)│ │ (idle) │ │ (idle) │       │
│  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘       │
│  Event-driven: sleep → trigger → work → checkpoint → sleep     │
│                    Message Bus (Redis)                           │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    SUPERMEMORY LAYER                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Vector Store │  │ Graph DB     │  │ Relational   │          │
│  │ (Pinecone)   │  │ (Neo4j)      │  │ (PostgreSQL) │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    INTEGRATION LAYER                             │
│  GitHub (Octokit) │ Shell │ File System │ LLM APIs (BYOK)      │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Component Details

#### 5.2.1 Agent Registry
- Stores agent definitions (YAML configs → PostgreSQL)
- Manages agent lifecycle (create, start, stop, delete)
- Tracks agent status (idle, active, error, stopped)

#### 5.2.2 Message Bus (Redis)
- Pub/sub for agent communication (event triggers)
- Queue for task distribution (Bull)
- Event streaming for monitoring

#### 5.2.3 Roundtable Manager
- Detects roundtable triggers (low confidence, conflicts, same file)
- Selects relevant expert agents
- Facilitates sequential discussion flow
- Records decisions to supermemory

#### 5.2.4 Task Queue
- Priority-based task distribution
- Work assignment based on agent capabilities
- Dead letter queue for failed tasks

#### 5.2.5 Checkpoint Manager
- Saves agent state after each step
- Restores state on crash/restart
- Cleanup of old checkpoints

#### 5.2.6 Event Bus
- GitHub webhooks → agent triggers
- Agent completion → downstream triggers
- Roundtable events → participant notification

---

## 6. Agent System

### 6.1 Agent Configuration (YAML)

```yaml
# agents/backend-1.yaml
name: Backend-1
role: Backend Developer
description: "Senior backend developer specializing in Node.js/TypeScript APIs"
template: senior-engineer

model: claude-sonnet-4
temperature: 0.3
max_tokens: 4096
confidence_threshold: 70

tools:
  - github
  - shell
  - file_system

triggers:
  - on: task_assigned
  - on: roundtable_invited
  - on: pr_review_requested

reports_to: cto
branch_prefix: backend-1

constraints:
  - "Follow existing code conventions (Devin pattern)"
  - "Verify libraries exist before using them"
  - "Never commit secrets or keys"
  - "Run tests before creating PR"
  - "Query supermemory before starting work"
```

### 6.2 Agent Templates (5 Built-in)

#### CTO/Architect Agent
- **Prompt source:** Qoder Quest Design
- **Triggers:** Architecture decisions, roundtable leadership, PR merge approval
- **Behavior:** Takes feature idea → analyzes repo → produces technical design doc

#### Senior Engineer Agent
- **Prompt source:** Devin AI
- **Triggers:** Task assigned, PR review requested
- **Behavior:** Convention mimicry, library verification, tests before PR

#### QA Engineer Agent
- **Prompt source:** Cursor (lint validation loops)
- **Triggers:** PR created
- **Behavior:** Run tests, validate code, report bugs, self-correction loops

#### Security Engineer Agent
- **Prompt source:** Devin AI
- **Triggers:** PR created
- **Behavior:** Audit code, scan for secrets, enforce security practices

#### Code Reviewer Agent
- **Prompt source:** Cursor (todo-driven execution)
- **Triggers:** PR ready for review
- **Behavior:** Review style, test coverage, architecture alignment

### 6.3 Agent Execution Flow

```
┌─────────────────┐
│ Event Trigger    │  (task assigned, PR created, roundtable invite)
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Resume from      │  (check for crash checkpoint)
│ Checkpoint?      │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Query            │
│ Supermemory      │  ← "What should I know?" (standup)
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Generate         │
│ Response         │  ← LLM call with full context
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Checkpoint       │  ← Save state (crash recovery)
│ State            │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Validate         │
│ Output           │  ← JSON schema + confidence check
└─────────────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐  ┌────────┐
│ >=70% │  │ <70%   │
│Proceed│  │Roundtable
└───────┘  └────────┘
    │
    ▼
┌─────────────────┐
│ Execute Actions  │  ← Git, shell, file ops
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Write to         │  ← Log what was done (standup update)
│ Supermemory      │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Sleep            │  ← Wait for next trigger
└─────────────────┘
```

### 6.4 Mandatory Output Schema

Every agent MUST respond with this JSON structure:

```json
{
  "metadata": {
    "agent_id": "uuid",
    "agent_name": "Backend-1",
    "timestamp": "2026-03-16T10:30:00Z"
  },
  "task": {
    "task_id": "uuid",
    "task_type": "implementation",
    "input_summary": "Build user authentication with JWT"
  },
  "execution": {
    "action_taken": "Created auth.ts with JWT login/signup endpoints",
    "tools_used": ["github", "shell", "file_system"],
    "files_modified": ["src/auth.ts", "src/middleware.ts"],
    "duration_ms": 5000,
    "tokens_used": { "input": 1500, "output": 800 }
  },
  "confidence": {
    "score": 85,
    "breakdown": {
      "understanding": 90,
      "approach": 80,
      "implementation": 85
    },
    "uncertainty_flags": [
      {
        "area": "rate_limiting",
        "description": "Uncertain about rate limiting approach",
        "severity": "medium"
      }
    ]
  },
  "memory_update": {
    "summary": "Built JWT auth with jose library. Login/signup endpoints. Refresh token flow.",
    "decisions": ["Using jose for JWT, not jsonwebtoken"],
    "tags": ["auth", "jwt", "backend"]
  },
  "next_actions": {
    "recommendation": "proceed",
    "pr_created": true,
    "pr_number": 42,
    "branch": "backend-1/auth-api"
  }
}
```

---

## 7. Supermemory System

### 7.1 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SUPERMEMORY                           │
├─────────────────────────────────────────────────────────┤
│  Ingestion Layer                                        │
│  ├── Code Repository Indexer (on agents init)           │
│  ├── Agent Action Logger (after each task)              │
│  ├── Roundtable Decision Recorder                       │
│  └── Checkpoint State Writer                            │
├─────────────────────────────────────────────────────────┤
│  Storage Layer                                          │
│  ├── Vector Store (Pinecone) — semantic search          │
│  ├── Graph Database (Neo4j, Phase 2) — relationships    │
│  └── Relational (PostgreSQL) — structured data          │
├─────────────────────────────────────────────────────────┤
│  Retrieval Layer                                        │
│  ├── Semantic Search (what's similar?)                  │
│  ├── Graph Traversal (what's related?) — Phase 2        │
│  ├── Temporal Query (what happened recently?)           │
│  └── Agent Query (what is agent X doing?)               │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Memory Types

| Type | Scope | Lifetime | Content |
|------|-------|----------|---------|
| Working Memory | Current task | Task duration | Current context, recent messages |
| Short-term Memory | Recent activity | 7 days, then summarized | Recent decisions, changes, discussions |
| Long-term Memory | Core knowledge | Permanent | Architecture decisions, conventions, lessons |
| Code Memory | Repository state | Synced with Git | File structure, patterns, dependencies |
| Checkpoint Memory | Agent state | Until task completes | Step-by-step progress for crash recovery |

### 7.3 Memory Operations

**Write (after agent completes work):**
```typescript
interface MemoryWrite {
  source: {
    agent_id: string;
    task_id: string;
    timestamp: Date;
  };
  content: {
    type: 'decision' | 'code' | 'conversation' | 'fact';
    data: any;
    summary: string;  // Human-readable summary
  };
  metadata: {
    project_id: string;
    tags: string[];
    confidence: number;
  };
}
```

**Query (before agent starts work — the "standup"):**
```typescript
interface MemoryQuery {
  query: string;              // "What should I know about auth?"
  filters?: {
    type?: string[];          // Only decisions? Only code?
    agents?: string[];        // What is BE-2 working on?
    time_range?: [Date, Date];
    tags?: string[];
  };
  limit: number;
  include_related?: boolean;  // Graph traversal (Phase 2)
}
```

### 7.4 Context Injection (Before Every Agent Execution)

```
Agent receives:
├── System Prompt (role, constraints, guardrails)
├── Task Description (current objective)
├── Supermemory Context:
│   ├── "What are other agents working on?" (avoid duplication)
│   ├── "Any decisions about this area?" (follow conventions)
│   ├── "What code exists?" (don't reinvent)
│   └── Agent's own past actions (continuity)
├── Checkpoint State (if resuming from crash)
└── Tool Definitions (available actions)
```

---

## 8. Roundtable Protocol

### 8.1 Trigger Conditions

| Condition | Threshold | Example |
|-----------|-----------|---------|
| Low Confidence | Score < 70% | "I'm only 60% sure about this API design" |
| Conflict Detection | Two agents disagree | BE-1 says REST, BE-2 says GraphQL |
| Same File | Two agents modifying same file | BE-1 and FE-1 both editing config.ts |
| Missing Information | Required data unavailable | "Cannot find database schema" |
| Human Request | User runs `roundtable start` | "Discuss this as a team" |

### 8.2 Discussion Flow

```
Phase 1: Context Sharing (Round 1)
├── Initiator presents problem + reasoning
├── Each expert reviews context from supermemory
└── Experts ask clarifying questions

Phase 2: Expert Input (Round 2)
├── Each expert provides analysis
├── References relevant past decisions
├── States confidence in their assessment
└── Flags areas of uncertainty

Phase 3: Discussion (Round 3+)
├── Experts debate conflicting viewpoints
├── Reference code, docs, best practices
├── Narrow down to 2-3 options
└── Highlight trade-offs

Phase 4: Resolution
├── If consensus → Decision recorded to supermemory
├── If deadlock → Escalate to human
└── Action items assigned to agents
```

### 8.3 Terminal Rendering

```
$ roundtable watch rt-001

  ┌─ Roundtable: API Rate Limiting ─────────────────────────┐
  │ Status: Active │ Participants: 3 │ Round: 2             │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │ [CTO] (confidence: 90%)                                 │
  │ Use token bucket algorithm with Redis. It handles       │
  │ distributed rate limiting well and we already have       │
  │ Redis in our stack.                                     │
  │ Source: Decision RT-003 (chose Redis for caching)       │
  │                                                          │
  │ [BE-1] (confidence: 75%)                                │
  │ Agreed on token bucket. I'll use rate-limiter-flexible  │
  │ package. Question: per-user or per-IP?                  │
  │                                                          │
  │ [Security] (confidence: 85%)                            │
  │ Per-user for authenticated endpoints, per-IP for        │
  │ public. Add sliding window for brute force.             │
  │ Source: OWASP rate limiting guidelines                  │
  │                                                          │
  │ [You] Type input or /decide to make final decision      │
  └──────────────────────────────────────────────────────────┘
```

### 8.4 Human Escalation

If roundtable deadlocks:
```
Terminal:
  ⚠ Roundtable RT-042 deadlocked after 3 rounds.
  Topic: "Database choice for payment service"
  Options:
    1. PostgreSQL (CTO, BE-1 vote) — proven, we know it
    2. MongoDB (BE-2 vote) — flexible schema for payments
  Trade-offs summarized below.
  Run: roundtable input rt-042 "Go with PostgreSQL"

Slack (if configured):
  🔔 Roundtable deadlocked: "Database choice for payment service"
  Your input needed. Run: agents roundtable input rt-042 "<decision>"
```

---

## 9. Hallucination Prevention Framework

### 9.1 Multi-Layer Defense

```
┌─────────────────────────────────────────────┐
│ Layer 1: Prompt Engineering                  │
│ ├── Role definition with constraints         │
│ ├── Patterns from Devin (convention mimicry) │
│ └── Explicit "I don't know" instructions     │
├─────────────────────────────────────────────┤
│ Layer 2: Structured Output                   │
│ ├── JSON schema enforcement                  │
│ ├── Confidence scoring mandatory             │
│ └── Uncertainty flagging                     │
├─────────────────────────────────────────────┤
│ Layer 3: Context Grounding                   │
│ ├── Supermemory retrieval (standup)           │
│ ├── Source attribution required               │
│ └── Code execution validation (sandbox, P2)  │
├─────────────────────────────────────────────┤
│ Layer 4: Validation                          │
│ ├── Schema validation                        │
│ ├── Self-correction loops (Cursor pattern)   │
│ └── Consistency checks                       │
├─────────────────────────────────────────────┤
│ Layer 5: Escalation                          │
│ ├── Confidence < 70% → roundtable            │
│ ├── Conflict → auto-roundtable               │
│ └── Deadlock → human escalation              │
└─────────────────────────────────────────────┘
```

### 9.2 Strict Prompting Rules (Included in Every Agent)

```
CRITICAL INSTRUCTIONS:

1. NEVER guess or make up information
2. If uncertain, set confidence_score < 70 and flag uncertainty
3. Always cite sources from supermemory or tool results
4. Use "I don't know" rather than hallucinating
5. Request clarification rather than assuming
6. Validate all code before claiming it works
7. Check facts against supermemory before stating
8. Follow existing code conventions (don't invent new patterns)
9. Verify libraries exist before using them
10. Never commit secrets or keys

OUTPUT RULES:
- Must use provided JSON schema
- Must include confidence_score (0-100)
- Must list uncertainty_flags if any doubt
- Must cite sources for factual claims
- Must include memory_update for what was learned/done
```

### 9.3 Code-Specific Safeguards

1. **Convention Mimicry** (Devin): Follow existing patterns, don't invent
2. **Library Verification** (Devin): Check package exists before importing
3. **Self-Correction** (Cursor): Detect skipped steps, auto-fix on next turn
4. **Lint Check**: Run linter after every edit (Cursor pattern — max 3 correction loops)
5. **Sandbox Execution** (Phase 2): Run in Docker before committing
6. **Diff Review**: Show changes before applying

---

## 10. GitHub Integration

### 10.1 Per-Agent Branch Strategy

```
main (stable — NOTHING merges without user approval)
  ├── backend-1/auth-api         ← BE-1 working on auth
  ├── backend-2/payment-service  ← BE-2 working on payments
  ├── frontend-1/login-page      ← FE-1 working on login UI
  └── qa/test-suite              ← QA writing tests
```

**Key principle:** Each agent works on its own branch. User can:
- **Approve** → merge to main
- **Request changes** → agent fixes
- **Reject** → branch deleted, main untouched, no other agents affected
- **Cherry-pick** → take only specific changes
- **Revert** → one command undoes merged work

### 10.2 Operations

| Operation | Agent Capability | Validation |
|-----------|-----------------|------------|
| Clone | All | N/A |
| Branch | All | Naming: `agent-name/task-desc` |
| Commit | All | Message format enforced |
| PR Create | All devs | Template required |
| PR Review | QA, Security, Reviewer | Structured review JSON |
| Merge | Only with user approval | Approval gate enforced |

### 10.3 Commit Message Format

```
[AGENT: agent-name] type: description

Examples:
[AGENT: BE-1] feat: add JWT authentication with refresh tokens
[AGENT: SEC-1] fix: remove hardcoded API key from config
[AGENT: QA-1] test: add auth integration tests
```

### 10.4 PR Review Flow

```
Agent creates PR
       │
       ▼
QA + Security auto-triggered (event-driven)
       │
       ▼
Reviews posted as structured JSON:
  {
    "review_id": "uuid",
    "pr_number": 42,
    "status": "changes_requested",
    "confidence": 90,
    "checks": {
      "syntax": { "passed": true },
      "security": { "passed": false, "issues": ["Hardcoded key line 23"] },
      "tests": { "passed": true, "coverage": "85%" }
    }
  }
       │
       ▼
Results sent to original dev agent → fixes issues
       │
       ▼
User reviews via CLI:
  $ agents pr list
  $ agents pr review 42
  $ agents pr approve 42  OR  agents pr reject 42
```

---

## 11. CLI Interface

### 11.1 Command Overview

```
AGENT MANAGEMENT
  agents login                         Auth via browser
  agents init                          Init project, link GitHub repo
  agents create <name> [options]       Create agent from template or custom
  agents list                          List all agents with status
  agents start <name>                  Start agent (new terminal process)
  agents stop <name>                   Stop agent gracefully
  agents status                        Live dashboard in terminal
  agents config <name>                 Edit agent YAML config
  agents assign <name> <task>          Assign task to agent
  agents logs <name> --follow          Stream agent logs real-time

ROUNDTABLES
  roundtable start <topic>             Start discussion
  roundtable list                      List active/past roundtables
  roundtable watch <id>                Watch live in terminal
  roundtable input <id> <message>      Add human input

MEMORY
  memory search <query>                Search supermemory
  memory show <id>                     Show memory entry
  memory graph <entity>                Graph relationships (Phase 2)
  memory stats                         Usage statistics

GITHUB
  agents pr list                       List open PRs
  agents pr review <number>            Review PR details
  agents pr approve <number>           Approve and merge
  agents pr reject <number>            Reject and delete branch
  agents revert <branch>               Revert agent's work

PROJECT (Phase 3)
  agents project list                  List all projects
  agents project switch <name>         Switch active project
  agents project create <name>         Create new project

TEAM (Phase 3)
  agents team invite <email> --role    Invite team member
  agents team list                     List team members
  agents team remove <email>           Remove member

MONITORING
  agents dashboard                     Live status dashboard
  agents cost [--period week/month]    Cost analytics
  agents alerts set [options]          Configure alerts
  agents audit log [options]           View audit trail (Phase 3)

TOOLS (Phase 3)
  agents tools create <file>           Register custom tool
  agents tools list                    List active tools
  agents tools run <name>              Execute tool manually

MARKETPLACE (Phase 2)
  agents marketplace search <query>    Search templates
  agents marketplace install <name>    Install template
  agents marketplace publish <file>    Submit template
```

### 11.2 Terminal Dashboard

```
$ agents status

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
  ║  │ FE-1   │ Frontend Dev │ 💤 idle  │ Waiting API   │    ║
  ║  │ QA     │ Tester       │ 💤 idle  │ Waiting PR    │    ║
  ║  │ SEC    │ Security     │ 💤 idle  │ Waiting PR    │    ║
  ║  └────────┴──────────────┴──────────┴───────────────┘    ║
  ║                                                          ║
  ║  ROUNDTABLES                                             ║
  ║  🔵 RT-042: Auth Token Strategy (3 participants)        ║
  ║                                                          ║
  ║  TODAY: 12 tasks done │ 4 PRs open │ 87 LLM calls      ║
  ╚══════════════════════════════════════════════════════════╝
```

---

## 12. API Specifications

The backend exposes a local API (used by CLI and optional web UI):

### 12.1 Agent Management
```
POST   /api/v1/agents              Create agent
GET    /api/v1/agents              List agents
GET    /api/v1/agents/:id          Get agent details + metrics
PATCH  /api/v1/agents/:id          Update agent config
DELETE /api/v1/agents/:id          Delete agent
POST   /api/v1/agents/:id/tasks    Assign task
POST   /api/v1/agents/:id/start    Start agent process
POST   /api/v1/agents/:id/stop     Stop agent process
```

### 12.2 Roundtables
```
POST   /api/v1/roundtables              Create roundtable
GET    /api/v1/roundtables              List roundtables
GET    /api/v1/roundtables/:id          Get roundtable (with discussion)
POST   /api/v1/roundtables/:id/input    Add human input
```

### 12.3 Memory
```
POST   /api/v1/memory/query        Query supermemory
POST   /api/v1/memory              Write to memory
GET    /api/v1/memory/:id          Get memory entry
DELETE /api/v1/memory/:id          Delete memory entry
POST   /api/v1/memory/graph/query  Graph query (Phase 2)
```

### 12.4 GitHub
```
POST   /api/v1/github/clone        Clone repo
POST   /api/v1/github/branch       Create branch
POST   /api/v1/github/commit       Create commit
POST   /api/v1/github/pr           Create PR
POST   /api/v1/github/pr/:id/review Add review
POST   /api/v1/github/pr/:id/merge Merge PR
DELETE /api/v1/github/branch/:name Delete branch (revert)
```

### 12.5 Checkpoints
```
POST   /api/v1/checkpoints         Save checkpoint
GET    /api/v1/checkpoints/:agent  Get latest checkpoint for agent
DELETE /api/v1/checkpoints/:agent  Clear checkpoints (after task complete)
```

---

## 13. Security & BYOK

### 13.1 BYOK Architecture

```
$ agents login → browser OAuth → token saved locally
$ agents config set anthropic_api_key sk-ant-...  → encrypted in local config

Agent needs LLM call:
  1. Read encrypted key from local config
  2. Decrypt in memory
  3. Call LLM API directly (no proxy, no middleman)
  4. Key never leaves local machine
  5. Key never logged or stored in supermemory
```

### 13.2 Key Security
- **Encryption**: AES-256-GCM at rest (local config file)
- **Transmission**: Direct HTTPS to LLM API (TLS 1.3)
- **Access**: Key never exposed to agents, logs, or supermemory
- **Rotation**: User-initiated via `agents config set`

### 13.3 Model Routing (Per-Agent)

| Task Type | Recommended Model | Cost Level |
|-----------|------------------|------------|
| Simple tasks, quick checks | Claude Haiku | Low (~$0.001/call) |
| Code writing, review | Claude Sonnet | Medium (~$0.01-0.05/call) |
| Architecture decisions | Claude Sonnet | Medium |
| Complex reasoning | Claude Sonnet/Opus | High |

### 13.4 Budget Controls

```yaml
# project.yaml
budget:
  daily_cap: 10        # $10/day max
  weekly_cap: 50       # $50/week max
  per_agent_daily: 5   # $5/day per agent
  alert_thresholds: [50, 80, 95]  # percent
  auto_pause: true     # pause agents at limit
```

---

## 14. Monitoring & Observability

### 14.1 Metrics (CLI-Based)

**Agent Metrics:**
- Tasks completed/failed
- Average confidence score
- Roundtable participation rate
- Token usage per task
- Response time
- Error rate

**System Metrics:**
- Queue depth (pending tasks)
- Memory query latency
- GitHub API rate limit usage
- LLM API rate limit usage

**Cost Metrics:**
- Cost per agent, per day/week/month
- Cost per task type
- Budget utilization percentage

### 14.2 Alerting

| Condition | Severity | Action |
|-----------|----------|--------|
| Agent stuck > 30 min | Warning | Terminal notification |
| Agent error rate > 10% | Critical | Terminal + Slack alert |
| LLM quota exceeded | Critical | Pause agent, notify |
| Roundtable deadlock > 1 hr | Warning | Escalate to human |
| Cost > 80% budget | Warning | Terminal + Slack alert |
| Cost > 100% budget | Critical | Auto-pause all agents |

### 14.3 Structured Logging

```json
{
  "timestamp": "2026-03-16T10:30:00Z",
  "level": "info",
  "component": "agent_orchestrator",
  "event": "task_completed",
  "agent_id": "uuid",
  "task_id": "uuid",
  "duration_ms": 5000,
  "tokens_used": 2300,
  "confidence": 85,
  "cost": 0.034,
  "trace_id": "uuid"
}
```

---

## 15. Performance Requirements

### 15.1 Response Times

| Operation | Target | Maximum |
|-----------|--------|---------|
| CLI command response | < 500ms | 1s |
| Agent task initiation | < 1s | 3s |
| Simple agent response | < 5s | 15s |
| Complex agent response | < 30s | 60s |
| Supermemory query | < 500ms | 2s |
| Roundtable initiation | < 3s | 10s |
| Checkpoint save | < 200ms | 1s |

### 15.2 Throughput

- **Concurrent Agents**: 100+ per project
- **Messages/Second**: 100+ through Redis
- **Memory Queries**: 1000+ per minute
- **GitHub Operations**: Rate limit respecting

---

## 16. Error Handling & Crash Recovery

### 16.1 Error Categories

| Category | Examples | Handling |
|----------|----------|----------|
| Agent Error | LLM timeout, invalid output | Retry with backoff (1s, 5s, 15s), then alert |
| Tool Error | GitHub API failure, shell error | Surface to agent, retry, log |
| System Error | DB down, Redis unreachable | Circuit breaker, queue for retry |
| User Error | Invalid config, bad API key | Immediate CLI feedback with guidance |

### 16.2 Crash Recovery Protocol

```
Agent Process Dies (power cut, internet loss, kill signal)
         │
         ▼
On Restart: agents start <name>
         │
         ▼
┌─────────────────┐
│ Check for        │
│ Checkpoint       │
└─────────────────┘
    │           │
    ▼           ▼
 Found        None
    │           │
    ▼           ▼
┌──────────┐  ┌──────────┐
│ Resume    │  │ Start    │
│ from last │  │ fresh    │
│ checkpoint│  │          │
└──────────┘  └──────────┘
    │
    ▼
Continue from next step
(completed steps preserved)
```

### 16.3 Circuit Breaker

For external dependencies (GitHub API, LLM APIs):
- **Closed**: Normal operation
- **Open**: After 3 consecutive failures, block requests for 60s
- **Half-Open**: Test with single request before closing

---

## 17. Testing Strategy

### 17.1 Unit Tests (80% coverage)
- Agent prompt generation
- Output validation logic
- Memory query/building
- Checkpoint save/restore
- YAML config parsing
- CLI command parsing

### 17.2 Integration Tests
- Agent task execution end-to-end
- Roundtable flow (start → discuss → decide)
- GitHub operations (branch → commit → PR)
- Supermemory read/write cycle
- Crash recovery (checkpoint → kill → resume)

### 17.3 E2E Tests (5 core journeys)
1. Create agent → Assign task → PR created → Review → Merge
2. Roundtable: Start → Discussion → Decision stored in memory
3. Multi-agent: Two agents check supermemory, avoid duplication
4. Crash recovery: Agent mid-task → kill → restart → resumes
5. Revert: Reject PR → branch deleted → main untouched

### 17.4 Hallucination Tests
- Adversarial prompts (try to make agent hallucinate)
- Low context scenarios (insufficient information)
- Source citation verification
- Confidence score accuracy (correlation with correctness)

---

## 18. Deployment & Infrastructure

### 18.1 Architecture (No Cloud Hosting)

```
User's Machine (Local)
├── CLI Tool: agents (npm install -g)
├── Agent Processes (terminal instances)
├── Docker (sandbox, Phase 2)
└── Optional Web UI (localhost:3000, Phase 3)

Cloud Services (All Free Tier)
├── Supabase — PostgreSQL (agents, tasks, checkpoints, teams)
├── Pinecone — Vector store (supermemory, namespaces per project)
├── Upstash — Redis (task queue, pub/sub, cache)
├── Neo4j Aura Free — Graph memory (Phase 2)
└── GitHub — Repos, branches, PRs

Cost: $0 platform hosting. User only pays for LLM API (BYOK).
```

### 18.2 Installation

```bash
npm install -g @agents/cli

agents login       # One-time auth
agents init        # Link to GitHub repo
agents create cto --template cto
agents start cto   # CTO agent running in terminal
```

---

## 19. Success Metrics

### 19.1 Technical Metrics

| Metric | Target |
|--------|--------|
| Hallucination rate | < 1% |
| Confidence accuracy | > 90% correlation with correctness |
| Crash recovery success | > 99% |
| Roundtable resolution (without human) | > 85% |
| Agent response (p95) | < 15s (Phase 1), < 10s (Phase 2) |

### 19.2 User Metrics

| Metric | Target |
|--------|--------|
| npm installs (first month) | > 100 |
| Tasks completed/week per user | 50+ |
| User retention (30d) | > 70% |
| GitHub stars (6 months) | > 1000 |

### 19.3 Business Metrics

| Metric | Target |
|--------|--------|
| Cost efficiency | 10x cheaper than human team |
| Time to value | < 30 min from install to first task |
| API cost per task | < $0.10 average |
| Paid conversions (Phase 3) | > 15% |

---

## 20. Future Roadmap

### Phase 1: MVP (Months 1-3)
- CLI application
- Agent system (5 templates, event-driven, per-agent terminals)
- Basic supermemory (Pinecone + PostgreSQL)
- Simple roundtables (manual, terminal UI)
- GitHub integration (per-agent branches)
- Crash recovery (checkpointing)
- BYOK

### Phase 2: Enhanced (Months 4-6)
- Auto-roundtables (confidence-based triggers)
- Agent marketplace (community templates)
- Code sandbox (Docker, local)
- Graph memory (Neo4j)
- Advanced CLI monitoring (dashboard, cost analytics)
- Slack/Discord notifications

### Phase 3: Scale (Months 7-12)
- Multi-project support
- Team collaboration (RBAC, approvals)
- Enterprise SSO (SAML, OIDC)
- Custom tool builder (YAML workflows)
- Advanced analytics
- Optional web UI (thin frontend)
- Pricing tiers

### Phase 4: Intelligence (Months 13+)
- Self-improving agents (learn from roundtable outcomes)
- Predictive task assignment
- Cross-project knowledge sharing
- AI agent marketplace (paid agents)
- IDE plugins (VS Code, JetBrains)
- Self-hosted enterprise option
- Local LLM support (Ollama)

---

## Appendix A: Glossary

| Term | Definition |
|------|-----------|
| **Agent** | AI entity with specific role, runs in its own terminal process |
| **Supermemory** | Shared knowledge system (vector + graph + relational) |
| **Roundtable** | Collaborative discussion protocol for uncertain decisions |
| **BYOK** | Bring Your Own Key — user provides LLM API keys |
| **Confidence Score** | Agent's self-assessment of certainty (0-100) |
| **Checkpoint** | Saved agent state for crash recovery |
| **Event-Driven** | Agents sleep until triggered by events |
| **Standup** | Agent querying supermemory before starting work |

## Appendix B: Prompt Engineering Sources

| Component | Source | Pattern |
|-----------|--------|---------|
| Orchestration | Claude Code 2.0 | Parent spawns stateless child agents |
| CTO Agent | Qoder Quest Design | Feature → technical design doc |
| Dev Guardrails | Devin AI | Convention mimicry, library verification, no secrets |
| Task Management | Cursor | Todo-driven, atomic tasks, self-correction |
| Pipeline | Kiro | Requirements → design → tasks with approval gates |
| Context | Manus | Planner/Knowledge/Datasource modules per agent |

## Appendix C: System Prompt Template

```
You are {agent_name}, a {agent_role} on the {project_name} project.

YOUR RESPONSIBILITIES:
{responsibilities}

TOOLS AVAILABLE:
{tools}

SUPERMEMORY PROTOCOL:
1. BEFORE starting any work, query supermemory:
   - "What are other agents working on?"
   - "Any decisions about {task_area}?"
   - "What code/patterns exist for this?"
2. AFTER completing work, write to supermemory:
   - Summary of what you did
   - Decisions you made and why
   - Files you modified

CRITICAL RULES:
1. NEVER guess or make up information
2. If uncertain (confidence < 70%), trigger a roundtable
3. Always cite sources from supermemory or tool results
4. Follow existing code conventions — don't invent new patterns
5. Verify libraries exist before using them
6. Never commit secrets or keys
7. Run tests before creating PR
8. Use structured JSON output format

BRANCH: Work on branch "{branch_prefix}/{task_description}"
REPORTS TO: {reports_to}
MODEL: {model}
```
