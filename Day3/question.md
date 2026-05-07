# day3_question.md

**Asker:** Gashaw Bekele
**Topic area:** Training and post-training mechanics — LoRA rank and what it can express
**Date:** 2026-05-07

---

## The Question

My Week 11 LoRA adapter (`gashawbekele/tenacious-bench-lora-path-a`, r=16, α=16,
Qwen2.5-0.5B-Instruct, 221 SFT pairs) produced a clear asymmetry:

- **Learned:** output length fell −18% (from 256 to 210 words on average)
- **Did not learn:** banned-phrase suppression — all three conditions (trained,
  prompted, baseline) produced the banned word at identical rates (Delta A = 0.00)

Both behaviors were represented in the 221 SFT training pairs,
yet only one transferred to the model's outputs.

**My specific gap:**
From a linear algebra standpoint, a LoRA update B×A has rank r —
meaning it can only move weight matrices within an r-dimensional subspace.

Are length reduction and token suppression expressible by the same class
of weight update, or does suppressing a token that appears in the input
context require a fundamentally different kind of transformation — one
that a low-rank update cannot represent regardless of how much training
data contains it?

---

## Why This Gap Matters for FDE Work

When I deploy LoRA fine-tuning for a client compliance requirement,
I am betting that the constraint is representable at the chosen rank.
Two FDE situations where this matters directly:

1. **Constraint fine-tuning** — a client needs an agent to never use
   specific competitor names or banned phrases. If token suppression is
   not representable at low rank, the model will appear to train normally
   (loss drops, other behaviors shift) while silently failing on the
   constraint — exactly what happened in my Week 11 experiment.

2. **Rank selection** — I chose r=16 following Unsloth defaults.
   Without understanding what rank controls conceptually, I cannot decide
   whether v0.2 should use r=32 (more expressive update) or a larger
   backbone (more parameters) — these are different fixes for different
   root causes.

---

## Explainer Scope

Please cover:

1. What the rank parameter in LoRA controls conceptually — what it means
   for a weight update to live in an r-dimensional subspace, and what
   classes of transformation that rules out.
2. Whether length scaling and token suppression belong to different
   transformation classes from the perspective of what a weight matrix
   update can express — and why one might be learnable at r=16 while
   the other is not.
3. The concept of intrinsic dimensionality of a fine-tuning task — how
   researchers think about how much "rank" a specific behavior requires,
   and where token suppression sits on that spectrum relative to style
   or length adjustments.
4. A conceptual rule for FDE practitioners: what types of fine-tuning
   objectives are safe bets for low-rank LoRA, and which signal that
   a higher rank or larger backbone is needed before training begins.

**Out of scope:** DPO, RLHF, reward models, quantization, specific
code implementations — focus only on the conceptual and mathematical
meaning of rank in supervised LoRA fine-tuning and what it implies
for constraint learnability.
