# Feishu Code-Flow Document Structure

Use this structure for the Feishu Docx document. Adapt section names to the user's module, but keep the flow-first order.

## Title

Use:

```text
<module-or-feature-name> 代码流程说明
```

For English projects, use:

```text
<module-or-feature-name> Code Flow Explanation
```

## Required Sections

### 1. 阅读范围

List what was inspected and what was excluded.

Include:

- Target module, feature, directory, or files.
- Search methods used, such as `rg`, build files, launch scripts, package manifests, or git history.
- Excluded paths such as vendor, third-party libraries, generated files, build outputs, caches, and model artifacts.

### 2. 一句话结论

Explain:

- What this code is responsible for.
- Where it sits in the system.
- What it receives as input.
- What it produces as output.
- Whether the conclusion is static-analysis only or verified by running commands/tests.

### 3. 总流程画板

Insert a Feishu whiteboard that shows the main flow.

The document text around the whiteboard should answer:

- What the diagram shows.
- Where the flow starts.
- Where the flow ends.
- Which files/functions are represented by the major nodes.

### 4. 入口链路

Use a table:

| Step | Evidence | Role in flow |
|---|---|---|
| 1 | `path/to/start.sh` | Starts the runtime process |
| 2 | `path/to/CMakeLists.txt` | Declares the executable target |
| 3 | `path/to/main.cpp::main` | Enters the runtime |

For ROS code, include launch files, package manifests, node executables, topics, services, and actions when present.

### 5. 核心代码流程

Write this section in execution order. Each subsection should represent one step in the runtime flow.

Use this pattern:

```text
Step N: <flow action>
Evidence: <file path and symbol>
Code: <minimal snippet>
Explanation: <why this step matters and what it passes to the next step>
Next: <next code location or runtime event>
```

### 6. 数据流/状态流

Use a table when possible:

| Data/state | Source | Transformation | Destination |
|---|---|---|---|
| sensor frame | subscriber callback | decode/filter | publisher/output API |

For event-driven systems, call out callbacks, timers, queues, subscriptions, interrupts, message handlers, or async tasks.

### 7. 关键对象和函数

Use a table:

| Symbol | File | Responsibility | Called by | Calls |
|---|---|---|---|---|

Keep this table selective. Include only symbols needed to understand the flow.

### 8. 风险、假设、未确认点

List:

- Hardware, environment, credentials, model files, external services, or configuration that were not available.
- Branches in code that were not traced.
- Behavior inferred from names or static code shape.
- Commands/tests that would verify the flow.

## Optional Sections

Add these only when useful:

- 构建与运行命令
- 配置项说明
- 错误处理流程
- 时序图
- 后续阅读路线
