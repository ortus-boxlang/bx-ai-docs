---
description: >-
  Validating external data sources and web search results before they reach
  the model or your users.
icon: database
---

# Data Validation

## External Data Source Validation

### Web Search Result Validation

**Web search results come from untrusted sources**. Always validate before using:

```javascript
class {
    function safeWebSearch( required string query ) {
        // 1. Validate user query
        var sanitizedQuery = sanitizeSearchQuery( arguments.query )

        // 2. Execute search
        var rawResults = aiWebSearch( sanitizedQuery )

        // 3. Validate each result
        var validatedResults = rawResults.map( result => {
            return {
                title: stripHTML( result.title ),
                url: validateURL( result.url ),
                snippet: redactPII( stripInjectionPatterns( result.snippet ) ),
                domain: extractDomain( result.url ),
                score: result.score,
                thumbnail: isURL( result.thumbnail ) ? result.thumbnail : "",
                publishedDate: parseDate( result.publishedDate )
            }
        } )

        // 4. Filter suspicious results
        return filterSuspiciousResults( validatedResults )
    }

    function validateURL( required string url ) {
        // Block known malicious domains
        var blockedDomains = [ "bit.ly", "tinyurl.com" ]  // Short URLs hide true destination
        var domain = extractDomain( arguments.url )

        if ( blockedDomains.contains( domain ) ) {
            throw "Blocked URL domain: #domain#"
        }

        // Validate URL format
        if ( !isValid( "url", arguments.url ) ) {
            throw "Invalid URL format: #arguments.url#"
        }

        return arguments.url
    }

    function stripHTML( required string text ) {
        // Remove script tags and event handlers
        var cleaned = arguments.text
        cleaned = reReplace( cleaned, "<script[^>]*>.*?</script>", "", "all" )
        cleaned = reReplace( cleaned, " on\w+\s*=", " ", "all" )  // Remove event handlers
        cleaned = htmlEditFormat( cleaned )  // HTML escape

        return cleaned
    }

    function stripInjectionPatterns( required string text ) {
        var patterns = [
            "ignore previous",
            "disregard all",
            "system:\s*",
            "new instructions:",
            "you are now"
        ]

        var cleaned = arguments.text
        for ( pattern in patterns ) {
            cleaned = reReplaceNoCase( cleaned, pattern, "[REDACTED]", "all" )
        }

        return cleaned
    }

    function filterSuspiciousResults( required array results ) {
        return results.filter( r => {
            // Filter out results with suspicious snippet content
            var suspiciousKeywords = [ "viagra", "casino", "loan", "click here" ]
            var snippet = r.snippet.toLowerCase()

            for ( keyword in suspiciousKeywords ) {
                if ( snippet.find( keyword ) > 0 ) {
                    return false
                }
            }

            return true
        } )
    }
}
```

### Document Loader Input Validation

**Loading documents from untrusted sources can introduce malicious content**:

```javascript
class {
    function safeLoadDocuments( required string filePath ) {
        // 1. Validate file path (prevent directory traversal)
        validateFilePath( arguments.filePath )

        // 2. Check file size (prevent memory exhaustion)
        var fileSize = getFileSize( arguments.filePath )
        if ( fileSize > 50_000_000 ) {  // 50 MB limit
            throw "File too large: #fileSize# bytes"
        }

        // 3. Check file type (whitelist allowed extensions)
        var allowedExtensions = [ "pdf", "txt", "md", "csv" ]
        var fileExt = listLast( arguments.filePath, "." ).toLowerCase()

        if ( !allowedExtensions.contains( fileExt ) ) {
            throw "File type not allowed: .#fileExt#"
        }

        // 4. Load documents with safety checks
        var loader = aiDocuments( fileExt, arguments.filePath )
        var documents = loader.load()

        // 5. Scan content for malicious patterns
        return documents.map( doc => {
            return {
                id: doc.id,
                content: sanitizeDocumentContent( doc.content ),
                metadata: doc.metadata
            }
        } )
    }

    function validateFilePath( required string filePath ) {
        // Prevent directory traversal attacks
        if ( arguments.filePath.find( ".." ) > 0 ) {
            throw "Invalid file path: directory traversal detected"
        }

        // Ensure absolute path is within allowed directory
        var allowedDir = expandPath( "/documents" )
        var absolutePath = expandPath( arguments.filePath )

        if ( !absolutePath.startsWith( allowedDir ) ) {
            throw "File path outside allowed directory: #arguments.filePath#"
        }
    }

    function sanitizeDocumentContent( required string content ) {
        // Remove embedded scripts
        var cleaned = content
        cleaned = reReplace( cleaned, "<script[^>]*>.*?</script>", "", "all" )

        // Limit content length
        if ( len( cleaned ) > 100_000 ) {
            cleaned = left( cleaned, 100_000 )
        }

        return cleaned
    }
}
```

### Vector Memory Poisoning Prevention

