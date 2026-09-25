# Technical Decisions (ADRs)

This document records Architecture Decision Records (ADRs) for significant technical choices made during the project.

---

## ADR-001: Frontend Framework - Next.js App Router

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need a modern React framework for the frontend with streaming support, server components, and good developer experience. The existing project is already using Next.js.

### Decision
Use Next.js 16 with App Router and React 19.

### Alternatives Considered
- **Vite + React:** Simpler but no built-in streaming/SSE support
- **Remix:** Good streaming but smaller ecosystem
- **Plain React + custom server:** Maximum control but more infrastructure work

### Consequences
- **Pros:** Server components for static layout, built-in streaming with SSE, mature ecosystem, existing project alignment
- **Cons:** Larger bundle, framework lock-in, Next.js-specific patterns

### Migration Implications
If moving away from Next.js, the App Router structure would need refactoring. Keep business logic in backend to minimize frontend coupling.

---

## ADR-002: Backend Runtime - Bun/Node.js Local Server

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need a local backend server to handle provider communication, storage, file processing, and tool execution. Must support streaming, high performance, and local execution.

### Decision
Use Bun (preferred) or Node.js for the local backend server. Single process architecture.

### Alternatives Considered
- **Deno:** Good security model but smaller ecosystem
- **Python (FastAPI):** Great for AI but not ideal for frontend integration
- **Go:** High performance but more complex TypeScript sharing

### Consequences
- **Pros:** TypeScript/JavaScript shared with frontend, high performance, native streaming support, single process simplicity
- **Cons:** Less mature than Node.js for some libraries, Bun-specific APIs

### Migration Implications
Backend can be swapped if needed; keep protocol (HTTP/SSE) stable. Business logic isolated in backend services.

---

## ADR-003: Local Database - SQLite (embedded)

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted (pending evaluation)

### Context
Need persistent local storage for sessions, messages, files, and configuration. Must support write-ahead logging, indexes, and efficient querying.

### Decision
Use SQLite with better-sqlite3 or libSQL for the embedded database. Consider Drizzle ORM for type-safe queries.

### Alternatives Considered
- **IndexedDB (browser):** No backend access, limited query capability
- **LevelDB/RocksDB:** Good performance but less SQL flexibility
- **PostgreSQL (local):** Heavy for local-first, requires separate process
- **JSON files:** Simple but no indexing, poor query performance

### Consequences
- **Pros:** Single file, WAL support, SQL queries, indexes, mature, portable
- **Cons:** Single writer limitation (mitigated by WAL), not distributed

### Migration Implications
Schema migrations handled by Drizzle or custom migration scripts. Export/backup is single file copy.

---

## ADR-004: Provider Abstraction - Unified Adapter Interface

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need to support multiple LLM providers (9Router, OpenAI, Anthropic, Google Gemini) with different APIs while maintaining a clean application interface.

### Decision
Implement a unified provider adapter interface with provider-specific implementations behind it.

```typescript
interface ProviderAdapter {
  listModels(profileId: string): Promise<ModelCatalogEntry[]>
  streamChat(params: ChatParams): AsyncIterable<ChatChunk>
  testConnection(profile: ProviderProfile): Promise<ConnectionResult>
  normalizeError(error: Error): ProviderError
}
```

### Alternatives Considered
- **Direct provider SDKs in application code:** Tight coupling, duplication
- **Single provider only:** Violates open-source requirement
- **Proxy all through 9Router:** Limits direct provider support

### Consequences
- **Pros:** Clean abstraction, easy to add providers, testable, provider-specific logic isolated
- **Cons:** Additional layer, adapter maintenance burden

### Migration Implications
New adapters implement the same interface. Existing code unchanged. Version adapters separately if needed.

---

## ADR-005: 9Router as Default Gateway

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
9Router is the recommended default gateway. It provides multi-provider routing, fallback, and unified endpoint.

### Decision
Use 9Router as the default and recommended provider. Implement OpenAI-compatible adapter for 9Router. 9Router handles provider-level routing and fallback; application only discovers and selects models.

### Alternatives Considered
- **Direct provider connections only:** More complex for users managing multiple providers
- **9Router-only:** Limits flexibility for users who want direct connections

### Consequences
- **Pros:** Simplifies multi-provider setup for users, leverages 9Router's routing/fallback, single endpoint
- **Cons:** Dependency on 9Router availability, user must run 9Router locally

### Migration Implications
9Router integration verified in Phase 0. If 9Router unavailable, direct provider profiles serve as fallback. Application must never assume 9Router is present.

---

## ADR-006: Streaming Protocol - Server-Sent Events (SSE)

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need real-time streaming from LLM providers to frontend. Must be simple, HTTP-compatible, and work with existing infrastructure.

### Decision
Use Server-Sent Events (SSE) for streaming responses. Backend forwards each provider chunk immediately with explicit flush.

### Alternatives Considered
- **WebSockets:** Bidirectional but more complex, proxy issues
- **HTTP long polling:** Higher latency, more complex
- **Next.js native streaming:** Limited to server components, less control

