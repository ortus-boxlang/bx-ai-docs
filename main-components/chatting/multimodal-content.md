---
description: >-
  Work with images, audio, video, and documents in your AI conversations.
icon: images
---

# Multimodal Content

## Multimodal Content

Work with images, audio, video, and documents in your AI conversations.

### Images

Vision-capable models can analyze images alongside text.

#### Using Image URLs

```java
answer = aiChat(
    aiMessage()
        .user( "What is in this image?" )
        .image( "https://example.com/photo.jpg" )
)
```

#### Embedding Local Images

```java
answer = aiChat(
    aiMessage()
        .user( "Analyze this screenshot for bugs" )
        .embedImage( "/screenshots/ui-issue.png", "high" )
)
```

#### Multiple Images

```java
comparison = aiChat(
    aiMessage()
        .user( "What changed between these versions?" )
        .embedImage( "/designs/v1.png" )
        .embedImage( "/designs/v2.png" )
)
```

#### Detail Levels

* `"auto"` (default): Model decides
* `"low"`: Faster, cheaper, 512x512 resolution
* `"high"`: Full detail for complex images

```java
// Quick description
aiChat(
    aiMessage()
        .user( "Brief description" )
        .image( imageUrl, "low" )
)

// Detailed analysis
aiChat(
    aiMessage()
        .user( "Count all objects" )
        .image( imageUrl, "high" )
)
```

### Audio

Process audio files with AI models that support audio understanding.

**Supported by:** OpenAI (GPT-4o-audio), Gemini

```java
// Transcribe audio
transcript = aiChat(
    aiMessage()
        .user( "Transcribe this meeting recording" )
        .embedAudio( "/meetings/standup-2024-11-26.mp3" )
)

// Analyze audio content
analysis = aiChat(
    aiMessage()
        .user( "What are the main topics discussed?" )
        .audio( "https://example.com/podcast-episode.mp3" )
)

// Extract action items
actionItems = aiChat(
    aiMessage()
        .system( "Extract action items and owners from meeting audio" )
        .user( "Process this meeting" )
        .embedAudio( "/meetings/planning-session.mp3" )
)
```

**Supported formats:** mp3, mp4, mpeg, mpga, m4a, wav, webm

### Video

Analyze video content with AI models that support video understanding.

**Supported by:** Gemini (gemini-1.5-pro, gemini-2.0-flash)

```java
// Analyze video content
summary = aiChat(
    aiMessage()
        .user( "Summarize what happens in this video" )
        .embedVideo( "/videos/demo.mp4" ),
    {},
    { provider: "gemini" }
)

// Security footage analysis
report = aiChat(
    aiMessage()
        .system( "You are a security analyst" )
        .user( "Describe any suspicious activities" )
        .video( "https://example.com/security-cam-01.mp4" ),
    {},
    { provider: "gemini" }
)

// Tutorial analysis
steps = aiChat(
    aiMessage()
        .user( "List the step-by-step instructions shown" )
        .embedVideo( "/tutorials/how-to-setup.mp4" ),
    { model: "gemini-2.0-flash-exp" },
    { provider: "gemini" }
)
```

**Supported formats:** mp4, mpeg, mov, avi, flv, mpg, webm, wmv

### Documents and PDFs

Analyze documents, PDFs, and text files.

**Supported by:** Claude (Opus, Sonnet), OpenAI (GPT-4o)

```java
// Analyze PDF document
summary = aiChat(
    aiMessage()
        .user( "Summarize the key points from this document" )
        .embedPdf( "/reports/quarterly-results.pdf" )
)

// Extract specific information
dates = aiChat(
    aiMessage()
        .user( "Extract all important dates and deadlines" )
        .document( "https://example.com/project-plan.pdf", "Project Plan" )
)

// Legal document review
findings = aiChat(
    aiMessage()
        .system( "You are a legal analyst" )
        .user( "Identify potential issues in this contract" )
        .embedDocument( "/legal/vendor-contract.pdf", "Vendor Agreement" ),
    {},
    { provider: "claude" }
)

// Compare documents
comparison = aiChat(
    aiMessage()
        .user( "What are the differences between these contracts?" )
        .embedPdf( "/contracts/version-1.pdf", "Original" )
        .embedPdf( "/contracts/version-2.pdf", "Revised" )
)
```

**Supported formats:** pdf, doc, docx, txt, xls, xlsx

### Mixed Multimodal

Combine multiple media types in a single conversation:

```java
// Document + Images
analysis = aiChat(
    aiMessage()
        .user( "Does the product match the specifications?" )
        .embedPdf( "/specs/product-spec.pdf" )
        .embedImage( "/photos/product-front.jpg" )
        .embedImage( "/photos/product-back.jpg" )
)

// Video + Audio sync check
result = aiChat(
    aiMessage()
        .user( "Is the audio properly synced with the video?" )
        .embedVideo( "/media/video-clip.mp4" )
        .embedAudio( "/media/audio-track.mp3" )
)

// Complete project review
review = aiChat(
    aiMessage()
        .system( "Review all materials for consistency and quality" )
        .user( "Analyze these project deliverables" )
        .embedDocument( "/project/requirements.pdf", "Requirements" )
        .embedImage( "/project/mockup.png" )
        .embedVideo( "/project/demo.mp4" )
        .embedAudio( "/project/stakeholder-feedback.mp3" )
)
```

### Provider Support Matrix

| Feature       | OpenAI                | Claude      | Gemini              | Ollama            |
| ------------- | --------------------- | ----------- | ------------------- | ----------------- |
| **Images**    | ✅ GPT-4o, GPT-4-turbo | ✅ Claude 3+ | ✅ All vision models | ✅ LLaVA, Bakllava |
| **Audio**     | ✅ GPT-4o-audio        | ❌           | ✅ Gemini 1.5+, 2.0  | ❌                 |
| **Video**     | ❌                     | ❌           | ✅ Gemini 1.5+, 2.0  | ❌                 |
| **Documents** | ✅ GPT-4o              | ✅ Claude 3+ | ⚠️ Via OCR          | ❌                 |

### File Size Guidelines

* **Images:** Up to 20MB per image
* **Audio:** Up to 25MB (OpenAI), 2GB (Gemini)
* **Video:** Up to 2GB (Gemini only)
* **Documents:** \~10MB for inline base64, larger files need upload API

**Note:** Large files consume significant context tokens. For files >10MB, consider using provider-specific file upload APIs (OpenAI `/v1/files`, Gemini File API).

