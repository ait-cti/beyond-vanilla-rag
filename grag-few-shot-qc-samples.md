## Prompt: Few-Shot QA Generation for Neo4j Graphs

Use the following prompt to generate **strict few-shot (question, query) examples** for training or evaluating an LLM-based QA system over a Neo4j graph.

```text
You are generating few-shot (question, query) examples for training a QA system over a Neo4j graph.
Produce EXACTLY <N> examples.

STRICT OUTPUT FORMAT — return ONLY a valid JSON array (no markdown, no commentary):
[
  {
    "question": "<natural-language question answerable by the graph>",
    "query": "<READ-ONLY Cypher that answers the question; prefer MATCH/OPTIONAL MATCH + RETURN>"
  },
  ... (total of exactly <N> items)
]

RULES FOR QUERIES:
- READ-ONLY Cypher only — allowed clauses:
  MATCH, OPTIONAL MATCH, WHERE, WITH, RETURN, DISTINCT, ORDER BY, LIMIT, SKIP, UNWIND, aggregation.
- FORBIDDEN anywhere (case-insensitive):
  CREATE, MERGE, SET, DELETE, REMOVE, DETACH, LOAD CSV, CALL, FOREACH, PERIODIC, APOC,
  and any procedure/function calls that write.
- Use only labels, relationships, and properties that exist in the provided inserts
  or in the authoritative schema below (if provided).
- Avoid hardcoded values not present in the inserts.
- Queries must be directly answerable, executable as-is, and not rely on parameters.
- Prefer short, clear variable names and minimal RETURN columns with aliases.

DIVERSITY REQUIREMENTS:
- Include a balanced mix of:
  * Simple lookups (single label)
  * One-hop relationships
  * Multi-hop paths
  * 2–3 aggregate queries using COUNT, COLLECT, size(), etc.
- Avoid duplicates or trivial rephrasings.

SELF-CHECK BEFORE OUTPUT (apply silently):
1. Exactly <N> items.
2. Each item contains valid "question" and "query" keys.
3. All queries use only allowed clauses.
4. JSON is syntactically valid and properly quoted.
5. Each question is answerable from the provided inserts/schema.

AUTHORITATIVE GRAPH SCHEMA (use this, do not invent anything else):
<PASTE_SCHEMA_HERE>

CYPHER INSERTS (reference for labels, relationships, and properties):
<PASTE_CYPHER_INSERTS_HERE>

Return ONLY the JSON array — no markdown, no explanation.
```
