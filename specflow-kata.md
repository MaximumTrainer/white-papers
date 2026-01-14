# LLM Spec Driven Kata: Task Prioritisation Engine 

## 1. Overview

### 1.1 Problem statement

You will build a **Task Prioritisation Engine** that accepts a list of tasks and returns them sorted by priority according to a **deterministic, explicitly specified scoring model**.

The kata is **spec first** and **LLM assisted**:

- You start by creating a clear, testable **specification**.
- You use the spec to generate the implementation.
- You iteratively refine the spec and implementation through **reviews**, **automated tests**, and **durability assessments**.

### 1.2 Learning objectives

By completing this kata, you will learn how to:

- Write a **clear, testable LLM driven specification**.
- Use **AWS Kiro** or **GitHub Copilot Spex Kit** to:
  - Generate code from a spec.
  - Generate and maintain automated tests from the spec.
- Run **structured review loops** on:
  - The specification.
  - The implementation.
- Perform **durability reviews**:
  - Identify brittle assumptions.
  - Design stress and failure mode tests.
  - Improve long term maintainability.

### 1.3 Target audience and difficulty

- **Audience:** Intermediate developers and tech leads familiar with basic testing and code review.
- **Difficulty:** Intermediate to upper intermediate.
- **Estimated time:** 2–4 focused hours, depending on depth of iteration.

### 1.4 Tools and prerequisites

- **Required:** Access to **AWS Kiro** or **GitHub Copilot Spex Kit**, or any equivalent LLM spec tooling.
- **Language:** You may use any programming language commonly supported by your environment (e.g., TypeScript, Python, Java, C#, Go).
- **Knowledge:** Basic familiarity with:
  - Writing unit tests.
  - Reading and writing JSON like structures.
  - Running automated test suites.

---

## 2. Domain description

### 2.1 Concept: Task Prioritisation Engine

You have a list of **tasks**. Each task has:

- `title`: short human readable description.
- `urgency`: integer rating, 1–5 (5 = very urgent).
- `importance`: integer rating, 1–5 (5 = very important).
- `dueDate`: optional ISO date string (e.g., `"2025-12-31"`).

The Task Prioritisation Engine must:

- **Validate** input tasks.
- **Compute a priority score** for each task using a deterministic scoring model.
- **Sort tasks** from highest to lowest priority.
- **Provide deterministic tie breaking rules** (e.g., earlier due date wins).

You will:

- Specify the exact **priority score formula**.
- Define **behaviours** for:
  - Missing or invalid fields.
  - Ties in score.
  - Large input sets.

---

## 3. Specification first workflow

Your first goal is to create a **high quality specification** for the engine, before writing any code.

### 3.1 Initial spec creation

Use your LLM tool to generate an initial spec with the following prompt (adapt as needed):

```text
Create a complete specification for a “Task Prioritisation Engine”.

The engine must:
- Accept a list of tasks, where each task has:
  - title (string, required, non-empty)
  - urgency (integer 1–5, required)
  - importance (integer 1–5, required)
  - dueDate (optional ISO 8601 date string, e.g. "2025-12-31")

- Define a deterministic scoring model that combines urgency, importance, and (if present) dueDate.
- Specify exactly how the score is calculated, including weights and any date-related adjustments.
- Define rules for:
  - Missing or invalid fields
  - Out-of-range numeric values
  - Malformed dates
  - Ties in score (tie-breaking strategy must be deterministic)

- Describe how tasks are returned:
  - Sorting order
  - Stability of sort (what happens when all tie-breakers are equal?)

- Specify at least 10 concrete, testable scenarios (including edge cases).

The spec must:
- Be implementation-agnostic (no language/framework assumptions).
- Express behaviours in a way that can be fully covered by automated tests.
- Be unambiguous and deterministic.

Output the specification in structured Markdown with:
- A “Domain model” section
- A “Scoring model” section
- A “Validation rules” section
- A “Sorting rules” section
- An “Examples and edge cases” section
- An “Acceptance criteria” section

