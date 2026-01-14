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

## Quick start (copy/paste)

### ASR: transcribe a file

```bash
curl -X POST "https://api.valsea.asia/transcribe" \
  -F "file=@audio.wav"
```

### Translation: translate text

```bash
curl -X POST "https://translation.valsea.asia/api/translate" \
  -H "Content-Type: application/json" \
  -d '{"text":"Good morning","source":"en","target":"id"}'
```

---

## ASR (Speech-to-Text)

### POST `/transcribe`

Transcribe an uploaded audio file.

- **URL**: `POST https://api.valsea.asia/transcribe`
- **Content-Type**: `multipart/form-data`

#### File requirements

- **Form field name**: `file`
- **Max size**: 25MB (recommended)
- **Formats**: WAV, MP3, AIFF, OGG, FLAC, WEBM, M4A (best-effort; WAV recommended)

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

#### JavaScript example (fetch)

```js
export async function transcribe({ file, language, accent = "sg", enableCorrection = true, enableTags = true }) {
  const form = new FormData();
  form.append("file", file);

  const params = new URLSearchParams();
  if (language) params.set("language", language);
  if (accent) params.set("accent", accent);
  params.set("enableCorrection", String(enableCorrection));
  params.set("enableTags", String(enableTags));

  const res = await fetch(`https://api.valsea.asia/transcribe?${params.toString()}`, {
    method: "POST",
    body: form,
  });
  if (!res.ok) throw new Error(`ASR failed: HTTP ${res.status}`);
  return await res.json(); // ignore unknown fields
}
```

#### Python example (requests)

```python
import requests

def transcribe(path, language=None, accent="sg", enable_correction=True, enable_tags=True):
    params = {
        "accent": accent,
        "enableCorrection": str(enable_correction).lower(),
        "enableTags": str(enable_tags).lower(),
    }
    if language:
        params["language"] = language

    with open(path, "rb") as f:
        r = requests.post("https://api.valsea.asia/transcribe", params=params, files={"file": f}, timeout=180)
    r.raise_for_status()
    return r.json()  # ignore unknown fields
```

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

#### JavaScript example (fetch)

```js
export async function translate({ text, source, target }) {
  const res = await fetch("https://translation.valsea.asia/api/translate", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ text, source, target }),
  });
  if (!res.ok) throw new Error(`Translate failed: HTTP ${res.status}`);
  return await res.json(); // ignore unknown fields
}
```

#### Python example (requests)

```python
import requests

def translate(text, source, target):
    r = requests.post(
        "https://translation.valsea.asia/api/translate",
        json={"text": text, "source": source, "target": target},
        timeout=30,
    )
    r.raise_for_status()
    return r.json()  # ignore unknown fields
```

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
