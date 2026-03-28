---
name: gsd:director
description: Architect/QA role — design requirements, review plans, checkpoint execution, run retrospectives
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Agent
  - WebSearch
  - WebFetch
---
<objective>
Architect/QA role that operates alongside GSD. Runs in a persistent session that holds
the system mental model. Communicates with executor sessions ONLY through .planning/
artifacts and canonical specs.

Modes:
- /gsd:director design     — Open-ended requirements conversation -> canonical specs
- /gsd:director review     — QA a GSD phase plan against specs + runtime path
- /gsd:director checkpoint — Verify execution output, catch drift
- /gsd:director retro      — Post-milestone retrospective that improves the director
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/director.md
</execution_context>

<context>
Optional argument: mode (design, review, checkpoint, retro).
If no mode specified, asks the user which mode.

For review mode, optional phase number argument.
For checkpoint mode, optional phase number argument.

Requires: .planning/ directory for review/checkpoint/retro modes.
Design mode can run without .planning/ (creates it).

Loads RETROSPECTIVE.md if it exists — accumulated learnings from prior milestones.
</context>

<process>
Execute the director workflow from @~/.claude/get-shit-done/workflows/director.md.
Detect mode from arguments, load required context, and enter the appropriate mode.
</process>
