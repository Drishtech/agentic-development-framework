# Agentic Documentation and Development Framework

This framework is LLM-agnostic and is designed for projects where the initial design is signed off with human intervention, but the subsequent implementation is executed by autonomous agents with minimal supervision. The goal is for any agent to be able to operate with minimal context without contradicting previous decisions.

## 0. Core Rule

The repo is the shared memory. Every agent must be able to operate by reading:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. Their `SPRINT-N.md`
5. Their `TASK-ID.md`
6. Contracts and decisions explicitly cited by the task

If a task requires inferring uncited context, the task is poorly written and the Checker must fail it.

## A. Final Directory Tree

ASCII names are used (`design`, not `diseño`) to avoid issues with tooling, shells, and smaller models. Content can be in Spanish or English.

| Path | Purpose | Readers | Single Writer | Mutability | Max tokens | If corrupted or lost |
|---|---|---:|---|---|---:|---|
| `README.md` | Navigable index and 90s pitch | human/planner/executor/checker | human | controlled editable | 1200 | Rebuilt from `docs/CURRENT.md` and roadmap |
| `AGENTS.md` | Mandatory contract for any LLM | planner/executor/checker | human | immutable post-sign-off except `CHANGE` | 1800 | Work blocked until restored |
| `docs/` | Document root | all | human | stable structure | n/a | Restored from git/backups |
| `docs/CURRENT.md` | Minimum living state: phase, active sprint, active decisions | all | planner | editable append-summarized | 1500 | Rebuilt from sprint/decisions |
| `docs/design/` | Signed Phase 1 design | human/planner/checker | human | immutable post-sign-off | n/a | Major changes blocked |
| `docs/design/DESIGN.md` | Full functional and technical design, divided by sections | human/planner/checker | human | immutable post-sign-off | 3500 | Sprint cannot start |
| `docs/design/SIGNOFF.md` | Human sign-off on scope, constraints, and acceptance | human/checker | human | append-only | 1000 | No valid baseline exists |
| `docs/decisions/` | ADRs | planner/checker/executor if cited | planner | append-only | n/a | Index used as partial recovery |
| `docs/decisions/DECISION-NNN.md` | Atomic architectural decision | all if cited | planner | append-only; supersede, do not edit | 1200 | If cited, task blocked |
| `docs/decisions/INDEX.md` | Short log of active/superseded decisions | all | checker | generated/validated | 1800 | Regenerated from ADRs |
| `docs/guardrails/` | Compact rules derived from decisions | planner/executor/checker | checker | generated/validated | n/a | Regenerated from ADRs |
| `docs/guardrails/ARCHITECTURE_LOCKS.md` | Active invariants that no sprint can contradict | all | checker | generated from ADRs | 1500 | Executor cannot start |
| `docs/implementation/` | Executable sprint plans | planner/executor/checker | planner | append-only per sprint | n/a | Restored from tasks |
| `docs/implementation/ROADMAP.md` | Planned phases, not a detailed commitment | human/planner | planner | editable via approved change | 1500 | Does not block active tasks |
| `docs/implementation/SPRINT-N.md` | Closed sprint plan | all | planner | immutable post-handoff | 2200 | Sprint cannot close |
| `docs/tasks/` | Atomic tasks executable by small models | executor/checker | planner | executor only completes evidence fields | n/a | Task blocked |
| `docs/tasks/TASK-ID.md` | Work unit with context, TDD, paths, gates | executor/checker | planner | evidence fields editable | 2500 | Not executable |
| `docs/changes/` | Changes to signed baseline | human/planner/checker | planner | append-only | n/a | Major changes blocked |
| `docs/changes/CHANGE-NNN.md` | Minor/major/breaking change request/approval | human/planner/checker | planner | append-only with signature | 1200 | Cannot alter design/ADR |
| `docs/testing/` | Strategy and validation contracts | planner/executor/checker | checker | editable via approved change | n/a | Checker uses existing tests and fails new coverage |
| `docs/testing/TEST_STRATEGY.md` | Pyramid, commands, minimums per change type | all | checker | editable via `CHANGE` | 1500 | Sprint cannot close |
| `docs/testing/ACCEPTANCE.md` | Global acceptance criteria | human/planner/checker | human | immutable post-sign-off | 1500 | Closure blocked |
| `docs/checks/` | Verification evidence | checker | checker | append-only | n/a | Checks re-run |
| `docs/checks/CHECK-SPRINT-N.md` | Checker result | human/checker | checker | append-only | 1500 | Sprint not closed |
| `scripts/verify_sprint.py` | Verifiable local gate | checker/planner | checker | editable with own test | 2000 | Manual closure invalid |
| `scripts/verify_docs.py` | Validates links, ADRs, changes, locks | checker | checker | editable with own test | 2000 | Drift not detectable |

