---
name: vscode-debugging
description: Configure VSCode debugging for Node.js + TypeScript projects
version: 1.0.0
source: session-extraction
---

# VSCode Debugging Configuration for Node.js + TypeScript

Configure breakpoint debugging in VSCode for Node.js + TypeScript projects.

## When to Use

- Setting up debugging for a new Node.js + TypeScript project
- Adding debug configurations to an existing project
- Debugging CLI tools, servers, or test files

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| Node.js >= 18 | Runtime environment |
| VSCode | With built-in JavaScript Debugger |
| tsx (recommended) | Run TypeScript directly without build |

## Quick Setup

### 1. Create `.vscode/launch.json`

```json
{
  "version": "2.0.0",
  "configurations": [
    {
      "name": "Debug CLI (tsx)",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["tsx"],
      "args": ["src/cli/index.ts"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "name": "Debug with Args",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["tsx"],
      "args": ["src/cli/index.ts", "${input:args}"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "name": "Debug Vitest (current file)",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["vitest", "run", "--no-file-parallelism"],
      "args": ["${file}"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "name": "Debug Compiled JS",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/dist/cli/index.js",
      "args": [],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "sourceMaps": true,
      "skipFiles": ["<node_internals>/**"]
    }
  ],
  "inputs": [
    {
      "id": "args",
      "type": "promptString",
      "description": "Command arguments",
      "default": ""
    }
  ]
}
```

### 2. Enable Source Maps (for compiled JS debugging)

Add to `tsconfig.json`:

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "declarationMap": true
  }
}
```

## Configuration Types

| Config | Use Case | Requirements |
|--------|----------|--------------|
| Debug CLI (tsx) | Direct TypeScript debugging | tsx in devDependencies |
| Debug with Args | Run with custom arguments | tsx in devDependencies |
| Debug Vitest | Debug test files | vitest in devDependencies |
| Debug Compiled JS | Debug after `npm run build` | sourceMap: true in tsconfig |

## Key Points

1. **tsx is preferred** - Runs TypeScript directly, no build step needed
2. **sourceMaps required** - Only for debugging compiled JavaScript
3. **${workspaceFolder}** - Adjust if .vscode is in a subdirectory
4. **skipFiles** - Hides Node.js internals from debugger

## Monorepo / Subdirectory Setup

If the Node.js project is in a subdirectory (e.g., `packages/cli/`):

```json
{
  "name": "Debug CLI",
  "args": ["packages/cli/src/index.ts"],
  "cwd": "${workspaceFolder}"
}
```

Or place `.vscode/` in the subdirectory and use:

```json
{
  "cwd": "${workspaceFolder}/packages/cli"
}
```

## Testing the Setup

1. Open a TypeScript file
2. Click left of line number to set a breakpoint (red dot)
3. Press `F5` to start debugging
4. Select a configuration from the dropdown

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Breakpoint not hit | Ensure correct `cwd` path |
| Source map errors | Add `"sourceMap": true` to tsconfig |
| Module not found | Check `args` path is relative to `cwd` |
| tsx not found | Run `npm install` or add tsx to devDependencies |
