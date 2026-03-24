---
name: auto-orchestrator
description: >
  Automatically selects and invokes the right skill(s) or plugin(s) for any task without the user needing to know which
  skill or plugin to use. Use this skill whenever a user asks you to do something and you recognize that a specialized
  skill or MCP plugin exists for that task — even if they haven't mentioned one by name. This includes tasks involving
  files (PDF, Word, Excel, PowerPoint), browser automation, frontend design, debugging, data pipelines, web apps,
  SEO, scheduling, security scanning, library documentation, or any specialized workflow.
  Trigger it proactively — if there's even a moderate chance a skill or plugin applies, use it.
---

# Auto-Orchestrator

You are the task router. Your job is to read the user's task, scan every currently installed **skill** and **MCP plugin**, decide which one(s) best handle it, and invoke them — in sequence if needed — without making the user think about names.

---

## Two kinds of tools you can route to

### 1. Skills
Installed skills appear in the `<system-reminder>` block under "The following skills are available".
**How to invoke:** `Skill("namespace:skill-name", args: "<user task>")`
Skills are high-level, guided workflows — prefer them when a skill clearly fits.

### 2. MCP Plugins
Installed MCP plugins appear in the `<system-reminder>` block as **deferred tools** — listed by name like `mcp__PluginName__tool_name`.
**How to invoke:** Use `ToolSearch` to fetch the tool schema first, then call the tool directly.
Plugins are lower-level — use them when no skill covers the task, or when calling a plugin tool is the most direct path.

---

## Routing Algorithm (apply every time)

```
1. Read the user's full task
2. Scan the LIVE system-reminder for:
   a. All available SKILLS (under "The following skills are available")
   b. All available MCP PLUGIN TOOLS (deferred tools starting with mcp__)
3. Match the task:
   a. Does a skill description/trigger clearly match? → prefer it (richer workflow)
   b. Does an MCP plugin cover it and no skill does? → use the plugin
   c. Does the task need BOTH? → chain them (skill first, plugin after, or vice versa)
4. If multiple skills match → pick the most specific one, or chain if both outputs are needed
5. If nothing matches → proceed with general capabilities
6. If a tool should exist but doesn't → tell the user, offer to create it with skill-creator
```

**Precedence rule:** Skills > Plugins when both could work. Skills have richer, more guided logic. But never ignore a plugin if it's the only thing that fits.

---

## How to Invoke Skills

```
Skill("namespace:skill-name", args: "<user's original task>")
```

Pass the user's full original request as args. The skill will handle the rest.

---

## How to Invoke MCP Plugins

MCP plugin tools are **deferred** — you must fetch their schema before calling them:

```
Step 1: ToolSearch("mcp__PluginName keyword")   ← fetch the schema
Step 2: Call the returned tool directly          ← invoke it
```

To identify which plugin server covers a task, group tools by their prefix:
- `mcp__Claude_in_Chrome__*` → Chrome browser automation (navigate, click, fill, screenshot, scrape)
- `mcp__Claude_Preview__*` → Preview & test local web apps in a browser
- `mcp__plugin_chrome-devtools-mcp_chrome-devtools__*` → Chrome DevTools (performance, Lighthouse, debugging)
- `mcp__plugin_aikido_aikido-mcp__*` → Security scanning (SAST, vulnerability detection)
- `mcp__plugin_context7_context7__*` → Live library documentation & code examples
- `mcp__mcp-registry__*` → Search and discover new MCP servers
- `mcp__scheduled-tasks__*` → Create and manage scheduled/recurring tasks

> **This list is illustrative.** Always check the live deferred-tools list in the system-reminder — new plugins you install will appear there automatically.

---

## Skill Routing Reference (illustrative, not exhaustive)

Always scan the live skill list first. These are examples of how descriptions map to skills:

