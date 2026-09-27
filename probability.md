# Distribution Protocol (internal evaluation, minimal external)

Goal: emit a **checkable belief distribution**, not a performed number. This skill enforces format; **true calibration is external** (holdout sets, temperature scaling, post-processing). Never fabricate `calibrated` values.

## 1. Close the question first

No open-ended free-form. Reduce to:

- **One proposition** (Noul): condition + scope + observable success criterion; or
- **Fixed options** (Choice/Score): options come from the user/context; never invent labels.

If ambiguous, use the most natural reading and note it in half a line of `basis`. Ask for clarification only if the two readings' distributions differ by more than 0.4, or use `insufficient`.

Bad: "What should we do?"  
Good: "Which model fits this task? A glm B ds C kimi D qwen"

## 2. Focus input

Judgments use **decision-relevant state only**: task text, constraints, explicit candidates, cited facts.  
Exclude: chatty history, unreferenced digressions, long unrelated documents.  
Cheap input ≠ dump everything—irrelevant context degrades judgment.

## 3. Internal evaluation (never exported)

Short, optional, private:

1. **Base rate**: reference-class prior; if none, start at 0.5 and mark "no base rate" in `basis`.
2. **Main evidence**: at most 3 items, tagged ↑/↓ (strong ↑↑ / mid ↑ / weak ↑weak). Consider counterevidence.
3. **Normalize** across the candidate space so **sum = 1**.
4. Near ties or more than 5 options: extra compare or two-stage (coarse filter → refine). **No mandatory O(n²)**.

Never write "I compared A/B…" into the user-visible answer.

## 4. Math constraints (mandatory outward)

1. **Choice**: every allowed option has a probability (including UNKNOWN when enabled). **Noul**: `p` implies the two-way pair. **Score**: point by default; full-scale optional.
2. Probabilities ∈ [0, 1].
3. **sum = 1** (±0.01) within one choice set (Noul: `p + (1-p) = 1`).
4. No verbal confidence words instead of numbers.
5. No winner without a full distribution (Choice/Noul).
6. Do not finalize before covering the options.
7. Single-event intervals (e.g. `0.55–0.75`) are allowed **outside** a full choice set and do not feed sum=1.

Program-side asserts (caller owns these):

```python
assert abs(sum(p.values()) - 1.0) < 0.01
assert all(0.0 <= v <= 1.0 for v in p.values())
assert choice in options or choice == "UNKNOWN"
```

## 5. Three semantics (do not mix)

| Level | Field | Meaning | Produced by |
|---|---|---|---|
| L1 | `confidence` / `c` | Self-rated correctness of this answer | model |
| L2 | `probabilities` / `p` | Belief over candidates | model |
| L3 | `calibrated` | Post-processed probability | **external program** |

Never narrate L1/L2 as "true accuracy." Verbalized numbers are prompt-sensitive and usually overconfident.  
L3 is a field contract only; the skill does not invent calibration.

## 6. UNKNOWN / INSUFFICIENT_INFORMATION

Allowed and encouraged:

- Options don't fit, key facts missing, broken premise → `status: insufficient_information` or `choice: UNKNOWN`.
- `confidence` is usually low.
- `basis` states what is missing.
- Suggest next step: what to supply / escalate / human check (**suggest only**).

Never hard-pick on thin information. Never answer judgment questions with "cannot estimate" alone—give a distribution or `insufficient`.

## 7. Precision and extremes

- Point estimates two decimals; thin evidence → interval (e.g. `P=0.55–0.75`) and lower confidence.
- Below 0.05 or above 0.95 only for logical necessity or overwhelming multi-source evidence.
- No fake precision (`0.8473` is not better than `0.85`).
- If every answer lands on 0.8/0.9: you skipped the base rate—return to §3.

## 8. When a distribution is required

Predictions, judgments, classification, scoring, feasibility, risk, or whenever the user asks.

Confidence alone is OK for: definitions, identities, facts retrievable from context, certain short answers.

Facts: unknown + verify path is OK. Judgments: distribution or `insufficient` is mandatory.

## 9. Batching

Each question runs §1–7 independently and gets its own field group. No blended "overall" verdict. Declare condition dependencies explicitly.

## 10. Minimal example

Question: "Which model should handle this coding task?" Options glm/ds/kimi/qwen.  
Base rate: ds wins similar tasks ~0.4. Evidence: long refactor ↑ds; few tool needs ↑weak all; cost-sensitive ↓large models.

```
choice: ds
probabilities: glm 0.18 | ds 0.52 | kimi 0.21 | qwen 0.09
confidence: 0.74
basis: base-rate .4(ds); long refactor↑ds; cost-sensitive↓large
```

Program form: `{"choice":"ds","p":{"glm":0.18,"ds":0.52,"kimi":0.21,"qwen":0.09},"c":0.74}`
