# Provider Verification

> **Note:** This document is populated during Phase 0 verification against a running 9Router instance and each provider's current official documentation. Untested providers are listed with status.

## Provider Verification Status

| Provider | Adapter | Verified | Method | Notes |
|----------|---------|----------|--------|-------|
| 9Router | openai-compatible | Pending | To be verified | Reported default, OpenAI-compatible |
| OpenAI | openai-compatible | Pending | To be verified | Direct provider support |
| Anthropic | anthropic | Pending | To be verified | Native messages API |
| Google Gemini | google-gemini | Pending | To be verified | Native Gemini API |

## Verification Procedure
Each provider is verified during Phase 0 using this checklist:
1. API contract verified against current official documentation
2. Streaming format tested
3. Error formats documented
4. Rate-limit/quota responses documented
5. Tool/function-calling passthrough verified (if supported)
6. Embeddings endpoint verified (if available)
7. Model listing endpoint verified
8. Base URL and port confirmed

## 9Router Verification (PENDING)

### Known Claims (Unverified)
- Default URL: http://localhost:20128 with API under /v1
- OpenAI-compatible endpoint
- Upstream provider keys configured inside 9Router (not in CairnAI)
- Supports "combos": virtual model IDs with automatic fallback
- May require an API key from clients
- May expose embeddings endpoint
- Number of supported providers and models varies by version

### What to Verify
- Actual base URL and port
- Whether API key required from clients
- How models and combos are listed and what metadata they carry
- Streaming format and mid-stream errors
- Tool/function-calling passthrough
- Whether embeddings endpoint exists and offers embedding models
- Error formats, rate-limit and quota responses
- Whether responses reveal actual upstream model used
- How model IDs behave over time

### Design Consequences (Verified or Pending)
1. CairnAI holds only base URL and optional 9Router API key for 9Router profiles
2. Provider-level routing and fallback is done by 9Router (not reimplemented in CairnAI)
3. One generic OpenAI-compatible adapter serves 9Router and many others
4. Model metadata may be missing or incomplete (app-level metadata layer with safe defaults)
5. Combos can hide which upstream model answered (record requested model + answering model separately)
6. Local-first default deployment (backend + 9Router on same machine)
7. Frontend never calls 9Router or any provider directly (backend only)
8. Gateway health is first-class state (9Router not running, no providers, exhausted, rate-limited)

## OpenAI-Compatible Adapter
- **Status:** Pending verification
- **Interface:** Chat completions API
- **Streaming:** Server-Sent Events
- **Tools:** Function calling
- **Models:** GET /v1/models

## Anthropic Adapter
- **Status:** Pending verification
- **Interface:** Messages API
- **Streaming:** Server-Sent Events
- **Tools:** Tool use
- **Models:** List models endpoint

## Google Gemini Adapter
- **Status:** Pending verification
- **Interface:** Generative Language API
- **Streaming:** Server-Sent Events
- **Tools:** Function calling
- **Models:** List models endpoint

## Adapter Interface
All adapters implement this interface:

```typescript
interface ProviderAdapter {
  listModels(profileId: string): Promise<ModelCatalogEntry[]>
  streamChat(params: ChatParams): AsyncIterable<ChatChunk>
  testConnection(profile: ProviderProfile): Promise<ConnectionResult>
  normalizeError(error: Error): ProviderError
}
```

## Model Catalog
- Models come from provider's model-listing endpoint (cached, refreshed in background)
- If provider has no listing, users add model IDs manually
- Model identified by (profile id, model id) pair
- Model metadata: context window, tool support, vision support
- Unknown capabilities shown as "unknown", never assumed supported

## Provider Presets
| Preset | Adapter Type | Base URL (default) | Notes |
|--------|-------------|-------------------|-------|
| 9Router | openai-compatible | http://localhost:20128/v1 | Default/recommended |
| Generic OpenAI | openai-compatible | (user-configured) | Any OpenAI-compatible endpoint |
| OpenAI | openai-compatible | https://api.openai.com/v1 | Direct |
| Anthropic | anthropic | https://api.anthropic.com | Native |
| Google Gemini | google-gemini | https://generativelanguage.googleapis.com | Native |

Presets only prefill the form; they never contain keys.

## Verification Notes
Findings will be recorded here after Phase 0 verification:
- Actual 9Router base URL and port
- API key requirement status
- Available endpoints and capabilities
- Streaming format details
- Error format details
- Each adapter's verified behavior