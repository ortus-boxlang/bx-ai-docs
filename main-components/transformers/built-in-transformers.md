---
description: >-
  Core built-in transformers shipped with BoxLang AI: code, JSON, and XML
  extractors, text cleaning, and the transform library.
icon: toolbox
---

# Built-In Transformers

## 🧰 Core Built-In Transformers

BoxLang AI ships with several powerful built-in transformers ready to use in your pipelines:

### CodeExtractorTransformer

Extracts code blocks from AI responses, particularly useful when AI returns code embedded in markdown formatting.

**Features:**

* Extract code from markdown code blocks (` ```language ... ``` `)
* Filter by programming language (or extract all)
* Extract single or multiple code blocks
* Include metadata (language, line numbers, etc.)
* Strip comments and normalize formatting
* Strict mode for error handling

**Configuration Options:**

| Option            | Type    | Default  | Description                                              |
| ----------------- | ------- | -------- | -------------------------------------------------------- |
| `language`        | string  | `"all"`  | Filter by language (`"all"`, `"python"`, `"java"`, etc.) |
| `multiple`        | boolean | `false`  | Extract all blocks (`true`) or first only (`false`)      |
| `returnMetadata`  | boolean | `false`  | Return metadata with code or just code string            |
| `stripComments`   | boolean | `false`  | Remove comments from extracted code                      |
| `trim`            | boolean | `true`   | Trim whitespace from code blocks                         |
| `stripMarkdown`   | boolean | `true`   | Look for markdown code blocks                            |
| `strictMode`      | boolean | `false`  | Throw error if no code found                             |
| `defaultLanguage` | string  | `"text"` | Default language when not specified                      |

**Basic Usage:**

````javascript
import bxModules.bxai.models.transformers.CodeExtractorTransformer;

// Create extractor for Python code
extractor = new CodeExtractorTransformer({
    language: "python",
    stripComments: true
});

// AI response with embedded code
aiResponse = """
Here's the solution:

```python
# Calculate sum
def add_numbers(a, b):
    return a + b

result = add_numbers(5, 3)
print(result)
````

Hope this helps! """;

// Extract just the Python code code = extractor.transform( aiResponse ); // Returns: "def add\_numbers(a, b):\n return a + b\n\nresult = add\_numbers(5, 3)\nprint(result)"

````

**Pipeline Integration:**

```javascript
// Use in an AI pipeline to extract code from responses
pipeline = aiMessage()
    .system( "You are a code generator" )
    .user( "Write a Python function to calculate fibonacci" )
    .toDefaultModel()
    .transform( r => r.content )
    .to( new CodeExtractorTransformer({
        language: "python",
        stripComments: false
    }) );

pythonCode = pipeline.run();
// Returns clean Python code ready to execute
````

**Extract Multiple Blocks:**

````javascript
extractor = new CodeExtractorTransformer({
    language: "all",
    multiple: true,
    returnMetadata: true
});

multiCodeResponse = """
Python example:
```python
print("Hello")
````

JavaScript example:

```javascript
console.log("Hello");
```

""";

blocks = extractor.transform( multiCodeResponse ); // Returns: \[ // { language: "python", code: 'print("Hello")' }, // { language: "javascript", code: 'console.log("Hello");' } // ]

````

**Use Cases:**
- ✅ Extracting code from AI code generation responses
- ✅ Processing documentation with embedded examples
- ✅ Building code execution pipelines
- ✅ Cleaning AI-generated code for storage or display

---

### JSONExtractorTransformer

Extracts and validates JSON from AI responses, handling markdown formatting and mixed text content.

**Features:**
- Extract JSON from markdown code blocks
- Find JSON in mixed text (finds `{...}` or `[...]`)
- Parse and validate JSON structure
- Extract specific paths using dot notation
- Schema validation
- Strict mode for error handling

**Configuration Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `stripMarkdown` | boolean | `true` | Remove markdown code block formatting |
| `strictMode` | boolean | `false` | Throw error if JSON invalid or not found |
| `extractPath` | string | `""` | Dot notation path to extract (e.g., `"data.users"`) |
| `returnRaw` | boolean | `false` | Return raw JSON string instead of parsed |
| `validateSchema` | boolean | `false` | Validate against provided schema |
| `schema` | struct | `{}` | JSON schema for validation |

**Basic Usage:**

```javascript
import bxModules.bxai.models.transformers.JSONExtractorTransformer;

// Create extractor
extractor = new JSONExtractorTransformer({
    stripMarkdown: true
});

// AI response with JSON in markdown
aiResponse = """
Here's the data you requested:

