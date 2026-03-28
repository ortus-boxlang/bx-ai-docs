# aiSkill

Discover and load AI skills from a directory tree, a single file, or create an inline skill definition.

## Syntax

```javascript
aiSkill( path, name, description, content, recurse )
```

## Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `path` | string | No | A directory or file path. If a directory, scanned recursively for `SKILL.md` files. If a file path, loads that skill directly. Defaults to `.ai/skills`. |
| `name` | string | No | Unique name for an inline skill (used when no `path` is given). |
| `description` | string | No | What the skill does and when to use it. If omitted, uses the first paragraph of the skill's markdown content. |
| `content` | string | No | Full instruction content for an inline skill (body text after YAML frontmatter). |
| `recurse` | boolean | No | Whether to scan subdirectories recursively when `path` is a directory. Defaults to `true`. |

## Returns

| Scenario | Returns |
| --- | --- |
| `path` is a directory | `Array` of `AiSkill` instances (may be empty if no skills found) |
| `path` is a file | Single `AiSkill` instance |
| `name` provided (inline) | Single `AiSkill` instance |
| No arguments | Empty `Array` |

## Examples

### Load All Skills from Default Directory

```javascript
// Loads all SKILL.md files found under .ai/skills/
skills = aiSkill()

agent = aiAgent(
    name           : "assistant",
    availableSkills: skills   // Lazy pool — AI activates on demand
)
```

### Load from a Custom Directory

```javascript
// Non-recursive scan of a single directory level
skills = aiSkill( path: "./prompts/personas", recurse: false )

agent = aiAgent(
    name  : "assistant",
    skills: skills   // Always-on — injected into every system message
)
```

### Load a Single Skill File

```javascript
// Load one specific skill
sqlSkill = aiSkill( path: ".ai/skills/sql-optimizer/SKILL.md" )

agent = aiAgent(
    name  : "data-analyst",
    skills: [ sqlSkill ]
)
```

### Create an Inline Skill

```javascript
// Define a skill in code — no file needed
conciseSkill = aiSkill(
    name       : "concise-writer",
    description: "Makes responses shorter and removes filler words",
    content    : """
        When writing responses:
        - Use bullet points for lists
        - Remove filler phrases like "certainly" and "of course"
        - Aim for under 100 words unless asked otherwise
    """
)

agent = aiAgent(
    name  : "assistant",
    skills: [ conciseSkill ]
)
```

### Combine Directory + Inline

```javascript
// Load shared team skills plus an inline override
skills = aiSkill()  // loads from .ai/skills/

agent = aiAgent(
    name           : "assistant",
    skills         : [ conciseSkill ],   // Always active
    availableSkills: skills              // AI activates as needed
)
```

## Related Pages

* [Skills](../../../main-components/skills.md) — Full skills documentation
* [aiGlobalSkills()](aiglobalskills.md) — Access globally shared skills
