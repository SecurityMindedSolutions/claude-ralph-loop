# Ralph Loop Toolkit

An autonomous task runner for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that works through a PRD (Product Requirements Document) one task at a time — no human intervention required between iterations.

## Why Ralph Exists

Claude Code is powerful but session-scoped. Complex projects require multiple iterations, and manually re-prompting after each task is tedious. Ralph solves this by:

1. Reading a structured PRD with checkboxed tasks
2. Picking the next incomplete task
3. Implementing it in a fresh Claude session
4. Marking it done and logging progress
5. Repeating until all tasks are complete (or hitting an iteration limit)

Each iteration is stateless — Ralph reads the PRD and progress log from disk, so it never loses context between runs. The PRD is the single source of truth.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  /ralph-plan │────▶│  prd.md      │────▶│  ralph.sh loop   │
│  (interactive│     │  progress.md │     │  (autonomous)    │
│   planning)  │     └──────────────┘     └────────┬────────┘
└─────────────┘                                    │
                                              per iteration:
                                           1. Read PRD + progress
                                           2. Pick next - [ ] task
                                           3. Mark - [~] (in progress)
                                           4. Implement with Claude
                                           5. Run verification
                                           6. Mark - [x] (done)
                                           7. Log to progress.md
```

### The Loop

`ralph.sh` runs a bash loop that invokes `claude -p` (Claude Code in prompt mode) for each iteration. The prompt is built from `prompt-template.md` with the PRD contents, progress log, iteration number, and working directory injected.

Each Claude session:
- Sees the full PRD with task status (`[ ]`, `[~]`, `[x]`)
- Reads the progress log to understand prior decisions
- Works on exactly one task
- Updates both files on disk before exiting

### Signals

- `RALPH_COMPLETE` — All tasks marked `[x]`, loop exits
- `RALPH_BLOCKED` — Current task can't proceed, skip to next

## Installation

Clone the repo and copy the skills into your Claude Code config directory:

```bash
# Clone
git clone https://github.com/SecurityMindedSolutions/claude-ralph-loop.git
cd claude-ralph-loop

# Install both skills
cp -R ralph-loop ~/.claude/skills/ralph-loop
cp -R ralph-plan ~/.claude/skills/ralph-plan
chmod +x ~/.claude/skills/ralph-loop/ralph.sh
```

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Bash 4+

## Usage

### Create a Plan

Inside a Claude Code session:

```
/ralph-plan
```

This starts an interactive conversation that produces:
- `tasks/ralph/{name}/prd.md` — the project requirements
- `tasks/ralph/{name}/progress.md` — empty progress log

### Run the Loop

From the terminal (standalone):

```bash
~/.claude/skills/ralph-loop/ralph.sh my-plan          # Up to 10 iterations (default)
~/.claude/skills/ralph-loop/ralph.sh my-plan 25       # Up to 25 iterations
~/.claude/skills/ralph-loop/ralph.sh my-plan --dry-run   # Preview the prompt
~/.claude/skills/ralph-loop/ralph.sh my-plan --status    # Check completion status
~/.claude/skills/ralph-loop/ralph.sh --list              # List all plans
```

Or from within a Claude Code session (runs in background with progress monitoring):

```
/ralph-loop my-plan
/ralph-loop my-plan 50
```

### PRD Format

```markdown
# PRD: Project Name

## Goal
One-line description

## Context
- Background, constraints, tech stack
- **Plan directory**: `tasks/ralph/{name}/`

## Tasks

### Phase 1: Foundation
- [ ] Task with explicit file paths. **Done when**: criteria
- [ ] Another task. **Done when**: criteria

### Phase 2: Core
- [ ] Task. **Done when**: criteria

## Verification
npm run lint && npm run build

## Notes
- Project conventions Ralph should know
```

Key principle: **explicit file paths everywhere**. Ralph starts each iteration with zero context — it must never guess where files live.

## Directory Structure

```
claude-ralph-loop/
├── README.md                   # This file
├── LICENSE                     # MIT
├── ralph-loop/                 # → copy to ~/.claude/skills/ralph-loop/
│   ├── SKILL.md                # /ralph-loop skill (monitor + launcher)
│   ├── ralph.sh                # Core loop script
│   ├── strip_codeblocks.pl     # Helper: strips fenced code blocks from markdown
│   ├── prompt-template.md      # Template injected into each Claude session
│   └── prd-template.md         # Blank PRD template for new plans
└── ralph-plan/                 # → copy to ~/.claude/skills/ralph-plan/
    └── SKILL.md                # /ralph-plan skill (interactive PRD builder)
```

## Design Decisions

- **Stateless iterations**: Each Claude session is independent. No conversation history carries over — only the PRD and progress log on disk. This makes the system resilient to crashes and context limits.
- **One task per iteration**: Keeps each session focused and prevents scope creep. If a task is too large, break it down in the PRD.
- **No git operations**: Ralph never commits. The user reviews and commits when ready.
- **Bash over plugins**: The core loop is a plain bash script, not a Claude Code plugin. This makes it portable, debuggable, and easy to modify.

## License

[MIT](LICENSE)
