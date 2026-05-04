# FSM Extractor — Claude Code Skill

A Claude Code custom slash command that extracts finite state machine patterns from source code and generates interactive Mermaid diagram viewers. Automatically identifies and isolates multiple independent FSMs in a codebase, producing a separate diagram and viewer for each.

## Installation

Copy `fsm.md` into your project's `.claude/commands/` directory:

```bash
# From your project root
mkdir -p .claude/commands
cp /path/to/fsm.md .claude/commands/fsm.md
```

Or install it globally so it's available in every project:

```bash
# Global commands directory
mkdir -p ~/.claude/commands
cp /path/to/fsm.md ~/.claude/commands/fsm.md
```

## Usage

Inside Claude Code, run:

```
/fsm ./src
```

Or with no argument to analyze the current directory:

```
/fsm
```

## What it does

1. Scans the target path for source files (`.py`, `.js`, `.ts`, `.cpp`, `.c`, `.java`, `.cs`, `.go`, `.rs`)
2. Reads the files and analyzes them for state machine patterns
3. Identifies and isolates each independent FSM
4. Generates a Mermaid `stateDiagram-v2` diagram per FSM
5. Writes output to `fsm-output/`:
   - `<fsm-name>.mmd` — raw Mermaid diagram for each FSM
   - `<fsm-name>.html` — interactive browser viewer per FSM with zoom/pan and SVG/PNG export
   - `index.html` — landing page linking to all discovered FSMs

## What it detects

- Explicit state variables and enums
- State transition functions
- Switch/case on state variables
- If/else chains checking state
- Event handlers that change state
- State pattern implementations (class-per-state)
- Implicit states from boolean flags

## Requirements

- [Claude Code CLI](https://claude.com/claude-code)

No other dependencies. The HTML viewer loads Mermaid and svg-pan-zoom from CDN.

## License

MIT
