# Quality Metric Builder - Product Requirements Document
**Project**: Gauntlet Capstone - AI-Powered Healthcare Quality Metric Automation  
**Owner**: Rashmi (DS Engineering Manager, Clarify Health)  
**Created**: January 2026  
**Status**: Draft v1.0

---

## Table of Contents
1. Problem Statement
2. Elevator Pitch
3. Executive Summary
4. Technical Summary
5. Technical Stack
6. List of Epics
7. List of User Stories  
8. List of Features
9. List of PRs
10. Best Practices
11. Architecture Diagrams
12. Methodology
13. List of Tickets
14. Ticket Breakdown
15. IDE Rules

---

## Problem Statement

Clarify Health's current quality metric development is manual, labor-intensive, and bottlenecked. Each metric takes 3-8 weeks, limiting the team to 2-3 metrics/month.

**Pain Points**:
- Manual SQL development with complex temporal logic
- Time-consuming patient deep-dives (5-10 patients per metric)
- Limited throughput (40+ total metrics, need 200+)
- Knowledge concentrated in 2-3 team members
- Slow feedback loops (days to incorporate changes)

**Impact**: Customer requests delayed months, competitive disadvantage, team burnout.

## Elevator Pitch

Quality Metric Builder automates 80% of Clarify's metric development workflow using LangGraph-orchestrated AI agents, reducing development time from weeks to days while preserving clinical oversight at critical checkpoints.

## Executive Summary

Transform metric development from manual bottleneck to AI-accelerated pipeline.

**Value**: 10x speed (weeks→days), 5x capacity (2-3/month→10-15/month), maintained quality via human review gates.

**Agents**: 
1. **Research Agent** - Searches Tier 1 (USPSTF, HEDIS, CMS, AHRQ, CDC), Tier 2 (specialty societies), Tier 3 (peer-reviewed literature)
2. **Specification Agent** - Extracts denominator/numerator logic, maps to clinical codes, runs specs against claims data
3. **Build Agent** - Tests against Medicare/Medicaid and commercial claims, generates individual denom/numerator components
4. **Feasibility QA Agent** - Validates claims source coverage, volume, opportunity, benchmarks, practice-reflectiveness; **finalizes SQL logic and metric specifications**
5. **[HUMAN REVIEW]** - Reviews finalized SQL logic and specifications from Feasibility QA Agent
6. **Query Builder** - Translates approved specs to production Athena SQL
7. **Execution Agent** - Runs queries via Athena/EMR
8. **Analysis Agent** - Computes volumes, distributions, benchmarks
9. **QA Agent** - Generates comprehensive report

**Human Checkpoints**: 
- **Checkpoint 1 (Post-Feasibility)**: Human reviews finalized SQL logic and metric specifications
- **Checkpoint 2 (Post-QA)**: Final sign-off on QA report

**Success**: Deploy 50+ metrics in 6 months, >90% spec approval rate, 100% clinical accuracy.

## Technical Summary

**Architecture**: Multi-agent LangGraph system with PostgreSQL state persistence.

**Key Tech**: Python 3.11+, FastAPI, LangGraph, React/TypeScript, AWS (Athena/S3/EMR), Claude Sonnet 4.

**Data Flow**: 
1. External Research Sources (USPSTF, HEDIS, CMS, specialty societies, PubMed) → Research Agent
2. Research Findings → Spec Agent → Extract Denom/Numer Logic → Map to Clinical Codes
3. Specifications → Build Agent → Test against Medicare/Medicaid & Commercial Claims → Generate Components
4. Components → Feasibility QA Agent → Claims Source/Volume/Opportunity/Comparison/Practice-Reflective Checks
5. Feasibility QA Agent → **Finalizes SQL Logic & Metric Specifications**
6. Finalized SQL + Specs → **[Human Review Checkpoint 1]** - Reviewer approves/rejects SQL logic and specifications
7. Approved SQL + Specs → Query Builder → Production Athena SQL → Execution Agent (Athena/Airflow)
8. Execution Results → Analysis Agent → QA Agent → Comprehensive Report
9. QA Report → **[Human Sign-off Checkpoint 2]** → COMPLETE

