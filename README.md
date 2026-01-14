# VALSEA Developer API Documentation

**Production services**

| Service | Base URL |
|---|---|
| **Speech-to-Text (ASR)** | `https://api.valsea.asia` |
| **Translation** | `https://translation.valsea.asia` |

---

## Engine / Provider Secrecy

VALSEA is presented as a **single unified API**.

- Do **not** depend on any engine/provider/model details.
- Responses may include **additional fields** over time; clients should **ignore unknown fields**.

---

## ASR (Speech-to-Text)

### POST `/transcribe`

Transcribe an uploaded audio file.

- **URL**: `POST https://api.valsea.asia/transcribe`
- **Content-Type**: `multipart/form-data`

#### Query parameters

| Name | Type | Required | Default | Description |
|---|---:|---:|---|---|
| `language` | string | No | `auto` | Language hint for best results. |
| `accent` | string | No | `sg` | Accent hint for post-processing corrections. |
| `enableCorrection` | boolean | No | `true` | Apply accent-aware mishear corrections. |
| `enableTags` | boolean | No | `true` | Extract semantic tags (times, dates, entities, etc.). |

#### `language` values

Use these (case-insensitive):

- `auto` (recommended default)
- `singlish`
- `en`
- `zh`
- `vi`
- `th`
- `id`
- `ms`
- `ta`
- `fil`

#### `accent` values

- `sg`, `my`, `id`, `th`, `vi`, `ph`, `cn`, `en`

#### Request (cURL)

```bash
curl -X POST "https://api.valsea.asia/transcribe?language=singlish&accent=sg" \
  -F "file=@audio.wav"
```

#### Response (JSON)

```json
{
  "text": "...final transcript...",
  "rawTranscript": "...original transcript...",
  "detectedLanguages": ["en-SG"],
  "accent_corrections": [],
  "semantic_tags": []
}
```

#### Response fields

| Field | Type | Notes |
|---|---|---|
| `text` | string | Final transcript (after optional post-processing). |
| `rawTranscript` | string | Base transcript before post-processing. |
| `detectedLanguages` | string[] | Best-effort language codes. |
| `accent_corrections` | array | Optional; may be missing/empty. |
| `semantic_tags` | array | Optional; may be missing/empty. |

#### Common examples

- **Auto-detect (default)**

```bash
curl -X POST "https://api.valsea.asia/transcribe" \
  -F "file=@audio.wav"
```

- **Vietnamese**

```bash
curl -X POST "https://api.valsea.asia/transcribe?language=vi" \
  -F "file=@audio.wav"
```

- **Chinese (Mandarin)**

```bash
curl -X POST "https://api.valsea.asia/transcribe?language=zh" \
  -F "file=@audio.wav"
```

- **Disable corrections + tags**

```bash
curl -X POST "https://api.valsea.asia/transcribe?enableCorrection=false&enableTags=false" \
  -F "file=@audio.wav"
```

---

### GET `/health`

- **URL**: `GET https://api.valsea.asia/health`

```json
{
  "status": "healthy",
  "service": "VALSEA ASR API",
  "version": "2.0",
  "supportedLanguages": ["en","zh","singlish","ta","vi","ms","id","th","fil"],
  "accents": ["sg","my","id","vi","th","ph","cn","en"],
  "features": {
    "transcription": true,
    "mishearCorrection": true,
    "semanticTagging": true
  },
  "timestamp": "2026-01-14T00:00:00.000Z"
}
```

### GET `/`

- **URL**: `GET https://api.valsea.asia/`

Returns basic service metadata.

---

## Translation

### POST `/api/translate`

Translate plain text.

- **URL**: `POST https://translation.valsea.asia/api/translate`
- **Content-Type**: `application/json`

#### Request body

```json
{
  "text": "Good morning, welcome to VALSEA!",
  "source": "en",
  "target": "id"
}
```

| Field | Type | Required | Notes |
|---|---|---:|---|
| `text` | string | ✅ | Max length: **10,000** characters. |
| `source` | string | ✅ | ISO language code. |
| `target` | string | ✅ | Must be in allowlist. |

#### Response

```json
{
  "translatedText": "Selamat pagi, selamat datang di VALSEA!"
}
```

> Clients must ignore unknown fields.

#### cURL

```bash
curl -X POST "https://translation.valsea.asia/api/translate" \
  -H "Content-Type: application/json" \
  -d '{"text":"Thank you","source":"en","target":"vi"}'
```

---

### GET `/api/translate/languages`

- **URL**: `GET https://translation.valsea.asia/api/translate/languages`

```json
{
  "allowedLanguages": ["en","id","ms","vi","th","zh-CN","zh-TW","tl","ta","km","lo"],
  "maxTextLength": 10000
}
```

---

### GET `/health` and `/health/deep`

- `GET https://translation.valsea.asia/health`
- `GET https://translation.valsea.asia/health/deep`

---

## Error format (recommended)

Some endpoints may return structured errors:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": {}
  }
}
```

Always handle non-200 responses.
