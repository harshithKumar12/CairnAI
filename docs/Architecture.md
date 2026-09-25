# Architecture

## 1. Overview
CairnAI is a modular personal AI workspace application consisting of a frontend web application, a local backend server, and integration layers for LLM providers, file processing, and tool systems. The architecture follows a clean module boundary pattern where each module exposes a public interface and communicates only through defined contracts.

## 2. System Architecture
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

## 3. Component Architecture

### Frontend Layer
**Role:** Render the UI, handle user interaction, communicate with backend.

**Components:**
- **App Shell:** Navigation container with sidebar, main canvas, optional context drawer
- **Sessions Sidebar:** Session list, search, new session, provider status, settings
- **Main Canvas:** Landing state or Conversation state with messages and composer
- **Composer:** Text input with attachment, skill, model, and send controls
- **Context Drawer:** Files, Retrieval, Tools, Skills, Model tabs
- **UI Components:** Reusable components following DESIGN.md tokens

**Key Constraint:** The frontend never calls LLM providers directly. All AI communication routes through the backend.

### Backend Layer
**Role:** Handle all business logic, storage operations, provider communication, file processing, and tool execution.

**Components:**
- **API Server:** REST/streaming endpoints for frontend communication
- **Session Service:** Create, read, update, delete sessions; manage session metadata
- **Conversation Service:** Message persistence, history retrieval, session switching
- **Provider Service:** Profile management, model catalog, connection testing
- **Streaming Service:** Manages LLM response streaming with cancellation
- **File Service:** Upload handling, validation, storage, processing coordination
- **RAG Service:** Document chunking, embedding, retrieval, context building
- **MCP Service:** Server management, tool discovery, execution orchestration
- **Context Builder:** Constructs LLM requests from system, session, conversation, RAG, tool, and skill contexts
- **Skills Service:** Skill indexing, activation management, token estimation

### Storage Layer
**Role:** Local persistent storage for all application data.

**Components:**
- **Workspace:** Root directory for all local data
- **Session Store:** Session metadata, titles, creation dates, archival status
- **Message Store:** Conversation messages indexed by session
- **File Store:** Uploaded file storage with metadata and processing status
- **Config Store:** Provider profiles, default model, theme preferences, secrets
- **Vector Store:** Optional embedding storage for retrieval (implementation TBD)

## 4. Repository Structure
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

## 5. Key Application Flows

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

## 6. Module Map
The module map is defined in docs/01-ARCHITECTURE.md section 33. Each module has:
- Name
- Responsibility
- Public interface
- Dependencies
- Folder

## 7. Technical Stack
- **Frontend:** Next.js 16 (App Router) with React 19 and TypeScript
- **UI Framework:** shadcn/ui with Tailwind CSS
- **Icons:** Lucide React (self-hosted)
- **Fonts:** Fraunces (display), Geist (interface), Geist Mono (technical) - self-hosted
- **Backend:** Bun/Node.js local server
- **Database:** Local embedded database (SQLite candidate) with WAL
- **Streaming:** Server-Sent Events
- **Forms:** react-hook-form + zod
- **State:** Backend-driven + URL state
- **Testing:** Unit, integration, end-to-end tests
- **Monitoring:** Structured logging + timestamps

## 8. Design System
The UI follows DESIGN.md exactly:
- Warm paper canvas with cool tones
- Fraunces serif for headlines, Geist sans for interface
- One accent color (blue) used sparingly
- Small, deliberate radii (no pills)
- Three gradients only (Shoreline, Deep water, Canvas fade)
- No purple gradients, no pill shapes, no fake metrics, no emoji, no em dashes
- No scroll or cursor animations
- Self-hosted fonts and icons

## 9. Security Architecture
- Backend-only provider communication
- API keys in OS credential store or encrypted local file
- Frontend never calls providers directly
- Uploaded documents and tool outputs labeled untrusted
- Path traversal protection
- Base URL validation
- No secrets in commits (pre-commit secret scan)