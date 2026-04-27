---
name: stpa-analyst
description: "Perform System-Theoretic Process Analysis (STPA) on project code or architecture. Use when: analysing a system for safety hazards, identifying unsafe control actions, deriving safety constraints from code, producing an initial STPA scaffolding for a project, generating requirements and quality goals from hazard analysis. Keywords: STPA, STAMP, hazard analysis, safety, loss, unsafe control action, UCA, loss scenario, control structure, constraint, risk analysis, RAFIA."
---

# STPA Analyst

Perform a System-Theoretic Process Analysis (STPA) on a software project or subsystem, producing a structured hazard analysis that can drive specification requirements, TSF quality statements, and test strategies.

STPA is a top-down hazard analysis method based on systems theory (STAMP model, developed by Nancy Leveson at MIT). Unlike failure-focused methods (FMEA, FTA), STPA identifies hazards that arise from unsafe *interactions* between components — including cases where every component is working correctly but the system as a whole enters a hazardous state.

## When to Use

- Analysing a new or existing software project for safety or reliability hazards
- Generating an initial STPA scaffolding that the user can refine and extend
- Deriving safety/reliability requirements from hazard analysis (feeding specification-writer)
- Identifying quality claims that need evidence (feeding tsf-author)
- Designing test scenarios that validate constraints (feeding developer)
- Reviewing whether existing requirements adequately cover identified hazards

## Handoff

### Inputs

- Project source code, architecture descriptions, or design documents
- User-defined system scope and purpose (or inferred from the code)
- Stakeholder-identified values at risk (what must not be lost)

### Outputs

- **STPA analysis document** (`docs/stpa-analysis.md`) — the full structured analysis
- **Derived constraints** — feed into **specification-writer** as candidate `req~` items
- **Quality claims** — feed into **tsf-author** as candidate TSF statements (safety-related)
- **Test scenarios** — feed into **developer** as fault injection and validation test cases

### Related Skills

This skill provides analytical input to the tsfagent triad:

1. **stpa-analyst** (this skill) analyses the system and derives constraints
2. **specification-writer** formalises constraints as `req~` items
3. **tsf-author** creates TSF quality statements for safety/reliability goals
4. **developer** implements constraints as code, tests, and CI evidence

## STPA Method Overview

STPA proceeds through four sequential steps. Each step's output feeds the next.

```
Step 1: Define Purpose          Step 2: Model Control Structure
   Losses                          Controllers
   Hazards                         Control Actions
   System-level Constraints        Feedback
                                   Controlled Processes
        │                               │
        ▼                               ▼
Step 3: Identify UCAs            Step 4: Identify Loss Scenarios
   Unsafe Control Actions           Why each UCA might occur
   Controller Constraints           Causal factors
                                    Additional constraints
        │
        ▼
   Derived Constraints → req~ items, TSF statements, test scenarios
```

## STPA Concepts

### Losses

Losses are undesirable outcomes involving something of value to stakeholders. They are defined at the system level, not the component level.

Examples:
- **L-1**: Loss of life or injury to people
- **L-2**: Loss or corruption of critical data
- **L-3**: Loss of system availability beyond acceptable downtime
- **L-4**: Violation of regulatory or compliance requirements

Losses are identified by considering: What could go wrong that stakeholders care about? What are the worst outcomes of system misbehaviour?

### Hazards

A hazard is a system state or set of conditions that, together with worst-case environmental conditions, will lead to a loss. Hazards are system-level — they describe the system's relationship with its environment, not internal component failures.

Format: **H-N**: `<System>` `<unsafe condition>` `[<link to losses>]`

Examples:
- **H-1**: System provides incorrect output to downstream consumer [L-2]
- **H-2**: System fails to respond within required time bound [L-3]
- **H-3**: System permits unauthorised access to protected resources [L-2, L-4]

### System-Level Constraints

For each hazard, define a constraint that, if satisfied, prevents or mitigates it.

Format: **SC-N**: `<System>` *must/must not* `<constraint>` `[<link to hazards>]`

Examples:
- **SC-1**: System must provide correct output or clearly indicate an error condition [H-1]
- **SC-2**: System must respond within the specified time bound under all load conditions [H-2]
- **SC-3**: System must authenticate all access requests and reject unauthorised ones [H-3]

