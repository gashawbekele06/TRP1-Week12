# TRP1 Week 12 — LoRA Adapter Serving: Merged vs Unmerged Inference

**Author:** Gashaw Bekele  
**Date:** 2026-05-04  
**Topic:** Inference-time mechanics — LoRA adapter serving

---

## Overview

This week's deep-dive investigates a second diagnostic layer on top of the Week 11 LoRA fine-tuning results: does the **serving mode** (merged vs unmerged) introduce silent divergence in output token distribution, and could that explain why a trained adapter appears to have no effect at evaluation time?

The adapter in question (`gashawbekele/tenacious-bench-lora-path-a`, r=16, α=16, Qwen2.5-0.5B-Instruct) was trained on 221 style-compliance SFT pairs. Training loss dropped from 3.08 → 0.42 and output length fell −18%, yet rubric scores on banned-phrase suppression were identical across trained, prompted, and baseline conditions (Delta A = 0.00).

---

## Repository Contents

| File | Description |
|------|-------------|
| [question.md](question.md) | The structured question Gashaw posed to Meseret — covers the specific diagnostic gap, why it matters for FDE work, and the explainer scope |
| [explainer.md](explainer.md) | Meseret's full technical answer — forward-pass arithmetic, divergence conditions, code example, and practical FDE rules |
| [main.py](main.py) | Project entry point (placeholder) |
| [pyproject.toml](pyproject.toml) | Python project configuration (requires Python ≥ 3.13) |

---

## Key Questions Answered

1. **Are merged and unmerged LoRA logits mathematically identical?**  
   Algebraically yes. In practice, fp16/bf16 rounding can produce gaps < 1e-3 — below the threshold that changes top-k selection under normal conditions.

2. **When do they silently diverge?**  
   Three conditions: floating-point accumulation differences, `lora_alpha/r` scaling applied inconsistently across code paths, and LoRA dropout left on at inference (`model.eval()` not called).

3. **What caused Delta A = 0.00 in Week 11?**  
   Primary cause: the 0.5B backbone attention-copies "bench" from the input context regardless of adapter weights — a capacity bottleneck, not a serving-mode bug. The serving-mode check is a second-order verification; it will not move the rubric score until backbone capacity is increased (e.g., Qwen2.5-1.5B in v0.2).

4. **Practical rule: merge or not?**  
   - **Merge** for single-adapter production serving where latency and memory matter.  
   - **Keep unmerged** during A/B testing, hot-swap multi-adapter serving, or when running logit comparison checks.

---

## Setup

```bash
# Create and activate virtual environment
python -m venv .venv
.venv\Scripts\activate      # Windows
# source .venv/bin/activate # Linux/macOS

# Install dependencies (none currently)
pip install -e .

# Run entry point
python main.py
```

---

## Related Work

- **Week 11:** LoRA fine-tuning on style-compliance SFT pairs — training, evaluation, and the `methodology_rationale.md` capacity-limit diagnosis.
- **Adapter on HuggingFace Hub:** `gashawbekele/tenacious-bench-lora-path-a`
- **Base model:** `unsloth/Qwen2.5-0.5B-Instruct`
