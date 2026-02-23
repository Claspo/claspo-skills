---
name: setup-claspo-starter
description: >
  Set up and run the Claspo Plugin Starter project locally. Use when user wants to:
  install claspo-plugin-starter, set up the starter project, run the claspo
  development environment, start the claspo editor locally, get started with
  claspo plugin development, or needs help with Node.js setup for claspo projects.
---

# Setup Claspo Starter

You are helping a non-technical user set up the Claspo Plugin Starter project on their local machine. Walk through each step interactively — explain what you're doing before each action, and ask for confirmation before making any system-level changes.

## Safety Rules

- Always explain what a command does before running it
- Ask confirmation before: installing nvm, changing Node version, killing processes on ports
- Never use `sudo` for npm commands
- Never modify files outside the project directory except `~/.nvm` (if user approves nvm installation)

## Step 1: Locate or Clone the Project

Check if the current directory is the claspo-plugin-starter project:

1. Look for `claspo.config.js` in the current directory
2. Check `package.json` for `"name": "claspo-plugin-starter"`

If both exist → proceed to Step 2.

If not found:
- Ask the user if they already have the project cloned somewhere else
- If no: offer to clone it from `https://github.com/Claspo/claspo-plugin-starter.git`
- After cloning, `cd` into the project directory

## Step 2: Detect Platform

Run `uname -s` to detect the operating system:

| Output | Platform |
|---|---|
| `Darwin` | macOS |
| `Linux` | Linux |
| `MINGW*` / `MSYS*` / `CYGWIN*` | Windows (Git Bash / MSYS2) |

If Windows without WSL:
- Warn the user that native Windows support is limited
- Suggest installing WSL (Windows Subsystem for Linux) first
- Provide link: https://learn.microsoft.com/en-us/windows/wsl/install
- If user wants to continue anyway, proceed but note potential issues

## Step 3: Check Node Version Manager

Detect if the user has a Node version manager installed. Check in this order:

**nvm** (nvm is a shell function, not a binary — do not use `which nvm`):
1. Check if `$NVM_DIR/nvm.sh` exists
2. Check if `$HOME/.nvm/nvm.sh` exists
3. On macOS, check Homebrew paths: `$(brew --prefix 2>/dev/null)/opt/nvm/nvm.sh`
4. Try `type nvm 2>/dev/null` to see if the function is loaded

**Alternatives:**
- `fnm`: check `which fnm`
- `volta`: check `which volta`

If a version manager is found → proceed to Step 4 using that manager.

If none found:
- Explain what nvm is: "nvm (Node Version Manager) lets you install and switch between Node.js versions without affecting your system. It installs to `~/.nvm` and adds a line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.)."
- Ask the user: "Would you like me to install nvm? This will create `~/.nvm` and update your shell profile."
- If yes: install via `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash`, then source the nvm script
- If no: fall back to checking the system Node directly in Step 4

## Step 4: Check Node.js Version

Required: Node.js >= 22.20.0 (from `package.json` engines field).

Run `node --version` and parse the version number.

**If Node is missing or version is too old:**

With nvm/fnm/volta:
- Run `nvm install` (reads version from `.nvmrc`) or the equivalent for fnm/volta
- Then `nvm use`

Without a version manager:
- Direct the user to https://nodejs.org to download Node.js 22.x LTS
- Wait for them to install and confirm

**After Node is ready:**
- Verify with `node --version` (should show v22.20.0+)
- Check `npm --version` (should be >= 8, which ships with Node 22)

## Step 5: Install Dependencies

Check if dependencies are already installed by looking for `node_modules/` in:
- Project root
- `quickstart/backend/`
- `quickstart/frontend/`

If all three exist: ask the user "Dependencies appear to be installed already. Would you like to reinstall them (clean install) or skip this step?"

To install, run from the project root:
```bash
npm run install:all
```

This runs: root `npm install` → backend `npm install` → frontend `npm install`.

**Common failure handling:**

| Error | Solution |
|---|---|
| `ERESOLVE` (dependency conflicts) | Try `npm install --legacy-peer-deps` in the failing directory |
| `EACCES` (permission denied) | Never use sudo. Check ownership of `~/.npm` and `node_modules/` |
| Network/timeout errors | Ask user to check internet connection, try again |
| `404 Not Found` for `@claspo/*` packages | Check `.npmrc` for registry configuration |

## Step 6: Check Port Availability

The dev servers use these ports:

| Service | Port |
|---|---|
| Components dev server | 9590 |
| Backend API | 3100 |
| Frontend (Editor) | 4202 |

For each port, check if it's in use:
```bash
lsof -i :<port> -sTCP:LISTEN
```

If a port is occupied:
- Identify the process name and PID
- Tell the user: "Port <port> is in use by <process> (PID <pid>). Would you like me to stop it?"
- Only kill the process if the user confirms

## Step 7: Start Dev Servers

Run from the project root:
```bash
npm run dev
```

Explain what starts:
- **Components dev server** on `http://localhost:9590` — serves compiled widget components
- **Backend API** on `http://localhost:3100` — mock Claspo API for local development
- **Frontend (Editor)** on `http://localhost:4202` — the Claspo visual editor

**Important:** Vite's `--open` flag opens `http://localhost:4202/` (the index page). The user should navigate to `http://localhost:4202/editor.html` to see the Claspo editor.

Run `npm run dev` in background mode so the terminal remains available for health checks.

## Step 8: Verify Setup

Wait a few seconds for servers to start, then run health checks:

1. **Components server:**
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://localhost:9590/
   ```
   Expected: `200`

2. **Backend API:**
   ```bash
   curl -s http://localhost:3100/health
   ```
   Expected: `{"status":"ok",...}`

3. **Frontend (Editor):**
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://localhost:4202/editor.html
   ```
   Expected: `200`

If any check fails:
- Check the server output in the terminal for compilation errors
- Suggest the user scroll up in the terminal to see error messages
- Common issues: TypeScript compilation errors, missing env variables

Report results to the user in a clear summary table.

## Step 9: Next Steps

Once all checks pass, summarize:

**What's running:**
- Claspo Editor at `http://localhost:4202/editor.html`
- Component renderer at `http://localhost:9590`
- Backend API at `http://localhost:3100`

**Useful commands:**
- Stop all servers: `Ctrl+C` in the terminal running `npm run dev`
- Restart servers: `npm run dev`
- Production build: `npm run build`

**Next:** To create your first custom component, use the `create-claspo-component` skill — just say "create a new claspo component".

**Documentation:** https://docs.claspo.io
