# Free Claude Code Setup Guide
> For non-persistent Linux systems (e.g. zsh shell)

---

## How It Works

This setup runs a **local proxy server on port 8082** that intercepts Claude Code's requests and reroutes them to a free LLM provider (NVIDIA NIM). You only need to set 2 environment variables — no modifications to Claude Code or the VS Code extension needed.

```
Claude Code (CLI / VSCode) ──► Free Claude Code Proxy (:8082) ──► NVIDIA NIM (free)
```

---

## One-Time Setup

### 1. Get a Free API Key
- Go to [build.nvidia.com](https://build.nvidia.com)
- Sign up and grab your free API key (starts with `nvapi-...`)

---

### 2. Install uv (Python package manager)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.zshrc
```

---

### 3. Install Node.js via nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
nvm install --lts

# Verify
node --version
npm --version
```

---

### 4. Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

---

### 5. Clone and Configure the Repo

```bash
git clone https://github.com/Alishahryar1/free-claude-code.git
cd free-claude-code
cp .env.example .env
```

Edit `.env` and add your NVIDIA NIM key:

```env
NVIDIA_NIM_API_KEY="nvapi-YOUR-KEY-HERE"
MODEL_OPUS="nvidia_nim/z-ai/glm4.7"
MODEL_SONNET="nvidia_nim/z-ai/glm4.7"
MODEL_HAIKU="nvidia_nim/z-ai/glm4.7"
MODEL="nvidia_nim/z-ai/glm4.7"
```

---

### 6. Set Env Vars Permanently

```bash
echo 'export ANTHROPIC_BASE_URL="http://localhost:8082"' >> ~/.zshrc
echo 'export ANTHROPIC_AUTH_TOKEN="freecc"' >> ~/.zshrc
source ~/.zshrc
```

---

### 7. Install the Claude Code VS Code Extension

```bash
code --install-extension anthropic.claude-code
```

Or manually: `Ctrl+Shift+X` → search **"Claude Code"** → Install.

---

### 8. Create Your Startup Script

```bash
nano ~/start-claude.sh
```

Paste this:

```bash
#!/bin/bash

# Load nvm + node
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"

# Start proxy in background
cd ~/free-claude-code
uv run uvicorn server:app --host 0.0.0.0 --port 8082 &
sleep 3

# Confirm it's running
curl -s http://localhost:8082 && echo ""
echo "✅ Claude Code proxy is ready!"
echo "👉 Now: cd into your project and run 'code .' or 'claude'"
```

Make it executable:

```bash
chmod +x ~/start-claude.sh
```

---

## Every Session (Non-Persistent Systems)

Since your system resets, run this at the start of every session:

```bash
bash ~/start-claude.sh
```

Then open any project:

```bash
cd ~/your-project   # or git clone your repo first
code .              # VS Code opens with Claude sidebar ready ✅
```

> ⚠️ Always keep the proxy terminal alive. If you close it, Claude Code stops working.

---

## Verify the Proxy is Running

```bash
curl http://localhost:8082
# Expected: {"status":"ok","provider":"nvidia_nim","model":"nvidia_nim/z-ai/glm4.7"}
```

---

## Using Claude Code in VS Code Sidebar

1. Start the proxy → `bash ~/start-claude.sh`
2. Open your project → `code .`
3. Click the **Claude icon** in the left activity bar
4. Start chatting about your project

---

## Using Claude Code in the Terminal

```bash
cd ~/your-project
claude
```

### Useful Commands Inside Claude

| Command | What it does |
|---|---|
| `/help` | See all available commands |
| `/clear` | Clear conversation, start fresh |
| `/compact` | Summarize history to save context |
| `/quit` | Exit Claude Code |

---

## Example Prompts for Building Projects

**Start from scratch:**
```
> Build a Node.js Express REST API with a users route and MongoDB connection
> Create a Python FastAPI project with user authentication
> Set up a Next.js app with Tailwind CSS and a homepage
```

**Work on existing code:**
```
> Read my app.js and add error handling to all routes
> Find and fix the bug in utils/helper.py
> Add input validation to my signup form
```

**Boilerplate tasks:**
```
> Write a README for this project
> Add .gitignore and set up git
> Write tests for all my API endpoints
> Dockerize this app
```

---

## Cloning a Repo and Using Claude Code

```bash
# 1. Start proxy (if not already running)
bash ~/start-claude.sh

# 2. Clone your project
git clone https://github.com/your-username/your-repo.git
cd your-repo

# 3. Open in VS Code (Claude sidebar works automatically)
code .
```

> The env vars in `~/.zshrc` mean every new VS Code window inherits the proxy settings automatically — as long as the proxy is running.

---

## Quick Reference Card

```
SESSION START
─────────────────────────────────────
bash ~/start-claude.sh        # start proxy

OPEN A PROJECT
─────────────────────────────────────
cd ~/my-project && code .     # open in VS Code
cd ~/my-project && claude     # open in terminal

CHECK PROXY
─────────────────────────────────────
curl http://localhost:8082    # should return {"status":"ok",...}
```
