# Watermark and Dream Log Schema

## `~/.daeanne/data/memory-consolidator/watermark.json`

```jsonc
{
  // ISO-8601 UTC timestamp of the last completed run. Only written after
  // every file write in a run has succeeded -- a partial run must not
  // advance this, so the next run retries the incomplete work.
  "lastRunAt": "2026-09-02T10:00:00Z",

  // "scheduled" | "manual" -- how the last run was triggered.
  "lastRunTrigger": "scheduled",

  // Every Mon-Sun ISO week (YYYY-Www) that has already been rolled up into
  // its week-*.md file and had its daily journals archived+stubbed. Used to
  // make roll-up idempotent without re-deriving it from file contents.
  "weeksRolledUpThrough": ["2026-W33", "2026-W34", "2026-W35"]
}
```

If this file is missing or fails to parse, treat the run as a first run:
proceed conservatively (favor flagging ambiguous cases over auto-resolving
them, since there's no established baseline of "what was already decided"
to compare against).

## Dream log: `~/.daeanne/data/memory-consolidator/dream-log/YYYY-MM-DD.md`

One file per run, named for the date the run executed (not the date of the
content it processed). Kept short and scannable -- this is Daeanne's own
operational history, not a user-facing report. A routine night should fit
comfortably in ~30 lines.

```markdown
# Dream log — YYYY-MM-DD

Trigger: scheduled | manual
Watermark before: <timestamp> | none (first run)

## Rolled up
- week-YYYY-WNN.md: rolled up N daily journals (YYYY-MM-DD .. YYYY-MM-DD)

## Merged
- wiki/entities/<name>.md: merged N duplicate/superseded facts (topics: ...)

## Pruned (archived, not deleted)
- notes/backlog.md: archived N resolved items -> archive/YYYY-MM-DD/backlog.md.pruned.md

## Flagged for Jeffrey (omit section entirely if empty -- most nights it's empty)
- Contradiction: wiki/entities/foo.md says X (as of DATE); journal DATE says Y. No clear resolution.
- Possibly-still-active backlog item not archived: "..." (notes/backlog.md, last touched DATE)

## Watermark after
lastRunAt: <timestamp>
weeksRolledUpThrough: <list>
```

Every claim in the log ("merged N facts", "rolled up week X") must be
independently checkable against the actual archive/live-file diffs written
during that same run -- the log is a record of what happened, not a
separate narrative.

## Archive layout

```
~/.daeanne/data/memory-consolidator/archive/YYYY-MM-DD/
  journal-YYYY-MM-DD.md          # full raw daily journal, pre-rollup
  <wiki-filename>.pre-merge.md   # full pre-merge section, for wiki/entity merges
  <notes-filename>.pruned.md     # appended per-run; full text of pruned items
```

`YYYY-MM-DD` here is the date the *archiving action* happened (the run
date), not the date of the archived content -- so everything from one run
lands in one dated folder regardless of which historical week or day it
originally belonged to.
