# Project Requirements Document (PRD)

## 1. Product Overview
CairnAI is a personal AI workspace that acts as a persistent LLM client. It provides a dedicated application for interacting with local and remote AI providers, with a focus on local-first operation, persistent sessions, and a calm, deliberate user experience.

**Core product statement:** A personal AI workspace with multi-model routing, persistent conversations, document RAG, streaming inference, file interaction, session-based history, local project/workspace storage, and tool/MCP integration.

## 2. Target Users
### Primary Target Users
- **Technical Professionals:** Developers, data scientists, researchers, and engineers who frequently interact with LLMs
- **Power Users:** Individuals who require persistent AI conversations for ongoing projects or learning
- **Privacy-Conscious Users:** Those who prefer to keep conversation data local rather than on external servers

### Secondary Target Users
- **Students and Educators:** Using AI for learning assistance and research
- **Writers and Content Creators:** Using AI for ideation and drafting
- **Business Professionals:** Using AI for analysis and decision support

## 3. Product Goals
1. **Local-First Persistence:** Maintain a persistent local workspace for all sessions, conversations, files, and configuration
2. **Multi-Provider Access:** Connect to 9Router (default) or any configured provider with the user's own API key
3. **Multi-Model Routing:** Discover and select models across providers with per-session and per-message selection
4. **Streaming Inference:** Stream responses in real-time with proper error handling
5. **Document Interaction:** Upload files, process them, and use retrieval-augmented generation (RAG)
6. **Tool/MCP Integration:** Integrate with MCP servers for tool usage with explicit user permission
7. **Session-Based History:** Maintain independent sessions with complete history
8. **Performance:** The application must never feel slow; zero perceptible application latency
9. **Security:** API keys and sensitive data never exposed to the frontend
10. **Open Source:** Anyone can clone the project, add their own API key, and start chatting

## 4. Product Non-Goals
- Building a competitive consumer chatbot platform
- Providing hosted/cloud-based AI model inference
- Creating proprietary LLMs or training infrastructure
- Multi-user collaboration in the initial release
- Billing, subscription, or usage tracking systems
- Mobile applications as a primary target (desktop-first)

## 5. Core Features

### 5.1 Sessions and Conversation History
- Create, rename, switch, archive, and delete independent sessions
- Each session has its own conversation history, file attachments, and configuration
- Session switching is instantaneous and never discards previous state
- Search/filter sessions by title and message content
- Session history survives application restarts, backend restarts, and computer restarts

### 5.2 Persistent Local Workspace
- Dedicated local workspace directory for all application data
- Sessions, conversations, files, metadata, and configuration stored locally
- Workspace location configurable
- Backup/export capability
- Migration path for future schema changes

### 5.3 LLM Provider Integration
- Unified provider abstraction interface
- 9Router as the default and recommended gateway
- Other providers supported through provider profiles (OpenAI-compatible, Anthropic, Google Gemini)
- Runtime provider management: add, edit, test, disable, remove
- Presets only prefill the form; presets never contain keys
- Provider-level routing and fallback handled by 9Router (not reimplemented in CairnAI)

### 5.4 Multi-Model Support
- Models discovered from configured providers
- Model selection at application default, per-session, and per-message levels
- Changing models never loses or rewrites history
- Model identified by (profile id, model id) pair
- Model metadata (context window, tool support, vision) from provider, overrides, or unknown

### 5.5 Streaming Responses
- Real-time streaming of LLM responses
- Incremental rendering with proper formatting (markdown, code blocks, tables, blockquotes)
- Stop and regenerate capabilities
- Error states handled gracefully with user-visible indicators
- Streaming is first-class, not faked with artificial delays

### 5.6 File Upload and Attachment
- Support PDF, TXT, Markdown, DOC/DOCX, JSON, CSV, and source code files
- Files associated with specific sessions (explicit ownership)
- No automatic cross-session context leakage
- Background processing (parsing, chunking, embedding) with progress tracking
- Content hash used to skip unchanged files

