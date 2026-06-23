---
name: feishu-doc
description: Compose Feishu/Lark Docx documents from a chat conversation. Use when the user asks to summarize, write up, or push the recent discussion to Feishu — covers code-flow explanations, build records, training journals, study notes with math, and tech-decision memos. Auto-selects from a fixed set of bricks based on conversation content. Calls lark-doc / lark-whiteboard for actual API operations.
---

# Feishu Doc

Compose a Feishu Docx document from the current conversation. When the user says "把刚才讨论的总结到飞书文档", the skill reviews the conversation, auto-selects bricks, assembles them, and writes the result via lark-cli.

**Core idea**: brick-based, not template-based. No fixed section structure — the document shape is driven by what the conversation actually contains.

---

## Trigger Rules

### Explicit trigger

The user message contains one of:
- "总结到飞书", "写到飞书", "写飞书文档", "飞书文档总结"
- "把刚才/刚刚/前面的（讨论/工作/内容）写文档"
- "更新飞书文档 `<doc-id>`", "追加到飞书文档 `<doc-id>`"
- "summarize to feishu", "write a feishu doc"

### Implicit trigger

Requires a prior explicit agreement in the session that the Feishu document is the target artifact, AND the conversation contains a solidifiable result (code / command / decision / formula). Without prior agreement, do not implicitly trigger.

### Do not trigger

- User is asking a concept ("what is X")
- Conversation is still exploratory, no solidifiable conclusion
- User specified a non-Feishu output ("write README", "put in spreadsheet")
- User is requesting an operation ("run it", "debug this")

---

## Brick List (12)

| # | Brick | Trigger feature |
|---|-------|-----------------|
| 1 | Metadata header | Hardware / paths / project name / date present (≥2) |
| 2 | One-line conclusion | **Always** |
| 3 | Flow whiteboard | Execution / data / call / state / control flow, or architecture relationships |
| 4 | Step log | Reproducible operation sequence with commands, config changes, or UI paths |
| 5 | Decision table | Choice between options / parameter selection / version switch |
| 6 | Code evidence | Source code references (file path / function / class name) |
| 7 | Math derivation | Formulas / loss / gradients / probability / algorithm proofs |
| 8 | Comparison table | Multi-object comparison (descriptive, not selection) |
| 9 | Version baseline | Environment / dependency versions |
| 10 | Risks & unverified | Inferences / assumptions / unverified claims / missing dependencies |
| 11 | Action items | Next steps with owners and acceptance criteria |
| 12 | Worked example | Concrete numerical walkthrough with actual numbers plugged in |

Full specifications in [`references/doc-bricks.md`](references/doc-bricks.md).

---

## Brick Selection Logic

Scan the conversation for content features. Each hit independently adds a brick. All are stackable:

```
References file/function/class?           → +Code evidence
Reproducible command/config/UI sequence?  → +Step log
"Choose X over Y / change A to B"?        → +Decision table
Formula / loss / gradient / derivation?   → +Math derivation
Concrete numbers plugged into a formula?  → +Worked example
Execution/data/call flow or architecture? → +Flow whiteboard
CUDA/Python/package versions?             → +Version baseline
Multi-object comparison (not selection)?  → +Comparison table
Inference / unverified / assumption?      → +Risks & unverified
Next steps with owner/deadline?           → +Action items
Hardware/path/project/date (≥2)?          → +Metadata header
(always)                                  → +One-line conclusion
```

**Feature-driven, not scenario-driven**: only check whether the conversation contains the corresponding feature. The same "training record" may or may not contain math depending on actual content.

**Split build records from concept notes**: when a conversation contains both reproducible operations (commands, configs, results) AND Q&A-style explanations ("why this parameter", "what does this concept mean"), produce TWO documents — a build record (step log + decision table + version baseline) and a separate concept notes document (flow whiteboard + math derivation + code evidence). Mixing them in one doc makes the build record too long and the explanations too sparse.

**Fallback**: if nothing hits besides "One-line conclusion", the conversation is insufficient to summarize. Return to the user and suggest a more specific output.

**Complexity guard**: a typical document should have the one-line conclusion plus a few core bricks plus supporting bricks. If nearly every brick hits, the conversation is likely too broad — consider splitting into multiple documents, or ask the user to scope it down.

