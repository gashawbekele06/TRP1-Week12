# synthesis.md — Week 12 Knowledge Synthesis

**Author:** Gashaw Bekele
**Date:** 2026-05-08
**Period covered:** Week 12, Days 1–4
**Artifact referenced:** `gashawbekele/tenacious-bench-lora-path-a` (Week 11)

---

## Overview

Week 12 began with a production artifact that produced a null result and a
diagnosis I could state but not fully defend. My Week 11 LoRA adapter
(`r=16, α=16, Qwen2.5-0.5B-Instruct, 221 SFT pairs`) dropped training loss
from 3.08 to 0.42 and reduced output length by 18%, yet banned-phrase
suppression scores were identical across all conditions (Delta A = 0.00). The
`methodology_rationale.md` named backbone capacity as the root cause but left
three diagnostic layers unresolved: serving mode, training objective class, and
evaluation measurement validity.

Over four days, eight technical gaps were closed — four through questions Gashaw
posed to his pairs, four through explainers Gashaw wrote for his pairs — and
every gap closure produced a concrete edit to a shipped artifact.

---

## Day 1 — LoRA Serving Mode: Algebraic Equivalence and Its Limits

**Gap (Gashaw, asker):** Are merged and unmerged LoRA weights mathematically
guaranteed to produce identical output token distributions, and under what
conditions does that guarantee break?

Before this day, "serving mode" was a suspected alternative explanation for the
null delta. The hypothesis was untested: if unmerged mode silently diverges from
merged, the adapter could be ignored at generation time even though training
succeeded.

Meseret's explainer resolved this by walking through the forward-pass arithmetic
in both modes. In merged mode, the adapter is fused once: `W' = W + (α/r)·B·A`,
and all subsequent passes execute a standard forward pass with no overhead. In
unmerged mode, the adapter branch `(α/r)·B·A·x` runs in parallel on every
forward pass and its output is added to the base pass. By the distributive law,
`W'·x = W·x + (α/r)·B·A·x` — the results are algebraically identical.

In fp16/bf16, floating-point accumulation introduces gaps below 1e-3 in logit
space — real but sub-threshold for top-1 token selection under normal conditions.
Three conditions break equivalence materially: (1) a scaling-factor bug where
`α/r` is applied inconsistently across code paths, (2) accumulated fp16 drift that
in practice does not change token selection, and (3) LoRA dropout left active at
inference because `model.eval()` was not called — the only condition that produces
a materially wrong output distribution, and the most common silent deployment bug.

A logit comparison run against the published adapter confirmed: `max_diff < 1e-3`
and `top1_same = True`. Serving mode was ruled out as a contributor to the null
delta. The `methodology_rationale.md` was updated with this verification, making
the backbone capacity diagnosis defensible against the most obvious engineering
challenge.

**Gap (Meseret, asker — Gashaw as explainer):** Meseret's question addressed a
related serving-mechanics gap in her own pipeline. Gashaw's explainer closed the
gap; Meseret confirmed closure in the evening call signoff.

---

## Day 2 — Tool Selection: Two Layers, One Lever

**Gap (Gashaw, asker):** When a function-calling agent has multiple tools
available and two tools serve overlapping purposes, what determines which one the
model selects? Is the decision made by the model, the framework, or both?

Before this day, tool selection was treated as a prompt-engineering art with no
principled basis. The failure in `query_agent.py` — numeric questions consistently
routed to `semantic_search` instead of `structured_query` — was understood as a
symptom but not diagnosable at the mechanism level.

Charlie's explainer (published at medium.com) separated the selection process into
two orthogonal layers. Layer 1 is entirely the model's: tool definitions are
serialized as text, injected into the context window, and the model performs
next-token prediction over them using the same weights it uses for all other output.
There is no separate router. Tool selection is language modeling over descriptions.
Layer 2 is the framework's: after the model has already decided which tool to call,
the framework enforces schema compliance — correct field names, required parameters,
valid JSON. Layer 2 constrains format only. It has no influence on which tool is
chosen.

