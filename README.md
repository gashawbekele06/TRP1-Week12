# TRP1 Week 12 — Deep-Dive Inquiry: LoRA, Tool Selection, Training Objectives, and Evaluation Statistics

**Author:** Gashaw Bekele
**Dates:** 2026-05-04 to 2026-05-08
**Artifact:** `gashawbekele/tenacious-bench-lora-path-a` (HuggingFace Hub)
**Base model:** `unsloth/Qwen2.5-0.5B-Instruct`

---

## Overview

Four days of paired technical inquiry, each day producing: a question, an explainer
(written or received), a grounding commit to a shipped artifact, a signoff confirming
gap closure, and a tweet thread making the finding public.

The underlying artifact is a style-compliance benchmark pipeline from Week 11 that
produced a null rubric result (Delta A = 0.00). Week 12 traced the null result to
three separable causes — a scoring extraction bug, a training objective mismatch,
and an underpowered evaluation — and produced concrete fixes for each.

---

## Repository Contents

### Summary Documents

| File | Description |
|------|-------------|
| [synthesis.md](synthesis.md) | Week synthesis — 8 gaps closed, cross-cutting themes, what changed in the artifact |
| [canonical_list.md](canonical_list.md) | Annotated reading list for FDE cohort (8 papers, load-bearing sections identified) |
| [portfolio_update.md](portfolio_update.md) | One-page technical summary for hiring manager or project reviewer |

---

### Day 1 — LoRA Serving Mode: Merged vs Unmerged
**Topic:** Inference-time mechanics | **Pair:** Meseret

| File | Description |
|------|-------------|
| [Day1/question.md](Day1/question.md) | Gashaw's question: are merged and unmerged LoRA algebraically equivalent, and when does that break? |
| [Day1/explainer.md](Day1/explainer.md) | Meseret's answer: forward-pass arithmetic, 3 divergence conditions, code example, FDE rule |
| [Day1/sources.md](Day1/sources.md) | Hu et al. (2021) §4.2 — the load-bearing citation |
| [Day1/morning_call_summary.md](Day1/morning_call_summary.md) | How the question was sharpened from "could serving mode explain it?" to a mechanism question |
| [Day1/evening_call_summary.md](Day1/evening_call_summary.md) | Feedback on Meseret's draft; 3 specific revisions requested and made |
| [Day1/signoff.md](Day1/signoff.md) | Gashaw confirms: max_diff < 1e-3, top1_same = True, serving mode ruled out |
| [Day1/grounding_commit.md](Day1/grounding_commit.md) | `methodology_rationale.md` updated with serving-mode verification paragraph |
| [Day1/thread.md](Day1/thread.md) | 6-tweet thread on merged vs unmerged LoRA equivalence |

**Key finding:** Serving mode is not a factor in the null result. The three conditions
that break merged/unmerged equivalence are known and checkable. `model.eval()` is the
most common silent bug.

---

### Day 2 — Tool Selection in Function-Calling Agents
**Topic:** Agent & tool-use internals | **Pair:** Charlie Lijalem

| File | Description |
|------|-------------|
| [Day2/question.md](Day2/question.md) | Gashaw's question: what mechanism selects a tool, and what makes a description reliable? |
| [Day2/explainer.md](Day2/explainer.md) | Charlie's answer (published on Medium): two-layer model, ToolScope measurement, three-component pattern |
| [Day2/sources.md](Day2/sources.md) | Yao et al. (ReAct), Schick et al. (Toolformer), Liu et al. (ToolScope) |
| [Day2/morning_call_summary.md](Day2/morning_call_summary.md) | Question sharpening: from 3 sub-questions to 1 specific routing failure |
| [Day2/evening_call_summary.md](Day2/evening_call_summary.md) | Feedback exchange; Gashaw rewrites tool descriptions during the call |
| [Day2/signoff.md](Day2/signoff.md) | Gashaw confirms: two-layer model understood, anti-trigger insight, descriptions rewrote |
| [Day2/grounding_commit.md](Day2/grounding_commit.md) | `query_agent.py` tool descriptions rewritten; 6/6 smoke test passing |
| [Day2/thread.md](Day2/thread.md) | 6-tweet thread on tool selection mechanism |

**Key finding:** The framework enforces format only. The model selects on description
content. Anti-trigger phrases ("NOT for X") suppress misrouting more reliably than
trigger phrases promote correct routing.

---

### Day 3 — LoRA Rank vs Training Objective for Token Suppression
**Topic:** Training and post-training mechanics | **Pair:** Nebiyou Abebe

