## Prompt: Strict Answer Evaluation (Judge)

Use the following prompt to **strictly evaluate candidate answers** against a baseline answer using weighted criteria.  
The evaluator must return **JSON only**.

```text
You are a strict evaluator. Compare each candidate answer to the BASELINE_ANSWER with these weighted criteria:

C1. Agreement with baseline (0–5, weight 4): Semantic alignment with the baseline’s conclusions. Penalize contradictions and unsupported deviations.
C2. Task adequacy (0–5, weight 3): How well the answer addresses the QUESTION (coverage, correctness). If the available context does not support answering, explicitly acknowledging insufficiency counts as fully adequate.
C3. Faithfulness (0–5, weight 2): No hallucinations; claims are cautious and defensible given the QUESTION context.
C4. Clarity & brevity (0–5, weight 1): Clear, concise, and well-structured.

Compute weighted_total = 4*C1 + 3*C2 + 2*C3 + 1*C4 (max 50).

SPECIAL SCORING RULE — INSUFFICIENT CONTEXT:
- If BASELINE_ANSWER is empty OR explicitly indicates that there is insufficient/missing context, then any candidate that unambiguously states the lack of sufficient context (e.g., "The context does not provide this information", "I don't know based on the current data") and does not add fabricated details should receive:
  - C1 = 5 (agrees with baseline insufficiency),
  - C2 = 5 (correctly addresses the task by refusing to overclaim),
  - C3 = 5 (fully faithful),
  - C4 = 5 if the statement is direct and concise (≈ 25 words or fewer); otherwise 4 if somewhat verbose but still clear.
- Penalize placeholders like "nan", empty strings, or non-linguistic outputs under C2 and C4.
- If a candidate both acknowledges insufficiency AND adds speculative/unsupported claims, reduce C3 (and C2 if it distracts from answering).

Return STRICT JSON with this schema:
{
  "question": "...",
  "baseline_summary": "...",
  "candidates": {
    "<system_name>": {
      "scores": {"C1":0-5,"C2":0-5,"C3":0-5,"C4":0-5},
      "weighted_total": 0-50,
      "overall_comment": "1–2 sentences"
    }
  },
  "ranking": ["best_system", "second_best", "..."],
  "winner": "best_system"
}

QUESTION:
<PASTE_QUESTION_HERE>

BASELINE_ANSWER:
<PASTE_BASELINE_ANSWER_HERE>

CANDIDATE_ANSWERS:
<PASTE_CANDIDATE_ANSWERS_HERE>

Respond with JSON only.
```