### Control Structure

A control structure diagram models the system as a hierarchy of controllers, control actions, feedback, and controlled processes. This is the central analytical model.

```
┌──────────────┐
│  Controller   │
│  (has process │
│   model)      │
└──────┬───────┘
       │ Control Actions (commands, signals, configs)
       ▼
┌──────────────┐
│  Controlled   │
│  Process      │
└──────┬───────┘
       │ Feedback (status, data, events)
       ▼
  Back to Controller
```

Key elements:
- **Controller**: A component that makes decisions and issues control actions (could be software, hardware, human, or organisational process)
- **Control action**: A command, signal, or configuration that a controller sends to influence a controlled process
- **Controlled process**: The component or subsystem being controlled
- **Feedback**: Information flowing back from the controlled process to the controller
- **Process model**: The controller's internal representation of the controlled process state (beliefs about current state, which may be incorrect)

### Unsafe Control Actions (UCAs)

For each control action in the control structure, systematically identify how it could be unsafe using four guide words:

| Guide Word | Question |
|------------|----------|
| **Not providing** | What hazard occurs if the control action is not provided when needed? |
| **Providing** | What hazard occurs if the control action is provided when it should not be? |
| **Too early / too late / wrong order** | What hazard occurs if the control action is provided at the wrong time? |
| **Stopped too soon / applied too long** | What hazard occurs if the control action is provided for the wrong duration? |

Format: **UCA-N**: `<Controller>` `<provides/does not provide>` `<control action>` `<context>`, leading to `<hazard>` `[<link to hazards>]`

Example:
- **UCA-1**: Scheduler does not provide timeout signal when a task exceeds its deadline, leading to system failing to respond within time bound [H-2]
- **UCA-2**: Auth module provides access grant for an expired or revoked token, leading to unauthorised access [H-3]

### Controller Constraints

For each UCA, derive a constraint on the controller that would prevent it:

- **CC-1**: Scheduler must provide a timeout signal when any task exceeds its configured deadline [UCA-1]
- **CC-2**: Auth module must reject access requests with expired or revoked tokens [UCA-2]

### Loss Scenarios

For each UCA, identify *why* it might occur. Loss scenarios explain the causal chain. Categories:

1. **Controller failures**
   - Incorrect algorithm or logic
   - Unsafe control action generated by controller
   - Incorrect process model (controller has wrong beliefs about system state)

2. **Control path failures**
   - Control action not received by controlled process
   - Control action delayed, corrupted, or reordered
   - Conflicting control actions from multiple controllers

3. **Controlled process failures**
   - Process does not execute control action correctly
   - Process state is different from what controller assumes

4. **Feedback failures**
   - Feedback not provided or delayed
   - Feedback incorrect or incomplete
   - Controller misinterprets correct feedback

Format: **LS-N**: `<Description of causal scenario>` `[<link to UCAs>]`

Example:
- **LS-1**: Scheduler's process model does not track task start time due to missing instrumentation, so timeout is never triggered [UCA-1]
- **LS-2**: Auth module's token validation uses a cached token state that is not invalidated on revocation [UCA-2]

## Procedure

### Analysing a Project

1. **Understand the system scope.**
   - Read the project's README, architecture docs, and main entry points
   - Identify what the system does, who its stakeholders are, and what its operational environment is
   - If the scope is unclear, ask the user to confirm: "What is the system's purpose? What are its boundaries?"

2. **Step 1: Define losses, hazards, and system-level constraints.**
   - Identify losses by asking: What could go wrong that stakeholders care about? Consider safety, data integrity, availability, compliance, financial impact, and reputation.
   - For each loss, identify hazards: What system states or conditions could lead to this loss?
   - For each hazard, define a system-level constraint that prevents or mitigates it
   - Present these to the user for review before proceeding — the entire analysis depends on correct loss/hazard identification

3. **Step 2: Model the control structure.**
   - Identify the controllers in the system (software components that make decisions)
   - Identify the controlled processes (components or external systems being controlled)
   - Map the control actions (commands, API calls, signals, configuration) between them
   - Map the feedback paths (return values, events, status, monitoring data)
   - Identify each controller's process model (what state does it track? what assumptions does it make?)
   - Draw the control structure using text diagrams
   - For code analysis: controllers are typically modules/classes that orchestrate behaviour; controlled processes are modules/classes that execute operations; control actions are function calls, API requests, messages, or configuration writes; feedback is return values, callbacks, events, or monitored state

