# Style Guide

---

## Syntax

- **Short sentences**: one technical point per sentence. Long sentences must be split.
- **Numbers over adjectives**: write "16.52M / 0.679%", not "very few parameters"; write "7.4 GB difference", not "significant VRAM gap".
- **Plain language over jargon stacks**: "install package" not "execute dependency deployment"; "runs through" not "end-to-end pipeline validation".

**Accuracy caveat**: preserve precise technical terms on first appearance; use plain language in the explanation sentence. Do not sacrifice accuracy for brevity.

---

## Anti-patterns (prohibited)

### Academic tone

Prohibited: "first principles", "paradigm shift", "systematically", "in a holistic manner", "on the basis of", "is of great significance".

Prohibited preamble: "This document will introduce...", "In summary...", "It can be seen that...".

Substitute: state facts and numbers directly.

**Technical-term exception**: terms like "end-to-end" that have a precise technical meaning in context are allowed. What is prohibited is using them as empty modifiers ("end-to-end empowerment"). If the term carries real meaning, keep it.

### Adjective stacking

Prohibited: "very", "extremely", "significant", "greatly" — unless immediately followed by a specific number.

### Trivia logging

Do not write: clearing temp files, renaming files, tweaking editor config, mood/reflections/"lessons learned".

Do write: facts that affect subsequent behavior.

Judgment principle: would a future reader trying to reproduce or troubleshoot get stuck without this line? If yes, write it.

---
## Callout Discipline

Color semantics are fixed:

| Color | Meaning |
|-------|---------|
| Blue | Convention / rule / definition / project anchor |
| Green | Locked conclusion / milestone |
| Yellow | Warning / side-effect |
| Red | Blocker / known issue |

Callouts are for emphasis that genuinely benefits from a visual container — conclusions, warnings, project anchors, reusable rules. Do not deliberately ration them; if a document needs 4 callouts because there are 4 items worth emphasizing, use 4.

The "why" in a step log is always a normal paragraph, not a callout. Do not use callouts for decoration or filler summaries.

---

## Required Elements

- **Numbers + units**: required when the content involves performance, time, VRAM, parameter count, or version numbers. Not required for pure concept notes or meeting minutes.
- **Status labels**: Verified / Inferred / Unverified, when the content involves code behavior or verification status.
- **Reasons**: decisions, changes, and parameter choices must include a one-sentence "why".
- **Evidence pointers**: cite code as `file.cpp::Class.method`, not "in some file".

---

## CJK-English Mixing

This section applies when the document content is primarily Chinese (the common case for Feishu notes).

- Technical terms stay in English: API, callback, loss, CUDA, LoRA, epoch, batch size
- Paths, file names, and commands stay in English
- Descriptive prose is in Chinese
- Do not force-translate established English terms (write "BatchNorm", not "批归一化")

---

## Tables vs Lists

Tables and lists are both ways to present a set of structured items. Choose based on data shape, not habit.

**Use a table when** each item has multiple attributes that need to be compared side by side — i.e., the content is two-dimensional (rows × columns). Examples: version comparison across components, hyperparameter before/after, decision candidates with metrics.

**Use a list when** the content is one-dimensional — a sequence of steps, a collection of properties, or a set of rules. Examples: node label conventions, construction rules, setup steps, risks items.

**Decision rule**: if each item has only one value to state, use a list. If each item has two or more values that need to be scanned together, use a table. When unsure, try the list first — it is almost always more readable and takes less space.

**Don't use a table to present a single column of values** — that is a list.

---

## What Not to Write

- No "development reflections" or "project significance"
- No scattered "next steps" in the body text — when next steps are needed, consolidate them into the Action Items brick or the Risks brick's verification method
- No fabricated dialogue (do not invent arguments that did not happen)
- No speculation beyond what the current session has confirmed (speculation must be labeled Inferred)
- No non-technical framing (interview prep, resume language, etc.) — keep technical documents focused on technical substance
