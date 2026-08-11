---
description: >-
  Pipeline events, debugging tools, performance optimization, error handling,
  and best practices for production use.
icon: screwdriver-wrench
---

# Advanced Pipeline Features

## 🎬 Pipeline Events

Pipelines emit events during execution that you can intercept for monitoring, auditing, and debugging.

### Available Events

| Event                 | When                      | Data                                                                 |
| --------------------- | ------------------------- | -------------------------------------------------------------------- |
| `beforeAIPipelineRun` | Before pipeline execution | `{ sequence, name, stepCount, steps, input, params, options }`       |
| `afterAIPipelineRun`  | After pipeline execution  | `{ sequence, name, stepCount, steps, input, result, executionTime }` |

### Event Interception

```javascript
BoxRegisterInterceptor( {
    interceptorObject: {
        beforeAIPipelineRun: ( event, interceptData ) => {
            println( "Pipeline starting: #interceptData.name#" )
            println( "Steps: #interceptData.stepCount#" )
        },
        afterAIPipelineRun: ( event, interceptData ) => {
            println( "Pipeline completed: #interceptData.name#" )
            println( "Time: #interceptData.executionTime#ms" )
        }
    }
} )

pipeline = aiMessage().user( "Hello" ).toDefaultModel()
result   = pipeline.run()
// Console output:
// Pipeline starting: AiRunnableSequence
// Steps: 2
// Pipeline completed: AiRunnableSequence
// Time: 1543ms
```

**Use cases:** performance monitoring, cost tracking (count tokens), error auditing, security logging.

---

## 🐛 Debugging Pipelines

### Print Pipeline Structure

Use `.print()` to see exactly what steps are in a pipeline:

```javascript
pipeline = aiMessage()
    .user( "Hello" )
    .toDefaultModel()
    .transform( r => r.content )
    .transform( t => t.uCase() )

pipeline.print()
// Output:
// AiRunnableSequence (4 steps)
//   [1] AiMessage (bxModules.bxai.models.AiMessage)
//   [2] AiModel   (bxModules.bxai.models.AiModel)
//   [3] Transform (bxModules.bxai.models.transformers.AiTransformRunnable)
//   [4] Transform (bxModules.bxai.models.transformers.AiTransformRunnable)
```

### Inspect Steps

Get detailed information about each step:

```javascript
steps = pipeline.getSteps()

for ( step in steps ) {
    println( "Step #step.index#:" )
    println( "  Name  : #step.name#" )
    println( "  Type  : #step.type#" )
    println( "  Params: #jsonSerialize( step.params )#" )
}
```

### Step-by-Step Execution

Execute each step manually to isolate problems:

```javascript
steps = pipeline.getSteps()
input = { name: "Alice" }

for ( step in steps ) {
    println( "Executing: #step.name#" )
    println( "Input    : #jsonSerialize( input )#" )

    input = step.step.run( input )

    println( "Output   : #jsonSerialize( input )#" )
    println( "─".repeat( 50 ) )
}
```

---

## ⚡ Performance Optimization

### Choose the Right Model

Use cheaper/faster models for simple tasks — reserve powerful models for complex reasoning:

```javascript
// Fast model for simple extraction
extractor = aiMessage()
    .user( "Extract email from: ${text}" )
    .to( aiModel( "openai", { model: "gpt-4o-mini" } ) )
    .transform( r => r.content )

// Smart model for complex reasoning
reasoner = aiMessage()
    .user( "Explain quantum computing: ${topic}" )
    .to( aiModel( "openai", { model: "gpt-4o" } ) )
    .transform( r => r.content )
```

### Minimize Transform Steps

Combine transformations where possible:

```javascript
// ❌ Four separate transforms
pipeline = aiMessage().user( "Hello" ).toDefaultModel()
    .transform( r => r.content )
    .transform( t => t.trim() )
    .transform( t => t.uCase() )
    .transform( t => "RESULT: " & t )

// ✅ Single combined transform
pipeline = aiMessage().user( "Hello" ).toDefaultModel()
    .transform( r => "RESULT: " & r.content.trim().uCase() )
```

### Cache Expensive Results

Avoid repeated AI calls for the same input:

```javascript
cache = {}

function getCachedResult( required any pipeline, required any input ) {
    key = hash( jsonSerialize( input ) )

    if ( cache.keyExists( key ) ) {
        return cache[ key ]
    }

    result = pipeline.run( input )
    cache[ key ] = result
    return result
}

result = getCachedResult( expensivePipeline, { topic: "AI" } )
result = getCachedResult( expensivePipeline, { topic: "AI" } )  // Returns from cache
```

---

## 🔒 Error Handling

### Try-Catch

Wrap pipeline execution to handle AI failures gracefully:

```javascript
try {
    result = pipeline.run( input )
} catch ( any e ) {
    println( "Pipeline error: #e.message#" )
    println( "Type         : #e.type#" )
    result = getDefaultResponse()
}
```

### Graceful Degradation

Provide rule-based fallbacks when AI is unavailable:

```javascript
function processWithFallback( required any input ) {
    try {
        return aiPipeline.run( input )
    } catch ( any e ) {
        println( "AI failed, using rule-based fallback" )
        return ruleBasedProcessor( input )
    }
}
```