```json
{
    "name": "John Doe",
    "age": 30,
    "email": "john@example.com"
}
````

Does this help? """;

// Extract and parse JSON data = extractor.transform( aiResponse ); // Returns: { name: "John Doe", age: 30, email: "john@example.com" }

````

**Pipeline Integration:**

```javascript
// Extract structured data from AI responses
pipeline = aiMessage()
    .system( "Return user data as JSON" )
    .user( "Get info for user ID ${userId}" )
    .toDefaultModel()
    .transform( r => r.content )
    .to( new JSONExtractorTransformer() );

userData = pipeline.run({ userId: 123 });
// Returns parsed struct, ready to use
````

**Path Extraction:**

````javascript
extractor = new JSONExtractorTransformer({
    extractPath: "data.users"
});

response = """
```json
{
    "status": "success",
    "data": {
        "users": [
            { "id": 1, "name": "Alice" },
            { "id": 2, "name": "Bob" }
        ]
    }
}
````

""";

users = extractor.transform( response ); // Returns: \[ { id: 1, name: "Alice" }, { id: 2, name: "Bob" } ]

````

**Schema Validation:**

```javascript
extractor = new JSONExtractorTransformer({
    validateSchema: true,
    schema: {
        required: ["name", "email"],
        properties: {
            name: { type: "string" },
            email: { type: "string", pattern: "^[^@]+@[^@]+\.[^@]+$" },
            age: { type: "numeric" }
        }
    }
});

// Will throw error if JSON doesn't match schema (when strictMode: true)
````

**Use Cases:**

* ✅ Extracting structured data from AI responses
* ✅ Building form auto-population from AI
* ✅ API response parsing
* ✅ Configuration generation

***

### XMLExtractorTransformer

Extracts and validates XML from AI responses, with support for XPath queries and case-sensitive parsing.

**Features:**

* Extract XML from markdown code blocks
* Find XML in mixed text (looks for `<?xml` or root tags)
* Parse and validate XML structure
* XPath queries for specific elements
* Case-sensitive or case-insensitive parsing
* Strict mode for error handling

**Configuration Options:**

| Option          | Type    | Default | Description                              |
| --------------- | ------- | ------- | ---------------------------------------- |
| `stripMarkdown` | boolean | `true`  | Remove markdown code block formatting    |
| `strictMode`    | boolean | `false` | Throw error if XML invalid or not found  |
| `xPath`         | string  | `""`    | XPath query to extract specific elements |
| `returnRaw`     | boolean | `false` | Return raw XML string instead of parsed  |
| `caseSensitive` | boolean | `true`  | Case-sensitive parsing                   |

**Basic Usage:**

````javascript
import bxModules.bxai.models.transformers.XMLExtractorTransformer;

// Create extractor
extractor = new XMLExtractorTransformer({
    stripMarkdown: true
});

// AI response with XML in markdown
aiResponse = """
Here's the config:

```xml
<?xml version="1.0"?>
<config>
    <database>
        <host>localhost</host>
        <port>5432</port>
    </database>
</config>
````

