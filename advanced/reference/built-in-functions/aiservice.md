---
description: Get a reference to a registered AI service provider — the direct invocation interface for AI providers.
icon: gear
---

# aiService

Get a reference to a registered AI service provider. This is the direct invocation interface for AI providers, as opposed to `aiModel()` which creates pipeline-friendly runnables.

## Syntax

```javascript
aiService(provider, options)
```

## Parameters

| Parameter  | Type          | Required | Default      | Description                                                                                                                                         |
| ---------- | ------------- | -------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | string        | No       | (config)     | The provider to use. If not provided, uses default from module configuration                                                                        |
| `options`  | string/struct | No       | (config/env) | Configuration options or simple API key string. If string, treated as API key. If struct, can include: `apiKey`, `baseURL`, `timeout`, `headers`, `logRequest`, `logResponse`, `maxRetries`, etc. If not provided, uses configuration or `<PROVIDER>_API_KEY` environment variable |

### Supported Providers

* **openai** - OpenAI (GPT models)
* **claude** - Anthropic Claude
* **gemini** - Google Gemini
* **ollama** - Ollama (local models)
* **groq** - Groq
* **grok** - xAI Grok
* **deepseek** - DeepSeek
* **mistral** - Mistral AI
* **cohere** - Cohere
* **huggingface** - Hugging Face
* **openrouter** - OpenRouter
* **perplexity** - Perplexity
* **voyage** - Voyage AI (embeddings)

## Returns

Returns an AI service provider instance (e.g., `OpenAIService`, `ClaudeService`) with methods:

* `invoke(request)` - Send synchronous request
* `invokeStream(request, callback)` - Stream response
* `configure(options)` - Configure the service with API key or options struct
* `getName()` - Get provider name
* `embed(request)` - Generate embeddings (if supported)

## Examples

### Get Default Service

```javascript
// Use default provider from config
service = aiService();

// Create request and invoke
request = aiChatRequest( "What is BoxLang?" );
response = service.invoke( request );
```

### Specific Provider

```javascript
// Get OpenAI service
openai = aiService( "openai" );

// Get Claude service
claude = aiService( "claude" );

// Get Ollama (local)
ollama = aiService( "ollama" );
```

### With API Key Override

```javascript
// Override API key (simple string - backward compatible)
service = aiService( "openai", "sk-custom-key-123" );

response = service.invoke( aiChatRequest( "Hello" ) );
```

### With Configuration Options (v2.1.0+)

```javascript
// Pass configuration struct with API key
service = aiService( "openai", {
    apiKey: "sk-custom-key-123"
});

// Custom base URL for OpenAI-compatible services
service = aiService( "ollama", {
    baseURL: "http://my-ollama-server:11434"
});

// Advanced configuration options
service = aiService( "openai", {
    apiKey: "sk-custom-key-123",
    timeout: 120,
    headers: {
        "X-Custom-Header": "value"
    },
    logRequest: true,
    logResponse: true,
    maxRetries: 3
});
```

### Direct Invocation

```javascript
// Create service and invoke directly
service = aiService( "claude" );

request = aiChatRequest()
    .addSystemMessage( "You are a helpful assistant" )
    .addUserMessage( "Explain AI" )
    .setParams({ temperature: 0.7 });

response = service.invoke( request );
println( response.content );
```

### Streaming Response

```javascript
// Stream from service
service = aiService( "openai" );

request = aiChatRequest( "Tell a story about dragons" );

service.invokeStream(
    request,
    ( chunk ) => writeOutput( chunk )
);
```

### Multiple Providers

```javascript
// Compare responses from different providers
question = aiChatRequest( "What is the speed of light?" );

openAI = aiService( "openai" );
claude = aiService( "claude" );
ollama = aiService( "ollama" );

results = {
    openai: openAI.invoke( question ),
    claude: claude.invoke( question ),
    ollama: ollama.invoke( question )
};

// Compare responses
results.each( ( provider, response ) => {
    println( "#provider#: #response.content#" );
});
```

### Environment Variable Detection

```javascript
// Auto-detects OPENAI_API_KEY from environment
service = aiService( "openai" ); // No API key needed

// Auto-detects CLAUDE_API_KEY
service = aiService( "claude" );

// Custom key still works
service = aiService( "openai", "sk-override-key" );
```

### Service Configuration

```javascript
// Get service and configure
service = aiService( "openai" );

// Check service info
println( "Provider: #service.getName()#" );

// Reconfigure if needed (simple API key)
service.configure( "new-api-key" );

// Or reconfigure with options struct
service.configure({
    apiKey: "new-api-key",
    timeout: 90,
    logResponse: true
});
```

### Reusable Services

