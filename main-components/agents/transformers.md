---
description: >-
  Using transformers with agents to process inputs and outputs, enabling
  structured extraction and multi-stage pipelines.
icon: arrow-right-arrow-left
---

# Agents with Transformers

Agents can use transformers to pre-process inputs and post-process outputs, enabling structured extraction, content cleaning, and multi-stage AI pipelines.

## Output Transformation

Transform agent responses automatically via pipeline:

```javascript
import bxModules.bxai.models.transformers.TextCleanerTransformer;

agent = aiAgent(
    name        : "Content Generator",
    instructions: "Generate content based on user requests"
)

// Post-process the agent's output
cleaner = new TextCleanerTransformer( {
    stripHTML        : true,
    removeExtraSpaces: true
} )

pipeline = aiMessage()
    .user( "${prompt}" )
    .to( agent )
    .transform( r => r.content )
    .to( cleaner )
    .transform( cleaned => {
        return {
            cleaned   : cleaned,
            wordCount : cleaned.listLen( " " ),
            charCount : len( cleaned )
        }
    } )

result = pipeline.run( { prompt: "Write about BoxLang AI" } )
println( "Word count: #result.wordCount#" )
```

## Input Processing

Pre-process user input before it reaches the agent:

```javascript
// Normalize input before sending to agent
inputCleaner = aiTransform( input => {
    return input
        .trim()
        .reReplace( "[^\w\s]", "", "all" )  // Remove special chars
        .reReplace( "\s+", " ", "all" )      // Normalize spaces
} )

agent = aiAgent(
    name        : "Clean Input Agent",
    instructions: "Process cleaned user inputs"
)

pipeline = inputCleaner
    .transform( cleaned => aiMessage().user( cleaned ) )
    .to( agent )

response = pipeline.run( "   What  is  BoxLang???   " )
// Input is cleaned before reaching the agent
```

## Structured Output from Agents

Use transformers to extract and parse structured data from agent responses:

```javascript
agent = aiAgent(
    name        : "Data Extractor",
    instructions: "Extract structured information from text as JSON"
)

jsonParser = aiTransform( response => {
    try {
        return jsonDeserialize( response.content )
    } catch( any e ) {
        return { error: "Invalid JSON", raw: response.content }
    }
} )

pipeline = aiMessage()
    .user( "Extract person data from: ${text}" )
    .to( agent )
    .transform( r => r.content )
    .to( jsonParser )

data = pipeline.run( { text: "John Doe, age 30, john@example.com" } )
println( "Name: #data.name#" )
println( "Email: #data.email#" )
```

## Multi-Stage Agent Processing

Chain multiple agents with transformers between them to form complex workflows:

```javascript
// Stage 1: Research
researcher = aiAgent(
    name        : "Researcher",
    instructions: "Research topics thoroughly and comprehensively"
)

// Stage 2: Summarize
summarizer = aiAgent(
    name        : "Summarizer",
    instructions: "Create concise, accurate summaries"
)

// Stage 3: Polish
editor = aiAgent(
    name        : "Editor",
    instructions: "Polish writing for clarity and professionalism"
)

// Stats transformer between summarizer and editor
statTransformer = aiTransform( text => {
    return {
        text      : text,
        sections  : text.split( "\n\n" ),
        wordCount : text.listLen( " " )
    }
} )

// Three-stage pipeline
pipeline = aiMessage()
    .user( "Research ${topic}" )
    .to( researcher )
    .transform( r => "Summarize: " & r.content )
    .to( summarizer )
    .transform( r => r.content )
    .to( statTransformer )
    .transform( data => "Edit and polish: " & data.text )
    .to( editor )

result = pipeline.run( { topic: "Quantum Computing" } )
println( result )
```

## Related Pages

* [Getting Started](getting-started.md) — Creating your first agent
* [Streaming](streaming.md) — Streaming output with agents
* [Advanced Patterns](advanced.md) — Pipeline integration and chaining
* [Transformers](../transformers/README.md) — All built-in transformer types
