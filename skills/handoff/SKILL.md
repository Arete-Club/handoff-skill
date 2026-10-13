---
name: handoff
description: >
  Preserve task continuity across AI coding agent sessions by writing
  or reading a concise HANDOFF.md. Export: use when the user wants to
  end a session, transfer current work to a new session, or create a
  handoff document ("handoff", "prepare handoff", "end this session",
  "continue in a new session", "세션 인계", "다음 세션으로 넘겨줘").
  Import: use when the user wants to resume previous work, provides a
  handoff document, or asks to continue from HANDOFF.md. Captures
  actionable task state, not conversation history. Not for full
  conversation summaries, cross-agent orchestration, long-term memory,
  automatic context sync, or replacing project documentation.
---

# Handoff

Transfer the minimum sufficient context for a new AI coding agent
session to continue an existing task in the same project accurately.

**Core principle: preserve task state, not conversation history.**

A handoff is measured not by how much it preserves, but by how
accurately the next session can resume with minimal unnecessary
context. Transfer what the next session needs, not everything the
previous session knows.

Out of scope: full conversation summarization, cross-agent
orchestration, long-term memory management, automatic context
synchronization, and replacing project documentation.

## Modes

- **Export**: the user is ending the session, moving work to a new
  session, or asks for a handoff document.
- **Import**: the user is resuming from a handoff, provides a handoff
  document, or asks to continue using `HANDOFF.md`.

## Rules for Both Modes

- Never fabricate test runs, verification results, decisions, or
  implementation results.
- Never delete project files as part of a handoff.
- Never include secrets, credentials, tokens, or sensitive
  environment variable values.
- Do not commit or push the handoff automatically.
- Respect project-specific instructions.

## Export

### Steps

1. Identify the current task and its objective.
2. Determine the latest known implementation state.
3. Extract decisions and their rationale.
4. Identify completed and remaining work.
5. Record relevant files and verification results.
6. Identify blockers, risks, and unresolved questions.
7. Specify the next concrete action.
8. Write `HANDOFF.md` to the project root using the format below.
   Ask before overwriting an existing handoff whose contents may
   still be needed. Without filesystem access, output the content
   for the user to save.

Do not modify application code during export.

### What to Capture

**Decisions.** Record decisions that materially affect future work:
what was decided, why, and rejected alternatives when relevant. Skip
discussions that did not influence the final decision.

**Task state.** Sort work into Completed, In Progress, Remaining, and
Blocked. Generated code is not Completed on that basis alone; keep
implementation status separate from verification status.

**Verification.** Label each verification item:

| Label | Meaning |
|-------|---------|
| VERIFIED | Confirmed through available evidence |
| UNVERIFIED | Implemented or claimed but not checked |
| FAILED | Verification was attempted and failed |
| NOT RUN | Verification has not been attempted |

Include test commands and results when available.

**Files.** List only the files needed to understand or resume the
work, preferably as workspace-relative paths (e.g.
`src/core/runner.py`). Do not copy entire source files into the
handoff.

**What to leave out.** Prefer decisions over discussion, current state
over chronological logs, file references over copied code, concrete
next actions over broad plans, and verified facts over speculation.
Exclude casual conversation, repeated explanations, superseded plans,
unrelated project information, raw tool output, and unnecessary
implementation details. Never drop information required to resume
safely.

### HANDOFF.md Format

Omit empty or irrelevant sections when possible.

```markdown
# Session Handoff

## 1. Objective

The current task and its expected outcome.

## 2. Current State

The implementation state in no more than five concise sentences.

## 3. Decisions & Rationale

| Decision | Rationale |
|----------|-----------|
| ... | ... |

## 4. Progress

### Completed
- ...

### In Progress
- ...

### Remaining
- ...

### Blocked
- ...

## 5. Verification

| Item | Status | Evidence |
|------|--------|----------|
| ... | VERIFIED / UNVERIFIED / FAILED / NOT RUN | ... |

## 6. Relevant Files

| Path | Purpose |
|------|---------|
| ... | ... |

## 7. Known Issues & Risks

- ...

## 8. Next Action

The first concrete action the next session should perform, with its
expected result or completion condition when useful.

## 9. Constraints

Requirements, restrictions, and explicit user preferences that
remain applicable.
```

### Done When

- The current objective is clear.
- Important decisions and their rationale are preserved.
- Completed and remaining work are distinguishable.
- Verification status is explicit.
- Relevant files are identified.
- The next action is executable without reconstructing the entire
  previous conversation.

## Import

### Steps

1. Read `HANDOFF.md`.
2. Identify the task objective and current state.
3. Verify the handoff against the current workspace (see below).
4. Reconstruct the immediate next action.
5. Briefly communicate the understood state.
6. Continue the task when requested.

### Verify Against the Workspace

A handoff is a snapshot of a previous session; the workspace may have
changed since. Before relying on its claims:

- Inspect referenced files when available.
- Check relevant repository state when necessary.
- Identify stale, missing, or conflicting information.
- Treat missing evidence as unverified.

When the handoff contradicts the current workspace, do not treat the
handoff as authoritative. Do not assume earlier verification remains
valid after relevant code changes.

### Preserve Decisions

Keep established decisions unless new evidence invalidates them,
requirements have changed, or the user explicitly asks to reconsider.
Do not reopen settled decisions without a reason.

### Resume Efficiently

Do not repeat completed planning, rewrite existing documentation
unnecessarily, reimplement verified functionality, or ask for
information already in the handoff. Prefer the smallest next
executable step.

### Handle Missing Context

If essential information is missing, search the available project
files, infer only what the evidence supports, and clearly state any
unresolved uncertainty. Ask the user only when the task cannot
proceed safely.

### Done When

- The task objective is understood.
- Current workspace state has been checked as needed.
- Existing decisions are preserved.
- The next action is identified.
- Work can resume without unnecessary replanning.
