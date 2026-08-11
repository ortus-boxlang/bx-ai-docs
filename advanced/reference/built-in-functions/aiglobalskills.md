---
description: Access the pool of AI skills auto-discovered from skillsDirectory at module startup and made available to every agent.
icon: graduation-cap
---

# aiGlobalSkills

Read-only accessor for the pool of skills BoxLang AI auto-discovered from `skillsDirectory` when the module started. Every agent created with `aiAgent()` gets these added as **available** (lazy-loaded via `loadSkill()`) skills automatically — no per-agent wiring needed.

## Syntax

```javascript
aiGlobalSkills()
```

## Parameters

No parameters.

## Returns

Returns an `Array` of `AiSkill` instances — the result of scanning `settings.skillsDirectory` (default `/.agents/skills`) at module startup. Returns an empty array if `autoLoadSkills` is `false`, `skillsDirectory` is empty, or the directory doesn't exist.

## Examples

### Inspect Global Skills

```javascript
globals = aiGlobalSkills()

println( "Global skills: #globals.len()#" )

globals.each( skill => {
    println( " - #skill.getName()#: #skill.getDescription()#" )
})
```

### Promote a Global Skill to Always-On

Global skills arrive as **available**, not always-on. Pull one out to make it always-on for a specific agent:

```javascript
coreSkill = aiGlobalSkills().filter( s => s.getName() == "company-tone" )

agent = aiAgent(
    name  : "assistant",
    skills: coreSkill   // Always-on for this agent, on top of the global available pool
)
```

### Configuring the Auto-Discovery Directory

```javascript
// config/boxlang.json — both settings already default to this
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

## Related Pages

* [Skills](../../../main-components/skills.md) — Full skills documentation
* [aiSkill()](aiskill.md) — Load skills from files or create inline
