# Prompt Engineering Guide

A practical, engineering-oriented guide for designing prompts in AI Agent, code generation, and content processing scenarios. This repository collects battle-tested patterns, iteration workflows, and domain-specific strategies.

---

## Table of Contents

- [Introduction](#introduction)
- [Prompt Structure](#prompt-structure)
- [Design Principles](#design-principles)
- [Iteration Workflow](#iteration-workflow)
- [Web Q&A Scenarios](#web-qa-scenarios)
- [Large-Project Development (OpenCode, Cursor, etc.)](#large-project-development-opencode-cursor-etc)
- [Evaluation Metrics](#evaluation-metrics)
- [Prefix Tags](#prefix-tags)

---

## Introduction

Prompt is the primary instruction interface between users and AI models. Its quality directly determines output accuracy, consistency, and safety. This guide avoids subjective aesthetics and focuses on measurable practices that work across tasks.

---

## Prompt Structure

A complete prompt consists of five independent modules. Compose them as needed.

| Module | Purpose | Example |
|--------|---------|---------|
| Role | Define expertise and perspective | "You are a backend architect with deep knowledge of Java concurrency." |
| Task | State what to accomplish explicitly | "Design a rate limiter pseudocode for 10M QPS." |
| Context | Provide background, constraints, or prior logs | "Current system uses Spring Cloud, no gateway protection." |
| Output Spec | Specify format, length, structure, style | "Output as Markdown table with columns: solution, pros/cons, applicable conditions." |
| Interaction Guide | Request step-by-step reasoning or intermediate outputs | "First analyze peak traffic patterns, then give your solution, finally explain fallback strategies." |

Keep each module single-responsibility. Merge or omit modules for simple tasks.

---

## Design Principles

### 1. Positive instructions over negative constraints

Models follow "do" statements better than "don't" statements.

- ❌ "Do not use casual tone, do not omit technical details."
- ✅ "Use technical writing style; list all parameters and edge conditions."

### 2. Explicit reasoning paths

For complex tasks, force the model to output its reasoning chain.

```
Answer in this order:
1. List all input constraints.
2. Propose at least two alternatives.
3. Give your recommendation with justification.
```

### 3. Few-shot example rules

- Use 3–6 examples.
- Balance categories in classification tasks.
- Sort examples from easiest to hardest (or from typical to edge cases) to mitigate position bias.

### 4. Anchor reset in long conversations

Transformer models pay more attention to the beginning and end of the context. In conversations longer than 4000 tokens, repeat the core objective at the end of each new query. Use prefixes like `[Core Goal]` or `[Hard Constraint]` to reinforce key points.

---

## Iteration Workflow

Follow this standard loop for prompt tuning. Log every change.

| Step | Action | Record |
|------|--------|--------|
| Baseline | Write initial prompt based on task decomposition | Version number, expected sample output |
| Smoke test | Run 3 typical inputs, check output completeness | Pass/fail items, error type labels |
| Root-cause analysis | Classify errors into: semantic misunderstanding, format deviation, information omission | Error distribution |
| Targeted fix | Modify only one module at a time; prioritize instruction rephrasing | Before/after comparison |
| Regression | Re-run the same test set, measure improvement | Accuracy and format compliance deltas |
| Release | Finalise prompt and write usage documentation | Boundary conditions, known limitations |

---

## Web Q&A Scenarios

When the model must answer based on provided web pages or search results, force it to stay within the given content.

### Context injection template

```
[Available Content]
(paste plain-text content here, keep paragraph markers)
[Content End]

[User Question]
(actual question)

[Response Rules]
- Answer only using the [Available Content].
- If the content does not contain the answer, respond exactly: "Cannot confirm based on available materials." Do not add external knowledge.
- When citing specific paragraphs, append the paragraph number in parentheses.
```

### Uncertainty handling

Add a confidence self-check:

"At the end of your answer, label confidence as High / Medium / Low. For Medium or Low, explicitly list what key information is missing."

### Multi-turn context management

In follow-up questions, output a "summary of evidence used" in each turn.

Example format:  
`Answer: ... Evidence summary: paragraphs 3, 5.`

---

## Large-Project Development (OpenCode, Cursor, etc.)

When using AI coding agents (OpenCode, Cursor, Devin) on repositories >5000 LOC with multiple modules, prompt design must shift from single-task to project-iteration mode. These tools decompose prompts into sub-tasks across multiple context windows, so you need stable boundaries.

### 1. Project-level system instructions

Place a persistent instruction file (e.g., `.opencode/instructions.md`) in the project root. It defines global behaviour for all sessions.

Required entries:

- Language and version (e.g., Python 3.11, Java 17)
- Package manager and registry (e.g., pnpm, npm registry)
- Test framework and coverage target (e.g., Jest, Pytest, ≥80% line coverage)
- Code style rules (e.g., ESLint, Google Java Format)
- Documentation language and comment density (at least one comment per function)

### 2. Module-level task decomposition

Do not submit the entire project at once. Break down by module, but include dependency and contract declarations.

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

When updating existing code, lock the scope and require impact analysis.

```
[Modification Target]
Replace the email regex in src/utils/validator.ts with pattern B (support new TLDs).

[Impact Analysis]
- Callers: registration, profile update, password reset.
- Check compatibility of these three call sites with the new regex; update their unit tests accordingly.

[Regression Check]
- Run `npm run test:validator` and ensure no new failures (except intentionally corrected cases).
- Output a before-after diff list covering at least 5 edge cases (e.g., .io, .ai).
```

### 4. Debug prompts

Attach the full stack trace and the last relevant commit hash. Require root-cause analysis and fix comparison.

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

### 5. Do-not-do list for large projects

- Avoid vague requests like "optimise the code" – specify performance, readability, or memory.
- Do not mix frontend UI changes with database migration instructions in one prompt – agents tend to modify too many files at once.
- If your project uses custom scripts or Makefiles, explicitly state the build and test commands.
- For agents with function-calling (OpenCode), explicitly allow or disallow terminal execution, file writes, and network access.

---

## Evaluation Metrics

Use these four hard metrics instead of subjective scoring.

| Metric | Calculation | Acceptable Threshold |
|--------|-------------|----------------------|
| Format compliance | Percentage of outputs matching the specified template | ≥95% |
| Key-element coverage | All mandatory information points are present | 100% |
| Factual hallucination rate | Ratio of unverified assertions to total assertions | ≤2% |
| Instruction-following deviation | Count of explicit instructions that were not followed | 0 |

---

## Prefix Tags

Standardised prefixes help the model parse your prompts more reliably, especially in long contexts.

- `[Hard Constraint]` – Non-negotiable boundary.
- `[Important]` – Allocate extra attention to this part.
- `[Optional]` – Nice-to-have improvements.
- `[Background]` – Context for reference, not mandatory.
- `[Counter-example]` – Explicitly show what the model should not do.

---

## License

MIT – feel free to adapt for your own projects.

---

# 提示词工程指南

一本面向 AI 智能体、代码生成和内容处理场景的工程化提示词设计实战指南。本仓库汇集了经过实战验证的模式、迭代工作流和领域特定策略。

---

## 目录

- [简介](#简介)
- [提示词结构](#提示词结构)
- [设计原则](#设计原则)
- [迭代工作流](#迭代工作流)
- [Web 问答场景](#web-问答场景)
- [大型项目开发（OpenCode、Cursor 等）](#大型项目开发opencodecursor-等)
- [评估指标](#评估指标)
- [前缀标签](#前缀标签)

---

## 简介

提示词是用户与 AI 模型之间的主要指令接口，其质量直接决定输出的准确性、一致性和安全性。本指南不讨论主观审美，只聚焦于跨任务可复现、可度量的实践方法。

---

## 提示词结构

一个完整的提示词由五个相互独立的模块组成，可按需组合。

| 模块 | 作用 | 示例 |
|------|------|------|
| 角色 (Role) | 定义专业领域与视角 | "你是一位对 Java 并发有深入研究的后端架构师。" |
| 任务 (Task) | 明确陈述要完成的目标 | "请设计一个支持 1000 万 QPS 的限流器伪代码。" |
| 上下文 (Context) | 提供背景、约束或历史日志 | "当前系统使用 Spring Cloud，无网关保护。" |
| 输出规范 (Output Spec) | 规定格式、长度、结构、风格 | "以 Markdown 表格输出，列包括：方案、优缺点、适用条件。" |
| 交互指引 (Interaction Guide) | 要求逐步推理或输出中间结果 | "先分析峰值流量模式，再给出方案，最后说明降级策略。" |

每个模块保持单一职责。对于简单任务，可以合并或省略部分模块。

---

## 设计原则

### 1. 正面指令优于负面约束

模型对"要做什么"的指令执行效果优于"不要做什么"。

- ❌ "不要使用随意语气，不要遗漏技术细节。"
- ✅ "使用技术写作风格，列出所有参数和边界条件。"

### 2. 明确的推理路径

对于复杂任务，强制模型输出其推理链条。

```
请按以下顺序作答：
1. 列出所有输入约束。
2. 提出至少两种备选方案。
3. 给出你的推荐并说明理由。
```

### 3. 少样本示例规则

- 使用 3–6 个示例。
- 分类任务中保持类别均衡。
- 示例按从易到难排序（或从典型到边界），以减轻位置偏差。

### 4. 长对话中的锚点重置

Transformer 模型对上下文的开头和结尾更敏感。在超过 4000 token 的对话中，应在每条新提问的结尾重复核心目标，并使用 `[Core Goal]` 或 `[Hard Constraint]` 等前缀强化关键点。

---

## 迭代工作流

提示词调优遵循以下标准循环，并记录每一次变更。

| 步骤 | 动作 | 记录 |
|------|------|------|
| 基线 | 基于任务拆解编写初始提示词 | 版本号、预期样例输出 |
| 冒烟测试 | 运行 3 个典型输入，检查输出完整性 | 通过/失败项、错误类型标签 |
| 根因分析 | 将错误归类为：语义误解、格式偏差、信息遗漏 | 错误分布 |
| 定向修复 | 一次只修改一个模块，优先调整指令措辞 | 修改前后对比 |
| 回归测试 | 重新运行同一测试集，度量改进幅度 | 准确率与格式合规度差值 |
| 发布 | 定稿提示词并编写使用文档 | 边界条件、已知限制 |

---

## Web 问答场景

当模型必须基于给定的网页或搜索结果作答时，强制其严格限定在给定内容范围内。

### 上下文注入模板

```
[可用内容]
（在此粘贴纯文本内容，保留段落标记）
[内容结束]

[用户问题]
（实际问题）

[回答规则]
- 只能使用 [可用内容] 作答。
- 如果内容中不含答案，请原样回复："根据现有材料无法确认。" 不要补充外部知识。
- 引用具体段落时，在括号中标注段落编号。
```

### 不确定性处理

添加置信度自检：

"在回答结尾标注置信度：高 / 中 / 低。若为中等或低，请明确列出缺失的关键信息。"

### 多轮上下文管理

在追问中，每轮都输出"已使用证据摘要"。

示例格式：  
`回答：…… 证据摘要：第 3、5 段。`

---

## 大型项目开发（OpenCode、Cursor 等）

当在超过 5000 行代码、多模块的仓库中使用 AI 编程智能体（OpenCode、Cursor、Devin）时，提示词设计必须从单任务模式转向项目迭代模式。这些工具会把提示词分解为跨多个上下文窗口的子任务，因此你需要稳定的边界。

### 1. 项目级系统指令

在项目根目录放置一份持久化指令文件（如 `.opencode/instructions.md`），为所有会话定义全局行为。

必需条目：

- 语言与版本（如 Python 3.11、Java 17）
- 包管理器与镜像源（如 pnpm、npm registry）
- 测试框架与覆盖率目标（如 Jest、Pytest，行覆盖率 ≥80%）
- 代码风格规则（如 ESLint、Google Java Format）
- 文档语言与注释密度（每个函数至少一条注释）

### 2. 模块级任务拆解

不要一次性提交整个项目。按模块拆解，但需包含依赖与契约声明。

```
[当前任务]
为用户认证模块实现登录端点（/api/v1/login）。

[模块边界]
- 只处理登录；不要改动注册或密码重置。
- 依赖：数据库连接池（已存在，路径：src/db/pool.ts）。
- 对外接口：返回 JWT 令牌 + 用户基本信息。

[上游契约]
- 前端期望：{ code: number, data: { token: string, expires_in: number }, message: string }。
- 错误码范围：1001–1005。

[下游依赖]
- 用户表结构：src/models/user.sql。
- Redis 缓存登录失败次数，键格式：login_fail:{ip}:{username}。

[验收标准]
- 连续 3 次失败后锁定账户 15 分钟。
- 单元测试覆盖：正常登录、密码错误、账户已锁定。
- 中位响应时间 < 200ms。

[输出约束]
- 只修改 src/controllers/auth.ts 和 src/services/auth.service.ts。
- 不要改动配置文件或新增依赖。
- 代码完成后，提供一条 pytest 命令用于运行测试。
```

### 3. 修改请求（变更提示词）

更新既有代码时，锁定修改范围并要求做影响分析。

```
[修改目标]
将 src/utils/validator.ts 中的邮箱正则替换为模式 B（支持新顶级域名）。

[影响分析]
- 调用方：注册、资料更新、密码重置。
- 检查这三个调用点与新正则的兼容性，并相应更新其单元测试。

[回归检查]
- 运行 `npm run test:validator`，确保无新增失败（除有意修正的用例）。
- 输出覆盖至少 5 个边界用例（如 .io、.ai）的修改前后差异列表。
```

### 4. 调试提示词

附上完整的堆栈信息与最后一次相关的提交哈希，要求进行根因分析与修复方案对比。

```
[错误症状]
生产环境报错：TypeError: Cannot read property 'id' of undefined，堆栈指向 src/middleware/auth.ts:48。

[上下文]
该文件在提交 7f3a2b1（令牌解析逻辑）中被重构。

[诊断要求]
1. 列出至少两条可能导致该错误的代码路径。
2. 针对每条路径说明触发条件。
3. 推荐一种修复方案并提供补丁片段。
4. 说明防止类似问题的防御性措施。
```

### 5. 大型项目的禁忌清单

- 避免"优化代码"这类模糊请求——需明确是性能、可读性还是内存。
- 不要在同一个提示词中混合前端 UI 修改与数据库迁移指令——智能体会一次性改动过多文件。
- 如果项目使用自定义脚本或 Makefile，请明确说明构建与测试命令。
- 对于支持函数调用的智能体（OpenCode），请明确允许或禁止终端执行、文件写入和网络访问。

---

## 评估指标

用以下四个硬指标替代主观评分。

| 指标 | 计算方式 | 可接受阈值 |
|------|----------|------------|
| 格式合规率 | 输出符合指定模板的比例 | ≥95% |
| 关键要素覆盖率 | 所有必需信息点均已出现 | 100% |
| 事实幻觉率 | 未经核实的断言占全部断言的比例 | ≤2% |
| 指令遵循偏差 | 未被遵循的显式指令数量 | 0 |

---

## 前缀标签

标准化的前缀标签能帮助模型更可靠地解析提示词，尤其是在长上下文中。

- `[Hard Constraint]`（硬约束）– 不可协商的边界。
- `[Important]`（重要）– 对该部分分配额外关注。
- `[Optional]`（可选）– 锦上添花的改进。
- `[Background]`（背景）– 供参考的上下文，非强制。
- `[Counter-example]`（反例）– 明确展示模型不应做什么。

---

## 许可证

MIT 许可——欢迎改编用于你自己的项目。
