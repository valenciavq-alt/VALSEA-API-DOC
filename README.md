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

## Recommended integration flow (matches the frontend)

1. **ASR (upload audio)** → get `rawTranscript` then `text`
2. **Semantic tags / annotated text** *(optional)* → `semantic_tags`, `annotated_text`
3. **Clarified English** *(optional)* → `clarified_text`
4. **Translation** *(optional)* → `translatedText`
5. **Use-case output** *(optional)* → workflow structured JSON

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
