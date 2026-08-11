---
description: >-
  Skills inject reusable domain knowledge into an agent's system context —
  either always-on or lazily loaded on demand.
icon: book-open
---

# Agent Skills

{% hint style="info" %}
**Since BoxLang AI v3.0+**
{% endhint %}

Skills are named blocks of domain knowledge or instructions that can be injected into an agent's system context. Think of them as modular expertise — a SQL optimization skill, a security review skill, a brand voice skill — that you can mix and match across agents without duplicating instructions.

There are two modes:

* **Always-on skills** (`skills:`) — content is injected into every system message
* **Lazy skills** (`availableSkills:`) — an index is shown to the AI; full content is only loaded when the AI explicitly requests it via the built-in `loadSkill()` tool

## Skill File Format

Skills are markdown files with optional YAML frontmatter, stored under `.agents/skills/` by convention:

```
.agents/skills/
  sql-optimizer/
    SKILL.md
  coding/
    boxlang-expert/
      SKILL.md
  security/
    SKILL.md
```

Each `SKILL.md` file:

```markdown
---
name: sql-optimizer
description: Expert SQL query optimization guidance. Use when writing or reviewing SQL queries.
---

## SQL Optimization Principles

Always use indexed columns in WHERE clauses...
Use EXPLAIN ANALYZE to profile slow queries...
Normalize data but denormalize for read-heavy tables...
```

If `description` is omitted, the first paragraph of the markdown body is used — matching the Agent Skills open standard.

## Loading Skills

```javascript
// Load all skills from the default .agents/skills directory
skills = aiSkill()

// Load from a custom path
skills = aiSkill( ".agents/my-skills" )

// Load a single skill file
skill = aiSkill( ".agents/skills/sql-optimizer/SKILL.md" )

// Create an inline skill (no file needed)
inlineSkill = aiSkill(
    name       : "tone",
    description: "Maintain professional and friendly tone",
    content    : "Always use first names, keep responses under 3 paragraphs..."
)
```

## Always-On Skills

Full skill content is injected into the system context on every agent call:

```javascript
agent = aiAgent(
    name  : "SQLReviewer",
    skills: aiSkill( ".agents/skills/sql-optimizer" )
)

// The SQL optimization instructions are always in context
response = agent.run( "Review this query: SELECT * FROM orders WHERE status = 'pending'" )
```

Multiple always-on skills:

```javascript
agent = aiAgent(
    name  : "ContentEditor",
    skills: [
        aiSkill( ".agents/skills/brand-voice" ),
        aiSkill( ".agents/skills/grammar-rules" )
    ]
)
```

## Lazy-Loaded Skills (`availableSkills`)

With lazy loading, only a compact index (name + description) is injected into the system context. The AI sees what's available and requests the full content of a skill when needed by calling the built-in `loadSkill()` tool:

```javascript
agent = aiAgent(
    name           : "EngineeringAssistant",
    availableSkills: aiSkill( ".agents/skills" )   // Scans entire directory
)
// The agent sees: "Available skills: sql-optimizer, boxlang-expert, security-review"
// When asked a SQL question, the AI calls loadSkill("sql-optimizer") to get the full content
```

## Combining Both Modes

Use always-on for globally relevant skills and lazy-loading for specialized knowledge:

```javascript
agent = aiAgent(
    name           : "PlatformAssistant",
    skills         : aiSkill( ".agents/skills/core" ),          // Always injected
    availableSkills: aiSkill( ".agents/skills/specialized" )    // Available on demand
)
```

## Global Skills

BoxLang AI auto-discovers skills at module startup from a configured directory and makes them **available** (lazy-loaded via `loadSkill()`, not always-on) to every agent — zero config if `.agents/skills` already exists in your project:

```javascript
// config/boxlang.json — both settings default to this already
{
    "modules": {
        "bxai": {
            "settings": {
                "skillsDirectory": "/.agents/skills",  // set to "" to disable
                "autoLoadSkills" : true
            }
        }
    }
}
```

See [Global Skills](../skills.md#global-skills) on the main Skills page for the full mechanism. Access the global pool at runtime:

```javascript
globalSkills = aiGlobalSkills()
println( "Global skills: #globalSkills.len()#" )
```

## Adding Skills After Construction

```javascript
agent = aiAgent( name: "Assistant" )

// Add always-on skill
agent.withSkills( [ aiSkill( ".agents/skills/tone" ) ] )

// Add lazy skill
agent.withAvailableSkills( [ aiSkill( ".agents/skills/advanced" ) ] )
```

## Related Pages

* [AI Skills](../skills.md) — Full skills reference with `aiSkill()` BIF
* [aiSkill BIF](../../advanced/reference/built-in-functions/aiskill.md) — BIF signature reference
* [aiGlobalSkills BIF](../../advanced/reference/built-in-functions/aiglobalskills.md) — Global skills access
