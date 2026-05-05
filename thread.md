# thread.md — Tweet Thread (Twitter-ready)

**Author:** Meseret  
**Topic:** LoRA adapter serving — merged vs unmerged  
**Date:** 2026-05-04  
**Instructions:** Copy each tweet block exactly as written. Post in order 1 → 6.  
All tweets are ≤ 280 characters. No markdown — plain text + emojis only.

---

## Tweet 1 / 6 — Hook  
*(~235 chars)*

```
My pair @GashawBekele trained a LoRA adapter.
Loss dropped 3.08 → 0.42. Output length fell 18%.
Rubric scores? Identical — trained, baseline, and prompted.

Does how you SERVE a LoRA adapter change the output?

Merged vs unmerged is not just a memory choice. 🧵
```

---

## Tweet 2 / 6 — The Mechanism  
*(~220 chars)*

```
Two ways to serve a LoRA adapter at inference:

Merged:   W' = W + (α/r)·BA once → standard forward pass
Unmerged: h = Wx + (α/r)·BAx    → adapter runs live every call

Algebraically identical.
3 real conditions break that guarantee ↓
```

---

## Tweet 3 / 6 — The 3 Failure Conditions  
*(~265 chars)*

```
Where merged ≠ unmerged in practice:

① fp16 rounding — gap < 0.001 in logits, rarely changes top token
② Scaling bug — α/r applied inconsistently across code paths
③ model.eval() not called — dropout stays ON, adapter output goes random

③ is silent. ③ is the dangerous one.
```

---

## Tweet 4 / 6 — The Verification Check  
*(~245 chars)*

```
Paste this before shipping any LoRA adapter:

logits_u = peft_model(**inputs).logits
logits_m = peft_model.merge_and_unload()(**inputs).logits

(logits_m - logits_u).abs().max()
→ healthy if < 0.001

(logits_m.argmax(-1) == logits_u.argmax(-1)).all()
→ healthy if True
```

---

## Tweet 5 / 6 — The FDE Rule  
*(~250 chars)*

```
The FDE rule:

✅ Merge for production (same speed as base model)
🔄 Keep unmerged for A/B testing or multi-client hot-swap
⚠️ Always call model.eval() — non-negotiable in unmerged mode

Adapter appears to have no effect?
→ Check eval()
→ Check α/r scaling
→ Run the logit check above
```

---

## Tweet 6 / 6 — Honest Finding + Link  
*(~255 chars + blog URL)*

```
The honest answer: serving mode was NOT the cause of my Delta A = 0.00.

max_diff < 0.001 ✓
top-1 tokens matched ✓

Root cause: 0.5B backbone attention-copies banned phrases from input.
A capacity problem — not a serving bug.

Full explainer 👇
[ADD BLOG URL HERE]
```

---

## Pre-post checklist

- [ ] Replace `[ADD BLOG URL HERE]` in Tweet 6 with your actual blog post URL  
- [ ] Post Tweet 1 first, then reply to each tweet in the chain (1 → 2 → 3 → 4 → 5 → 6)  
- [ ] Do NOT add formatting symbols — no asterisks, no backticks, no markdown  
- [ ] Each tweet is a reply to the previous one, keeping the thread connected  
