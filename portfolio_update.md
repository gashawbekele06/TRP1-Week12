# portfolio_update.md — Week 12 Portfolio Summary

**Author:** Gashaw Bekele
**Date:** 2026-05-08
**Audience:** FDE hiring manager or technical reviewer
**Artifact:** `gashawbekele/tenacious-bench-lora-path-a` (HuggingFace Hub)

---

## What I Built

A benchmark pipeline (`tenacious-bench`) for evaluating style-compliance in a
sales agent: banned-phrase suppression, response length, required signal phrases,
and JSON schema adherence. The pipeline runs paired ablations comparing a trained
LoRA adapter against a prompted baseline and a raw baseline, scores rubric
criteria programmatically, and produces a confidence-interval report. The adapter
was trained on 221 SFT pairs (Qwen2.5-0.5B-Instruct, r=16, α=16) and published
to HuggingFace Hub.

---

## What I Discovered

**Week 11 produced a null result.** All three conditions — trained, prompted,
baseline — scored identically on the banned-phrase criterion (Delta A = 0.00).
Training loss dropped 3.08 → 0.42 and output length fell 18%, which confirmed the
adapter learned something. The null rubric score meant either the adapter was being
ignored at inference, or the scoring pipeline was broken, or the training objective
could not enforce the constraint.

**Week 12 traced the null result to three separable causes:**

1. **Scoring bug (corrected).** The `score_output()` function was evaluating the
   full conversation string — input + output — instead of the assistant response
   only. The banned phrase appeared in the input field (`bench_summary`), so every
   output scored as containing it regardless of what the model generated. Fixing the
   extraction function revealed the true delta: +7.0 pp (baseline 53.2% → trained
   60.1% on n=3 corrected tasks).

2. **Training objective mismatch (diagnosed).** Standard SFT maximizes the
   probability of chosen tokens with no gradient signal to suppress rejected ones.
   ORPO (Hong et al., 2024) documented this precisely: SFT increases log-probability
   of both chosen and rejected responses together. The banned phrase remained likely
   not because the rank was too low or the backbone too small, but because the
   training objective never generated a suppression gradient. The fix is
   `bad_words_ids` at decoding time (immediate, no retraining) and ORPO with
   rejected responses (second priority).

3. **Underpowered evaluation (quantified).** The corrected ablation at n=3 has
   10.4% statistical power. The null result at n=3 is expected even if the effect
   is real — it is a measurement failure, not a model failure. The correct effect
   size for a paired rubric score is Cohen's d_z = 0.304 (not Cohen's h, which
   inflates the required n by ~5×). At d_z = 0.304, 80% power requires 85 tasks.
   Expanding the held-out set from 17 to 85 tasks is the evaluation priority for v0.2.

---

## What Changed in the Artifact

| Component | Before Week 12 | After Week 12 |
|---|---|---|
| Scoring | Evaluated full conversation string | Extracts assistant response only |
| `ablation_results.json` | delta: 0.000, p: 1.0, CI: [0, 0] | delta: +0.070, p: 0.25, CI: [-0.196, +0.203] |
| `methodology_rationale.md` | "Capacity limitation of backbone" | Training objective mismatch; v0.2 fix hierarchy stated |
| `improvement_report.md` | Not present | Created; before/after table, per-task scores, v0.2 queue |
| `bad_words_ids` | Not present | Function implemented in `scoring_evaluator.py` |
| `held_out/` | 3 tasks | 17 tasks; target n=85 documented |
| Tool descriptions (`query_agent.py`) | Mechanism-only descriptions | Trigger + examples + anti-trigger; 6/6 smoke test passing |
| Power analysis | d ≈ 0.35, n ≈ 66 (informal) | d_z = 0.304, n = 85 (Lakens 2013) |

---

## Technical Skills Demonstrated

**LoRA deployment diagnostics.** Can distinguish between three causes of a null
adapter result — serving mode divergence, training objective mismatch, and
evaluation measurement failure — and rule each one out or in with a code-level check.
Can explain the algebraic equivalence of merged vs unmerged LoRA (Hu et al. 2021 §4.2)
and the three conditions that break it.

**Function-calling agent design.** Understands the two-layer model of tool selection
(model selects on description content, framework enforces schema format only) and
can apply the trigger + examples + anti-trigger pattern to eliminate routing failures
in multi-tool agents. Can cite ToolScope (8.8–38.6% accuracy degradation) as the
quantitative justification.

**Evaluation statistics.** Can select the correct effect size formula for a given
metric type (Cohen's h for raw proportions, Cohen's d_z for paired aggregate scores)
and explain the distributional reason for the choice. Can compute Wilson score CIs,
Cohen's κ, F1, and paired bootstrap tests. Can design a power-adequate evaluation
before running it.

**Training objective analysis.** Can explain why SFT cannot enforce hard token
suppression (positive-only objective, no negative gradient) and prescribe the correct
fix class: decoding-layer enforcement (bad_words_ids) for immediate compliance,
ORPO/DPO for permanent training-time enforcement.

---

## Artifacts Publicly Available

| Artifact | Location |
|---|---|
| LoRA adapter | `gashawbekele/tenacious-bench-lora-path-a` (HuggingFace Hub) |
| Benchmark pipeline | `TRP1-Week12` repository (GitHub) |
| Tool selection explainer (by pair) | medium.com/@chalielijalem/how-tool-selection-actually-works-in-function-calling-agents |
| Weekly synthesis | `synthesis.md` in this repository |
| Annotated reading list | `canonical_list.md` in this repository |

---

## v0.2 Priority Queue

1. Enable `bad_words_ids` in `scoring_evaluator.py` (implemented, not yet wired to
   `model.generate()` call in ablation runner)
2. Expand `held_out/` from 17 to 85 tasks by moving from `train_filtered/`
3. Rerun ablation at n=85 to reach 80% power at observed d_z = 0.304
4. If n=85 result is still not significant: retrain with ORPO using rejected responses
   that contain the banned phrase ("bench")
5. If constraint passes but score is still below target: Qwen2.5-1.5B backbone

The target is a defensible, reproducible result at 80% power — not a larger model.
