# Document Bricks

A Feishu document is assembled from bricks. The AI selects which bricks to include based on conversation content features. No fixed template is preset.

Each brick defines: trigger condition, content requirements, Feishu element, and good/bad examples.

---

## 1. Metadata Header

**Trigger**: conversation involves hardware models, absolute paths, project names, or key dates/versions, with at least two present. Pure discussion without environment anchoring does not trigger.

**Content**: each line is a key-value pair covering project, hardware, path, date. Do not write reflections or project significance.

**Feishu element**: blue callout, emoji 📌, at most one per document.

Good:
```xml
<callout emoji="📌" background-color="light-blue" border-color="blue">
<p>Project: Model Fine-tuning / Hardware: 4090 24G / Date: 2026-06-21</p>
<p>Path: /data/project/run1</p>
</callout>
```

Bad (treating the callout as a summary with filler):
```xml
<callout emoji="📌" background-color="light-blue" border-color="blue">
<p>This is a very important training record documenting the entire process...</p>
<p>I hope this will be helpful for future work...</p>
</callout>
```

---

## 2. One-line Conclusion

**Trigger**: always present in every document. Positioned right after the metadata header or at the very top.

**Content**: one to three short sentences. Cover at least two of: what this document records, the key result, the current status. No preamble, no adjective stacking, no academic tone.

**Feishu element**: green callout, emoji ✅ or 🎯, at most one per document.

Good:
```xml
<callout emoji="✅" background-color="light-green" border-color="green">
<p>SFT first convergence: loss 0.42, trainable 16.52M (0.679%), 4h12m.</p>
</callout>
```

Bad (preamble + academic tone + no numbers):
```xml
<callout emoji="🎯" background-color="light-green" border-color="green">
<p>This training achieved some decent progress, fully reflecting first principles...</p>
</callout>
```

---

## 3. Flow Whiteboard

**Trigger**: conversation contains execution flow, data flow, call flow, state flow, control flow, architecture relationships, component interactions, or timing relationships.

**Content**:
- Node count should be appropriate — not so few that the diagram adds no information, not so many that it becomes unreadable. If a single diagram gets too dense, split into a main diagram plus detail diagrams.
- Node labels: action + symbol/path. Short. No line breaks or HTML.
- Start point must be a real runtime entry (startup command, entry function, external trigger), not an arbitrary file.
- End point must be an observable output (publish, write, return, API response, saved file).
- Inferred or uncertain links must be labeled `inferred` or `unverified`.

Whether a whiteboard is actually needed is judged by the rules in `whiteboard-rules.md`, not just by hitting this brick.

**Feishu element**: whiteboard block. Type selection and fallback path in `whiteboard-rules.md`.

Good:
```xml
<whiteboard type="mermaid">
flowchart TD
  A["startup script"] --> B["load config"]
  B --> C["build target"]
  C --> D["entry function"]
  D --> E["initialize"]
  E --> F["callback/loop"]
  F --> G["output result"]
</whiteboard>
```

Bad (nodes are paragraphs with HTML and subjective evaluation):
```xml
<whiteboard type="mermaid">
flowchart TD
  A["The very important first step of setup: you need to prepare hardware first<br/>including GPU selection and memory estimation"] --> B["then install CUDA"]
  B --> C["finally run training"]
</whiteboard>
```

Below the whiteboard, add a mapping table:

| Diagram node | Corresponding code/file | Note |
|--------------|------------------------|------|
| ASR node | src/asr_node.cpp::AsrNode | Receives microphone input |
| LLM node | src/llm_node.cpp::onRequest | Handles /llm_request |

---

## 4. Step Log

**Trigger**: conversation has a reproducible operation sequence, where each step has a command, a config change, or a UI path. Typical: environment setup, training, deployment, troubleshooting.

**Content**: each step has 4 required parts:

```
Goal: what this step does (one sentence)
Command: pasteable command / config change / UI path
Verify: what to check after running (command / output / status table)
Why: why this step, why this value (one sentence)
```

"Why" is what distinguishes this skill from a command-paste notebook. A step without it is non-compliant.

**Feishu element**: h3 heading + `<pre lang="bash">` command block + verification table (with ✅/⏳/❌). "Why" is a standalone short paragraph, not a callout.

