# Architecture Document

## Architecture Overview
CairnAI is a modular personal AI workspace application consisting of a frontend web application, a local backend server, and integration layers for LLM providers, file processing, and tool systems. The architecture follows a clean module boundary pattern where each module exposes a public interface and communicates only through defined contracts.

## Architectural Goals
1. **Local-First:** All user data resides locally; backend only communicates with providers for inference
2. **Modular Boundaries:** Each component has clear responsibilities and public interfaces
3. **Provider Agnosticism:** The application abstracts over provider implementations through a unified adapter interface
4. **Streaming-First:** The architecture is designed around streaming responses from LLM providers
5. **Performance:** Hot-path operations meet strict latency budgets; the app adds zero perceptible latency
6. **Security:** Credentials never reach the frontend; backend handles all provider communication

## Architectural Principles
1. **One module at a time:** Modules are built and verified sequentially
2. **Interface-first:** Public interfaces are defined before implementation
3. **Dependency direction:** UI → Application Services → Modules → Storage/Gateways (no cycles)
4. **No reaching into internals:** Modules communicate only through public interfaces
5. **Small files:** Single-purpose files; avoid files over ~300 lines
6. **Lazy initialization:** Heavy resources initialize on-demand or during idle

## System Context
```
┌─────────────────────────────────────────────────┐
│                  User's Machine                  │
│                                                  │
│  ┌──────────────┐    ┌──────────────────────┐    │
│  │  Frontend    │◄──►│  Backend Server      │    │
│  │  (Next.js)   │    │  (Bun/Node)          │    │
│  └──────┬───────┘    └──────────┬───────────┘    │
│         │                       │                 │
│  ┌──────▼───────┐    ┌──────────▼───────────┐    │
│  │  Local       │    │  LLM Gateway        │    │
│  │  Workspace   │    │  (Provider Adapter) │    │
│  │  Storage     │    │                     │    │
│  └──────────────┘    └──────────┬───────────┘    │
│                                 │                 │
│                    ┌────────────▼─────────┐       │
│                    │  9Router / Providers │◄──────┘
│                    │  (External)          │
│                    └──────────────────────┘
└─────────────────────────────────────────────────┘
```

## High-Level Architecture
The application is divided into the following major components:

**Frontend Layer:** Next.js application with React, rendering the UI shell, sidebar, composer, and context drawer. Communicates exclusively with the backend via HTTP API calls. Never touches LLM providers directly.

**Backend Layer:** Local server process that handles all business logic, storage operations, provider communication, file processing, and tool execution. This is the only component that communicates with external LLM providers.

**Storage Layer:** Local embedded database with structured storage for sessions, conversations, messages, files, and configuration. Supports persistence across restarts and provides efficient querying.

**Gateway Layer:** Provider abstraction layer that normalizes different provider APIs into a unified interface. Handles 9Router integration, model listing, streaming, and error normalization.

## Component Architecture

### Frontend Components
- **App Shell:** Navigation container with sidebar, main canvas, and optional context drawer
- **Sessions Sidebar:** Session list, search, new session, provider status, settings
- **Main Canvas:** Landing state or Conversation state with messages and composer
- **Composer:** Text input with attachment, skill, model, and send controls
- **Context Drawer:** Files, Retrieval, Tools, Skills, Model tabs
- **UI Components:** Reusable components following DESIGN.md tokens

### Backend Components
- **API Server:** REST/streaming endpoints for frontend communication
- **Session Service:** Create, read, update, delete sessions; manage session metadata
- **Conversation Service:** Message persistence, history retrieval, session switching
- **Provider Service:** Profile management, model catalog, connection testing
- **Streaming Service:** Manages LLM response streaming with cancellation
- **File Service:** Upload handling, validation, storage, and processing coordination
- **RAG Service:** Document chunking, embedding, retrieval, and context building
- **MCP Service:** Server management, tool discovery, execution orchestration
- **Context Builder:** Constructs LLM requests from system, session, conversation, RAG, tool, and skill contexts
- **Skills Service:** Skill indexing, activation management, token estimation

### Storage Components
- **Workspace:** Root directory for all local data
- **Session Store:** Session metadata, titles, creation dates, archival status
- **Message Store:** Conversation messages indexed by session
- **File Store:** Uploaded file storage with metadata and processing status
- **Config Store:** Provider profiles, default model, theme preferences, secrets
- **Vector Store:** Optional embedding storage for retrieval (implementation TBD)

