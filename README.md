# Handoff

```text
 _                           _            __    __
| |__     __ _   _ __     __| |   ___    / _|  / _|
| '_ \   / _` | | '_ \   / _` |  / _ \  | |_  | |_
| | | | | (_| | | | | | | (_| | | (_) | |  _| |  _|
|_| |_|  \__,_| |_| |_|  \__,_|  \___/  |_|   |_|

```

A skill that lets AI coding agents hand off work between sessions
through a short, verifiable `HANDOFF.md`, so you can keep sessions
short without losing the thread.

## Overview

Handoff has two modes:

- **Export**: at the end of a session, the agent writes `HANDOFF.md`
  to the project root. It records the objective, current state,
  decisions and their rationale, progress, verification status,
  relevant files, risks, the next concrete action, and constraints
  that still apply.
- **Import**: at the start of a new session, the agent reads
  `HANDOFF.md`, checks its claims against the current workspace, and
  picks up from the next action.

It transfers task state, not conversation history.

Out of scope: full conversation summaries, long-term memory,
cross-agent orchestration, automatic context sync, and replacing
project documentation.

## Why Handoff?

### The problem: long contexts drop things

While working with AI coding agents, I kept running into the same
failure: as the context window grew, the model started losing track
of earlier context, such as decisions made an hour ago, constraints
stated at the start, or which tests had actually run. The longer the
session, the more confidently it built on things it had partly
forgotten.

My fix was to keep sessions short. But short sessions move the risk
to the boundary between them. If the handoff is lossy, every new
session starts with a gap, or with a summary it trusts more than it
should.

Handoff is built for that boundary.

### What it does differently

- **Decisions keep their reasons.** Each decision is recorded with
  its rationale and, when relevant, the alternatives that were
  rejected, so the next session doesn't reopen it.
- **"Written" is not "done".** Implementation status and verification
  status are tracked separately. Every check is labeled `VERIFIED`,
  `UNVERIFIED`, `FAILED`, or `NOT RUN`, and results are never
  invented.
- **The handoff is not trusted blindly.** On import, the agent checks
  the handoff against the actual workspace. When they disagree, the
  workspace wins.
- **Less is the goal.** Current state over chronology, file paths
  over copied code, one concrete next action over a broad plan.

### Compared to built-in options in Claude Code

| | What carries over | Trade-off |
|---|---|---|
| `/compact` | A summary of the conversation, inside the same session | The summary isn't a file you can review or edit |
| `--resume` / `--continue` | The full previous conversation | The long context comes back with it |
| **Handoff** | A short `HANDOFF.md` in your project | A fresh session starts from a state you can read and correct |

## Installation

The installable skill is the `skills/handoff/` folder.

### Claude Code

For all your projects:

```bash
git clone https://github.com/Arete-Club/handoff-skill.git
mkdir -p ~/.claude/skills
cp -r handoff-skill/skills/handoff ~/.claude/skills/
```

For a single project, copy it into that project's `.claude/skills/`
instead:

```bash
mkdir -p .claude/skills
cp -r /path/to/handoff-skill/skills/handoff .claude/skills/
```

Claude Code picks up changes in these folders without a restart. If
the `skills` folder did not exist when the session started, run
`/reload-skills`.

### Other agents

`SKILL.md` uses only the `name` and `description` frontmatter fields
and does not depend on agent-specific tools, so it should work with
other agents that load `SKILL.md` skills. Copy `skills/handoff/` to
wherever your agent loads skills from.

## Usage

### Recommended workflow

1. Work in short, focused sessions.
2. Before the context gets long, export a handoff.
3. Read `HANDOFF.md` and fix anything that's wrong. It's plain
   Markdown.
4. Start a fresh session (`/clear` or a new terminal).
5. Import the handoff and continue.

### Export

Ask in plain language, for example:

- "handoff" / "prepare handoff"
- "end this session" / "continue in a new session"
- "세션 인계" / "다음 세션으로 넘겨줘"

In Claude Code you can also run `/handoff`.

The agent writes `HANDOFF.md` to the project root. It does not modify
application code, include secrets, or commit the file. If a
`HANDOFF.md` already exists and may still be needed, it asks before
overwriting. Whether to commit `HANDOFF.md` or add it to `.gitignore`
is up to you.

### Import

In a new session, for example:

- "Resume from HANDOFF.md"
- "Continue the previous work"

The agent reads the handoff, checks the referenced files and
repository state, tells you briefly what it understood, and continues
when you ask.

## Example

A hand-written `HANDOFF.md` for a hypothetical task, to show the
format. Empty sections such as Blocked are omitted.

```markdown
# Session Handoff

## 1. Objective

Add per-user rate limiting to the public API. Requests over the limit
return 429 with a `Retry-After` header.

## 2. Current State

The limiter middleware is implemented and wired into the router. Unit
tests pass. The integration test for the 429 response is not written
yet.

## 3. Decisions & Rationale

| Decision | Rationale |
|----------|-----------|
| Token bucket, not fixed window | Fixed window was rejected because it allows bursts at window boundaries. |
| Counters in Redis | The API runs on multiple instances, so in-memory counters would not be shared. |

## 4. Progress

### Completed
- Limiter middleware

### In Progress
- Integration test for the 429 response

### Remaining
- Per-plan limits from config

## 5. Verification

| Item | Status | Evidence |
|------|--------|----------|
| Middleware unit tests | VERIFIED | `pytest tests/test_rate_limit.py`: 12 passed |
| 429 response format | UNVERIFIED | Implemented, no test yet |
| Behavior when Redis is down | NOT RUN | - |

## 6. Relevant Files

| Path | Purpose |
|------|---------|
| `src/middleware/rate_limit.py` | Limiter implementation |
| `tests/test_rate_limit.py` | Unit tests |
| `config/limits.yaml` | Limit values; per-plan section not added yet |

## 7. Known Issues & Risks

- Fail-open vs. fail-closed when Redis is unreachable is not decided.

## 8. Next Action

Write `tests/integration/test_rate_limit_api.py` asserting a 429 with
`Retry-After` once the limit is exceeded. Done when it passes locally.

## 9. Constraints

- No new runtime dependencies beyond the existing Redis client.
```
