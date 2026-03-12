# Vibe Coding Guide — Converting Agents to Talent Market Format

> **For AI coders (Claude Code, Cursor, Copilot, etc.):**
> This guide tells you exactly how to convert an existing AI agent project
> into the Talent Market template format so it can be published on
> [carbonkites.com](https://carbonkites.com).

## Target Format

Every talent is a directory with this structure:

```
my-talent/
├── profile.yaml              # Required — identity & config
├── DESCRIPTION.md            # Optional — rich description, demos, case studies
├── avatar.jpg                # Optional — talent avatar (png/jpg/svg/webp)
├── skills/                   # Required — one folder per skill
│   └── skill-name/
│       └── SKILL.md          # Frontmatter + instructions
├── tools/                    # Optional — one folder per tool
│   ├── .mcp.json             # MCP server definitions (standard format)
│   ├── code-search/
│   │   ├── TOOL.md           # Tool description & usage docs
│   │   └── manifest.yaml     # Tool metadata (name, type, params)
│   └── run-tests/
│       ├── TOOL.md
│       ├── manifest.yaml
│       └── run.sh            # Tool implementation (if custom)
├── launch.sh                 # Optional — self-hosted startup
├── heartbeat.sh              # Optional — health check
└── manifest.json             # Optional — settings UI schema
```

---

## Step-by-Step: Convert Any Agent

### 1. Create `profile.yaml`

This is the only required file. Fill in every field:

```yaml
id: my-agent                    # Unique ID (lowercase, hyphens ok)
name: My Agent                  # Display name
avatar: avatar.jpg              # Optional — talent avatar image
description: >
  What this agent does, its strengths, and typical use cases.
  Be specific — this is shown on the marketplace card.
role: Engineer                  # Engineer | Designer | Manager | Researcher | Analyst | Assistant
hosting: company                # company | self | remote
auth_method: api_key            # api_key | cli | oauth
api_provider: openrouter        # openrouter | anthropic | custom
llm_model: ""                   # e.g. "claude-sonnet-4-20250514", empty = platform default
temperature: 0.7
hiring_fee: 0.0                 # USD, 0 = free
salary_per_1m_tokens: 0.0
agent_family: ""                # claude | openclaw | omctalent | "" (custom)
skills:
  - skill-name-1
  - skill-name-2
tools: []
personality_tags:
  - autonomous
  - thorough
system_prompt_template: >
  You are [Agent Name], a [role] that specializes in [domain].
  [Core instructions, constraints, and behavioral guidelines.]
```

### 2. Convert Skills to `skills/<name>/SKILL.md`

Each skill is a **folder** with a `SKILL.md` file inside. The file must have YAML frontmatter:

```markdown
---
name: Skill Display Name
description: One-line summary — used to decide when to activate this skill.
---

# Skill Name

Detailed instructions for the agent when this skill is active.

## Guidelines
- What to do
- What not to do
- Quality standards

## Examples
- Example input → expected output
```

### 3. Create Tool Folders under `tools/`

Each tool is a **folder** containing its docs, metadata, and optionally implementation code.

**MCP tools** — place `.mcp.json` under `tools/` (standard format):

```
tools/
├── .mcp.json               # MCP server definitions
├── filesystem/
│   └── TOOL.md             # What this tool does, when to use it
└── github/
    └── TOOL.md
```

`tools/.mcp.json`:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-filesystem"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": ""
      }
    }
  }
}
```

**Custom tools** — each gets a folder with `TOOL.md`, `manifest.yaml`, and implementation:

```
tools/
└── run-tests/
    ├── TOOL.md              # Usage docs
    ├── manifest.yaml        # Metadata
    └── run.sh               # Implementation
```

`tools/run-tests/TOOL.md`:
```markdown
---
name: run-tests
description: Execute the project test suite and report results.
---

# Run Tests

Runs the full test suite. Use after code changes to verify correctness.

## Usage
Invoke this tool to run `npm test` and return pass/fail results.
```

`tools/run-tests/manifest.yaml`:
```yaml
name: run-tests
type: shell
command: bash run.sh
parameters:
  - name: filter
    type: string
    description: Test name filter pattern
    required: false
```

---

## Converting from Claude Code Agent

A Claude Code agent typically has:
- `CLAUDE.md` — project instructions and constraints
- `.mcp.json` — MCP server configurations
- No `profile.yaml`

### Conversion steps:

**1. Extract system prompt from `CLAUDE.md`**

Read `CLAUDE.md` and copy its content into `profile.yaml` → `system_prompt_template`. This is the agent's core personality and behavior definition.

```bash
# Read the source
cat /path/to/claude-agent/CLAUDE.md
```

**2. Extract skills from the codebase**

Look for distinct capabilities described in `CLAUDE.md` or in any `skills/` directory. Each logical capability becomes a skill folder:

```bash
mkdir -p my-talent/skills/code-review
# Write SKILL.md with the relevant instructions from CLAUDE.md
```

If `CLAUDE.md` has sections like "## Code Review", "## Debugging", "## Refactoring" — each becomes a separate skill.

**3. Copy `.mcp.json` into `tools/`**

```bash
mkdir -p my-talent/tools
cp /path/to/claude-agent/.mcp.json my-talent/tools/.mcp.json
```

No conversion needed — `.mcp.json` keeps the standard format. Optionally create a `TOOL.md` for each MCP server to document its usage:

```bash
mkdir -p my-talent/tools/filesystem
cat > my-talent/tools/filesystem/TOOL.md << 'EOF'
---
name: filesystem
description: Read and search files in the project directory.
---