## Frontend Architecture
The frontend is a Next.js application (App Router) using React 19 with TypeScript. UI components are built using shadcn/ui primitives with Tailwind CSS styling following the DESIGN.md design tokens.

**Rendering Strategy:**
- Server components for static layout elements
- Client components for interactive features (composer, sidebar, drawer)
- Streaming responses rendered incrementally with React Suspense patterns
- Virtualized message lists and session lists for performance
- Theme handling via `data-theme` attribute with system preference detection

**State Management:**
- Backend-driven state; frontend fetches on navigation
- Optimistic UI for message sending
- Session state managed via URL query/hash for persistence across reloads
- Context drawer content loaded on demand

**Key Constraint:** The frontend never calls LLM providers directly. All AI communication routes through the backend.

## Backend Architecture
The backend runs as a local server process (Bun/Node.js) serving the API for the frontend. It handles all business logic, provider communication, and data management.

**Process Layout:** Single local backend process with frontend served locally. No microservices, no unnecessary hops between UI, backend, and provider.

**API Design:**
- REST endpoints for CRUD operations (sessions, files, providers)
- Server-Sent Events (SSE) or compatible streaming for LLM responses
- WebSocket alternative for real-time status updates
- All endpoints served from the local backend only

**Security Boundaries:**
- Frontend → Backend: HTTP calls with authentication tokens (session-based)
- Backend → Providers: Direct HTTP calls with credentials from secure storage
- No credentials ever reach the frontend
- Base URL validation prevents redirect attacks

## LLM Gateway Architecture
The gateway layer abstracts all LLM provider interactions through a unified adapter interface.

```
Application
    │
    v
LLM Gateway Interface
    │
    ├── OpenAI-Compatible Adapter (9Router, generic providers)
    ├── Anthropic Adapter
    ├── Google Gemini Adapter
    └── [Future: Additional adapters]
```

**Adapter Interface:**
- `listModels(providerId): Promise<ModelCatalogEntry[]>`
- `streamChat(params): AsyncIterable<Chunk>`
- `testConnection(profile): Promise<ConnectionResult>`
- `normalizeError(error): ProviderError`

**Provider Profiles:**
Each profile contains: id, display name, adapter type, base URL, credential reference, optional headers, enabled flag, per-model metadata overrides.

**9Router-Specific Handling:**
- Holds only base URL and optional API key
- Upstream provider keys stay inside 9Router
- Model IDs include combos (virtual model IDs grouping multiple models)
- Reports actual upstream model used when available

## Streaming Architecture
Streaming is a first-class architectural concern. The architecture is designed around real-time chunk delivery.

**Streaming Path:**
1. User sends message → frontend shows instantly (optimistic UI)
2. Frontend sends request to backend
3. Backend forwards to provider via adapter
4. Provider streams chunks → backend receives each chunk
5. Backend forwards each chunk to frontend immediately
6. Frontend renders each chunk incrementally
7. Backend persists message in parallel (not blocking)

**Key Rules:**
- No buffering by middleware on the stream path
- Each chunk forwarded with explicit flush
- Stop cancels upstream request and stops rendering within one frame
- Connections reused (keep-alive, connection pooling)
- App-added time per chunk: under one frame

## Workspace Architecture
The workspace is the local application data directory. It contains all user data, sessions, files, configuration, and metadata.

**Workspace Structure:**
```
workspace/
├── sessions/
│   ├── {session-id}/
│   │   ├── session.json
│   │   ├── messages.db (or JSONL)
│   │   └── attachments/
│   └── ...
├── files/
│   └── {session-id}/
├── config/
│   ├── providers.json
│   ├── settings.json
│   └── secrets/ (encrypted)
├── skills/
│   ├── global/
│   └── workspace/
└── metadata.json
```

**Persistence:** Workspace survives application restarts, backend restarts, and computer restarts. Backup/export capability included. Migration path for future format changes.

## Session Architecture
Sessions are first-class entities in the application. Each session encapsulates a conversation, associated files, tool history, and configuration.

**Session Entity:**
- Unique identifier (UUID)
- Title (editable)
- Creation timestamp
- Last modified timestamp
- Archival status
- Associated model configuration
- List of attached file references
- Active skills per session

**Session Manager:** Creates, reads, updates, deletes sessions. Manages session switching with instant restoration. Handles archiving and deletion with confirmation.

