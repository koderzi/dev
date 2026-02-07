# dev

Universal GPU-accelerated dev container for all projects. Clone project repos inside the container and use [mise](https://mise.jdx.dev/) for per-project runtime isolation.

## What's Included

| Tool | Purpose |
|------|---------|
| **mise** | Polyglot runtime manager (Python, Node.js, Ruby, Go, Java, etc.) |
| **Node.js 22** | Pre-installed globally via mise |
| **Google Chrome** | Browser for web dev and automation |
| **Google Antigravity** | Installed via official APT repo, with auto-updates |
| **Desktop GUI** | Virtual desktop streamed via noVNC (auto-opens in browser) |
| **Gemini CLI** | Google AI coding assistant (`gemini`) |
| **Claude Code** | Anthropic AI coding assistant (`claude`) |
| **chrome-devtools-mcp** | Chrome DevTools MCP server (globally installed, pre-configured in Gemini CLI with `headless=false`) |

## Quick Start

1. In VS Code, open Command Palette → **"Dev Containers: Clone Repository in Named Volume"**.

2. Enter `https://github.com/koderzi/dev.git` and give it a name.

3. **Clone your projects** into the workspace:
   ```bash
   git clone https://github.com/koderzi/some-project.git
   cd some-project
   ```

4. **Set up the project's runtime** using mise:
   ```bash
   # Python project
   mise use python@3.13
   mise config set env._.python.venv.path .venv
   mise config set env._.python.venv.create true

   # Node.js project
   mise use node@22

   # Multiple runtimes
   mise use python@3.13 node@22
   ```

5. **Install project dependencies** as usual:
   ```bash
   pip install -r requirements.txt   # Python
   npm install                        # Node.js
   composer install                   # PHP
   ```

## GPU Support

This template is configured with NVIDIA GPU passthrough by default (`--gpus all --runtime=nvidia`).

**To run without GPU**, edit `.devcontainer/devcontainer.json` and remove the GPU-related `runArgs`:
```jsonc
"runArgs": [
    "--security-opt",
    "seccomp=unconfined"
],
```

## Desktop GUI

A virtual desktop is available via noVNC on port 6080. It auto-opens in your browser when the container starts.

## AI Tools

All AI coding assistants are pre-installed globally:

```bash
# Gemini CLI (pre-configured with chrome-devtools-mcp, headless=false)
gemini

# Claude Code
claude

# Chrome DevTools MCP (starts automatically with Gemini)
chrome-devtools-mcp
```

## Project Isolation with mise

Each project folder gets its own `mise.toml` with isolated runtimes and virtual environments:

```
(workspace root)/
├── project-a/          # Python 3.13 + .venv
│   └── mise.toml
├── project-b/          # Node.js 22
│   └── mise.toml
└── project-c/          # Python 3.11 + Node.js 20
    └── mise.toml
```

When you `cd` into a project directory, mise automatically activates the correct runtimes.
