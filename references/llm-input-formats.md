# LLM Input Format Selection Guide

Which serialization format to use when structuring LLM prompts, context, and subagent handoffs.

## Decision Framework

```
Q1: Is this data for LLM consumption?
  ├─ YES → Q2
  └─ NO  → JSON (public API) or YAML (config/human edit)

Q2: Is the data flat/tabular/row-based?
  ├─ YES → TOON (max token efficiency for LLMs)
  └─ NO  → Q3

Q3: Do you need JSON compatibility?
  ├─ YES → TRON (gradual migration from existing JSON workflows)
  └─ NO  → Q4

Q4: Does the handoff mix instructions + structured data?
  ├─ YES → YAML (best for mixed content, comments supported, ~15% token savings vs JSON)
  └─ NO  → TOON (pure data, no prose)

Q5: Is readability more important than token count?
  ├─ YES → YAML
  └─ NO  → TOON or TRON
```

## Format Quick Reference

| Scenario | Best Format | Why |
|----------|-------------|-----|
| Subagent handoff (instructions + context) | **YAML** | Mixed instruction+data, comments, hierarchy, ~15% fewer tokens |
| Flat/tabular data (lists, users, items) | **TOON** | ~47-50% token savings vs JSON |
| RAG document chunks with metadata | **TRON** | Nested class defs, JSON-toolchain compatible |
| Function/tool schemas | **TOON** | 32% more compact than JSON |
| Few-shot examples | **TOON** | 31% more examples per context budget |
| Public API output | **JSON** | Universal standard, every tool supports it |
| Config files (human-edited) | **YAML** | Comments, clean syntax, best readability |
| Time series / logs | **CSV** or **TOON** | CSV max efficiency, TOON if structure matters |
| Deeply nested data + LLM | **TRON** | Nested class definitions, JSON compatibility |
| Context-window-constrained | **TOON** | 4× more data fits in 128K vs JSON |

## Cost Impact (GPT-4 pricing, 10K records)

| Format | Savings vs JSON |
|--------|----------------|
| TOON | 75% |
| TRON | 60% |
| YAML | 41% |
| JSON | baseline |

## Average Token Savings vs JSON (14 benchmark scenarios)

- **TOON:** ~35% across all scenarios, up to 47% on flat data
- **TRON:** ~31% across all scenarios
- **YAML:** ~14-16% consistent savings on flat and nested

## When Each Format Breaks Down

| Format | Weakness |
|--------|----------|
| TOON | Deeply nested data (becomes hard to read, loses efficiency) |
| TRON | Non-uniform objects (class definitions don't help) |
| JSON | Token overhead (quotes, braces, commas add up fast) |
| YAML | Indentation-sensitive. No machine-parseable strict mode for LLM output |
| CSV | Flat data only. No nesting. No null/empty handling in standard form |

## Subagent Handoff: Preferred Format = YAML

When passing context to a `delegate_task` subagent (following better-prompt's hierarchy + relationships + feedback loop), **use YAML**:

```yaml
# HANDOFF: Execution phase
goal: Scrape phone numbers from 30 Andalusian schools
source: Junta CSV

components:
  input: /root/centros_andalucia_2024_25.csv
  tools: [curl, observa, phonenumbers]

relationships:
  CSV → url_list → scrape_web → extract_phone → validate → output
  fallback: JSON-LD → grep → headless_browser

# Feedback loop
feedback: >
  When done, verify each phone matches +34 + 9 digits.
  Report how many were found per method.
```

**Why YAML wins for handoffs:**
- Natural indentation expresses better-prompt hierarchy (overview → components → details)
- Comments (`#`) annotate sections without consuming parseable tokens
- Mixed instruction text + structured data reads naturally
- ~15% fewer tokens than equivalent JSON
- Subagent LLMs parse it as reliably as JSON

**Use TOON** for handoffs only when the context is **pure tabular data** (e.g., pass 500 rows to process). For anything with instructions, relationships, and feedback — YAML.

## Sources

- [TOON vs JSON vs YAML: Token Efficiency Breakdown for LLM](https://medium.com/@ffkalapurackal/toon-vs-json-vs-yaml-token-efficiency-breakdown-for-llm-5d3e5dc9fb9c) — Frason Francis (Nov 2025)
- [TOON vs TRON vs JSON vs YAML vs CSV for LLM Apps](https://www.piotr-sikora.com/blog/2025-12-05-toon-tron-csv-yaml-json-format-comparison) — Piotr Sikora (Dec 2025). 14 test scenarios, full benchmarks.