---

## Assembly Order

Fixed pyramid order:

```
1. Metadata header (if hit)
2. One-line conclusion (always)
3. Flow whiteboard (if hit; placed early to anchor the reader's mental model)
4. Core bricks (in the conversation's logical order):
   Decision table / Step log / Code evidence / Math derivation / Worked example
5. Reference bricks (static reference data):
   Version baseline / Comparison table
6. Action items (if hit)
7. Risks & unverified (if hit, always last)
```

The relative order of core bricks follows the conversation's logical flow:
- Study notes: math derivation → code implementation
- Build records: decision table → step log
- Code flow: flow whiteboard → code evidence (in runtime order)

---

## Scenario Recipes

Typical brick combinations for common scenarios. These provide weighted confirmation of the selection result. Not mandatory templates.

| Scenario | Keywords | Brick combination |
|----------|----------|-------------------|
| Code flow doc | "explain how code runs" | Header + conclusion + whiteboard + code evidence (runtime order) + risks |
| Build record | "record setup" | Header + conclusion + whiteboard (if complex) + decision table + step log + version baseline + risks |
| Study notes | "study notes" "summarize paper" | Conclusion + math derivation + worked example (if any) + code evidence (if any) + whiteboard (if complex) + risks |
| Tech decision | "decision discussion" | Conclusion + decision table + whiteboard (architecture) + comparison table + action items + risks |
| Meeting notes | "meeting summary" | Conclusion + decision table + action items + risks |
| Troubleshooting | "debug recap" "postmortem" | Conclusion + step log + code evidence (error location) + decision table (fix candidates) + risks |

Details in [`references/scenario-recipes.md`](references/scenario-recipes.md).

---

## Build Record vs Concept Notes Split

## Split build records from concept notes: when a project has both **operational steps** (what was done, with commands and data) AND **conceptual explanations** (why an approach works, with diagrams and derivations), split into two documents:

- **Build Record** (primary): Step log + decision table + version baseline + risks. Pure facts and numbers. No concept explanations.
- **Concept Notes** (secondary): Flow whiteboard + math derivation + code evidence + comparison table. Explains *why* things work, with diagrams.

**Why split**: Build records are consulted for "what version / what command / what result". Concept notes are consulted for "why does this work / how does this algorithm behave". Mixing both in one document makes neither purpose clean.

**Concept notes can grow incrementally**: when a user asks to "add explanation of concept X to the notes", append a new `<h1>` section to the existing concept notes document. Each concept is self-contained (a heading + tables/formulas/diagrams), so ordering doesn't matter. Use `<h1>` numbering (1, 2, 3...) and update the one-line conclusion to reflect the total count.

**Mermaid whiteboard color pitfalls** (tested 2026-06-22): Feishu's mermaid renderer renders `classDef`, `linkStyle`, `fill`, and `stroke` directives as **visible raw text** in the document body — users see garbage lines like `classDef default fill:#e8f0fe,stroke:#1a73e8...` appearing as paragraph content. **Never put classDef/linkStyle inside a mermaid whiteboard.** If color matters, use `svg` inline — do not attempt mermaid styling even as a first try. When the user reports "画板颜色看不清/有问题", switch the whiteboard type from `mermaid` to `svg` with the One Dark palette. Keep all content the same — only change the rendering type.
- 4 build record docs (one per phase: env setup / data engineering / SFT / GRPO)
- 1 concept notes doc (17 concepts: completion-only masking, image token formula, GRPO flow, reward components, vLLM architecture, etc.)

Build records are consulted for "what version / what command / what result". Concept notes are consulted for "why does this work / how does this algorithm behave". Mixing both makes neither clean.

---

## Preserved Capability: Code Flow Documents

When the conversation is about code, the skill follows the original core principle from `lark-code-flow-doc`:

```
Explain code by flow, not by file order.
```

Organize by runtime path, not by file order:

```
startup → build target → entry function → initialization → callback/loop → output
```

The whiteboard must come before code evidence to give the reader the global view first. Code evidence follows in runtime order, each segment with file:symbol + Next pointer. See "Code flow" recipe in `references/scenario-recipes.md` for the full preserved structure.

---

## Downstream Skill Collaboration

