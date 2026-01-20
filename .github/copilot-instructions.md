# BoxLang AI Documentation - AI Agent Instructions

## Project Overview

This is the **GitBook documentation repository** for the BoxLang AI Module (v2.x). The main module code lives in the sibling `bx-ai` repository - this repo contains ONLY user-facing documentation.

**Repository Purpose:**
- User documentation organized in GitBook format
- Code examples should reference actual BIF signatures from `bx-ai/src/main/bx/bifs/`
- Maintained separately from module code for GitBook publishing

## Documentation Structure

```
bx-ai-docs/
├── SUMMARY.md              # GitBook navigation/TOC (CRITICAL - edit for structure changes)
├── README.md               # Landing page/introduction
├── getting-started/        # Installation, quickstart, concepts
├── main-components/        # Core features (chatting, agents, tools, memory, pipelines)
│   ├── chatting/          # Multi-file: basic, advanced, service, structured output
│   ├── memory/            # Multi-tenant memory systems
│   ├── messages/          # Message templates and context
│   └── pipelines/         # Composable AI workflows
├── rag/                   # RAG, embeddings, document loaders
├── advanced/              # MCP, events, utilities
│   └── reference/         # BIF reference docs (aiChat, aiAgent, aiModel, etc.)
├── deployment/            # Production, security
├── extending-boxlang-ai/  # Custom providers, transformers, loaders, memory
└── readme/                # Release history, FAQ
```

## Critical File: SUMMARY.md

**ALWAYS update `SUMMARY.md` when adding/moving/removing documentation pages.** GitBook uses this for navigation. Structure:

```markdown
* [Page Title](path/to/file.md)
  * [Nested Page](path/to/nested.md)
```

## BoxLang Code Conventions in Docs

### ⚠️ DO NOT HALLUCINATE - Verify Against Source Code

When documenting BoxLang AI BIFs or methods, **ALWAYS verify against actual source code** in the `bx-ai` repository:
- BIF signatures: `bx-ai/src/main/bx/bifs/*.bx`
- Class methods: `bx-ai/src/main/bx/models/*.bx`

### Correct Code Patterns

```javascript
// ✅ CORRECT: aiChat() with named parameters
result = aiChat(
    messages: "What is BoxLang?",
    params: { temperature: 0.7 },
    options: { returnFormat: { name: "string", age: "numeric" } }
)

// ✅ CORRECT: aiModel() with provider name and params
model = aiModel( provider: "openai", params: { model: "gpt-4o" } )

// ✅ CORRECT: aiAgent() - no .build() method
agent = aiAgent( name: "Helper", memory: vectorMemory )
    .withInstructions( "You are helpful" )

// ✅ CORRECT: Pipeline with .transform() shorthand
pipeline = aiModel( provider: "openai" )
    .transform( text => text.toUpper() )
    .transform( text => text.trim() )

// ✅ CORRECT: Structured output via options.returnFormat
person = aiChat(
    messages: "Extract: John is 30",
    options: { returnFormat: { name: "string", age: "numeric" } }
)
```

### ❌ NEVER Use These Patterns

```javascript
// ❌ WRONG: Model name as first param
model = aiModel( "gpt-4o" )

// ❌ WRONG: .build() doesn't exist
agent = aiAgent().build()

// ❌ WRONG: .withMemory() doesn't exist (pass in constructor)
agent = aiAgent().withMemory( memory )

// ❌ WRONG: .structuredOutput() doesn't exist
result = aiChat( "Extract data" ).structuredOutput( schema )

// ❌ WRONG: structured: parameter doesn't exist
result = aiChat( messages: "Extract", structured: { ... } )

// ❌ WRONG: Verbose pipeline syntax
pipeline.to( aiTransform( closure ) )  // Use .transform( closure )
```

## Code Block Syntax Highlighting

- **Use `javascript` for BoxLang examples** - Provides best syntax highlighting until BoxLang is natively supported
- **Use `java` only for actual Java code** (rare in this docs repo)
- BoxLang has CFML-like syntax, so JavaScript highlighting works better than Java

