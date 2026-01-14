> **Latest full API spec (ASR + Translate + Workflow):** see [`VALSEA_API_DOCUMENTATION.md`](./VALSEA_API_DOCUMENTATION.md)

# VALSEA Developer API Docs

**Production services**

| Service | Base URL |
|---|---|
| **Speech-to-Text (ASR)** | `https://api.valsea.asia` |
| **Translation** | `https://translation.valsea.asia` |
| **Workflow (optional)** | `https://api.valsea.app` |

---

## Engine / Provider Secrecy

VALSEA is presented as a **single unified API**.

- Do **not** depend on any internal vendor/provider/model identifiers.
- Responses may include **additional fields** over time; clients should **ignore unknown fields**.

---

## How to use these APIs (entry points + common patterns)

You can call any endpoint independently, or chain multiple endpoints together.

**Entry points**
- **ASR (audio → transcript)**: `POST /transcribe`
- **Text annotations (text → tags + annotated text)**: `POST /annotate`
- **Clarified English (text → clearer English)**: `POST /clarify`
- **Translation (text → translatedText)**: `POST https://translation.valsea.asia/api/translate`
- **Use-case output (structured JSON)** *(optional)*: workflow service endpoints

**Common patterns**
- **ASR only**: `/transcribe`
- **ASR + tags/annotations**: `/transcribe?enableTags=true&annotate=true` or `/annotate`
- **ASR + clarified English**: `/transcribe?clarify=true` or `/clarify`
- **Translation only**: translation gateway endpoints
- **Use-case only**: workflow endpoints (structured output)

---

## Quick start (copy/paste)

### ASR: transcribe a file

```bash
curl -X POST "https://api.valsea.asia/transcribe" \
  -F "file=@audio.wav"
```

### ASR: transcribe + tags + annotate + clarified English (single call)

```bash
curl -X POST "https://api.valsea.asia/transcribe?enableTags=true&annotate=true&clarify=true" \
  -F "file=@audio.wav"
```

### Text-only post-processing (no audio upload)

```bash
curl -X POST "https://api.valsea.asia/annotate" \
  -H "Content-Type: application/json" \
  -d '{"text":"walao wait so long lah","accent":"sg","enableCorrection":true,"enableTags":true}'

curl -X POST "https://api.valsea.asia/clarify" \
  -H "Content-Type: application/json" \
  -d '{"text":"walao wait so long lah","accent":"sg"}'
```

### Translation: translate text

```bash
curl -X POST "https://translation.valsea.asia/api/translate" \
  -H "Content-Type: application/json" \
  -d '{"text":"Good morning","source":"en","target":"id"}'
```

---

## Full API spec

Read the complete, developer-facing spec here:
- [`VALSEA_API_DOCUMENTATION.md`](./VALSEA_API_DOCUMENTATION.md)
