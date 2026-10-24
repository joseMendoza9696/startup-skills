# Agent Teams: Technical Reference

Source: https://code.claude.com/docs/en/agent-teams (fetched 2026-10-10). Experimental; re-check the source if something stops working.

## Contents
- Enabling
- Components
- Storage locations
- Display modes
- Controlling teammates (UI)
- Model selection
- Plan approval
- Tasks
- Subagent definitions as teammates
- Permissions
- Context and communication
- Shutdown
- Hooks
- Token usage and caching
- Troubleshooting
- Limitations
- Related docs

## Enabling

In `settings.json` (user, project, or local):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Or export it in the shell. Without it: no team is set up, no team dirs are written, and Claude does not spawn or propose teammates.

**Side effect:** while enabled, any subagent Claude *names* launches as a teammate (Claude names subagents on its own so it can message them later), with no confirmation. To revert, set the variable to `"0"` (applies live on save; the next named subagent is a plain subagent). Precedence: user settings `0` overrides a shell export, but project/local/`--settings`/managed settings that set `1` win.

## Components

| Component | Role |
| :- | :- |
| Team lead | Main session; spawns teammates and coordinates. Fixed for the session |
| Teammates | Separate Claude Code instances, each with its own context window |
| Task list | Shared work items; states pending, in progress, completed; supports dependencies |
| Mailbox | Messaging between agents |

## Storage locations

- Team config: `~/.claude/teams/{team-name}/config.json` (removed at session end)
- Task list: `~/.claude/tasks/{team-name}/` (persists locally, never uploaded; retention follows `cleanupPeriodDays`)
- Mailboxes: `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`
- `{team-name}` = `session-` + first 8 chars of the session ID
- Config holds runtime state (session IDs, tmux pane IDs). **Don't edit or pre-author it.** A project-level `.claude/teams/teams.json` is NOT recognized.
- The `members` array lists names and agent IDs; the lead has agent type `team-lead`. Teammates can read it to discover each other.
- Mailbox entries are validated on read: malformed ones are reported and removed, valid ones still delivered. A message counts as sent only if the mailbox write succeeds; otherwise the sender gets an error.

## Display modes

| Mode | Behavior |
| :- | :- |
| `"in-process"` (default) | All teammates inside the main terminal; works anywhere |
| `"auto"` | Split panes if already in tmux or iTerm2 with `it2`; else in-process |
| `"tmux"` | Split panes; auto-detects tmux vs iTerm2 |
| `"iterm2"` | iTerm2 native panes; requires the `it2` CLI (errors with an install command if missing) |

Global setting in `~/.claude/settings.json`:

```json
{ "teammateMode": "auto" }
```

Per session: `claude --teammate-mode auto` (experimental; not in `--help`).

Setup: install tmux via package manager (`tmux -CC` in iTerm2 is recommended; tmux works best on macOS). For iTerm2, install the `it2` CLI and enable **iTerm2 → Settings → General → Magic → Enable Python API**.

Split panes are unsupported in the VS Code integrated terminal, Windows Terminal, and Ghostty. Inside the VS Code extension, use in-process mode.

## Controlling teammates (UI)

The agent panel sits below the prompt input.

- **Up/Down**: select a teammate
- **Enter**: open its transcript and message it directly
- **Escape**: clear selection; while viewing a teammate, interrupts its current turn
- **x** on a selected teammate: stop it
- **Ctrl+T**: toggle the task list
- Idle rows hide 30s after the whole panel goes idle and return on the teammate's next turn (still running and addressable). More than 3 idle collapse into `N idle agents`; Enter expands, Esc collapses.
- Split-pane: click a pane to interact.

While viewing an in-process teammate, plain text and skills go to the teammate; built-in commands go to the lead. `/compact`, `/clear`, `/rewind` act on the lead (confirmation required). `/model` and `/fast` don't run from this view. A teammate's model and fast mode are fixed at spawn.

## Model selection (first match wins)

1. Model named in the spawn prompt for that teammate
2. The subagent definition's `model` (`inherit` = lead's model)
3. `CLAUDE_CODE_SUBAGENT_MODEL` (if not `inherit`)
4. The lead's current model

