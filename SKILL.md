---
name: ontocausal
version: 1.0.0
description: Typed knowledge graph + causal inference engine for structured agent memory and composable skills. Enhanced with causal reasoning, root cause analysis (5 Whys), and success attribution. Use when creating/querying entities (Person, Project, Task, Event, Document), linking related objects, enforcing constraints, planning multi-step actions as graph transformations, or when skills need to share state. Trigger on "remember", "what do I know about", "link X to Y", "show dependencies", "why did this fail", "what caused success", entity CRUD, or cross-skill data access.
---

# OntoCausal

A typed vocabulary + constraint system for representing knowledge as a verifiable graph, **enhanced with causal inference engine**.

## Core Concept

Everything is an **entity** with a **type**, **properties**, and **relations** to other entities. Every mutation is validated against type constraints before committing.

```
Entity: { id, type, properties, relations, created, updated }
Relation: { from_id, relation_type, to_id, properties }
```

## 🌟 Causal Inference Enhancement (New)

### Root Cause Analysis (5 Whys)

Automatically trace failure reasons with 5 Whys method:

```python
from ontocausal import CausalAnalyzer

analyzer = CausalAnalyzer()

# Log failed task
analyzer.add_event(
    id="task_001",
    type="Task",
    outcome="failure",
    context={"error": "Build failed", "logs": "..."}
)

# Get root cause chain (5 Whys)
chain = analyzer.root_cause_analysis("task_001")
for step in chain:
    print(f"Level {step.level}: {step.why}")
    print(f"Evidence: {step.evidence}")
```

### Success Attribution

Extract key factors from successful outcomes:

```python
# Log successful task
analyzer.add_event(
    id="task_002",
    type="Task",
    outcome="success",
    context={"actions": ["step1", "step2", "step3"]}
)

# Get success attribution
attribution = analyzer.success_attribution("task_002")
print(f"Key factors: {attribution.key_factors}")
print(f"Recommendations: {attribution.recommendations}")
```

### Decision Simulation

Simulate decisions and predict outcomes:

```python
simulation = analyzer.simulate_decision(
    scenario={"project": "claw-mesh", "resources": "limited"},
    decision={"action": "deploy_now", "risk": "high"}
)

print(f"Predicted outcome: {simulation.predicted_outcome}")
print(f"Confidence: {simulation.confidence}")
print(f"Risks: {simulation.risks}")
```

## When to Use

| Trigger | Action |
|---------|--------|
| "Remember that..." | Create/update entity |
| "What do I know about X?" | Query graph |
| "Link X to Y" | Create relation |
| "Show all tasks for project Z" | Graph traversal |
| "What depends on X?" | Dependency query |
| "Why did this fail?" | Root cause analysis (5 Whys) |
| "What caused this success?" | Success attribution |
| "What if we do X?" | Decision simulation |
| Planning multi-step work | Model as graph transformations |
| Skill needs shared state | Read/write ontology objects |

## Core Types

```yaml
# Agents & People
Person: { name, email?, phone?, notes? }
Organization: { name, type?, members[] }

# Work
Project: { name, status, goals[], owner? }
Task: { title, status, due?, priority?, assignee?, blockers[] }
Goal: { description, target_date?, metrics[] }

# Time & Place
Event: { title, start, end?, location?, attendees[], recurrence? }
Location: { name, address?, coordinates? }

# Information
Document: { title, path?, url?, summary? }
Message: { content, sender, recipients[], thread? }
Thread: { subject, participants[], messages[] }
Note: { content, tags[], refs[] }

# Resources
Account: { service, username, credential_ref? }
Device: { name, type, identifiers[] }
Credential: { service, secret_ref }  # Never store secrets directly

# Meta
Action: { type, target, timestamp, outcome? }
Policy: { scope, rule, enforcement }
```

## Storage

Default: `memory/ontocausal/graph.jsonl` and `memory/ontocausal/causal.jsonl`

```jsonl
{"op":"create","entity":{"id":"p_001","type":"Person","properties":{"name":"Alice"}}}
{"op":"create","entity":{"id":"proj_001","type":"Project","properties":{"name":"Website Redesign","status":"active"}}}
{"op":"relate","from":"proj_001","rel":"has_owner","to":"p_001"}
```

Query via scripts or direct file ops. For complex graphs, migrate to SQLite.

### Append-Only Rule

When working with existing ontology data or schema, **append/merge** changes instead of overwriting files. This preserves history and avoids clobbering prior definitions.

## Workflows

### Create Entity

