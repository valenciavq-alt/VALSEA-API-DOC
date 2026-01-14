# VALSEA API Documentation

**Speech Intelligence for Southeast Asian Accents & Languages**

Version: 2.1.0  
Status: Production  

**Base URLs:**
| Service | URL |
|---------|-----|
| ASR (Speech-to-Text) | `https://api.valsea.asia` |
| Translation | `https://translation.valsea.asia` |
| Workflow Processing (optional) | `https://api.valsea.app` |

---

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Rate Limits](#rate-limits)
4. [Core APIs](#core-apis)
   - [Speech-to-Text (ASR)](#speech-to-text-asr)
   - [Translation](#translation)
   - [Workflow Processing](#workflow-processing)
5. [Use Case Formatters](#use-case-formatters)
6. [Supported Languages](#supported-languages)
7. [Error Handling](#error-handling)
8. [SDKs & Examples](#sdks--examples)

---

## Overview

VALSEA provides AI-powered speech intelligence optimized for Southeast Asian languages and accents. Our API enables:

- **Speech-to-Text**: Transcribe audio with support for Singlish, Chinglish, Vietnamese, Indonesian, Thai, Tagalog, and more
- **Translation**: Neural machine translation between SEA languages
- **Workflow Processing**: Transform transcripts into structured business outputs (meeting minutes, sales summaries, service logs, subtitles)

### Key Features

| Feature | Description |
|---------|-------------|
| 🌏 **SEA-Optimized** | Best-in-class accuracy for Southeast Asian accents |
| 🔄 **Code-Switching** | Handles mixed-language speech (e.g., Singlish with Mandarin) |
| ⚡ **Real-time** | Optional real-time support (separate service / deployment) |
| 📊 **Workflow Ready** | AI-powered formatting for business use cases |
| 🔒 **Secure** | Server-side processing, no client-side API keys |

---

## Authentication

### Authentication (Public Beta)

The public endpoints documented below are currently available **without authentication**.

If you need enterprise authentication (API keys / request signing), contact `sales@valsea.app`.

### Request Signing (Enterprise)

For enterprise clients, request signing is available for additional security. Contact sales@valsea.app for details.

---

## Rate Limits

| Plan | Requests/min | Audio Minutes/month | Characters/month (Translation) |
|------|-------------|---------------------|--------------------------------|
| Free | 10 | 60 | 100,000 |
| Pro | 100 | 1,000 | 1,000,000 |
| Enterprise | Custom | Custom | Custom |

Rate limit headers are included in all responses:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067200
```

---

## Core APIs

### Speech-to-Text (ASR)

#### POST /transcribe

Transcribe an audio file with automatic language detection and SEA-optimized post-processing.

**Base URL:** `https://api.valsea.asia`

**Request:**

```http
POST https://api.valsea.asia/transcribe
Content-Type: multipart/form-data
```

**Form fields:**

| Field | Type | Required | Description |
|------|------|----------|-------------|
| `file` | File | Yes | Audio file (WAV, MP3, AIFF, M4A, OGG, FLAC, WEBM) |

**Query parameters (optional):**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `language` | String | `auto` | Language hint: `auto`, `singlish`, `en`, `zh`, `vi`, `th`, `id`, `ms`, `ta`, `fil` |
| `accent` | String | `sg` | Accent hint for corrections: `sg`, `my`, `id`, `th`, `vi`, `ph`, `cn`, `en` |
| `enableCorrection` | Boolean | `true` | Apply accent-aware mishear corrections |
| `enableTags` | Boolean | `true` | Extract semantic tags (times, dates, entities, etc.) |

**Response:**

```json
{
  "text": "walao you sibei jialat leh go and dabao so long, 你到底要回来了吗",
  "rawTranscript": "walao you sibei jialat leh go and dabao so long, 你到底要回来了吗",
  "detectedLanguages": ["en-SG"],
  "accent_corrections": [],
  "semantic_tags": []
}
```

Notes:
- `accent_corrections` and `semantic_tags` may be missing or empty; clients should handle both.
- Clients should ignore unknown fields (new fields may be added over time).

**Example (cURL):**

```bash
curl -X POST "https://api.valsea.asia/transcribe?language=singlish&accent=sg" \
  -F "file=@meeting.mp3"
```

---

#### GET /health

Basic health check.

```http
GET https://api.valsea.asia/health
```

---

#### GET /

Service info.

```http
GET https://api.valsea.asia/
```

---

### Translation

**Base URL:** `https://translation.valsea.asia`

#### POST /api/translate

Translate text between supported languages. This endpoint is optimized for Southeast Asian language pairs and is protected by an allowlist for cost/control.

**Request:**

```http
POST https://translation.valsea.asia/api/translate
Content-Type: application/json
```

```json
{
  "text": "Good morning, welcome to VALSEA!",
  "source": "en",
  "target": "id"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | String | Yes | Text to translate (max 10,000 characters) |
| `source` | String | Yes | Source language code |
| `target` | String | Yes | Target language code (must be in allowlist) |

**Allowed Target Languages:**

| Code | Language |
|------|----------|
| `en` | English |
| `id` | Indonesian (Bahasa Indonesia) |
| `ms` | Malay (Bahasa Melayu) |
| `vi` | Vietnamese |
| `th` | Thai |
| `zh-CN` | Simplified Chinese |
| `zh-TW` | Traditional Chinese |
| `tl` | Tagalog/Filipino |
| `ta` | Tamil |
| `km` | Khmer |
| `lo` | Lao |

**Response:**

Notes:
- Clients **must ignore unknown fields** in the JSON response (the service may include additional metadata for debugging/observability).

```json
{
  "translatedText": "Selamat pagi, selamat datang di VALSEA!"
}
```

**Example (cURL):**

```bash
curl -X POST "https://translation.valsea.asia/api/translate" \
  -H "Content-Type: application/json" \
  -d '{"text": "Thank you for your help", "source": "en", "target": "vi"}'
```

**Response:**

```json
{
  "translatedText": "Cảm ơn bạn đã giúp đỡ"
}
```

---

#### GET /api/translate/languages

List supported translation languages.

**Response:**

```json
{
  "allowedLanguages": ["en", "id", "ms", "vi", "th", "zh-CN", "zh-TW", "tl", "ta", "km", "lo"],
  "maxTextLength": 10000
}
```

---

### Workflow Processing

Transform transcripts into structured business outputs using AI-powered semantic processing.

Note: Workflow endpoints are hosted on the optional workflow service (`https://api.valsea.app`) and may require separate access controls.

#### POST /api/v1/semantic/process

Process a transcript with a specific use case formatter.

**Request:**

```http
POST /api/v1/semantic/process
Content-Type: application/json
```

```json
{
  "audio_id": "uuid-from-transcription",
  "use_case": "meeting_minute"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `audio_id` | String | Yes | Audio ID from transcription |
| `use_case` | String | Yes | Use case: `meeting_minute`, `sales_summary`, `service_log`, `subtitle` |

**Response:**

```json
{
  "audio_id": "abc-123-def",
  "status": "semantic_ready",
  "semantic": {
    "use_case": "meeting_minute",
    "summary": "Team discussed Q1 project timeline and assigned tasks",
    "date": "2026-01-14",
    "attendees": [...],
    "agenda_items": [...],
    "action_items_summary": [...]
  },
  "api_processing_time_ms": 2345
}
```

---

#### GET /api/v1/semantic/get

Retrieve processed semantic result.

**Request:**

```http
GET /api/v1/semantic/get?id={audio_id}
```

---

## Use Case Formatters

### 📝 Meeting Minute (`meeting_minute`)

Structured meeting notes with attendees, agenda, decisions, and action items.

**Output Schema:**

```json
{
  "use_case": "meeting_minute",
  "summary": "Brief 1-2 sentence overview",
  "date": "2026-01-14",
  "time": "14:30",
  "duration_minutes": 25,
  "attendees": [
    {
      "name": "Speaker 1",
      "role": "Project Manager",
      "participated": true
    }
  ],
  "agenda_items": [
    {
      "topic": "Project timeline",
      "discussion": "Summary of what was discussed",
      "decisions": ["Decision 1", "Decision 2"],
      "action_items": [
        {
          "task": "Complete API documentation",
          "assigned_to": "John",
          "due_date": "2026-01-20",
          "priority": "high"
        }
      ]
    }
  ],
  "key_decisions": ["Decision with context"],
  "action_items_summary": [...],
  "next_meeting": {
    "date": "2026-01-21",
    "time": "14:00",
    "topics": ["Follow-up topic"]
  }
}
```

---

### 💼 Sales Summary (`sales_summary`)

Sales conversation analysis with deal information, objections, and CRM-ready data.

**Output Schema:**

```json
{
  "use_case": "sales_summary",
  "summary": "Overview of sales conversation",
  "participants": {
    "salesperson": "Sales Rep Name",
    "customer": {
      "name": "Customer Name",
      "company": "Company Inc",
      "role": "Decision Maker"
    }
  },
  "product_interest": [
    {
      "product": "Product Name",
      "interest_level": "high",
      "features_discussed": ["Feature 1", "Feature 2"]
    }
  ],
  "pain_points": [...],
  "objections": [
    {
      "objection": "Price concerns",
      "response": "How it was addressed",
      "status": "resolved"
    }
  ],
  "next_steps": [...],
  "deal_stage": "qualification",
  "probability": 0.75,
  "estimated_value": 50000,
  "currency": "SGD",
  "sentiment": "positive",
  "key_quotes": ["Important customer quote"]
}
```

---

### 🛠 Service Log (`service_log`)

Technical service call documentation with troubleshooting steps and resolution.

**Output Schema:**

```json
{
  "use_case": "service_log",
  "summary": "Brief summary of service call",
  "participants": {
    "technician": "Tech Name",
    "customer": {
      "name": "Customer Name",
      "account_id": "ACC-12345"
    }
  },
  "issue_category": "technical",
  "problem_description": "Detailed issue description",
  "symptoms": ["Symptom 1", "Symptom 2"],
  "troubleshooting_steps": [
    {
      "step": 1,
      "action": "Checked system logs",
      "result": "Found error in log file",
      "timestamp": "09:05"
    }
  ],
  "root_cause": "Identified root cause",
  "resolution": {
    "solution": "Applied fix",
    "steps_taken": ["Step 1", "Step 2"],
    "verification": "How verified",
    "resolution_time_minutes": 15
  },
  "status": "resolved",
  "priority": "high",
  "ticket_id": "TICKET-12345",
  "follow_up_actions": [...]
}
```

---

### 🎬 Subtitle (`subtitle`)

Time-synced subtitle format ready for SRT/VTT export.

**Output Schema:**

```json
{
  "use_case": "subtitle",
  "title": "Video Title",
  "language": "en",
  "duration_seconds": 360,
  "subtitles": [
    {
      "sequence": 1,
      "start_time": "00:00:00,000",
      "end_time": "00:00:03,500",
      "text": "First subtitle line",
      "speaker": "Speaker 1"
    }
  ],
  "chapters": [
    {
      "title": "Chapter 1",
      "start_time": "00:00:00",
      "end_time": "00:05:00"
    }
  ],
  "metadata": {
    "speakers": ["Speaker 1", "Speaker 2"],
    "total_lines": 120
  }
}
```

---

## Supported Languages

### Speech-to-Text (ASR)

| Region | Languages/Dialects | Accent Support |
|--------|-------------------|----------------|
| 🇸🇬 Singapore | English, Mandarin, Malay, Tamil, Singlish | ✅ Native |
| 🇲🇾 Malaysia | English, Malay, Mandarin, Tamil, Manglish | ✅ Native |
| 🇮🇩 Indonesia | Indonesian, Javanese, English | ✅ Native |
| 🇻🇳 Vietnam | Vietnamese, English | ✅ Native |
| 🇹🇭 Thailand | Thai, English | ✅ Native |
| 🇵🇭 Philippines | Tagalog, English, Taglish | ✅ Native |
| 🇰🇭 Cambodia | Khmer, English | ✅ Native |
| 🇱🇦 Laos | Lao, English | ✅ Native |

### Code-Switching Support

VALSEA handles seamless code-switching between:

- **Singlish**: English + Mandarin + Hokkien + Malay
- **Taglish**: Tagalog + English
- **Manglish**: Malay + English
- **Chinglish**: Chinese-accented English

---

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "additional_context"
    }
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_REQUEST` | 400 | Malformed request body |
| `MISSING_FIELD` | 400 | Required field not provided |
| `INVALID_LANGUAGE` | 400 | Language not in allowlist |
| `TEXT_TOO_LONG` | 413 | Text exceeds 10,000 characters |
| `FILE_TOO_LARGE` | 413 | Audio file exceeds 100MB |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `UNAUTHORIZED` | 401 | Invalid or missing API key |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `TRANSCRIPTION_FAILED` | 502 | ASR service error |
| `TRANSLATION_FAILED` | 502 | Translation service error |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## SDKs & Examples

### JavaScript/TypeScript

```typescript
import { ValseaClient } from '@valsea/sdk';

const valsea = new ValseaClient({
  apiKey: process.env.VALSEA_API_KEY,
});

// Transcribe audio
const result = await valsea.transcribe({
  file: audioBuffer,
  language: 'auto',
  region: 'sg',
});

console.log(result.transcription);

// Translate text (uses translation.valsea.asia)
const translation = await valsea.translate({
  text: 'Good morning!',
  source: 'en',
  target: 'id',
});

console.log(translation.translatedText);

// Process workflow
const workflow = await valsea.processWorkflow({
  audioId: result.request_id,
  useCase: 'meeting_minute',
});

console.log(workflow.semantic);
```

### Python

```python
from valsea import ValseaClient

client = ValseaClient(api_key="YOUR_API_KEY")

# Transcribe audio
with open("meeting.mp3", "rb") as f:
    result = client.transcribe(
        file=f,
        language="auto",
        region="sg"
    )

print(result.transcription)

# Translate text
translation = client.translate(
    text="Good morning!",
    source="en",
    target="vi"
)

print(translation.translated_text)

# Process workflow
workflow = client.process_workflow(
    audio_id=result.request_id,
    use_case="sales_summary"
)

print(workflow.semantic)
```

### cURL Examples

```bash
# Transcribe audio
curl -X POST "https://api.valsea.asia/transcribe?accent=sg" \
  -F "file=@audio.mp3"

# Translate text
curl -X POST "https://translation.valsea.asia/api/translate" \
  -H "Content-Type: application/json" \
  -d '{"text": "Thank you", "source": "en", "target": "id"}'

# Note: Subtitle export formats (SRT/VTT) are not exposed on `api.valsea.asia` at this time.
# If you need subtitle files, contact support for the current recommended workflow.

# Process meeting minutes (optional workflow service)
curl -X POST "https://api.valsea.app/api/v1/semantic/process" \
  -H "Content-Type: application/json" \
  -d '{"audio_id": "abc-123", "use_case": "meeting_minute"}'
```

---

## Health & Status

### GET /health

Basic health check.

ASR health:

```http
GET https://api.valsea.asia/health
```

Translation health:

```http
GET https://translation.valsea.asia/health
GET https://translation.valsea.asia/health/deep
```

### GET https://translation.valsea.asia/health/deep

Translation service health with latency check.

```json
{
  "status": "ok",
  "timestamp": "2026-01-14T10:30:00.000Z",
  "service": "translate-gateway",
  "translation": {
    "healthy": true,
    "latencyMs": 245
  }
}
```

---

## Support

- **Documentation**: https://docs.valsea.app
- **API Status**: https://status.valsea.app
- **Email**: support@valsea.app
- **Enterprise Sales**: sales@valsea.app

---

© 2026 VALSEA. Built for Southeast Asia.
