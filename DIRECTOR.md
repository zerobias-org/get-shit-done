# GSD Director

A persistent architect/QA role that operates alongside GSD. The director runs in its
own terminal session, holds the system mental model for an entire milestone, and
communicates with GSD executor sessions only through `.planning/` artifacts.

## Why

GSD is good at breaking decided work into tasks and executing them. It is bad at:

- **Requirements discovery** — GSD's discuss-phase is a menu, not a conversation.
  It creates tunnel vision without coherence. Complex requirements emerge through
  back-and-forth discussion, not multiple-choice forms.

- **Systemic thinking** — GSD plans artifacts (files to create), not behaviors
  (system states to achieve). It doesn't trace runtime paths ("when this container
  boots, what's the first thing that happens?"). This produces plans that compile
  but don't run.

- **Execution discipline** — GSD executors don't reliably read CLAUDE.md, check
  branches, run type checkers, or respect project invariants. They fill gaps with
  guesses instead of reading existing code.

- **Cross-session memory** — GSD agents lose context between sessions. Decisions
  made in one session get violated in the next because the reasoning behind them
  isn't preserved.

The director fills these gaps. Design requirements through real conversation. Review
plans before execution. Catch drift during execution. Accumulate learnings across
milestones.

## How It Works

```
Terminal 1: /gsd:director          (persistent — survives the milestone)
Terminal 2: /gsd:plan-phase 6      (GSD work — comes and goes)
Terminal 3: /gsd:execute-phase 6   (GSD work — comes and goes)
```

The director observes GSD's `.planning/` artifacts (read-only) and maintains its own
workspace at `.planning/director/` (write-only). It never modifies GSD artifacts.
GSD never touches the director's workspace.

When the director session ends (context limit, overnight, etc.), it passivates its
understanding to `.planning/director/SESSION-STATE.md`. On restart, it resumes by
loading that state.

## Modes

### `/gsd:director design`

Open-ended conversation to establish requirements, architecture, and constraints.
Produces canonical specs that downstream GSD agents must conform to.

This is a **conversation**, not a form:
- Follow threads deep — one topic at a time until resolved
- Push back when something seems wrong
- Surface implied requirements the user hasn't stated
- Trace runtime paths and failure modes
- Run research spikes for unknowns

Outputs: Canonical specs (ARCHITECTURE.md, CONTRACTS.md, INVARIANTS.md, etc.),
REQUIREMENTS.md, and for deployable systems a mandatory BOOT-SEQUENCE.md.

### `/gsd:director review [phase]`

QA a GSD phase plan before execution. Checks:

- **Spec conformance** — Does the plan contradict canonical specs?
- **Runtime path** — Walk through what happens when the built artifact runs. Are all
  preconditions met?
- **Dependencies** — Are wave dependencies correct? Does the plan assume outputs from
  plans that haven't run?
- **Completeness** — Are there requirements the planner missed? Does it defer work
  that should be done now?
- **Failure prediction** — Based on accumulated patterns, what will the executor get
  wrong?

Issues are categorized: BLOCK (must fix), FLAG (likely problem), NOTE (minor).

### `/gsd:director checkpoint [phase]`

Verify execution output. The user relays errors, behavior, or agent actions from the
executor session. The director:

- Diagnoses against specs and runtime model
- Prescribes the fix (never defers, never says "mark as known issue")
- Catalogs the failure pattern for the retrospective

### `/gsd:director watch [interval]`

Poll `.planning/` for GSD activity. Alerts when new plans or summaries appear.
Default interval: 5 minutes.

```
Watching .planning/ for changes (every 5m)...

[10:10] Change detected:
  - NEW: .planning/phases/10-auto-update/10-01-PLAN.md
  - NEW: .planning/phases/10-auto-update/10-02-PLAN.md

  Phase 10 plans created. Review? [y/n/details]
```

The director session stays alive between polls, holding full context.

### `/gsd:director retro`

Post-milestone retrospective. Reviews what was caught, what was missed, extracts
failure patterns, and updates RETROSPECTIVE.md. The retrospective is cumulative —
each milestone adds learnings, nothing is removed. Future director sessions load it
and apply all accumulated rules.

### `/gsd:director` (no args)

Auto-resumes from `.planning/director/SESSION-STATE.md` if it exists. Summarizes
the last session's state, reports what changed in `.planning/` since then, and asks
how to proceed.

## Director's Workspace

```
.planning/director/
  SESSION-STATE.md   Mental model snapshot — what we're building, key constraints,
                     open items, what to do next. Updated on passivation.
  DECISIONS.md       Why decisions were made + anti-patterns (what the agent will
                     try wrong). The reasoning that canonical specs don't capture.
  WATCH-LIST.md      Failure patterns to check for. Accumulated from RETROSPECTIVE.md
                     and current milestone observations.
```

## Canonical Specs

The director produces specs organized by GSD agent role:

| Spec | Who reads it | What it contains |
|------|-------------|-----------------|
| ARCHITECTURE.md | Planner, plan-checker | Decisions, constraints, prohibitions |
| CONTRACTS.md | Executor, debugger | APIs, protocols, state machines |
| INVARIANTS.md | Verifier, auditor | Testable assertions |
| TEST-STRATEGY.md | Test agents | Test layers, coverage |
| UI-SPEC.md | UI agents | Screens, rendering, interaction |
| BOOT-SEQUENCE.md | Planner, executor | State machine from zero to running |

Plans that contradict a canonical spec are invalid. If a spec needs to change,
update the spec first (with justification), then plan against the updated spec.

## Typical Workflow

```
1. Start milestone
   /gsd:director design
   - Discuss requirements, architecture, technology choices
   - Produce canonical specs + REQUIREMENTS.md
   - Director passivates state

2. GSD creates roadmap (separate session)
   /gsd:new-milestone or manual ROADMAP.md

3. Director reviews roadmap
   /gsd:director review
   - Check phase sequencing, requirement coverage, runtime path

4. GSD plans each phase (separate session)
   /gsd:plan-phase N

5. Director reviews each plan
   /gsd:director review N
   - Check against specs, trace runtime, predict failures

6. GSD executes (separate session)
   /gsd:execute-phase N

7. Director monitors (watch mode or user-relayed checkpoints)
   /gsd:director watch
   /gsd:director checkpoint N

8. End of milestone
   /gsd:director retro
   - Extract patterns, update RETROSPECTIVE.md, improve for next time
```

## Installation

The director skill comes with the zerobias-org GSD fork. To install:

```bash
# Clone the fork
git clone git@github.com:zerobias-org/get-shit-done.git

# Copy skill files to your GSD installation
cp get-shit-done/commands/gsd/director.md ~/.claude/get-shit-done/../commands/gsd/
cp get-shit-done/get-shit-done/workflows/director.md ~/.claude/get-shit-done/workflows/
```

Or install the full fork as your GSD:
```bash
npx zerobias-org/get-shit-done --claude --global
```
