# canonical_list.md — Annotated Reading List for FDE Cohort

**Curated by:** Gashaw Bekele
**Context:** Week 12 deep-dive on LoRA serving, tool selection, training objectives,
and evaluation statistics — grounded in a production artifact with real gaps to close
**Date:** 2026-05-08

---

## How to Use This List

Each entry includes: what question it answers, what the load-bearing section is,
and why an FDE practitioner needs it specifically. Papers are ordered by the Week 12
day they became relevant — not by topic.

---

## Day 1 — LoRA Serving Mode

### 1. Hu et al. (2021) — LoRA: Low-Rank Adaptation of Large Language Models

**Citation:** Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... &
Chen, W. (2021). LoRA: Low-Rank Adaptation of Large Language Models. *arXiv:2106.09685*.
https://arxiv.org/abs/2106.09685

**What it answers:** The mathematical foundation of merged vs unmerged LoRA serving.
Section 4.2 establishes the algebraic equivalence: `W'·x = W·x + (α/r)·B·A·x` by
the distributive law. This is the proof that merged and unmerged modes produce
identical logits under exact arithmetic.

**Load-bearing section:** §4.2 "Practical Benefits and Limitations." One paragraph,
but it contains the merged-serving justification that every FDE adapter deployment
depends on.

**Why FDE practitioners need it:** Before reading §4.2, "merged vs unmerged" is
either a performance tradeoff or a mystery. After reading it, you know exactly when
the equivalence holds and what conditions break it (scaling bug, fp16 accumulation,
dropout active at inference). Cite this paragraph whenever you document a serving
decision in a model card or deployment review.

---

## Day 2 — Tool Selection in Function-Calling Agents

### 2. Yao et al. (2023) — ReAct: Synergizing Reasoning and Acting in Language Models

**Citation:** Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y.
(2023). ReAct: Synergizing Reasoning and Acting in Language Models. *ICLR 2023*.
https://arxiv.org/abs/2210.03629

**What it answers:** How tool selection is visible as a reasoning process, not a
routing black box. ReAct prompts force the model to emit a reasoning trace before
acting — and in those traces, you see the model explicitly reasoning
"I need X, tool Y handles X" before committing to a tool call.

**Load-bearing section:** §3 "ReAct Prompting" — the trace examples show the
selection mechanism more clearly than any theoretical description.

**Why FDE practitioners need it:** The common misconception is that the framework
routes tool calls. ReAct makes it visible that the model is doing all the selection
work. Understanding this changes how you write tool descriptions: they are not
documentation, they are the only lever for the selection decision.

---

### 3. Schick et al. (2023) — Toolformer: Language Models Can Teach Themselves to Use Tools

**Citation:** Schick, T., Dwivedi-Soni, J., Dessì, R., Raileanu, R., Lomeli, M.,
Zettlemoyer, L., ... & Scialom, T. (2023). Toolformer: Language Models Can Teach
Themselves to Use Tools. *NeurIPS 2023*. https://arxiv.org/abs/2302.04761

**What it answers:** Where tool-use capability comes from — not from explicit
selection supervision but from fine-tuning on examples where tool calls appear in
context and produce useful results. The model learns the association between query
type and tool through next-token prediction, not through a dedicated router.

**Load-bearing section:** §3 "Data Augmentation Pipeline" — this is where the
mechanism is clearest. The model sees completed tool calls in the training data
and generalizes the pattern.

**Why FDE practitioners need it:** Explains why rewriting tool descriptions works.
If selection came from a trained router, description rewriting would have no effect.
Because selection is language modeling over descriptions, description content is the
entire variable. This is also why anti-trigger phrases are more powerful than
trigger phrases — the model was trained on explicit avoidance patterns.

---

### 4. Liu et al. (2025) — ToolScope: Evaluating Tool Description Quality for LLM Agents

**Citation:** Liu, Y. et al. (2025). ToolScope: Evaluating Tool Description Quality
for LLM Agents. *arXiv:2501.XXXXX* [exact arXiv ID to be confirmed].

**What it answers:** The quantitative cost of overlapping tool descriptions.
ToolScope measured 8.8–38.6% degradation in selection accuracy across benchmarks
when tool descriptions overlap, compared to clearly distinct ones.

**Load-bearing section:** The benchmark results table — the range (8.8–38.6%) is
the number that turns "write clearer descriptions" from a style preference into a
correctness requirement.

**Why FDE practitioners need it:** You need a quantitative argument for why tool
descriptions deserve engineering effort. This is it. When a colleague says
"the model is smart enough to figure it out," the ToolScope number is the rebuttal.

---

## Day 3 — Training Objectives and Token Suppression

### 5. Hong et al. (2024) — ORPO: Monolithic Preference Optimization without Reference Model

**Citation:** Hong, J., Lee, N., & Thorne, J. (2024). ORPO: Monolithic Preference
Optimization without Reference Model. *arXiv:2403.07691*.
https://arxiv.org/abs/2403.07691

**What it answers:** Why standard SFT cannot suppress tokens. Section 3.2 of ORPO
documents the core problem: in pilot experiments, SFT on chosen responses increased
log-probability of both chosen and rejected responses simultaneously. The odds-ratio
term in ORPO adds an explicit negative signal on the rejected response, which SFT
lacks entirely.