```bash
python3 scripts/ontocausal.py create --type Person --props '{"name":"Alice","email":"alice@example.com"}'
```

### Query

```bash
python3 scripts/ontocausal.py query --type Task --where '{"status":"open"}'
python3 scripts/ontocausal.py get --id task_001
python3 scripts/ontocausal.py related --id proj_001 --rel has_task
```

### Link Entities

```bash
python3 scripts/ontocausal.py relate --from proj_001 --rel has_task --to task_001
```

### Validate

```bash
python3 scripts/ontocausal.py validate  # Check all constraints
```

### Causal Analysis (New)

```bash
# Root cause analysis for failed task
python3 scripts/ontocausal.py causal --analysis root_cause --id task_001

# Success attribution for successful task
python3 scripts/ontocausal.py causal --analysis success_attribution --id task_002

# Decision simulation
python3 scripts/ontocausal.py causal --analysis simulate --scenario '{"project":"test"}' --decision '{"action":"deploy"}'
```

## Constraints

Define in `memory/ontocausal/schema.yaml`:

```yaml
types:
  Task:
    required: [title, status]
    status_enum: [open, in_progress, blocked, done]
  
  Event:
    required: [title, start]
    validate: "end >= start if end exists"

  Credential:
    required: [service, secret_ref]
    forbidden_properties: [password, secret, token]  # Force indirection

relations:
  has_owner:
    from_types: [Project, Task]
    to_types: [Person]
    cardinality: many_to_one
  
  blocks:
    from_types: [Task]
    to_types: [Task]
    acyclic: true  # No circular dependencies
```

## Skill Contract

Skills that use ontocausal should declare:

```yaml
# In SKILL.md frontmatter or header
ontocausal:
  reads: [Task, Project, Person]
  writes: [Task, Action]
  uses_causal: true  # Enable causal inference
  preconditions:
    - "Task.assignee must exist"
  postconditions:
    - "Created Task has status=open"
```

## Planning as Graph Transformation

Model multi-step plans as a sequence of graph operations:

```
Plan: "Schedule team meeting and create follow-up tasks"

1. CREATE Event { title: "Team Sync", attendees: [p_001, p_002] }
2. RELATE Event -> has_project -> proj_001
3. CREATE Task { title: "Prepare agenda", assignee: p_001 }
4. RELATE Task -> for_event -> event_001
5. CREATE Task { title: "Send summary", assignee: p_001, blockers: [task_001] }
```

Each step is validated before execution. Rollback on constraint violation.

## Causal Integration Patterns

### With Causal Inference

Log ontology mutations as causal actions (built-in):

```python
# When creating/updating entities, also log to causal action log
action = {
    "action": "create_entity",
    "domain": "ontocausal", 
    "context": {"type": "Task", "project": "proj_001"},
    "outcome": "created"
}
```

### Cross-Skill Communication

```python
# Email skill creates commitment
commitment = ontocausal.create("Commitment", {
    "source_message": msg_id,
    "description": "Send report by Friday",
    "due": "2026-01-31"
})

# Task skill picks it up
tasks = ontocausal.query("Commitment", {"status": "pending"})
for c in tasks:
    ontocausal.create("Task", {
        "title": c.description,
        "due": c.due,
        "source": c.id
    })
```

## Quick Start

```bash
# Initialize ontocausal storage
mkdir -p memory/ontocausal
touch memory/ontocausal/graph.jsonl
touch memory/ontocausal/causal.jsonl

# Create schema (optional but recommended)
python3 scripts/ontocausal.py schema-append --data '{
  "types": {
    "Task": { "required": ["title", "status"] },
    "Project": { "required": ["name"] },
    "Person": { "required": ["name"] }
  }
}'

# Start using
python3 scripts/ontocausal.py create --type Person --props '{"name":"Alice"}'
python3 scripts/ontocausal.py list --type Person
```

## References

- `references/schema.md` — Full type definitions and constraint patterns
- `references/queries.md` — Query language and traversal examples
- `references/causal.md` — Causal inference engine documentation (new)

## Instruction Scope

Runtime instructions operate on local files (`memory/ontocausal/graph.jsonl`, `memory/ontocausal/causal.jsonl`, and `memory/ontocausal/schema.yaml`) and provide CLI usage for create/query/relate/validate/causal; this is within scope. The skill reads/writes workspace files and will create the `memory/ontocausal` directory when used. Validation includes property/enum/forbidden checks, relation type/cardinality validation, acyclicity for relations marked `acyclic: true`, and Event `end >= start` checks; other higher-level constraints may still be documentation-only unless implemented in code.
