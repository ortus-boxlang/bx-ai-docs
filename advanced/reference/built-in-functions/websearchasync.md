# webSearchAsync

Asynchronous variant of `webSearch()` that returns a `BoxFuture`.

## Syntax

```javascript
webSearchAsync( query, options = {} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Search query text |
| `options` | struct | No | Same options as `webSearch()` |

## Returns

A `BoxFuture` that resolves to the same normalized results array returned by `webSearch()`.

## Execution Model

`webSearchAsync()` returns a `BoxFuture` so searches can run without blocking the current request thread.

## Examples

### Basic async usage

```javascript
future = webSearchAsync( "BoxLang release notes" )
results = future.get()
```

### Promise-style chaining

```javascript
webSearchAsync( "AI agent security" )
    .then( results => {
        println( "Found #results.len()# results" )
        return results
    } )
    .catch( err => {
        println( "Search failed: #err.message ?: err#" )
    } )
```

### Parallel async searches

```javascript
braveFuture = webSearchAsync( "BoxLang AI", { provider: "brave" } )
exaFuture = webSearchAsync( "BoxLang AI", { provider: "exa", type: "neural" } )

braveResults = braveFuture.get()
exaResults = exaFuture.get()
```

## Notes

- Input options and provider behavior match `webSearch()`.
- Failures resolve as future errors; handle with `try/catch` around `.get()` or `.catch()` in chain mode.