**Integrations**: Argos (observations), Airflow (EMR clusters), Athena (claims queries), Medicare/Medicaid data, Commercial claims data.

## Technical Stack

**Backend**: Python 3.11+, FastAPI, LangGraph, SQLAlchemy, Celery, anthropic SDK, boto3
**Frontend**: React 18, TypeScript, Vite, TanStack Query, Zustand, shadcn/ui, Tailwind
**Data**: PostgreSQL 15, Redis 7, AWS Athena, S3, EMR
**Monitoring**: Prometheus, Grafana, OpenTelemetry, Sentry
**DevOps**: Docker, GitHub Actions, AWS

## List of Epics

1. **Epic 1**: Agent Orchestration Framework (LangGraph, checkpoints, state persistence)
2. **Epic 2**: Research & Specification Agents (Tier 1/2/3 sources, spec generation, code mapping)
3. **Epic 2b**: Build & Feasibility Agents (claims testing, component building, feasibility QA checks)
4. **Epic 3**: Query Builder & Execution (SQL generation, Athena, Airflow)
5. **Epic 4**: Analysis & QA Agents (volumes, distributions, benchmarks, deep-dives, reports)
6. **Epic 5**: Web Dashboard (request form, spec review, QA review, status tracking)
7. **Epic 6**: Data Access & Integration (Argos, Athena, Airflow, S3, Medicare/Medicaid, Commercial claims)
8. **Epic 7**: Clinical Code Management (ICD-10/CPT/HCPCS/GPI database, validation API)
9. **Epic 8**: Testing & Observability (tests, logging, monitoring, alerts)

## List of User Stories

**Research Phase**:
- US-1: Submit metric request in natural language
- US-2: Research Agent searches Tier 1 national organizations (USPSTF, HEDIS, CMS, AHRQ, CDC)
- US-3: Research Agent searches Tier 2 specialty societies (AMA, AAOS, AAFP, ACC/AHA, etc.)
- US-3b: Research Agent searches Tier 3 peer-reviewed literature (PubMed, JAMA, NEJM, NIH, CDC publications)
- US-4: Spec Agent extracts denominator/numerator logic

**Specification & Build Phase**:
- US-5: Spec Agent maps concepts to clinical codes in claims data
- US-5a: Spec Agent runs/tests specifications against Medicare/Medicaid claims datasets
- US-5b: Spec Agent runs/tests specifications against commercial claims datasets
- US-5c: Build Agent generates individual denominator/numerator components in datasets (metric development build)

**Initial Feasibility QA Phase**:
- US-5d: Feasibility QA - Claims Source check (can we measure all metric components using claims data?)
- US-5e: Feasibility QA - Volume check (is there enough provider and patient volume to make it a suitable metric?)
- US-5f: Feasibility QA - Opportunity check (does the metric result/distribution show sufficient opportunity for improvement? Do results distinguish high vs low performers?)
- US-5g: Feasibility QA - Comparison check (are evidence-based benchmarks or results available for comparison to Clarify output?)
- US-5h: Feasibility QA - Practice-reflective check (are results reflective of physician practice patterns, not billing nuances or data abnormalities?)
- US-5i: Feasibility QA Agent finalizes SQL logic for denominator/numerator/exclusions
- US-5j: Feasibility QA Agent compiles complete metric specification document with all components

**Human Review Phase (Checkpoint 1)**:
- US-6: Reviewer sees finalized SQL logic and comprehensive metric specification with citations and feasibility results
- US-6a: Reviewer validates SQL logic accuracy and completeness
- US-6b: Reviewer approves specification and SQL logic for production build
- US-7: Reviewer rejects spec/SQL with feedback (loops back to Research or Spec phase)

**Query & Execution Phase**:
- US-8: Query Builder translates specs to Athena SQL
- US-9: Query Builder accesses Argos observations
- US-10: Execution Agent runs Athena queries
- US-11: Execution Agent triggers Airflow EMR clusters
- US-12: Execution Agent handles failures gracefully