# Filesystem Tool

MCP server for file system access. Provides read, search, and directory listing.
EOF
```

**4. Set agent_family and auth_method**

```yaml
agent_family: claude
auth_method: cli          # Claude Code uses CLI-based auth
hosting: company           # or "self" if it runs independently
```

**5. Example: full conversion**

Source `CLAUDE.md`:
```markdown
# Code Review Bot
You are a thorough code reviewer. You check for bugs, style issues,
security vulnerabilities, and performance problems.
Always explain your reasoning and suggest fixes.
```

Result `profile.yaml`:
```yaml
id: code-review-bot
name: Code Review Bot
description: >
  Thorough code reviewer that checks for bugs, style issues,
  security vulnerabilities, and performance problems.
role: Engineer
hosting: company
auth_method: cli
api_provider: anthropic
agent_family: claude
skills:
  - code-review
  - security-audit
personality_tags:
  - thorough
  - security-focused
system_prompt_template: >
  You are a thorough code reviewer. You check for bugs, style issues,
  security vulnerabilities, and performance problems.
  Always explain your reasoning and suggest fixes.
```

Result `skills/code-review/SKILL.md`:
```markdown
---
name: Code Review
description: Review code for bugs, style issues, and best practices violations.
---

# Code Review

Review pull requests and code changes for correctness, style, performance,
and security issues. Always explain the reasoning behind each finding.

## Checklist
- Logic errors and edge cases
- Naming conventions and code style
- Error handling completeness
- Performance implications
- Security vulnerabilities (injection, XSS, etc.)
```

---

## Converting from OpenClaw Agent

An OpenClaw agent typically has:
- A graph-based workflow definition (YAML/JSON)
- Channel configurations (WhatsApp, Telegram, Slack, etc.)
- `launch.sh` or Docker setup for self-hosting

### Conversion steps:

**1. Map the workflow graph to skills**

Each node/stage in the OpenClaw graph becomes a skill. For example, a "message-router" node becomes `skills/message-routing/SKILL.md`.

**2. Set hosting and agent_family**

```yaml
agent_family: openclaw
hosting: self              # OpenClaw agents are typically self-hosted
auth_method: api_key
```

**3. Preserve launch.sh**

Copy `launch.sh` directly — the platform uses it to start self-hosted agents.

**4. Extract channel configs into skills**

If the agent supports multiple channels, create a skill describing channel-specific behavior:

```
skills/
├── multi-channel-comms/
│   └── SKILL.md           # Channel list, routing rules, commands
└── voice-interaction/
    └── SKILL.md           # TTS, wake words, voice-specific behavior
```

---

## Converting from a Generic Python/Node Agent

For custom agents (LangChain, AutoGen, CrewAI, etc.):

**1. Identify the system prompt**

Look in the source code for:
- `system_message`, `system_prompt`, or `instructions` variables
- Agent class constructor arguments
- Config files (`.env`, `config.yaml`, `agents.yaml`)

Copy it to `system_prompt_template`.

**2. Identify skills/capabilities**

Look for:
- Tool definitions → `tools/manifest.yaml`
- Distinct task handlers → each becomes a skill folder
- Agent roles in multi-agent setups → each role is a separate talent

**3. Set hosting appropriately**

| Your setup | `hosting` value |
|-----------|----------------|
| Runs on our platform (stateless, API-based) | `company` |
| Runs on user's machine (`launch.sh`) | `self` |
| Runs externally, connects via HTTP webhook | `remote` |

---

## Validation Checklist

Before publishing, verify:

- [ ] `profile.yaml` has `id`, `name`, `description`, `role`, `system_prompt_template`
- [ ] `avatar.jpg` (or `.png`/`.jpg`/`.webp`) exists — shown on talent cards in the marketplace
- [ ] Each skill in `profile.yaml` → `skills` has a matching `skills/<name>/SKILL.md`
- [ ] Each `SKILL.md` has `---` frontmatter with `name` and `description`
- [ ] `description` in profile is specific (not "An AI agent that helps with tasks")
- [ ] `system_prompt_template` contains the full behavioral instructions
- [ ] `hiring_fee` is set (0 for free)
- [ ] If self-hosted: `launch.sh` exists and is executable
- [ ] No secrets/API keys are hardcoded anywhere
- [ ] `DESCRIPTION.md` exists with agent overview, use cases, and optionally demos/success stories
- [ ] No illegal content, political propaganda, or harmful material in any file

## Publishing

```bash
# Push to GitHub
git init && git add -A && git commit -m "Initial talent"
gh repo create my-talent --public --push

# Register on Talent Market
# Go to https://carbonkites.com → Add Talent → paste your repo URL
```

Or via API:
```bash
curl -X POST https://carbonkites.com/api/v1/repos \
  -H "X-API-Key: tm_live_..." \
  -H "Content-Type: application/json" \
  -d '{"url": "https://github.com/you/my-talent"}'
```
