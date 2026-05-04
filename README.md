# FSM Extractor — Claude Code Skill

A Claude Code custom slash command that extracts finite state machine patterns from source code and generates interactive Mermaid diagram viewers. Automatically identifies and isolates multiple independent FSMs in a codebase, producing a separate diagram and viewer for each.

## What is a Claude Code skill?

A skill is a markdown file (`.md`) that acts as a reusable prompt template for Claude Code. When you type `/fsm` in Claude Code, the contents of `fsm.md` are expanded into instructions that Claude follows — reading your files, analyzing them, and writing output — all within a single invocation. No external scripts, no dependencies, no API keys beyond what Claude Code already uses.

Skills live in `.claude/commands/` (project-level) or `~/.claude/commands/` (global). The filename becomes the slash command name.

## Installation

**Project-level** (available to anyone who clones the repo):

```bash
mkdir -p .claude/commands
cp /path/to/robot-fsm-skill/fsm.md .claude/commands/fsm.md
```

**Global** (available in every project on your machine):

```bash
mkdir -p ~/.claude/commands
cp /path/to/robot-fsm-skill/fsm.md ~/.claude/commands/fsm.md
```

**One-liner from GitHub** (global install):

```bash
mkdir -p ~/.claude/commands && curl -o ~/.claude/commands/fsm.md https://raw.githubusercontent.com/YOURUSER/robot-fsm-skill/master/fsm.md
```

## Usage

Inside Claude Code:

```
/fsm ./src
```

Analyze the current directory:

```
/fsm
```

Analyze a specific subdirectory:

```
/fsm ./src/controllers
```

## What it does

1. **Scans** the target path for source files (`.py`, `.js`, `.ts`, `.cpp`, `.c`, `.java`, `.cs`, `.go`, `.rs`)
2. **Reads** relevant files, prioritizing those with state-related names when there are many
3. **Identifies** each independent state machine — separating them by state variable, class, or module boundaries
4. **Generates** a Mermaid `stateDiagram-v2` diagram per FSM
5. **Writes** output to `fsm-output/`:
   - `<fsm-name>.mmd` — raw Mermaid diagram for each FSM
   - `<fsm-name>.html` — interactive browser viewer per FSM (zoom, pan, SVG/PNG export)
   - `index.html` — landing page linking to all discovered FSMs with descriptions

## What it detects

| Pattern | Example |
|---------|---------|
| State enums/variables | `state = "IDLE"`, `enum State { ... }` |
| Transition functions | `transition_to()`, `setState()`, `changeState()` |
| Switch/case on state | `switch (this.state) { case RUNNING: ... }` |
| If/else state chains | `if self.state == "moving": ...` |
| Event handlers | `on("stop", () => state = STOPPED)` |
| Class-per-state pattern | `class IdleState`, `class RunningState` |
| Implicit boolean states | `is_moving`, `is_paused`, `has_error` flags |
| Cross-FSM dependencies | One machine triggering transitions in another |

## How it separates multiple FSMs

The skill considers state machines **independent** when they:
- Use different state variables or enums
- Live in different classes, modules, or subsystems
- Can transition without depending on each other's state

It **combines** them into one diagram only when they share the same state variable or are tightly coupled (transitions depend on each other's current state).

## Output structure

```
fsm-output/
  index.html                  # Landing page linking to all FSMs
  navigation-controller.mmd   # Raw Mermaid for FSM 1
  navigation-controller.html  # Interactive viewer for FSM 1
  gripper-state-machine.mmd   # Raw Mermaid for FSM 2
  gripper-state-machine.html  # Interactive viewer for FSM 2
  ...
```

If only one FSM is found, files are named `state-machine.mmd` and `view-diagram.html`.

## Requirements

- [Claude Code CLI](https://claude.com/claude-code)

No other dependencies. The HTML viewers load Mermaid and svg-pan-zoom from CDN at runtime.

## How it works (for AI context)

This section helps Claude Code (or other LLMs) understand the skill when referenced.

The skill file (`fsm.md`) contains:
- `$ARGUMENTS` placeholder — replaced with whatever the user types after `/fsm`
- Step-by-step instructions for file discovery, reading, analysis, and output generation
- Complete HTML templates (with `%%MERMAID_DIAGRAM%%` and `%%FSM_TITLE%%` placeholders) that get written as output files
- Criteria for separating vs. combining state machines

The skill uses Claude Code's built-in tools (Glob, Read, Write) — it does NOT shell out to any external process. Claude reasons over the code directly as the LLM, rather than calling itself via subprocess.

## License

MIT