## B. Real and Ready-to-copy Templates

### `README.md`

```markdown
# {{PROJECT_NAME}}

## What it is
System: {{PROJECT_NAME}}.
Primary user: {{PRIMARY_USER_ROLE}}.
Observable outcome: {{ONE_SENTENCE_OUTCOME}}.

## Current state
Active phase: {{PHASE_NUMBER}}.
Active sprint: {{SPRINT_NUMBER_OR_NONE}}.
Signed baseline: `docs/design/SIGNOFF.md`.
Living summary: `docs/CURRENT.md`.

## Read in 90 seconds
1. `AGENTS.md` for mandatory rules.
2. `docs/CURRENT.md` for current state.
3. `docs/guardrails/ARCHITECTURE_LOCKS.md` for non-negotiable constraints.
4. Active sprint in `docs/implementation/SPRINT-{{NNN}}.md`.

## Documentation map
- Signed design: `docs/design/DESIGN.md`
- Decisions: `docs/decisions/INDEX.md`
- Implementation: `docs/implementation/`
- Tasks: `docs/tasks/`
- Testing: `docs/testing/`
- Approved changes: `docs/changes/`

## Standard commands
Install: `{{INSTALL_COMMAND}}`
Fast tests: `{{FAST_TEST_COMMAND}}`
Full tests: `{{FULL_TEST_COMMAND}}`
Verify sprint: `python scripts/verify_sprint.py docs/implementation/SPRINT-{{NNN}}.md`
```

### `AGENTS.md`

```markdown
# AGENTS.md

## Mandatory Contract

This repo is worked on using Planner, Executor, and Checker. No agent can skip its role.

## Non-negotiable Rules

1. Red/green TDD mandatory for every behavior change.
2. YAGNI: Do not create abstractions, services, queues, adapters, or configurations not required by a task.
3. DRY: If a semantic repetition occurs for the third time, stop and propose a refactoring within the task or open a change.
4. No agent contradicts `docs/guardrails/ARCHITECTURE_LOCKS.md`.
5. No agent edits documents outside its R/W/V permission scope.
6. If context is missing, the Executor must mark `BLOCKED_CONTEXT_MISSING`, do not infer.
7. Every signed design change requires `CHANGE-NNN.md`.
8. Every active decision change requires a new ADR that supersedes the previous one.

## Maximum Reading per Role

Planner:
- Must read `README.md`, `AGENTS.md`, `docs/CURRENT.md`, `docs/decisions/INDEX.md`, `docs/guardrails/ARCHITECTURE_LOCKS.md`.
- Can read a maximum of 3 additional design documents per sprint.
- Can read a maximum of 2 previous complete sprints.

Executor:
- Must read `AGENTS.md`, `docs/CURRENT.md`, their `SPRINT-N.md`, their `TASK-ID.md`, and cited ADRs.
- Must not read the entire repo.
- Must not touch `docs/design/`, `docs/decisions/`, `docs/changes/`.

Checker:
- Must read `AGENTS.md`, `SPRINT-N.md`, sprint tasks, diff of modified files, declared tests, and cited ADRs.
- Validates, does not implement features.

## Blocking Rule

Use exactly one of these marks:
- `BLOCKED_CONTEXT_MISSING`
- `BLOCKED_DECISION_CONFLICT`
- `BLOCKED_TEST_UNCLEAR`
- `BLOCKED_SCOPE_TOO_LARGE`

Every mark must include the file, section, and specific question.
```

