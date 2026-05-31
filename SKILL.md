---
name: better-prompt
description: Use when asked to improve, rewrite, or strengthen a prompt for any LLM task, be it for yourself or for invoking a subagent. Also use when a prompt is producing shallow, incomplete, or structurally weak responses.
---

# Better Prompt — Structured Input Pattern

## Overview

The biggest lever on LLM output quality is **how you structure the input**, not which model you use or how long the prompt is.

Validated by HELIOS (arxiv 2601.14598): switching from raw text dumps to hierarchical structured input took Gemini 2.0-Flash from 45% → 94.9% task success and GPT-4.1-Mini from 71.4% → 96.5% — **with no model change, no fine-tuning**.

The same principle applies to any prompt, not just code analysis.

---

## The Core Pattern

**Raw dump → Hierarchical structure + explicit relationships + feedback loop**

### Step 1 — Add hierarchy

Organize context top-down: overview → components → details. The structure itself carries semantic meaning the LLM uses for reasoning.

```
# BAD — flat raw dump
Here is my service code: [500 lines pasted]
Why is it failing?

# GOOD — hierarchical
System: order management API
Components: OrderController → OrderService → OrderRepository → DB
Flow: POST /orders → validate → persist → notify
Problem area: OrderService.createOrder() — silent failure on duplicate keys
Relevant code: [20 lines, scoped to the problem]
Expected: DuplicateOrderException
Actual: returns null
```

### Step 2 — Make relationships explicit

Don't just list things — state how they connect, depend on, or affect each other.

```
# BAD
We have users, orders, and payments.

# GOOD
Users place Orders (1:N). Orders trigger Payments (1:1).
A failed Payment must roll back the Order status — that link is the bug surface.
```

### Step 3 — Add a feedback loop

Ask the LLM to verify or stress-test its own answer. This mirrors HELIOS's feedback pass (+~2–5% on top of structured input).

```
After answering, check: does your solution break if the order was already partially fulfilled?
```

---

## Quick Reference

| Symptom | Fix |
|---------|-----|
| Vague / generic answer | Add hierarchy — the LLM doesn't know what matters |
| Missing edge cases | Add explicit relationships — connect the parts |
| Correct but shallow | Add feedback loop — ask it to challenge its own answer |
| Too long, unfocused | Scope the context — include only the relevant layer |
| Wrong abstraction level | State the level explicitly: "focus on the service layer, not the DB" |

---

## When Improving Someone Else's Prompt

1. Identify what's a raw dump → restructure it into levels
2. Identify missing relationships → add explicit connectors
3. Identify missing scope → trim to what's relevant to the question
4. Append a feedback instruction → "after answering, verify that..."

---

## Format Selection — What Serialization to Use

The way you serialize structured content matters for both token efficiency and subagent parsing:

- **Subagent handoffs (instructions + context):** Use **YAML**. Natural hierarchy via indentation, comments supported, ~15% fewer tokens than JSON. See `references/llm-input-formats.md` for the full decision framework.
- **Pure data (flat/tabular):** Use **TOON** for ~47% token savings vs JSON.
- **JSON-compatible compression:** Use **TRON** (JSON superset with class definitions).
- **Public APIs:** Use **JSON** — universal standard.
- **Human-edited configs:** Use **YAML** — comments, clean syntax.

The key insight from 14 benchmark scenarios: format choice + hierarchical structure are independent levers. Even TOON data benefits from being organized overview→components→details.

## Common Mistakes

- **More context ≠ better** — unstructured volume hurts. Scope ruthlessly.
- **Hierarchy without relationships** — listing levels is not enough; connect them.
- **Skipping the feedback loop** — it's a cheap addition with consistent upside.
