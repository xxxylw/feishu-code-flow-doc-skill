# Scenario Recipes

Recipes are "suggested combinations", not mandatory templates. The AI still independently judges each brick per the feature checklist in `SKILL.md`. Recipes only provide weighted confirmation when the selection result matches a typical scenario.

---

## Recipe 1: Code Flow Document

**Keywords**: "explain how code runs", "code flow", "analyze this module"

**Bricks**:

1. Metadata header (if hardware/paths are involved)
2. One-line conclusion
3. Flow whiteboard (main execution flow, placed at top)
4. Code evidence (in runtime order, each segment with file:symbol + Next)
5. Risks & unverified

**Preserved structure from lark-code-flow-doc**: the following sub-structures should appear within the code evidence brick when applicable:

- **Reading scope**: what paths were inspected, what was excluded (vendor/generated/build dirs)
- **Entry chain**: a table from startup command → build declaration → executable target → main function → first business call
- **Data/state flow**: a table showing where data comes from, how it transforms, where it goes
- **Key symbols table**: important classes/functions/config files and their roles

**Special notes**:
- Core principle: explain code by flow, not by file order
- Whiteboard must come before code evidence to anchor the reader's mental model
- Code evidence follows runtime path: startup → build → entry → init → callback/loop → output

---

## Recipe 2: Build Record

**Keywords**: "record setup", "environment build", "整理搭建记录"

**Bricks**:

1. Metadata header
2. One-line conclusion
3. Flow whiteboard (only when steps have branches or stage transitions)
4. Decision table (if there were selections)
5. Step log (core: goal → command → verify → why)
6. Version baseline
7. Risks & unverified

**Special notes**:
- Step log is the main body; each step must have a command/config change and verification
- "Why" is one sentence, not expanded
- Version baseline comes after the step log as an environment snapshot

---

## Recipe 3: Training Journal

**Keywords**: "training record", "training log", "训练记录"

**Bricks**:

1. Metadata header
2. One-line conclusion
3. Decision table (hyperparameter selection)
4. Step log (training command + monitoring)
5. Comparison table (vs previous version)
6. Version baseline
7. Risks & unverified

**Special notes**:
- Hyperparameter decision table must include "why this value was changed"
- Comparison table records old vs new differences
- No mood/reflections — only facts and numbers

---

## Recipe 4: Study Notes

**Keywords**: "study notes", "summarize paper", "学习笔记"

**Bricks**:

1. One-line conclusion (what problem this algorithm/concept solves)
2. Math derivation (intuition first, then derivation, each step with "why")
3. Code evidence (if an implementation exists)
4. Flow whiteboard (only when algorithm steps have branches or state transitions)
5. Risks & unverified (label which parts are Inferred)

**Special notes**:
- Math derivation must have prose bridging between formulas — no formula walls
- Write the intuitive explanation first (plain language), then the formal derivation
- Common mistakes may use a yellow callout (respect the ≤2 callout limit)

---

## Recipe 5: Tech Decision

**Keywords**: "decision discussion", "architecture selection", "技术调研"

**Bricks**:

1. One-line conclusion
2. Decision table (candidate comparison + adopt/reject reasons)
3. Flow whiteboard (architecture diagram, if system design is involved)
4. Comparison table (candidate detail comparison)
5. Action items (next steps)
6. Risks & unverified

**Special notes**:
- Decision table must have a clear conclusion (which option is chosen, which is not)
- Action items must have executable acceptance criteria

---

## Recipe 6: Meeting Notes

**Keywords**: "meeting summary", "discussion results", "会议纪要"

**Bricks**:

1. One-line conclusion
2. Decision table (decisions reached)
3. Action items (item / owner / deadline / acceptance)
4. Risks & unverified (unresolved disagreements)

**Special notes**:
- No running transcript — only decisions and action items
- Record disagreements honestly, do not smooth them over

---

## Recipe 7: Troubleshooting Postmortem

**Keywords**: "debug recap", "postmortem", "排障复盘"

**Bricks**:

1. One-line conclusion (what the root cause was)
2. Step log (reproduction sequence: trigger → symptom → isolation → fix → verify)
3. Code evidence (error location, root cause code)
4. Decision table (candidate fixes considered, which was chosen and why)
5. Risks & unverified (side effects of the fix, untested edge cases)
6. Action items (preventive measures, follow-up tests)

**Special notes**:
- Follow the full chain: problem → root cause → fix → verification. Do not jump to the fix.
- The reproduction sequence is the most valuable part — it lets others verify the fix independently
- If multiple fix candidates were considered, record why the others were rejected
