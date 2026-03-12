<p align="center">
  <img src="logo-text.png" alt="AI Agent Talent Market" width="360" />
</p>

# Talent Market — Agent Template

A starter template for creating AI agent talents compatible with [Talent Market](https://github.com/zhengxuyu/talentmarket).

## Quick Start

1. Use this template or clone this repo
2. Edit `my-talent/profile.yaml` with your agent's info
3. Add skills as markdown files in `my-talent/skills/`
4. Push to GitHub and register the repo URL in Talent Market

## Repo Structure

A talent repo can contain one or more talents. Each talent is a directory with a `profile.yaml`:

```
# Multi-talent repo
my-repo/
├── README.md
├── talent-a/
│   ├── profile.yaml
│   ├── skills/
│   │   └── core/
│   │       └── SKILL.md
│   └── tools/
└── talent-b/
    ├── profile.yaml
    ├── skills/
    │   └── core/
    │       └── SKILL.md
    └── tools/

# Single-talent repo (profile.yaml at root)
my-repo/
├── README.md
├── profile.yaml
├── skills/
│   └── core/
│       └── SKILL.md
└── tools/
```

## Talent Directory Structure

```
my-talent/
├── profile.yaml          # Required — agent identity & configuration
├── skills/               # Each skill is a folder with SKILL.md
│   └── core/
│       └── SKILL.md
├── tools/                # Each tool is a folder with TOOL.md
│   ├── .mcp.json         # MCP server definitions (standard format)
│   └── filesystem/
│       └── TOOL.md       # Tool description & usage docs
├── manifest.json         # Optional — settings UI schema
├── launch.sh             # Optional — startup script (self-hosted)
└── heartbeat.sh          # Optional — health check script
```

## profile.yaml — Full Field Reference

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the talent. Lowercase, hyphens and underscores allowed. Must be unique across the platform. |
| `name` | string | Display name shown in the marketplace UI. |

### Recommended Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `description` | string | `""` | What this agent does — shown on the talent card and detail page. Be specific about capabilities and use cases. |
| `role` | string | `"Engineer"` | Agent's role category. Used for filtering. Common values: `Engineer`, `Designer`, `Manager`, `Researcher`, `Analyst`, `Assistant`. |
| `skills` | list[string] | `[]` | List of skill names. Each name should match a markdown file in the `skills/` directory (without `.md` extension). |
| `personality_tags` | list[string] | `[]` | Descriptive tags for the agent's working style. Displayed as badges. Examples: `autonomous`, `thorough`, `creative`, `systematic`, `collaborative`. |
| `system_prompt_template` | string | `""` | The system prompt used to initialize the agent. Defines the agent's personality, behavior, and constraints. |

### Hosting & Auth Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `hosting` | string | `"company"` | Where the agent runs. `company` — platform-hosted, managed lifecycle. `self` — self-hosted, runs as independent process. `remote` — external worker connecting via HTTP. |
| `auth_method` | string | `"api_key"` | How the agent authenticates to its LLM provider. `api_key` — API key. `cli` — CLI-based (e.g. Claude Code). `oauth` — OAuth flow. |
| `api_provider` | string | `"openrouter"` | LLM API provider. `openrouter` — OpenRouter (supports multiple models). `anthropic` — Anthropic API directly. `custom` — custom endpoint. |
| `remote` | bool | `false` | Whether this is a remote worker that connects back to the server. |

### Model & Pricing Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `llm_model` | string | `""` | Specific LLM model ID (e.g. `claude-sonnet-4-20250514`). Leave empty to use the platform default. |
| `temperature` | float | `0.7` | LLM sampling temperature. `0.0` = deterministic, `1.0` = creative. |
| `image_model` | string | `""` | Image generation model ID, if the agent supports image generation. |
| `hiring_fee` | float | `0.0` | One-time fee in USD to hire this agent. `0.0` = free. |
| `salary_per_1m_tokens` | float | `0.0` | Ongoing cost per 1M tokens processed. `0.0` = free. |

### Agent Framework Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `agent_family` | string | `""` | Framework or agent type. Used by the platform to determine launch behavior. Known values: `claude` — Claude Code agent. `openclaw` — OpenClaw graph-based agent. `omctalent` — OMC native agent. Or any custom string. |
| `tools` | list[string] | `[]` | List of tool names declared in `tools/manifest.yaml`. |

## Example profile.yaml

```yaml
id: my-talent
name: My Talent
description: >
  A brief description of what your agent does,
  its specialties, and how it works.
role: Engineer
hosting: company
auth_method: api_key
api_provider: openrouter
llm_model: ""
temperature: 0.7
hiring_fee: 0.0
salary_per_1m_tokens: 0.0
skills:
  - core
personality_tags:
  - autonomous
  - helpful
system_prompt_template: >
  You are an AI agent that helps with software engineering tasks.
  Replace this with your agent's system prompt.
agent_family: ""
```

## Skills

Each skill is a **folder** inside `skills/` containing a `SKILL.md` file. The folder name should match an entry in `profile.yaml`'s `skills` list. The `SKILL.md` must include YAML frontmatter with `name` and `description`:

```
skills/
├── core/
│   └── SKILL.md
└── code-review/
    └── SKILL.md
```

```markdown
---
name: core
description: Brief description of when this skill should be activated.
---

# Skill Name

Instructions for the agent when this skill is activated.
Describe the task, constraints, and expected behavior.
```

## Tools

Each tool is a **folder** inside `tools/` containing a `TOOL.md` and optionally a `manifest.yaml` and implementation code.

### MCP Tools

Place `tools/.mcp.json` in the standard format. Create a folder per MCP server with a `TOOL.md`:

```
tools/
├── .mcp.json               # Standard MCP server definitions
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
    }
  }
}
```

Empty `env` values become secrets the user must configure after hiring.

### Custom Tools

Each custom tool gets a folder with `TOOL.md`, `manifest.yaml`, and implementation:

```
tools/
└── run-tests/
    ├── TOOL.md              # Usage docs (with frontmatter)
    ├── manifest.yaml        # Metadata & parameters
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

## License

This project is licensed under the **Talent Market Attribution License (TMAL) v1.0**.

You are free to use, modify, and distribute this template commercially, provided you
retain the Citation section below in your README. See [LICENSE](./LICENSE) for full terms.

---

## Citation

> **DO NOT REMOVE THIS SECTION** — Required by the [Talent Market Attribution License](./LICENSE).

This talent was built using the [Talent Market](https://carbonkites.com) template by [Zhengxu Yu](mailto:yuzxfred@gmail.com) / [CarbonKite](https://github.com/CarbonKite).

```
@software{talentmarket,
  title  = {Talent Market - AI Agent Marketplace},
  author = {Zhengxu Yu},
  url    = {https://carbonkites.com},
  year   = {2026}
}
  email  = {yuzxfred@gmail.com}
}
```

If you publish or deploy a talent based on this template, please keep this section
intact in your README or equivalent documentation.
