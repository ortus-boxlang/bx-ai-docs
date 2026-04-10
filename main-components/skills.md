---
description: >-
  Skills are reusable markdown instruction files that can be injected into agent
  context automatically or loaded on demand.
icon: graduation-cap
---

# AI Skills

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Skills are reusable markdown instruction files that give agents specialized knowledge and behavior. Each skill is a focused unit of expertise — like "SQL Optimization", "BoxLang Expert", or "Code Reviewer".

## 🎓 What are Skills?

A skill is a markdown file (named `SKILL.md`) containing instructions, examples, and domain knowledge. Skills follow the **Agent Skills open standard** — each skill lives in its own named subdirectory under `.ai/skills/`.

```
.ai/skills/
  sql-optimizer/
    SKILL.md
  coding/
    boxlang-expert/
      SKILL.md
    code-reviewer/
      SKILL.md
```

## Skill File Format

Each `SKILL.md` uses optional YAML frontmatter followed by markdown content:

```markdown
---
name: SQL Optimizer
description: >
  Expert at writing and optimizing SQL queries for performance.
  Use this skill when working with database queries, indexes,
  or query performance.
---

# SQL Optimization Expert

You are an expert SQL query optimizer. When writing or reviewing queries:

1. Always consider index usage — prefer index-covered queries
2. Avoid `SELECT *` — specify only required columns
3. Use `EXPLAIN` to analyze query plans
4. Prefer JOINs over subqueries where possible
5. Use CTEs (WITH clauses) for complex multi-step logic

## Common Patterns

**Pagination:**
Use keyset pagination instead of OFFSET for large tables:
```sql
SELECT * FROM orders WHERE id > :lastId ORDER BY id LIMIT 20
```
```

If `description` is omitted from frontmatter, BoxLang AI uses the first paragraph of content as the description.

## Two Modes of Operation

### Always-On Skills (`skills`)

Always-on skills are loaded unconditionally and injected into the agent's system context on every call. Use these for core behavioral rules the agent should always follow.

```javascript
import bxModules.bxai.models.skills.AiSkill;

agent = aiAgent(
    name  : "Assistant",
    skills: [
        // Load from a file
        AiSkill::fromPath( ".ai/skills/core/SKILL.md" ),

        // Or use the aiSkill() BIF
        aiSkill( ".ai/skills/professional-tone/SKILL.md" ),

        // Or define inline
        aiSkill(
            name       : "tone",
            description: "Maintain a professional, concise tone",
            content    : "Always respond concisely and professionally. Avoid jargon."
        )
    ]
)
```

### Lazy-Loaded Skills (`availableSkills`)

Available skills form a pool that the agent can load on demand via a built-in `loadSkill()` tool. The agent is shown skill names and descriptions and decides which are relevant to each query. This avoids inflating the context window with unneeded instructions.

```javascript
// Scan a whole directory — every SKILL.md becomes an available skill
agent = aiAgent(
    name           : "Specialist",
    availableSkills: aiSkill( ".ai/skills" )  // Scans recursively by default
)
```

When a user asks about SQL optimization, the agent finds the `sql-optimizer` skill and calls `loadSkill("sql-optimizer")` before answering.

### Combining Both

```javascript
coreSkills = [
    aiSkill( ".ai/skills/tone/SKILL.md" ),
    aiSkill( ".ai/skills/safety/SKILL.md" )
]

specialistSkills = aiSkill( ".ai/skills/domains" )  // 20+ domain skills

agent = aiAgent(
    name           : "Expert",
    skills         : coreSkills,        // Always loaded
    availableSkills: specialistSkills   // Loaded on demand
)
```

## Using the `aiSkill()` BIF

```javascript
// Load all skills from default directory (.ai/skills)
skills = aiSkill()

// Load all skills from a custom directory
skills = aiSkill( ".ai/custom-skills" )

// Load a single skill from a file
skill = aiSkill( ".ai/skills/sql-optimizer/SKILL.md" )

// Create an inline skill without a file
skill = aiSkill(
    name       : "formatter",
    description: "Format responses as markdown tables",
    content    : "Always present tabular data as markdown tables with headers."
)

// Disable recursive scan
skills = aiSkill( ".ai/skills", recurse: false )
```

See [aiSkill() Reference](../advanced/reference/built-in-functions/aiskill.md) for full parameter documentation.

## Global Skills

Global skills are automatically injected into **every** agent's system context — even agents that don't explicitly declare any skills. Configure them in module settings:

```javascript
// ModuleConfig.bx / Application settings
moduleSettings = {
    globalSkills: aiSkill( ".ai/global-skills" )
}
```

Use `aiGlobalSkills()` to inspect the current global pool at runtime:

```javascript
globals = aiGlobalSkills()
println( "Global skills loaded: #globals.len()#" )
```

## Fluent API

```javascript
agent = aiAgent( name: "Assistant" )
    .withSkills( [ coreSkill, toneSkill ] )
    .withAvailableSkills( aiSkill( ".ai/skills" ) )
```

## 🌐 Discover, Install, and Publish Community Skills

The open Agent Skills ecosystem lets you discover reusable skills at [skills.sh](https://skills.sh/) and install them with the official CLI from [vercel-labs/skills](https://github.com/vercel-labs/skills).

### Discover Skills on skills.sh

Browse by topic, popularity, or repository owner:

- Directory: [https://skills.sh/](https://skills.sh/)
- Official collection: [https://skills.sh/official](https://skills.sh/official)
- Source and CLI docs: [https://github.com/vercel-labs/skills](https://github.com/vercel-labs/skills)

Once you find a skill repository (for example `owner/repo`), install it into your project.

### Install Skills with the npx CLI

Use the official `skills` CLI:

```bash
# Install from a GitHub repo (project-local install)
npx skills add owner/repo

# List skills in a repo before installing
npx skills add owner/repo --list

# Install one specific skill from the repo
npx skills add owner/repo --skill my-skill-name

# Install globally instead of project-local
npx skills add owner/repo -g
```

After install, wire the skill into your BoxLang AI agent as always-on or available-on-demand:

```javascript
agent = aiAgent(
  name           : "BoxLang Assistant",
  skills         : [ aiSkill( "/.ai/skills/boxlang-expert/SKILL.md" ) ],
  availableSkills: aiSkill( "/.ai/skills" )
)
```

### Create Your Own Skill

You can scaffold a skill quickly:

```bash
# Create a new skill folder with a SKILL.md template
npx skills init boxlang-api-designer
```

Then ensure frontmatter includes a clear `name` and `description`:

```markdown
---
name: boxlang-api-designer
description: Helps design BoxLang modules and APIs with practical conventions.
---

# BoxLang API Designer

## When to Use

Use when creating or reviewing BoxLang module APIs, naming, and documentation.
```

### Publish to GitHub with BoxLang Tags

To make your skill easy to discover, publish it in a public GitHub repository and add relevant repository topics.

Recommended topics:

- `boxlang`
- `agent-skills`
- `ai`
- `boxlang-ai` (if the skill is specific to this module)

Repository layout can be either a single root `SKILL.md` or a `skills/` directory containing one or more skills.

Then users can install directly from your repo:

```bash
npx skills add your-org/your-skills-repo --skill boxlang-api-designer
```

## Related Pages

* [Agent Skills (agents/)](agents/skills.md) — Skills in the context of agent configuration
* [aiSkill() Reference](advanced/reference/built-in-functions/aiskill.md) — Full BIF reference
* [aiGlobalSkills() Reference](advanced/reference/built-in-functions/aiglobalskills.md) — Global skills BIF
