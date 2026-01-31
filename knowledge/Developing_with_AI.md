# Developer Guide: Mastering Cursor IDE Agents & Models

## 1. Understanding Agent Mode (Composer)
Agent mode is the "autonomous" version of the AI. Unlike a standard chat that just gives you code to copy, the Agent can:
* **Multi-file Editing:** Apply changes across your entire project simultaneously.
* **Terminal Execution:** Run tests, install packages, and fix errors based on the output.
* **Context Discovery:** Search your codebase to find where a specific logic resides without you telling it.

### The Sub-Modes
* **Agent (Default):** Executes changes immediately. Use for building features.
* **Plan Mode:** Researches and writes a step-by-step Markdown plan first. **Best for complex refactors.**
* **Ask Mode:** Read-only. Use this when you want to explore the code without risking accidental changes.

---

## 2. The "Model" Breakdown
A **Model** is the underlying LLM (Large Language Model) processing your request. Think of it as choosing the right "brain" for the task.



### Comparison Table
| Model | Strength | Best For... |
| :--- | :--- | :--- |
| **Claude 3.5 Sonnet** | Best overall coding logic. | Feature development, debugging. |
| **o1-preview / o1-mini** | Deep reasoning / "Thinking". | Complex algorithms, "impossible" bugs. |
| **GPT-4o** | Versatility and instruction following. | Documentation, boilerplate, unit tests. |
| **Cursor Small** | Instant speed. | Quick syntax fixes, small edits. |

---

## 3. Why Switch? (The Developer's Strategy)
Even with a company-paid account, choosing the right model preserves your **productivity** and **usage limits**:

1.  **Latency (Speed):** Don't use a "Thinking" model (o1) for a CSS fix; the 30-second wait will break your flow. Use **Sonnet** or **Small**.
2.  **Context Window:** If you need the AI to understand a bug spanning 10 files, use **Claude 3.5** or **Gemini 1.5 Pro** for their massive "memory."
3.  **Rate Limits:** High-tier models often have "fast-request" quotas. Save your "Fast" Claude 3.5 uses for the hardest parts of your sprint.
4.  **Reasoning vs. Repetition:** Use **GPT-4o** for repetitive tasks like writing 20 similar unit tests, but switch to **o1** when the logic of those tests is genuinely difficult to solve.

---

## 4. Pro-Tips
* **Rules for the AI:** Create a `.cursorrules` file in your root directory to define your project's tech stack (e.g., React, TypeScript) so the Agent doesn't have to guess.
* **Terminal Integration:** If the Agent runs a command that fails, it will read the error and try to self-correct. Let it finish—it's often faster than fixing it yourself.