**Load-bearing section:** §3 "Method" — specifically the motivation paragraph and
Equation 2. The pilot result (SFT increases log P of rejected responses) is the
cleanest possible evidence that SFT's positive-only objective cannot enforce hard
constraints.

**Why FDE practitioners need it:** Every compliance fine-tuning project — "never
say X," "always follow format Y" — implicitly assumes SFT will suppress the banned
behavior. ORPO's pilot result shows this assumption is wrong. If you are fine-tuning
for a constraint, you need either ORPO/DPO with rejected examples, or decoding-layer
enforcement via `bad_words_ids`. SFT alone will fail, and it will fail silently.

---

### 6. Welleck et al. (2019) — Neural Text Generation with Unlikelihood Training

**Citation:** Welleck, S., Kulikov, I., Roller, S., Dinan, E., Cho, K., & Weston, J.
(2019). Neural Text Generation with Unlikelihood Training. *arXiv:1908.04375*.
https://arxiv.org/abs/1908.04375

**What it answers:** The first rigorous statement of the problem ORPO later
solves — standard cross-entropy maximizes token likelihoods without any mechanism
to suppress specific tokens. Unlikelihood training adds a negative loss term on
tokens you want to avoid.

**Load-bearing section:** §3.1 "Sequence-Level Unlikelihood Loss" — the loss
formulation is clear and the examples show what it prevents.

**Why FDE practitioners need it:** Historical context for why `bad_words_ids` is
the pragmatic fix. Unlikelihood training, DPO, and ORPO all attack the same problem
at training time. `bad_words_ids` attacks it at inference time without retraining.
Knowing the 2019 framing makes the whole line of work legible.

---

## Day 4 — Effect Size and Statistical Power

### 7. Cohen, J. (1988) — Statistical Power Analysis for the Behavioral Sciences (2nd ed.)

**Citation:** Cohen, J. (1988). *Statistical Power Analysis for the Behavioral
Sciences* (2nd ed.). Lawrence Erlbaum Associates.

**What it answers:** When to use Cohen's h and when it does not apply. Chapter 7,
§7.2 defines Cohen's h as `h = 2·arcsin(√p₂) − 2·arcsin(√p₁)` and explicitly
restricts it to raw proportions from binomial samples — `p̂ = count/n` where variance
is `p(1−p)/n`. Cohen never applies h to weighted composites or aggregate scores.

**Load-bearing section:** Chapter 7, §7.2 — the boundary condition for h. Read
exactly one page. Chapter 2, §2.2 for Cohen's d (the continuous mean formulation).

**Why FDE practitioners need it:** The book is frequently cited without being read.
Most practitioners know "use Cohen's h for proportions, Cohen's d for means" but
cannot state the variance-stabilization rationale. Knowing why the arcsine transform
exists — it corrects for p(1−p)/n variance varying with p — immediately tells you
when it breaks: whenever your estimator is not a raw proportion from a binomial.

---

### 8. Lakens, D. (2013) — Calculating and Reporting Effect Sizes to Facilitate Cumulative Science

**Citation:** Lakens, D. (2013). Calculating and reporting effect sizes to facilitate
cumulative science: A practical primer for t-tests and ANOVAs. *Frontiers in
Psychology, 4*, 863. https://doi.org/10.3389/fpsyg.2013.00863

**What it answers:** Which variant of Cohen's d to use for paired designs.
Equation 6 defines `d_z = mean(Δ) / std(Δ)` where Δᵢ = score_condition1,ᵢ −
score_condition2,ᵢ. Table 1 explains why d_z is preferred over d_s (pooled-SD form)
for within-subjects designs: paired differences have smaller variance than raw
scores, so d_z is larger and predicts a smaller required n — which is correct
when the unit of analysis is the paired difference.

**Load-bearing section:** §"Cohen's d for within-subjects designs" (Equation 6)
and §"Which effect size to report" (Table 1). Both fit on one page.

**Why FDE practitioners need it:** Any LLM evaluation with paired conditions
(same task scored under condition A and condition B) is a within-subjects design.
Using d_s instead of d_z over-estimates the required n. Using Cohen's h on a
weighted rubric score inflates the required n by ~5×. This paper is the citation
that justifies the correct formula in any eval design document.

---

## Quick Reference: Which Formula to Use

| Metric type | Correct formula | Common mistake |
|---|---|---|
| `passing_tasks / total_tasks` from a single binary criterion | Cohen's h | — |
| Aggregate score, weighted rubric, per-task mean | Cohen's d_z (paired) | Cohen's h — inflates n by ~5× |
| Two independent group means | Cohen's d (pooled SD) | d_z (understates n) |
| Paired within-subject differences | Cohen's d_z | d_s (overstates n) |

---

## What to Read First

If you have 30 minutes: Hu et al. (2021) §4.2, Lakens (2013) §"d for
within-subjects" + Table 1, Cohen (1988) §7.2. These three sections resolve the
most common FDE mistakes in LoRA deployment, eval sizing, and effect size choice.

If you have two hours: read all eight entries in order. The Week 12 arc from
Day 1 to Day 4 is a natural reading sequence — each day's papers build on the
previous day's mechanism.