### 5.7 Document RAG
- Retrieval modes: Off, Selected Files, Auto
- Contextual retrieval from attached documents
- Numbered citation chips inline in responses
- Retrieved passages labeled "Untrusted document content"
- Retrieval runs in parallel with other request preparation under hard time budget

### 5.8 MCP / Tool Integration
- MCP server configuration and management
- Server discovery and tool discovery
- Tool invocation with explicit user permission
- Tool results incorporated into conversation context
- Tools labeled "Untrusted tool output"
- Permission controls: Ask, Allow for this session, Deny

### 5.9 Skills System
- Skill folders with SKILL.md files
- Workspace skills, global skills, and starter skills
- Progressive loading: name+description always, body on activation, supporting files on demand
- Activation by user (/ picker) or model (load_skill tool when supported)
- Per-session activation
- Token cost display for index and active skills
- Compact action for long sessions (summarize older messages into brief)

### 5.10 Local Workspace Storage
- Dedicated local application data directory
- Sessions, conversations, files, metadata, and configuration
- Persistence across restarts
- Backup/export without sensitive credentials
- Migration support

## 6. User Experience Requirements
- **Design System:** Follow DESIGN.md exactly (colors, typography, spacing, components)
- **Tone:** Calm, private, composed, editorial, local, plain-spoken, unhurried
- **Layout:** Sessions sidebar (288px) + main canvas + optional context drawer (360px)
- **Landing State:** Hero, composer, suggestion chips, model strip, feature trio, how it works, footer
- **Conversation State:** Topbar, messages, composer (docked), context drawer
- **No sign-in/sign-up/account UI**
- **No pricing/billing/upgrade prompts**
- **No external requests for design assets** (self-host fonts and icons)
- **No purple gradients, pill shapes, fake metrics, emoji, em dashes**
- **No scroll or cursor animations**
- **Keyboard shortcuts:** New session, search, sidebar toggle, drawer toggle, send, stop

## 7. Technical Requirements
- **Frontend:** Next.js App Router, React 19, TypeScript, Tailwind CSS, shadcn/ui
- **Backend:** Local server process (Bun/Node.js)
- **Storage:** Local embedded database (SQLite candidate) with WAL
- **Streaming:** Server-Sent Events with explicit flush
- **Providers:** 9Router (default), OpenAI, Anthropic, Google Gemini
- **Security:** Backend-only provider communication, OS credential store for keys
- **Performance:** Hot-path latency budgets (section 27 of SPEC.txt)
- **Modularity:** One module at a time, interface-first, no circular dependencies
- **Documentation:** Version-controlled docs in docs/ directory

## 8. Acceptance Criteria
- 9Router integration works
- Other providers can be added, edited, disabled, removed at runtime
- Model can be changed for default, per session, and mid-session
- Every provider listed as supported has a passing integration test or documented manual verification
- API credentials handled securely (never logged, exported, committed, or shown again)
- Responses stream
- Conversations and sessions persist
- Sessions can be switched
- Session history is visible
- Session data remains isolated
- Files can be uploaded and associated with sessions
- Document processing works
- RAG works
- MCP integration works
- Tools can be executed through the intended architecture
- Skills can be created, imported, enabled, activated per session
- Only skill names and descriptions in request until activated, token costs visible
- Compact works without losing visible history
- Provider/model abstraction exists
- Error handling is robust
- Security considerations implemented
- Tests exist for critical paths
- Code follows module map with no internal module reach-in
- PROGRESS.md current and matches code
- Performance budgets measured and met or documented as missed
- Documentation matches implementation
- Setup reproducible
- Project structure clean

## 9. Requirement IDs
See docs/02-REQUIREMENTS.md for stable requirement IDs (FR-xxx, NFR-xxx, SEC-xxx, UX-xxx, ARCH-xxx).

## 10. Success Metrics
- All hot-path latency budgets met
- Zero perceptible application latency
- Sessions switch instantly
- First token delivered within 30ms of direct provider call overhead
- Workspace persists across restarts
- No credential leakage in logs, exports, or UI
- All providers listed as supported verified
- All critical paths have tests
- Documentation matches implementation
- Setup reproducible