```javascript
// Create services once, use many times
openaiService = aiService( "openai" );
claudeService = aiService( "claude" );

// Use in loop
questions = ["What is AI?", "What is ML?", "What is DL?"];

openaiAnswers = questions.map( q => {
    return openaiService.invoke( aiChatRequest( q ) ).content;
});

claudeAnswers = questions.map( q => {
    return claudeService.invoke( aiChatRequest( q ) ).content;
});
```

### With Custom Options

```javascript
// Build request with options
service = aiService( "openai" );

request = aiChatRequest( "Hello" )
    .setParams({
        model: "gpt-4",
        temperature: 0.9,
        max_tokens: 500
    })
    .setOptions({
        timeout: 60,
        logResponse: true
    });

response = service.invoke( request );
```

### Error Handling

```javascript
// Handle service errors
service = aiService( "openai" );

try {
    response = service.invoke( aiChatRequest( "Test" ) );
} catch ( any e ) {
    println( "Service error: #e.message#" );
    // Fallback to different provider
    service = aiService( "claude" );
    response = service.invoke( aiChatRequest( "Test" ) );
}
```

### Tool-Augmented Generation

```javascript
// Use service with tools
service = aiService( "openai" );

tools = [
    aiTool( "getWeather", "Get weather", ( city ) => weatherAPI.get( city ) ),
    aiTool( "getTime", "Get time", () => now() )
];

request = aiChatRequest( "What's the weather in Paris?" )
    .setTools( tools );

response = service.invoke( request );
```

### Embedding Service

```javascript
// Get embedding service
service = aiService( "openai" );

// Create embedding request
embedding = aiEmbed( "Hello World" )
    .setParams({ model: "text-embedding-3-small" });

// Generate embeddings
response = service.embed( request );
println( response.data.first().embedding );
```

### Provider Comparison Utility

```javascript
// Utility to compare providers
function compareProviders( prompt, providers ) {
    return providers.reduce( ( results, provider ) => {
        service = aiService( provider );
        response = service.invoke( aiChatRequest( prompt ) );
        results[ provider ] = response.content;
        return results;
    }, {} );
}

results = compareProviders(
    "Explain quantum computing",
    ["openai", "claude", "ollama"]
);
```

### Service Factory Pattern

```javascript
// Factory for different use cases
function getServiceForTask( task ) {
    switch( task ) {
        case "creative":
            return aiService( "claude" ); // Better for creative
        case "code":
            return aiService( "openai" ); // Better for code
        case "local":
            return aiService( "ollama" ); // Free local
        default:
            return aiService(); // Default
    }
}

service = getServiceForTask( "creative" );
response = service.invoke( aiChatRequest( "Write a poem" ) );
```

### Long-Running Service

```javascript
// Service for long conversations
service = aiService( "openai" );
conversation = aiChatRequest()
    .addSystemMessage( "You are a helpful assistant" );

// Multi-turn interaction
while ( !done ) {
    userInput = getUserInput();
    conversation.addUserMessage( userInput );

    response = service.invoke( conversation );
    conversation.addAssistantMessage( response.content );

    println( response.content );
}
```

### Provider Detection

```javascript
// Check available providers
supportedProviders = [
    "openai", "claude", "gemini", "ollama",
    "groq", "grok", "deepseek", "mistral",
    "cohere", "huggingface", "perplexity", "voyage"
];

println( "Available AI Providers:" );
supportedProviders.each( provider => {
    println( "  - #provider#" );
});
```

## Notes

* 🔑 **Auto-Detection**: Automatically detects `<PROVIDER>_API_KEY` environment variables
* 🎯 **Direct Control**: Use for direct service invocation (not pipelines)
* 🔄 **Reusability**: Create service once, invoke multiple times
* 📦 **Provider Agnostic**: Same interface across all providers
* 🔧 **Configuration**: Supports runtime API key override
* 🚀 **Events**: Fires `onAIProviderCreate` event for interceptors
* 💡 **vs aiModel()**: Use `aiService()` for direct calls, `aiModel()` for pipelines

## Related Functions

* [`aiModel()`](aimodel.md) - Create model runnables for pipelines
* [`aiChat()`](aichat.md) - Simple synchronous chat interface
* [`aiChatAsync()`](aichatasync.md) - Asynchronous chat interface
* [`aiChatStream()`](aichatstream.md) - Streaming chat interface
* [`aiChatRequest()`](aichatrequest.md) - Build request objects

## Best Practices

✅ **Reuse services** - Create once, invoke multiple times for efficiency

✅ **Use environment variables** - Set `<PROVIDER>_API_KEY` for automatic detection

✅ **Handle errors** - Wrap invocations in try/catch for robustness

✅ **Choose right provider** - Select based on task requirements (creative, code, local)

✅ **Configure once** - Set API keys at service creation, not per request

❌ **Don't create per request** - Services are reusable, create once

❌ **Don't use in pipelines** - Use `aiModel()` instead for pipeline integration

❌ **Don't hardcode keys** - Use environment variables or secure configuration
