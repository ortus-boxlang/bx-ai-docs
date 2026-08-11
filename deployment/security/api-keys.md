---
description: >-
  Protecting API keys and secrets: environment variables, secrets managers, key rotation, and environment-scoped keys.
icon: key
---

# API Key Management

### Never Hardcode Keys

```javascript
// ❌ WRONG - Hardcoded keys
apiKey = "sk-1234567890abcdef"

// ❌ WRONG - Keys in code files
settings = {
    openai: {
        apiKey: "sk-1234567890abcdef"
    }
}

// ✅ RIGHT - Environment variables
apiKey = getSystemSetting( "OPENAI_API_KEY" )

// ✅ RIGHT - Detect from environment automatically
response = aiChat( "Hello" )  // Auto-detects OPENAI_API_KEY
```

### Secrets Manager Integration

#### AWS Secrets Manager

```javascript
class {

    property name="secretsClient" inject="aws:SecretsManager";

    function getAPIKey( required string secretName ) {
        try {
            var secret = variables.secretsClient.getSecretValue({
                SecretId: arguments.secretName
            })

            return dejsonSerialize( secret.SecretString ).apiKey

        } catch ( any e ) {
            writeLog(
                "Failed to retrieve secret: #arguments.secretName#",
                "error"
            )
            throw e
        }
    }

    function configureAI() {
        var openaiKey = getAPIKey( "prod/ai/openai" )
        var claudeKey = getAPIKey( "prod/ai/claude" )

        // Configure providers with API keys
        aiService( "openai" ).configure({ apiKey: openaiKey })
        aiService( "claude" ).configure({ apiKey: claudeKey })

        // Or configure with additional options
        aiService( "openai" ).configure({
            apiKey: openaiKey,
            timeout: 120,
            logRequest: false
        })
    }
}
```

#### Azure Key Vault

```javascript
class {

    property name="keyVaultClient" inject="azure:KeyVault";
    property name="vaultUrl" default="https://myvault.vault.azure.net/";

    function getAPIKey( required string secretName ) {
        try {
            var secret = variables.keyVaultClient.getSecret(
                variables.vaultUrl,
                arguments.secretName
            )

            return secret.value

        } catch ( any e ) {
            writeLog(
                "Failed to retrieve secret from Key Vault: #arguments.secretName#",
                "error"
            )
            throw e
        }
    }
}
```

#### HashiCorp Vault

```javascript
class {

    property name="vaultAddress" default="https://vault.company.com";
    property name="vaultToken";

    function init() {
        // Authenticate to Vault
        variables.vaultToken = getSystemSetting( "VAULT_TOKEN" )
        return this
    }

    function getAPIKey( required string path ) {
        var response = http( "#variables.vaultAddress#/v1/#arguments.path#" )
            .header( "X-Vault-Token", variables.vaultToken )
            .send()

        if ( response.statusCode == 200 ) {
            var data = dejsonSerialize( response.fileContent )
            return data.data.apiKey
        }

        throw "Failed to retrieve secret from Vault"
    }
}
```

### Key Rotation

```javascript
class singleton {

    property name="keys" type="struct";
    property name="rotationSchedule" type="struct";

    function init() {
        variables.keys = {}
        variables.rotationSchedule = {}
        loadKeys()

        // Schedule automatic rotation (every 30 days)
        scheduleRotation()

        return this
    }

    function loadKeys() {
        providers = [ "openai", "claude", "gemini" ]

        for ( provider in providers ) {
            variables.keys[ provider ] = {
                current: getSecretFromVault( "#provider#/api-key-current" ),
                next: getSecretFromVault( "#provider#/api-key-next" ),
                rotatedAt: now()
            }
        }

        writeLog( "API keys loaded for #providers.len()# providers" )
    }

    function rotateKeys( required string provider ) {
        lock name="keyrotation_#arguments.provider#" type="exclusive" timeout="10" {
            writeLog( "Starting key rotation for #arguments.provider#" )

            // Move next key to current
            var oldKey = variables.keys[ arguments.provider ].current
            variables.keys[ arguments.provider ].current = variables.keys[ arguments.provider ].next

            // Generate new next key (provider-specific)
            variables.keys[ arguments.provider ].next = generateNewKey( arguments.provider )
            variables.keys[ arguments.provider ].rotatedAt = now()

            // Update vault
            updateVault( "#arguments.provider#/api-key-current", variables.keys[ arguments.provider ].current )
            updateVault( "#arguments.provider#/api-key-next", variables.keys[ arguments.provider ].next )

            // Revoke old key (after grace period)
            scheduleKeyRevocation( arguments.provider, oldKey, 3600 )  // 1 hour

            writeLog( "Key rotation completed for #arguments.provider#" )
            notifyOps( "API key rotated", { provider: arguments.provider } )
        }
    }

    function getKey( required string provider ) {
        // Check if rotation is due (30 days)
        var daysSinceRotation = dateDiff( "d", variables.keys[ arguments.provider ].rotatedAt, now() )

        if ( daysSinceRotation >= 30 ) {
            rotateKeys( arguments.provider )
        }

        return variables.keys[ arguments.provider ].current
    }

    function scheduleRotation() {
        // Schedule rotation check daily
        BoxAnnounce( "onScheduledTask", {
            name: "checkKeyRotation",
            interval: "daily",
            task: () => {
                for ( provider in variables.keys ) {
                    var daysSince = dateDiff( "d", variables.keys[ provider ].rotatedAt, now() )
                    if ( daysSince >= 30 ) {
                        rotateKeys( provider )
                    }
                }
            }
        } )
    }
}
```

### Key Scope Limitation

**Use separate keys for different environments**:

```javascript
// config/ai-config.bx
function getAPIKeys() {
    var env = getSystemSetting( "ENVIRONMENT", "production" )

    var keyMappings = {
        "development": {
            openai: getSystemSetting( "OPENAI_DEV_KEY" ),
            claude: getSystemSetting( "CLAUDE_DEV_KEY" )
        },
        "staging": {
            openai: getSystemSetting( "OPENAI_STAGING_KEY" ),
            claude: getSystemSetting( "CLAUDE_STAGING_KEY" )
        },
        "production": {
            openai: getSecretFromVault( "prod/openai/key" ),
            claude: getSecretFromVault( "prod/claude/key" )
        }
    }

    return keyMappings[ env ]
}
```
