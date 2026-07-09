# OpenCode Goal Command

An OpenCode custom command that adds a durable, evidence-checked `/goal`
workflow to a project.

The command stores goal state in `.opencode/goals/current.md`, so a long-running
objective can be started, inspected, paused, resumed, edited, cleared, and
completed across multiple OpenCode turns.

## What It Provides

- `/goal <objective>` creates one active project goal.
- `/goal` or `/goal status` shows the current goal without starting work.
- `/goal resume` continues the next recorded checkpoint.
- `/goal pause` pauses goal execution.
- `/goal edit <objective>` updates the objective while preserving useful
  progress and evidence.
- `/goal complete` audits the work and marks the goal complete only when
  concrete evidence proves every requirement is satisfied.
- `/goal clear` archives and removes the current goal state.

This is a prompt-level implementation. It gives OpenCode a durable goal
lifecycle, but it does not provide true background scheduling or automatic idle
continuation by itself.

## Repository Layout

```text
.
├── opencode/
│   ├── agents/
│   │   └── goal-runner.md
│   ├── commands/
│   │   └── goal.md
│   ├── goals/
│   │   ├── .gitkeep
│   │   └── archive/
│   │       └── .gitkeep
│   ├── package.json
│   └── package-lock.json
├── .opencode/
│   └── ...
└── opencode-goal-command-plan.md
```

`opencode/` contains the reusable command package files. `.opencode/` mirrors
those files for local project use.

## Installation

Copy the command files into the target project's `.opencode` directory:

```sh
mkdir -p .opencode/commands .opencode/agents .opencode/goals/archive
cp opencode/commands/goal.md .opencode/commands/goal.md
cp opencode/agents/goal-runner.md .opencode/agents/goal-runner.md
touch .opencode/goals/.gitkeep .opencode/goals/archive/.gitkeep
```

If you are already working in this repository, the `.opencode/` directory is
already populated.

## Usage

Start a new goal:

```text
/goal Refactor the report exporter and prove the output is unchanged
```

Check the current goal:

```text
/goal
```

Continue work from the next recorded checkpoint:

```text
/goal resume
```

Pause work:

```text
/goal pause
```

Update the objective:

```text
/goal edit Refactor the report exporter and add regression tests
```

Run a completion audit:

```text
/goal complete
```

Archive and remove the current goal:

```text
/goal clear
```

## Goal State

Active state is stored at:

```text
.opencode/goals/current.md
```

Archived goals are stored under:

```text
.opencode/goals/archive/
```

The state file uses markdown with frontmatter fields such as `id`, `status`,
`created_at`, `updated_at`, `iteration`, `budget_iterations`,
`blocker_signature`, and `blocker_repeat_count`.

Supported statuses are:

```text
active | paused | blocked | complete | cleared | budget_limited
```

Only one unfinished goal may exist at a time. Starting a new objective while a
goal is active, paused, blocked, or budget-limited should be refused until the
current goal is completed, edited, or cleared.

## Design Notes

The command is designed around evidence-first progress:

- Work is split into bounded checkpoints.
- Progress is recorded in `.opencode/goals/current.md`.
- Completion requires an explicit audit against success criteria.
- Blocked status requires repeated evidence of the same blocker, not a single
  failed attempt.

See `opencode-goal-command-plan.md` for the original design plan and the mapping
from Codex-style goals to OpenCode custom commands.
