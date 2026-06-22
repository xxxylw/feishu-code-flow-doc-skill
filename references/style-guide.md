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

- **At most 2 per document**
- Color semantics are fixed:

| Color | Meaning | Limit |
|-------|---------|-------|
| Blue | Title block / project anchor | ≤1 |
| Green | Locked conclusion / milestone | ≤1 |
| Yellow | Warning / side-effect | As needed, 0-1 |
| Red | Blocker / known issue | As needed, 0-1 |

- The total cap (≤2) takes priority over individual color limits.
- The "why" in a step log is a normal paragraph, not a callout.
- Do not use callouts for decoration or summaries.

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

## What Not to Write

- No "development reflections" or "project significance"
- No scattered "next steps" in the body text — when next steps are needed, consolidate them into the Action Items brick or the Risks brick's verification method
- No fabricated dialogue (do not invent arguments that did not happen)
- No speculation beyond what the current session has confirmed (speculation must be labeled Inferred)
- No non-technical framing (interview prep, resume language, etc.) — keep technical documents focused on technical substance
