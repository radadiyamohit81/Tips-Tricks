To make this permanent, copy the content below into a file named setup_claude.md or just copy-paste the code block directly into your terminal's configuration file (like ~/.zshrc or ~/.bashrc).
## Claude Code + Copilot Proxy Setup Guide

# Claude Code with GitHub Copilot Proxy Setup
Follow these steps to route Claude Code through your GitHub Copilot subscription.
## 1. Install & Start ProxyEnsure you have the proxy installed and running.
```bash
# Install (if not already done)
npm install -g copilot-api

# Start and follow browser login prompts
copilot-api start

Your proxy should now be listening at http://localhost:4141.
## 2. Configure Environment Variables
Add these to your shell profile (~/.zshrc for Mac/Zsh or ~/.bashrc for Linux/Bash) to avoid re-typing them:

# Core Proxy Settings
export ANTHROPIC_BASE_URL="http://localhost:4141"
export ANTHROPIC_API_KEY="sk-ant-copilot-proxy"
export ANTHROPIC_MODEL="claude-3-5-sonnet"
# Bypass Login Screens
export ANTHROPIC_AUTH_TOKEN="dummy-bypass"

## 3. Apply Changes
Run this to refresh your current terminal session:

source ~/.zshrc  # or ~/.bashrc

## 4. Run Claude Code
Now simply run the command from your project folder:

claude

## 5. Check Usage

https://ericc-ch.github.io/copilot-api/?endpoint=http://localhost:4141/usage

## Troubleshooting

* Error 404: Ensure ANTHROPIC_BASE_URL does NOT end with /v1.
* Error 403: Unset any Google/Vertex variables: unset CLAUDE_CODE_USE_VERTEX.
* Verify Proxy: Run curl http://localhost:4141/v1/models to check connectivity.


Would you like the instructions on how to **automate starting the proxy** whenever you open your computer?