**Analysis & QA Phase**:
- US-13: Analysis Agent computes volumes
- US-14: Analysis Agent calculates distributions
- US-15: Analysis Agent compares to benchmarks
- US-16: QA Agent identifies outliers
- US-17: QA Agent simulates patient deep-dives
- US-18: QA Agent generates comprehensive report
- US-19: Reviewer examines QA report for sign-off

**Dashboard Phase**:
- US-20: View all metrics with current status
- US-21: See audit trail for decisions
- US-22: Receive review notifications
- US-23: Export specs to Confluence
- US-24: Monitor agent performance

## List of Features

1. Natural Language Metric Request Interface
2. Multi-Source Clinical Research Engine (Tier 1: National orgs, Tier 2: Specialty societies, Tier 3: Peer-reviewed literature)
3. Metric Specification Generator (LLM-powered)
4. Clinical Code Lookup Service (validation/search/expansion)
5. Claims Data Code Mapping Service (ICD-10, CPT, HCPCS, GPI mapping)
6. Medicare/Medicaid Claims Testing Engine
7. Commercial Claims Testing Engine
8. Denominator/Numerator Component Builder
9. Feasibility QA - Claims Source Validator
10. Feasibility QA - Volume Analyzer
11. Feasibility QA - Opportunity Assessor (high/low performer differentiation)
12. Feasibility QA - Benchmark Comparison Engine
13. Feasibility QA - Practice-Reflective Validator (billing nuance detection)
14. SQL Logic Finalizer (denom/numer/exclusion SQL generation)
15. Metric Specification Document Compiler (consolidates all components, citations, feasibility results)
16. Human Review Workflow - SQL & Spec Validation Interface
17. SQL Query Generator (production Athena-compatible)
18. Argos Observation Integration
19. Athena Query Execution Engine
20. Airflow EMR Cluster Orchestration
21. Volume & Distribution Analyzer
22. Benchmark Comparison Engine (external benchmarks)
23. Patient Deep-Dive Simulator
24. Comprehensive QA Report Generator
25. Interactive QA Review Dashboard
26. Workflow Status & Progress Tracker
27. Audit Trail & Versioning System
28. Notification & Alert System
29. Error Handling & Human Escalation
30. Clinical Code Database & Management

## List of PRs

| PR# | Title | Epic | Files | LOC |
|-----|-------|------|-------|-----|
| PR-1 | Project Setup | Epic 1 | 15+ | 500 |
| PR-2 | LangGraph State Machine | Epic 1 | 5 | 800 |
| PR-3 | Agent Base Classes | Epic 1 | 4 | 600 |
| PR-4 | PubMed Research Agent | Epic 2 | 4 | 700 |
| PR-5 | Web Scraping Agents | Epic 2 | 5 | 900 |
| PR-6 | Specification Generator | Epic 2 | 5 | 800 |
| PR-7 | Clinical Code Lookup | Epic 7 | 4 | 600 |
| PR-8 | Specification Review UI | Epic 5 | 8 | 1200 |
| PR-9 | SQL Query Generator | Epic 3 | 6 | 1000 |
| PR-10 | Argos Client | Epic 6 | 3 | 400 |
| PR-11 | Athena Service | Epic 3 | 4 | 800 |
| PR-12 | Airflow Client | Epic 6 | 3 | 400 |
| PR-13 | Execution Agent | Epic 3 | 5 | 700 |
| PR-14 | Volume Analysis | Epic 4 | 4 | 600 |
| PR-15 | Distribution Analysis | Epic 4 | 4 | 600 |
| PR-16 | Benchmark Comparison | Epic 4 | 3 | 500 |
| PR-17 | Patient Deep-Dive | Epic 4 | 5 | 800 |
| PR-18 | QA Report Generator | Epic 4 | 6 | 900 |
| PR-19 | QA Review Dashboard | Epic 5 | 10 | 1500 |
| PR-20 | Workflow Dashboard | Epic 5 | 7 | 1000 |
| PR-21 | Audit Trail | Epic 8 | 7 | 700 |
| PR-22 | Notifications | Epic 5 | 5 | 600 |
| PR-23 | Error Handling | Epic 8 | 4 | 600 |
| PR-24 | Monitoring | Epic 8 | 9 | 800 |
| PR-25 | Integration Tests | Epic 8 | 8 | 1200 |

