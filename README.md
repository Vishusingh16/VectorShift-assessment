# VectorShift Pipeline Builder

A visual drag-and-drop pipeline builder with a React frontend and Python/FastAPI backend.

---
<img width="1895" height="802" alt="Image 1 of vector shift" src="https://github.com/user-attachments/assets/cf88c032-cf66-4977-b723-371924844256" />
<img width="1892" height="836" alt="Image 3 vector shift" src="https://github.com/user-attachments/assets/8bfe3929-0edb-4cd6-8acc-1d2108cc3261" />
<img width="1913" height="872" alt="Image2 Vectorshift" src="https://github.com/user-attachments/assets/6521c2d0-1a37-49f4-a47a-0b233bd5b2cd" />

## Prerequisites

- **Node.js** (v16+) and **npm**
- **Python** (v3.9+) and **pip**

---

## How to Run

### Step 1 -- Install and start the backend

```bash
cd backend
pip install fastapi uvicorn pydantic
uvicorn main:app --reload
```

The API starts at http://localhost:8000. Visit that URL to see `{"Ping":"Pong"}` and confirm it is running.

### Step 2 -- Install and start the frontend

Open a **second** terminal:

```bash
cd frontend
npm install
npm start
```

The React app opens at http://localhost:3000. Both servers must be running at the same time.

---

## What Was Implemented

### Part 1 -- Node Abstraction (`frontend/src/nodes/BaseNode.js`)

A single reusable `BaseNode` component provides the shared structure for every node:

- **Header** -- color-coded title bar styled via a CSS class per node type.
- **Body** -- renders whatever children are passed in (form fields, text, etc.).
- **Handles** -- input (left) and output (right) connection points with automatic even spacing and optional labels.
- **Helper components** -- `NodeTextField`, `NodeSelectField`, `NodeTextArea`, and `NodeField` for building node UIs with one line each.

Creating a brand-new node takes roughly 15 lines:

```jsx
import { useState } from 'react';
import { BaseNode, NodeTextField } from './BaseNode';

export const MyNode = ({ id }) => {
  const [value, setValue] = useState('');
  return (
    <BaseNode
      id={id}
      title="My Node"
      inputs={[{ id: 'in', label: 'Input' }]}
      outputs={[{ id: 'out' }]}
      className="my-node"
    >
      <NodeTextField
        label="Value"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
    </BaseNode>
  );
};
```

Then register it in `ui.js` (nodeTypes) and `toolbar.js` (DraggableNode entry).

The four original nodes were refactored to use the abstraction, and five new nodes were added to demonstrate its flexibility:

| Node        | Inputs               | Outputs              | Fields                    |
|-------------|----------------------|----------------------|---------------------------|
| Input       | --                   | value                | Name, Type (Text/File)    |
| Output      | value                | --                   | Name, Type (Text/Image)   |
| LLM         | system, prompt       | response             | Info text                 |
| Text        | dynamic per `{{var}}`| output               | Resizable textarea        |
| Filter      | data                 | match, reject        | Operator, Condition       |
| API         | headers, body        | response             | HTTP Method, URL          |
| Conditional | input                | true, false          | Condition type, Value     |
| Merge       | input_a, input_b     | output               | Strategy selector         |
| Note        | --                   | --                   | Annotation textarea       |

### Part 2 -- Styling (`frontend/src/index.css`)

Every component is styled into a clean, modern design:

- **Toolbar** -- white bar with color-bordered draggable chips for each node type.
- **Nodes** -- white cards with rounded corners, subtle box shadows, and a color-tinted header per type.
- **Handles** -- circular dots that highlight blue on hover.
- **Canvas** -- light-gray dotted grid with styled minimap and controls.
- **Submit** -- gradient button (blue-to-purple) with hover lift effect.
- **Alert modal** -- animated overlay with stat cards and a color-coded DAG badge.

### Part 3 -- Text Node Logic (`frontend/src/nodes/textNode.js`)

Two enhancements to the Text node:

1. **Dynamic resizing** -- the node's width grows with the longest line of text and the height grows with the number of lines, so everything stays visible.