### Validation Steps

Validate at every critical boundary:

```javascript
pipeline = aiMessage()
    .user( "Generate JSON: ${prompt}" )
    .toDefaultModel()
    .transform( r => r.content )
    .transform( content => {
        try {
            return jsonDeserialize( content )
        } catch ( any e ) {
            throw( type: "ValidationError", message: "Invalid JSON response" )
        }
    } )
    .transform( data => {
        if ( !data.keyExists( "result" ) ) {
            throw( type: "ValidationError", message: "Missing 'result' key" )
        }
        return data
    } )
```

---

## 📚 Best Practices

### Design Principles

✅ **Single Responsibility** — each step does one thing well
✅ **Immutability** — never modify pipeline state during execution
✅ **Composition** — build complex workflows from simple components
✅ **Reusability** — design pipelines as parameterized templates
✅ **Explicit > Implicit** — be clear about each data transformation

### Common Patterns

```javascript
// Extract-Transform-Load (ETL)
etlPipeline = dataLoader
    .to( dataTransformer )
    .to( dataSaver )

// Template-Execute-Format
workflow = messageTemplate
    .toDefaultModel()
    .transform( r => formatOutput( r ) )

// Validate-Process-Validate
safePipeline = inputValidator
    .to( aiProcessor )
    .to( outputValidator )
```

### Anti-Patterns to Avoid

❌ **Overly long pipelines** (>10 steps) — break into named sub-pipelines
❌ **Side effects in transforms** — keep transforms pure (no DB writes, no external calls)
❌ **Tight coupling** — don't hardcode provider-specific logic inside transforms
❌ **Missing error handling** — always handle AI failures gracefully
❌ **Ignoring performance** — profile expensive operations before deploying

---

## ⚡ Async Pipeline Execution

Every `IAiRunnable` — including full pipeline sequences — exposes `runAsync()`, which dispatches execution to the `io-tasks` virtual thread pool and returns a **`BoxFuture`**. This lets you kick off expensive AI calls without blocking the current thread.

```javascript
pipeline = aiModel( "openai" )
    .transform( text => text.trim() )
    .transform( text => "Summary: #text#" )

// Non-blocking — returns immediately with a future
future = pipeline.runAsync( "Explain quantum computing in one paragraph" )

// Do other work here...

// Block when you actually need the result
result = future.get()
println( result )
```

You can also use `.then()` for a callback pattern:

```javascript
pipeline.runAsync( "What is BoxLang?" ).then( function( result ) {
    println( "Completed: #result#" )
})
```

### Running Multiple Pipelines Concurrently

The real power of `runAsync()` is running multiple independent pipelines at the same time:

```javascript
pipelines = [
    aiModel( "openai" ).runAsync( "Summarize topic A" ),
    aiModel( "openai" ).runAsync( "Summarize topic B" ),
    aiModel( "openai" ).runAsync( "Summarize topic C" )
]

// All three run concurrently; collect when needed
summaries = pipelines.map( f => f.get() )
summaries.each( s => println( s ) )
```

---

## 🔀 Parallel Pipelines with `aiParallel()`

`aiParallel()` is purpose-built for fan-out scenarios: send the same input to multiple named runnables **concurrently** and receive all results in a single named struct.

```javascript
parallel = aiParallel({
    openai:   aiModel( "openai",  { params: { model: "gpt-4o-mini" } } ),
    claude:   aiModel( "claude",  { params: { model: "claude-3-haiku-20240307" } } ),
    mistral:  aiModel( "mistral", { params: { model: "mistral-small-latest" } } )
})

results = parallel.run( "What is the capital of France?" )

println( results.openai  )  // "Paris"
println( results.claude  )  // "Paris"
println( results.mistral )  // "The capital of France is Paris."
```

Because `AiRunnableParallel` implements `IAiRunnable`, it composes naturally into larger pipelines with `.transform()` or `.to()`:

```javascript
pipeline = aiParallel({
    fast:  aiModel( "groq" ),
    smart: aiModel( "openai" )
}).transform( function( results ) {
    return "Fast:  #results.fast##chr(10)#Smart: #results.smart#"
})

combined = pipeline.run( "Explain quantum entanglement in one sentence" )
println( combined )
```

### Model Evaluation / A/B Testing

`aiParallel()` makes comparing outputs across providers or prompt variants trivial:

```javascript
evaluations = aiParallel({
    control:   aiModel( "openai",  { params: { temperature: 0.5 } } ),
    variant_a: aiModel( "openai",  { params: { temperature: 1.0 } } ),
    variant_b: aiModel( "claude" )
}).run( "Write a tagline for a cloud-native BoxLang platform" )

evaluations.each( function( variant, text ) {
    println( "=== #variant# ===" )
    println( text )
})
```

---

## Related Pages

* [Building Pipelines](building.md) — Methods and data flow
* [Transforms](transforms.md) — Pre- and post-processing
* [Multi-Model Workflows](multi-model.md) — Complex multi-stage patterns
* [Streaming](streaming.md) — Real-time response handling
* [Events Reference](../../advanced/events/README.md) — Full event catalog
* [aiParallel BIF Reference](../../advanced/reference/built-in-functions/aiparallel.md)
