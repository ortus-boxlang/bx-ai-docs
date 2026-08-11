---
description: BoxLang AI aiWebSearchAsync() built-in function reference
icon: magnifying-glass
---

# aiWebSearchAsync

Asynchronous variant of `aiWebSearch()` that returns a `BoxFuture`.

## Syntax

```javascript
aiWebSearchAsync( query, options = {} )
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Search query text |
| `options` | struct | No | Same options as `aiWebSearch()` |

## Returns

A `BoxFuture` that resolves to the same normalized results array returned by `aiWebSearch()`.

## Execution Model

`aiWebSearchAsync()` returns a `BoxFuture` so searches can run without blocking the current request thread.

## Examples

### Basic async usage

```javascript
future = aiWebSearchAsync( "BoxLang release notes" )
results = future.get()
```

### Promise-style chaining

```javascript
aiWebSearchAsync( "AI agent security" )
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
braveFuture = aiWebSearchAsync( "BoxLang AI", { provider: "brave" } )
exaFuture = aiWebSearchAsync( "BoxLang AI", { provider: "exa", type: "neural" } )

braveResults = braveFuture.get()
exaResults = exaFuture.get()
```

## Notes

- Input options and provider behavior match `aiWebSearch()`.
- Failures resolve as future errors; handle with `try/catch` around `.get()` or `.catch()` in chain mode.
