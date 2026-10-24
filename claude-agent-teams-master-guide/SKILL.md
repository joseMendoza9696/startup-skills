---
name: claude-agent-teams-master-guide
description: Playbook for deciding whether to use a Claude Code agent team, subagents, or a single session, and for designing, prompting, sizing, running, and troubleshooting agent teams (lead + teammates, shared task list, mailbox). Use this skill whenever the user mentions agent teams, teammates, a team lead, parallel Claude agents, "spawn N agents", multi-agent review or debugging with competing hypotheses, CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS, teammateMode, or TeammateIdle/TaskCreated/TaskCompleted hooks, even if they don't say "agent team" explicitly. Also use it when the user asks how to parallelize a large task across several Claude instances, or why Claude spawned subagents instead of a team.
---

# Claude Code Agent Teams: Master Guide

Agent teams are an **experimental** Claude Code feature (source: https://code.claude.com/docs/en/agent-teams, snapshot 2026-10-10). Behavior may change. If something here fails in practice, re-check the source page before improvising.

A team is several full Claude Code instances: one **lead** (the main session) plus **teammates**, coordinated through a shared task list and a mailbox. Each teammate has its own context window, so cost scales with the number of active teammates. That cost is only worth paying when parallel work benefits from teammates talking to each other.

## Workflow

Follow these steps in order when helping the user with a team.

### 1. Decide: team, subagent, or single session

| Situation | Use |
| :- | :- |
| Sequential steps, many dependencies, same-file edits | Single session |
| Focused helper whose only output is a result (lookup, verify, summarize) | Subagent |
| Parallel work where workers must share findings, challenge each other, or coordinate | **Agent team** |
| Separate sessions the user runs themselves that need to pass notes | Cross-session messaging |
| Manual parallel work without automated coordination | Git worktrees |

Default to the lighter option. Say so plainly if the task doesn't justify a team; recommending against a team is a valid and often correct answer.

Strong fits: research/review from different lenses, new modules each owned by one teammate, debugging with competing hypotheses, cross-layer changes (frontend/backend/tests). Poor fits: sequential work, same-file edits, routine or tiny tasks.

### 2. Run the pre-flight checklist

Verify each item (check the actual settings files rather than assuming):

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set in `settings.json`/`settings.local.json` `env` or the shell. Restart the session if it was just added.
- The session is **interactive**. `-p`, headless, and Agent SDK sessions never spawn teammates.
- Common operations are pre-approved in permissions. Teammate prompts bubble up to the lead and create friction otherwise.
- Work is split so **each teammate owns different files**.
- Each teammate has a clear deliverable (function, test file, review, findings doc).
- CLAUDE.md holds the project context teammates need. They load it automatically.
- Split panes need tmux, or iTerm2 with `it2`. They are unsupported in the VS Code integrated terminal, Windows Terminal, and Ghostty, where in-process mode is the answer.

### 3. Write a self-contained spawn prompt

Teammates receive only CLAUDE.md, MCP servers, skills, and the spawn prompt. The lead's conversation history does **not** carry over, so anything the teammate can't infer must be in the prompt. Include:

1. **Role / lens** (security, perf, UX, devil's advocate, hypothesis #3)
2. **Scope**: exact paths (e.g. `src/auth/`)
3. **Facts it can't infer**: stack choices, constraints (e.g. "JWT in httpOnly cookies")
4. **Deliverable and format**: e.g. "findings with severity ratings"
5. **Name**, so the user can address it later
6. **Model**, if it matters (e.g. "use Sonnet for each teammate")
7. **File ownership**, so teammates don't overwrite each other

Ready-made templates for review, debugging, parallel implementation, design exploration, and plan-first work are in `references/prompt-templates.md`. Read it when drafting the actual prompt.

Always use the words "agent team" / "teammates" explicitly. Claude sometimes launches subagents instead, and subagents appear in the same panel, so the panel doesn't prove a team formed.

### 4. Size the team and the tasks

- Start with **3-5 teammates**. Three focused teammates usually beat five scattered ones.
- Aim for **5-6 tasks per teammate**, so the lead can reassign if someone gets stuck. If the lead isn't creating enough, tell it to split the work finer.
- Task size: too small and coordination overhead dominates; too large and the teammate runs long without check-ins; right is a self-contained unit with a clear deliverable.

### 5. Run with discipline

- If the user is new to teams, start with research or review tasks (no code writing) to learn the dynamics.
- Monitor and steer. Redirect failing approaches early rather than letting a team run unattended.
- If the lead starts implementing itself or declares done early, tell it: `Wait for your teammates to complete their tasks before proceeding`, or to keep going.
- For ambiguous problems, use adversarial structure (teammates trying to disprove each other). It defeats anchoring on the first plausible theory.
- Enforce quality gates with hooks: `TeammateIdle`, `TaskCreated`, `TaskCompleted` (exit code 2 blocks the action and sends feedback). Details in `references/technical-reference.md`.
- Tasks sometimes don't get marked complete, which blocks dependents. Verify the work, then update status manually or nudge via the lead.

### 6. Avoid the common anti-patterns

| Anti-pattern | Fix |
| :- | :- |
| Spawn prompt assumes shared conversation context | Make it self-contained |
| Two teammates own the same file | Assign disjoint file sets |
| Team for a sequential or tiny task | Single session or subagent |
| Too many teammates | 3-5; scale only if the work truly parallelizes |
| Lead implements instead of waiting | Tell it to wait for teammates |
| Relying on `/resume` to restore teammates | It can't; spawn new ones |
| Hand-editing team config | It's runtime state and gets overwritten |
| Expecting per-teammate permission modes at spawn | Unsupported; change after spawn |
| Teammates spawning teammates | Unsupported; only the lead manages the team |

## Hard limits to state up front

State the relevant ones before the user commits to a design, because they can't be worked around: one team per session; no nested teams; the lead is fixed for the session; no session resumption for in-process teammates; permission modes can't be set per teammate at spawn; non-interactive sessions never spawn teammates.

## Side effect worth warning about

While the experimental flag is enabled, any subagent Claude *names* launches as a **teammate** without asking for confirmation. If the user wants plain subagents, set the variable to `"0"` (applies live on save).

## Where to look next

Read `references/technical-reference.md` when the question is about any of: enabling and precedence rules, display modes (`teammateMode`, tmux, iTerm2), UI controls and keyboard shortcuts, model selection order, plan approval, reusing `.claude/agents/` definitions as teammates, permissions, storage paths, shutdown, hooks, token/caching costs, troubleshooting symptoms, or the full limitations list.

## Answering style

Give a recommendation, not a survey. When the user describes a task, answer in this order: (1) team vs. alternative and why, (2) the concrete spawn prompt, using the templates, (3) any pre-flight gaps found. Keep the prompt ready to paste.
