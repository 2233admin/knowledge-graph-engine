# Knowledge Graph Engine (KGE)

**Typed knowledge graph + causal inference engine for structured agent memory and composable skills.**

Enhanced with causal reasoning, root cause analysis (5 Whys), and success attribution.

## Features

- **Typed Knowledge Graph**: Entities, relations, properties with validation
- **Causal Inference Engine**: Root cause analysis (5 Whys), success attribution, decision simulation
- **Composable Skills**: Cross-skill state sharing via knowledge graph
- **Validation**: Type constraints, relation cardinality, acyclic checks

## Quick Start

```bash
# Initialize KGE storage
mkdir -p memory/knowledge-graph-engine
touch memory/knowledge-graph-engine/graph.jsonl
touch memory/knowledge-graph-engine/causal.jsonl

# Start using
python3 scripts/ontocausal.py create --type Person --props '{"name":"Alice"}'
```

## GitHub Repository

https://github.com/2233admin/knowledge-graph-engine

## License

MIT
