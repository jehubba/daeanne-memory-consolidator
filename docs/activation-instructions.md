# Activation Instructions

## 1. Register the agent skill in VS Code / Copilot CLI

```powershell
Copy-Item "AGENT.md" "$env:USERPROFILE\.copilot\agents\memory-consolidator.agent.md"
```

## 2. Provision data directories

These are OS-wide shared resources this agent owns, not repo-local:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.daeanne\data\memory-consolidator\archive" | Out-Null
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.daeanne\data\memory-consolidator\dream-log" | Out-Null
# watermark.json is created by the agent itself on first run -- do not
# pre-seed it, so the agent correctly detects and logs a first run.
```

## 3. Add a Dispatcher scheduled job (daily, 3am Pacific)

This job's only purpose is to wake a fresh Daeanne process with the right
`task_type` marker at the right time. **It must not cause the scheduled
task to dispatch a second Dispatcher sub-task for the agent itself** -- the
Daeanne process that wakes for this job runs `copilot --agent
memory-consolidator` **inline**, in-process, exactly like the existing
chief-dreaming-officer and engineering-director scheduled jobs. Confirm
that pattern by inspecting one of those jobs first:

```powershell
Invoke-RestMethod "http://127.0.0.1:47777/scheduler/crons" -Headers $dh |
    Where-Object { $_.prompt -match "ChiefDreamingOfficer" -or $_.prompt -match "EngineeringDirector" }
```

Then create a matching job, converting 3am Pacific to the scheduler's
local/UTC convention used by those existing jobs:

```powershell
$job = @{
    name        = "memory-consolidator-daily"
    jobType     = "Daily"
    taskType    = "Generic"
    prompt      = "task_type: MemoryConsolidation`ntrigger: scheduled"
    runAt       = "<3am Pacific, converted to the same convention CDO/engineering-director use>"
    correlationIdTemplate = "memory-consolidator-{yyyyMMdd}"
} | ConvertTo-Json
Invoke-RestMethod "http://127.0.0.1:47777/scheduler/crons" -Method Post -Body $job `
    -ContentType "application/json" -Headers $dh
```

## 4. Update Daeanne's own instructions (`daeanne.agent.md`)

One required addition, a small clarification within the self-improvement
protocol's "proceed" threshold (not a structural rewrite):

**Add a dispatch section** (alongside the existing `## Dispatching
chief-dreaming-officer` / `## Dispatching engineering-director` sections,
using the same *inline, not Dispatcher sub-task* pattern):

```markdown
## Dispatching memory-consolidator

Invoke inline (never as an async Dispatcher sub-task with parentTaskId --
see the AgentBuilder/CDO recursion note below) daily at 3am Pacific
(scheduled job wakes this task, which then runs the CLI call itself), or
on-demand when Jeffrey says "run dream cycle", "consolidate memory", "clean
up your memory", "organize your notes/wiki", "dream time".

```powershell
copilot --agent memory-consolidator --prompt "task_type: MemoryConsolidation`ntrigger: manual"
```

If the agent's response includes a "Needs Jeffrey" section, turn it into an
outbound email via `/outbox/email` yourself -- the agent does not send
email directly. If the response has no such section, send nothing; a quiet
dream cycle is a successful one.

**Agent file:** `C:\Users\Jeffrey\.copilot\agents\memory-consolidator.agent.md`

**Recursion note:** the scheduled job for this agent creates exactly one
Dispatcher task (`taskType: Generic`, prompt `task_type:
MemoryConsolidation`). The Daeanne process that wakes for that task must
run the `copilot --agent memory-consolidator` CLI call itself, inline,
within its own process -- it must not create a second Dispatcher task for
the agent. This mirrors the existing chief-dreaming-officer and
engineering-director invocation pattern and avoids the AgentBuilder/CDO
recursion class already documented in this repo.
```

## 5. Update `~/.daeanne/wiki/architecture.md` (draft below -- Daeanne applies at activation)

Per the Agent Builder Agent's standard practice, this build authors the
patch but does not apply it -- Daeanne applies it when the agent goes live.

### Architecture Update

**Component Reference table addition:**

| Component | Description | Invocation | Data |
|---|---|---|---|
| memory-consolidator | Nightly dream-cycle memory-hygiene agent -- dedups wiki/entity facts, rolls up fully-elapsed weekly journals, archives-then-prunes stale backlog/reminders, flags contradictions. Distinct from chief-dreaming-officer (agent-roster review only). | Inline CLI call from Daeanne (`copilot --agent memory-consolidator`), scheduled daily 3am Pacific or on-demand ("run dream cycle", "consolidate memory", "dream time") | `~/.daeanne/data/memory-consolidator/{archive,dream-log,watermark.json}` |

**New communication flows introduced:**
- One new inbound trigger: the daily 3am Pacific Scheduler job
  (`taskType: Generic`, `task_type: MemoryConsolidation`).
- One new conditional outbound path: when the agent's response contains a
  "Needs Jeffrey" section (unresolved wiki/journal contradiction, or a
  backlog/reminder item that looked ambiguous enough not to auto-archive),
  Daeanne queues an email via `/outbox/email`. No new direct email-sending
  capability was added to the agent itself -- it remains
  Daeanne-mediated, same as every other agent in this OS.
- No new GitHub operations.

**Mermaid `Copilot Agents` subgraph addition** (one line):
```
memory-consolidator["memory-consolidator<br/>(nightly dream cycle, inline)"]
```

## 6. First run

Trigger a manual run to confirm end-to-end wiring, including the
empty-watermark first-run path:

```powershell
copilot --agent memory-consolidator --prompt "task_type: MemoryConsolidation`ntrigger: manual`nintent: run dream cycle"
```

Confirm afterward:
- `~/.daeanne/data/memory-consolidator/watermark.json` now exists.
- A dream log exists for today's date.
- `today` and `today - 1`'s daily journal files are byte-identical to
  before the run.
- If any week was rolled up, its daily journal files are now stub files and
  a matching archive folder exists with the full originals.