- `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (v2.1.257+): skips 1 and 2; uses `CLAUDE_CODE_SUBAGENT_MODEL` if set and not `inherit`, else the lead's model.
- Before v2.1.251, `CLAUDE_CODE_SUBAGENT_MODEL` came first.
- `teammateDefaultModel` was removed in v2.1.234; name the model in the prompt instead.
- Models are checked against the org `availableModels` allowlist. Blocked family aliases resolve to the newest permitted version (Anthropic API / Claude Platform on AWS); other blocked values fall back to the lead's model.
- Teammates inherit the lead's effort level by default (split-pane since v2.1.186).

## Plan approval

A teammate spawned while the **lead is in plan mode** works read-only until its plan is ready, then sends a plan approval request. Claude Code **auto-approves** it in the lead's session without lead review; the teammate then exits plan mode and implements. Its edits and commands still go through normal permission prompts. Switch the lead to plan mode *before* asking for the teammate.

## Tasks

- The lead creates tasks; teammates work them. Pending tasks with unresolved dependencies can't be claimed. Dependencies auto-unblock when a prerequisite completes.
- The **lead assigns** explicitly, or teammates **self-claim**: a finished teammate picks the next unassigned, unblocked task. Claiming uses file locking (no races).
- Only agents with the Task tools (TaskCreate/TaskGet/TaskList/TaskUpdate) use the shared list; others coordinate by messages.
- Known issue: teammates sometimes fail to mark tasks complete, blocking dependents. Verify the work, update status manually, or have the lead nudge.

## Subagent definitions as teammates

Define a role once in `.claude/agents/` (project), user, managed, or plugin scope, then ask for a teammate of that agent type.

| Field | Effect on teammate |
| :- | :- |
| `tools` | Limits tools. In-process: `SendMessage` added (plus Task tools if the session has them) |
| `model` | Used if the spawn prompt names none (both modes) |
| `disallowedTools` | In-process: removed (SendMessage and Task tools stay anyway) |
| `effort` | In-process only |
| Body | In-process: **appended** to the default system prompt. Split-pane: **replaces** it |
| `skills` | **Not applied** (teammate loads skills from project/user settings) |
| `mcpServers` | Split-pane: applied. In-process: ignored (loads project/user MCP servers) |

A stopped in-process teammate is revived in-session when messaged (conversation restored). Project `.claude/agents/` definitions are re-applied only if the folder itself is trusted (trusting a parent doesn't count); otherwise it returns with only baseline tools.

## Permissions

- Teammates start with the lead's permission mode, **except** `dontAsk` (not inherited). `--dangerously-skip-permissions` on the lead applies to all teammates.
- A teammate's mode can be changed after spawn, but not set per teammate at spawn.
- Teammate permission prompts appear in the **lead's** session; approve there.
- Messages between agents are labeled as coming from another Claude session, not the user. A teammate can't approve prompts or give consent on the user's behalf, and can't relay a denied action to another teammate to bypass checks.
- In auto mode, the classifier treats relayed approval claims as untrusted and reviews every inter-agent message (including shutdown and plan approval) before delivery; blocked messages never arrive.

## Context and communication

- A teammate loads CLAUDE.md, MCP servers, and skills (respecting `--setting-sources` if the lead used it; before v2.1.281 split-pane teammates loaded all sources), plus the spawn prompt. **No lead conversation history.**
- Messages are delivered automatically (no polling).
- When a teammate stops it notifies the lead with its final answer; API-error endings notify with the error text.
- Messaging is one recipient per message; to reach everyone, send one message per recipient.
- The lead names teammates at spawn; **specify names in the spawn instruction** for predictable references.

## Shutdown

Ask the lead, e.g. `Ask the researcher teammate to shut down`. The lead sends a shutdown request; the teammate may approve or reject with an explanation. It finishes its current request or tool call first (can be slow). Team dirs are cleaned up at session end (the task list dir persists).

## Hooks (quality gates)

| Hook | Fires | Exit code 2 does |
| :- | :- | :- |
| `TeammateIdle` | Teammate about to go idle | Sends feedback, keeps the teammate working |
| `TaskCreated` | Task being created | Blocks creation, sends feedback |
| `TaskCompleted` | Task being marked complete | Blocks completion, sends feedback |

Use these to require passing tests, clean lint, or a deliverable file before a task can close.

## Token usage and caching

- Cost scales with the number of active teammates.
- In-process teammate requests use a 5-minute cache TTL by default (even on subscription). Set `subagentPromptCacheTtl` to `1h` to extend it (1h cache writes cost more).
- See https://code.claude.com/docs/en/costs#agent-team-token-costs

## Troubleshooting

| Symptom | Cause / fix |
| :- | :- |
| Teammates not appearing | In-process: check the agent panel (arrows + Enter); the row may be hidden-idle, not stopped. The task may be too simple for Claude to spawn a team. Split panes: `which tmux`; for iTerm2 verify `it2` and the Python API |
| Claude used subagents, not a team | Re-ask explicitly for "an agent team" |
| Teammates spawn when you wanted subagents | Set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` to `"0"` |
| Too many permission prompts | Pre-approve common ops in permission settings before spawning |
| Teammate stopped after an error | Open its transcript; give instructions or spawn a replacement. A message wakes a teammate waiting to retry an API request |
| Lead declared done too early | Tell it to keep going |
| Lead implementing instead of delegating | "Wait for your teammates to complete their tasks before proceeding" |
| Task stuck | Check whether the work is actually done; update status manually or nudge via the lead |
| Orphaned tmux session | `tmux ls`, then `tmux kill-session -t <session-name>` |
| Mailbox write failure | https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox |
| Teammate agent definition not restored | Trust the exact folder holding the agent file |

## Limitations

- **No session resumption with in-process teammates**: `/resume` and `/rewind` don't restore them; the lead may message nonexistent teammates. Tell it to spawn new ones.
- Task status can lag; shutdown can be slow.
- **One team per session**; no additional named teams; can't share across sessions.
- **No nested teams**: teammates can't spawn teammates.
- In-process teammates can't run background subagents (`background: true` errors; `run_in_background: true` errors or runs in foreground).
- The lead is fixed; no promotion or transfer.
- Permission modes can't be set per teammate at spawn.
- Split panes unsupported in the VS Code integrated terminal, Windows Terminal, Ghostty.
- Non-interactive (`-p`, Agent SDK) sessions never spawn teammates.

## Related docs

- Subagents: https://code.claude.com/docs/en/sub-agents
- Cross-session messaging: https://code.claude.com/docs/en/cross-session-messaging
- Hooks: https://code.claude.com/docs/en/hooks
- Permissions: https://code.claude.com/docs/en/permissions
- Costs: https://code.claude.com/docs/en/costs
- Git worktrees: https://code.claude.com/docs/en/worktrees
