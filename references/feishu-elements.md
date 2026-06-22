# Feishu XML Reference

This file contains only the correct Feishu Docx XML syntax. It does not duplicate the full `lark-doc` skill documentation — it serves as a quick reference for this skill during writing.

---

## Core Rule: Do Not Escape Tags

Tags themselves stay as-is. Only text content *inside* tags needs escaping.

- ✗ `&lt;p&gt;content&lt;/p&gt;` (tags escaped too)
- ✓ `<p>A &amp; B: 1 &lt; 2</p>` (tags intact, only `&` and `<` in text escaped)

Escape table: `<` → `&lt;`, `>` → `&gt;`, `&` → `&amp;`

---

## Standard Tags

`p, h1-h9, ul, ol, li, table, thead, tbody, tr, th, td, blockquote, pre, code, hr, img, b, em, u, del, a, br, span` — same semantics as HTML.

---

## callout (highlight box)

```xml
<callout emoji="✅" background-color="light-green" border-color="green">
<p>Locked conclusion content</p>
</callout>
```

Attributes:
- `emoji`: emoji character (💡 ✅ ❌ 📝 ❓ ❗ 👍 ❤️ 📌 🏁 ⭐), defaults to 💡
- `background-color`: `gray` + `light-{color}` + `medium-{color}`
- `border-color` / `text-color`: base 7 colors (red, orange, yellow, green, blue, purple, gray)
- Child blocks: text, headings, lists, todos, quotes only

Color semantics (defined in `style-guide.md`):

| Color | Meaning |
|-------|---------|
| Blue | Title block / project anchor |
| Green | Locked conclusion / milestone |
| Yellow | Warning / side-effect |
| Red | Blocker / known issue |

---

## table

```xml
<table>
<colgroup><col span="2" width="120"/></colgroup>
<thead><tr>
<th background-color="light-gray">Header A</th>
<th background-color="light-gray">Header B</th>
</tr></thead>
<tbody>
<tr><td>cell</td><td>cell</td></tr>
</tbody>
</table>
```

- With a header: first row in `<thead>` using `<th>`, rest in `<tbody>` using `<td>`
- `<th>` / `<td>` support `background-color` and `vertical-align` (top/middle/bottom)
- Merged cells: only the starting cell outputs `colspan` / `rowspan`
- `<colgroup>` / `<col>` for column width, placed right after `<table>`

thead uses `background-color="light-gray"` uniformly.

---

## pre / code (code block)

```xml
<pre lang="python" caption="Optional caption"><code>print("hello")</code></pre>
```

- Code must be inside the inner `<code>`, never directly under `<pre>`
- `lang`: language tag (see table below)
- `caption`: optional, code block title

**Code content escaping**: code frequently contains `<`, `>`, `&` (e.g. `vector<T>`, `a && b`, `x < y`, XML/HTML snippets). These must be escaped as `&lt;`, `&gt;`, `&amp;` inside `<code>`, otherwise the payload breaks.

Language tags:

| Content | Tag |
|---------|-----|
| CMake | cmake |
| C++ | C++ |
| C | c |
| Python | python |
| Shell / CLI | bash |
| XML | xml |
| YAML | yaml |
| JSON | json |
| Call chains / logs | text |

---

## latex (formulas)

Inline:

```xml
<p>The policy gradient objective <latex>J(\theta) = \mathbb{E}[R(\tau)]</latex> requires differentiating w.r.t. θ.</p>
```

Block-level: wrap each formula in its own `<p>`, bridge with prose between formulas:

```xml
<p>Objective function:</p>
<p><latex>J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[R(\tau)]</latex></p>
<p>Using the log-derivative trick:</p>
<p><latex>\nabla_\theta J(\theta) = \mathbb{E}[\nabla_\theta \log \pi_\theta(\tau) \cdot R(\tau)]</latex></p>
```

---

## whiteboard

```xml
<whiteboard type="svg">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 70">
  <defs>
    <marker id="arr" markerWidth="8" markerHeight="8" refX="7" refY="3" orient="auto">
      <path d="M0,0 L0,6 L7,3 z" fill="#56b6c2"/>
    </marker>
  </defs>
  <rect x="10" y="13" width="120" height="44" rx="6" fill="#212337" stroke="#61afef" stroke-width="2"/>
  <text x="70" y="40" text-anchor="middle" fill="#abb2bf" font-size="13" font-family="sans-serif">start</text>
  <line x1="130" y1="35" x2="166" y2="35" stroke="#56b6c2" stroke-width="2" marker-end="url(#arr)"/>
  <rect x="170" y="13" width="120" height="44" rx="6" fill="#1a2e29" stroke="#98c379" stroke-width="2"/>
  <text x="230" y="40" text-anchor="middle" fill="#abb2bf" font-size="13" font-family="sans-serif">output</text>
</svg>
</whiteboard>
```

- `type`: `mermaid` / `svg` / `blank` (avoid `plantuml` — Feishu parse failure)
- Simple diagrams without custom colors: `mermaid` inline
- Colored diagrams: `svg` inline with One Dark palette (see `whiteboard-rules.md` for full color scheme)

See `whiteboard-rules.md` for full strategy.

---

## grid / column (multi-column layout)

```xml
<grid>
<column width-ratio="0.5">
<p>Left column content</p>
</column>
<column width-ratio="0.5">
<p>Right column content</p>
</column>
</grid>
```

`width-ratio` values across columns should sum to 1.

---

## checkbox (todo items)

```xml
<checkbox done="true">Completed item</checkbox>
<checkbox done="false">Pending item</checkbox>
```

Suitable for simple todo lists. For action items requiring owner/deadline/acceptance criteria, use a table (brick 11).

---

## Inline Style Nesting Order

Inline style tags must be nested in this fixed order (outer → inner), closed in strict reverse:

```
<a> → <b> → <em> → <del> → <u> → <code> → <span> → text
```

---

## UTF-8 Write Safety

When writing content with non-ASCII text (Chinese, etc.), do not pipe through PowerShell with `--content -` — it corrupts UTF-8 into `?` characters.

**Default path**: write the XML payload to a UTF-8 file, pass it by relative `@file` path:

```bash
lark-cli docs +update --api-version v2 --doc "<doc>" --command append --content "@tmp/payload.xml"
```

Inline `--content '<p>...</p>'` is acceptable only for short ASCII-only debugging. For any real document, always use `@file`.

After writing, verify: fetch a fragment containing non-ASCII text and check for `?` corruption.

---

## Existing Title Protection

Before updating an existing document, always fetch first:

```bash
lark-cli docs +fetch --api-version v2 --doc "<doc>"
```

If the response contains an existing `<title>`, do not overwrite or rewrite it. Only append body content using `append` or `block_insert_after`.

---

## Common lark-cli Commands

```bash
# Create document
lark-cli docs +create --api-version v2 --content '<title>Title</title><p>Content</p>'

# Append content to existing document
lark-cli docs +update --api-version v2 --doc "<doc>" --command append --content '<p>Content</p>'

# Read document
lark-cli docs +fetch --api-version v2 --doc "<doc>"

# Create/update from UTF-8 file (recommended for any non-trivial content)
lark-cli docs +create --api-version v2 --content "@tmp/payload.xml"
lark-cli docs +update --api-version v2 --doc "<doc>" --command append --content "@tmp/payload.xml"
```
