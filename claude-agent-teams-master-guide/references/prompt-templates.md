# Spawn Prompt Templates

Adapt names, paths, and facts to the task. Teammates see none of the lead's conversation, so keep facts explicit.

## Parallel review

```text
Spawn three teammates to review PR #142:
- "security": security implications
- "perf": performance impact
- "tests": test coverage validation
Each reviews independently and reports findings with severity. Do not edit files.
Wait for all three, then synthesize.
```

## Competing hypotheses debugging

```text
<symptom description>.
Spawn 5 agent teammates, each investigating a different hypothesis. Have them
talk to each other to try to disprove each other's theories, like a scientific
debate. Update the findings doc with whatever consensus emerges.
```

## Parallel implementation (disjoint ownership)

```text
Spawn 3 teammates, Sonnet each:
- "api": owns app/Http/** and routes/**
- "ui": owns resources/js/**
- "tests": owns tests/**
Do not edit files outside your ownership. Message the owner if you need a change.
Create 5-6 tasks per teammate. Wait for teammates to finish before synthesizing.
```

## Exploration / design

```text
I'm designing <thing>. Spawn three teammates to explore from different angles:
one on UX, one on technical architecture, one playing devil's advocate.
```

## Plan-first on risky work

Switch the lead into plan mode **first**, then ask:

```text
Spawn an architect teammate to refactor the authentication module.
```

The teammate works read-only until its plan is ready. Claude Code auto-approves the plan in the lead's session without lead review; edits and commands afterwards still go through normal permission prompts.

## Using a saved role

```text
Spawn a teammate using the security-reviewer agent type to audit the auth module.
```

Requires a definition in `.claude/agents/` (or user/managed/plugin scope). See `technical-reference.md` for which fields apply to teammates.

## Nudges for the lead

- `Wait for your teammates to complete their tasks before proceeding`
- `Split the work into 5-6 tasks per teammate`
- `Ask the researcher teammate to shut down`
- `Spawn replacement teammates; the old ones can't be resumed`