**Total**: ~18,000 LOC (12K backend, 6K frontend)

## Best Practices

### Code Style
- Python: Black formatting, Ruff linting, mypy type checking, Google-style docstrings REQUIRED
- TypeScript: Prettier, ESLint strict, no `any` types, functional components only
- NO one-liner comments - code should be self-documenting
- Docstrings/JSDoc REQUIRED for all classes and functions

### Design Patterns
- SOLID principles: Single responsibility, dependency injection
- Idempotent agents: Same input → same output
- State immutability: Never mutate LangGraph state, return new object
- Error boundaries: Try/catch around agent execution with escalation

### Security
- NEVER commit secrets - use AWS Secrets Manager
- Parameterized SQL queries ONLY - no f-strings (SQL injection risk)
- Validate all inputs with Pydantic
- Minimal PHI exposure - anonymize in logs

### Testing
- 80%+ test coverage required
- Mock external services (PubMed, Athena, Argos, Claude API)
- Test edge cases (null, empty, errors)
- Regression tests against golden metrics dataset

### Take a Breath Before Coding
MANDATORY: Before writing code, (1) read ticket completely, (2) review dependencies, (3) check similar code, (4) write implementation plan in comments, (5) identify edge cases, (6) THEN implement.

## Architecture Diagrams

### System Architecture
```
User → Web Dashboard → FastAPI → LangGraph
                                     ↓
    ┌──────────────────────────────────────────────────────────────────────────┐
    │                         RESEARCH & SPECIFICATION PHASE                     │
    │                                                                            │
    │  Research Agent ──────────────────────────────────────► Spec Agent         │
    │       │                                                      │             │
    │       ├─► Tier 1: USPSTF, HEDIS, CMS, AHRQ, CDC             │             │
    │       ├─► Tier 2: AMA, AAOS, AAFP, ACC/AHA                  │             │
    │       └─► Tier 3: PubMed, JAMA, NEJM, NIH                   ▼             │
    │                                                   Extract Denom/Numer      │
    │                                                   Map to Clinical Codes    │
    └──────────────────────────────────────────────────────────────────────────┘
                                     ↓
    ┌──────────────────────────────────────────────────────────────────────────┐
    │                         BUILD & FEASIBILITY PHASE                          │
    │                                                                            │
    │  Build Agent ──► Test Against Claims Data                                  │
    │       │              │                                                     │
    │       │              ├─► Medicare/Medicaid datasets                        │
    │       │              └─► Commercial claims datasets                        │
    │       │                                                                    │
    │       └──────────► Generate Individual Denom/Numer Components              │
    │                              ↓                                             │
    │  Feasibility QA Agent ──► Initial QA Checks                                │
    │       │                                                                    │
    │       ├─► Claims Source: Can we measure all components?                    │
    │       ├─► Volume: Enough provider/patient volume?                          │
    │       ├─► Opportunity: Sufficient improvement opportunity?                 │
    │       ├─► Comparison: Benchmarks available?                                │
    │       └─► Practice-reflective: True practice patterns?                     │
    │                              ↓                                             │
    │  ┌────────────────────────────────────────────────────────────────────┐   │
    │  │              FINALIZE SQL LOGIC & SPECIFICATIONS                    │   │
    │  │  • Finalize denominator/numerator/exclusion SQL                     │   │
    │  │  • Compile complete metric specification document                   │   │
    │  │  • Include all citations, codes, and feasibility results            │   │
    │  └────────────────────────────────────────────────────────────────────┘   │
    └──────────────────────────────────────────────────────────────────────────┘
                                     ↓
    ┌──────────────────────────────────────────────────────────────────────────┐
    │                      [HUMAN CHECKPOINT 1]                                  │
    │                                                                            │
    │  Reviewer validates:                                                       │
    │    • Finalized SQL logic (denom/numer/exclusions)                         │
    │    • Complete metric specification                                         │
    │    • Feasibility QA results                                               │
    │    • Clinical accuracy and code mappings                                  │
    │                                                                            │
    │  Actions: APPROVE (proceed) or REJECT with feedback (loop back)           │
    └──────────────────────────────────────────────────────────────────────────┘
                                     ↓
    ┌──────────────────────────────────────────────────────────────────────────┐
    │                      QUERY & EXECUTION PHASE                               │
    │                                                                            │
    │  Query Builder → Execution Agent → Analysis Agent → QA Agent               │
    │       ↓              ↓                  ↓              ↓                   │
    │   Argos/SQL     Athena/Airflow       Athena        Claude                  │
    │  (Production)                                                              │
    └──────────────────────────────────────────────────────────────────────────┘
                                     ↓
                           [HUMAN CHECKPOINT 2]
                              QA Sign-off
                                     ↓
                   S3 Results → PostgreSQL → Audit Trail
```