### `docs/design/DESIGN.md`

```markdown
# DESIGN.md

Status: SIGNED | DRAFT
Project: {{PROJECT_NAME}}
Design version: {{DESIGN_VERSION}}
Human owner: {{HUMAN_OWNER}}
Signed in: `docs/design/SIGNOFF.md`

## 1. Objective
Business outcome: {{BUSINESS_OUTCOME}}
In-scope users: {{IN_SCOPE_USERS}}
Out-of-scope users: {{OUT_OF_SCOPE_USERS}}

## 2. Signed Scope
Included:
- {{IN_SCOPE_CAPABILITY_1}}
- {{IN_SCOPE_CAPABILITY_2}}

Explicitly excluded:
- {{OUT_OF_SCOPE_CAPABILITY_1}}
- {{OUT_OF_SCOPE_CAPABILITY_2}}

## 3. Domain
Entities:
- {{ENTITY_NAME}}: required fields={{FIELDS}}, invariants={{INVARIANTS}}

Rules:
- {{RULE_ID}}: {{RULE_STATEMENT}}, source={{SOURCE}}

## 4. Interfaces
API/UI/CLI:
- Contract ID: {{CONTRACT_ID}}
- Operation: {{OPERATION_NAME}}
- Valid input: {{VALID_INPUT_SHAPE}}
- Valid output: {{VALID_OUTPUT_SHAPE}}
- Expected errors: {{ERROR_CODES}}

## 5. Data
Persistence: {{PERSISTENCE_CHOICE}}
Migrations required: {{YES_NO}}
Sensitive data: {{SENSITIVE_DATA_CLASSES}}

## 6. Constraints
Performance: {{PERFORMANCE_LIMIT}}
Security: {{SECURITY_LIMIT}}
Operations: {{OPERATIONS_LIMIT}}
Cost: {{COST_LIMIT}}

## 7. Accepted Testing
Minimum command: {{FAST_TEST_COMMAND}}
Full command: {{FULL_TEST_COMMAND}}
Global acceptance criteria: `docs/testing/ACCEPTANCE.md`

## 8. Signed Risks
- Risk: {{RISK_NAME}}
  Probability: LOW | MEDIUM | HIGH
  Impact: LOW | MEDIUM | HIGH
  Mandatory mitigation: {{MITIGATION}}
```

### `docs/implementation/SPRINT-N.md`

```markdown
# SPRINT-{{NNN}}

Status: PLANNED | IN_PROGRESS | CHECKING | CLOSED | BLOCKED
Planner: {{PLANNER_ID}}
Checker: {{CHECKER_ID}}
Created: {{YYYY-MM-DD}}
Change class: MINOR | MAJOR | BREAKING
Authorized by: PLANNER | HUMAN

## Sprint Objective
Verifiable outcome: {{OBSERVABLE_OUTCOME}}

## Mandatory Reading for Executor
1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. This file
5. Assigned task in `docs/tasks/`

## Relevant Active Decisions
- {{DECISION_KEY}} -> `docs/decisions/DECISION-{{NNN}}.md`

## Tasks
- `docs/tasks/TASK-{{TASK_ID_1}}.md` owner={{EXECUTOR_ROLE}} status=READY
- `docs/tasks/TASK-{{TASK_ID_2}}.md` owner={{EXECUTOR_ROLE}} status=READY

## Allowed Paths
- {{ALLOWED_PATH_PATTERN_1}}
- {{ALLOWED_PATH_PATTERN_2}}

## Forbidden Paths
- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- {{FORBIDDEN_PATH_PATTERN}}

## Validation Commands
Expected red test before implementation: {{RED_TEST_COMMAND}}
Minimum green test: {{FAST_TEST_COMMAND}}
Full green test: {{FULL_TEST_COMMAND}}

## Handoff to Executor
The Executor can only start if all tasks are in `READY` status and every task has acceptance criteria, expected red test, allowed paths, and relevant ADRs.

## Handoff to Checker
The sprint moves to `CHECKING` when all tasks are in `IMPLEMENTED` and contain red/green evidence.
```