### Consequences
- **Pros:** Simple, HTTP-native, works through proxies, automatic reconnection, one-way streaming matches LLM pattern
- **Cons:** One-way only (no client→server during stream), connection limits in browsers

### Migration Implications
Protocol stable. Can add WebSocket alongside if bidirectional needed later.

---

## ADR-007: Credential Storage - OS Credential Store + Encrypted File Fallback

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
API keys must be stored securely. Never in plain text, never in workspace database, never exported.

### Decision
Use OS credential store (libsecret on Linux, Keychain on macOS, Credential Manager on Windows) as primary. Encrypted local file (age/argon2) as fallback when OS store unavailable.

### Alternatives Considered
- **Plain text with file permissions:** Insufficient protection
- **Encrypted database only:** No OS integration, key management complexity
- **Environment variables only:** Not user-friendly for runtime configuration

### Consequences
- **Pros:** OS-level security, no master password needed, keys never in application memory longer than necessary
- **Cons:** Cross-platform complexity, fallback needed for headless/CI environments

### Migration Implications
Key storage abstraction allows swapping implementations. Keys referenced by ID in provider profiles.

---

## ADR-008: RAG Embedding Source - Evaluated in Phase 0

**Date:** $(date +%Y-%m-%d)
**Status:** Pending (Phase 0 evaluation)

### Context
Document retrieval requires embeddings. Two options: 9Router embeddings endpoint (if available) or local embedding model.

### Decision
Deferred to Phase 0. Evaluate both options:
1. 9Router embeddings endpoint (if exists and functional)
2. Local embedding model (e.g., BGE, E5, nomic-embed-text via llama.cpp or similar)

ADR will be created based on verification results.

### Alternatives Considered
- **OpenAI embeddings:** Requires separate API key, external dependency
- **Cohere embeddings:** External dependency
- **No embeddings (keyword search):** Lower quality retrieval

---

## ADR-009: Vector Store - SQLite-VEC or Local Vector DB

**Date:** $(date +%Y-%m-%d)
**Status:** Pending (depends on ADR-008)

### Context
Need vector similarity search for RAG retrieval. Must be local and embedded.

### Decision
Use sqlite-vec (SQLite extension) if using SQLite, or ChromaDB/lancedb as standalone. Decision pending database choice and embedding approach.

### Alternatives Considered
- **pgvector:** Requires PostgreSQL
- **Qdrant:** Separate process
- **FAISS:** In-memory, persistence complexity

---

## ADR-010: File Processing - Background Workers

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
File upload, parsing, chunking, and embedding must not block the UI.

### Decision
Implement file processing pipeline in background workers (Web Workers or separate processes). Skip unchanged files via content hash. Lazy-load heavy parsers.

### Alternatives Considered
- **Synchronous in backend:** Blocks request handling
- **Async but single-threaded:** Still blocks event loop
- **Queue-based (Redis):** Overkill for local-first

### Consequences
- **Pros:** UI never blocked, progress tracking, parallel processing
- **Cons:** More complex state management, eventual consistency

---

## ADR-011: MCP Integration - Permission-Based Execution

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
MCP tools must be executed safely with user consent.

### Decision
All tool executions require explicit user permission. Three modes: Allow once, Allow for session, Deny. Tools labeled "Untrusted tool output" in context. Skills cannot change permissions.

### Alternatives Considered
- **Auto-allow trusted servers:** Security risk
- **No permission (developer mode):** Not for production
- **Pre-approved tool lists:** Complex configuration

### Consequences
- **Pros:** User control, security, audit trail
- **Cons:** Friction for frequent tool use

---

## ADR-012: Skills - Progressive Loading with Token Display

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Skills save tokens by progressive loading: names/descriptions always in context, bodies only when activated.

### Decision
Implement three-level skill loading:
- Level 1 (always): name + description (~200 chars)
- Level 2 (on activation): full SKILL.md body
- Level 3 (on demand): supporting files referenced in body

Display token estimates in UI. Max 20 enabled skills in index. Warn above 2000 tokens body, reject above 8000.

### Alternatives Considered
- **Full injection:** Simpler but token waste
- **No skills:** Less functionality
- **Model-driven only:** Requires tool calling support

---

## ADR-013: Theme Handling - data-theme Attribute with Inline Script

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need to apply theme before first paint to avoid flash. Support Light, Dark, System.

### Decision
Use `data-theme` attribute on `<html>` element. Set via inline script in `<head>` reading from localStorage/settings store. Theme tokens defined as CSS variables.

### Alternatives Considered
- **CSS media query only:** No manual override
- **Class-based (.dark):** More complex, flash risk
- **Next.js theme provider:** Adds dependency, still needs first-paint script

### Consequences
- **Pros:** No flash, single token set, simple CSS, supports System preference
- **Cons:** Inline script in head, localStorage access

---

## ADR-014: Session State in URL

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Active session should be restorable on page reload.

### Decision
Reflect active session in URL query parameter (e.g., `?session=<id>`) or hash. Landing state (no session) is default.

### Alternatives Considered
- **localStorage only:** Not shareable, no history
- **Hash routing:** Compatible with static export

