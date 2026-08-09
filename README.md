# Prompt Engineering Guide

# 提示词工程指南

A practical, engineering-oriented guide for designing prompts in AI Agent, code generation, and content processing scenarios. This repository collects battle-tested patterns, iteration workflows, and domain-specific strategies.

一本面向 AI 智能体、代码生成和内容处理场景的工程化提示词设计实战指南。本仓库汇集了经过实战验证的模式、迭代工作流和领域特定策略。

---

## Table of Contents

## 目录

- [Introduction](#introduction)
- [简介](#introduction)
- [Prompt Structure](#prompt-structure)
- [提示词结构](#prompt-structure)
- [Design Principles](#design-principles)
- [设计原则](#design-principles)
- [Iteration Workflow](#iteration-workflow)
- [迭代工作流](#iteration-workflow)
- [Web Q&A Scenarios](#web-qa-scenarios)
- [Web 问答场景](#web-qa-scenarios)
- [Large‑Project Development (OpenCode, Cursor, etc.)](#large-project-development-opencode-cursor-etc)
- [大型项目开发（OpenCode、Cursor 等）](#large-project-development-opencode-cursor-etc)
- [Evaluation Metrics](#evaluation-metrics)
- [评估指标](#evaluation-metrics)
- [Prefix Tags](#prefix-tags)
- [前缀标签](#prefix-tags)

---

## Introduction

## 简介

Prompt is the primary instruction interface between users and AI models. Its quality directly determines output accuracy, consistency, and safety. This guide avoids subjective aesthetics and focuses on measurable practices that work across tasks.

提示词是用户与 AI 模型之间的主要指令接口，其质量直接决定输出的准确性、一致性和安全性。本指南不讨论主观审美，只聚焦于跨任务可复现、可度量的实践方法。

---

## Prompt Structure

## 提示词结构

A complete prompt consists of five independent modules. Compose them as needed.

一个完整的提示词由五个相互独立的模块组成，可按需组合。

| Module | Purpose | Example |
|--------|---------|---------|
| Role | Define expertise and perspective | "You are a backend architect with deep knowledge of Java concurrency." |
| Task | State what to accomplish explicitly | "Design a rate limiter pseudocode for 10M QPS." |
| Context | Provide background, constraints, or prior logs | "Current system uses Spring Cloud, no gateway protection." |
| Output Spec | Specify format, length, structure, style | "Output as Markdown table with columns: solution, pros/cons, applicable conditions." |
| Interaction Guide | Request step‑by‑step reasoning or intermediate outputs | "First analyze peak traffic patterns, then give your solution, finally explain fallback strategies." |

| 模块 | 作用 | 示例 |
|------|------|------|
| 角色 (Role) | 定义专业领域与视角 | "你是一位对 Java 并发有深入研究的后端架构师。" |
| 任务 (Task) | 明确陈述要完成的目标 | "请设计一个支持 1000 万 QPS 的限流器伪代码。" |
| 上下文 (Context) | 提供背景、约束或历史日志 | "当前系统使用 Spring Cloud，无网关保护。" |
| 输出规范 (Output Spec) | 规定格式、长度、结构、风格 | "以 Markdown 表格输出，列包括：方案、优缺点、适用条件。" |
| 交互指引 (Interaction Guide) | 要求逐步推理或输出中间结果 | "先分析峰值流量模式，再给出方案，最后说明降级策略。" |

Keep each module single‑responsibility. Merge or omit modules for simple tasks.

每个模块保持单一职责。对于简单任务，可以合并或省略部分模块。

---

## Design Principles

## 设计原则

### 1. Positive instructions over negative constraints

### 1. 正面指令优于负面约束

Models follow "do" statements better than "don't" statements.

模型对"要做什么"的指令执行效果优于"不要做什么"。

- ❌ "Do not use casual tone, do not omit technical details."
- ❌ "不要使用随意语气，不要遗漏技术细节。"
- ✅ "Use technical writing style; list all parameters and edge conditions."
- ✅ "使用技术写作风格，列出所有参数和边界条件。"

### 2. Explicit reasoning paths

### 2. 明确的推理路径

For complex tasks, force the model to output its reasoning chain.

对于复杂任务，强制模型输出其推理链条。

```
Answer in this order:
1. List all input constraints.
2. Propose at least two alternatives.
3. Give your recommendation with justification.
```

### 3. Few‑shot example rules

### 3. 少样本示例规则

- Use 3–6 examples.
- 使用 3–6 个示例。
- Balance categories in classification tasks.
- 分类任务中保持类别均衡。
- Sort examples from easiest to hardest (or from typical to edge cases) to mitigate position bias.
- 示例按从易到难排序（或从典型到边界），以减轻位置偏差。

### 4. Anchor reset in long conversations

### 4. 长对话中的锚点重置

Transformer models pay more attention to the beginning and end of the context. In conversations longer than 4000 tokens, repeat the core objective at the end of each new query. Use prefixes like `[Core Goal]` or `[Hard Constraint]` to reinforce key points.

Transformer 模型对上下文的开头和结尾更敏感。在超过 4000 token 的对话中，应在每条新提问的结尾重复核心目标，并使用 `[Core Goal]` 或 `[Hard Constraint]` 等前缀强化关键点。

---

## Iteration Workflow

## 迭代工作流

Follow this standard loop for prompt tuning. Log every change.

提示词调优遵循以下标准循环，并记录每一次变更。

| Step | Action | Record |
|------|--------|--------|
| Baseline | Write initial prompt based on task decomposition | Version number, expected sample output |
| Smoke test | Run 3 typical inputs, check output completeness | Pass/fail items, error type labels |
| Root‑cause analysis | Classify errors into: semantic misunderstanding, format deviation, information omission | Error distribution |
| Targeted fix | Modify only one module at a time; prioritize instruction rephrasing | Before/after comparison |
| Regression | Re‑run the same test set, measure improvement | Accuracy and format compliance deltas |
| Release | Finalise prompt and write usage documentation | Boundary conditions, known limitations |

| 步骤 | 动作 | 记录 |
|------|------|------|
| 基线 | 基于任务拆解编写初始提示词 | 版本号、预期样例输出 |
| 冒烟测试 | 运行 3 个典型输入，检查输出完整性 | 通过/失败项、错误类型标签 |
| 根因分析 | 将错误归类为：语义误解、格式偏差、信息遗漏 | 错误分布 |
| 定向修复 | 一次只修改一个模块，优先调整指令措辞 | 修改前后对比 |
| 回归测试 | 重新运行同一测试集，度量改进幅度 | 准确率与格式合规度差值 |
| 发布 | 定稿提示词并编写使用文档 | 边界条件、已知限制 |

---

## Web Q&A Scenarios

## Web 问答场景

When the model must answer based on provided web pages or search results, force it to stay within the given content.

当模型必须基于给定的网页或搜索结果作答时，强制其严格限定在给定内容范围内。

### Context injection template

### 上下文注入模板

```
[Available Content]
(paste plain‑text content here, keep paragraph markers)
[Content End]

[User Question]
(actual question)

[Response Rules]
- Answer only using the [Available Content].
- If the content does not contain the answer, respond exactly: "Cannot confirm based on available materials." Do not add external knowledge.
- When citing specific paragraphs, append the paragraph number in parentheses.
```

### Uncertainty handling

### 不确定性处理

Add a confidence self‑check:

添加置信度自检：

"At the end of your answer, label confidence as High / Medium / Low. For Medium or Low, explicitly list what key information is missing."

"在回答结尾标注置信度：高 / 中 / 低。若为中等或低，请明确列出缺失的关键信息。"

### Multi‑turn context management

### 多轮上下文管理

In follow‑up questions, output a "summary of evidence used" in each turn.

在追问中，每轮都输出"已使用证据摘要"。

Example format:  
示例格式：  
`Answer: ... Evidence summary: paragraphs 3, 5.`

---

## Large‑Project Development (OpenCode, Cursor, etc.)

## 大型项目开发（OpenCode、Cursor 等）

When using AI coding agents (OpenCode, Cursor, Devin) on repositories >5000 LOC with multiple modules, prompt design must shift from single‑task to project‑iteration mode. These tools decompose prompts into sub‑tasks across multiple context windows, so you need stable boundaries.

当在超过 5000 行代码、多模块的仓库中使用 AI 编程智能体（OpenCode、Cursor、Devin）时，提示词设计必须从单任务模式转向项目迭代模式。这些工具会把提示词分解为跨多个上下文窗口的子任务，因此你需要稳定的边界。

### 1. Project‑level system instructions

### 1. 项目级系统指令

Place a persistent instruction file (e.g., `.opencode/instructions.md`) in the project root. It defines global behaviour for all sessions.

在项目根目录放置一份持久化指令文件（如 `.opencode/instructions.md`），为所有会话定义全局行为。

Required entries:

必需条目：

- Language and version (e.g., Python 3.11, Java 17)
- 语言与版本（如 Python 3.11、Java 17）
- Package manager and registry (e.g., pnpm, npm registry)
- 包管理器与镜像源（如 pnpm、npm registry）
- Test framework and coverage target (e.g., Jest, Pytest, ≥80% line coverage)
- 测试框架与覆盖率目标（如 Jest、Pytest，行覆盖率 ≥80%）
- Code style rules (e.g., ESLint, Google Java Format)
- 代码风格规则（如 ESLint、Google Java Format）
- Documentation language and comment density (at least one comment per function)
- 文档语言与注释密度（每个函数至少一条注释）

### 2. Module‑level task decomposition

### 2. 模块级任务拆解

Do not submit the entire project at once. Break down by module, but include dependency and contract declarations.

不要一次性提交整个项目。按模块拆解，但需包含依赖与契约声明。

```
[Current Task]
Implement the login endpoint (/api/v1/login) for the user authentication module.

[Module Boundary]
- Handle login only; do not touch registration or password reset.
- Dependencies: database connection pool (exists, path: src/db/pool.ts).
- Exposed interface: returns JWT token + user basic info.

[Upstream Contract]
- Frontend expects: { code: number, data: { token: string, expires_in: number }, message: string }.
- Error code range: 1001–1005.

[Downstream Dependencies]
- User table schema: src/models/user.sql.
- Redis cache for failed login attempts, key format: login_fail:{ip}:{username}.

[Acceptance Criteria]
- Lock account for 15 minutes after 3 failed attempts.
- Unit tests cover: normal login, wrong password, account locked.
- Median response time < 200ms.

[Output Constraints]
- Only modify src/controllers/auth.ts and src/services/auth.service.ts.
- Do not change config files or add new dependencies.
- After code, provide one pytest command to run the tests.
```

### 3. Modification requests (change prompts)

### 3. 修改请求（变更提示词）

When updating existing code, lock the scope and require impact analysis.

更新既有代码时，锁定修改范围并要求做影响分析。

```
[Modification Target]
Replace the email regex in src/utils/validator.ts with pattern B (support new TLDs).

[Impact Analysis]
- Callers: registration, profile update, password reset.
- Check compatibility of these three call sites with the new regex; update their unit tests accordingly.

[Regression Check]
- Run `npm run test:validator` and ensure no new failures (except intentionally corrected cases).
- Output a before‑after diff list covering at least 5 edge cases (e.g., .io, .ai).
```

### 4. Debug prompts

### 4. 调试提示词

Attach the full stack trace and the last relevant commit hash. Require root‑cause analysis and fix comparison.

附上完整的堆栈信息与最后一次相关的提交哈希，要求进行根因分析与修复方案对比。

```
[Error Symptom]
Production error: TypeError: Cannot read property 'id' of undefined, stack points to src/middleware/auth.ts:48.

[Context]
This file was refactored in commit 7f3a2b1 (token parsing logic).

[Diagnosis Requirements]
1. List at least two possible code paths leading to this error.
2. For each, specify the triggering conditions.
3. Recommend a fix and provide a patch snippet.
4. Explain defensive measures to prevent similar issues.
```

### 5. Do‑not‑do list for large projects

### 5. 大型项目的禁忌清单

- Avoid vague requests like "optimise the code" – specify performance, readability, or memory.
- 避免"优化代码"这类模糊请求——需明确是性能、可读性还是内存。
- Do not mix frontend UI changes with database migration instructions in one prompt – agents tend to modify too many files at once.
- 不要在同一个提示词中混合前端 UI 修改与数据库迁移指令——智能体会一次性改动过多文件。
- If your project uses custom scripts or Makefiles, explicitly state the build and test commands.
- 如果项目使用自定义脚本或 Makefile，请明确说明构建与测试命令。
- For agents with function‑calling (OpenCode), explicitly allow or disallow terminal execution, file writes, and network access.
- 对于支持函数调用的智能体（OpenCode），请明确允许或禁止终端执行、文件写入和网络访问。

---

## Evaluation Metrics

## 评估指标

Use these four hard metrics instead of subjective scoring.

用以下四个硬指标替代主观评分。

| Metric | Calculation | Acceptable Threshold |
|--------|-------------|------------------------|
| Format compliance | Percentage of outputs matching the specified template | ≥95% |
| Key‑element coverage | All mandatory information points are present | 100% |
| Factual hallucination rate | Ratio of unverified assertions to total assertions | ≤2% |
| Instruction‑following deviation | Count of explicit instructions that were not followed | 0 |

| 指标 | 计算方式 | 可接受阈值 |
|------|----------|------------|
| 格式合规率 | 输出符合指定模板的比例 | ≥95% |
| 关键要素覆盖率 | 所有必需信息点均已出现 | 100% |
| 事实幻觉率 | 未经核实的断言占全部断言的比例 | ≤2% |
| 指令遵循偏差 | 未被遵循的显式指令数量 | 0 |

---

## Prefix Tags

## 前缀标签

Standardised prefixes help the model parse your prompts more reliably, especially in long contexts.

标准化的前缀标签能帮助模型更可靠地解析提示词，尤其是在长上下文中。

- `[Hard Constraint]` – Non‑negotiable boundary.
- `[Hard Constraint]`（硬约束）– 不可协商的边界。
- `[Important]` – Allocate extra attention to this part.
- `[Important]`（重要）– 对该部分分配额外关注。
- `[Optional]` – Nice‑to‑have improvements.
- `[Optional]`（可选）– 锦上添花的改进。
- `[Background]` – Context for reference, not mandatory.
- `[Background]`（背景）– 供参考的上下文，非强制。
- `[Counter‑example]` – Explicitly show what the model should not do.
- `[Counter‑example]`（反例）– 明确展示模型不应做什么。

---

## License

## 许可证

MIT – feel free to adapt for your own projects.

MIT 许可——欢迎改编用于你自己的项目。
