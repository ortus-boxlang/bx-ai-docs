---
description: >-
  Enterprise guide to implementing multi-tenant memory isolation in BoxLang AI
  applications
icon: users-gear
---

# Multi-Tenant Memory Guide

This guide covers implementing **secure, isolated memory** for multi-user and multi-conversation applications using BoxLang AI's built-in multi-tenant support.

## 📖 Overview

Multi-tenant memory isolation enables:

### 🏗️ Multi-Tenant Isolation Architecture

```mermaid
graph TB
    subgraph "Application Layer"
        APP[Application]
    end

    subgraph "User A"
        UA[userId: alice]
        CA1[conversationId: support]
        CA2[conversationId: sales]
    end

    subgraph "User B"
        UB[userId: bob]
        CB1[conversationId: support]
        CB2[conversationId: billing]
    end

    subgraph "Memory Storage"
        MA1[(Memory: alice-support)]
        MA2[(Memory: alice-sales)]
        MB1[(Memory: bob-support)]
        MB2[(Memory: bob-billing)]
    end

    APP --> UA
    APP --> UB

    UA --> CA1
    UA --> CA2
    UB --> CB1
    UB --> CB2

    CA1 --> MA1
    CA2 --> MA2
    CB1 --> MB1
    CB2 --> MB2

    style APP fill:#BD10E0
    style UA fill:#4A90E2
    style UB fill:#4A90E2
    style MA1 fill:#50E3C2
    style MA2 fill:#50E3C2
    style MB1 fill:#50E3C2
    style MB2 fill:#50E3C2
```

* **👥 Per-User Isolation**: Separate conversations for each user
* **💬 Per-Conversation Isolation**: Multiple conversations per user
* **🔒 Data Security**: Automatic filtering prevents data leakage
* **🏢 Enterprise Scale**: Shared infrastructure with isolated data
* **📊 Analytics**: Track conversations by user/conversation identifiers

### Why Multi-Tenant Memory?

Without multi-tenant isolation, all users share the same conversation history:

```java
// ❌ WRONG: All users see same conversation
memory = aiMemory( memory: "window", config: { maxMessages: 10 } )
agent = aiAgent( name: "Assistant", memory: memory )

// User Alice
agent.run( "My name is Alice" )

// User Bob sees Alice's conversation!
agent.run( "What's the user's name?" )  // "Alice" (data leakage!)
```

With multi-tenant isolation, each user gets isolated memory:

```java
// ✅ CORRECT: Isolated per user
aliceMemory = aiMemory( memory: "window",
    userId: "alice",
    config: { maxMessages: 10 }
)

bobMemory = aiMemory( memory: "window",
    userId: "bob",
    config: { maxMessages: 10 }
)

// Completely isolated conversations
```

***


## 🎯 Core Concepts

### 👤 UserId - User-Level Isolation

```mermaid
graph LR
    A[Application] --> U1[User: alice]
    A --> U2[User: bob]
    A --> U3[User: charlie]

    U1 --> M1[(Memory)]
    U2 --> M2[(Memory)]
    U3 --> M3[(Memory)]

    style A fill:#BD10E0
    style U1 fill:#4A90E2
    style U2 fill:#4A90E2
    style U3 fill:#4A90E2
    style M1 fill:#50E3C2
    style M2 fill:#50E3C2
    style M3 fill:#50E3C2
```

The `userId` parameter isolates conversations at the **user level**:

```java
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: "user123",  // User identifier
    config: { maxMessages: 10 }
)
```

**Use cases:**

* Single conversation per user
* User-specific context retention
* Customer support with user history
* Personal AI assistants

### 💬 ConversationId - Conversation-Level Isolation

The `conversationId` parameter enables **multiple conversations per user**:

```mermaid
graph TB
    U[User: alice] --> C1[Conversation: support]
    U --> C2[Conversation: sales]
    U --> C3[Conversation: billing]

    C1 --> M1[(Memory 1)]
    C2 --> M2[(Memory 2)]
    C3 --> M3[(Memory 3)]

    style U fill:#4A90E2
    style C1 fill:#F5A623
    style C2 fill:#F5A623
    style C3 fill:#F5A623
    style M1 fill:#50E3C2
    style M2 fill:#50E3C2
    style M3 fill:#50E3C2
```

```java
supportChat = aiMemory( memory: "window",
    key: createUUID(),
    userId: "user123",
    conversationId: "support-ticket-456",  // Conversation identifier
    config: { maxMessages: 10 }
)

salesChat = aiMemory( memory: "window",
    key: createUUID(),
    userId: "user123",
    conversationId: "sales-inquiry-789",  // Different conversation
    config: { maxMessages: 10 }
)
```

**Use cases:**

* Multiple chat windows
* Topic-based conversations
* Ticket-based support systems
* Project-specific contexts

### Combined Isolation

Use **both** userId and conversationId for complete isolation:

```java
memory = aiMemory( memory: "window",
    key: createUUID(),
    userId: "user123",           // Who owns this conversation
    conversationId: "chat456",   // Which conversation within user
    config: { maxMessages: 10 }
)
```

This provides:

* **User isolation**: User A can't see User B's conversations
* **Conversation isolation**: User A's chat1 is separate from their chat2
* **Complete privacy**: Each conversation is fully isolated

***


## 📖 Sub-Pages

| Page | Description |
|---|---|
| [Implementation Patterns](implementation-patterns.md) | Implementation patterns, memory type strategies, and vector memory multi-tenancy |
| [Security & Performance](security-and-performance.md) | Security considerations and performance optimization for multi-tenant memory |
| [Enterprise & Migration](enterprise-and-migration.md) | Enterprise patterns, migration guide, and troubleshooting |

## See Also

* [Memory Systems Guide](../) - Standard conversation memory
* [Vector Memory Guide](../vector-memory/README.md) - Semantic search with isolation
* [Agents Documentation](../../agents/) - Using memory in agents
* [Security Guide](../../../deployment/security/README.md) - Application security

***

**Need Help?** Join the [BoxLang Discord](https://discord.gg/boxlang) or check the [GitHub Discussions](https://github.com/ortus-boxlang/bx-ai/discussions) for community support.
