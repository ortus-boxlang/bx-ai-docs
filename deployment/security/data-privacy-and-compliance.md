---
description: >-
  Handling sensitive data appropriately, PII detection and redaction,
  encryption, and GDPR/HIPAA compliance.
icon: user-lock
---

# Data Privacy & Compliance

## Data Privacy

### Local vs Cloud Providers

**Choose providers based on privacy requirements**:

| Provider         | Data Location | Training on Your Data | Retention             | Best For                    |
| ---------------- | ------------- | --------------------- | --------------------- | --------------------------- |
| **Ollama**       | Local only    | No                    | Never sent            | Maximum privacy, on-premise |
| **LM Studio**    | Local only    | No                    | Never sent            | Desktop, development        |
| **OpenAI**       | Cloud         | No (since March 2023) | 30 days               | General use                 |
| **Claude**       | Cloud         | No                    | Not used for training | General use                 |
| **Azure OpenAI** | Your region   | No                    | Controlled by you     | Enterprise, compliance      |

### Data Minimization

**Send only necessary data to AI**:

```javascript
class {
    function prepareUserData( required struct user ) {
        // ❌ WRONG - Send full user object
        // prompt = "Process this user: #jsonSerialize( arguments.user )#"

        // ✅ RIGHT - Send only necessary fields
        var safeData = {
            userId: hashUserId( arguments.user.id ),  // Hash IDs
            preferences: arguments.user.preferences,
            // Don't send: email, phone, address, SSN, etc.
        }

        return safeData
    }

    function hashUserId( required string userId ) {
        // One-way hash for analytics
        return hash( arguments.userId, "SHA-256" )
    }
}
```

### PII Detection and Redaction

```javascript
class {
    function detectPII( required string text ) {
        var patterns = {
            email: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
            phone: "\b\d{3}[-.]?\d{3}[-.]?\d{4}\b",
            ssn: "\b\d{3}-\d{2}-\d{4}\b",
            creditCard: "\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b",
            ipAddress: "\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b"
        }

        var found = []

        for ( type in patterns ) {
            var matches = reMatch( patterns[ type ], arguments.text )
            if ( !matches.isEmpty() ) {
                found.append( type )
                writeLog( "PII detected: #type#", "warning" )
            }
        }

        return found
    }

    function redactPII( required string text ) {
        var redacted = arguments.text

        // Redact email addresses
        redacted = reReplace(
            redacted,
            "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
            "[EMAIL REDACTED]",
            "all"
        )

        // Redact phone numbers
        redacted = reReplace(
            redacted,
            "\b\d{3}[-.]?\d{3}[-.]?\d{4}\b",
            "[PHONE REDACTED]",
            "all"
        )

        // Redact SSN
        redacted = reReplace(
            redacted,
            "\b\d{3}-\d{2}-\d{4}\b",
            "[SSN REDACTED]",
            "all"
        )

        // Redact credit cards
        redacted = reReplace(
            redacted,
            "\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b",
            "[CARD REDACTED]",
            "all"
        )

        return redacted
    }

    function safeAIChat( required string prompt ) {
        // Check for PII before sending
        var piiTypes = detectPII( arguments.prompt )

        if ( !piiTypes.isEmpty() ) {
            writeLog( "Redacting PII before AI call: #piiTypes.toList()#", "warning" )
            var safeProm = redactPII( arguments.prompt )
            return aiChat( safePrompt )
        }

        return aiChat( arguments.prompt )
    }
}
```

### Encryption

**Encrypt sensitive data at rest and in transit**:

```javascript
class {
    property name="encryptionKey";

    function init() {
        // Load encryption key from secure storage
        variables.encryptionKey = getSystemSetting( "ENCRYPTION_KEY" )
        return this
    }

    function encryptData( required string data ) {
        return encrypt(
            arguments.data,
            variables.encryptionKey,
            "AES",
            "Base64"
        )
    }

    function decryptData( required string encrypted ) {
        return decrypt(
            arguments.encrypted,
            variables.encryptionKey,
            "AES",
            "Base64"
        )
    }

    function storeConversation( required string userId, required array messages ) {
        // Encrypt before storing
        var encrypted = encryptData( jsonSerialize( arguments.messages ) )

        queryExecute(
            "INSERT INTO conversations (user_id, messages, created_at)
             VALUES (:userId, :messages, :createdAt)",
            {
                userId: arguments.userId,
                messages: encrypted,
                createdAt: now()
            }
        )
    }

    function loadConversation( required string userId ) {
        var query = queryExecute(
            "SELECT messages FROM conversations WHERE user_id = :userId",
            { userId: arguments.userId }
        )

        if ( query.recordCount > 0 ) {
            var decrypted = decryptData( query.messages )
            return dejsonSerialize( decrypted )
        }

        return []
    }
}
```

