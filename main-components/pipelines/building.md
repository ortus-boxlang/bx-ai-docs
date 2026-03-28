---
description: >-
  Three ways to build pipelines, how data flows through steps, and how to
  configure parameters and options.
icon: hammer
---

# Building Pipelines

## 🔨 Three Ways to Build

All three approaches produce the same underlying `AiRunnableSequence`.

### Method 1: Fluent Chaining with `.to()`

The most common approach — chain components using `.to()`:

```javascript
pipeline = aiMessage()
    .user( "Translate '${text}' to ${language}" )
    .to( aiModel( "openai" ) )
    .to( aiTransform( r => r.content ) )

result = pipeline.run( {
    text    : "Hello, world!",
    language: "Spanish"
} )
// Result: "¡Hola, mundo!"
```

**How it works:**

* Each `.to()` call creates a new `AiRunnableSequence`
* The sequence contains all previous steps + the new step
* Pipelines are **immutable** — chaining creates new sequences

### Method 2: Helper Methods

Convenience methods for common patterns:

```javascript
// .toDefaultModel() — Connect to the default configured model
pipeline = aiMessage()
    .user( "Hello" )
    .toDefaultModel()

// .toModel( provider, apiKey ) — Connect to a specific model
pipeline = aiMessage()
    .user( "Hello" )
    .toModel( "claude", myApiKey )

// .transform( fn ) — Add a transformation step
pipeline = aiMessage()
    .user( "Count to 5" )
    .toDefaultModel()
    .transform( r => r.content.split( "," ).len() )
```

### Method 3: Explicit Sequence

For advanced scenarios, create sequences manually:

```javascript
import bxModules.bxai.models.runnables.AiRunnableSequence;

step1 = aiMessage().user( "Step 1" )
step2 = aiModel( "openai" )
step3 = aiTransform( r => r.content )

// Combine into a named sequence
pipeline = new AiRunnableSequence( [ step1, step2, step3 ] )

// Or use the aiRunnableSequence() BIF
pipeline = aiRunnableSequence( [ step1, step2, step3 ] )
```

**When to use:**

* Building pipelines dynamically
* Conditional step inclusion
* Complex branching logic

---

## 📥 Input and Output Flow

### Data Passing

Each step receives the **previous step's output** as its **input**:

```javascript
pipeline = aiTransform( x => x * 2 )        // Step 1: double
    .to( aiTransform( x => x + 10 ) )        // Step 2: add 10
    .to( aiTransform( x => x / 2 ) )         // Step 3: halve

result = pipeline.run( 5 )
// Flow: 5 → 10 → 20 → 10
// Result: 10
```

### The `_input` System Variable

When chaining AI stages with `AiMessage` templates, the previous stage's output is automatically available as `${_input}` — no extra transform needed.

**Basic usage:**

```javascript
// Code generation → Review pipeline
pipeline = aiMessage( "Write code to ${task}" )
    .toDefaultModel()
    .pipe(
        // Previous stage's output is automatically available as ${_input}
        aiMessage( "Review this code: ${_input}" )
            .toModel( "claude" )
    )

result = pipeline.run( { task: "sort an array" } )
```

**With structured output:**

```javascript
// Extract data → Generate content from individual fields
pipeline = aiMessage( "Extract person from: ${text}" )
    .toDefaultModel()
    .withStructuredOutput( { name: "string", age: "numeric" } )
    .pipe(
        // Struct fields are flattened to ${_input_fieldName}
        aiMessage( "Write bio for ${_input_name}, age ${_input_age}" )
            .toModel( "openai" )
    )

result = pipeline.run( { text: "John Doe is 30" } )
```

**Key points:**

* `${_input}` contains the complete previous stage output
* For struct outputs, fields are also available as `${_input_fieldName}`
* Original context variables from `.run()` are still accessible
* Stages are only connected through `_input` — they remain encapsulated

**Multi-stage example:**

