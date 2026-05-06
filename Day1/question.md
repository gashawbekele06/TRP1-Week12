# Inference-time mechanics — LoRA Adapter Serving

**Asker:** Gashaw Bekele  
**Explainer:** Meseret  
**Topic area:** Inference-time mechanics — LoRA adapter serving  
**Date:** 2026-05-04  

---

## The Question

When a LoRA adapter is loaded for inference, the weights can either be
**merged** into the base model before generation (W' = W + B×A, permanent)
or applied **unmerged** as a side branch during each forward pass.

In my Week 11 project I trained a LoRA adapter
(`gashawbekele/tenacious-bench-lora-path-a`, r=16, α=16,
Qwen2.5-0.5B-Instruct) on 221 style-compliance SFT pairs.
Training loss dropped from 3.08 to 0.42 and output length fell −18%,
confirming the adapter learned something — yet rubric scores on
banned-phrase suppression were identical across trained, prompted,
and baseline conditions (Delta A = 0.00).

My `methodology_rationale.md` already diagnoses the primary cause:
the 0.5B backbone attention-copies banned phrases (e.g. "bench")
directly from the input context regardless of adapter weights —
a capacity limitation, not a pipeline failure. That diagnosis is
documented and accepted.

**My specific gap — the second diagnostic layer:**  
Once the backbone capacity limit is accepted as the root cause,
does the serving mode (merged vs unmerged) introduce any *additional*
divergence in output token distribution? Specifically: are merged and
unmerged LoRA weights mathematically guaranteed to produce identical
logits, or are there conditions — small rank, short context, inference
without dropout — where they silently diverge, making a trained
adapter behave as if it were not present?

---

## Why This Gap Matters for FDE Work

This is not only my situation. Two common FDE scenarios depend on
getting this right:

1. **Adapter deployment** — every time I ship a fine-tuned adapter
   for a client, I make a merge vs unmerged serving decision without
   knowing whether it changes the output distribution. A silent
   divergence means I could certify a model that ignores its own
   adapter weights in production.

2. **Endpoint migration** — a client switching from a PEFT unmerged
   inference server to a vLLM merged endpoint expects identical
   outputs. If the two modes diverge under any condition, the
   migration introduces silent regression with no visible error.

---

## Explainer Scope

Please cover:

1. The forward-pass arithmetic of merged vs unmerged LoRA — where
   exactly do the B×A weights enter the computation, and are they
   mathematically identical in both modes under fp16/bf16 precision?
2. Any known conditions where the two modes produce different outputs
   (numerical precision gaps, scaling factors, dropout at inference).
3. A minimal code example showing merged vs unmerged inference on the
   same prompt and comparing output logits or top-k tokens directly.
4. The practical FDE rule: when to merge and when not to, and what
   to check first if a deployed adapter appears to have no effect.

**Out of scope:** full LoRA training theory, quantization (int4/int8)
effects, multi-adapter serving — focus only on the merge vs unmerged
serving decision and its effect on output distribution under standard
fp16/bf16 inference.
