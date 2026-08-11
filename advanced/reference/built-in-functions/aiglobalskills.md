---
description: Access the globally shared pool of AI skills that are automatically injected into every agent's system context.
icon: graduation-cap
---

# aiGlobalSkills

Access the globally shared pool of AI skills that are automatically injected into every agent's system context.

## Syntax

```javascript
aiGlobalSkills()
```

## Parameters

No parameters.

## Returns

Returns an `Array` of `AiSkill` instances configured as global skills in `ModuleConfig.bx`. Returns an empty array if no global skills are configured.

## Examples

### Inspect Global Skills

```javascript
globals = aiGlobalSkills()

println( "Global skills: #globals.len()#" )

globals.each( skill => {
    println( " - #skill.getName()#: #skill.getDescription()#" )
})
```

### Use Global Skills with an Agent

```javascript
// Global skills are already injected automatically.
// This example shows how to add them explicitly alongside custom skills.
agent = aiAgent(
    name  : "assistant",
    skills: aiGlobalSkills().append( myCustomSkill )
)
```

### Configure Global Skills in ModuleConfig.bx

Global skills are registered in your module's `ModuleConfig.bx` and apply to every agent across the application:

```javascript
// ModuleConfig.bx (inside a BoxLang module only)
variables.moduleSettings = {
    globalSkills: aiSkill( path: ".ai/skills/global" )
}
```

For application-level registration without a module, add skills directly to each agent or use `availableSkills` with a shared array variable.

## Related Pages

* [Skills](../../../main-components/skills.md) — Full skills documentation
* [aiSkill()](aiskill.md) — Load skills from files or create inline