Good:
```xml
<h3>Step 3: Install vllm 0.5.4</h3>
<p>Goal: install vllm 0.5.4 for inference server, CUDA 12.1 compatible.</p>
<pre lang="bash" caption="Install vllm"><code>pip install vllm==0.5.4</code></pre>
<table>
<thead><tr><th background-color="light-gray">Check</th><th background-color="light-gray">Expected</th><th background-color="light-gray">Status</th></tr></thead>
<tbody>
<tr><td>version</td><td>0.5.4</td><td>✅</td></tr>
<tr><td>import</td><td>no error</td><td>✅</td></tr>
</tbody>
</table>
<p>Why: 0.5.4 is the highest stable version compatible with CUDA 12.1 + transformers 4.42.</p>
```

Bad (no version, no verification, no reason):
```xml
<h3>Step 3: Install vllm</h3>
<p>pip install vllm</p>
<p>Done.</p>
```

---

## 5. Decision Table

**Trigger**: conversation contains "choose between X and Y", "why X not Y", "change A to B", "tune C to D".

**Content**: each row must have an "adopt/reject + reason". Listing options without reasons is not allowed. Reasons must contain specific metric differences or constraints, not "gut feeling".

**Feishu element**: table, thead with `background-color="light-gray"`.

Good:
```xml
<table>
<thead><tr>
<th background-color="light-gray">Framework</th>
<th background-color="light-gray">VRAM</th>
<th background-color="light-gray">First-token latency</th>
<th background-color="light-gray">Decision</th>
</tr></thead>
<tbody>
<tr><td>vllm 0.5.4</td><td>14.2 GB</td><td>38 ms</td><td>Adopt: most stable on CUDA 12.1</td></tr>
<tr><td>vllm 0.6.2</td><td>14.0 GB</td><td>92 ms</td><td>Reject: latency regression on 12.1</td></tr>
<tr><td>TGI 2.0</td><td>16.8 GB</td><td>41 ms</td><td>Reject: exceeds VRAM budget</td></tr>
</tbody>
</table>
```

Bad (no metrics, no reasons):
```xml
<table>
<thead><tr><th background-color="light-gray">Option</th><th background-color="light-gray">Note</th></tr></thead>
<tbody>
<tr><td>vllm</td><td>easy to use</td></tr>
<tr><td>TGI</td><td>not great</td></td></tr>
</tbody>
</table>
```

---

## 6. Code Evidence

**Trigger**: conversation references source code (file path, function name, class name), or discusses what code does / what was changed.

**Content**: each code evidence segment has 5 required parts:

```
Flow position: where this code sits in the runtime path
Evidence: file.cpp::func or file.py::Class.method
Code: minimal snippet with language tag, key lines annotated
Explanation: what it does and why it advances the flow
Next: where the flow continues
```

Comments explain flow intent (data in/out, what it triggers, state changes), not syntax. Below the code block, summarize the annotated key lines in prose. Uncertain behavior is labeled Verified / Inferred / Unverified.

**Feishu element**: `<pre lang="...">` code block. See language tag table in `feishu-elements.md`.

Good:
```xml
<p>Flow position: ASR-to-LLM handoff point</p>
<p>Evidence: src/asr_node.cpp::AsrNode::onFinal</p>
<pre lang="C++"><code>ros::Publisher llm_pub =
    nh.advertise&lt;std_msgs::String&gt;("/llm_request", 10);  // Opens ASR → LLM channel
msg.data = text;       // Loads finalized ASR sentence
llm_pub.publish(msg);  // Triggers LLM node subscription</code></pre>
<p>This code hands the finalized ASR sentence to the LLM node — the relay point of the end-to-end flow. Next: src/llm_node.cpp::onRequest.</p>
```

Bad (pastes entire file, no comments, wrong tag case, no Next):
```xml
<pre lang="cpp"><code>... 200 lines of the entire onFinal function with no comments ...</code></pre>
<p>This is the code for asr_node.cpp. It's important.</p>
```

---

## 7. Math Derivation

**Trigger**: formulas appear (LaTeX notation, loss, gradient, probability distribution, algorithm proof).

**Content**:
- Before each formula: one sentence explaining what it describes
- After each formula: one sentence explaining how it leads to the next step / why it is written this way
- Pasting a wall of formulas without explanation is not allowed
- Key variables are defined on first appearance (with shape / dimension / range)

**Feishu element**: inline `<latex>`, block-level uses multiple `<p><latex></latex></p>` with prose bridging between formulas.

Good:
```xml
<p>The policy gradient objective maximizes expected return:</p>
<p><latex>J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[R(\tau)]</latex></p>
<p>where τ is a trajectory and R(τ) is the total reward. Taking the gradient w.r.t. θ directly is intractable. Using the log-derivative trick:</p>
<p><latex>\nabla_\theta J(\theta) = \mathbb{E}_{\tau}[\nabla_\theta \log \pi_\theta(\tau) \cdot R(\tau)]</latex></p>
<p>This converts "differentiating the distribution" into "differentiating the log-probability" — the former cannot be estimated by sampling, the latter can.</p>
```