**Session Switching:** Loads session data from storage on switch. Restores all messages, file associations, and configuration. Never destroys previous session state.

## Conversation Architecture
Conversations consist of an ordered sequence of messages within a session.

**Message Entity:**
- Unique identifier
- Session ID (foreign key)
- Role (user/assistant/system)
- Content (text + attachments)
- Timestamp
- Metadata (model used, provider, tokens, citations)

**Conversation Flow:**
1. User sends message → persisted immediately
2. Backend processes with context builder
3. LLM streams response → each chunk rendered
4. Final response persisted with metadata
5. Context builder caches token counts incrementally

**Context Builder:** Separates system instructions, conversation history, session context, selected files, active skills, retrieved documents, tool definitions, and model configuration into a unified request context.

## File Processing Architecture
Files uploaded by users are processed through a pipeline: Upload → Validation → Storage → Metadata → Processing → Content Extraction → Chunking → Optional Embedding → Retrieval.

**Upload:** Drag-and-drop or file picker. Dashed accent overlay during upload. Validates file type and size.

**Processing:** Runs in background workers. Parsers handle PDF, TXT, MD, DOCX, JSON, CSV, source code. Content hash used to skip unchanged files.

**Storage:** Files stored per-session in workspace directory. Never automatically shared across sessions. Explicit import action required for cross-session sharing.

**Chunking:** Text split into manageable chunks with overlap for retrieval. Configurable chunk size and overlap.

## RAG Architecture
Retrieval-Augmented Generation provides context from documents without injecting all documents into every request.

**RAG Modes:**
- **Off:** No retrieval
- **Selected Files:** Only explicitly attached files
- **Auto:** Automatic retrieval based on query relevance

**Flow:**
1. User query → Context Builder determines retrieval need
2. Retriever queries vector store for relevant passages
3. Passages ranked by relevance
4. Context Builder assembles prompt with cited passages
5. LLM generates response with citations visible to user

**Embedding Source:** Evaluated in Phase 0 against 9Router embeddings endpoint vs local embedding model. Decision documented in ADR.

**Citation Display:** Numbered chips inline in responses. Clicking opens popover with file name, location, and passage. Passage labeled "Untrusted document content."

## MCP Architecture
Model Context Protocol integration enables tool usage through external servers.

**MCP Layer:**
- Server configuration and management
- Tool discovery per server
- Tool execution with permission prompts
- Result incorporation into conversation context

**Permission Model:**
- **Ask:** Prompt user before execution
- **Allow:** Allow for this session
- **Deny:** Block execution

**Security:** No arbitrary tool execution without explicit user permission. Tools labeled "Untrusted tool output." Tool permissions cannot be changed by skills.

## Tool Execution Architecture
Tools are executed through a dedicated execution layer that manages permissions, results, and error handling.

**Flow:**
1. Tool call requested by model or user
2. Permission prompt displayed (tool name, server, arguments)
3. User chooses Allow once, Allow for session, or Deny
4. If allowed: tool executed, result captured
5. Result added to conversation context with "Untrusted tool output" label
6. Errors handled gracefully with user-visible status

## Data Architecture
The data model includes the following entities with clear relationships:

- **Workspace** → contains → Sessions
- **Session** → contains → Messages, Attachments, Tool History
- **Attachment** → has → Documents → has → Chunks → has → Embeddings
- **Provider Profile** → has → Model Catalog Entries
- **MCP Server** → has → Tools
- **Skill** → has → Sessions (many-to-many via activation)

**Storage Strategy:** Embedded local database with write-ahead logging, indexes for hot queries (sessions by recency, messages by session and order), prepared statements, no N+1 queries, windowed loading.

## Context Management
Context is built by composing multiple sources into a unified LLM request.

**Context Sources:**
1. System Instructions (application behavior, copy rules)
2. Conversation History (with incremental caching)
3. Session Context (model, skills, files)
4. RAG Context (retrieved passages)
5. Tool Context (available tools and results)
6. Skill Context (activated skill bodies)

**Context Builder Logic:**
- Incremental token counting per message
- Active session's recent messages cached in memory
- Never re-reads or re-tokenizes full history every turn
- Retrieval runs in parallel with other request preparation
- Hard time budget for retrieval (typically under 100ms)