### `docs/tasks/TASK-ID.md`

```markdown
# TASK-{{ID}}

Status: READY | IN_PROGRESS | IMPLEMENTED | CHECK_FAILED | CLOSED | BLOCKED
Sprint: `docs/implementation/SPRINT-{{NNN}}.md`
Executor: {{EXECUTOR_ID_OR_ROLE}}
Change class: MINOR | MAJOR | BREAKING

## Atomic Objective
Upon completing this task, the following will be true: {{SINGLE_VERIFIABLE_STATEMENT}}

## Minimum Context
Read only:
- `AGENTS.md`
- `docs/CURRENT.md`
- `docs/guardrails/ARCHITECTURE_LOCKS.md`
- `docs/implementation/SPRINT-{{NNN}}.md`
- This task
- `docs/decisions/DECISION-{{NNN}}.md` because {{DECISION_REASON}}

## Decisions Touched
- Uses: {{DECISION_KEY}}
- Changes: NONE | `CHANGE-{{NNN}}.md`

## Allowed Files
- {{ALLOWED_FILE_OR_GLOB}}

## Forbidden Files
- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- {{TASK_SPECIFIC_FORBIDDEN_GLOB}}

## Mandatory TDD
Red test that must fail first:
- Test file: {{TEST_FILE}}
- Test name: {{TEST_NAME}}
- Command: {{RED_TEST_COMMAND}}
- Exact expected failure reason: {{EXPECTED_FAILURE_REASON}}

Red evidence pasted by Executor:
```text
PENDING_RED_OUTPUT
```

Minimum allowed implementation:
- {{MINIMAL_IMPLEMENTATION_BOUNDARY}}

Required green tests:
- {{FAST_TEST_COMMAND}}
- {{FULL_OR_TARGETED_TEST_COMMAND}}

Green evidence pasted by Executor:
```text
PENDING_GREEN_OUTPUT
```

## Mandatory Cases
Happy path:
- {{HAPPY_PATH_ASSERTION}}

Edge cases:
- {{EDGE_CASE_1}}
- {{EDGE_CASE_2}}

Negative cases:
- {{NEGATIVE_CASE_1}}

## Anti-minimal-effort
The Executor must complete:
- [ ] Red test exists before production code.
- [ ] Test fails for the expected reason, not due to syntax/config errors.
- [ ] Every acceptance criterion has at least one assertion.
- [ ] No abstractions were added outside the atomic objective.
- [ ] Existing logic was not duplicated; if duplication exists, indicate file and justification.
- [ ] Forbidden files were not touched.
- [ ] No new `TODO`, `FIXME`, `PLACEHOLDER`, or skips remain.

## Acceptance Criteria
- AC1: {{ASSERTABLE_ACCEPTANCE_CRITERION}}
- AC2: {{ASSERTABLE_ACCEPTANCE_CRITERION}}

## Executor Notes
Changed files:
- PENDING_CHANGED_FILE_LIST

Decisions made within the task:
- NONE

Blockers:
- NONE
```

### `docs/changes/CHANGE-NNN.md`

```markdown
# CHANGE-{{NNN}}

Status: PROPOSED | APPROVED | REJECTED | APPLIED
Class: MINOR | MAJOR | BREAKING
Requested by: HUMAN | PLANNER | CHECKER
Approver required: PLANNER | HUMAN
Approved by: {{APPROVER_ID}}
Date: {{YYYY-MM-DD}}

## Requested Change
Change from: {{CURRENT_SIGNED_BEHAVIOR_OR_DECISION}}
Change to: {{NEW_BEHAVIOR_OR_DECISION}}

## Reason
Operational reason: {{CONCRETE_REASON}}
Cost of not doing it: {{CONCRETE_RISK}}

## Impact
Signed design affected:
- `docs/design/DESIGN.md#{{SECTION_ID}}` | NONE

Decisions affected:
- `docs/decisions/DECISION-{{NNN}}.md` | NONE

Contracts affected:
- {{CONTRACT_ID}} | NONE