The practical implication: you can call the wrong tool with perfectly valid JSON
and receive no error. ToolScope (Liu et al., 2025) quantified the cost of
description overlap: 8.8–38.6% degradation in selection accuracy when tools have
overlapping descriptions. The highest-signal fix is the anti-trigger phrase —
"NOT for similarity-based queries" suppresses misrouting more reliably than a
trigger phrase promotes correct routing, because explicit negative examples in
training produced stronger avoidance behavior than semantic similarity matching.

All three tool descriptions in `query_agent.py` were rewritten using the
trigger + examples + anti-trigger pattern. A 6-query smoke test covering the
`semantic_search` / `structured_query` boundary passed 6/6 after the rewrite.

**Gap (Charlie, asker — Gashaw as explainer):** Charlie's question addressed the
image tokenization pipeline in GPT-4o-mini vision processing — specifically, what
`detail: high` does to an image before it reaches the model, and how image tokens
combine with text tokens. Gashaw's explainer covered the tiling algorithm, the
token budget arithmetic, and the unified-sequence attention mechanism. Charlie
confirmed gap closure in the evening call.

---

## Day 3 — Training Objective vs Rank Capacity: A Critical Distinction

**Gap (Gashaw, asker):** My LoRA adapter learned length reduction but not
banned-phrase suppression. Are these expressible by the same class of low-rank
weight update, or does token suppression require a fundamentally different
transformation?

Before this day, the `methodology_rationale.md` used the phrase "capacity
limitation of the backbone" without distinguishing rank capacity from objective
capacity. These are different failure modes with different fixes, and the
document was conflating them.

Nebiyou's explainer separated the two concepts. Rank capacity determines which
weight directions can be updated — a rank-16 update can represent 16 linearly
independent directions in weight space. Length reduction is a dense, many-token
preference: across 221 examples, hundreds of positions consistently point toward
shorter completions. This pattern sits in a low-dimensional subspace and is
learnable at r=16.

Token suppression is a sparse, local, negative constraint: "when this exact token
is likely in the current context, do not choose it." This constraint has no direct
gradient path in a standard SFT objective. The SFT loss maximizes the probability
of chosen tokens — it provides no explicit signal to lower the probability of
rejected tokens. ORPO (Hong et al., 2024) demonstrated that SFT on chosen
responses actually increases log-probability of both chosen and rejected responses
simultaneously. The banned phrase remained likely not because r=16 was too small,
but because the training objective never generated a gradient that suppressed it.

A decisive proof: when the model was instructed to use the banned word and a
`logit_bias=-100` block was active, it failed to produce the word across 5/5
trials. Without the block, it produced it 5/5 times. This confirmed that
probability-shifting (SFT, prompting) and token-blocking (logit masking) are
mechanically different enforcement classes. The fix hierarchy for v0.2 is: first
`bad_words_ids` (no retraining, direct enforcement), then ORPO with rejected
responses containing the banned phrase, then larger backbone only if both fail.

The `methodology_rationale.md` was updated to replace "capacity limitation of
the backbone" with a precise diagnosis: the training objective is the bottleneck,
not the rank or backbone size.

**Gap (Nebiyou, asker — Gashaw as explainer):** Nebiyou's question addressed
statistical power for proportion tests — specifically, whether his n=50 was
sufficient to detect a +14 pp lift given p=0.23. Gashaw's explainer covered the
Cohen's h power formula, one-tailed vs two-tailed thresholds (n=50 clears the
one-tailed target but not two-tailed at n=60), and the bootstrap CI construction.
The power table confirmed Nebiyou's eval size was sufficient for the observed
effect size. Nebiyou confirmed gap closure in `nebiyou_signoff.md`.

---

## Day 4 — Evaluation Measurement: Cohen's h vs Cohen's d_z

**Gap (Gashaw, asker):** My power analysis using Cohen's h said I needed 410
held-out tasks. A colleague's calculation using Cohen's d gave 85. The 5× gap
could not be a rounding error — one formula had to be wrong for this data type.

