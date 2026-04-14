# aiParallel

Create a parallel runnable that fans-out a single input to multiple named runnables concurrently and collects results into a named struct.

## Syntax

```javascript
aiParallel( runnables )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `runnables` | struct | ✅ Yes | Named struct where each key becomes a result key and each value is an `IAiRunnable` (model, agent, pipeline, etc.) |

## Returns

An **`AiRunnableParallel`** instance that implements `IAiRunnable`. It can be executed directly or composed into larger pipelines.

### Calling `.run( input )`

```javascript
results = aiParallel( runnables ).run( input )
```

Returns a **struct** where each key matches a `runnables` key and each value is that runnable's output for the given input. All runnables execute concurrently using the `io-tasks` virtual thread pool.

| Method | Description |
|---|---|
| `run( input, params, options )` | Execute all runnables in parallel; returns named result struct |
| `runAsync( input, params, options )` | Non-blocking parallel execution; returns `BoxFuture<struct>` |
| `stream( callback, input, params )` | Not applicable — use `run()` for parallel execution |
| `transform( closure )` | Chain a transformation after parallel execution (fluent) |
| `to( runnable )` | Chain another runnable after this one (fluent) |

## Examples

### Fan-out to multiple models

```javascript
parallel = aiParallel({
    openai:  aiModel( "openai",  { params: { model: "gpt-4o-mini" } } ),
    claude:  aiModel( "claude",  { params: { model: "claude-3-haiku-20240307" } } ),
    mistral: aiModel( "mistral", { params: { model: "mistral-small-latest" } } )
})

results = parallel.run( "What is the capital of France?" )

println( results.openai  )  // "Paris"
println( results.claude  )  // "Paris"
println( results.mistral )  // "The capital of France is Paris."
```

### Async parallel execution

```javascript
parallel = aiParallel({
    fast:   aiModel( "groq" ),
    smart:  aiModel( "openai", { params: { model: "gpt-4o" } } )
})

future = parallel.runAsync( "Summarize quantum computing in one sentence" )

// Do other work while models run concurrently...

results = future.get()
println( "Fast:  #results.fast#" )
println( "Smart: #results.smart#" )
```

### Chain a transform after parallel results

Because `AiRunnableParallel` implements `IAiRunnable`, it composes naturally into pipelines:

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

### Compose into a larger pipeline

```javascript
summarizer = aiModel( "openai", { params: { temperature: 0 } } )

pipeline = aiParallel({
    en: aiModel( "openai", { params: { model: "gpt-4o-mini" } } ),
    es: aiModel( "mistral" )
}).to( summarizer )

// summarizer receives: { en: "...", es: "..." }
summary = pipeline.run( "Describe BoxLang AI in one paragraph" )
```

### Side-by-side model evaluation

```javascript
prompt = "Write a haiku about programming"

evaluations = aiParallel({
    openai:    aiModel( "openai"  ),
    claude:    aiModel( "claude"  ),
    gemini:    aiModel( "gemini"  ),
    mistral:   aiModel( "mistral" )
}).run( prompt )

// Display all haikus
evaluations.each( function( modelName, haiku ) {
    println( "=== #modelName# ===" )
    println( haiku )
    println()
})
```

## Notes

- All runnables execute concurrently using virtual threads (`io-tasks` executor)
- The result struct preserves the same keys as the input `runnables` struct
- An empty `runnables` struct returns an empty result struct (no error)
- Result order in the struct reflects insertion order, not completion order

## Events Fired

`aiParallel()` itself does not fire additional events. Each runnable within the parallel group fires its own events (`beforeAIModelInvoke`, `afterAIModelInvoke`, etc.) independently.

## See Also

* [Parallel Pipelines Guide](../../../main-components/pipelines/advanced.md)
* [Pipeline Building](../../../main-components/pipelines/building.md)
* [aiModel](aimodel.md)
* [aiAgent](aiagent.md)
