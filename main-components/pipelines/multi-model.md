---
description: >-
  Multi-step workflows, leveraging different models' strengths, and building
  reusable pipeline templates.
icon: diagram-project
---

# Multi-Model Workflows

## 🎭 Multi-Step Workflows

Pipelines excel at orchestrating distinct phases, where each stage has a focused role.

### Draft-Refine Pattern

Use a fast model for initial drafts, then refine with a stronger model:

```javascript
// Stage 1: fast draft
drafter = aiMessage()
    .system( "Generate quick content drafts" )
    .user( "Write about: ${topic}" )
    .to( aiModel( "openai", { model: "gpt-4o-mini" } ) )
    .transform( r => r.content )

// Stage 2: quality refinement
refiner = aiMessage()
    .system( "Improve and expand content while maintaining the core message" )
    .user( "Enhance this draft: ${draft}" )
    .to( aiModel( "openai", { model: "gpt-4o" } ) )
    .transform( r => r.content )

topic  = "AI in healthcare"
draft  = drafter.run( { topic: topic } )
refined = refiner.run( { draft: draft } )
```

**Benefits:**

* Faster initial generation (cheap model)
* Higher quality final output (stronger model)
* Cost optimization — only use the expensive model for refinement

### Analysis-Enhancement Pattern

Analyze content first, then generate a tailored response:

```javascript
analyzer = aiMessage()
    .user( "Analyze sentiment: ${text}" )
    .to( aiModel( "openai", { model: "gpt-4o-mini" } ) )
    .transform( r => r.content )

responder = aiMessage()
    .system( "Generate a response matching the sentiment: ${sentiment}" )
    .user( "Respond to: ${original}" )
    .toDefaultModel()
    .transform( r => r.content )

review    = "This product is amazing! Best purchase ever."
sentiment = analyzer.run( { text: review } )
response  = responder.run( { sentiment: sentiment, original: review } )
```

### Validation Pipeline

Generate, validate, then retry if needed:

```javascript
generator = aiMessage()
    .user( "Generate a valid email for ${name}" )
    .toDefaultModel()
    .transform( r => r.content )

validator = aiTransform( email => {
    isValid = email.reFind( "^[\w\.\-]+@[\w\.\-]+\.\w+$" ) > 0
    return {
        email  : email,
        valid  : isValid,
        message: isValid ? "Valid" : "Invalid email format"
    }
} )

pipeline   = generator.to( validator )
maxRetries = 3

for ( i = 1; i <= maxRetries; i++ ) {
    result = pipeline.run( { name: "John Doe" } )
    if ( result.valid ) {
        println( "Generated valid email: #result.email#" )
        break
    }
    println( "Attempt #i# failed: #result.message#" )
}
```

---

## 🔀 Multi-Model Pipelines

### Model Specialization

Use each model for what it does best:

```javascript
quickModel = aiModel( "openai",   { model: "gpt-4o-mini",          temperature: 0.7 } )
smartModel = aiModel( "openai",   { model: "gpt-4o",               temperature: 0.3 } )
codeModel  = aiModel( "deepseek", { model: "deepseek-coder",       temperature: 0.2 } )

// Quick summary → deep analysis → code generation
workflow = aiMessage()
    .user( "Summarize: ${doc}" )
    .to( quickModel )
    .transform( r => r.content )
    .transform( summary => {
        return aiMessage()
            .user( "Analyze in detail: ${summary}" )
            .to( smartModel )
            .run( { summary: summary } ).content
    } )
    .transform( analysis => {
        return aiMessage()
            .user( "Generate code based on: ${analysis}" )
            .to( codeModel )
            .run( { analysis: analysis } ).content
    } )
```

### Dynamic Model Selection

Choose the model at runtime based on task characteristics:

```javascript
function createAdaptivePipeline( complexity ) {
    model = complexity > 5
        ? aiModel( "openai", { model: "gpt-4o" } )
        : aiModel( "openai", { model: "gpt-4o-mini" } )

    return aiMessage()
        .user( "Process: ${input}" )
        .to( model )
        .transform( r => r.content )
}

simplePipeline  = createAdaptivePipeline( 3 )
complexPipeline = createAdaptivePipeline( 8 )
```

---

## ♻️ Reusable Templates

One of the most powerful features of pipelines is **reusability** — define once, execute with different inputs.

### Parameterized Pipelines

Create templates that accept different inputs on every run:

```javascript
greeter = aiMessage()
    .system( "You are a friendly greeter" )
    .user( "Greet ${name} in ${style} style" )
    .toDefaultModel()
    .transform( r => r.content )

// Each execution is independent
formal = greeter.run( { name: "Dr. Smith",    style: "formal" } )
casual = greeter.run( { name: "Bob",          style: "casual" } )
pirate = greeter.run( { name: "Captain Jack", style: "pirate" } )

println( formal )  // "Good day, Dr. Smith. How may I assist you?"
println( casual )  // "Hey Bob! What's up?"
println( pirate )  // "Ahoy, Captain Jack! What brings ye here?"
```

### Pipeline Factories

Generate customized pipelines on demand:

```javascript
function createTranslator( required string targetLanguage ) {
    return aiMessage()
        .system( "You are a professional translator" )
        .user( "Translate to #targetLanguage#: ${text}" )
        .toDefaultModel()
        .transform( r => r.content )
}

spanishTranslator = createTranslator( "Spanish" )
frenchTranslator  = createTranslator( "French" )
germanTranslator  = createTranslator( "German" )

text = "Hello, how are you?"
println( spanishTranslator.run( { text: text } ) )  // "Hola, ¿cómo estás?"
println( frenchTranslator.run(  { text: text } ) )  // "Bonjour, comment allez-vous?"
println( germanTranslator.run(  { text: text } ) )  // "Hallo, wie geht es dir?"
```

### Composable Building Blocks

Build complex pipelines from small, reusable components:

```javascript
// Reusable components
contentExtractor = aiTransform( r    => r.content )
uppercaser       = aiTransform( text => text.uCase() )
trimmer          = aiTransform( text => text.trim() )
wordCounter      = aiTransform( text => {
    return { text: text, words: text.split( " " ).len() }
} )

// Compose different variations
basicPipeline     = aiMessage().user( "Say hello" ).toDefaultModel().to( contentExtractor )
formattedPipeline = basicPipeline.to( trimmer ).to( uppercaser )
analysisPipeline  = basicPipeline.to( wordCounter )

result1 = basicPipeline.run()      // "Hello there!"
result2 = formattedPipeline.run()  // "HELLO THERE!"
result3 = analysisPipeline.run()   // { text: "Hello there!", words: 2 }
```

## Related Pages

* [Building Pipelines](building.md) — Pipeline construction and data flow
* [Transforms](transforms.md) — Pre- and post-processing data
* [Streaming](streaming.md) — Real-time streaming with pipelines
* [Advanced](advanced.md) — Events, debugging, error handling
