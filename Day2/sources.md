# Sources — Charlie Lijalem's Explainer for Gashaw Bekele
## Week 12 | Agent & Tool-Use Internals

---

## Canonical Sources Used in explainer.md

**[1] Primary Paper — ReAct**
Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023).
*ReAct: Synergizing Reasoning and Acting in Language Models.*
International Conference on Learning Representations (ICLR 2023).
https://arxiv.org/abs/2210.03629

Role in explainer: Establishes that tool selection is model reasoning over descriptions —
not a separate routing system. The ReAct framework makes the reasoning trace before tool
selection visible, proving Layer 1 (which tool) is language modeling over tool definitions.

---

**[2] Primary Paper — Toolformer**
Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Zettlemoyer, L.,
Cancedda, N., & Scialom, T. (2023).
*Toolformer: Language Models Can Teach Themselves to Use Tools.*
Advances in Neural Information Processing Systems (NeurIPS 2023).
https://arxiv.org/abs/2302.04761

Role in explainer: Shows that tool selection behavior emerges from fine-tuning on examples
where tool calls appear in context — not from explicit selection supervision. Grounds the
claim that descriptions drive selection through learned semantic matching.

---

**[3] Measurement Source — ToolScope**
Liu et al. (2025). *ToolScope.*
https://arxiv.org/abs/2510.20036

Role in explainer: Provides the quantitative claim — 8.8–38.6% accuracy degradation when
tool descriptions overlap. This is the empirical basis for treating overlapping descriptions
as a correctness bug, not a tuning preference.

---

## Tool Used

Three-component description pattern applied to `query_agent.py` tool descriptions
(trigger + examples + anti-trigger), then verified against 6 test queries covering the
`semantic_search` / `structured_query` overlap boundary. See `grounding_commit.md` for
full before/after and test results.