***

## Compliance

### GDPR Compliance

**Requirements for EU data**:

```javascript
class {
    // Right to Access
    function exportUserData( required string userId ) {
        return {
            conversations: getConversations( arguments.userId ),
            memory: getMemory( arguments.userId ),
            auditLog: getUserActivity( arguments.userId ),
            vectorData: getVectorMemory( arguments.userId )
        }
    }

    // Right to Erasure (Right to be Forgotten)
    function deleteUserData( required string userId ) {
        transaction {
            // Delete conversations
            queryExecute(
                "DELETE FROM conversations WHERE user_id = :userId",
                { userId: arguments.userId }
            )

            // Delete memory
            queryExecute(
                "DELETE FROM ai_memory WHERE user_id = :userId",
                { userId: arguments.userId }
            )

            // Delete vector embeddings
            var vectorMemory = aiVectorMemory( "chroma" )
            vectorMemory.delete({ userId: arguments.userId })

            // Anonymize audit logs (keep for compliance)
            queryExecute(
                "UPDATE ai_audit_log
                 SET data = jsonb_set(data, '{userId}', '\"[DELETED]\"')
                 WHERE data->>'userId' = :userId",
                { userId: arguments.userId }
            )

            writeLog( "User data deleted for GDPR compliance: #arguments.userId#" )
        }
    }

    // Data Portability
    function exportUserDataJSON( required string userId ) {
        var data = exportUserData( arguments.userId )
        return jsonSerialize( data )
    }

    // Consent Management
    function recordConsent(
        required string userId,
        required string consentType,
        required boolean granted
    ) {
        queryExecute(
            "INSERT INTO user_consent (user_id, consent_type, granted, recorded_at)
             VALUES (:userId, :consentType, :granted, :recordedAt)",
            {
                userId: arguments.userId,
                consentType: arguments.consentType,
                granted: arguments.granted,
                recordedAt: now()
            }
        )
    }

    function checkConsent( required string userId, required string consentType ) {
        var query = queryExecute(
            "SELECT granted FROM user_consent
             WHERE user_id = :userId
             AND consent_type = :consentType
             ORDER BY recorded_at DESC
             LIMIT 1",
            {
                userId: arguments.userId,
                consentType: arguments.consentType
            }
        )

        return query.recordCount > 0 && query.granted
    }
}
```

### HIPAA Compliance

**Requirements for healthcare data**:

```javascript
class {
    // Business Associate Agreement (BAA)
    // Only use HIPAA-compliant providers:
    // - Azure OpenAI (with BAA)
    // - Local Ollama deployment
    // NOT: OpenAI public API, Claude public API

    function ensureHIPAACompliance() {
        var allowedProviders = [ "azure-openai", "ollama" ]
        var currentProvider = getSystemSetting( "AI_PROVIDER" )

        if ( !allowedProviders.contains( currentProvider ) ) {
            throw "Provider #currentProvider# is not HIPAA compliant. Use Azure OpenAI with BAA or local Ollama."
        }
    }

    // PHI must be encrypted at rest
    function storePatientData( required struct patient ) {
        var encrypted = encryptPHI( jsonSerialize( arguments.patient ) )

        queryExecute(
            "INSERT INTO patient_data (data, created_at) VALUES (:data, :createdAt)",
            {
                data: encrypted,
                createdAt: now()
            }
        )
    }

    // Minimum necessary rule
    function preparePatientContext( required struct patient ) {
        // Only include minimum necessary PHI
        return {
            age: arguments.patient.age,
            gender: arguments.patient.gender,
            conditions: arguments.patient.conditions
            // DON'T include: name, SSN, address, etc.
        }
    }
}
```

### Data Retention Policies

```javascript
class {
    function applyRetentionPolicy() {
        var retentionDays = getSystemSetting( "DATA_RETENTION_DAYS", 90 )
        var cutoffDate = dateAdd( "d", -retentionDays, now() )

        // Delete old conversations
        queryExecute(
            "DELETE FROM conversations WHERE created_at < :cutoffDate",
            { cutoffDate: cutoffDate }
        )

        // Delete old memory
        queryExecute(
            "DELETE FROM ai_memory WHERE created_at < :cutoffDate",
            { cutoffDate: cutoffDate }
        )

        // Archive audit logs (don't delete for compliance)
        queryExecute(
            "UPDATE ai_audit_log
             SET archived = true
             WHERE created_at < :cutoffDate AND archived = false",
            { cutoffDate: cutoffDate }
        )

        writeLog( "Data retention policy applied: deleted data older than #retentionDays# days" )
    }
}
```