This skill only defines "what to write and how to organize". Feishu API calls use existing skills:

- **Create/update document**: `lark-doc` skill, `docs +create` / `docs +update --api-version v2`
- **Create/update whiteboard**: `lark-whiteboard` skill
- **Existing title protection**: after fetch, if a `<title>` exists, do not overwrite — only append body content
- **UTF-8 write safety**: for non-ASCII content, always use `@file` relative path, never PowerShell pipe

Feishu XML syntax and command reference in [`references/feishu-elements.md`](references/feishu-elements.md).

---

## Workflow

0. **Process image source material**: if the user provides PPT screenshots, PDF slides, or other image-based source material, read them via the `codex` skill (`ask_codex.sh -i image.png`), NOT `vision_analyze` or subagents. For PDFs, render pages to PNG first (e.g. PyMuPDF at 200 DPI), then send to codex for transcription. This is a hard user preference.
1. **Check trigger**: apply trigger rules; if not triggered, stop
2. **Review conversation**: scan content features, determine which bricks hit
3. **Select bricks**: stack bricks per the feature checklist
4. **Order**: arrange bricks per assembly order
5. **Read references**: for hit bricks, read the corresponding section in `doc-bricks.md`; for whiteboards, read `whiteboard-rules.md`; check `style-guide.md` for style rules
6. **Generate XML**: use correct syntax from `feishu-elements.md`
7. **Write to Feishu**: create or update the document via lark-cli. Use `@file` relative path for any non-ASCII content (UTF-8 safety).
8. **Verify**: fetch written fragments, check title, whiteboard, code blocks, tables, and CJK text are correct

## Whiteboard SVG Construction Tips (learned from production, 2026-06-22)

When constructing SVG whiteboards by hand:

- **Calculate viewBox from coordinates**: The viewBox must encompass all elements. For a 6-node vertical flowchart with 74px spacing between 44px-tall nodes: height = 6×44 + 5×30 (gaps) + margins ≈ 470. Width = max node width (220) + 2×40 margin ≈ 300.
- **Arrow loop-back**: To draw a return arrow (e.g. "next prompt" loop in GRPO), use a `<path>` with quadratic bezier: `d="M startX startY Q controlX controlY endX endY"`. Set `stroke-dasharray="4,3"` for dashed lines.
- **Node y-positions**: For n nodes with node_height=44 and gap=30: node[i].y = 10 + i×74. Arrow between nodes: y1 = node[i].y + 44, y2 = node[i+1].y.
- **Horizontal split layout** (e.g. vLLM colocate memory diagram): Use a dashed `<rect>` as container border, two `<rect>` nodes side by side with a connecting `<line>` between them.
- **Keep labels short**: Node text should be action + key metric only (e.g. "vLLM 推理侧 约 11GB"). Move detailed breakdowns to the mapping table below the SVG.

## Whiteboard Critical Pitfall

**Never put `classDef`, `linkStyle`, or any mermaid style directives inside a `<whiteboard type="mermaid">`.** Feishu does not silently strip them — it renders them as **visible raw text** in the document body, polluting the output with garbage lines like `classDef default fill:#e8f0fe...`. If you need colored diagrams, use `<whiteboard type="svg">` with the One Dark palette from `whiteboard-rules.md`. This pitfall was confirmed in production on 2026-06-22.

## Visual-Only Edits: Do Not Rewrite Content

When the user asks to change ONLY a visual property of an existing whiteboard (colors, layout, arrow style, node positions), do NOT also rewrite the diagram content (labels, node text, structure). Change only the requested visual property.

**Why**: The user has already reviewed and approved the content. Silently rewriting labels or restructuring nodes while "fixing colors" forces them to re-review everything and erodes trust. They asked for a paint job, not a remodel.

**Pattern to follow**:
1. User: "画板颜色不对，换一下" → Change ONLY the SVG fill/stroke/marker colors, keep all labels identical
2. User: "画板配色看不清箭头" → Change ONLY the arrow color or background contrast
3. User: "这个流程图改成横向" → Change ONLY the layout direction

**Anti-pattern**: User says "change colors" → Agent also simplifies node labels, removes nodes, replaces mermaid with ASCII art, changes the entire diagram. Never do this.