### Consequences
- **Pros:** Shareable links, browser history, reload restores session
- **Cons:** URL length limits (not an issue for UUIDs)

---

## ADR-015: Component Library - shadcn/ui with DESIGN.md Tokens

**Date:** $(date +%Y-%m-d)
**Status:** Accepted

### Context
Need accessible, customizable UI components matching DESIGN.md specifications.

### Decision
Use shadcn/ui primitives as base. Map DESIGN.md CSS variables to Tailwind theme. Custom components built on top. Self-host all fonts and icons (Lucide).

### Alternatives Considered
- **Radix UI directly:** More work, no styling
- **Custom components only:** Reinventing accessibility
- **MUI/Chakra:** Heavy, hard to customize to design system

### Consequences
- **Pros:** Accessible primitives, full customization, design token integration, tree-shakable
- **Cons:** More setup, component maintenance

---

## ADR-016: Testing Strategy - Unit + Integration + E2E

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Need comprehensive testing covering provider layer, streaming, persistence, file upload, RAG, MCP, and session management.

### Decision
Three-tier testing:
- **Unit:** Provider adapters, context builder, document processing, chunking, retrieval, tool handling, session management
- **Integration:** 9Router, streaming, persistence, file upload, RAG, MCP
- **E2E:** Full flows (session creation → message → stream → restart → history), session isolation, provider/model switching, skills

Performance tests against local mock provider.

### Alternatives Considered
- **E2E only:** Slow, flaky, hard to debug
- **Unit only:** Misses integration issues
- **No tests:** Unacceptable for portfolio project

---

## ADR-017: Performance Measurement - Mock Provider + Real Provider

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Must validate latency budgets against both controlled and real conditions.

### Decision
Instrument hot path with timestamps: send pressed, request dispatched, first byte from provider, first token rendered, stream complete. Expose in developer view. Measure against local mock streaming provider (for app overhead) and at least one real provider profile.

### Alternatives Considered
- **Real provider only:** Network variance obscures app overhead
- **Mock only:** Doesn't validate real-world behavior

---

## ADR-018: Document Parser - Modular with Content Hash

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Support multiple file types without hardcoded pipelines.

### Decision
Implement document processor abstraction with pluggable parsers. Each parser handles specific types. Content hash used to skip reprocessing unchanged files. Chunking strategy configurable per document type.

### Alternatives Considered
- **Single parser with if/else:** Not extensible
- **External service:** Not local-first

---

## ADR-019: Context Building - Incremental Caching

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Context building must not re-read or re-tokenize full history every turn.

### Decision
Cache token counts per message. Keep active session's recent messages in memory. Context builder assembles request from sources incrementally. Retrieval runs in parallel with other preparation.

### Alternatives Considered
- **Full history every turn:** Simple but violates performance budgets
- **Summarization only:** Loses detail

---

## ADR-020: Error Normalization - Distinct Provider Error States

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Provider errors must be user-actionable, not generic.

### Decision
Normalize all provider errors into distinct states: unreachable, unauthorized, rate limited, quota exhausted, model not found, malformed stream, timeout. Each has specific banner with action.

### Alternatives Considered
- **Pass-through provider errors:** Inconsistent, often unclear
- **Generic error with retry:** Not actionable

---

## ADR-021: Legal Pages - Static Routes /privacy and /terms

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Required for public launch per DESIGN.md section 17.

### Decision
Implement as static Next.js pages with shared layout. No sidebar. Single reading column (720px max). Real content written by project owner, reviewed by qualified person.

### Alternatives Considered
- **External links:** Not compliant
- **Dynamic pages:** Unnecessary complexity

---

## ADR-022: Favicon - Cairn Mark SVG with Theme Support

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Required for public launch per DESIGN.md section 17.1.

### Decision
Create SVG favicon from cairn mark with `prefers-color-scheme` support for light/dark. Provide 32px PNG/ICO fallback and 180px Apple touch icon. Remove framework default icons.

### Alternatives Considered
- **Framework default:** Violates launch gate

---

## ADR-023: Secret Scanning - Pre-commit + CI

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Prevent real keys from being committed.

### Decision
Configure pre-commit hook and CI job for secret scanning. Include .gitignore for secrets and workspace. Ship .env.example with placeholders only.

### Alternatives Considered
- **CI only:** Catches after commit
- **Manual review:** Unreliable

---

## ADR-024: Package Manager - Bun

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Project uses bun per AGENTS.md.

### Decision
Use bun for all package management, installation, and script running. Lockfile committed.

### Alternatives Considered
- **npm/yarn/pnpm:** Not per project instructions

---

## ADR-025: Type Checking - TypeScript Strict Mode

**Date:** $(date +%Y-%m-%d)
**Status:** Accepted

### Context
Type safety is critical for maintainability.

### Decision
Use TypeScript strict mode. Run `bun run typecheck` after every change. Do not run build unless explicitly requested.

### Alternatives Considered
- **Loose types:** Faster development but more runtime errors
- **No typecheck:** Unacceptable for engineering project