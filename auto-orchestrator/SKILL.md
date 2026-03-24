---
name: auto-orchestrator
description: >
  Automatically selects and invokes the right skill(s) for any task without the user needing to know which skill to use.
  Use this skill whenever a user asks you to do something and you recognize that a specialized skill exists for that task —
  even if they haven't mentioned a skill by name. This includes tasks involving files (PDF, Word, Excel, PowerPoint),
  browser automation, frontend design, debugging, data pipelines, web apps, SEO, scheduling, security scanning,
  or any specialized workflow. If the user's request maps to a known skill, invoke this skill to route to the right one.
  Trigger it proactively — if there's even a moderate chance a skill applies, use it.
---

# Auto-Orchestrator

You are the skill router. Your job is to read the user's task, decide which installed skill(s) best handle it, and invoke them — in sequence if needed — without making the user think about skill names.

## How to Route

### Step 1: Understand the task
Read the full user request. Identify:
- **What** they want produced or accomplished (a file? a live webpage? a data pipeline fix?)
- **What format or domain** it involves (PDF, Excel, browser, Python, Airflow, etc.)
- **How complex** it is (single skill, or a chain of skills?)

### Step 2: Match to the skill map below
Pick the best matching skill(s). When multiple apply, think about the logical order — e.g., design first, then test; build first, then audit.

### Step 3: Invoke the skill(s) using the Skill tool
Call `Skill` with the matched skill name. Pass the user's original task as args. If the task chains multiple skills, run them in order, passing relevant outputs forward.

---

## Skill Map

### Document & File Processing
| User intent | Skill to invoke |
|---|---|
| Read, create, edit, extract text from a **PDF** | `anthropic-skills:pdf` |
| Create or edit a **Word document** (.docx) | `anthropic-skills:docx` |
| Work with **Excel / spreadsheets** (.xlsx, .csv with formulas) | `anthropic-skills:xlsx` |
| Create or edit a **PowerPoint presentation** (.pptx) | `anthropic-skills:pptx` |

### Web & Browser
| User intent | Skill to invoke |
|---|---|
| Automate a browser, fill forms, take screenshots, scrape a site | `anthropic-skills:browser-use` |
| Navigate websites, click, extract data for AI agents | `anthropic-skills:agent-browser` |
| Test a local web application (Playwright, UI checks) | `anthropic-skills:webapp-testing` |
| Audit a website for performance, accessibility, SEO, best practices | `anthropic-skills:web-quality-audit` |

### Design & Frontend
| User intent | Skill to invoke |
|---|---|
| Build or improve a UI, landing page, dashboard, or visual component | `anthropic-skills:frontend-design` |
| Create SEO-driven pages at scale from templates or data | `anthropic-skills:programmatic-seo` |

### Debugging
| User intent | Skill to invoke |
|---|---|
| Debug a bug, test failure, or unexpected behavior — structured approach | `anthropic-skills:systematic-debugging` |
| Deep diagnosis, profiling, root cause analysis | `anthropic-skills:debugging-strategies` |
| Debug an Apache Airflow DAG failure | `anthropic-skills:debugging-dags` |

### Data & AI Pipelines
| User intent | Skill to invoke |
|---|---|
| Work with Apache Airflow DAGs, tasks, or pipelines | `anthropic-skills:debugging-dags` |
| Build or query data pipelines, answer business questions from a warehouse | *(use data engineering skills if available, otherwise proceed directly)* |
| Credit risk data cleaning, variable screening | `anthropic-skills:datanalysis-credit-risk` |

### Claude API & Agent SDK
| User intent | Skill to invoke |
|---|---|
| Build an app using the Claude API or Anthropic SDK (`import anthropic`) | `claude-api` |

### Productivity & Meta
| User intent | Skill to invoke |
|---|---|
| Schedule a recurring task or set up automation | `anthropic-skills:schedule` |
| Upload content to NotebookLM, generate podcasts or summaries | `anthropic-skills:anything-to-notebooklm` |
| Query a NotebookLM notebook | `anthropic-skills:notebooklm` |
| Flutter app needing local data persistence / SQLite | `anthropic-skills:flutter-working-with-databases` |
| Optimize token usage for cost-effective Claude Code use | `anthropic-skills:token-efficiency` |
| Find and install a skill for a new kind of task | `anthropic-skills:find-skills` |
| Execute a written implementation plan step by step | `anthropic-skills:executing-plans` |

---

## Chaining Skills

Some tasks need more than one skill in sequence. Common chains:

- **"Build me a landing page and audit it"** → `frontend-design` → `web-quality-audit`
- **"Create an Excel report from this PDF"** → `pdf` (extract) → `xlsx` (build report)
- **"Create a presentation from this Word doc"** → `docx` (read) → `pptx` (build slides)
- **"Test my web app and fix bugs"** → `webapp-testing` → `systematic-debugging`

For chains: run the first skill, collect its output, then feed that output into the next skill invocation.

---

## When No Skill Matches

If nothing in the skill map clearly fits, proceed with your general capabilities. Don't force an irrelevant skill onto the task — it's better to do it directly than to route poorly.

If the task is genuinely novel and a skill *should* exist but doesn't, mention it to the user and offer to create one with `anthropic-skills:skill-creator`.

---

## Example Routing

**User:** "Can you read this contract.pdf and pull out all the dates and parties involved?"
→ Invoke `anthropic-skills:pdf` with the user's full request.

**User:** "I need a nice dashboard to show our Q4 metrics"
→ Invoke `anthropic-skills:frontend-design`.

**User:** "My Airflow DAG keeps failing on the transform step"
→ Invoke `anthropic-skills:debugging-dags`.

**User:** "Build a Next.js landing page and make sure it scores well on Lighthouse"
→ Invoke `anthropic-skills:frontend-design` first, then `anthropic-skills:web-quality-audit`.

**User:** "Write a Python script using the Anthropic SDK to summarize PDFs"
→ Invoke `claude-api`.