4. **Step 3: Identify unsafe control actions.**
   - For each control action in the control structure, apply the four guide words systematically
   - Consider the context: under what conditions is this control action issued? What state is the system in?
   - Link each UCA to one or more hazards
   - Derive a controller constraint for each UCA
   - This is the most labour-intensive step — be thorough but prioritise control actions on critical paths

5. **Step 4: Identify loss scenarios.**
   - For each UCA (or at least the highest-priority ones), identify plausible causal scenarios
   - Consider all four categories: controller failures, control path failures, controlled process failures, feedback failures
   - For software systems, pay particular attention to:
     - Stale or incorrect process models (cached state, race conditions)
     - Missing or delayed feedback (async operations, unreliable networks)
     - Conflicting control actions (concurrent controllers, distributed systems)
     - Incorrect assumptions about controlled process behaviour (API contracts, error handling)

6. **Derive outputs.**
   - Compile all controller constraints and any additional constraints from loss scenarios
   - Map constraints to candidate `req~` items for the specification-writer
   - Map hazards and constraints to candidate TSF quality statements
   - Map loss scenarios to candidate test scenarios (including fault injection tests)

### Writing the Analysis Document

Produce the analysis as `docs/stpa-analysis.md` with this structure:

```markdown
# STPA Analysis: <System Name>

## Scope

<System description, purpose, boundaries, stakeholders, operational environment.>

## Losses

| ID | Loss | Stakeholders |
|----|------|-------------|
| L-1 | ... | ... |

## Hazards

| ID | Hazard | Losses |
|----|--------|--------|
| H-1 | ... | L-1 |

## System-Level Constraints

| ID | Constraint | Hazards |
|----|-----------|---------|
| SC-1 | ... | H-1 |

## Control Structure

<Text diagram(s) showing controllers, control actions, feedback, controlled processes.>

### Controllers

| Controller | Responsibilities | Process Model |
|-----------|-----------------|---------------|
| ... | ... | ... |

### Control Actions and Feedback

| From | To | Control Action | Feedback |
|------|----|---------------|----------|
| ... | ... | ... | ... |

## Unsafe Control Actions

| ID | Controller | Control Action | Guide Word | Context | Hazard |
|----|-----------|---------------|------------|---------|--------|
| UCA-1 | ... | ... | Not providing | ... | H-1 |

## Controller Constraints

| ID | Constraint | UCAs |
|----|-----------|------|
| CC-1 | ... | UCA-1 |

## Loss Scenarios

| ID | Scenario | UCAs | Category |
|----|----------|------|----------|
| LS-1 | ... | UCA-1 | Process model |

## Derived Outputs

### Candidate Requirements

| Constraint | Suggested req~ ID | Suggested Needs |
|-----------|-------------------|-----------------|
| CC-1 | req~scheduler-timeout~1 | impl, utest, itest |

### Candidate TSF Statements

| Hazard/Constraint | Suggested TSF ID | Suggested publish.group |
|------------------|------------------|------------------------|
| H-1, SC-1 | PROJECT-OUTPUT_CORRECTNESS | Safety |

### Candidate Test Scenarios

| Loss Scenario | Test Type | Description |
|--------------|-----------|-------------|
| LS-1 | itest (fault injection) | Verify scheduler triggers timeout when task monitoring is disabled |
```

## Scope Guidance for Software Projects

When analysing software (as opposed to a physical system), adapt STPA concepts:

| STPA Concept | Software Interpretation |
|-------------|----------------------|
| **Loss** | Data loss/corruption, service outage, security breach, compliance violation, incorrect output affecting downstream systems |
| **Hazard** | System state that leads to a loss: inconsistent data, unbounded resource consumption, unauthenticated access, silent failure |
| **Controller** | Module/class/service that orchestrates behaviour, makes decisions, manages state |
| **Control action** | Function call, API request, message/event, configuration write, database transaction |
| **Feedback** | Return value, callback, event, health check, log, monitoring metric |
| **Process model** | Internal state, cache, configuration, assumptions encoded in code logic |
| **Unsafe control action** | A call/message that is missing, mistimed, mis-targeted, or applied for wrong duration |
| **Loss scenario** | Race condition, stale cache, dropped message, unhandled error, incorrect retry logic, API contract violation |

