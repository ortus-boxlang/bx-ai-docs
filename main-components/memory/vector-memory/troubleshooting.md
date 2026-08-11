---
description: >-
  Common vector memory errors and how to resolve them, from dimension
  mismatches to connection and performance issues.
icon: wrench
---

# Troubleshooting

### Common Issues

**1. Dimension Mismatch**

```
Error: Vector dimension mismatch
```

Solution: Ensure embedding model dimensions match collection configuration

**2. Connection Errors**

```
Error: Could not connect to vector database
```

Solution: Verify host, port, and network accessibility. Check firewall rules.

**3. API Key Issues**

```
Error: Unauthorized
```

Solution: Verify API keys for both embedding provider and vector database

**4. Slow Performance**

```
Searches taking too long
```

Solution:

* Enable caching for embeddings
* Use appropriate index type (Milvus, Qdrant)
* Reduce limit parameter
* Consider smaller embedding model

**5. Out of Memory**

```
Error: OutOfMemoryException (BoxVector)
```

Solution: Switch to persistent vector database (Chroma, Postgres, etc.)

