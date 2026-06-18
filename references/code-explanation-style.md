# Code Explanation Style

Write code explanations as evidence for the runtime flow.

## Snippet Pattern

Each important snippet should include:

```text
Flow position: where this code sits in the runtime path.
Evidence: file path and symbol.
Code: minimal snippet with correct language tag.
Explanation: what it does and why it moves the flow forward.
Next: where the flow continues.
```

## Good Example

```text
Flow position: build target definition
Evidence: src/base_control/CMakeLists.txt
```

```cmake
add_executable(base_control_node src/base_control_node.cpp)
ament_target_dependencies(base_control_node rclcpp std_msgs)
```

Explanation:

This declares `base_control_node` as the executable target. Later launch or startup commands run this target, so this CMake block connects the build configuration to the runtime entry.

Next:

Trace `src/base_control_node.cpp::main`.

## Bad Example

Avoid:

```text
This file is important and contains build logic.
```

It is too vague and does not explain the flow.

## Language Tags

Use:

- `cmake` for `CMakeLists.txt` and `.cmake`.
- `C++` for C++ source and headers.
- `c` for C source and headers.
- `python` for Python.
- `bash` for shell scripts and CLI command blocks.
- `xml` for launch XML, package manifests, and XML config.
- `yaml` for YAML.
- `json` for JSON.
- `text` for call chains, logs, and non-source-code traces.

## Snippet Size

Keep snippets small:

- Prefer 5-20 lines.
- Omit unrelated branches.
- Do not paste whole files.
- Replace omitted sections with short comments only when needed for readability.

## Key-Line Comments

Do not paste code blocks as unexplained raw code. For every code block, add concise comments on the key lines or small key groups that make the runtime flow understandable.

Use comments to explain flow intent, not syntax:

- Why this task, subscription, callback, queue, message, or state update matters.
- What data enters and leaves this line or group.
- Which downstream function, topic, file, hardware output, or API call it triggers.

Avoid commenting every obvious line. A good snippet has a few high-signal comments plus the prose `Explanation` and `Next` sections below it.

Example:

```C++
ros::Publisher llm_pub =
    nh.advertise<std_msgs::String>("/llm_request", 10);  // Opens the ASR -> LLM handoff topic.

msg.data = text;      // Carries the finalized ASR sentence.
llm_pub.publish(msg); // Triggers the LLM node's /llm_request subscriber.
```

## Explanation Focus

Explain:

- Inputs: parameters, messages, files, environment variables, callbacks, hardware events.
- State: objects, fields, queues, buffers, flags, configuration.
- Outputs: return values, publishes, writes, sends, commands, hardware control, logs.
- Flow transition: what code location or runtime event happens next.

Do not explain obvious syntax. Explain why this code matters to the larger runtime path.

After each code block, explicitly summarize the key commented lines in prose. The prose should answer: what this code receives, what it changes, what it emits, and where the flow continues.

## Uncertainty Language

Use clear uncertainty labels:

- `Verified:` for behavior confirmed by running a command/test or by direct code evidence.
- `Inferred:` for behavior concluded from code structure but not executed.
- `Unverified:` for behavior that depends on hardware, credentials, external services, model files, or missing runtime context.

Never hide uncertainty.
