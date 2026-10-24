---
name: process-task-list
description: Executes and maintains a Markdown task list (such as tasks/tasks-prd-*.md) one main task at a time, ticking checkboxes, keeping the Relevant Files section accurate, and pausing for user approval after each main task, without ever committing code. Use this skill whenever the user asks to implement, work through, continue, or resume a task list or PRD tasks, says "empieza con la tarea 1.0", "siguiente tarea", "implement the tasks", or points at a tasks-*.md file, even if they don't name the skill. Usually run after generate-tasks.
---

# Process Task List

Work through a task list file to implement a PRD, keeping the file as the live record of progress. Pausing at main-task boundaries keeps the user in control of a long implementation; the checklist keeps the state recoverable across sessions.

## Before starting

Open the task list, find the next unchecked sub-task, and confirm the rest of the context (PRD, architecture rules in `CLAUDE.md` / `.claude/rules/`) as needed. Always know which sub-task is next before touching code.

## Implementation flow

- Work through the sub-tasks of the current **main task** in order.
- After finishing each **sub-task**, immediately change `[ ]` to `[x]` in the file, then continue to the next sub-task of the same main task. Update the file after any significant work, not just at the end.
- When **all** sub-tasks under a parent are `[x]`:
  1. **Clean up**: remove temporary files and temporary code (debug logs, scratch scripts, commented-out experiments).
  2. Mark the **parent task** `[x]`.
  3. **Stop and wait** for the user's go-ahead ("yes" / "y" / "go") before starting the next main task. Briefly summarize what was done.
- Never start the next main task without that explicit approval.

## Keeping the file accurate

- Add new tasks or sub-tasks as they emerge during implementation (missing steps, discovered dependencies).
- Maintain the **Relevant Files** section: list every file created or modified, each with a one-line description of its purpose. Keep it current, since the user relies on it to review changes.

## Do not commit

Do not run `git commit` (or push) after finishing sub-tasks or main tasks. The user commits manually. You may mention that a main task is ready to be committed.
