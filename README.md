# daeanne-memory-consolidator

Nightly "dream cycle" memory-hygiene agent for the Daeanne OS. Consolidates
and organizes Daeanne's own long-term memory store (daily journals, weekly
notes, personal wiki, entity files, backlog/ideas/reminders notes) based on
what has accumulated since the last cycle.

Inspired by biological sleep-based memory consolidation (Jeffrey's framing,
2026-09-02 email "Dreaming"): new/recent memories get reviewed, reinforced
if still relevant, and reorganized into stable long-term structure. Nothing
is ever silently deleted — everything pruned or superseded is archived
first.

**This is not chief-dreaming-officer.** CDO is a creative/skeptical review
of the *agent roster* (should agents be retired/added/10x'd). This agent
does not evaluate agents at all — its sole job is memory hygiene on
Daeanne's own notes/wiki/journal files.

## What's in this repo

| Path | Purpose |
|---|---|
| `AGENT.md` | Full agent definition — identity, pipeline, self-eval criteria |
| `docs/watermark-and-dream-log-schema.md` | File formats for `watermark.json` and dream log entries |
| `docs/activation-instructions.md` | How to register this agent, provision its data directories, and schedule it |
| `docs/build-review.md` | Self-evaluation record for this build |

## Invocation pattern

This agent is invoked **inline**, synchronously, directly by Daeanne — the
same invocation class as `chief-dreaming-officer` and
`engineering-director`. It is **never** dispatched as an async Dispatcher
sub-task with `parentTaskId`, and the scheduled 3am job must never cause a
second Dispatcher hop back to this agent (see AGENT.md → Integration Points
for the full recursion-avoidance rationale).

**Scheduled** (Scheduler API creates a Daily job; on wake, Daeanne runs this
inline within her own task process):
```powershell
copilot --agent memory-consolidator --prompt "task_type: MemoryConsolidation`ntrigger: scheduled"
```

**On-demand** (Jeffrey says "run dream cycle", "consolidate memory", "clean
up your memory", "organize your notes/wiki", "dream time"):
```powershell
copilot --agent memory-consolidator --prompt "task_type: MemoryConsolidation`ntrigger: manual`nintent: run dream cycle"
```

## Data it owns

- `~/.daeanne/data/memory-consolidator/archive/YYYY-MM-DD/` — permanent
  archive of everything pruned, rolled up, or superseded, written before
  any corresponding live-file edit.
- `~/.daeanne/data/memory-consolidator/dream-log/YYYY-MM-DD.md` — one
  short, scannable log per run.
- `~/.daeanne/data/memory-consolidator/watermark.json` — last-processed
  timestamp and rolled-up-weeks list, for idempotency across runs.

## Scope note

This build covers the agent definition and its data-directory contract. It
does not write the daily/weekly Scheduler job itself, the
`daeanne.agent.md` dispatch-section addition, or the `architecture.md`
wiki update — those are activation-time changes applied by Daeanne per
`docs/activation-instructions.md`, following the same pattern used for
every other Daeanne-OS agent build.