Data/migrations:
- NONE | {{MIGRATION_ID}}

## Mandatory Action
- [ ] If changing signed design, update version and `SIGNOFF.md`.
- [ ] If changing active ADR, create new ADR with `Supersedes`.
- [ ] If changing public contract, classify as BREAKING.
- [ ] If changing testing strategy, update `docs/testing/`.

## Outcome
Applied in sprint: `docs/implementation/SPRINT-{{NNN}}.md`
Checker report: `docs/checks/CHECK-SPRINT-{{NNN}}.md`
```

### `docs/decisions/DECISION-NNN.md`

```markdown
# DECISION-{{NNN}}: {{DECISION_TITLE}}

Status: PROPOSED | ACCEPTED | SUPERSEDED | REJECTED
Decision key: {{STABLE_KEY_UPPER_SNAKE}}
Date: {{YYYY-MM-DD}}
Owner: PLANNER
Supersedes: NONE | DECISION-{{NNN}}
Superseded by: NONE | DECISION-{{NNN}}
Change request: NONE | `docs/changes/CHANGE-{{NNN}}.md`

## Context
Specific problem: {{PROBLEM_STATEMENT}}
Applicable constraints:
- {{CONSTRAINT}}

## Decision
It is decided: {{ONE_CLEAR_DECISION}}

## Consequences
Positive:
- {{POSITIVE_CONSEQUENCE}}

Accepted negative:
- {{NEGATIVE_CONSEQUENCE}}

## Invariant for ARCHITECTURE_LOCKS
Lock ID: {{LOCK_ID}}
Rule: {{NON_NEGOTIABLE_RULE}}
Applies to paths:
- {{PATH_PATTERN}}

## How to Validate
Checker must verify:
- {{VALIDATION_RULE_1}}
- {{VALIDATION_RULE_2}}

## Discarded Alternatives
- Alternative: {{ALTERNATIVE_NAME}}
  Discarded because: {{SPECIFIC_REASON}}