| File | Description |
|------|-------------|
| [Day3/question.md](Day3/question.md) | Gashaw's question: are length reduction and token suppression expressible by the same weight update class? |
| [Day3/explainer.md](Day3/explainer.md) | Gashaw's explainer for Nebiyou (statistical power for proportion tests) |
| [Day3/sources.md](Day3/sources.md) | Hong et al. (ORPO), Welleck et al. (Unlikelihood Training), Cohen (1988) |
| [Day3/morning_call_summary.md](Day3/morning_call_summary.md) | Question sharpening for both Gashaw and Nebiyou |
| [Day3/evening_call_summary.md](Day3/evening_call_summary.md) | Feedback exchange; training objective vs rank distinction clarified |
| [Day3/signoff.md](Day3/signoff.md) | Gashaw confirms: SFT objective mismatch is the primary cause; v0.2 fix hierarchy stated |
| [Day3/grounding_commit.md](Day3/grounding_commit.md) | `methodology_rationale.md` updated: "capacity limitation" replaced with precise objective diagnosis |
| [Day3/nebiyou_signoff.md](Day3/nebiyou_signoff.md) | Nebiyou confirms Gashaw's statistical power explainer closed his gap |
| [Day3/nebiyou_grounding_commit.md](Day3/nebiyou_grounding_commit.md) | Nebiyou's artifact updated based on Gashaw's explainer |
| [Day3/thread.md](Day3/thread.md) | 6-tweet thread on SFT objective mismatch and token suppression |

**Key finding:** The primary cause of the asymmetry is the training objective, not
rank or backbone size. SFT increases log-probability of both chosen and rejected
responses (ORPO pilot result). `bad_words_ids` is the correct first fix.

---

### Day 4 — Cohen's h vs Cohen's d_z for Weighted Rubric Scores
**Topic:** Evaluation and statistics | **Pair:** Eyobed Feleke

| File | Description |
|------|-------------|
| [Day4/question.md](Day4/question.md) | Gashaw's question: h gives n=410, d gives n=85 — which is correct for a weighted rubric score? |
| [Day4/eyobed_explainer.md](Day4/eyobed_explainer.md) | Eyobed's answer: Poisson-binomial mixture variance, d_z formula, power table |
| [Day4/eyobed_sources.md](Day4/eyobed_sources.md) | Cohen (1988) §7.2, Lakens (2013) eq.(6), Python verification |
| [Day4/explainer.md](Day4/explainer.md) | Gashaw's explainer for Eyobed (imbalanced binary judge: Wilson CI, κ, F1, FNR) |
| [Day4/sources.md](Day4/sources.md) | Wilson (1927), Cohen (1960) — both load-bearing |
| [Day4/evening_call_summary.md](Day4/evening_call_summary.md) | Feedback in both directions; 3 specific revisions each |
| [Day4/signoff.md](Day4/signoff.md) | Gashaw confirms: d_z=0.304, n=85, Poisson-binomial distinction understood |
| [Day4/grounding_commit.md](Day4/grounding_commit.md) | `improvement_report.md` and `methodology_rationale.md` updated: n=66→85, h→d_z |
| [Day4/thread.md](Day4/thread.md) | 6-tweet thread on h vs d_z for LLM rubric scores |

**Key finding:** Cohen's h applies to raw proportions (binomial variance p(1−p)/n
only). Weighted rubric scores are Poisson-binomial mixtures. h inflates required n
by ~5× near p̄ ≈ 0.57. The correct formula is Cohen's d_z = mean(Δ)/std(Δ).

---

## Public Artifacts

| # | Topic | Blog Post | Tweet Thread |
|---|-------|-----------|-------------|
| 1 | LoRA Merged vs Unmerged | [Medium ↗](https://medium.com/@gashawbekelek/lora-merged-vs-unmerged-the-three-conditions-that-break-equivalence-e055d6692cf3) | ⬜ to be posted |
| 2 | How Tool Selection Works | [Medium ↗](https://medium.com/@gashawbekelek/how-tool-selection-actually-works-in-function-calling-agents-3e42bb32c6e8) | ⬜ to be posted |
| 3 | Why SFT Cannot Suppress Tokens | ⬜ to be published | ⬜ to be posted |
| 4 | Cohen's h vs Cohen's d_z | ⬜ to be published | ⬜ to be posted |
| 5 | Week 12 Synthesis | ⬜ to be published | ⬜ to be posted |

---

## Week 11 Artifact Changes (Applied During Week 12)

| File | Change |
|------|--------|
| `ablations/run_ablations.py` | Added `extract_assistant_response()`; rewired `score_output()` to use it |
| `ablation_results.json` | delta: 0.000 → +0.070; p: 1.0 → 0.25; CI: [0,0] → [-0.196, +0.203] |
| `scoring_evaluator.py` | Added `build_bad_words_ids()`, `wilson_ci()`, `cohen_kappa()` |
| `tenacious_bench_v0.1/held_out/` | Expanded from 3 to 17 tasks |
| `methodology_rationale.md` | Serving-mode verification; training objective diagnosis; d_z power analysis |
| `improvement_report.md` | New file: before/after table, per-task scores, v0.2 priority queue |

---

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate      # Windows
pip install -e .
python main.py
```

---

## Key Numbers

| Metric | Value |
|--------|-------|
| Corrected delta (n=3) | +0.070 (baseline 53.2% → trained 60.1%) |
| Power at n=3 | 10.4% |
| Power at n=17 | 27.8% |
| Power at n=85 | 80.1% (target) |
| Effect size | Cohen's d_z = 0.304 |
| Required n for 80% power | 85 tasks |
| Tool routing accuracy after rewrite | 6/6 smoke test passing |
| κ (binary judge) | 0.78 (substantial agreement) |
