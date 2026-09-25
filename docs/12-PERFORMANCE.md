# Performance

## Overview
Performance is a core requirement. When using CairnAI, the application itself must never feel slow. The goal is zero perceptible application latency; network time and model inference time at the provider are outside the application's control.

## Hot Paths
1. Opening the app
2. Pressing send until the first token is on screen
3. Streaming render
4. Switching sessions
5. Typing in the composer
6. Opening the model picker
7. Uploading a file (the UI stays responsive)
8. Scrolling long conversations

## Latency Budgets

| Metric | Target | Notes |
|--------|--------|-------|
| Key press to visible feedback | < 100ms | All interactive feedback |
| Typing (keypress to pixels) | Within one frame (16ms) | Text input responsiveness |
| Send to request leaving machine | < 20ms | App-added time only |
| Per streamed chunk (received to pixels) | < 1 frame | No buffering |
| First-token overhead (vs direct provider) | < 30ms | Application overhead |
| Session switch (loaded) | < 100ms | Cached sessions |
| Session switch (any session) | < 200ms | Even long history |
| Cold start to interactive UI | < 2s | Initial load |
| Warm start to interactive UI | < 1s | Subsequent loads |
| Opening model picker | < 50ms | From cache, no network |
| Activating/deactivating skill | < 50ms | Index cached |
| Streaming with code blocks | 60 fps sustained | No stutter |

## Measurement Methodology
- **Local mock provider:** Measures app overhead without network variance
- **Real provider (9Router):** Validates end-to-end performance
- **Timestamp instrumentation:** Send pressed → Request dispatched → First byte from provider → First token rendered → Stream complete

## Architecture Requirements for Performance

### Streaming Path
1. Pass chunks through as they arrive
2. Forward each provider chunk to frontend immediately with explicit flush
3. No buffering by middleware, compression, or proxies on event streams
4. Reuse connections (keep-alive, connection pooling, HTTP/2)
5. Stop cancels the upstream request and stops rendering within one frame

### Send Path
1. Before the request leaves, do only what changes the request
2. Persist user message in parallel with provider request (not blocking)
3. Mark turn complete only after message is durable
4. Buffer assistant tokens in memory, flush to storage in batches
5. Context building: incremental and cached (cache per-message, keep recent messages in memory)
6. Retrieval: runs only when needed, in parallel with preparation, under hard time budget
7. Optimistic UI: user message appears instantly, placeholder immediately after

### Background Work
1. File parsing, chunking, embedding in background workers with progress
2. Chat and UI never wait on background work
3. Skip unchanged files using content hash
4. Lazy-load heavy parsers
5. Initialize MCP servers, parsers, embedding models lazily or during idle time

### Storage
1. Local embedded database with write-ahead logging
2. Indexes for hot queries (sessions by recency, messages by session+order)
3. Prepared statements
4. No N+1 queries
5. Windowed loading of long histories
6. Index-based search for sessions and messages

### Frontend
1. Virtualize message list and session list
2. Render streaming text incrementally
3. Batch DOM updates to at most one per animation frame
4. Memoize completed blocks
5. Do not re-parse or re-render whole message for every token
6. Defer or incrementally apply syntax highlighting
7. Keep initial bundle small (code-split drawer, settings, heavy libraries)
8. Keep main-thread tasks under 50ms
9. Move heavy work to workers
10. Cache model lists per provider profile (stale-while-revalidate)
11. No network call blocks model picker opening

## Performance Instrumentation
- Hot-path timestamps exposed in developer view or logs
- Never log message content
- Benchmark script available
- Automated budget checks (regression detection)
- Results recorded in this document

## Current Measurements
| Budget | Target | Measured | Status |
|--------|--------|----------|--------|
| Key press to feedback | < 100ms | Not measured | Not measured |
| Send to dispatch | < 20ms | Not measured | Not measured |
| Per chunk render | < 1 frame | Not measured | Not measured |
| First-token overhead | < 30ms | Not measured | Not measured |
| Session switch (loaded) | < 100ms | Not measured | Not measured |
| Session switch (long history) | < 200ms | Not measured | Not measured |
| Cold start | < 2s | Not measured | Not measured |
| Warm start | < 1s | Not measured | Not measured |
| Model picker open | < 50ms | Not measured | Not measured |
| Skill activate/deactivate | < 50ms | Not measured | Not measured |
| Streaming code blocks | 60fps | Not measured | Not measured |
| File upload responsiveness | UI responsive | Not measured | Not measured |
| Scrolling long conversations | Smooth | Not measured | Not measured |

## Optimization Techniques
- **Connection warming:** Warm connection to default provider at startup and when composer gains focus
- **Incremental context:** Cache token counts per message
- **Background processing:** Workers for file parsing/embedding
- **Lazy loading:** Heavy parsers, MCP servers, embedding models
- **Virtualized lists:** Both session list and message list
- **Code splitting:** Drawer, settings, heavy libraries split from main bundle
- **Batching:** DOM updates batched per animation frame
- **Memoization:** Completed message blocks memoized
- **Stale-while-revalidate:** Model lists cached with background refresh

## Regression Detection
- Automated performance tests in CI
- Budget violations fail builds
- Historical trending of measurements
- Alerting on regression thresholds

## Notes
- Numbers shown in README must be measured, not claimed
- Test setup must be documented
- "App overhead" is the metric that matters (network/provider latency excluded)
- Measurements must be reproducible
- The 9Router findings will be recorded once verified against a running instance