```javascript
// Simplify → Translate → Formalize
pipeline = aiMessage( "Simplify: ${text}" )
    .toDefaultModel()
    .pipe(
        aiMessage( "Translate to Spanish: ${_input}" )
            .toModel( "openai" )
    )
    .pipe(
        aiMessage( "Make this more formal: ${_input}" )
            .toModel( "claude" )
    )

result = pipeline.run( { text: "The servers crashed!" } )
// Stage 1: "The servers stopped working"
// Stage 2: "Los servidores dejaron de funcionar"
// Stage 3: "Los sistemas experimentaron una interrupción del servicio"
```

### Input Types by Component

| Component       | Accepts                    | Example                             |
| --------------- | -------------------------- | ----------------------------------- |
| **AiMessage**   | Struct (bindings)          | `{ name: "Alice", role: "admin" }`  |
| **AiModel**     | Messages array / AiMessage | `[{ role: "user", content: "Hi" }]` |
| **AiTransform** | Any type                   | String, struct, array, etc.         |
| **AiAgent**     | String (user message)      | `"What's the weather?"`             |

### Output Types

The final output depends on the last step in the pipeline:

```javascript
// Model output: full AI response struct
pipeline1 = aiMessage().user( "Hi" ).toDefaultModel()
result1 = pipeline1.run()
// { content: "Hello!", model: "gpt-4", usage: {...}, ... }

// After a content transform: plain string
pipeline2 = pipeline1.transform( r => r.content )
result2 = pipeline2.run()
// "Hello!"

// After a packaging transform: custom struct
pipeline3 = pipeline2.transform( content => {
    return {
        message  : content,
        length   : content.len(),
        timestamp: now()
    }
} )
result3 = pipeline3.run()
// { message: "Hello!", length: 6, timestamp: ... }
```

---

## ⚙️ Parameters and Options

### Default Parameters

Set parameters that apply to all executions of a pipeline:

```javascript
pipeline = aiMessage()
    .user( "Explain: ${topic}" )
    .toDefaultModel()
    .withParams( {
        model    : "gpt-4o",
        temperature: 0.7,
        maxTokens: 500
    } )

// All executions use these defaults
result1 = pipeline.run( { topic: "AI" } )
result2 = pipeline.run( { topic: "ML" } )
```

### Runtime Parameter Overrides

Override defaults at execution time:

```javascript
pipeline = aiMessage()
    .user( "Write about: ${topic}" )
    .toDefaultModel()
    .withParams( { temperature: 0.7, maxTokens: 500 } )

// Creative output: override temperature at runtime
creative = pipeline.run(
    { topic: "poetry" },
    { temperature: 1.0 }   // Runtime override
)

// Factual output: different temperature
factual = pipeline.run(
    { topic: "science" },
    { temperature: 0.2 }   // Runtime override
)
```

**Merge behavior:** runtime parameters **override** defaults; unspecified parameters use defaults.

### Options vs Parameters

**Parameters** configure the AI provider (model, temperature, etc.).
**Options** configure the runnable behavior (returnFormat, timeout, logging).

```javascript
pipeline = aiMessage()
    .user( "Hello" )
    .toDefaultModel()
    .withParams( {
        model      : "gpt-4o",   // AI provider param
        temperature: 0.7         // AI provider param
    } )
    .withOptions( {
        returnFormat: "single",  // Runnable option
        timeout     : 30,        // Runnable option
        logging     : true       // Runnable option
    } )

// All three argument slots at runtime
result = pipeline.run(
    {},                         // Input bindings
    { temperature: 1.0 },       // Parameter overrides
    { returnFormat: "raw" }     // Option overrides
)
```

## Related Pages

* [Transforms](transforms.md) — Pre- and post-processing data
* [Multi-Model Workflows](multi-model.md) — Chaining models and reusable templates
* [Streaming](streaming.md) — Real-time streaming pipelines
* [Advanced](advanced.md) — Events, debugging, error handling
