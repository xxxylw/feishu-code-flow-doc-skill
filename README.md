# Feishu Doc Skill

![Feishu Doc Skill hero](assets/lark-code-flow-docs-hero.png)

A skill for composing Feishu/Lark Docx documents from chat conversations. It guides an AI agent to review the current conversation, auto-select document "bricks" based on content features, assemble them, and write the result to Feishu via `lark-cli`.

## What This Skill Does

User says "把刚才讨论的总结到飞书文档" (summarize our discussion to a Feishu doc). The skill:

1. Reviews the conversation content
2. Scans for content features (code, commands, decisions, formulas, flows, version info)
3. Selects matching bricks from a fixed set of 11
4. Assembles bricks in a stable reading order
5. Writes the document via `lark-cli docs --api-version v2`

**Core idea**: brick-based, not template-based. No fixed section structure — the document shape is driven by what the conversation actually contains.

## Brick System

The skill auto-selects from 11 bricks based on conversation features:

| Brick | Triggers when conversation has |
|-------|-------------------------------|
| Metadata header | Hardware / paths / project name / date (≥2) |
| One-line conclusion | Always |
| Flow whiteboard | Execution / data / call / state / control flow |
| Step log | Reproducible command/config/UI sequences |
| Decision table | Choice between options / parameter selection |
| Code evidence | Source code references (file:symbol) |
| Math derivation | Formulas / loss / gradients / proofs |
| Comparison table | Multi-object comparison (descriptive, not selection) |
| Version baseline | Environment / dependency versions |
| Risks & unverified | Inferences / assumptions / unverified claims |
| Action items | Next steps with owners and acceptance criteria |

See `references/doc-bricks.md` for full brick specifications.

## Preserved Capability: Code Flow Documents

The original `lark-code-flow-doc` behavior is preserved as one scenario recipe. When the conversation is about code, the skill follows the original core rule:

```text
Explain code by flow, not by file order.
```

Startup → build target → entry function → initialization → callback/loop → output.

The full preserved structure (reading scope, entry chain, data/state flow, key symbols table) is documented in the "Code flow" recipe in `references/scenario-recipes.md`.

## Scenario Recipes

Typical brick combinations for common scenarios:

| Scenario | Keywords | Bricks |
|----------|----------|--------|
| Code flow | "explain how code runs" | Header + conclusion + whiteboard + code evidence + risks |
| Build record | "record setup" | Header + conclusion + decision table + step log + version baseline + risks |
| Training journal | "training record" | Header + conclusion + decision table + step log + comparison table + version baseline + risks |
| Study notes | "study notes" "summarize paper" | Conclusion + math derivation + code evidence + whiteboard + risks |
| Tech decision | "decision discussion" | Conclusion + decision table + whiteboard + comparison table + action items + risks |
| Meeting notes | "meeting summary" | Conclusion + decision table + action items + risks |
| Troubleshooting | "debug recap" "postmortem" | Conclusion + step log + code evidence + decision table + risks + action items |

See `references/scenario-recipes.md` for details.

## UTF-8 Write Safety

For any content with non-ASCII text, write to a UTF-8 file and pass it by relative `@file` path:

```bash
lark-cli docs +update --api-version v2 --doc "<doc>" --command append --content "@tmp/payload.xml"
```

Do not pipe through PowerShell — it corrupts UTF-8 to `?`. After writing, fetch a fragment and verify. Inline `--content` is for ASCII-only debugging only.

## Repository Contents

```
SKILL.md                     ← Entry: trigger rules + brick selection + assembly order
references/
  doc-bricks.md              ← 11 brick specifications
  whiteboard-rules.md        ← Whiteboard strategy (when to draw, type selection, fallback)
  style-guide.md             ← Writing style (anti-patterns, callout discipline, language)
  scenario-recipes.md        ← Typical brick combinations per scenario
  feishu-elements.md         ← Feishu XML reference + UTF-8 write safety
agents/
  openai.yaml                ← Codex UI metadata
```

## Install Locally

```bash
git clone https://github.com/xxxylw/feishu-code-flow-doc-skill.git
```

Copy `SKILL.md` and `references/` into your AI agent's skills directory (e.g. `~/.codex/skills/feishu-doc/`).

## Example Prompts

```text
Summarize our discussion into a Feishu document.
```

```text
把刚才讨论的总结到飞书文档。
```

```text
Use this skill to explain how this ROS node starts, subscribes, processes data, and publishes output. Write it into a Feishu document with a whiteboard.
```

```text
Use this skill to document this training run — hyperparameter choices, commands, results, and risks.
```

## Notes

This skill defines the documentation standard (what to write and how to organize). Actual Feishu API operations rely on the existing `lark-doc` and `lark-whiteboard` skills plus `lark-cli`.
