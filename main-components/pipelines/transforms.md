---
description: >-
  Using transformations to clean, reshape, and enrich data as it flows through
  a pipeline — before, after, and around AI calls.
icon: arrows-rotate
---

# Transform Pipelines

Transformations are the glue that connects incompatible steps and shapes data at every stage of a pipeline.

## Simple Transformations

Extract, format, or modify data in a single step:

```javascript
// Extract content from an AI response
pipeline = aiMessage()
    .user( "Say hello" )
    .toDefaultModel()
    .transform( r => r.content )

// Chain multiple transforms
pipeline = aiMessage()
    .user( "List 3 colors separated by commas" )
    .toDefaultModel()
    .transform( r => r.content )                           // Extract
    .transform( text => text.split( "," ) )                // Split into array
    .transform( arr => arr.map( s => s.trim() ) )          // Trim each
    .transform( arr => arr.filter( s => s.len() > 0 ) )    // Remove empties

result = pipeline.run()
// Result: ["Red", "Blue", "Green"]
```

## Pre-Processing

Clean or enhance input **before** it reaches the AI model:

```javascript
cleanPipeline = aiTransform( input => {
        // Remove extra whitespace
        return input.reReplace( "\s+", " ", "all" ).trim()
    } )
    .transform( input => {
        // Add structured context
        return "User question: " & input & "\nPlease respond professionally."
    } )
    .to( aiModel( "openai" ) )

messyInput = "  What    is    BoxLang?   "
result = cleanPipeline.run( messyInput )
// AI receives: "User question: What is BoxLang?\nPlease respond professionally."
```

## Post-Processing

Process AI output **after** generation:

````javascript
// Extract a code block from an AI Markdown response
codePipeline = aiMessage()
    .system( "Generate code wrapped in ```javascript blocks" )
    .user( "Write a function to ${task}" )
    .toDefaultModel()
    .transform( r => r.content )
    .transform( content => {
        matches = content.reMatch( "```javascript(.*?)```" )
        return matches.len() > 0 ? matches[ 1 ].trim() : ""
    } )

result = codePipeline.run( { task: "reverse a string" } )
// Result: "function reverse(str) { return str.split('').reverse().join(''); }"
````

## Bidirectional Processing

Combine pre-processing and post-processing in a single pipeline:

```javascript
fullPipeline = aiTransform( input => input.trim() )          // Pre: clean
    .transform( input => "QUERY: " & input )                 // Pre: add context
    .to( aiModel( "openai" ) )                               // AI generation
    .transform( r => r.content )                             // Post: extract
    .transform( content => content.uCase() )                 // Post: format
    .transform( content => {                                 // Post: package
        return {
            response : content,
            length   : content.len(),
            processed: now()
        }
    } )
```

## Using Named Transformer Types

BoxLang AI ships with several built-in transformer classes for common tasks:

```javascript
import bxModules.bxai.models.transformers.TextCleanerTransformer;
import bxModules.bxai.models.transformers.JSONExtractorTransformer;
import bxModules.bxai.models.transformers.CodeExtractorTransformer;

// Clean text output
cleaner = new TextCleanerTransformer( { stripHTML: true, removeExtraSpaces: true } )

// Parse JSON from markdown fenced blocks
jsonExtractor = new JSONExtractorTransformer( { stripMarkdown: true } )

// Extract a specific language's code block
codeExtractor = new CodeExtractorTransformer( { language: "python" } )

pipeline = aiMessage()
    .user( "Generate a JSON object with user { name, age }" )
    .toDefaultModel()
    .to( jsonExtractor )

result = pipeline.run()
// result is a BoxLang struct, not a raw JSON string
println( result.name )
println( result.age )
```

See [Transformers](../transformers/README.md) for the full reference on built-in transformer types.

## Related Pages

* [Building Pipelines](building.md) — Pipeline construction and data flow
* [Multi-Model Workflows](multi-model.md) — Chaining models and reusable templates
* [Transformers](../transformers/README.md) — Built-in transformer types
