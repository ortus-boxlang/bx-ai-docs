---
description: >-
  Real-world transformer recipes and best practices for building
  production-ready data transformations.
icon: flask
---

# Practical Examples

## Practical Examples

### Markdown to HTML

```java
markdownTransform = aiTransform( response => {
    markdown = response.content ?: ""

    // Simple markdown parsing
    html = markdown
        .reReplace( "#{3} (.*)", "<h3>\1</h3>", "all" )
        .reReplace( "#{2} (.*)", "<h2>\1</h2>", "all" )
        .reReplace( "#{1} (.*)", "<h1>\1</h1>", "all" )
        .reReplace( "\*\*(.*?)\*\*", "<strong>\1</strong>", "all" )
        .reReplace( "\*(.*?)\*", "<em>\1</em>", "all" )

    return html
} )

pipeline = aiMessage()
    .user( "Write markdown about ${topic}" )
    .toDefaultModel()
    .to( markdownTransform )
```

### SQL Generator

````java
sqlTransform = aiTransform( response => {
    sql = response.content
        .reReplace( "(?s).*```sql\n(.*?)```.*", "\1", "one" )
        .trim()

    return {
        sql: sql,
        safe: !sql.reFindNoCase( "drop|delete|truncate" ),
        parameterized: sql.contains( "?" ) || sql.contains( ":" )
    }
} )

pipeline = aiMessage()
    .user( "Write SQL to ${task}" )
    .toDefaultModel()
    .to( sqlTransform )
````

### Response Cache

```java
class {
    property name="cache" type="struct";

    function init() {
        variables.cache = {}
        return this
    }

    function getCachedTransform() {
        return aiTransform( response => {
            key = hash( response.content )

            if( structKeyExists( variables.cache, key ) ) {
                return variables.cache[ key ]
            }

            processed = processResponse( response )
            variables.cache[ key ] = processed

            return processed
        } )
    }

    function processResponse( response ) {
        // Your processing logic
        return response.content.trim()
    }
}
```

### Multi-Format Output

```java
formatter = aiTransform( response => {
    content = response.content ?: ""

    return {
        text: content,
        html: "<p>" & content.replace( "\n", "</p><p>" ) & "</p>",
        markdown: content,
        json: serializeJSON( { content: content } ),
        length: len( content ),
        words: content.listLen( " " )
    }
} )
```

## Best Practices

1. **Keep Transforms Simple**: One responsibility per transform
2. **Handle Errors**: Use try/catch in transforms
3. **Document Logic**: Comment complex transformations
4. **Test Transforms**: Unit test transformation functions
5. **Chain Appropriately**: Logical sequence of operations
6. **Return Consistent Types**: Predictable output format
7. **Use Named Transforms**: For reusability

