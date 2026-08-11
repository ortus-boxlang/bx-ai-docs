---
description: >-
  Reusable patterns for building custom vector memory implementations, such
  as hybrid search and connection pooling.
icon: shapes
---

# Common Patterns

## Common Patterns

### Pattern 1: Wrapper Pattern

Wrap existing memory to add functionality:

```js
class extends="BaseVectorMemory" {
    property name="wrappedMemory";

    function configure( required struct config ) {
        super.configure( arguments.config );
        variables.wrappedMemory = arguments.config.wrappedMemory;
        return this;
    }

    // Add functionality before/after delegation
    function add( required any message ) {
        preProcess( arguments.message );
        variables.wrappedMemory.add( arguments.message );
        postProcess( arguments.message );
        return this;
    }
}
```

### Pattern 2: Adapter Pattern

Adapt existing clients to IVectorMemory interface:

```js
class extends="BaseVectorMemory" {
    property name="thirdPartyClient";

    function configure( required struct config ) {
        super.configure( arguments.config );
        variables.thirdPartyClient = createThirdPartyClient( arguments.config );
        return this;
    }

    function add( required any message ) {
        // Adapt to third-party API
        variables.thirdPartyClient.insertDocument(
            collection: variables.collection,
            document: adaptMessage( arguments.message )
        );
        return this;
    }
}
```

### Pattern 3: Composite Pattern

Combine multiple vector memories:

```js
class extends="BaseVectorMemory" {
    property name="memories" type="array";

    function configure( required struct config ) {
        super.configure( arguments.config );
        variables.memories = arguments.config.memories ?: [];
        return this;
    }

    function add( required any message ) {
        variables.memories.each( memory => {
            memory.add( arguments.message );
        } );
        return this;
    }

    function getRelevant( required string query, numeric limit = 5 ) {
        var allResults = [];

        variables.memories.each( memory => {
            allResults.append( memory.getRelevant( arguments.query, arguments.limit ), true );
        } );

        return mergeAndRank( allResults, arguments.limit );
    }
}
```

