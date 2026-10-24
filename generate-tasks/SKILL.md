---
name: generate-tasks
description: Generates a step-by-step implementation task list in Markdown from an existing PRD file, in two phases (high-level parent tasks first, then sub-tasks after the user says "Go"), grounded in the current codebase and its architecture rules. Use this skill whenever the user points to a PRD (e.g. tasks/prd-*.md) and wants tasks, a task list, an implementation plan, a breakdown, or says "genera las tareas del PRD", "break this PRD down", or "plan the implementation". Saves tasks/tasks-[prd-file-name].md. Usually run after create-prd and before process-task-list.
---

# Generate Tasks from a PRD

Turn a PRD into a checklist a **junior developer** can follow, aware of the existing codebase. The two-phase flow exists so the user can correct the high-level plan cheaply before you spend effort on detail.

## Process

1. **Get the PRD.** The user names a specific PRD file. If they don't, look in `tasks/` for `prd-*.md` and ask which one if there's more than one.
2. **Analyze the PRD.** Read the functional requirements, user stories, non-goals, and technical considerations.
3. **Assess the current state of the codebase.** Before planning, look at the existing infrastructure, architectural patterns, and conventions, and find existing components, utilities, and files that can be reused or must be modified. Plans that ignore what already exists duplicate code and break conventions.
   - Read the project's architecture guidance if present: `CLAUDE.md` and anything under `.claude/rules/` (for example `feature-sliced-design.md` or `project-architecture.md`). Follow those folder structures and rules when deciding where new files belong. If a file doesn't exist, skip it.
4. **Phase 1: parent tasks.** Create the output file with only the main high-level tasks. Use judgment on the count; roughly 4-7 is typical, fewer for small features.
5. **Pause for confirmation.** Show the parent tasks and say something like: "I have generated the high-level tasks based on the PRD. Ready to generate the sub-tasks? Respond with 'Go' to proceed." Then wait. Do not continue until the user answers "Go" (or gives changes, which you apply first).
6. **Phase 2: sub-tasks.** Break each parent task into small, actionable sub-tasks that logically follow from it and cover the implementation details implied by the PRD. Follow existing codebase patterns where relevant without being constrained by them. Include test sub-tasks where applicable.
7. **Identify relevant files.** List files to create or modify, with matching test files, each with a one-line reason.
8. **Save** the final document.

## Output

- Format: Markdown
- Location: `tasks/` at the project root
- Filename: `tasks-[prd-file-name].md`, where the name matches the input PRD's base name (`prd-user-profile-editing.md` becomes `tasks-prd-user-profile-editing.md`)

Use exactly this structure:

```markdown
## Relevant Files

- `path/to/potential/file1.ts` - Why this file is relevant (e.g., main component for this feature).
- `path/to/file1.test.ts` - Unit tests for `file1.ts`.
- `lib/utils/helpers.ts` - Utility functions needed for calculations.

## Tasks

- [ ] 1.0 Parent Task Title
  - [ ] 1.1 Sub-task description
  - [ ] 1.2 Sub-task description
- [ ] 2.0 Parent Task Title
  - [ ] 2.1 Sub-task description
- [ ] 3.0 Parent Task Title (may have no sub-tasks if purely structural or configuration)
```

Do not start implementing anything. When done, tell the user the file path and that the `process-task-list` skill can execute it.
