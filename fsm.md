# FSM Extractor Skill

Analyze source code to extract finite state machine patterns and generate interactive Mermaid diagram viewers. Identifies and isolates multiple independent FSMs in a codebase, producing a separate diagram and viewer for each.

## Input

The user will provide a path to analyze: $ARGUMENTS

If no path is provided, use the current working directory.

## Steps

### 1. Find source files

Use the Glob tool to find source files in the target path. Search for these patterns:
- `**/*.py`
- `**/*.js`
- `**/*.ts`
- `**/*.cpp`
- `**/*.c`
- `**/*.java`
- `**/*.cs`
- `**/*.go`
- `**/*.rs`

Exclude files in `node_modules`, `venv`, `.venv`, `build`, `dist`, `.git`, `__pycache__`, and `target` directories.

### 2. Read the files

Read each matching file. Skip files that are clearly not relevant (e.g., config files, lock files, generated code). If there are many files (>15), prioritize files whose names suggest state logic: files containing words like "state", "controller", "machine", "fsm", "robot", "handler", "manager", "engine", or "mode".

### 3. Identify and isolate distinct FSMs

Analyze the code to find ALL separate state machines. A codebase often contains multiple independent FSMs. Identify each one separately by looking for:

- Explicit state variables or enums (e.g., `state = "IDLE"`, `State.RUNNING`, `enum State`)
- State transition functions (e.g., `transition_to()`, `setState()`, `changeState()`)
- Switch/case statements on state variables
- If/else chains checking state
- Event handlers that change state
- Robot control states (idle, moving, stopped, error, etc.)
- State pattern implementations (classes per state)
- Flags that form implicit states (e.g., `is_moving`, `is_paused`, `has_error`)

For each FSM you find, record:
- A short descriptive name (e.g., "NavigationController", "GripperStateMachine", "ConnectionManager")
- Which files and code sections it spans
- Its states and transitions
- How it is independent from or interacts with other FSMs in the codebase

Two state machines are **separate** if they:
- Use different state variables or enums
- Live in different classes, modules, or subsystems
- Can transition independently of each other

Two state machines should be **combined** only if they:
- Share the same state variable
- Are tightly coupled with transitions that depend on each other's state

### 4. Generate Mermaid diagrams

For **each** identified FSM, produce a `stateDiagram-v2` Mermaid diagram that captures:
- All identified states for that FSM
- Transitions between states, labeled with the events/conditions that trigger them
- Start state (`[*] --> InitialState`)
- End states if applicable (`FinalState --> [*]`)
- Use clear, descriptive PascalCase state names
- Add notes for complex states if needed
- If two FSMs interact (one triggers transitions in another), add a note on the relevant transition indicating the cross-FSM dependency

### 5. Write output files

Create an output directory called `fsm-output` inside the target path (or current working directory if none was given).

For **each** FSM identified, write two files using a kebab-case version of the FSM name:

**a) `fsm-output/<fsm-name>.mmd`** — the raw Mermaid diagram code for that FSM.

**b) `fsm-output/<fsm-name>.html`** — a self-contained interactive HTML viewer for that FSM.

If only one FSM is found, name the files `state-machine.mmd` and `view-diagram.html`.

Also write **c) `fsm-output/index.html`** — a landing page that links to all the individual viewers, showing each FSM's name and a brief description. Use this template, replacing `%%FSM_LIST%%` with an `<li>` for each FSM:

```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>State Machines</title>
    <style>
      body {
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Arial, sans-serif;
        margin: 0;
        padding: 40px;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        min-height: 100vh;
      }
      .container {
        max-width: 800px;
        margin: 0 auto;
      }
      h1 {
        color: white;
        text-align: center;
        margin-bottom: 10px;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
      }
      .subtitle {
        color: rgba(255, 255, 255, 0.9);
        text-align: center;
        margin-bottom: 40px;
        font-size: 14px;
      }
      .fsm-list {
        list-style: none;
        padding: 0;
      }
      .fsm-list li {
        background: white;
        border-radius: 12px;
        padding: 24px 30px;
        margin-bottom: 16px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        transition: transform 0.2s ease, box-shadow 0.2s ease;
      }
      .fsm-list li:hover {
        transform: translateY(-2px);
        box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
      }
      .fsm-list a {
        text-decoration: none;
        color: #667eea;
        font-size: 20px;
        font-weight: 600;
      }
      .fsm-list a:hover {
        color: #5568d3;
      }
      .fsm-desc {
        color: #666;
        font-size: 14px;
        margin-top: 6px;
      }
      .fsm-files {
        color: #999;
        font-size: 12px;
        margin-top: 4px;
        font-family: monospace;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h1>State Machines</h1>
      <p class="subtitle">Extracted from source code by FSM Extractor</p>
      <ul class="fsm-list">
        %%FSM_LIST%%
      </ul>
    </div>
  </body>
</html>
```

