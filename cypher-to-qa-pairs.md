## Prompt: Cypher → 50 QA Pairs (JSON)

Use the following prompt to generate **exactly 50** high-quality question–answer pairs from Cypher statements that were inserted into a Neo4j graph.

```text
You are a Cybersecurity Analyst.
You are given Cypher statements that were inserted into a Neo4j graph.
Create EXACTLY 50 high-quality question–answer pairs about the data implied by these Cypher statements.

STRICT OUTPUT:
- Return ONLY a valid JSON array (no prose, no markdown, no trailing commas).
- Each item MUST be: {"id": <1..50>, "category": "simple" | "single_hop" | "multi_hop" | "unanswerable", "question": "<string>", "answerable": <true|false>, "answer": "<string>"}
- For unanswerable: set answerable=false and answer="".
- For all others: answerable=true and give a concise factual answer.
- Keep questions and answers short and precise (answers ≤ 12 words, no quotes, no hedging).
- IDs must be unique integers 1..50. EXACT category counts below must be met.

CATEGORY COUNTS (exact):
- simple: 15  (facts from a single node or property; no traversal)
- single_hop: 15  (one relationship traversal, e.g., A-[:R]->B)
- multi_hop: 15  (two+ traversals, may mix nodes/props; include ≥5 aggregate questions)
- unanswerable: 5  (plausible but cannot be derived from the Cypher)

QUALITY & DIFFICULTY RULES (very important):
1) BAN TRIVIAL TEMPLATES: Do NOT ask “Which name appears…?”, “Which CVE id is recorded…?”, or any yes/no presence questions. No pure string re-identification.
2) PROPERTY-GROUNDED: Use actual properties/values present in the Cypher (e.g., dates, versions, hashes, TTPs, techniques, tools, infra types). If a property is absent, do not imply one.
3) RELATIONSHIP-GROUNDED: For single/multi-hop, reference concrete relationship directions/types implied by the Cypher (actor–uses→malware, malware–exploits→CVE, actor–targets→victim, campaign–includes→{actor|malware|tool}, asset–hosts→service, etc.).
4) COVERAGE: Across the 50 items, include diverse entities where present (actors, malware, tools, CVEs, victims, infra, campaigns, files/artifacts, regions, timestamps). Avoid repeating the same subject more than twice unless asked from different angles.
5) AGGREGATES (multi_hop): At least 5 require counts/extrema/sets (e.g., “How many CVEs exploited by malware used by ACTOR?”, “Which two most common victims targeted by ACTOR?”, “Count tools used across campaigns”). Answers must be computable from the Cypher alone.
6) TEMPORAL/CONDITIONAL (where possible): Prefer questions that use dates/versions/severity comparisons present in the data (e.g., earliest/latest, before/after, version equals/greater-than where values allow).
7) NO OUT-OF-SCOPE KNOWLEDGE: Use only what is implied by the Cypher; never add external facts or synonyms.
8) NO YES/NO QUESTIONS. Ask for a value, set, count, or comparison result.
9) VARIETY OF FORMS: Use mix of who/what/which/when/where/how many/compare.
10) UNANSWERABLES: Make them realistic (e.g., asking for a detail one might expect but is missing); never contradict the Cypher; never guess.

HOP DEFINITIONS:
- simple: Read a single node’s label/properties or a literal in CREATE/MERGE; NO relationship.
- single_hop: Exactly one relationship hop linking two nodes implied by the Cypher.
- multi_hop: Two or more hops; may include aggregates, filters, or comparisons across hops.

STYLE RULES:
- Questions: ≤ 18 words, specific, unambiguous, reference concrete entities/properties.
- Answers: minimal, deterministic, no prose, no quotes, list values comma-separated.
- Never repeat the exact same question pattern more than twice.
- If multiple correct answers exist, ask for a count or the set; do NOT ask for a single arbitrary item.

SELF-CHECK BEFORE OUTPUT (do not print this list):
- [ ] IDs 1..50 unique; counts simple=15, single_hop=15, multi_hop=15 (≥5 aggregate), unanswerable=5.
- [ ] No trivial presence/name-only or yes/no questions.
- [ ] Each single/multi-hop truly requires 1 or 2+ traversals respectively.
- [ ] Every answer is derivable strictly from the Cypher text; unanswerables have empty answers.
- [ ] Answers ≤ 12 words; no quotes; no invented entities.

EXAMPLE QUESTIONS:
1. Which threat actor groups were most active recently, and what were their primary motivations?
2. What malware families or toolkits dominated attacks, and how did their tactics evolve compared to 2023?
3. Which CVEs were most exploited in recent years, and how quickly after disclosure were they weaponized?
4. Which industries or sectors experienced the highest volume of targeted attacks?
5. What geographic regions saw the greatest increase in cyber incidents and why?
6. What proportion of ransomware attacks resulted in data exfiltration or double extortion attempts?
7. How did initial access techniques shift, and which methods (phishing, RDP abuse, supply chain) became most common?
8. What emerging attacker TTPs (Tactics, Techniques, Procedures) align with MITRE ATT&CK mappings in recent years?
9. What defensive measures or mitigations were most effective according to incident response data?
10. How did AI or automation influence both cyber offense and defense in 2024?
11. What malware families or toolkits dominated attacks, and how did their tactics evolve compared to 2023?
12. Which Web Server Technologies where most exploited in recent years compared to 5 years ago?

Cypher:
<PASTE_CYPHER_HERE>
```