## Quality Guidance

### Appropriate Depth

This skill produces an **initial scaffolding** — a structured starting point that guides the user into a deeper analysis. Communicate this clearly:

- The analysis is a first pass based on code structure and documented behaviour
- Loss and hazard identification requires domain expertise the analyst may lack — always present for user review
- The UCA table will not be exhaustive — prioritise control actions on critical paths and security boundaries
- Loss scenarios are hypotheses — they need validation through code review and testing

### Iteration

STPA is inherently iterative (as emphasised in the RAFIA methodology). The initial analysis should be revisited when:

- New components or interactions are added
- Test results reveal unexpected behaviours
- Loss scenarios are confirmed or refuted
- The system's scope or environment changes

### Connecting to the tsfagent Pipeline

The derived outputs table is the handoff point:

1. **Candidate requirements** → Give these to specification-writer for formalisation as `req~` items. The constraints should map naturally to falsifiable requirements with `Needs: impl, utest` or `Needs: impl, utest, itest` for fault injection scenarios.

2. **Candidate TSF statements** → Give these to tsf-author. Safety-related constraints map well to TSF statements under a "Safety" or "Reliability" publish group. The loss scenarios inform the `_CONTEXT` evidence strategy.

3. **Candidate test scenarios** → Give these to developer. Loss scenarios describe the *what* and *why* of test cases. Fault injection tests (simulating the causal factors in loss scenarios) are particularly valuable for validating constraints.

## Example: Analysing a Task Scheduler

### Scope

A background task scheduler that accepts jobs via an API, queues them, and dispatches them to worker processes. Workers report completion or failure back to the scheduler.

### Losses

| ID | Loss |
|----|------|
| L-1 | Jobs are lost or not executed |
| L-2 | System resources exhausted (workers, memory, disk) |
| L-3 | Sensitive job data exposed to unauthorised parties |

### Hazards

| ID | Hazard | Losses |
|----|--------|--------|
| H-1 | Job accepted but never dispatched to a worker | L-1 |
| H-2 | Failed job not retried or reported | L-1 |
| H-3 | Workers allocated without bound | L-2 |
| H-4 | Job payload accessible without authentication | L-3 |

### Control Structure (excerpt)

```
┌─────────────┐
│   API Layer  │
│ (validates   │
│  & enqueues) │
└──────┬───────┘
       │ enqueue(job)
       ▼
┌─────────────┐     dispatch(job)     ┌─────────────┐
│  Scheduler   │ ──────────────────► │   Worker     │
│ (tracks jobs,│                      │  Pool        │
│  workers)    │ ◄────────────────── │              │
└─────────────┘   result(job_id,      └─────────────┘
                   status)
```

### Unsafe Control Actions (excerpt)

| ID | Controller | Control Action | Guide Word | Context | Hazard |
|----|-----------|---------------|------------|---------|--------|
| UCA-1 | Scheduler | dispatch(job) | Not providing | Job is queued and workers are available | H-1 |
| UCA-2 | Scheduler | dispatch(job) | Providing | No workers available, causes unbounded allocation | H-3 |
| UCA-3 | Worker Pool | result(status) | Not providing | Job completes or fails but no result sent back | H-2 |
| UCA-4 | API Layer | enqueue(job) | Providing | Request is unauthenticated | H-4 |

### Derived Outputs (excerpt)

| Constraint | Suggested req~ ID | Suggested Needs |
|-----------|-------------------|-----------------|
| Scheduler must dispatch all queued jobs within bounded time | req~job-dispatch-timeout~1 | impl, utest, itest |
| Scheduler must not allocate workers beyond configured pool limit | req~worker-pool-limit~1 | impl, utest |
| Worker must report result for every dispatched job | req~job-result-reporting~1 | impl, utest, itest |
| API must authenticate all job submission requests | req~job-api-auth~1 | impl, utest |
