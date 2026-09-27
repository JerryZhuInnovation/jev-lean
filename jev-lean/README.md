# jev-lean

**Lean, calibrated, low-hallucination response discipline for any LLM agent.**  
A portable [skill](./jev-lean/SKILL.md) (not installed by default) inspired by Jev / System One-style structured decisions—adapted for generative models.

中文：精简输出 + 真分布/置信度 + 最小幻觉的**通用**响应规范；判断题输出完整概率分布与 `confidence`，支持 `UNKNOWN`。正文协议为英文，**按用户语言作答**。

## Why

| Problem | What this skill does |
|---|---|
| Filler, restatement, token waste | Hard constraints; quality red lines never cut |
| Ad-hoc "80% sure" theater | Full candidate **distribution (sum=1)** + separate `confidence` |
| Fabricated facts | `[confirmed]` / `[inferred]` / `[guess]`; facts may be blank |
| Open-ended mush | Closed question + fixed options first |

**Not** an agent router, model registry, or backend—those stay in the host. This skill only disciplines **how the model answers**.

## Install (pick one; not pre-installed)

| Target | Path |
|---|---|
| MiMo Desktop (global) | `~/.config/mimocode/skills/jev-lean/` |
| MiMo project | `<project>/.mimocode/skills/jev-lean/` |
| Claude Code / open agents | `~/.agents/skills/jev-lean/` or `<project>/.agents/skills/jev-lean/` |

Copy the `jev-lean/` folder. Directory name must match frontmatter `name`.

```bash
cp -R jev-lean ~/.config/mimocode/skills/
```

## Trigger

- Slash: `/jev-lean`
- zh: 少说废话、省 token、精简、给置信度、估算概率、判断一下、jev、最小幻觉
- en: be concise, save tokens, give confidence, estimate probability, no small talk, minimal hallucination

## What you get

**Three primitives** (machine-consumable):

```
choice: A
probabilities: A 0.72 | B 0.21 | C 0.07
confidence: 0.81
basis: base-rate .30; E1↑; E2↓
```

- **Choice** — full distribution + winner (or `UNKNOWN`)
- **Score** — `7/10` (+ optional full-scale distribution)
- **Noul** — `p` for a yes/no proposition
- **Unknown** — `status: insufficient_information` instead of a forced pick
- **Shapes** — `compact` JSON for programs; readable-minimal default; expand only for process/reasoning

**Language:** protocol is English; **answers follow the user's language**; keys stay English.

**Calibration honesty:** `confidence` / `probabilities` are **uncalibrated** self-reports. Real calibration (L3 `calibrated`) is an external program concern.

## Layout

```
.
├── README.md
├── LICENSE
├── .gitignore
└── jev-lean/
    ├── SKILL.md
    └── references/
        ├── probability.md
        ├── decision-format.md
        └── anti-hallucination.md
```

## Validation contract

| Side | Responsibility |
|---|---|
| Skill / model | Required fields, sum=1, legal unknown, no CoT in decision blocks |
| Caller / program | Schema, range/sum asserts, allowed options, optional historical calibration |

## Non-goals (v1)

Model registry, decision backend switching, escalation orchestration, test suites, temperature scaling—host or future engineering, not this skill.

## License

[MIT](./LICENSE)