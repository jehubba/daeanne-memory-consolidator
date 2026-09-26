---
applyTo: "**/*.md"
---

# Markdown Review Guidance

Apply this checklist when reviewing changes to any Markdown (`*.md`) file in this
repository, in addition to any repo-wide guidance in `.github/copilot-instructions.md`.

## Checklist

1. **Flag content reversion.** If the diff removes or reverts wording, sections, or
   fixes that were clearly added to correct a prior problem (e.g. a documented bug
   fix, a corrected instruction, a resolved TODO), call this out explicitly — even
   if the rest of the diff looks like a legitimate edit. Silent reversion of prior
   fix content is a known failure mode in this repo and must never pass review
   without comment.
2. **Flag large diffs in agent/skill spec files.** If a single Markdown file whose
   path matches an agent, skill, or spec definition (e.g. `*.agent.md`, `SKILL.md`,
   `*.instructions.md`, `*.prompt.md`, or files under a `manifests/`, `agents/`, or
   `skills/` directory) changes by more than approximately 100 lines in one diff,
   flag it as a large change requiring extra scrutiny — request a clear rationale
   if one is not already present in the PR description or commit message.
3. **Require an explicit summary of intent for skill/agent definition changes.** Any
   change to a skill or agent definition file (as described above) must be
   accompanied by an explicit, human-readable summary of *why* the change was made
   and *what* behavior it is intended to alter. If the PR description or commit
   message does not include this, request it before approving.

These checks are advisory guardrails for Copilot code review, not a replacement for
human review. When in doubt, prefer flagging over staying silent.
