# Prompt Engineering Guide

A practical, engineering-oriented guide for designing prompts in AI Agent, code generation, and content processing scenarios. This repository collects battle-tested patterns, iteration workflows, and domain-specific strategies.

---

## Table of Contents
- [Introduction](#introduction)
- [Prompt Structure](#prompt-structure)
- [Design Principles](#design-principles)
- [Iteration Workflow](#iteration-workflow)
- [Web Q&A Scenarios](#web-qa-scenarios)
- [Large‑Project Development (OpenCode, Cursor, etc.)](#large-project-development-opencode-cursor-etc)
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
| Interaction Guide | Request step‑by‑step reasoning or intermediate outputs | "First analyze peak traffic patterns, then give your solution, finally explain fallback strategies." |

Keep each module single‑responsibility. Merge or omit modules for simple tasks.

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

### 3. Few‑shot example rules

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
| Root‑cause analysis | Classify errors into: semantic misunderstanding, format deviation, information omission | Error distribution |
| Targeted fix | Modify only one module at a time; prioritize instruction rephrasing | Before/after comparison |
| Regression | Re‑run the same test set, measure improvement | Accuracy and format compliance deltas |
| Release | Finalise prompt and write usage documentation | Boundary conditions, known limitations |

---

## Web Q&A Scenarios

When the model must answer based on provided web pages or search results, force it to stay within the given content.

### Context injection template

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

Add a confidence self‑check:

"At the end of your answer, label confidence as High / Medium / Low. For Medium or Low, explicitly list what key information is missing."

### Multi‑turn context management

In follow‑up questions, output a "summary of evidence used" in each turn.

Example format:  
`Answer: ... Evidence summary: paragraphs 3, 5.`

---

## Large‑Project Development (OpenCode, Cursor, etc.)

When using AI coding agents (OpenCode, Cursor, Devin) on repositories >5000 LOC with multiple modules, prompt design must shift from single‑task to project‑iteration mode. These tools decompose prompts into sub‑tasks across multiple context windows, so you need stable boundaries.

### 1. Project‑level system instructions

Place a persistent instruction file (e.g., `.opencode/instructions.md`) in the project root. It defines global behaviour for all sessions.

Required entries:
- Language and version (e.g., Python 3.11, Java 17)
- Package manager and registry (e.g., pnpm, npm registry)
- Test framework and coverage target (e.g., Jest, Pytest, ≥80% line coverage)
- Code style rules (e.g., ESLint, Google Java Format)
- Documentation language and comment density (at least one comment per function)

### 2. Module‑level task decomposition

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
- Output a before‑after diff list covering at least 5 edge cases (e.g., .io, .ai).
```

### 4. Debug prompts

Attach the full stack trace and the last relevant commit hash. Require root‑cause analysis and fix comparison.

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

- Avoid vague requests like "optimise the code" – specify performance, readability, or memory.
- Do not mix frontend UI changes with database migration instructions in one prompt – agents tend to modify too many files at once.
- If your project uses custom scripts or Makefiles, explicitly state the build and test commands.
- For agents with function‑calling (OpenCode), explicitly allow or disallow terminal execution, file writes, and network access.

---

## Evaluation Metrics

Use these four hard metrics instead of subjective scoring.

| Metric | Calculation | Acceptable Threshold |
|--------|-------------|------------------------|
| Format compliance | Percentage of outputs matching the specified template | ≥95% |
| Key‑element coverage | All mandatory information points are present | 100% |
| Factual hallucination rate | Ratio of unverified assertions to total assertions | ≤2% |
| Instruction‑following deviation | Count of explicit instructions that were not followed | 0 |

---

## Prefix Tags

Standardised prefixes help the model parse your prompts more reliably, especially in long contexts.

- `[Hard Constraint]` – Non‑negotiable boundary.
- `[Important]` – Allocate extra attention to this part.
- `[Optional]` – Nice‑to‑have improvements.
- `[Background]` – Context for reference, not mandatory.
- `[Counter‑example]` – Explicitly show what the model should not do.

---

## License

MIT – feel free to adapt for your own projects.
