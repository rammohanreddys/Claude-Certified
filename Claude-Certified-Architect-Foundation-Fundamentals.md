# Claude Architect Foundation Fundamentals:

**Core Exam Domains Covered**

**Domain 1:** **Agentic Architecture & Orchestration** — Designing autonomous loops, coordinator–subagent systems, subagent context sharing, task decomposition, Agent SDK hooks, and session state management (resumption and forking).

**Domain 2:** **Tool Design & MCP Integration** — Designing effective tool interfaces and descriptions, handling structured error responses, distributing tools across agents, and integrating Model Context Protocol (MCP) servers into Claude Code.

**Domain 3:** **Claude Code Configuration & Workflows** — Configuring CLAUDE memory files (hierarchy, scoping, modularity), building custom commands and Claude Skills, applying path-specific rules, choosing between Plan Mode and direct execution, and integrating Claude Code into CI/CD pipelines.

**Domain 4:** **Prompt Engineering & Structured Output** — Writing high-precision prompts, using few-shot prompting, enforcing JSON Schemas, building self-correcting extraction pipelines, batch processing, and designing multi-instance/multi-pass review architectures.

**Domain 5:** **Context Management & Reliability** — Managing context across long-running conversations and large codebases, handling uncertainty, preserving information provenance, and designing escalation, human-review, and ambiguity-resolution workflows.

## Domain 1: Agentic Architecture & Orchestration:

This PDF presentation (**Domain 1: Agentic Architecture & Orchestration** by Ankit Mistry) covers how to build, coordinate, control, and manage AI agents (specifically using Claude) beyond simple single-turn chatbots.

The presentation is organized into seven core sections:

---

### 1. The Agentic Loop (Lecture 1.1)

* **Chatbot vs. Agent:** A standard chatbot takes a prompt, replies once, and finishes. An agent operates in a continuous **loop** (`Perceive → Reason → Act → Observe`), calling tools and using the results to decide its next action until the task is complete.
* **The `stop_reason` Signal:** Every model response includes a `stop_reason` field that controls the loop:
* `tool_use` → Run the requested tool, feed the output back to the model as a `tool_result`, and continue looping.
* `end_turn` → The agent is finished; exit the loop.


* **Model-Driven vs. Hardcoded:** Let the model dynamically choose which tools to call based on context rather than hardcoding brittle `if/else` decision trees.
* **Three Anti-Patterns to Avoid:** Ignoring `stop_reason`, failing to set an exit condition (causing infinite loops), and failing to feed tool results back to the model.

---

### 2. Coordinator-Subagent Pattern (Lecture 1.2)

* **Why Split Work:** Cramming research, analysis, and writing into one agent's context degrades quality. Splitting work across specialized subagents keeps each agent's context clean and focused.
* **Hub-and-Spoke Architecture:** Subagents do not communicate directly with each other; everything flows through a central **Coordinator**. The coordinator selects and routes tasks, manages errors, synthesizes results, and triggers **iterative refinement** (sending work back if quality falls short).
* **Execution Modes:** Run subagents in **parallel** when tasks are independent, and **sequentially** when one step depends on the output of the previous step.
* **Narrow-Decomposition Trap:** Ensure the subtasks add up to the full user request—splitting too narrowly leaves parts of the original question unanswered.

---

### 3. Spawning & Context Passing (Lecture 1.3)

* **Spawning with the `Task` Tool:** A coordinator delegates work by calling a `Task` tool, which launches a fresh subagent with its own loop and isolated workspace. Each subagent is defined by four properties: `description`, `prompt`, `tools` (scoped strictly to what that subagent needs), and an optional `model`.
* **No Automatic Inheritance:** Subagents start completely blank—they do not inherit the coordinator's conversation history or another subagent's findings.
* **Best Practices for Handoffs:**
* Inject all required background and **complete, structured findings** (not vague one-line summaries) directly into the subagent's prompt.
* Give **goals and quality bars** (the *what*), not rigid step-by-step scripts (the *how*).



---

### 4. Enforcement & Handoffs (Lecture 1.4)

* **Programmatic vs. Prompt Rules:** Instructions in prompts are *probabilistic* (usually followed), whereas rules enforced in code are *deterministic* (guaranteed 100% of the time).
* **Prerequisite Gates:** If breaking a rule causes serious harm (e.g., issuing a refund before verifying identity), enforce a code gate that blocks the tool from running until the prerequisite condition is met.
* **Human Escalation:** When a case is too risky or ambiguous, stop and escalate to a human with a structured handoff summary (customer details, issue, actions already taken, and recommended next step).

---

### 5. Agent SDK Hooks (Lecture 1.5)

* **What Hooks Are:** Code functions that execute automatically around tool calls, providing deterministic control:
* **`PreToolUse`:** Runs *before* a tool executes. It can inspect, modify inputs, or block dangerous calls (e.g., blocking any refund over $500 and routing it to a human).
* **`PostToolUse`:** Runs *after* a tool executes, before the model reads the output. Useful for logging or cleaning/normalizing messy data.



---

### 6. Task Decomposition Strategies (Lecture 1.6)

* **Choosing the Right Pattern:**
* **Fixed, known steps** → Use **Prompt Chaining** (a predictable pipeline where Step 1's output feeds Step 2).
* **Dynamic, unknown steps** (e.g., debugging or open-ended research) → Use **Adaptive Decomposition** with an **Orchestrator-Workers** setup, discovering subtasks as findings emerge.


* **Multi-Level Passes:** Complex reviews (like code audits) often combine both: run independent **per-file passes in parallel**, followed by a **cross-file pass** to check how everything connects.

---

### 7. Session State Management (Lecture 1.7)

* **Resume & Continue:** Sessions save conversation history, tool calls, and results on disk (`--resume` or `--continue`). Be careful of the **stale-transcript trap**: if external files or data changed while paused, a resumed session will still trust outdated tool results.
* **Forking (`fork_session`):** Creates a safe copy of a session so you can experiment on a disposable branch without polluting your main ("trunk") session.
* **When to Start Fresh:** If a session's context becomes stale or cluttered, summarize the essential facts and start a clean session rather than resuming.

---

Would you like to dive deeper into any specific lecture, walk through a practical code example of these patterns, or test your knowledge with some practice questions on this material?