### Workflow State Machine
```
PENDING → RESEARCHING → GENERATING_SPEC → MAPPING_CODES 
                                               ↓
                              TESTING_MEDICARE_MEDICAID → TESTING_COMMERCIAL
                                                               ↓
                                                    BUILDING_COMPONENTS
                                                               ↓
                              ┌─────────────────────────────────┐
                              │    FEASIBILITY_QA_CHECKS        │
                              │  ├─ Claims Source Validation    │
                              │  ├─ Volume Analysis             │
                              │  ├─ Opportunity Assessment      │
                              │  ├─ Benchmark Comparison        │
                              │  └─ Practice-Reflective Check   │
                              └─────────────────────────────────┘
                                               ↓
                              ┌─────────────────────────────────┐
                              │    FINALIZING_SQL_AND_SPECS     │
                              │  ├─ Finalize Denom SQL          │
                              │  ├─ Finalize Numer SQL          │
                              │  ├─ Finalize Exclusion SQL      │
                              │  └─ Compile Spec Document       │
                              └─────────────────────────────────┘
                                               ↓
                                    AWAITING_SPEC_REVIEW
                                               ↓
                                     [HUMAN CHECKPOINT 1]
                              Reviewer validates SQL + Specs
                                               ↓
                                Approved → BUILDING_QUERIES → EXECUTING
                                Rejected → (loop to RESEARCHING or GENERATING_SPEC)
                                                                  ↓
                                ANALYZING → GENERATING_QA → AWAITING_QA_REVIEW
                                                                  ↓
                                                        [HUMAN CHECKPOINT 2]
                                                                  ↓
                                                Approved → COMPLETE
                                                Rejected → (loop to BUILDING_QUERIES)
```

## Methodology

**Agile 2-week sprints**:
- Sprint 1-2: Epic 1 (Agent framework)
- Sprint 3-4: Epic 2 (Research & specs)
- Sprint 5-6: Epic 3 (Query building)
- Sprint 7-8: Epic 4 (Analysis & QA)
- Sprint 9-10: Epic 5 (Dashboard)
- Sprint 11-12: Epics 6+7 (Integrations & codes)
- Sprint 13-14: Epic 8 (Testing & monitoring)
- Sprint 15: Pilot with 3-5 real metrics

**Claude Code Workflow**:
1. Assign ticket
2. Branch: `feature/TICKET-XXX-description`
3. Claude Code: "Implement TICKET-XXX following PRD"
4. Incremental commits
5. Local tests
6. Push & create PR
7. Code review
8. Merge to main

## List of Tickets