Each `<li>` should follow this format:
```html
<li>
  <a href="<fsm-name>.html">FSM Display Name</a>
  <div class="fsm-desc">Brief description of what this state machine controls</div>
  <div class="fsm-files">Files: file1.py, file2.py</div>
</li>
```

For each individual FSM viewer HTML, use this template, replacing `%%MERMAID_DIAGRAM%%` with the Mermaid code and `%%FSM_TITLE%%` with the FSM's display name:

```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>%%FSM_TITLE%% - State Machine Diagram</title>
    <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/svg-pan-zoom@3.6.1/dist/svg-pan-zoom.min.js"></script>
    <style>
      body {
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Arial, sans-serif;
        margin: 0;
        padding: 20px;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        min-height: 100vh;
      }
      .container {
        max-width: 100%;
        margin: 0 auto;
      }
      h1 {
        color: white;
        text-align: center;
        margin-bottom: 10px;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
      }
      .subtitle {
        color: rgba(255, 255, 255, 0.9);
        text-align: center;
        margin-bottom: 30px;
        font-size: 14px;
      }
      .controls {
        background: white;
        padding: 15px;
        border-radius: 8px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        margin-bottom: 20px;
        display: flex;
        justify-content: center;
        flex-wrap: wrap;
        gap: 10px;
      }
      button {
        padding: 10px 20px;
        background: #667eea;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.3s ease;
      }
      button:hover {
        background: #5568d3;
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
      }
      button:active {
        transform: translateY(0);
      }
      .back-link {
        display: inline-block;
        color: rgba(255, 255, 255, 0.9);
        text-decoration: none;
        margin-bottom: 20px;
        font-size: 14px;
      }
      .back-link:hover {
        color: white;
      }
      #diagram-container {
        background: white;
        padding: 30px;
        border-radius: 12px;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        overflow: visible;
        width: 90%;
        max-width: 1400px;
        height: 700px;
        margin: 0 auto;
        position: relative;
      }
      #diagram-container svg {
        width: 100%;
        height: 100%;
      }
      .mermaid {
        width: 100%;
        height: 100%;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <a class="back-link" href="index.html">&larr; All State Machines</a>
      <h1>%%FSM_TITLE%%</h1>
      <p class="subtitle">Generated by FSM Extractor</p>

      <div class="controls">
        <button onclick="downloadSVG()">Download SVG</button>
        <button onclick="downloadPNG()">Download PNG</button>
      </div>

      <div id="diagram-container">
        <div class="mermaid">%%MERMAID_DIAGRAM%%</div>
      </div>
    </div>

    <script>
      mermaid.initialize({
        startOnLoad: true,
        theme: "default",
        securityLevel: "loose",
        flowchart: { useMaxWidth: false },
      });

      let panZoomInstance = null;

      window.addEventListener("load", () => {
        setTimeout(() => {
          const svg = document.querySelector("#diagram-container svg");
          if (svg && typeof svgPanZoom !== "undefined") {
            panZoomInstance = svgPanZoom(svg, {
              zoomEnabled: true,
              controlIconsEnabled: true,
              fit: true,
              center: true,
              minZoom: 0.3,
              maxZoom: 10,
              zoomScaleSensitivity: 0.3,
              dblClickZoomEnabled: true,
              mouseWheelZoomEnabled: true,
              preventMouseEventsDefault: true,
              contain: false,
            });
          }
        }, 500);
      });

      function downloadSVG() {
        const svg = document.querySelector("#diagram-container svg");
        if (svg) {
          const svgString = new XMLSerializer().serializeToString(svg);
          const blob = new Blob([svgString], { type: "image/svg+xml" });
          const a = document.createElement("a");
          a.href = URL.createObjectURL(blob);
          a.download = "%%FSM_TITLE%%.svg";
          a.click();
          URL.revokeObjectURL(a.href);
        }
      }

      function downloadPNG() {
        const svg = document.querySelector("#diagram-container svg");
        if (svg) {
          const canvas = document.createElement("canvas");
          const ctx = canvas.getContext("2d");
          const svgData = new XMLSerializer().serializeToString(svg);
          const img = new Image();
          img.onload = function () {
            canvas.width = img.width * 2;
            canvas.height = img.height * 2;
            ctx.fillStyle = "white";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            canvas.toBlob(function (blob) {
              const a = document.createElement("a");
              a.href = URL.createObjectURL(blob);
              a.download = "%%FSM_TITLE%%.png";
              a.click();
              URL.revokeObjectURL(a.href);
            });
          };
          img.src = "data:image/svg+xml;base64," + btoa(unescape(encodeURIComponent(svgData)));
        }
      }
    </script>
  </body>
</html>
```

### 6. Report to the user

Tell the user:
- How many FSMs were found and a brief summary of each (name, states, key transitions)
- Each Mermaid diagram in its own ```mermaid code block so they can see them inline
- If FSMs interact with each other, describe the dependencies
- The paths to all output files
- That they can open `index.html` in their browser to navigate between all diagrams, or open any individual viewer directly