Bad (three formulas pasted consecutively, no bridging):
```xml
<p>Policy gradient formulas:</p>
<p><latex>J(\theta) = \mathbb{E}[R(\tau)]</latex></p>
<p><latex>\nabla_\theta J = \mathbb{E}[\nabla \log \pi \cdot R]</latex></p>
<p><latex>\approx \frac{1}{N}\sum_i \nabla \log \pi(\tau_i) R(\tau_i)</latex></p>
<p>That's REINFORCE.</p>
```

---

## 8. Comparison Table

**Trigger**: multiple objects compared item by item, and the comparison is descriptive (not a selection decision).

**Difference from Decision Table (brick 5)**: the decision table carries an "adopt/reject" action. The comparison table only records differences without drawing a conclusion.

**Distinguishing rule**: if the content contains "adopt / reject / why change / why this value", it goes to the decision table. The comparison table shows facts and differences only — no "why changed" column.

**Content**: first column is the dimension/metric, remaining columns are one per object. Numbers preferred. A "difference note" column, if present, states the factual difference (e.g. "+4 / -8 GB"), not a reason for change.

Good:
```xml
<table>
<thead><tr>
<th background-color="light-gray">Hyperparam</th>
<th background-color="light-gray">P0 baseline</th>
<th background-color="light-gray">P2 adjusted</th>
<th background-color="light-gray">Difference</th>
</tr></thead>
<tbody>
<tr><td>LoRA rank</td><td>16</td><td>8</td><td>-8</td></tr>
<tr><td>batch size</td><td>4</td><td>8</td><td>+4</td></tr>
<tr><td>lr</td><td>2e-4</td><td>1e-4</td><td>-1e-4</td></tr>
</tbody>
</table>
```

Bad (adjectives instead of numbers, no difference):
```xml
<table>
<thead><tr><th background-color="light-gray">Item</th><th background-color="light-gray">P0</th><th background-color="light-gray">P2</th></tr></thead>
<tbody>
<tr><td>LoRA rank</td><td>high</td><td>low</td></tr>
<tr><td>speed</td><td>slow</td><td>fast</td></tr>
</tbody>
</table>
```

---

## 9. Version Baseline

**Trigger**: conversation involves CUDA / Python / framework / model / package versions, and these versions may update as work progresses.

**Content**: columns for component / version / status / note. Status uses ✅ (verified) / ⏳ (in progress) / ❌ (known issue). Notes only contain hard constraints or known issues.

Good:
```xml
<table>
<thead><tr>
<th background-color="light-gray">Component</th>
<th background-color="light-gray">Version</th>
<th background-color="light-gray">Status</th>
<th background-color="light-gray">Note</th>
</tr></thead>
<tbody>
<tr><td>CUDA</td><td>12.1</td><td>✅</td><td>Compatible with driver 535+</td></tr>
<tr><td>vllm</td><td>0.5.4</td><td>✅</td><td>0.6.x has latency regression on 12.1</td></tr>
<tr><td>PEFT</td><td>0.11.1</td><td>⏳</td><td>LoRA merge pending verification</td></tr>
</tbody>
</table>
```

Bad (imprecise versions, no status, no constraints):
```xml
<table>
<thead><tr><th background-color="light-gray">Component</th><th background-color="light-gray">Version</th></tr></thead>
<tbody>
<tr><td>CUDA</td><td>latest</td></tr>
<tr><td>vllm</td><td>installed</td></tr>
</tbody>
</table>
```

---

## 10. Risks & Unverified

**Trigger**: inferences / unverified claims / assumptions / missing external dependencies / untraced branches.

**Content**: each item has three elements — phenomenon + status label + verification method. Status label is one of Verified / Inferred / Unverified.

Items that are "to be verified" (no owner/deadline yet) go here. Items with a clear owner/deadline/acceptance criteria go to Action Items (brick 11).

**Feishu element**: bullet list (not callout). Use h3 heading.

Good:
```xml
<h3>Risks, Assumptions, Unverified Points</h3>
<ul>
<li>LoRA merge inference consistency — Unverified — run eval set, compare logits diff</li>
<li>Long-context (&gt;4k) behavior — Inferred — training samples average 1.2k, never tested 4k+</li>
<li>Chinese tokenizer math-symbol splitting — Verified — spot-checked 20 BPE splits</li>
</ul>
```

Bad (using a callout as a one-line disclaimer):
```xml
<callout emoji="❗" background-color="light-red" border-color="red">
<p>The above is for reference only and may be inaccurate.</p>
</callout>
```