**Adversaries can pollute vector databases with malicious embeddings**:

```javascript
class {
    function safeVectorMemoryAdd(
        required string text,
        required struct metadata,
        required string userId
    ) {
        // 1. Validate text content
        var sanitized = sanitizeEmbeddingText( arguments.text )

        // 2. Validate metadata doesn't contain injection
        var safeMetadata = validateMetadata( arguments.metadata )

        // 3. Check for semantic anomalies (if embeddings are suspiciously similar to system prompts)
        var embedding = generateEmbedding( sanitized )
        if ( detectAnomalousEmbedding( embedding ) ) {
            writeLog( "Anomalous embedding detected for user #arguments.userId#", "warning" )
            return false
        }

        // 4. Add to vector memory with user isolation
        var vectorMemory = aiMemory( memory: "boxvector", config: {
            userId: arguments.userId
        } )

        vectorMemory.add( sanitized, safeMetadata )

        return true
    }

    function detectAnomalousEmbedding( required array embedding ) {
        // Compare against known attack embeddings or system prompt embeddings
        var systemPromptEmbedding = generateEmbedding( "You are a helpful assistant" )

        // Calculate cosine similarity
        var similarity = cosineSimilarity( arguments.embedding, systemPromptEmbedding )

        // Flag if too similar to system prompt (indicates injection attempt)
        if ( similarity > 0.9 ) {
            return true
        }

        return false
    }
}
```

***

## Web Search Specific Security

### API Key & Rate Limiting

```javascript
class {
    function configureWebSearchSafely() {
        // ✅ Load API keys from secure storage, NOT hardcoded
        var braveKey = getSystemSetting( "BRAVE_API_KEY" )
        var googleKey = getSystemSetting( "GOOGLE_API_KEY" )

        if ( !len( braveKey ) && !len( googleKey ) ) {
            throw "No web search API keys configured"
        }

        return {
            providers: [
                { name: "brave", apiKey: braveKey },
                { name: "google", apiKey: googleKey }
            ]
        }
    }

    function enforceWebSearchRateLimit( required string userId ) {
        var query = queryExecute(
            "SELECT COUNT(*) as cnt FROM web_search_audit
             WHERE user_id = :userId
             AND created_at > DATE_SUB(NOW(), INTERVAL 1 HOUR)",
            { userId: arguments.userId }
        )

        var searchesPerHour = query.cnt
        var limit = 50  // 50 searches per hour per user

        if ( searchesPerHour >= limit ) {
            throw "Web search rate limit exceeded for user #arguments.userId#"
        }
    }
}
```

### Search Query Sanitization

```javascript
class {
    function sanitizeWebSearchQuery( required string query ) {
        // 1. Limit length
        var sanitized = left( arguments.query, 500 )

        // 2. Remove special characters that could cause API issues
        sanitized = reReplace( sanitized, "[^\\w\\s\\-'\"()]", " ", "all" )

        // 3. Trim whitespace
        sanitized = trim( sanitized )
        sanitized = reReplace( sanitized, "\\s{2,}", " ", "all" )

        // 4. Block known malicious search operators
        if ( sanitized.findNoCase( "inurl:" ) > 0 ||
             sanitized.findNoCase( "filetype:" ) > 0 ) {
            throw "Search operators not allowed"
        }

        return sanitized
    }

    function safeWebSearch( required string userQuery ) {
        // Sanitize
        var query = sanitizeWebSearchQuery( arguments.userQuery )

        // Rate limit
        enforceWebSearchRateLimit( session.user.id )

        // Execute with safe defaults
        var results = aiWebSearch( query, {
            provider: "brave",
            maxResults: 10,
            timeout: 10
        } )

        // Audit log
        queryExecute(
            "INSERT INTO web_search_audit (user_id, query, result_count, created_at)
             VALUES (:userId, :query, :resultCount, :createdAt)",
            {
                userId: session.user.id,
                query: query,
                resultCount: results.len(),
                createdAt: now()
            }
        )

        return results
    }
}
```

### Domain Filtering

```javascript
class {
    function filterWebSearchResults( required array results ) {
        // Block known malicious/spam domains
        var blockedDomains = [
            "spam-site.com",
            "malware-distribution.net",
            "phishing-domain.biz",
            "clickbait-farm.site"
        ]

        var blockedPatterns = [
            "(.*)\\.tk$",      // Cheap TLDs often used for spam
            "(.*)\\.ml$",
            "(.*)\\.ga$",
            "bit\\.ly.*",      // URL shorteners hide destination
            "tinyurl.*"
        ]

        return results.filter( r => {
            var domain = extractDomain( r.url )

            // Check against blocked list
            if ( blockedDomains.contains( domain ) ) {
                return false
            }

            // Check against patterns
            for ( pattern in blockedPatterns ) {
                if ( reFind( pattern, domain ) > 0 ) {
                    return false
                }
            }

            return true
        } )
    }
}
```
