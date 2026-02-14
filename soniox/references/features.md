# Soniox Features & Configuration Reference

## Table of Contents

1. [Authentication](#authentication)
2. [Models](#models)
3. [Supported Languages](#supported-languages)
4. [Language Hints](#language-hints)
5. [Language Restrictions](#language-restrictions)
6. [Language Identification](#language-identification)
7. [Speaker Diarization](#speaker-diarization)
8. [Context Customization](#context-customization)
9. [Timestamps](#timestamps)
10. [Confidence Scores](#confidence-scores)
11. [Translation](#translation)

---

## Authentication

Create account at `https://console.soniox.com/signup`. API keys are per-project: **My First Project > API Keys**.

```sh
export SONIOX_API_KEY=<YOUR_API_KEY>
```

---

## Models

### Current Models

| Model | Type | Status |
|-------|------|--------|
| `stt-rt-v4` | Real-time | **Active** -- recommended |
| `stt-async-v4` | Async | **Active** -- recommended |
| `stt-rt-v3` | Real-time | Active, auto-routes to v4 after **2026-02-28** |
| `stt-async-v3` | Async | Active, auto-routes to v4 after **2026-02-28** |

### Aliases

| Alias | Points to |
|-------|-----------|
| `stt-rt-v3-preview` | `stt-rt-v3` |
| `stt-rt-preview-v2` | `stt-rt-v3` |
| `stt-async-preview-v1` | `stt-async-v3` |

### Key Capabilities (v4)

- 60+ languages, human-parity accuracy
- Multilingual mid-sentence switching
- Audio up to 5 hours per session/file
- Improved speaker diarization, context handling, translation
- `max_endpoint_delay_ms` for end-of-speech delay control (real-time)
- `context_version: 2`

---

## Supported Languages

| Language | Code | Language | Code | Language | Code |
|----------|------|----------|------|----------|------|
| Afrikaans | `af` | Greek | `el` | Norwegian | `no` |
| Albanian | `sq` | Gujarati | `gu` | Persian | `fa` |
| Arabic | `ar` | Hebrew | `he` | Polish | `pl` |
| Azerbaijani | `az` | Hindi | `hi` | Portuguese | `pt` |
| Basque | `eu` | Hungarian | `hu` | Punjabi | `pa` |
| Belarusian | `be` | Indonesian | `id` | Romanian | `ro` |
| Bengali | `bn` | Italian | `it` | Russian | `ru` |
| Bosnian | `bs` | Japanese | `ja` | Serbian | `sr` |
| Bulgarian | `bg` | Kannada | `kn` | Slovak | `sk` |
| Catalan | `ca` | Kazakh | `kk` | Slovenian | `sl` |
| Chinese | `zh` | Korean | `ko` | Spanish | `es` |
| Croatian | `hr` | Latvian | `lv` | Swahili | `sw` |
| Czech | `cs` | Lithuanian | `lt` | Swedish | `sv` |
| Danish | `da` | Macedonian | `mk` | Tagalog | `tl` |
| Dutch | `nl` | Malay | `ms` | Tamil | `ta` |
| English | `en` | Malayalam | `ml` | Telugu | `te` |
| Estonian | `et` | Marathi | `mr` | Thai | `th` |
| Finnish | `fi` | Turkish | `tr` | Ukrainian | `uk` |
| French | `fr` | Urdu | `ur` | Vietnamese | `vi` |
| Galician | `gl` | Welsh | `cy` | German | `de` |

Retrieve programmatically via `GET /v1/models`.

---

## Language Hints

**Parameter:** `language_hints` (string[])

Biases recognition toward specified languages **without restricting** others.

```json
{"language_hints": ["en", "es"]}
```

Use when you know expected languages or audio includes similar-sounding languages.

---

## Language Restrictions

**Parameters:** `language_hints` + `language_hints_strict: true`

Strongly biases output to **only** specified languages (best-effort, not hard guarantee).

```json
{
  "language_hints": ["en"],
  "language_hints_strict": true
}
```

Single language = most robust. Multiple languages reduce accuracy with heavy accents.

---

## Language Identification

**Parameter:** `enable_language_identification: true`

Detects language at **token level** with sentence-level coherence:

```json
{"text": "How", "language": "en"}
{"text": "Cómo", "language": "es"}
```

- Embedded foreign words keep sentence's language tag
- Real-time: tags may revise as context arrives
- Combine with `language_hints` for better accuracy

---

## Speaker Diarization

**Parameter:** `enable_speaker_diarization: true`

Labels up to **15 speakers** per session:

```json
{"text": "How", "speaker": "1"}
{"text": "I am", "speaker": "2"}
```

- Async mode has significantly higher accuracy than real-time (full audio context)
- Real-time: temporary speaker switches may occur, stabilize with context
- Accuracy decreases with many similar-voiced speakers

---

## Context Customization

**Parameter:** `context` (object) -- max **8,000 tokens** (~10,000 characters)

### Sections

| Section | Type | Purpose |
|---------|------|---------|
| `general` | `{key, value}[]` | Domain, topic, participants |
| `text` | string | Free-form background text |
| `terms` | string[] | Domain-specific words/phrases |
| `translation_terms` | `{source, target}[]` | Custom translation mappings |

### Full Example

```json
{
  "context": {
    "general": [
      {"key": "domain", "value": "Healthcare"},
      {"key": "topic", "value": "Diabetes management"},
      {"key": "doctor", "value": "Dr. Martha Smith"}
    ],
    "text": "Patient diagnosed with Type 2 diabetes, on Metformin 500mg twice daily.",
    "terms": ["Metformin", "Celebrex", "Amoxicillin Clavulanate Potassium"],
    "translation_terms": [
      {"source": "Mr. Smith", "target": "Sr. Smith"},
      {"source": "MRI", "target": "RM"}
    ]
  }
}
```

### Tips

- Keep `general` to ~10 key-value pairs max
- Use `terms` for consistent spelling/casing of proper nouns
- Use `translation_terms` to preserve names/brands unchanged

---

## Timestamps

**Included by default** -- no configuration needed.

Each token includes `start_ms` and `end_ms` (milliseconds):

```json
{"text": "Beau", "start_ms": 300, "end_ms": 420}
{"text": "ti", "start_ms": 420, "end_ms": 540}
```

Granularity: **token (sub-word)** level.

---

## Confidence Scores

**Included by default**. Each token has `confidence` (float, 0.0-1.0):

```json
{"text": "Beau", "confidence": 0.82}
{"text": "ful", "confidence": 0.98}
```

Low values from: noise, accents, unclear speech, rare vocabulary.

---

## Translation

Works between **any pair** of supported languages.

### One-Way

Transcribes and translates into a single target language.

### Two-Way

Bidirectional between two languages in real-time.

### Availability

- **Real-time API:** streaming transcription + translation
- **Async API:** file-based transcription + translation

Use `translation_terms` in context to control specific term translations.

---

## Quick Parameter Reference

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model` | string | -- | `stt-rt-v4` or `stt-async-v4` |
| `language_hints` | string[] | `[]` | ISO codes to bias recognition |
| `language_hints_strict` | boolean | `false` | Restrict to hinted languages |
| `enable_language_identification` | boolean | `false` | Tag tokens with language |
| `enable_speaker_diarization` | boolean | `false` | Tag tokens with speaker ID |
| `context` | object | `null` | Context (general/text/terms/translation_terms) |
| `max_endpoint_delay_ms` | number | -- | End-of-speech delay (v4 RT only) |