""";

// Extract and parse XML config = extractor.transform( aiResponse ); // Returns parsed XML document object

````

**Pipeline Integration:**

```javascript
// Extract XML configuration from AI
pipeline = aiMessage()
    .system( "Generate XML config" )
    .user( "Create database config for ${environment}" )
    .toDefaultModel()
    .transform( r => r.content )
    .to( new XMLExtractorTransformer() );

xmlConfig = pipeline.run({ environment: "production" });
// Returns parsed XML, ready to process
````

**XPath Queries:**

````javascript
extractor = new XMLExtractorTransformer({
    xPath: "//database/host"
});

response = """
```xml
<config>
    <database>
        <host>localhost</host>
        <port>5432</port>
    </database>
</config>
````

""";

hosts = extractor.transform( response ); // Returns array of matching nodes: \[localhost]

````

**Use Cases:**
- ✅ Extracting XML configs from AI responses
- ✅ Processing SOAP/XML API responses
- ✅ Configuration file generation
- ✅ RSS/Atom feed parsing

---

### TextCleanerTransformer

Cleans and normalizes text content by removing unwanted characters, HTML tags, and formatting.

**Features:**
- HTML tag stripping
- Extra whitespace normalization
- Line break handling
- Trimming and cleanup

**Usage:**

```javascript
import bxModules.bxai.models.transformers.TextCleanerTransformer;

// Create cleaner with options
cleaner = new TextCleanerTransformer({
    stripHTML: true,
    removeExtraSpaces: true,
    trim: true
});

// Clean text
cleanText = cleaner.transform( dirtyText );

// In a pipeline
pipeline = aiMessage()
    .user( "Process this: ${rawText}" )
    .toDefaultModel()
    .transform( r => r.content )
    .to( cleaner )
    .transform( cleaned => "Cleaned: #cleaned#" );

result = pipeline.run({ rawText: "<p>Hello   World!</p>" });
// "Cleaned: Hello World!"
````

**Configuration Options:**

```javascript
cleaner = new TextCleanerTransformer({
    trim: true,                  // Trim start/end whitespace
    removeExtraSpaces: true,     // Collapse multiple spaces
    stripHTML: true,             // Remove HTML tags
    normalizeLineBreaks: true,   // Standardize \n, \r\n, \r
    removeEmptyLines: false      // Keep or remove empty lines
});
```

### AiTransformRunnable

A wrapper class that converts any lambda function into a pipeline-compatible transformer. This is what `aiTransform()` BIF creates internally.

**Features:**

* Converts functions to IAiRunnable interface
* Fluent API support
* Pipeline integration
* Named transformers

**Usage:**

```javascript
import bxModules.bxai.models.transformers.AiTransformRunnable;

// Create transformer from function
transformer = new AiTransformRunnable(
    transformFn: ( data ) => data.ucase(),
    name: "uppercase-transformer"
);

// Use in pipeline
result = aiMessage()
    .user( "Say hello" )
    .toDefaultModel()
    .transform( r => r.content )
    .to( transformer )
    .run();

// Or use the aiTransform() BIF (recommended)
transformer = aiTransform( data => data.ucase() )
    .withName( "uppercase" );
```

**Chaining Multiple Transformers:**

```javascript
// Build transformer chain
extractContent = aiTransform( r => r.content );
cleanText = new TextCleanerTransformer({ stripHTML: true });
uppercase = aiTransform( s => s.ucase() );

pipeline = aiMessage()
    .user( "Generate text" )
    .toDefaultModel()
    .to( extractContent )
    .to( cleanText )
    .to( uppercase );

result = pipeline.run();
```
## Transform Library

````java
class {
    function extractContent() {
        return aiTransform( r => r.content )
    }

    function toUpperCase() {
        return aiTransform( s => s.ucase() )
    }

    function toLowerCase() {
        return aiTransform( s => s.lcase() )
    }

    function trim() {
        return aiTransform( s => s.trim() )
    }

    function parseJSON() {
        return aiTransform( s => deserializeJSON( s ) )
    }

    function extractCode( language = "" ) {
        return aiTransform( r => {
            pattern = "(?s).*```#language#\n(.*?)```.*"
            return r.content.reReplace( pattern, "\1", "one" ).trim()
        } )
    }

    function wordCount() {
        return aiTransform( s => s.listLen( " " ) )
    }

    function summarize( maxWords = 50 ) {
        return aiTransform( s => {
            words = s.listToArray( " " )
            if( words.len() <= maxWords ) return s
            return words.slice( 1, maxWords ).toList( " " ) & "..."
        } )
    }
}

// Usage
lib = new TransformLibrary()

pipeline = aiMessage()
    .user( "Explain AI" )
    .toDefaultModel()
    .to( lib.extractContent() )
    .to( lib.trim() )
    .to( lib.wordCount() )
````