2. **Variable handles** -- typing a valid JavaScript variable name inside `{{ }}` (e.g. `{{ name }}`) creates a new input handle on the left side of the node, labeled with the variable name. Handles update live as you type and duplicates are ignored. The node calls `useUpdateNodeInternals` so React Flow recalculates connection positions.

### Part 4 -- Backend Integration (`frontend/src/submit.js` + `backend/main.py`)

- **Submit button** reads nodes and edges from the Zustand store and sends a `POST` request to `http://localhost:8000/pipelines/parse` with `{ nodes, edges }`.
- **Backend endpoint** counts nodes and edges, then runs **Kahn's algorithm** (topological sort via in-degree tracking) to determine whether the graph is a Directed Acyclic Graph.
- **Response format**: `{ num_nodes: int, num_edges: int, is_dag: bool }`.
- **Alert modal** displays the result with node/edge counts and a green "Valid DAG" or red "Not a DAG" badge.
- **CORS middleware** is enabled so the frontend can reach the backend across origins.

---

## How to Test

### Test 1 -- Drag and drop nodes

Drag any of the nine node types from the toolbar onto the canvas. Each node should appear as a styled card with its own color-coded header.

### Test 2 -- Connect nodes

Drag from a green handle (right side of an Input node) to a purple handle (left side of an LLM node). A smooth animated edge should appear.

### Test 3 -- Text node dynamic resize

1. Drag a **Text** node onto the canvas.
2. Type a long sentence -- the node grows wider.
3. Press Enter multiple times -- the node grows taller.

### Test 4 -- Text node variable handles

1. In the Text node textarea, type `Hello {{ name }}, age {{ age }}`.
2. Two input handles appear on the left side, labeled **name** and **age**.
3. Delete `{{ age }}` -- the **age** handle disappears.
4. Connect another node's output to the **name** handle.

### Test 5 -- Submit a valid DAG

1. Create a chain: **Input** -> **LLM** -> **Output**.
2. Click **Submit Pipeline**.
3. The modal shows 3 nodes, 2 edges, and a green "Valid DAG" badge.

### Test 6 -- Submit a graph with a cycle

1. Connect two nodes in a loop (A -> B -> A).
2. Click **Submit Pipeline**.
3. The modal shows a red "Not a DAG -- Cycle detected" badge.

### Test 7 -- New node types

| Drag this  | Check for                                        |
|------------|--------------------------------------------------|
| Filter     | 1 input (Data), 2 outputs (Match, Reject)        |
| API        | 2 inputs (Headers, Body), 1 output (Response)    |
| Conditional| 1 input (Input), 2 outputs (True, False)         |
| Merge      | 2 inputs (Input A, Input B), 1 output            |
| Note       | No handles at all, dashed border, text area only |

---

## Project Structure

```
VectorShift assesment/
├── README.md                         # This file
├── backend/
│   └── main.py                       # FastAPI server
└── frontend/
    ├── package.json
    └── src/
        ├── App.js                    # Root layout
        ├── toolbar.js                # Draggable node toolbar
        ├── ui.js                     # React Flow canvas
        ├── submit.js                 # Submit button + alert modal
        ├── store.js                  # Zustand state (nodes, edges)
        ├── draggableNode.js          # Toolbar drag item component
        ├── index.css                 # All styles
        └── nodes/
            ├── BaseNode.js           # Shared node abstraction
            ├── inputNode.js          # Input node
            ├── outputNode.js         # Output node
            ├── llmNode.js            # LLM node
            ├── textNode.js           # Text node (dynamic + variables)
            ├── filterNode.js         # Filter node
            ├── apiNode.js            # API node
            ├── conditionalNode.js    # Conditional node
            ├── mergeNode.js          # Merge node
            └── noteNode.js           # Note/annotation node
```

## Tech Stack

| Layer    | Technology                     |
|----------|--------------------------------|
| Frontend | React 18, React Flow v11       |
| State    | Zustand v4                     |
| Backend  | Python, FastAPI, Pydantic      |
| Styling  | Plain CSS (no UI library)      |