## Document Splitting: Build Records vs Concept Notes

## Split build records from concept notes: when a project has both **operational steps** (what was done, with commands and data) AND **conceptual explanations** (why an approach works, with diagrams and derivations), split into two documents:

- **Build Record** (primary): Step log + decision table + version baseline + risks. Pure facts and numbers. No concept explanations.
- **Concept Notes** (secondary): Flow whiteboard + math derivation + code evidence + comparison table. Explains *why* things work, with diagrams.

## Concept Notes: Migrate from Prior Build Records

When creating a standalone Concept Notes document, scan prior build records (local draft XML files from earlier sessions) for technical explanations that were removed during build-record cleanup. These are fragments that the user originally wrote but were stripped from the build record because they were "why" explanations, not "what was done" steps.

**Why migrate**: the user explicitly asked "之前的疑惑都迁移过来了吗" — they want a single reference document where all conceptual explanations live. Missing prior explanations means the user has to point them out manually.

**Process**:
1. Read all local draft XMLs from previous sessions (e.g. `feishu/p0_draft.xml`, `feishu/p1_draft.xml`)
2. Extract "Why" blocks, "原理" explanations, derivation chains, and concept definitions
3. Add them as numbered sections in the Concept Notes document
4. The user will review and confirm coverage

## Concept Notes: Reward Component Explanations

When a project involves a multi-component reward function (common in RL/GRPO work), the Concept Notes document should include a section explaining each reward component — not just the formula or the routing table. For each component:

1. What it does (one sentence)
2. Why it's needed (the failure mode it prevents or the signal it measures)
3. Key implementation detail (e.g., "Levenshtein ratio for partial match" — the specific technique, not just "similarity score")

This level of detail is what makes the document useful for interview review. A routing table without per-component explanation is insufficient.

**Why split**: Build records are consulted for "what version / what command / what result". Concept notes are consulted for "why does this work / how does this algorithm behave". Mixing both in one document makes neither purpose clean.

**Concept notes can grow incrementally**: when a user asks to "add explanation of concept X to the notes", append a new `<h1>` section to the existing concept notes document. Each concept is self-contained (a heading + tables/formulas/diagrams), so ordering doesn't matter. Use `<h1>` numbering (1, 2, 3...) and update the one-line conclusion to reflect the total count.

**Mermaid whiteboard color pitfalls** (tested 2026-06-22): Feishu's mermaid renderer renders `classDef`, `linkStyle`, `fill`, and `stroke` directives as **visible raw text** in the document body — users see garbage lines like `classDef default fill:#e8f0fe,stroke:#1a73e8...` appearing as paragraph content. **Never put classDef/linkStyle inside a mermaid whiteboard.** If color matters, use `svg` inline — do not attempt mermaid styling even as a first try. When the user reports "画板颜色看不清/有问题", switch the whiteboard type from `mermaid` to `svg` with the One Dark palette. Keep all content the same — only change the rendering type.

## Callout Discipline

Color semantics:
- Blue 📝: convention / rule / definition / project anchor
- Green ✅: locked conclusion / milestone
- Yellow ⚠️: warning / side-effect
- Red ❗: blocker / known issue

Callouts are for emphasis that genuinely benefits from a visual container. Do not deliberately ration them. The "why" in a step log is always a plain `<p>` paragraph, not a callout. Do not use callouts for decoration or filler summaries.

See `references/style-guide.md` § Tables vs Lists for when to use a table vs a list.

---

## References Index

| File | Responsibility |
|------|----------------|
| [`references/doc-bricks.md`](references/doc-bricks.md) | Full specifications for all 12 bricks (trigger / content / Feishu element / good-bad examples) |
| [`references/whiteboard-rules.md`](references/whiteboard-rules.md) | Whiteboard strategy (three-tier judgement + type selection + SVG One Dark color scheme) |
| [`references/style-guide.md`](references/style-guide.md) | Style rules (syntax / anti-patterns / callout discipline / tables vs lists / CJK-English mixing) |
| [`references/scenario-recipes.md`](references/scenario-recipes.md) | Scenario recipes (typical brick combinations) |
| [`references/feishu-elements.md`](references/feishu-elements.md) | Feishu XML reference + UTF-8 write safety |
