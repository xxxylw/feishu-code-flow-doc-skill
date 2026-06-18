---
name: lark-code-flow-doc
description: Create Feishu/Lark Docx code-flow documentation with lark-cli. Use when the user wants an AI agent to read a codebase or code path, explain how the code runs, create or update a Feishu document, include Feishu whiteboard diagrams for execution flow/data flow/call flow, and embed correctly tagged code snippets such as cmake, cpp, c, python, bash, xml, yaml, json, or text.
---

# Lark Code Flow Doc

Use this skill to produce Feishu documents that explain code by runtime flow. The final artifact is a Lark/Feishu Docx document, usually with one or more whiteboards for the main execution flow.

This skill defines the documentation standard. For actual Feishu operations, use the existing `lark-doc` and `lark-whiteboard` skills and follow their required authentication, XML, fetch, update, and whiteboard workflows.

## Core Rule

Explain code by flow, not by file order.

When content involves any kind of flow, such as execution flow, data flow, state flow, control flow, callback flow, call flow, message flow, startup flow, or error-handling flow, represent it with a Feishu whiteboard first. Use tables or prose as supporting detail, not as the only representation.

Preserve existing Feishu document titles. If `docs +fetch --api-version v2` returns an existing `<title>`, do not update, replace, overwrite, or re-create that title. Write only body blocks after the existing title unless the user explicitly asks to rename the document.

Prefer:

```text
startup -> build target -> entry function -> initialization -> callback/loop -> output
```

Avoid:

```text
file A summary -> file B summary -> file C summary
```

## Workflow

1. Confirm the requested scope: repository, module, feature, bug path, launch path, or specific files.
2. Inspect the code with CLI tools before writing: use `rg --files`, `rg`, `git log`, build files, launch scripts, package manifests, and entry-point searches.
3. Exclude third-party, generated, build, cache, and vendored directories unless the user explicitly targets them.
4. Identify the runtime path: startup command/script, build declaration, executable target, entry function, core classes/functions, callbacks/loops, and outputs.
5. Draft the document structure from `references/doc-structure.md`.
6. Create the main diagram plan from `references/flow-diagram-rules.md`.
7. Write code evidence and explanations using `references/code-explanation-style.md`.
8. Before updating an existing Feishu document, fetch it with `docs +fetch --api-version v2` and check for `<title>`. If a title exists, preserve it and omit `<title>` from update payloads; prefer `append` or `block_insert_after` body updates over `overwrite`.
9. Create or update the Feishu document with `lark-cli docs --api-version v2`, following the UTF-8 write-safety rules below.
10. Insert or update Feishu whiteboards for every important flow section, including data/state/control/callback/call/message flows. Keep diagram source in the document when useful.
11. Fetch or inspect the created document enough to verify that the original title is unchanged, headings, diagrams, code blocks, tables, references, and non-ASCII text were inserted correctly.

## UTF-8 Write Safety

When writing XML or Markdown that contains non-ASCII text, especially Chinese, do not rely on a Windows PowerShell pipeline such as `@'...'@ | lark-cli ... --content -`. It can corrupt UTF-8 content into `?` characters before `lark-cli` receives it.

Prefer one of these safe paths:

1. Write the document payload to a UTF-8 file, then pass it by relative `@file` path:

```powershell
# File must be UTF-8. Path must be relative to the current working directory.
lark-cli docs +update --api-version v2 --doc "<doc>" --command append --content "@tmp/payload.xml"
```

2. Use a runtime that guarantees UTF-8 stdin and has the same `lark-cli` auth/config environment as the shell. If that environment is uncertain, fall back to the UTF-8 `@file` path.

After every write, fetch a small outline or keyword fragment that includes representative non-ASCII text:

```powershell
lark-cli docs +fetch --api-version v2 --doc "<doc>" --scope keyword --keyword "一句话结论|核心流程" --detail with-ids
```

If the fetched result contains `?` in place of expected non-ASCII text, immediately rewrite the affected content through a UTF-8 `@file` payload and verify again.

## Feishu Output Contract

The document must include:

- A title that names the module or feature and says it is a code-flow explanation when creating a new document. For an existing Feishu document, preserve the existing title exactly unless the user explicitly requests a rename.
- A reading scope section that lists included and excluded paths.
- A one-paragraph conclusion explaining responsibility, inputs, outputs, and system position.
- A main flow whiteboard for execution flow, data flow, or call flow.
- Feishu whiteboards for any section whose main content is a flow; for example data flow, state flow, control flow, callback flow, message flow, or error-handling flow.
- An entry-chain section from startup/build configuration to the first business function.
- A flow-ordered explanation of key code snippets.
- A table of important functions/classes/configuration files.
- A section for risks, assumptions, and unverified points.

## Evidence Rules

- Cite file paths for every important claim.
- Prefer function/class names and build target names over vague prose.
- Include only the smallest code snippet that proves the point.
- Mark uncertain claims as unverified instead of presenting them as facts.
- If code cannot be run locally, state that the documentation is based on static CLI inspection.

## Code Block Language Tags

Use accurate code block language tags in Feishu/Markdown content:

| Content | Tag |
|---|---|
| CMake | `cmake` |
| C++ | `cpp` |
| C | `c` |
| Python | `python` |
| Shell scripts or CLI commands | `bash` |
| XML, launch XML, package manifests | `xml` |
| YAML configs | `yaml` |
| JSON configs | `json` |
| Plain call chains or logs | `text` |

Do not use untagged code fences for source code.

## References

- Read `references/doc-structure.md` before drafting the Feishu document.
- Read `references/flow-diagram-rules.md` before creating or updating a whiteboard.
- Read `references/code-explanation-style.md` before writing code snippets and explanations.
