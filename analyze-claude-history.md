# Analyze My Claude Code Usage Patterns

You are an expert in prompt engineering, context management, and agentic workflows.

## Step 1: Extract (run this first, before analysis)

The raw history file is massive and full of noise (tool results, file contents, system prompts). Extract only the signal before analyzing:

```bash
cat ~/.claude/history.jsonl | jq -r '
  # Keep only user messages and assistant text (no tool calls/results)
  select(.role == "user" or (.role == "assistant" and .type != "tool_result")) |
  {role, text: (.content | if type == "array" then map(select(.type == "text") | .text) | join(" ") else . end)}
' > /tmp/claude-history-lean.jsonl
```

If that doesn't match the schema, inspect the first 3 lines of the file to understand the structure, then adapt the extraction to produce: **only my (user) messages and a one-line summary of each assistant action** (tool name + first 80 chars of text). Strip all tool results, file contents, and system messages.

Target: reduce to <20% of original size before analysis.

## Step 2: Sample

Don't analyze everything. From the extracted data:

- Count total sessions
- Pick ~30 sessions evenly distributed across the full date range (early, middle, recent)
- Favor diversity: include short sessions, long sessions, debugging sessions, feature work, config changes

Note which sessions you sampled.

## Step 3: Analyze

From the sampled sessions, identify:

### Top 10 Habits & Patterns

For each:

1. **Name** — short label
2. **What I do** — the behavior
3. **Why it's effective** — the principle behind it
4. **Evidence** — quote or paraphrase 2-3 real examples

### Top 10 Mistakes & Weaknesses

For each:

1. **Name** — short label
2. **What I do** — the behavior
3. **Why it hurts** — cost in tokens, time, or output quality
4. **Evidence** — 2-3 real examples
5. **Fix** — one concrete recommendation

## Guidelines

- Be brutally honest. Flattery is useless — I want signal.
- Prioritize by impact, not frequency.
- If a pattern is both strength and weakness depending on context, put it where it dominates and note the nuance.
- Compare against best practices for agentic coding assistants (tool delegation, context management, task decomposition, verification loops).