## Security Boundaries
**Credential Protection:**
- API keys stored in OS credential store or encrypted local file
- Never displayed, logged, exported, or committed
- Backend-only access
- Base URL validation prevents redirect attacks

**Content Safety:**
- Uploaded documents marked as untrusted
- Tool outputs labeled as untrusted
- Retrieved passages labeled as untrusted
- Skill bodies reviewed before enabling (imported skills)
- Path traversal protection for file operations

**Network Security:**
- Frontend never calls providers directly
- Backend validates all provider base URLs
- HTTP warnings for non-local hosts
- No secrets in commits (pre-commit secret scan)

## Error Handling
**Normalized Provider Error States:**
- Unreachable (provider not responding)
- Unauthorized (API key rejected)
- Rate Limited (upstream limit hit)
- Quota Exhausted
- Model Not Found
- Malformed Stream
- Timeout

**Error Display:**
- Each error state has a distinct banner with icon, plain message, and action
- Never generic "something went wrong"
- Error messages never leak keys or file contents
- Specific actions provided (Open Settings, Retry, Change model, etc.)

**Recovery:**
- Partial message storage prevents data loss
- Crashes leave recoverable partial messages
- Backend restart restores workspace state
- Stream interruption allows Continue/Regenerate

## Observability
**Structured Logging:**
- Request start/end timestamps
- Model call events
- Streaming state changes
- Latency measurements
- File processing events
- Retrieval events
- Tool execution events
- Session operations

**Constraints:**
- Never log API keys
- Avoid logging sensitive file contents or private data
- Timestamps exposed in developer view/logs
- No message content in logs

## Scalability Considerations
- Single backend process (no microservices)
- One session never slows another
- Background workers for file processing and embedding
- Lazy initialization of MCP servers, parsers, embedding models
- Indexed storage for efficient querying
- Windowed loading for long conversation histories
- Cached model lists per provider profile

## Performance Considerations
**Hot Path Latency Budgets:**
- Keypress feedback: < 100ms
- Send to request dispatch: < 20ms
- Per-stream chunk render: < 1 frame
- First-token overhead: < 30ms
- Session switch (loaded): < 100ms
- Cold start: < 2 seconds

**Architecture Decisions for Performance:**
- Streaming path: pass-through with explicit flush, no buffering
- Send path: persist message in parallel with provider request
- Context building: incremental and cached
- Retrieval: parallel, under hard time budget
- Frontend: virtualized lists, incremental rendering, batch DOM updates

## Deployment Architecture
**Default:** Local-first deployment. Backend and 9Router on the same machine. Frontend served locally by backend.

**Remote Consideration:** If deployed remotely, cannot reach localhost 9Router unless deliberately exposed. Documented as a constraint.

**Development:** Next.js dev server on port 4000 (or configured port). Backend server co-located.

## Technology Choices
| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Frontend | Next.js 16 (App Router) | Server components, streaming support, mature ecosystem |
| UI Framework | React 19 | Concurrent features for streaming |
| Styling | Tailwind CSS | Design token integration, responsive |
| Components | shadcn/ui | Accessible, customizable primitives |
| Icons | Lucide | Bundled, 1.5px stroke, consistent |
| Fonts | Fraunces + Geist (self-hosted) | Open-source, matches design system |
| Backend | Bun/Node.js | High performance, native streaming |
| Database | TBD (SQLite/local embedded) | Local-first, WAL, indexes |
| Streaming | SSE / native streams | Real-time chunk delivery |
| Forms | react-hook-form + zod | Validation, type safety |
| State | Backend-driven + URL state | Persistence across reloads |
| Testing | TBD | Unit, integration, E2E |
| Monitoring | Structured logging + timestamps | Observability without overhead |

## Architectural Trade-offs
1. **Local Storage vs Cloud Sync:** Local storage chosen for privacy and simplicity. Sync deferred as extension point.
2. **Single Backend vs Microservices:** Single process for simplicity and performance. Extension point noted for future scaling.
3. **Embedded Database vs File System:** Embedded database chosen for query flexibility. File system used for actual file storage.
4. **SSE vs WebSocket:** SSE chosen for streaming simplicity and HTTP compatibility.
5. **Client-Side Rendering vs SSR:** Hybrid approach. Server components for static layout, client components for interactivity.