| ID | Title | Epic | Priority | SP | Dependencies |
|----|-------|------|----------|-----|--------------|
| TICKET-001 | Initialize monorepo | Epic 1 | P0 | 3 | None |
| TICKET-002 | LangGraph state machine | Epic 1 | P0 | 8 | 001 |
| TICKET-003 | Agent base classes | Epic 1 | P0 | 5 | 002 |
| TICKET-004 | PubMed research agent | Epic 2 | P0 | 5 | 003 |
| TICKET-005 | Web scraping agents | Epic 2 | P1 | 8 | 003 |
| TICKET-006 | Specification generator | Epic 2 | P0 | 8 | 004,005 |
| TICKET-007 | Spec review UI | Epic 5 | P0 | 8 | 006 |
| TICKET-008 | SQL query generator | Epic 3 | P0 | 8 | 006 |
| TICKET-009 | Argos integration | Epic 6 | P1 | 5 | 001 |
| TICKET-010 | Athena service | Epic 3 | P0 | 8 | 008 |
| TICKET-011 | Airflow client | Epic 6 | P1 | 5 | 001 |
| TICKET-012 | Execution orchestrator | Epic 3 | P0 | 5 | 010,011 |
| TICKET-013 | Volume analysis | Epic 4 | P0 | 5 | 012 |
| TICKET-014 | Distribution analysis | Epic 4 | P0 | 5 | 012 |
| TICKET-015 | Benchmark comparison | Epic 4 | P1 | 5 | 004,013 |
| TICKET-016 | Patient deep-dive | Epic 4 | P1 | 8 | 012 |
| TICKET-017 | QA report generator | Epic 4 | P0 | 8 | 013-016 |
| TICKET-018 | QA review dashboard | Epic 5 | P0 | 8 | 017 |
| TICKET-019 | Metric request form | Epic 5 | P0 | 3 | 001 |
| TICKET-020 | Workflow dashboard | Epic 5 | P1 | 5 | 002 |
| TICKET-021 | Audit trail viewer | Epic 5 | P2 | 5 | 002 |
| TICKET-022 | Notifications | Epic 5 | P1 | 5 | 002 |
| TICKET-023 | S3 artifact manager | Epic 6 | P1 | 3 | 001 |
| TICKET-024 | Auth & authorization | Epic 6 | P1 | 5 | 001 |
| TICKET-025 | Clinical code database | Epic 7 | P0 | 5 | 001 |
| TICKET-026 | Code lookup service | Epic 7 | P0 | 5 | 025 |
| TICKET-027 | Code versioning | Epic 7 | P2 | 3 | 025 |
| TICKET-028 | Unit testing framework | Epic 8 | P0 | 3 | 001 |
| TICKET-029 | Integration tests | Epic 8 | P1 | 5 | 002 |
| TICKET-030 | E2E tests | Epic 8 | P1 | 8 | 017 |
| TICKET-031 | Structured logging | Epic 8 | P1 | 3 | 001 |
| TICKET-032 | Prometheus & Grafana | Epic 8 | P1 | 5 | 001 |
| TICKET-033 | Error handling | Epic 8 | P1 | 5 | 002 |

**Total**: 33 tickets, 173 story points, 14-16 sprints (7-8 months)

## Ticket Breakdown (Detailed Examples)

### TICKET-001: Initialize monorepo
**Epic**: Epic 1 | **Priority**: P0 | **SP**: 3 | **Time**: 2-3 days

**Subtasks**:
1. Create directory structure (backend/, frontend/, shared/, docs/)
2. Configure pyproject.toml with dependencies (fastapi, langgraph, anthropic, boto3, sqlalchemy, redis, etc.)
3. Configure package.json (react, typescript, vite, tanstack-query, zustand, shadcn/ui)
4. Create docker-compose.yml (postgres, redis, backend, frontend)
5. Set up GitHub Actions CI/CD (lint, test, build)
6. Write README.md with setup instructions
7. Verify: `docker-compose up` → all services start

**Acceptance Criteria**:
- Monorepo structure created
- Dependencies install successfully
- Docker Compose brings up all services
- CI/CD pipeline passes
- README has clear setup steps

---

### TICKET-002: LangGraph state machine with checkpoints
**Epic**: Epic 1 | **Priority**: P0 | **SP**: 8 | **Time**: 4-5 days | **Depends**: TICKET-001

