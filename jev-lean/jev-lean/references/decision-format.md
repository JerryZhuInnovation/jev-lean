# Decision Output Format (Choice / Score / Noul + shapes)

Legal shapes for structured judgment. Machine-parseable and human-readable (or compact on purpose). No preamble, no phatic close, no long chain-of-thought.

**Answer body language** follows the user's message; keys stay English.

## Shape selection

| Signal | Shape |
|---|---|
| Program / API / router / JSON / schema; keys look like variables | compact |
| Default | readable-minimal |
| Process / reasons / derivation / teaching | expand as needed (numbers first) |

Purpose may be **clearly inferable**: e.g. "JSON for the router" → compact without asking.

## Field contract

| Field | compact key | Where | Required | Notes |
|---|---|---|---|---|
| `choice` | `choice` | Choice | yes | One option, or `UNKNOWN` |
| `probabilities` | `p` | Choice | yes | **All candidates**, sum=1 (±0.01) |
| `score` | `score` | Score | yes | `n/k` or number + scale |
| `noul` | `noul` | Noul | yes | `true`/`false` |
| `p` (truth prob.) | `p` | Noul | yes | P(proposition); implies false = 1-p |
| `confidence` | `c` | all | yes | 0–1, two decimals; self-rating, **uncalibrated** |
| `status` | `status` | unknown | when unknown | `insufficient_information` |
| `basis` | `basis` | all | yes | **One line**; base rate + main evidence. Program may use `null` |
| `calibrated` | `cal` | optional | no | External post-process only; never faked |

`basis` stays short unless the user asked for derivation. Decision blocks contain no reasoning process.

## Choice

```
choice: <LABEL>
probabilities: <L1> 0.xx | <L2> 0.yy | <L3> 0.zz
confidence: 0.xx
basis: <one line>
```

Rules:
- Options from user/context; never invent.
- **Every allowed option** has a probability; up to 5 options → all; more than 5 may use top-k + `other`, and `other` counts toward sum.
- Descending probabilities; sum=1.
- Near tie (gap below 0.08 between top two): keep two decimals; confidence ≤ 0.65.
- Thin info → `choice: UNKNOWN` or `status: insufficient_information`; never hard-pick.

compact:

```json
{"choice":"A","p":{"A":0.72,"B":0.21,"C":0.07},"c":0.81,"basis":"base-rate .30;E1↑;E2↓"}
```

High-cardinality two-stage (internal filter, one outward distribution):

```
stage1: top-k=[A, C, F]
choice: C
probabilities: C 0.48 | A 0.33 | F 0.19
confidence: 0.70
basis: coarse top3 then refine; full set not enumerated
```

## Score

```
score: <n>/<k>
confidence: 0.xx
basis: <one line>
```

Rules:
- Scale from the user; default 1–10. Never invent a scale.
- Integer scale → integer score; `.5` only if the scale has half-steps.
- Multi-dimension: one score per dimension; the caller weights. If the user wants one total, give the total and name main drivers in `basis`.
- Full-scale distribution **only if the consumer needs it**:

```
score: 8/10
distribution: 5:0.02 | 6:0.05 | 7:0.12 | 8:0.32 | 9:0.32 | 10:0.17
```

## Noul

```
noul: true|false
p: 0.xx
confidence: 0.xx
basis: <one line>
```

Rules:
- `p` = P(proposition is true); distribution `{true: p, false: 1-p}`.
- `p>0.5 → true`; `p<0.5 → false`; `p=0.50` → conservative false, lower confidence.
- Proposition must be unique in referent.
- Guardrails: model outputs `p`; thresholds belong to the caller.

## UNKNOWN

```
status: insufficient_information
confidence: 0.15
basis: no task description; cannot choose among A–D
```

or `choice: UNKNOWN` when it is an explicit candidate.  
One suggestion line (what to supply / escalate / human)—no orchestration.

## Confidence gate (caller reference)

| confidence | Suggested handling |
|---|---|
| ≥0.90 | may auto-execute |
| 0.60–0.90 | execute with audit |
| below 0.60 or insufficient | escalate / human / gather info |

The model's duty is honest numbers and unknowns—not acting as the gate.

## Non-decision short answers (readable-minimal)

```
<Conclusion, 1–3 sentences.>
[P=0.xx | confidence=0.yy]   # only if uncertain
[why: <one line>]              # only if c<0.7 or asked
```

## Batched output

Independent questions → multiple blocks, blank line between; one field group per question. Never merge two questions into one `basis`/`choice`.

## Validation responsibility

| Side | Owns |
|---|---|
| Skill / model | Required fields, sum=1, legal ranges, legal UNKNOWN, no reasoning spam |
| Program / caller | Schema, sum/range asserts, allowed options, optional L3 calibration |

On failure the caller retries or escalates; not orchestrated here.

## Anti-examples (invalid)

- "I lean toward A, fairly confident." → missing distribution/c/basis
- "P=0.85, confidence=0.85" → not distinguished
- "probabilities: A 1.00" on thin evidence → fake precision
- Winner without distribution → protocol break
- Hard pick B on missing info → should be UNKNOWN
- "Sure, let me analyze" / "Hope this helps" → meta-talk
- "First compared A/B then…" in a decision block → process leak
- `{"t":"c"}` for a human-only ask → wrong shape
