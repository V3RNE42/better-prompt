# Better Prompt

**Structure beats model. Every time.**

Validated by HELIOS ([arxiv 2601.14598](https://arxiv.org/abs/2601.14598)): switching from raw text to hierarchical structured input took Gemini 2.0-Flash from **45% → 94.9%** task success — no model change, no fine-tuning.

---

## The Recipe

Three ingredients, one pattern:

**1. Hierarchy** — Organize top-down: overview → components → details  
**2. Relationships** — Say *how* things connect, don't just list them  
**3. Feedback loop** — Ask the LLM to verify its own answer

That's it.

```
# Before — flat dump
Here is my service code: [500 lines]
Why is it failing?

# After — structured
System: order management API
Components: OrderController → OrderService → Repository → DB
Problem: OrderService.createOrder() — silent failure on duplicate keys
Relevant code: [20 lines scoped to the issue]
```

---

## Quick Fixes

| Symptom | What's missing |
|---------|----------------|
| Vague answers | **Hierarchy** — the LLM doesn't know what matters |
| Misses edge cases | **Relationships** — connect the parts |
| Shallow responses | **Feedback loop** — challenge its own answer |
| Rambling | **Scope** — include only what's relevant |

---

## Common Pitfalls

- **Dumping more text** — unstructured volume makes things *worse*, not better
- **Hierarchy without connections** — levels alone aren't enough
- **Skipping verification** — a feedback loop is the cheapest performance gain you'll ever get

---

## Also in this repo

- [`SKILL.md`](SKILL.md) — full guide with examples and format selection
- [`references/llm-input-formats.md`](references/llm-input-formats.md) — which serialization to use (YAML, TOON, TRON, JSON) and when

## License

MIT — do what you want, attribution appreciated.

## Source

HELIOS paper [arxiv 2601.14598](https://arxiv.org/abs/2601.14598) — hierarchical CFG/call-graph structured input for LLM decompilation.

Core finding: structure of input > model choice > prompt length
