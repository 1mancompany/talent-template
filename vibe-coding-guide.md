# Vibe Coding Guide — Converting Agents to Talent Market Format

> **For AI coders (Claude Code, Cursor, Copilot, etc.):**
> This guide tells you exactly how to convert an existing AI agent project
> into the Talent Market template format so it can be published on
> [one-man-company.com](https://one-man-company.com).

## Target Structure

Every talent is a directory with this layout:

```
<talent-id>/
├── profile.yaml              ← REQUIRED (agent metadata)
├── DESCRIPTION.md             ← REQUIRED (full agent description, displayed on detail page)
├── avatar.png                 ← RECOMMENDED (talent avatar image, png/jpg/svg/webp)
├── skills/                    ← REQUIRED
│   └── core.md               ←   skill definition template
├── tools/                     ← REQUIRED if agent has tools
│   ├── .mcp.json              ←   MCP server definitions (standard format)
│   ├── manifest.yaml          ←   tool declarations (simple agents)
│   ├── <tool-name>/           ←   one folder per custom tool
│   │   ├── TOOL.md            ←     tool description & usage docs
│   │   ├── manifest.yaml      ←     tool metadata (name, type, params)
│   │   └── run.sh             ←     tool implementation (if custom)
├── launch.sh                  ← Optional — self-hosted startup script
├── heartbeat.sh               ← Optional — health check script
└── manifest.json              ← Optional — settings UI schema
```

## Output Checklist

**Minimum required files (ALL must exist):**

- [ ] `profile.yaml` — with all required fields including `avatar`
- [ ] `DESCRIPTION.md` — verbatim copy of source body
- [ ] `skills/core.md` — skill definition template
- [ ] `tools/manifest.yaml` — tool declarations (or `tools/` with proper structure if agent has tools)

**Verify before finishing:** All files exist? `profile.yaml` has `avatar`? `DESCRIPTION.md` is verbatim?

---

## Step 1: Determine the `talent-id`

- Use the source filename (without `.md` extension) as the talent ID
- Must be lowercase, hyphens allowed, no spaces
- Example: `engineering-senior-developer.md` → `engineering-senior-developer`

---

## Step 2: Create `DESCRIPTION.md` (DO THIS FIRST)

This is the public-facing description displayed on the talent detail page. It contains the agent's full personality, instructions, and methodology.

**Rule: VERBATIM COPY. Zero modifications.**

1. Take the source markdown file
2. Find the closing `---` of the YAML frontmatter
3. Copy EVERYTHING after it — every character, every emoji, every heading, every code block
4. Write it to `DESCRIPTION.md`

**DO NOT:**
- Remove or change emoji characters (e.g., keep `## 🧠 Your Identity` exactly as-is)
- Reformat, rewrite, summarize, or "clean up" any content
- Add or remove blank lines
- Change heading levels or wording

The content must be **byte-for-byte identical** to the source body. When in doubt, copy more rather than less.

---

## Step 3: Create `profile.yaml`

Extract metadata from the source frontmatter and fill in this template exactly:

```yaml
id: <talent-id>
name: <from source frontmatter "name" field>
avatar: avatar.png
description: >
  <from source frontmatter "description" field — keep original wording>
role: <specific job role — see Role Guidelines below>
hosting: self
auth_method: api_key
api_provider: anthropic
llm_model: ""
temperature: 0.7
hiring_fee: 0.0
salary_per_1m_tokens: 0.0
skills:
  - core
personality_tags:
  - <tag1>
  - <tag2>
system_prompt_template: >
  You are <name>. <first 1-2 sentences of description>
agent_family: ""
```

### Avatar Rules

The `avatar` field points to an **image file** in the talent directory. It is the agent's visual identity shown on talent cards and detail pages.

**Supported formats:** `.png`, `.jpg`, `.jpeg`, `.svg`, `.webp`

**How it works:**
1. Place an image file named `avatar.png` (or `.jpg`/`.svg`/`.webp`) in the talent root directory
2. Set `avatar: avatar.png` in `profile.yaml`
3. The backend auto-detects the file and converts it to a base64 data URI for rendering

**If no avatar image is available:**
- Omit the `avatar` field from `profile.yaml` (or set it to `""`)
- The frontend will automatically display a role-based fallback emoji
- This is perfectly acceptable — do NOT put emoji characters in the `avatar` field (it breaks the `<img>` tag)

**Important:** The `avatar` field must be a **filename** (e.g., `avatar.png`), NOT an emoji character. Emojis in this field will cause broken images on the frontend.

### Required Fields (must ALL be present and correct)

| Field | Value | Rule |
|-------|-------|------|
| `id` | `<talent-id>` | From source filename, lowercase with hyphens |
| `name` | `<display name>` | From source frontmatter `name`. If missing, derive from filename |
| `avatar` | `avatar.png` | Filename of avatar image in talent directory. Omit if no image available |
| `description` | `<text>` | From source frontmatter `description`. Keep original wording exactly |
| `role` | `<job title>` | A specific job title from the Role Table below |
| `hosting` | `self` | Always `self` — never `company` |
| `api_provider` | `anthropic` | Always `anthropic` — never `openrouter` |
| `skills` | `[core]` | Always exactly `["core"]` |
| `system_prompt_template` | `<short>` | Format: `"You are <name>. <description summary>"` — max 2 sentences |

### Required Fields with Fixed Defaults

| Field | Value |
|-------|-------|
| `auth_method` | `api_key` |
| `llm_model` | `""` (empty string) |
| `temperature` | `0.7` (use `0.3` only if the agent does financial/compliance work requiring deterministic output) |
| `hiring_fee` | `0.0` |
| `salary_per_1m_tokens` | `0.0` |
| `agent_family` | `""` (empty string) |

### Recommended Fields

| Field | Rule |
|-------|------|
| `personality_tags` | 2-5 tags from the Personality Tags list below |
| `tools` | List of tool names if the agent has tools |

---

## Step 4: Create `skills/core.md`

This is a skill definition template. Use the following default content:

```markdown
# Core Skill

This is the agent's primary skill. It defines the agent's main capability
and working methodology.

## Guidelines

- Follow the instructions in the agent's system prompt
- Apply domain expertise as described in the agent profile
- Maintain the agent's defined personality and communication style
- Deliver outputs that match the agent's role and specialization
```

> **Note:** `skills/core.md` is a template for the skill framework. The agent's full description and instructions live in `DESCRIPTION.md`. Do NOT copy the source agent body into `skills/core.md`.

If the source agent has multiple distinct skills/capabilities, you may create additional skill files. Each skill is a folder with a `SKILL.md` inside:

```
skills/
├── core.md                    # Default skill (always present)
├── code-review/
│   └── SKILL.md               # Additional skill with frontmatter
└── security-audit/
    └── SKILL.md
```

Each `SKILL.md` must have YAML frontmatter:

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
```

---

## Step 5: Set Up `tools/`

### Simple agents (no tools)

Create a default `tools/manifest.yaml`:

```yaml
# Tool manifest — declare tools this agent can use
# Uncomment and customize as needed

# builtin_tools:
#   - Read
#   - Write
#   - Bash
```

### Agents with MCP tools

Place a `.mcp.json` under `tools/` using the standard MCP format. Optionally create a `TOOL.md` for each server to document usage:

```
tools/
├── .mcp.json                  # MCP server definitions
├── filesystem/
│   └── TOOL.md                # What this tool does, when to use it
└── github/
    └── TOOL.md
```

**`tools/.mcp.json`:**
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

### Agents with custom tools

Each custom tool gets its own folder with `TOOL.md`, `manifest.yaml`, and optionally an implementation script:

```
tools/
└── run-tests/
    ├── TOOL.md                # Usage docs
    ├── manifest.yaml          # Metadata
    └── run.sh                 # Implementation
```

**`tools/run-tests/TOOL.md`:**
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

**`tools/run-tests/manifest.yaml`:**
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

## Role Guidelines

Assign the **most specific job title** that matches what the agent actually does. Do NOT use generic labels.

| Agent Domain | Available Roles |
|-------------|----------------|
| Design | `Designer`, `UI Designer`, `UX Designer`, `UX Researcher`, `Brand Designer` |
| Engineering | `Engineer`, `Frontend Engineer`, `Backend Engineer`, `DevOps Engineer`, `SRE`, `Security Engineer`, `Data Engineer`, `Database Engineer`, `AI Engineer`, `ML Engineer`, `Mobile Engineer`, `Blockchain Engineer`, `Embedded Engineer`, `Software Architect`, `Code Reviewer`, `Technical Writer` |
| Marketing | `Marketer`, `SEO Specialist`, `Content Creator`, `Content Strategist`, `Social Media Marketer`, `Growth Marketer`, `E-commerce Marketer`, `Community Manager`, `ASO Specialist` |
| Paid Media & Advertising | `Media Buyer`, `PPC Specialist`, `Programmatic Buyer`, `Ad Strategist` |
| Sales | `Sales Coach`, `Sales Strategist`, `Sales Engineer`, `Sales Analyst` |
| Product | `Product Manager`, `Product Researcher` |
| Project Management | `Project Manager`, `Producer`, `Operations Manager` |
| Game Development | `Game Engineer`, `Game Designer`, `Technical Artist`, `Narrative Designer`, `Audio Engineer`, `Level Designer` |
| Spatial / XR | `XR Engineer` |
| Testing & QA | `QA Engineer`, `Performance Engineer`, `Accessibility Tester` |
| Support & Operations | `Support`, `Data Analyst`, `Analyst`, `Operations` |
| Finance & Compliance | `Finance`, `Compliance Auditor`, `Compliance Specialist`, `Security Auditor` |
| Specialized | `Recruiter`, `Consultant`, `Advisor`, `Developer Advocate`, `Strategist`, `Training Designer`, `Orchestrator` |

**How to pick:** Read the agent's name and description. Find the role that most precisely describes their primary function. If nothing fits precisely, use the closest match. Never fall back to just "Specialist".

---

## Personality Tags

Choose 2-5 from this list, based on the agent's described working style and personality:

| Tag | When to use |
|-----|-------------|
| `autonomous` | Works independently, self-directed |
| `systematic` | Follows structured processes, methodical |
| `creative` | Focuses on innovation, originality, artistic expression |
| `analytical` | Data-driven, metrics-focused, evidence-based |
| `collaborative` | Team-oriented, works across disciplines |
| `detail-oriented` | Precise, meticulous, pixel-perfect |
| `strategic` | Big-picture thinking, long-term planning |
| `thorough` | Comprehensive coverage, leaves nothing unchecked |
| `performance-focused` | Optimization-oriented, speed/efficiency matters |
| `security-focused` | Security/compliance-aware, risk-conscious |

---

## Converting from Claude Code Agent

A Claude Code agent typically has:
- `CLAUDE.md` — project instructions and constraints
- `.mcp.json` — MCP server configurations
- No `profile.yaml`

### Conversion steps:

**1. Extract system prompt from `CLAUDE.md`**

Read `CLAUDE.md` and copy its content into `profile.yaml` → `system_prompt_template` (summarized to 1-2 sentences). The full `CLAUDE.md` content goes into `DESCRIPTION.md`.

**2. Extract skills from the codebase**

Look for distinct capabilities described in `CLAUDE.md` or in any `skills/` directory. Each logical capability becomes a skill folder.

If `CLAUDE.md` has sections like "## Code Review", "## Debugging" — each becomes a separate skill.

**3. Copy `.mcp.json` into `tools/`**

```bash
mkdir -p my-talent/tools
cp /path/to/claude-agent/.mcp.json my-talent/tools/.mcp.json
```

No conversion needed — `.mcp.json` keeps the standard format.

**4. Set agent_family and auth_method**

```yaml
agent_family: claude
auth_method: cli
hosting: self
```

---

## Converting from OpenClaw Agent

An OpenClaw agent typically has a graph-based workflow definition, channel configurations, and `launch.sh`.

### Conversion steps:

1. **Map the workflow graph to skills** — each node/stage becomes a skill folder
2. **Set `agent_family: openclaw`** and `hosting: self`
3. **Preserve `launch.sh`** — the platform uses it to start self-hosted agents
4. **Extract channel configs into skills** — create skills for channel-specific behavior

---

## Converting from a Generic Python/Node Agent

For custom agents (LangChain, AutoGen, CrewAI, etc.):

1. **Find the system prompt** — look for `system_message`, `system_prompt`, or `instructions` in source code or config
2. **Identify skills** — distinct task handlers → each becomes a skill folder
3. **Identify tools** — tool definitions → `tools/manifest.yaml` or custom tool folders
4. **Set hosting** — `company` (stateless API), `self` (runs on user's machine), or `remote` (external HTTP webhook)

---

## Common Mistakes — Read This

| # | Mistake | Correct Behavior |
|---|---------|-----------------|
| 1 | Modifying `DESCRIPTION.md` content (removing emojis, reformatting) | Copy source body **exactly as-is**, byte-for-byte |
| 2 | Forgetting `DESCRIPTION.md` | This is the MOST IMPORTANT file. Create it FIRST |
| 3 | Copying source body into `skills/core.md` instead of `DESCRIPTION.md` | Source body → `DESCRIPTION.md`. `skills/core.md` uses the default template |
| 4 | Putting emoji in `avatar` field | `avatar` must be an image filename (e.g., `avatar.png`), NOT an emoji. Omit the field if no image is available |
| 5 | Setting `hosting: company` | Must be `self` |
| 6 | Setting `api_provider: openrouter` | Must be `anthropic` |
| 7 | Using generic role like "Specialist" | Use the most specific role from the Role Table |
| 8 | Making `system_prompt_template` too long | Keep it to 1-2 sentences max |
| 9 | Forgetting `tools/` directory | Always create it, even if just a template manifest |
| 10 | Wrong `id` (doesn't match directory name) | Must match exactly |

---

## Validation Checklist

Before publishing, verify:

- [ ] `profile.yaml` has `id`, `name`, `avatar`, `description`, `role`, `system_prompt_template`
- [ ] `avatar` is an image filename (e.g., `avatar.png`) and the file exists, OR the field is omitted
- [ ] `DESCRIPTION.md` exists with verbatim agent content
- [ ] `skills/core.md` exists (template or custom)
- [ ] Each skill in `profile.yaml` → `skills` has a matching file
- [ ] `tools/` directory exists with at least `manifest.yaml`
- [ ] `description` is specific (not "An AI agent that helps with tasks")
- [ ] `hosting: self` and `api_provider: anthropic` are set correctly
- [ ] No secrets/API keys are hardcoded anywhere
- [ ] No illegal content, political propaganda, or harmful material in any file

---

## Publishing

**IMPORTANT: Create a new, separate repo for each converted talent. Do NOT commit into this template repo.**

```bash
# Create a new directory for the talent (NOT inside the template repo)
mkdir /tmp/my-talent && cd /tmp/my-talent

# ... create all talent files (profile.yaml, DESCRIPTION.md, skills/, tools/, etc.) ...

# Push to GitHub as a new repo
git init && git add -A && git commit -m "Initial talent"
gh repo create my-talent --public --push

# Register on Talent Market
# Go to https://one-man-company.com → Add Talent → paste your repo URL
```

Or via API:
```bash
curl -X POST https://one-man-company.com/api/v1/repos \
  -H "X-API-Key: tm_live_..." \
  -H "Content-Type: application/json" \
  -d '{"url": "https://github.com/you/my-talent"}'
```

---

## Full Example

**Source file:** `marketing-seo-specialist.md`

```markdown
---
name: SEO Specialist
description: Expert in technical SEO, content optimization, and search strategy
color: blue
emoji: 🔍
vibe: Gets your site to page one and keeps it there.
---

# 🔍 SEO Specialist Agent

You are **SEO Specialist**, an expert in technical SEO...

## 🧠 Your Identity & Memory
- **Role**: Search engine optimization expert
...
```

### Output:

**`marketing-seo-specialist/DESCRIPTION.md`:**
```markdown
# 🔍 SEO Specialist Agent

You are **SEO Specialist**, an expert in technical SEO...

## 🧠 Your Identity & Memory
- **Role**: Search engine optimization expert
...
```

Note: The emojis `🔍` and `🧠` are **preserved exactly** from the source.

**`marketing-seo-specialist/profile.yaml`:**
```yaml
id: marketing-seo-specialist
name: SEO Specialist
avatar: avatar.png
description: >
  Expert in technical SEO, content optimization, and search strategy
role: SEO Specialist
hosting: self
auth_method: api_key
api_provider: anthropic
llm_model: ""
temperature: 0.7
hiring_fee: 0.0
salary_per_1m_tokens: 0.0
skills:
  - core
personality_tags:
  - analytical
  - strategic
  - thorough
system_prompt_template: >
  You are SEO Specialist. Expert in technical SEO, content optimization, and search strategy.
agent_family: ""
```

**`marketing-seo-specialist/skills/core.md`:**
```markdown
# Core Skill

This is the agent's primary skill. It defines the agent's main capability
and working methodology.

## Guidelines

- Follow the instructions in the agent's system prompt
- Apply domain expertise as described in the agent profile
- Maintain the agent's defined personality and communication style
- Deliver outputs that match the agent's role and specialization
```

**`marketing-seo-specialist/tools/manifest.yaml`:**
```yaml
# Tool manifest — declare tools this agent can use
# Uncomment and customize as needed

# builtin_tools:
#   - Read
#   - Write
#   - Bash
```
