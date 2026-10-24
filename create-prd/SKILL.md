---
name: create-prd
description: Creates a Product Requirements Document (PRD) in Markdown from a short feature idea, by first asking clarifying questions with lettered/numbered options and then writing a PRD a junior developer can implement. Use this skill whenever the user wants a PRD, product spec, feature spec, requirements doc, or says things like "write a PRD for...", "spec out this feature", "I have an idea for a feature", "necesito un PRD", or "documenta los requisitos de esta funcionalidad", even if they don't use the word PRD. Saves the result as tasks/prd-[feature-name].md and never starts implementing. Typically followed by the generate-tasks skill.
---

# Create PRD

Turn a brief feature request into a clear, actionable PRD. The reader is a **junior developer**, so requirements must be explicit and unambiguous, with little jargon. The PRD defines the *what* and *why*; the developer works out the *how*.

## Process

1. **Receive the prompt.** The user gives a short description of a feature.
2. **Ask clarifying questions before writing anything.** A PRD written from a one-line prompt hides assumptions that surface later as rework, so this step is not optional. Adapt the questions to the request and skip areas the prompt already answers. Offer options as lettered or numbered lists so the user can reply tersely ("1A, 2C, 3B").
3. **Generate the PRD** from the prompt plus the answers, using the structure below.
4. **Save it** as `prd-[feature-name].md` in the `tasks/` directory at the project root (create it if missing). Use a short kebab-case feature name.
5. **Stop.** Do not start implementing the PRD.

If the user answers only some questions, proceed with sensible defaults for the rest and record them under Open Questions rather than asking again.

## Clarifying question areas

- **Problem/goal:** What problem does this solve, or what outcome do we want?
- **Target user:** Who is the primary user?
- **Core functionality:** What key actions should the user be able to perform?
- **User stories:** "As a [user], I want [action] so that [benefit]."
- **Acceptance criteria:** How will we know it works? What defines success?
- **Scope/boundaries:** What should this feature explicitly *not* do?
- **Data requirements:** What data is displayed or manipulated?
- **Design/UI:** Existing mockups or guidelines? Desired look and feel?
- **Edge cases:** Error conditions or unusual situations to handle?

Example of the expected question format:

```text
1. Who is the primary user of this feature?
   A. Job seekers
   B. Employers
   C. Internal admins
   D. Other (please specify)

2. What should happen when the upload fails?
   A. Show an inline error and let the user retry
   B. Fail silently and log the error
   C. Other (please specify)
```

## PRD structure

Use these sections in this order:

1. **Introduction/Overview**: the feature, the problem it solves, and the goal.
2. **Goals**: specific, measurable objectives.
3. **User Stories**: narratives describing usage and benefit.
4. **Functional Requirements**: numbered, concise, testable statements (e.g. "1. The system must allow users to upload a profile picture.").
5. **Non-Goals (Out of Scope)**: what the feature will not include.
6. **Design Considerations** (optional): mockup links, UI/UX requirements, relevant components.
7. **Technical Considerations** (optional): known constraints, dependencies, integration points (e.g. "Should integrate with the existing Auth module").
8. **Success Metrics**: how success is measured (e.g. "Reduce support tickets about X by 20%").
9. **Open Questions**: anything unresolved, including defaults you assumed.

Omit an optional section if you have nothing real to put in it; don't pad.

## Output

- Format: Markdown
- Location: `tasks/` at the project root
- Filename: `prd-[feature-name].md`

After saving, tell the user the path and mention they can run the `generate-tasks` skill next to break the PRD into an implementation task list.
