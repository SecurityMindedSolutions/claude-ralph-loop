# Ralph Loop Toolkit

An autonomous task runner for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). You write a PRD with checkboxed tasks, Ralph works through them one at a time - no human in the loop.

## Two Components

### `/ralph-plan` - Interactive PRD Builder

A Claude Code skill that guides you through creating a structured PRD. It explores your codebase, helps you break work into small tasks with acceptance criteria, and writes the plan to `tasks/ralph/{name}/prd.md`.

See [ralph-plan/README.md](ralph-plan/README.md) for details.

### `/ralph-loop` - Autonomous Task Runner

A bash loop that invokes `claude -p` for each task. Each iteration reads the PRD, picks the next incomplete task, implements it, runs verification, marks it done, and logs progress. Iterations are stateless - only the PRD and progress log on disk carry context between runs.

See [ralph-loop/README.md](ralph-loop/README.md) for details.

## Installation

```bash
git clone https://github.com/SecurityMindedSolutions/claude-ralph-loop.git
cd claude-ralph-loop

cp -R ralph-loop ~/.claude/skills/ralph-loop
cp -R ralph-plan ~/.claude/skills/ralph-plan
chmod +x ~/.claude/skills/ralph-loop/ralph.sh
```

Requires [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) and Bash 4+.

## Quick Start

```
/ralph-plan              # Create a plan interactively
/ralph-loop my-plan      # Run it (up to 10 iterations)
/ralph-loop my-plan 25   # Run with higher iteration limit
```

## License

[MIT](LICENSE)