The root cause was a distributional mismatch. Cohen's h applies the arcsine
transform `φ = 2·arcsin(√p)` to variance-stabilize a binomial proportion
`p̂ = count/n`, where variance is `p(1−p)/n` — a function of p. My rubric score
is a weighted sum of four heterogeneous Bernoulli checks:
`S = (0.285·X₁ + 0.285·X₂ + 0.095·X₃ + 0.05·X₄) / 0.715`. Its variance is
`Σ(wᵢ/W)²·pᵢ(1−pᵢ)` — a Poisson-binomial mixture. The arcsine transform is
not calibrated for this variance structure. Applied near p̄ ≈ 0.57, it compresses
the apparent effect and inflates the required n by ~5×.

The correct formula is Cohen's d_z for paired designs (Lakens 2013):
`d_z = mean(Δ) / std(Δ) = 0.070 / 0.230 = 0.304`. At d_z = 0.304, 80% power
requires ~85 tasks. The Week 11 null result at n=3 is fully explained by the
10.4% power at that sample size. The decision rule is clean: when the metric is
`count/n` from a single binomial, use Cohen's h; when it is an aggregate score
or weighted rubric, use Cohen's d_z.

Both `improvement_report.md` and `methodology_rationale.md` were updated to
reflect the corrected effect size, the power table at n=3/17/50/85/100, and
the concrete expansion target: move ~68 tasks from `train_filtered/` to
`held_out/` to reach n=85.

**Gap (Eyobed, asker — Gashaw as explainer):** Eyobed's question addressed
evaluation metrics for an imbalanced binary judge — 41 test examples with an
~80% PASS base rate. Gashaw's explainer showed that 92.7% accuracy is only 12
pp above a naive always-PASS baseline. Cohen's κ = 0.78 and F1 = 95.4% provide
the correct signal on the imbalanced set. The Wilson CI [80.6%, 97.5%] and the
two-proportion z-test (p = 0.047) confirmed the improvement over a 76.92%
baseline is just significant at α = 0.05. The 18% false-negative rate on
disqualification tasks is the number that matters most for a compliance judge —
and the one that accuracy hides. Eyobed confirmed gap closure verbally on the
evening call.

---

## Cross-Cutting Themes

**What a gap actually is.** Across all four days, the morning call sharpenings
converged on the same discipline: a gap is not a vague uncertainty — it is a
named mechanism you cannot currently explain, traceable to a specific failure in
a specific artifact. Meseret's push on Day 1 ("name the mechanism you're missing")
and Charlie's push on Day 2 ("you're asking three separate questions, pick one")
both forced this specificity.

**Positive-only training cannot enforce hard constraints.** Days 1, 3, and 4
each touched a version of this theme. SFT increases probability of desired tokens
but provides no gradient signal to suppress undesired ones. This is the correct
diagnosis for the Week 11 null result and the reason `bad_words_ids` is the
highest-priority fix for v0.2 — it enforces at the logit level, not the gradient
level, and works regardless of what the adapter learned.

**Small n is not evidence of no effect.** At n=3, the ablation had 10.4% power.
At n=17, it had 27.8%. The Week 11 null result at n=3 was a measurement failure,
not a model failure. The corrected scoring fix revealed a +7.0 pp delta that was
always there; it was invisible because the scoring pipeline was evaluating the
full conversation string rather than the assistant response only.

**Effect size formula choice determines experiment feasibility.** Cohen's h at
n=410 vs Cohen's d_z at n=85 is not a minor implementation detail — it is the
difference between an achievable evaluation target and one that would likely
never be collected.

---

## What Changed in the Artifact

| Before Week 12 | After Week 12 |
|---|---|
| Null delta at n=3, cause unclear | Corrected score: +0.070 delta, 10.4% power explains null |
| "Capacity limitation of backbone" | Training objective mismatch; SFT cannot enforce hard constraints |
| Serving mode untested | max_diff < 1e-3, top1_same = True; serving ruled out |
| n=66 for 80% power (d ≈ 0.35, informal) | n=85 for 80% power (d_z = 0.304, Lakens 2013) |
| Tool descriptions: mechanism only | Trigger + examples + anti-trigger; 6/6 smoke test passing |
| held_out: 3 tasks | held_out: 17 tasks; target: 85 |

The artifact is not finished. The v0.2 priority queue is: (1) `bad_words_ids`
enforcement, (2) expand `held_out` to 85 tasks and rerun ablation, (3) ORPO
retraining with rejected responses. Each item now has a principled justification
from a closed gap.
