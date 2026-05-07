# signoff.md — Day 3

**Asker:** Gashaw Bekele
**Explainer:** Nebiyou Abebe
**Date:** 2026-05-07

---

## Gap Closure Judgment

**Status: CLOSED**

---

## What I Understand Now That I Did Not Before

My question was: are length reduction and token suppression expressible by the
same class of low-rank weight update, or does suppressing a token require a
fundamentally different transformation?

Before Nebiyou's explainer I had one hypothesis — rank capacity — and I assumed
it was probably the right answer. My methodology_rationale.md used the phrase
"capacity limitation of the backbone" without distinguishing between rank capacity
and objective capacity. These are different things and I was conflating them.

After the explainer I can now state the following with confidence:

**The primary cause of the asymmetry is the training objective, not the rank.**
Standard SFT is a positive-only imitation objective: it increases the probability
of chosen tokens but provides no gradient signal that explicitly lowers the
probability of rejected tokens. ORPO (Hong et al., 2024) documents this precisely:
in pilot experiments, SFT on chosen responses *increased* the log-probability of
both chosen and rejected responses together. Welleck et al. (2019) identified the
same problem and proposed unlikelihood training to add an explicit negative signal.

Length reduction transfers from SFT because it is a dense, consistent, many-token
preference — the model sees hundreds of positions in 221 examples all pointing
toward shorter completions. Token suppression is sparse and local: "when this exact
token is likely in context, do not choose it." That negative constraint has no
direct gradient path in a cross-entropy objective.

**The experiment.py proof is decisive.** When the model was *instructed* to use
the banned word and a logit_bias=-100 block was active, it could not produce the
word across 5/5 trials. Without the block, it produced it 5/5 times despite identical
instructions. This shows that probability-shifting (SFT, prompt instruction) and
token-blocking (logit_bias, bad_words_ids) are mechanically different classes of
enforcement — one nudges, the other prohibits. My SFT training was the nudge class.

**What this changes in my v0.2 plan:**
The fix is not simply a larger backbone. The fix is either (a) ORPO or DPO with
rejected responses that contain the banned phrase, giving an explicit negative
gradient signal, or (b) decoding-layer enforcement via bad_words_ids regardless of
what the adapter learned. I now know which to try first and why.