---

## 11. Action Items

**Trigger**: after a meeting, troubleshooting, or decision discussion, there are clear next steps.

**Content**: item / owner / deadline / acceptance criteria. Owner unknown → write "TBD", do not fabricate. Acceptance criteria must be executable.

**Feishu element**: table.

Good:
```xml
<table>
<thead><tr>
<th background-color="light-gray">Item</th>
<th background-color="light-gray">Owner</th>
<th background-color="light-gray">Deadline</th>
<th background-color="light-gray">Acceptance criteria</th>
</tr></thead>
<tbody>
<tr><td>Run smoke test</td><td>TBD</td><td>TBD</td><td>100 steps, no error, logs produced</td></tr>
</tbody>
</table>
```

Bad (no executable definition):
```xml
<table>
<thead><tr><th background-color="light-gray">Item</th><th background-color="light-gray">Owner</th></tr></thead>
<tbody>
<tr><td>continue optimizing</td><td>TBD</td></tr>
</tbody>
</table>
```

---

## 12. Worked Example

**Trigger**: the document walks through a concrete calculation with actual numbers plugged in — e.g. "compute GAE for one episode", "trace one sample through the reward function", "show how loss changes across three training steps". This is distinct from Math Derivation (brick 7): Math Derivation is symbolic/abstract; Worked Example is numerical/specific.

**Content**: four parts, in order.

1. **Setup paragraph**: state parameters and input data once, upfront. Compute intermediate quantities here and list their values inline. Do not re-derive them step-by-step — they are setup, not the point.
2. **Main formula(s) as standalone blocks**: each principal formula (the recursion/iteration rule) gets its own block-level `<latex>`, bridged by one explanation sentence before the walkthrough.
3. **Step-by-step walkthrough inside a `<blockquote>`**: one `<p>` per step. Each paragraph is one calculation (inline `<latex>`) plus one explanation sentence explaining WHY this result matters (a trend, a surprise, the mechanism), not just restating the number.
4. **Optional result table AFTER the walkthrough**: only if later content needs to reference the values.

**Rules**:
- **No bullet lists in the walkthrough**: each step is its own `<p>`, optionally leading with a bold label.
- **No code blocks for the main recursion**: the step-by-step must read as explanation, not a REPL transcript.
- **Density**: compute derived inputs once in prose; spend the step budget on the main recursion, one explanation sentence per step.

**Relationship to Math Derivation**: they are complementary. Math Derivation gives the formula; Worked Example plugs in numbers. A document can have one without the other. When both hit, Math Derivation comes first.

**Feishu element**: `<blockquote>` containing `<p>` paragraphs with inline `<latex>`. Standalone formulas use block-level `<p><latex></latex></p>` before the blockquote.

Good:
```xml
<p>The TD error and GAE recursion are:</p>
<p><latex>\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)</latex></p>
<p><latex>\hat{A}_t = \delta_t + \gamma\lambda\hat{A}_{t+1}</latex></p>
<p>With γ=0.99, we get δ₀=1.292, δ₁=0.690, δ₂=0.797, δ₃=-0.800. γλ=0.9405.</p>
<blockquote>
<p><b>t=3</b>  <latex>\hat{A}_3 = -0.800</latex></p>
<p>Last step has no future reward, so advantage equals the TD error directly.</p>
<p><b>t=2</b>  <latex>\hat{A}_2 = 0.797 + 0.9405 \times (-0.800) = 0.045</latex></p>
<p>Current δ is positive, but gets pulled back by the next step's negative advantage, nearly zeroing out.</p>
<p><b>t=1</b>  <latex>\hat{A}_1 = 0.690 + 0.9405 \times 0.045 = 0.732</latex></p>
<p>Next-step advantage is near zero, so this is basically just the current δ.</p>
<p><b>t=0</b>  <latex>\hat{A}_0 = 1.292 + 0.9405 \times 0.732 = 1.980</latex></p>
<p>Positive signal accumulates along the trajectory, growing larger toward earlier steps.</p>
</blockquote>
```

Bad (formula wall, no explanation, code block instead of prose):
```xml
<p>δ₀ = 1.292, δ₁ = 0.690, δ₂ = 0.797, δ₃ = -0.800</p>
<pre lang="text"><code>
Â₃ = -0.800
Â₂ = 0.797 + 0.9405×(-0.800) = 0.045
Â₁ = 0.690 + 0.9405×0.045 = 0.732
Â₀ = 1.292 + 0.9405×0.732 = 1.980
</code></pre>
```