```

## C. Planner -> Executor -> Checker Flow

### Planner

Reads at most:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. `docs/guardrails/ARCHITECTURE_LOCKS.md`
5. `docs/decisions/INDEX.md`
6. Up to 3 documents from `docs/design/`
7. Up to 2 previous sprints

Produces:

- `SPRINT-N.md`
- One or more `TASK-ID.md`
- `CHANGE-NNN.md` if touching signed baseline
- `DECISION-NNN.md` if creating/superseding architecture

Cannot touch:

- Production code
- Red/green evidence
- Checker reports

Exact Handoff:

> Change `SPRINT-N.md` to `PLANNED` and all its tasks to `READY`. If any task lacks an expected red test, allowed paths, and relevant decisions, the handoff is invalid.

### Executor

Reads at most:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. `SPRINT-N.md`
5. Their `TASK-ID.md`
6. ADRs cited by the task
7. Code files allowed by the task

Produces:

- Red test
- Minimum code
- Green test
- Evidence pasted in `TASK-ID.md`
- List of changed files

Cannot touch:

- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- `docs/testing/**` unless the task explicitly allows it

Exact Handoff:

> Change task status to `IMPLEMENTED`, paste red/green output, list modified files, and leave no open blockers.

### Checker

Reads at most:

1. `AGENTS.md`
2. `SPRINT-N.md`
3. Sprint tasks
4. `docs/guardrails/ARCHITECTURE_LOCKS.md`
5. Cited ADRs
6. Diff of modified files
7. Declared tests

Produces:

- `docs/checks/CHECK-SPRINT-N.md`
- Update to `docs/CURRENT.md`
- Regeneration/validation of `docs/decisions/INDEX.md`
- Regeneration/validation of `ARCHITECTURE_LOCKS.md`

Cannot touch:

- Production code, unless the user explicitly requests a CI fix
- Signed design

Exact Handoff:

> If all gates pass, mark sprint `CLOSED`, tasks `CLOSED`, write check report, and update `CURRENT.md`.

## D. Executable Definition of "Closed Sprint"

A sprint is considered closed only if this command finishes with exit code `0`:

```bash
python scripts/verify_sprint.py docs/implementation/SPRINT-005.md
```

Verifiable conditions:

1. `SPRINT-N.md` exists and status=`CHECKING` or `CLOSED`.
2. All referenced tasks exist.
3. Every task has status=`IMPLEMENTED` or `CLOSED`.
4. Every task contains `RED_OUTPUT` evidence distinct from `PENDING_RED_OUTPUT`.
5. Every task contains `GREEN_OUTPUT` evidence distinct from `PENDING_GREEN_OUTPUT`.
6. No task contains `TODO`, `FIXME`, `PLACEHOLDER`, or `PENDING_`.
7. Every modified file matches `allowed paths`.
8. No modified file matches `forbidden paths`.
9. Every `Decision key` exists in `docs/decisions/INDEX.md`.
10. No task changes a decision without an approved `CHANGE-NNN.md`.
11. If there is a major/breaking `CHANGE`, it has human approval.
12. Declared tests were executed and their output is logged.
13. `docs/checks/CHECK-SPRINT-N.md` exists with verdict=`PASS`.
14. `docs/CURRENT.md` references the closed sprint.
15. `ARCHITECTURE_LOCKS.md` is synced with accepted ADRs.

## E. Anti-drift Mechanism

"Using ADRs" is not enough. The mechanism relies on three layers.

### 1. Stable Decision Key

Each accepted ADR defines a `Decision key`, for example:

```text
EMAIL_PROVIDER_SENDGRID
AUTH_SESSION_JWT
DB_POSTGRES_PRIMARY
```

Tasks do not cite long narratives; they cite keys.

### 2. Mandatory Short Index

`docs/decisions/INDEX.md` contains only active decisions:

```markdown
# Decision Index

- key: EMAIL_PROVIDER_SENDGRID
  status: ACCEPTED
  adr: docs/decisions/DECISION-004.md
  lock: LOCK-EMAIL-001
  applies_to: src/email/**, config/email/**
  superseded_by: NONE
```

This allows sprint 12 to read a short file instead of sprints 1-11.

### 3. Generated Architecture Locks

`ARCHITECTURE_LOCKS.md` translates ADRs into rules executable via review:

```markdown
# Architecture Locks

## LOCK-EMAIL-001
Decision key: EMAIL_PROVIDER_SENDGRID
Rule: Production email delivery must use SendGrid through the email adapter boundary.
Applies to:
- `src/email/**`
- `config/email/**`
Checker validation:
- No direct AWS SES client in production email paths.
- Email provider config uses SENDGRID_API_KEY.
```

Anti-drift rule:

> If a task touches a path covered by a lock, it must cite its decision key. If it contradicts the lock, it must have an approved `CHANGE-NNN.md` and a new ADR with `Supersedes`.

## F. Anti-minimal-effort Mechanism

The framework forces behavior through gates, not goodwill:

1. Red/green TDD is in `TASK-ID.md` and `verify_sprint.py` fails if evidence is missing.
2. Every acceptance criterion must map to a test or assertion.
3. The Executor cannot write "I tested manually" as a substitute for the command.
4. The Checker rejects tasks with `PENDING_`, `TODO`, new skips, or forbidden files.
5. The Planner must write edge cases and negative cases; if they don't exist, the task is not `READY`.
6. YAGNI is enforced with "minimum allowed implementation": if the Executor adds unlisted infrastructure, it fails.
7. DRY is enforced with mandatory duplication declaration; if it introduces a third repetition without refactoring or justification, it fails.
8. Allowed paths prevent the agent from "fixing" the problem by touching unauthorized zones.
9. Locks prevent silent contradictions with previous architecture.

## G. R/W/V Matrix by Document and Role

| Document | Human | Planner | Executor | Checker |
|---|---|---|---|---|
| `README.md` | R/W/V | R | R | V |
| `AGENTS.md` | R/W/V | R | R | V |
| `docs/CURRENT.md` | R | W | R | W/V |
| `docs/design/DESIGN.md` | R/W/V | R | R if cited | V |
| `docs/design/SIGNOFF.md` | W/V | R | R | V |
| `docs/decisions/DECISION-NNN.md` | R/V | W | R if cited | V |
| `docs/decisions/INDEX.md` | R | R | R | W/V |
| `docs/guardrails/ARCHITECTURE_LOCKS.md` | R | R | R | W/V |
| `docs/implementation/ROADMAP.md` | V | W | R | V |
| `docs/implementation/SPRINT-N.md` | R/V | W | R | V |
| `docs/tasks/TASK-ID.md` | R | W | W only evidence | V |
| `docs/changes/CHANGE-NNN.md` | W/V if major | W | R if cited | V |
| `docs/testing/TEST_STRATEGY.md` | R/V | R | R | W/V |
| `docs/testing/ACCEPTANCE.md` | W/V | R | R | V |
| `docs/checks/CHECK-SPRINT-N.md` | R | R | R | W |
| `scripts/verify_sprint.py` | R | R | R | W/V |

## H. Change Classification

### Minor

Definition:

- Does not change public contract.
- Does not change persisted data.
- Does not change active ADR.
- Does not change signed scope.
- Adds or fixes expected internal behavior.

Authorizer: Planner.

Action:

- Create sprint/tasks.
- Does not require `CHANGE-NNN.md`, unless it touches signed text.

### Major

Definition:

- Adds new capability.
- Adds integration.
- Adds compatible persisted table/field.
- Changes testing strategy.
- Supersedes an ADR without breaking the public contract.

Authorizer: Human.

Action:

- Create `CHANGE-NNN.md`.
- Create or supersede ADR if applicable.
- Update roadmap and `CURRENT.md`.

### Breaking

Definition:

- Changes public API.
- Breaks data compatibility.
- Removes signed behavior.
- Changes auth, permissions, critical provider, or operational contract.
- Requires non-reversible migration.

Authorizer: Human with explicit sign-off.

Action:

- Approved `CHANGE-NNN.md`.
- New design version or signed annex.
- New ADR with `Supersedes`.
- Migration and rollback plan.
- Compatibility or expected-breakage tests.

## Mandatory Test Cases

### Case 1 - Phase 1 from scratch

Project: Inventory management REST API for a small store.

#### Documents at the end of Phase 1, before code

```text
inventory-api/
├── README.md
├── AGENTS.md
└── docs/
    ├── CURRENT.md
    ├── design/
    │   ├── DESIGN.md
    │   └── SIGNOFF.md
    ├── decisions/
    │   ├── DECISION-001.md
    │   ├── DECISION-002.md
    │   └── INDEX.md
    ├── guardrails/
    │   └── ARCHITECTURE_LOCKS.md
    ├── implementation/
    │   └── ROADMAP.md
    └── testing/
        ├── TEST_STRATEGY.md
        └── ACCEPTANCE.md
```

#### Human Signed Content

The human signs off on:

- `docs/design/DESIGN.md`
- `docs/design/SIGNOFF.md`
- `docs/testing/ACCEPTANCE.md`
- `AGENTS.md`

Example of initial decisions:

- `DECISION-001`: use REST JSON with `/v1` versioning.
- `DECISION-002`: use PostgreSQL as primary DB.
- `DECISION-003`: JWT authentication for private endpoints.

#### What the first agent will read and in what order

First Planner:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. `docs/design/DESIGN.md`
5. `docs/testing/ACCEPTANCE.md`
6. `docs/decisions/INDEX.md`
7. `docs/guardrails/ARCHITECTURE_LOCKS.md`

First Executor:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/implementation/SPRINT-001.md`
4. Their `docs/tasks/TASK-INV-001.md`
5. Cited ADRs: for example `DECISION-001`, `DECISION-002`

Does not read the entire design if the task already extracts the necessary context.

### Case 2 - Sprint 5: low stock alerts via email

Previous state: auth, products, and basic reports already implemented.

Feature: send low stock alerts via email.

#### What the Planner reads and why only that

Planner reads:

1. `README.md`: confirm map and commands.
2. `AGENTS.md`: role rules.
3. `docs/CURRENT.md`: current state and previous sprint.
4. `docs/decisions/INDEX.md`: detect active decisions.
5. `docs/guardrails/ARCHITECTURE_LOCKS.md`: view constraints.
6. `docs/design/DESIGN.md#products`: understand stock.
7. `docs/design/DESIGN.md#notifications` if it exists; if not, creates a major change.
8. `docs/testing/ACCEPTANCE.md`: global criteria.

Only those because the feature touches stock, email, and testing; it doesn't need to read full auth or full reports if their contracts are indexed.

#### What the Planner produces

Produces:

- `docs/changes/CHANGE-005.md`, `MAJOR` class, because it adds email integration.
- `docs/decisions/DECISION-004.md`, key `EMAIL_PROVIDER_SENDGRID`.
- Validated update to `docs/decisions/INDEX.md`.
- Validated update to `docs/guardrails/ARCHITECTURE_LOCKS.md`.
- `docs/implementation/SPRINT-005.md`.
- `docs/tasks/TASK-EMAIL-001.md`: email adapter with red test.
- `docs/tasks/TASK-STOCK-002.md`: low stock detection.
- `docs/tasks/TASK-ALERT-003.md`: low stock event -> email integration.
- `docs/tasks/TASK-CHECK-004.md`: no duplicate alert tests.

#### What the small model Executor reads

For `TASK-ALERT-003.md` it reads:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. `docs/implementation/SPRINT-005.md`
5. `docs/tasks/TASK-ALERT-003.md`
6. `docs/decisions/DECISION-004.md`
7. Allowed files:
   - `src/stock/**`
   - `src/email/**`
   - `tests/stock/**`
   - `tests/email/**`

Does not read auth, reports, or full sprints 1-4.

#### What happens if in sprint 9 they want to migrate SendGrid to AWS SES

`DECISION-004` is not edited.

It is created:

- `docs/changes/CHANGE-009.md`, `MAJOR` class if the internal interface is maintained; `BREAKING` if public contract/operational config changes.
- `docs/decisions/DECISION-012.md`, key `EMAIL_PROVIDER_AWS_SES`, with `Supersedes: DECISION-004`.
- `DECISION-004` becomes `SUPERSEDED`.
- `docs/decisions/INDEX.md` replaces the active key.
- `ARCHITECTURE_LOCKS.md` changes from "no AWS SES" to "use AWS SES via adapter".

Sprint 9 Checker fails if production code using SendGrid remains outside an explicit migration or if a task uses the old key.

#### How the Checker validates without reading the entire repo

Checker reads:

1. `SPRINT-005.md`
2. Sprint tasks
3. Diff of modified files
4. `DECISION-004.md`
5. `ARCHITECTURE_LOCKS.md`
6. Declared tests

Validates:

- There is a red test for low stock.
- There is a red test for sending email.
- Emails are not duplicated for the same condition.
- The email provider is behind the adapter.
- There is no SendGrid client outside `src/email/**`.
- Green commands are pasted.
- Touched paths are the allowed ones.

## Mandatory Self-Critique

1. The framework relies on local documentation discipline. It breaks if the team accepts merges without running `verify_sprint.py`; in that scenario, the locks exist but protect nothing.

2. The 4000 token limit per file forces summarization. It breaks in regulated or scientific domains where a decision requires extensive evidence; partial solution: split ADRs per decision and move long evidence to non-mandatory annexes.

3. Anti-drift validation does not understand deep code semantics. It breaks if an agent contradicts a decision with an indirect abstraction that doesn't match paths or patterns; mitigation: locks with precise paths, architecture tests, and human review for major changes.

## References

- Diátaxis: I take the separation by reader's need type; discard extensive documentation that forces the Executor to read tutorials.
- AGENTS.md: I take the operational contract at the repo root; discard long non-verifiable instructions.
- Karpathy LLM Wiki: I take the idea of small, explicit, agent-oriented context; discard relying on the model's implicit reasoning.
- Michael Nygard's ADRs: I take append-only decisions with consequences; discard ADRs as a passive historical file without locks or validation.
