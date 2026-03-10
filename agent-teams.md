# Agent Teams — Quick Reference & Prompt Templates

> Experimental feature. Enable first:
> ```json
> // settings.json
> { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
> ```

## When to Use Agent Teams vs Subagents

| Use agent teams when...                          | Use subagents when...                    |
|:-------------------------------------------------|:-----------------------------------------|
| Teammates need to talk to each other             | Only the result matters                  |
| Work benefits from debate or competing views     | Tasks are focused and independent        |
| Complex coordination across layers/domains       | You want lower token cost                |
| You want to interact with individual workers     | Sequential hand-off is fine              |

**Rule of thumb:** 3-5 teammates, 5-6 tasks per teammate. Start with research/review before attempting parallel implementation.

---

## Setup

### Display modes

| Mode           | Setting                        | How it works                                           |
|:---------------|:-------------------------------|:-------------------------------------------------------|
| **In-process** | `"teammateMode": "in-process"` | All in one terminal. `Shift+Down` to cycle teammates.  |
| **Split panes**| `"teammateMode": "tmux"`       | Each teammate in its own pane. Requires tmux or iTerm2.|
| **Auto**       | `"teammateMode": "auto"`       | Split if inside tmux, in-process otherwise. (Default)  |

Override per session: `claude --teammate-mode in-process`

---

## Navigation & Interaction (In-Process Mode)

| Key             | Action                                     |
|:----------------|:-------------------------------------------|
| `Shift+Down`    | Cycle to next teammate (wraps to lead)     |
| `Enter`         | View a teammate's session                  |
| `Escape`        | Interrupt teammate's current turn          |
| `Ctrl+T`        | Toggle the shared task list                |
| Type + `Enter`  | Send message directly to current teammate  |

In **split-pane mode**, click into any pane to interact directly.

---

## Prompt Templates

### Research & Exploration

```
I'm exploring [TOPIC/PROBLEM]. Create an agent team:
- Teammate 1: [ANGLE_1] (e.g., UX research)
- Teammate 2: [ANGLE_2] (e.g., technical architecture)
- Teammate 3: Devil's advocate — challenge the other teammates' findings

Have them discuss and synthesize a recommendation.
```

**Example:**
```
I'm designing a CLI tool that helps developers track TODO comments across
their codebase. Create an agent team to explore this from different angles: one
teammate on UX, one on technical architecture, one playing devil's advocate.
```

### Parallel Code Review

```
Create an agent team to review PR #[NUMBER]. Spawn three reviewers:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

### Competing Hypothesis Debugging

```
[DESCRIBE THE BUG/SYMPTOM].
Spawn [N] agent teammates to investigate different hypotheses.
Have them talk to each other to try to disprove each other's theories,
like a scientific debate. Update findings when consensus emerges.
```

**Example:**
```
Users report the app exits after one message instead of staying connected.
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to
each other to try to disprove each other's theories, like a scientific
debate. Update the findings doc with whatever consensus emerges.
```

### Parallel Implementation

```
Create a team with [N] teammates to [TASK] in parallel.
Use [MODEL] for each teammate.

Teammate assignments:
- [NAME_1]: [SCOPE — specific files/modules]
- [NAME_2]: [SCOPE]
- [NAME_3]: [SCOPE]

No two teammates should edit the same file.
```

**Example:**
```
Create a team with 4 teammates to refactor these modules in parallel.
Use Sonnet for each teammate.
```

### Guarded Implementation (Plan Approval)

```
Spawn a [ROLE] teammate to [TASK].
Require plan approval before they make any changes.
Only approve plans that [CRITERIA].
```

**Example:**
```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
Only approve plans that include test coverage and don't modify the database schema.
```

### Cross-Layer Coordination

```
Create an agent team for [FEATURE]:
- Frontend teammate: owns src/components/ and src/pages/
- Backend teammate: owns src/api/ and src/services/
- Test teammate: owns test/ — writes integration tests as the others finish

Frontend and backend work in parallel. Test teammate depends on both.
```

---

## Steering the Team Mid-Flight

| Situation                         | What to say to the lead                                              |
|:----------------------------------|:---------------------------------------------------------------------|
| Lead starts doing work itself     | "Wait for your teammates to complete their tasks before proceeding"  |
| Need more granular tasks          | "Split the remaining work into smaller pieces, 5-6 tasks per mate"  |
| Teammate going down a wrong path  | Message them directly via `Shift+Down`: "Stop — try [ALTERNATIVE]"  |
| Task appears stuck                | "Check if [TASK] is actually done and update its status"             |
| Done with a teammate              | "Ask the [ROLE] teammate to shut down"                               |
| Done with everything              | "Clean up the team" (shut down all teammates first)                  |

---

## Quality Gates with Hooks

Use hooks to enforce rules automatically:

- **`TeammateIdle`** — runs when a teammate is about to go idle. Exit code 2 sends feedback and keeps them working.
- **`TaskCompleted`** — runs when a task is marked complete. Exit code 2 blocks completion with feedback.

---

## Gotchas

- **No shared conversation history.** Teammates get project context (CLAUDE.md, MCP, skills) but NOT the lead's chat history. Put key details in the spawn prompt.
- **File conflicts.** Two teammates editing the same file = overwrites. Partition file ownership.
- **No session resumption.** `/resume` won't restore in-process teammates. Spawn new ones after resume.
- **One team per session.** Clean up before starting a new team.
- **No nested teams.** Teammates can't spawn their own teams.
- **Permissions inherited at spawn.** All teammates get the lead's permission mode. Change individually after spawn if needed.
- **Token cost scales linearly** with teammates. Don't create a team for routine tasks.