## Future Extension Points
- Additional provider adapter types
- In-app local model inference
- Multimodal models (image, audio)
- Voice input
- Web search via MCP
- Agents and scheduled tasks
- Memory/persistence across sessions
- Additional MCP servers
- Plugin systems
- Model benchmarking
- Token/cost analytics
- Optional cloud synchronization
- Authentication/authorization layer

## Architecture Decision Summary
Key decisions made in Phase 0:
1. **Frontend:** Next.js App Router with React 19 (existing project)
2. **Backend:** Local server process (Bun/Node.js)
3. **Storage:** Local embedded database (implementation TBD, SQLite candidate)
4. **Streaming:** Server-Sent Events with explicit flush
5. **9Router:** Default gateway with OpenAI-compatible adapter
6. **Security:** Backend-only provider communication, OS credential store
7. **RAG:** Modular pipeline with configurable embedding source
8. **MCP:** Permission-based tool execution layer
9. **Context Builder:** Incremental caching with token estimation

## Repository Structure
```
cairnai/
├── docs/                    # Engineering documentation (this file set)
│   ├── 00-BRD.md
│   ├── 01-ARCHITECTURE.md
│   ├── 02-REQUIREMENTS.md
│   ├── 03-TECHNICAL-DECISIONS.md
│   ├── 04-DATA-MODEL.md
│   ├── 05-API-SPECIFICATION.md
│   ├── 06-RAG-ARCHITECTURE.md
│   ├── 07-MCP-ARCHITECTURE.md
│   ├── 08-SECURITY.md
│   ├── 09-UI-ARCHITECTURE.md
│   ├── 10-IMPLEMENTATION-PLAN.md
│   ├── 11-TESTING-STRATEGY.md
│   ├── 12-PERFORMANCE.md
│   └── 13-PROVIDERS.md
├── src/
│   ├── app/                 # Next.js App Router pages and layouts
│   ├── components/          # UI components (shadcn/ui + custom)
│   │   ├── ui/             # Base shadcn/ui components
│   │   └── [custom]/       # Application-specific components
│   ├── hooks/              # Custom React hooks
│   └── lib/                # Library utilities
├── backend/                # Backend server code (if separate)
├── public/                 # Static assets
├── skills/                 # Starter skills
│   └── starter/
├── workspace/              # Local workspace (created on first run)
├── docs/                   # (duplicate for docs folder - see above)
├── .env.example            # Environment variable template
├── .gitignore              # Git ignore rules
├── package.json            # Dependencies
├── DESIGN.md               # Design system specification
├── SPEC.txt                # Project execution specification
├── PROGRESS.md             # Project progress tracking
├── RULES.md                # AI boundaries and rules
├── CLAUDE.md               # Claude Code instructions pointer
└── README.md               # Project overview
```

## Key Application Flows

### Send Message and Stream
1. User types message and presses Enter (or Send)
2. Frontend optimistically renders user message
3. Frontend sends POST /api/chat/stream to backend
4. Backend validates, builds context, calls LLM adapter
5. Adapter streams chunks from provider
6. Backend forwards each chunk to frontend via SSE
7. Frontend renders each chunk incrementally
8. Backend persists user message in parallel with streaming
9. Final assistant message persisted with metadata

### Persist Session
1. Session created/modified → stored in session store
2. Messages persisted to message store indexed by session ID
3. Metadata (model, tokens, files) stored with message
4. File attachments stored per-session with processing status
5. On restore: session metadata + messages loaded by session ID

### Switch Session
1. User selects session from sidebar
2. Session Manager loads session data from storage
3. All messages, files, and configuration restored
4. UI renders Conversation state with restored content
5. Switching is instant; previous session preserved

### Upload File to RAG
1. User attaches file via composer or context drawer
2. File validated (type, size)
3. Stored per-session in workspace
4. Background worker processes: parsing → chunking → embedding
5. Chunks indexed in vector store
6. File chip appears in context drawer with status

### MCP Tool Call
1. Model requests tool execution
2. Backend receives tool call via adapter
3. Permission prompt displayed to user
4. User approves (Allow once/session/deny)
5. Tool executed via MCP client
6. Result captured, labeled "Untrusted tool output"
7. Result incorporated into conversation context

### Add Provider and Change Model
1. User opens Settings → Providers
2. Adds provider profile (preset or custom)
3. Tests connection → verifies base URL
4. Fetches model catalog (if available)
5. Model selected from catalog or added manually
6. Model change applies to next message (mid-session allowed)
7. History preserved across model changes