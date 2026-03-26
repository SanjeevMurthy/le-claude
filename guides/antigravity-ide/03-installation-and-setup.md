# 03 — Installation and Setup

## System Requirements

| Requirement | Minimum | Recommended |
|---|---|---|
| **OS** | macOS 12+, Windows 10+, Ubuntu 20.04+ | macOS 14+, Windows 11, Ubuntu 22.04+ |
| **RAM** | 8 GB | 16 GB+ |
| **Storage** | 2 GB free | 10 GB+ (for project indexing) |
| **Network** | Broadband internet | Low-latency connection |
| **Account** | Google account (personal Gmail) | Google Workspace account |

## Installation Steps

### Step 1: Download Antigravity

Download the installer from [antigravity.google](https://antigravity.google):

```bash
# macOS (Apple Silicon)
# Download .dmg from official site

# macOS (Intel)
# Download .dmg (x64) from official site

# Linux
# Download .deb or .tar.gz from official site

# Windows
# Download .exe installer from official site
```

### Step 2: Initial Configuration

On first launch, Antigravity walks through a setup wizard:

1. **Link Google Account** — Sign in with your Gmail account (required for API access)
2. **Import VS Code Settings** — Optionally import existing keybindings, themes, and extensions
3. **Select Theme** — Choose from built-in themes or keep your VS Code theme
4. **Select Default Model** — Choose your primary AI model:
   - Gemini 3.1 Pro (recommended for complex tasks)
   - Gemini 3 Flash (recommended for speed)
   - Claude Sonnet 4.6 / Opus 4.6
   - GPT-OSS-120B

### Step 3: Configure Agent Policies

Three critical security settings need immediate attention:

#### Review Policy

Controls how much autonomy agents have over code changes:

| Setting | Behavior | Use When |
|---|---|---|
| **Always proceed** | Agent auto-approves all actions | Trusted environments, personal projects |
| **Agent decides** | Agent asks for approval when uncertain | **Recommended default** |
| **Request review** | Manual approval required for every action | Production codebases, sensitive code |

#### Terminal Execution Policy

Controls whether agents can run terminal commands:

| Setting | Behavior | Risk Level |
|---|---|---|
| **Off** | Never auto-execute commands | Safest |
| **Auto** | Agent decides when to auto-execute | Moderate |
| **Turbo** | Always auto-execute (configurable allow/deny lists) | Highest autonomy |

> **Best Practice:** Start with "Auto" and configure allow/deny lists:
> ```
> # Allow list (safe commands)
> npm run test
> npm run lint
> git status
> git diff
> 
> # Deny list (dangerous commands)
> rm -rf
> git push --force
> npm publish
> docker rm
> ```

#### Agent Mode

| Mode | Behavior |
|---|---|
| **Planning** | Agent creates implementation plan before coding (recommended for complex tasks) |
| **Fast** | Agent executes immediately without planning (for simple tasks) |

### Step 4: Configure Workspace

```
my-project/
├── .agents/               # Workspace-specific agent configuration
│   ├── rules/             # Rules that apply to this workspace
│   │   └── coding-style.md
│   ├── skills/            # Skills available in this workspace
│   │   └── my-skill/
│   │       └── SKILL.md
│   └── workflows/         # Workflows available in this workspace
│       └── deploy.md
├── .vscode/               # VS Code settings (inherited by Antigravity)
│   └── settings.json
├── src/
└── package.json
```

### Step 5: Install Browser Extension (Optional)

For full browser automation capabilities, install the Antigravity Browser Extension:

1. Open Chrome/Chromium
2. Navigate to the extension install page (provided in-app)
3. Enable the extension
4. The agent can now control the browser for UI testing and verification

## Migrating from VS Code

Antigravity is fully backward-compatible with VS Code:

- **Extensions** — Available through Open VSX Registry (not the VS Code Marketplace directly)
- **Settings** — `settings.json`, `keybindings.json` are fully supported
- **Themes** — All VS Code themes work
- **Language Support** — All language servers and formatters work

> **Note:** Some proprietary VS Code Marketplace extensions may not be available on Open VSX. Community alternatives typically exist.

## Global Configuration Structure

```
~/.gemini/
├── GEMINI.md                          # Global rules (always active)
├── antigravity/
│   ├── settings.json                  # IDE settings
│   ├── skills/                        # Global skills (available in all projects)
│   │   └── my-global-skill/
│   │       └── SKILL.md
│   ├── global_workflows/              # Global workflows
│   │   └── my-workflow.md
│   ├── brain/                         # Persistent knowledge & memory
│   │   └── <conversation-id>/
│   │       ├── task.md
│   │       ├── implementation_plan.md
│   │       └── walkthrough.md
│   └── knowledge/                     # Knowledge Items (KIs)
│       └── <ki-id>/
│           ├── metadata.json
│           └── artifacts/
└── mcp_config.json                    # MCP server configuration
```

## Verifying Installation

After setup, verify everything is working:

1. Open a project workspace
2. Open the Agent Sidebar (Ctrl/Cmd + Shift + A)
3. Type: "Describe the structure of this project"
4. The agent should:
   - Index the project
   - Produce a clear architectural summary
   - Reference specific files and directories

If the agent responds with a detailed project overview, your installation is complete.

---

*Previous: [02 — Antigravity Architecture](./02-antigravity-architecture.md) | Next: [04 — Core Configuration](./04-core-configuration.md)*
