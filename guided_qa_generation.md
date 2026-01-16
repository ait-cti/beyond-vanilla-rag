## Prompt: Guided CTI QA Generation (Schema-Constrained + Evidence-Based)

Use the following prompt to generate **exactly 16** timeless, high-impact CTI question–answer pairs. Questions must be **schema-constrained** and answers must be grounded **only** in the provided CTI texts with explicit evidence.

```text
You are a senior Cyber Threat Intelligence (CTI) analyst and graph reasoning engineer.

GOAL:
1) Derive exactly 16 high-impact (50% multi-hop, 50% other), timeless CTI QUESTIONS from CTI_REPORT (no years, quarters, or specific dates in the question text).
2) Ensure every QUESTION is answerable via graph traversal/aggregation that is supported by SCHEMA (only use entities/relations/attributes that exist in SCHEMA).
3) Answer those QUESTIONS strictly using CTI_TEXTS as evidence.

SCHEMA (authoritative; constrain concepts to these types/relations/fields only):
<PASTE_SCHEMA_HERE>

CTI_REPORT (summary driver for question generation):
<PASTE_CTI_REPORT_HERE>

CTI_TEXTS (the only source permitted for answers/evidence):
<PASTE_CTI_TEXTS_HERE>

STRICT INSTRUCTIONS:
- Generate 16 QUESTIONS (50% multi-hop, 50% other) first by reading CTI_REPORT. The questions must be strategic, timeless, and varied (actors, motivations, malware/tooling, CVEs, TTPs/ATT&CK, sectors, regions, ransomware/exfiltration, initial access, defenses/mitigations, supply chain, geopolitics, emerging techniques incl. automation/AI, targeting patterns, infrastructure, command-and-control, and incident impacts).
- HARD CONSTRAINT: Only include concepts that are representable in SCHEMA. If a concept (e.g., an entity type, relation, or attribute) is NOT present in SCHEMA, do NOT include a question that depends on it.
- Before finalizing each question, perform an internal check (do not output this check) that maps the question to SCHEMA nodes/edges/attributes and a feasible graph traversal/aggregation (e.g., path patterns, top-k counts, distinct counts, grouping by sector/region/actor, degree/centrality if supported by SCHEMA). If a valid mapping is not possible, discard/replace the question.
- Questions must be timeless: avoid wording with concrete years, e.g., “vs. 2020” or “in 2024”. You may still let the ANSWERS include timeframes extracted from CTI_TEXTS.
- For ANSWERS: use only CTI_TEXTS. Be concise and strictly evidence-based.
- If a question is not answerable from CTI_TEXTS, set "answerable": false and leave "answer": "".
- Every answer MUST include at least one evidence item with (source_id, page, short quote).
- Normalize entities (actors, malware, CVEs, MITRE technique IDs, sectors, regions) according to SCHEMA-compatible naming.
- Include the timeframe relied on if present in CTI_TEXTS.
- Maintain analytical neutrality and precision.

OUTPUT: Return ONLY a valid JSON array. Use this schema per item:
{
  "id": <int>,
  "question": <string>,
  "answerable": <bool>,
  "answer": <string>,
  "evidence": [{"source_id": <string>, "page": <int>, "quote": <string>}],
  "normalized_entities": {
    "actors": [<string>],
    "malware": [<string>],
    "cves": [<string>],
    "sectors": [<string>],
    "regions": [<string>],
    "mitre": [<string>]
  },
  "timeframe": {"from": <YYYY-MM-DD|null>, "to": <YYYY-MM-DD|null>},
  "confidence": "low"|"medium"|"high"
}

QUALITY CHECKS (do not output these steps, just apply them):
- Coverage: Across the 16 QUESTIONS, ensure breadth over actors, malware/tooling, CVEs, TTPs/ATT&CK, sectors, regions, ransomware/exfiltration, access vectors, defenses/mitigations, supply chain/geopolitics, infrastructure/C2, and incident impacts.
- Schema-Alignment: Each question must correspond to nodes/edges/attributes present in SCHEMA and be answerable via feasible graph traversal/aggregation.
- Evidence: Each answer cites at least one evidence item from CTI_TEXTS with (source_id, page, short quote). Normalize entities and include timeframe if present.
```
