# Sandboxed AI Agent Devcontainers

Run Claude Code, OpenAI Codex, or Custom LLM (via Aider) inside hardened Docker containers so the AI agents can operate with full filesystem and network access without risking your host machine. Each container is purpose-built as the sandbox boundary — the agents run unrestricted *inside* the container while Docker provides the isolation.

This page includes three independent devcontainer configurations that share the same architecture but target different AI coding agents:

| Variant | Agent | Backend | Config Directory |
|---------|-------|---------|------------------|
| **Claude Code** | Anthropic Claude Code CLI | Claude Pro/Team/Enterprise subscription | `.devcontainer/claude-code/` |
| **Codex** | OpenAI Codex CLI | ChatGPT Plus/Team/Enterprise subscription | `.devcontainer/openai-codex/` |
| **Custom LLM** | Aider CLI | Any OpenAI-compatible API (on-prem LLMs, commercial APIs) | `.devcontainer/customllm-aider/` |

---

## Prerequisites

### Required Software

1. **Docker Desktop** (or Docker Engine on Linux)
   - macOS / Windows: [Install Docker Desktop](https://docs.docker.com/desktop/)
   - Linux: [Install Docker Engine](https://docs.docker.com/engine/install/)

2. **Visual Studio Code** with the **Dev Containers** extension
   - Install the extension: `ms-vscode-remote.remote-containers`
   - Or from the Extensions panel, search "Dev Containers"

3. **Git** — the host `~/.gitconfig` is mounted read-only into the container so your identity and preferences carry over.

### Optional (for terminal-only usage)

4. **Dev Container CLI** — allows building and running devcontainers without VS Code:

   ```bash
   npm install -g @devcontainers/cli
   ```

### Required Host Files

The containers expect two files on your host machine. If either is missing, the container will fail to start.

| Host Path | Purpose |
|-----------|---------|
| `~/.gitconfig` | Mounted read-only so `git commit` uses your name/email inside the container. |
| `~/.config/agent-guardrails/AGENTS.md` | Shared agent instructions file mounted read-only at `~/AGENTS.md` in the container. |

**Custom LLM only:** The `OPENAI_API_KEY` environment variable must be set in your host shell. Add it to your shell profile so it persists across sessions:

```bash
# Add to ~/.bashrc, ~/.zshrc, or equivalent
export OPENAI_API_KEY="your-api-key-here"
```

If you don't yet have a `~/.gitconfig`, initialize one:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

To create the guardrails file, copy the contents from the [AGENTS.md](#agentsmd) section at the bottom of this page:

```bash
mkdir -p ~/.config/agent-guardrails
# Then paste the AGENTS.md content into ~/.config/agent-guardrails/AGENTS.md
```

### Subscriptions and Authentication

Each agent authenticates separately after the container is running:

- **Claude Code**: Run `claude` in the container terminal. It will prompt you to authenticate via browser using your Anthropic account (Claude Pro, Team, or Enterprise plan required).
- **Codex**: Run `codex` in the container terminal. It will prompt you to authenticate via browser using your OpenAI account (ChatGPT Plus, Team, or Enterprise plan required).
- **Custom LLM**: No interactive login required. Set `OPENAI_API_KEY` in your host shell environment (the devcontainer picks it up via `${localEnv:OPENAI_API_KEY}`) and configure `OPENAI_API_BASE` in `devcontainer.json` before building (see the Custom LLM file reference below for details).

Authentication tokens for Claude Code and Codex are persisted in named Docker volumes so you only need to log in once per container identity.

---

## Repository Layout

Place the devcontainer files in your project's `.devcontainer` directory. Each variant lives in its own subdirectory. Create the files using the contents provided in the [File Contents](#file-contents) section below.

```
your-project/
├── .devcontainer/
│   ├── claude-code/
│   │   ├── devcontainer.json
│   │   ├── Dockerfile
│   │   ├── .zshrc
│   │   ├── starship.toml
│   │   └── post_install.py
│   ├── openai-codex/
│   │   ├── devcontainer.json
│   │   ├── Dockerfile
│   │   ├── .zshrc
│   │   ├── starship.toml
│   │   └── post_install.py
│   └── customllm-aider/
│       ├── devcontainer.json
│       ├── Dockerfile
│       ├── .zshrc
│       ├── starship.toml
│       └── post_install.py
└── ... (your project files)
```

---

## Running in VS Code

### Opening a Devcontainer

1. Open your project folder in VS Code.
2. Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on macOS) and select **Dev Containers: Reopen in Container**.
3. If multiple devcontainer configurations exist, VS Code will prompt you to choose. Select **Claude Code Sandbox**, **Codex Sandbox**, or **Custom LLM Sandbox**.
4. VS Code will build the Docker image (first run takes a few minutes) and reopen your workspace inside the container.

### First-Time Setup After Container Starts

Once the container is running and you have a terminal in VS Code:

**Claude Code:**

```bash
# Authenticate with your Anthropic account
claude

# Claude will print a URL — open it in your browser to complete OAuth
```

**Codex:**

```bash
# Authenticate with your OpenAI account
codex

# Codex will print a URL — open it in your browser to complete OAuth
```

**Custom LLM:**

```bash
# Custom LLM reads OPENAI_API_BASE, OPENAI_API_KEY, and AIDER_MODEL from the environment
# (set in devcontainer.json containerEnv — update these before building)
# Just run aider to start:
aider

# To override the model for a single session:
aider --model openai/your-model-name
```

### Rebuilding the Container

If the Dockerfile or devcontainer.json changes, rebuild:

- `Ctrl+Shift+P` → **Dev Containers: Rebuild Container**

Or to rebuild without cache:

- `Ctrl+Shift+P` → **Dev Containers: Rebuild Container Without Cache**

---

## Running from the Terminal (Without VS Code)

You can use the Dev Container CLI to build and run containers headlessly — useful for CI pipelines, SSH-based workflows, or if you prefer a different editor.

### Install the CLI

```bash
npm install -g @devcontainers/cli
```

### Build and Start

From your project root (the directory containing `.devcontainer/`):

**Claude Code:**

```bash
# Build and start the container, then drop into a shell
devcontainer up --workspace-folder . --config .devcontainer/claude-code/devcontainer.json
devcontainer exec --workspace-folder . --config .devcontainer/claude-code/devcontainer.json zsh
```

**Codex:**

```bash
devcontainer up --workspace-folder . --config .devcontainer/openai-codex/devcontainer.json
devcontainer exec --workspace-folder . --config .devcontainer/openai-codex/devcontainer.json zsh
```

**Custom LLM:**

```bash
devcontainer up --workspace-folder . --config .devcontainer/customllm-aider/devcontainer.json
devcontainer exec --workspace-folder . --config .devcontainer/customllm-aider/devcontainer.json zsh
```

### Running Agent Commands Directly

Once the container is up, you can exec directly into the agent:

```bash
# Run Claude Code on a task
devcontainer exec --workspace-folder . \
  --config .devcontainer/claude-code/devcontainer.json \
  claude "explain the main function in src/app.py"

# Run Codex on a task
devcontainer exec --workspace-folder . \
  --config .devcontainer/openai-codex/devcontainer.json \
  codex "explain the main function in src/app.py"

# Run Custom LLM on a task
devcontainer exec --workspace-folder . \
  --config .devcontainer/customllm-aider/devcontainer.json \
  aider --message "explain the main function in src/app.py"
```

### Stopping the Container

Containers are named `<variant>-<project-folder>` for easy identification:

| Variant | Container Name Pattern |
|---------|----------------------|
| Claude Code | `claude-<project-folder>` |
| Codex | `codex-<project-folder>` |
| Custom LLM | `cllm-<project-folder>` |

```bash
# List running devcontainers
docker ps --filter "label=devcontainer.local_folder"

# Stop by name
docker stop claude-my-api
docker stop codex-my-api
docker stop cllm-my-api
```

---

## Architecture Overview

### Container Base

All three containers are built on `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` and include:

| Component | Version | Purpose |
|-----------|---------|---------|
| Python | 3.13 via `uv` (3.12 for Custom LLM) | Runtime for scripts and projects |
| Node.js | 22 (via `fnm`) | Runtime for agent CLIs and JS projects |
| uv | 0.10.0 | Fast Python package management |
| git-delta | 0.18.2 | Improved git diffs with syntax highlighting |
| fzf | 0.67.0 | Fuzzy finder for files and history |
| Starship | latest | Cross-shell prompt with git/language context |
| Oh My Zsh | 1.2.1 | Zsh framework with git plugin |
| ripgrep, fd, jq, tmux | system | Modern CLI search and session tools |
| ast-grep | latest (via uv) | AST-based code search and linting |
| GitHub CLI | latest (via feature) | `gh` for pull requests, issues, etc. |

### Security Model

The containers use a defense-in-depth approach:

- **Non-root user**: All operations run as the `vscode` user (UID mapped to your host UID via `updateRemoteUserUID`).
- **Named containers**: Each container is named `<variant>-<project-folder>` via `--name` in `runArgs` (e.g., `claude-my-api`, `codex-my-api`, `cllm-my-api`), making them easy to identify in `docker ps` and target with `docker exec`/`docker stop`. Note that Docker requires unique container names — you cannot run the same variant for two projects simultaneously without changing the name.
- **`init: true`**: A proper init process (tini) reaps zombie processes.
- **Read-only host mounts**: Your `~/.gitconfig` and `AGENTS.md` are mounted read-only — the agent cannot modify your host git config or guardrails.
- **Named volumes for state**: Agent configs, shell history, and GitHub CLI tokens persist across container rebuilds in isolated Docker volumes.
- **Workspace bind mount**: Your project directory is mounted read-write at `/workspace` with `consistency=delegated` for macOS performance.

The **Codex** variant adds additional hardening via `runArgs`:

| Flag | Purpose |
|------|---------|
| `--security-opt=no-new-privileges` | Prevents privilege escalation inside the container |
| `--pids-limit=512` | Caps the number of processes to prevent fork bombs |
| `--memory=4g` | Limits container memory to 4 GB |

All three variants grant `NET_ADMIN` and `NET_RAW` capabilities for network tooling (iptables, DNS testing).

### Agent Sandbox Modes

Since Docker itself is the sandbox, the agents run with relaxed internal permissions:

- **Claude Code**: `bypassPermissions` mode — Claude Code skips its own permission prompts since the container provides isolation.
- **Codex**: `sandbox_mode = "danger-full-access"` with `approval_policy = "on-request"` — Codex disables its internal sandbox and uses an on-request approval flow.
- **Custom LLM**: No internal sandbox to configure — Aider operates directly on the filesystem by design, making Docker the sole isolation boundary.

### Post-Install Configuration

The `post_install.py` script runs automatically on container creation (`postCreateCommand`) and handles:

1. **Agent configuration** (Claude Code and Codex only) — writes the sandbox mode settings described above. Custom LLM requires no agent-specific configuration.
2. **Tmux setup** — 200k line scrollback, mouse support, vi keybindings, true color.
3. **Directory ownership** — fixes mounted volume permissions if they were created as root.
4. **Git configuration** — creates a local `.gitconfig.local` that includes your host config and adds `delta` as the pager plus a global `.gitignore` for common artifacts.

### Environment Variables

Key environment variables set in `containerEnv`:

| Variable | Value | Purpose |
|----------|-------|---------|
| `GIT_CONFIG_GLOBAL` | `~/.gitconfig.local` | Points git to the container-local config (which includes your host config) |
| `STARSHIP_CONFIG` | `~/.config/starship.toml` | Starship prompt config path |
| `UV_LINK_MODE` | `copy` | Avoids hardlink issues across filesystem boundaries |
| `NPM_CONFIG_IGNORE_SCRIPTS` | `true` | Prevents npm lifecycle scripts from running (security) |
| `NPM_CONFIG_AUDIT` | `true` | Enables npm audit on install |
| `NPM_CONFIG_MINIMUM_RELEASE_AGE` | `1440` | Only installs npm packages published ≥24 hours ago |
| `NODE_OPTIONS` | `--max-old-space-size=4096` | 4 GB heap for Node.js processes |
| `OPENAI_API_BASE` | *(Custom LLM only)* | URL of your OpenAI-compatible LLM endpoint |
| `OPENAI_API_KEY` | *(Custom LLM only)* | API key for the LLM endpoint (sourced from host via `${localEnv:OPENAI_API_KEY}`) |
| `AIDER_MODEL` | *(Custom LLM only)* | Default model name (e.g., `openai/your-model-name`) |

---

## Persistent State and Volumes

Named Docker volumes keep the following data across container rebuilds:

| Volume | Mount Point | Contents |
|--------|-------------|----------|
| `*-bashhistory-*` | `/commandhistory` | Zsh and bash history (200k lines) |
| `*-config-*` | `~/.claude`, `~/.codex`, or `~/.aider` | Agent auth tokens and settings |
| `*-gh-*` | `~/.config/gh` | GitHub CLI authentication |

To completely reset a container's state, delete its volumes:

```bash
# List volumes for a specific variant
docker volume ls | grep claude-code
docker volume ls | grep codex
docker volume ls | grep aider

# Remove specific volumes (container must be stopped)
docker volume rm <volume_name>
```

---

## Troubleshooting

### Container fails to start with mount errors

Ensure both required host files exist:

```bash
ls -la ~/.gitconfig
ls -la ~/.config/agent-guardrails/AGENTS.md
```

### Agent CLI not found after container start

The post-install script runs via `uv run --no-project`. If it fails, check the creation log:

- In VS Code: check the **Dev Containers** output panel during build.
- From the terminal: `devcontainer up` prints build logs to stdout.

You can also re-run manually inside the container:

```bash
uv run --no-project /opt/post_install.py
```

### Permission denied on `/commandhistory` or config directories

The post-install script attempts to fix ownership automatically. If it fails, run manually:

```bash
sudo chown -R $(id -u):$(id -g) /commandhistory ~/.claude ~/.codex ~/.aider ~/.config/gh
```

### Slow file access on macOS

The workspace mount uses `consistency=delegated` which should help. If performance is still poor, consider using a named volume for the workspace instead of a bind mount, or use Docker's VirtioFS file sharing backend (Docker Desktop → Settings → General → File sharing implementation).

### Rebuilding after Dockerfile changes

If you've modified the Dockerfile or any copied files (`.zshrc`, `starship.toml`, `post_install.py`), you need to rebuild:

```bash
# VS Code
Ctrl+Shift+P → "Dev Containers: Rebuild Container Without Cache"

# Terminal
devcontainer up --workspace-folder . \
  --config .devcontainer/claude-code/devcontainer.json \
  --remove-existing-container
```

---

## Customization

### Adding project-specific dependencies

Add additional `RUN` commands to the Dockerfile. For Python packages, prefer `uv`:

```dockerfile
RUN uv tool install ruff
```

For Node packages:

```dockerfile
RUN export PATH="$FNM_DIR:$PATH" && eval "$(fnm env)" && npm install -g typescript
```

### Changing the default editor

The default editor is `nano`. To switch to `vim`, update the `ENV` lines in the Dockerfile:

```dockerfile
ENV EDITOR=vim
ENV VISUAL=vim
```

### Adding VS Code extensions

Add extension IDs to the `customizations.vscode.extensions` array in `devcontainer.json`:

```json
"extensions": [
  "anthropic.claude-code",
  "ms-python.python",
  "dbaeumer.vscode-eslint"
]
```

### Modifying agent guardrails

Edit `~/.config/agent-guardrails/AGENTS.md` on your host. Changes take effect immediately since it's a bind mount (no rebuild needed).

---

## File Contents

All files for all three variants are provided below. Create the directory structure shown in [Repository Layout](#repository-layout) and paste each file's contents.

### AGENTS.md

Save to `~/.config/agent-guardrails/AGENTS.md` on your host machine.

```markdown
# Agent Guardrails

## Read/Command Approval Guardrail

- Before reading any file or running any command, stop and ask for explicit user approval in the current turn.
- Do not perform read operations (for example: `cat`, `sed`, `less`, `rg`, `ls`) until the user replies with a clear approval.
- Do not run any command (including non-writing commands) until the user replies with a clear approval.
- Approval is single-use per action set: after one approved read/command batch, ask again before any further reads or commands.
- Never infer consent from prior messages, task intent, open tabs, or previous approvals.
- If approval is not explicit, take no filesystem or command action.

## Write Approval Guardrail

- Before creating, modifying, renaming, or deleting any file, stop and ask for explicit user approval in the current turn.
- Do not perform any write operation until the user replies with a clear approval (for example: "approved", "yes", or equivalent).
- This applies to all file-changing actions, including `apply_patch`, redirection (`>`), here-doc writes, formatters, generators, and commands that may update files indirectly.
- Reading files is allowed without approval unless restricted by other guardrails.
- Approval is single-use per action set: after one approved write batch, ask again before any additional file changes.
- Never infer consent from prior messages, task intent, or previous approvals.

## Path Scope Guardrail

- Only read, write, or run commands inside the exact folder explicitly named by the user for that request.
- If no folder is named, ask before doing any file or command work.
- Before accessing any other folder (including sibling folders), stop and ask for permission first.
- Do not run broad discovery commands (for example `rg --files`, `find`, or repo-wide searches) outside the named folder.

## Operational Rule

- Treat these guardrails as mandatory defaults for every session unless the user explicitly overrides them in the current request.
- Never access parent directories without explicit per-request approval.
```