**Subtasks**:
1. Define MetricWorkflowState TypedDict with all fields
2. Create StateGraph with nodes for each agent
3. Add conditional edges for approval routing
4. Implement interrupt_before for human checkpoints
5. Set up PostgreSQL checkpointer for persistence
6. Create MetricWorkflow database model
7. Build WorkflowOrchestrator service (start/resume/status)
8. Add API endpoints (POST /metrics, POST /metrics/{id}/approve)
9. Unit tests for state transitions and checkpoints

**Acceptance Criteria**:
- StateGraph defined with all agent nodes
- Checkpoints interrupt workflow for human input
- State persists to PostgreSQL
- Workflow resumes after approval
- Rejection loops back correctly
- API endpoints working

---

### TICKET-004: PubMed research agent
**Epic**: Epic 2 | **Priority**: P0 | **SP**: 5 | **Time**: 3-4 days | **Depends**: TICKET-003

**Subtasks**:
1. Create PubMedClient with search_articles and fetch_article_details
2. Implement rate limiting (3 req/sec)
3. Build PubMedResearchAgent with execute method
4. Extract search keywords using Claude
5. Extract benchmarks from abstracts using Claude
6. Rank findings by relevance
7. Store findings in database
8. Unit tests with mocked PubMed API

**Acceptance Criteria**:
- PubMed search retrieves relevant articles
- Benchmarks extracted with citations
- Rate limiting prevents throttling
- Findings ranked correctly
- All interactions logged
- Tests pass with mocked API

---

(Additional tickets follow similar detailed format...)

## IDE Rules (Claude Code)

**File**: `.claude_code_rules`

```markdown
# Quality Metric Builder - AI Assistant Rules

## Project: Healthcare quality metric automation with LangGraph + multi-agent system

## Code Style - MANDATORY

### Python
- Black formatting (100 chars), Ruff linting, mypy strict
- Type hints REQUIRED for all signatures
- Docstrings REQUIRED (Google style)
- NO one-liner comments

### TypeScript
- Prettier, ESLint strict, NO `any` types
- Functional components with hooks only
- Props interfaces REQUIRED
- JSDoc for complex logic

## Design Principles

### SOLID
- Single responsibility per agent
- Dependency injection (services injected, not instantiated)
- Open/closed (extend via tools, not modification)

### LangGraph
- Idempotent agents (same input → same output)
- Never mutate state (return new dict)
- Checkpoint after every agent
- interrupt_before for human checkpoints

### Security
- NEVER commit secrets (use AWS Secrets Manager)
- Parameterized SQL only (no f-strings - SQL injection)
- Validate all inputs (Pydantic)
- Minimal PHI in logs

## Take a Breath Before Coding

BEFORE implementing ANY code:
1. Read ticket description completely
2. Review dependencies and PRD section
3. Check for similar existing code
4. Write implementation plan as comments
5. Identify edge cases
6. THEN implement

## Testing Rules
- 80%+ coverage required
- Mock external services (PubMed, Athena, Argos, Claude)
- Test edge cases (null, empty, errors)
- Test naming: `test_<function>_<scenario>_<expected>`

## Dependencies

### Always Check Latest Docs
Before using any library:
1. Check PyPI/npm for latest stable version
2. Read official docs
3. Use Context7 MCP to fetch latest docs if available
4. Test in isolation before integrating

### Pre-Approved
Backend: fastapi, langchain, langgraph, anthropic, boto3, sqlalchemy, pydantic
Frontend: react, typescript, tanstack-query, zustand, shadcn/ui, tailwindcss

## Commit Format
```
<type>(<scope>): <subject>

<body>

Refs: TICKET-XXX
```
Types: feat, fix, docs, refactor, test
Scopes: research, spec, query, execution, analysis, qa, ui, api, db

## Checklist Before Commit
- [ ] Linting passes (ruff/eslint)
- [ ] Type checking passes (mypy/tsc)
- [ ] Tests pass and added for new code
- [ ] Docstrings added
- [ ] No secrets committed
- [ ] No debug code (print/console.log)
- [ ] Audit logging added for agent actions
```

---

*End of PRD*
