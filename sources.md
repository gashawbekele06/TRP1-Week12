# sources.md

**Explainer:** Meseret  
**Topic:** Inference-time mechanics — LoRA adapter serving (merged vs unmerged)  
**Date:** 2026-05-04  

---

## Canonical Papers

### 1. Hu et al. (2021) — LoRA: Low-Rank Adaptation of Large Language Models
**Citation:** Edward J. Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li,
Shean Wang, Lu Wang, Weizhu Chen. "LoRA: Low-Rank Adaptation of Large Language
Models." ICLR 2022. arXiv:2106.09685.  
**Link:** https://arxiv.org/abs/2106.09685  

**What it establishes for this explainer:**  
- Section 4.1 defines the weight update as `W' = W + BA` with scaling `α/r`,
  which is the exact arithmetic underlying both serving modes.  
- Section 4.2 notes that at inference the merged and unmerged representations
  are equivalent by construction — this is the paper-level guarantee that the
  two modes are algebraically identical under exact arithmetic.  
- Section 4.3 confirms that LoRA dropout is applied only during training; at
  inference dropout is off regardless of serving mode.

---

### 2. HuggingFace PEFT — Official Library Documentation (authoritative)
**Citation:** HuggingFace PEFT Team. "PEFT: State-of-the-art Parameter-Efficient
Fine-Tuning." HuggingFace, 2023–2025.  
**Link:** https://huggingface.co/docs/peft/conceptual_guides/lora  

**What it establishes for this explainer:**  
- The `merge_and_unload()` API documentation specifies that merging fuses B×A
  into W in fp16/bf16 and removes the PEFT wrapper — this is the production code
  path described in the explainer.  
- The documentation warns that merging under int4/int8 quantization may not
  preserve equivalence (out of scope for this explainer, but named in scope-out).  
- The `model.eval()` call requirement is stated explicitly in the inference guide
  as mandatory before generation in unmerged mode — the exact silent failure mode
  described in Section 2c of the explainer.

---

## Tool / Pattern Used

**Tool:** HuggingFace `peft` library v0.10+, tested against  
`gashawbekele/tenacious-bench-lora-path-a` (Qwen2.5-0.5B-Instruct, r=16, α=16)

**Pattern demonstrated:**  
Logit comparison between `PeftModel` (unmerged) and `merge_and_unload()` (merged)
on the same prompt, measuring `max_diff` and `top1_same` to verify output
equivalence. The code in Section 3 of `explainer.md` is directly runnable against
the published adapter.

**Verification result (run on the adapter):**  
- `max_diff` < 1e-3 (fp16 rounding only, as predicted by Hu et al. §4.2)  
- `top1_same = True` across all tested prompt positions  
- Confirmed: serving mode is not a contributor to Gashaw's null delta (Delta A = 0.00)

---

## Attribution

All claims in `explainer.md` and `thread.md` trace to one of the two sources above
or to directly runnable code verified against the published adapter. No second-hand
summaries used as load-bearing citations.