| User intent | Example skill |
|---|---|
| Anything with a PDF | `anthropic-skills:pdf` |
| Word document (.docx) | `anthropic-skills:docx` |
| Excel / spreadsheet (.xlsx) | `anthropic-skills:xlsx` |
| PowerPoint (.pptx) | `anthropic-skills:pptx` |
| Build a UI, landing page, dashboard | `anthropic-skills:frontend-design` |
| Web app quality / Lighthouse audit | `anthropic-skills:web-quality-audit` |
| Browser automation (AI agent) | `anthropic-skills:browser-use` or `anthropic-skills:agent-browser` |
| Test a local web app (Playwright) | `anthropic-skills:webapp-testing` |
| Debug a bug or test failure | `anthropic-skills:systematic-debugging` |
| Debug an Airflow DAG | `anthropic-skills:debugging-dags` |
| Claude API / Anthropic SDK code | `claude-api` |
| Schedule a recurring task | `anthropic-skills:schedule` |
| SEO pages at scale | `anthropic-skills:programmatic-seo` |
| Find/install a new skill | `anthropic-skills:find-skills` |
| Execute a written implementation plan | `anthropic-skills:executing-plans` |
| Run a security scan (Aikido) | `aikido:scan` |

---

## Plugin Routing Reference (illustrative, not exhaustive)

Always scan the live deferred-tools list first. These are examples:

| User intent | Plugin prefix to search for |
|---|---|
| "Look up docs for [library]", "how do I use [API]" | `mcp__plugin_context7_context7` |
| "Open Chrome and...", "click on...", "fill this form", "take a screenshot" | `mcp__Claude_in_Chrome` |
| "Preview my app", "open my local server in a browser" | `mcp__Claude_Preview` |
| "Run a Lighthouse audit", "profile my page", "debug in DevTools" | `mcp__plugin_chrome-devtools-mcp_chrome-devtools` |
| "Scan for security issues", "check for vulnerabilities" | `mcp__plugin_aikido_aikido-mcp` |
| "Find an MCP server for...", "what plugins exist for..." | `mcp__mcp-registry` |
| "Schedule this to run every...", "create a cron job" | `mcp__scheduled-tasks` |

---

## Chaining Examples

Some tasks need a skill AND a plugin, or multiple skills in sequence:

- **"Build a landing page and audit it"** → `frontend-design` skill → `web-quality-audit` skill
- **"Extract data from this PDF into Excel"** → `pdf` skill → `xlsx` skill
- **"Build this UI and preview it in the browser"** → `frontend-design` skill → `mcp__Claude_Preview` plugin
- **"Look up the Stripe API docs then write the integration code"** → `mcp__plugin_context7_context7` plugin → `claude-api` skill
- **"Write this feature then scan it for security issues"** → write code directly → `aikido:scan` skill or `mcp__plugin_aikido_aikido-mcp` plugin
- **"Create a presentation from this Word doc"** → `docx` skill → `pptx` skill

For chains: complete step 1, collect its output, feed it as context into step 2.

---

## Worked Examples

**User:** "Look up how to use the OpenAI streaming API"
→ Scan live skills → no exact match → scan live plugins → find `mcp__plugin_context7_context7__*`
→ `ToolSearch("context7 docs")` → call `context7__resolve-library-id` then `context7__query-docs`

**User:** "Go to github.com and find the trending Python repos"
→ Scan live skills → `agent-browser` or `browser-use` skill may match → invoke it
→ If neither is installed, scan plugins → find `mcp__Claude_in_Chrome__navigate` → ToolSearch → invoke

**User:** "Run a security scan on the code I just wrote"
→ Scan live skills → find `aikido:scan` → invoke it via Skill tool
→ (Plugin `mcp__plugin_aikido_aikido-mcp` also available but skill is preferred)

**User:** "Build a Next.js landing page, preview it, and then audit its Lighthouse score"
→ Three steps: `frontend-design` skill → `mcp__Claude_Preview` plugin (open preview) → `web-quality-audit` skill (Lighthouse)

**User:** "My Airflow DAG keeps failing on the transform step"
→ Scan live skills → `anthropic-skills:debugging-dags` matches exactly → invoke it

**User:** "Extract revenue figures from sales.pdf and put them in a spreadsheet"
→ Two skills chain: `anthropic-skills:pdf` (extract) → `anthropic-skills:xlsx` (build spreadsheet)

---

## When Nothing Matches

If no skill or plugin clearly fits, proceed with your general capabilities. Don't force a poor match.

If a tool *should* exist but doesn't, tell the user and offer to create a skill with `anthropic-skills:skill-creator`.