## Documentation Writing Style

### Use Emojis Strategically
- ✅ Improve readability and visual scanning
- 💡 Use 1-2 per section/heading where helpful
- Examples: ✅ Good, ❌ Bad, 🚨 Warning, 📖 Documentation, 💡 Tip

### Code Examples
- Keep simple and focused on one concept
- Use clear, descriptive variable names (not cryptic abbreviations)
- Comment only when code isn't self-explanatory
- Show realistic use cases, not abstract examples

### Mermaid Diagrams
Many pages include Mermaid diagrams for flows (see [aiagent.md](advanced/reference/built-in-functions/aiagent.md), [basic-chatting.md](main-components/chatting/basic-chatting.md)). Use for:
- Sequence diagrams (request/response flows)
- Flowcharts (decision trees, pipelines)
- Architecture diagrams

## Interceptor Documentation

**Module vs Application Registration:**

- **Module registration** (ONLY in BoxLang modules): Use `ModuleConfig.bx` with `interceptors` array
- **Application/script registration**: Use `BoxRegisterInterceptor()` BIF
  - Reference: https://boxlang.ortusbooks.com/boxlang-language/reference/built-in-functions/system/boxregisterinterceptor

Document both approaches where relevant, clearly labeled.

## Cross-Repository Context

**Main module repo (`bx-ai`):**
- Source code: `src/main/bx/` (BoxLang), `src/main/java/` (Java runtime)
- Examples: `examples/` (60+ runnable examples across 8 categories)
- Tests: `src/test/java/` (JUnit 5 harness executing BoxLang test code)
- Build: Gradle (`./gradlew build`, `./gradlew shadowJar`)

**This docs repo (`bx-ai-docs`):**
- GitBook markdown only
- No build process
- Published to https://ai.ortusbooks.com/

## Common Documentation Tasks

### Adding a New Page
1. Create markdown file in appropriate directory
2. **UPDATE `SUMMARY.md`** with new entry
3. Follow existing page structure (frontmatter, headings, examples)
4. Use proper code block syntax (javascript for BoxLang)
5. Verify any BIF signatures against source code

### Updating BIF Reference
1. Check actual signature in `bx-ai/src/main/bx/bifs/{bifName}.bx`
2. Update [advanced/reference/built-in-functions/{bifName}.md](advanced/reference/built-in-functions/)
3. Include parameter table, examples, return formats
4. Add Mermaid diagrams for complex flows

### Fixing Hallucinated Code
1. Search for pattern: `grep -r "pattern" --include="*.md" .`
2. Verify correct pattern in source code
3. Use `multi_replace_string_in_file` for batch fixes across multiple files
4. Check both English and Spanish bootcamp lessons if applicable

## GitBook Frontmatter

Pages use YAML frontmatter for metadata:

```markdown
---
description: Brief description for SEO and GitBook
icon: house-chimney-heart
---

# Page Title
```

Common icons: `message`, `robot`, `wrench`, `brain`, `memory`, `book`, etc.

## Verification Checklist

Before committing documentation changes:

- [ ] Code examples use correct BIF signatures (verified against source)
- [ ] No `.build()`, `.withMemory()`, `.structuredOutput()`, `structured:` param
- [ ] `aiModel()` uses `provider:` parameter, not model name directly
- [ ] Pipelines use `.transform()` shorthand, not `.to( aiTransform() )`
- [ ] Structured output uses `options: { returnFormat: ... }`
- [ ] `SUMMARY.md` updated if pages added/moved/removed
- [ ] Code blocks use `javascript` syntax for BoxLang examples
- [ ] Emojis used strategically (not overboard)
- [ ] Links use relative paths (not absolute URLs)

## Resources

- Main module repo: `../bx-ai` (sibling directory)
- BoxLang language docs: https://boxlang.ortusbooks.com/
- Published docs: https://ai.ortusbooks.com/
- GitHub repo: https://github.com/ortus-boxlang/bx-ai
