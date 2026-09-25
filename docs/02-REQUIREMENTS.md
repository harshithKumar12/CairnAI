# Requirements Document

This document contains all requirement IDs for CairnAI, organized by category.

## Functional Requirements (FR-xxx)

### Workspace & Persistence
FR-001: The application shall maintain a local workspace directory for storing all user data
FR-002: Workspace data shall persist across application restarts, system reboots, and updates
FR-003: Users shall be able to specify the workspace location or accept a default location

### Session Management
FR-004: Users shall be able to create, name, and manage multiple independent chat sessions
FR-005: Each session shall maintain its own conversation history, file attachments, and configuration
FR-006: Switching between sessions shall be instantaneous and preserve all session state
FR-007: Users shall be able to search/filter sessions by name, content, or date
FR-008: Sessions shall support archiving and deletion with confirmation

### Conversation Management
FR-009: All user and assistant messages shall be stored locally and associated with their session
FR-010: Message history shall be available immediately upon session restoration
FR-011: The application shall support exporting and importing conversation histories
FR-012: Messages shall retain formatting (markdown, code blocks, etc.) in storage

### LLM Provider Integration
FR-013: The application shall provide a unified interface for interacting with multiple LLM providers
FR-014: Users shall be able to add, remove, enable, and disable provider profiles at runtime
FR-015: Each provider profile shall store connection information separately from sensitive credentials
FR-016: Provider switching shall not disrupt active conversations or require session restart

### Multi-Model Support
FR-017: Users shall be able to select from available models provided by configured providers
FR-018: Model selection shall be available at application default, per-session, and per-message levels
FR-019: Changing models shall not affect existing conversation history
FR-020: The application shall display both requested and actual responding model information when available

### Streaming Responses
FR-021: All LLM responses shall be streamed to the user interface in real-time
FR-022: Users shall be able to stop ongoing responses and regenerate if needed
FR-023: Streaming shall support proper rendering of markdown, code blocks, and other formatted content
FR-024: Error states during streaming shall be handled gracefully with user-visible indicators

### File Upload & Attachment
FR-025: Users shall be able to attach files to specific sessions for context and reference
FR-026: Uploaded files shall be stored locally and associated with the session they were uploaded to
FR-027: The application shall validate file types and sizes before processing
FR-028: File processing (parsing, chunking, etc.) shall occur in background workers without blocking UI

### Document Retrieval & RAG
FR-029: The application shall support retrieving relevant passages from attached documents
FR-030: Retrieval shall be configurable (off, selected files only, or automatic)
FR-031: Retrieved context shall be clearly attributed to source documents with citation mechanism
FR-032: The system shall distinguish between general conversation and retrieval-assisted responses

### MCP Tool Integration
FR-033: Users shall be able to configure and manage MCP servers through the application interface
FR-034: Available tools from connected MCP servers shall be discoverable and executable
FR-035: Tool execution shall require explicit user permission before running
FR-036: Tool results shall be incorporated into the conversation context with appropriate attribution

### Skills System
FR-037: Users shall be able to create, import, enable, and disable skills for token-efficient prompting
FR-038: Skills shall be stored locally and scoped to individual sessions or globally available
FR-039: Skill activation/deactivation shall not require application restart
FR-040: The system shall display estimated token costs for active skills

## Non-Functional Requirements (NFR-xxx)

### Performance
NFR-001: Keypress to visible feedback: under 100ms
NFR-002: App-added latency between send and request: under 20ms
NFR-003: First-token overhead vs direct provider call: under 30ms
NFR-004: Session switching: under 100ms for cached sessions, under 200ms for uncached
NFR-005: Cold start to interactive UI: under 2 seconds

### Security
NFR-006: API keys and credentials shall never be stored in plain text in the workspace
NFR-007: Credentials shall be stored using OS credential store or encrypted local storage
NFR-008: API keys shall never be logged, exposed in UI, or included in exports
NFR-009: Uploaded files and tool outputs shall be marked as untrusted and not executed as code
NFR-010: Frontend shall never communicate directly with LLM providers (backend-only communication)

### Reliability
NFR-011: Application shall recover gracefully from backend/provider failures
NFR-012: Partial message storage shall prevent data loss during crashes
NFR-013: Workspace backups shall be possible without including sensitive credentials
NFR-014: Application shall handle provider rate limits and quota exhaustion with user-visible warnings

### Usability
NFR-015: Interface shall follow the design system specified in DESIGN.md
NFR-016: Keyboard shortcuts shall be provided for common actions
NFR-017: Error messages shall be specific and actionable (no generic "something went wrong")
NFR-018: The application shall respect user preferences for theme (light/dark/system)

### Scalability
NFR-019: Local storage shall efficiently handle thousands of messages per session
NFR-020: Workspace shall support multiple concurrent sessions without significant performance degradation
NFR-021: Background processing shall not block the main UI thread
NFR-022: Memory usage shall remain bounded during extended sessions

## Security Requirements (SEC-xxx)

SEC-001: API keys shall be stored using OS credential store or encrypted local storage
SEC-002: API keys shall never be visible in UI after initial entry
SEC-003: No sensitive data shall appear in logs, error messages, or exports
SEC-004: Frontend shall make no direct calls to LLM providers
SEC-005: Uploaded documents shall be marked as untrusted and not executed as code
SEC-006: Tool outputs shall be labeled as untrusted
SEC-007: Retrieved passages shall be labeled as untrusted document content
SEC-008: Imported skills shall be reviewed before enabling
SEC-009: Path traversal protection shall be implemented for all file operations
SEC-010: Base URL validation shall prevent redirect attacks
SEC-011: Pre-commit secret scanning shall prevent credential commits
SEC-012: Provider base URLs shall be validated (http/https only, localhost allowed with warnings)
SEC-013: Errors shall be normalized into distinct provider error states
SEC-014: Error messages shall be specific and actionable with clear user actions
SEC-015: Skills shall not be able to change tool permissions or provider settings
SEC-016: Tool calls shall always go through explicit permission prompts

## User Experience Requirements (UX-xxx)

UX-001: Interface shall follow DESIGN.md visual specifications exactly
UX-002: New session shortcut: Cmd+Shift+O or Ctrl+Shift+O
UX-003: Session search shortcut: Cmd+K or Ctrl+K
UX-004: Sidebar toggle shortcut: Cmd+B or Ctrl+B
UX-005: Context drawer toggle shortcut: Cmd+/ or Ctrl+/
UX-006: Send shortcut: Enter
UX-007: Newline shortcut: Shift+Enter
UX-008: Stop response shortcut: Esc
UX-009: Interface shall show real provider state: real models, real status, real errors
UX-010: Session switching shall be obvious and safe
UX-011: Copy shall follow DESIGN.md section 0.3 rules (concrete, specific, no banned words)
UX-012: The application shall self-host every font and icon
UX-013: The application shall not add sign-in, sign-up, account, pricing or upgrade UI
UX-014: Routes shall be limited to /, /privacy and /terms only
UX-015: No pill or circular buttons, chips or inputs
UX-016: No purple gradients or unauthorized gradients
UX-017: No testimonials, reviews, counters, star counts or "trusted by" rows
UX-018: No emoji, stock photos, generated images or 3D blobs
UX-019: No em dashes in UI copy, legal pages, README text or docs shown to users
UX-020: No scroll or cursor animations
UX-021: No builder or "Made with" badges
UX-022: The application shall not use Venice's or Captions' logos, wordmarks, fonts or exact palettes
UX-023: No hardcoded provider logos, provider counts or model lists
UX-024: No white text on accent for body-size text
UX-025: No marketing claims for unimplemented features

## Architectural Requirements (ARCH-xxx)

ARCH-001: Application shall follow modular decomposition principles (section 1A)
ARCH-002: Each module shall have a defined public interface before implementation
ARCH-003: Modules shall communicate only through public interfaces (no reaching into internals)
ARCH-004: Dependencies shall flow: UI → Application Services → Modules → Storage/Gateways
ARCH-005: Files shall be small and single-purpose (avoid >300 lines where possible)
ARCH-006: Hot-path modules shall be benchmarked before integration (section 27)
ARCH-007: Provider gateway, storage, vector store and MCP client shall sit behind interfaces
ARCH-008: Vertical integration slice shall be implemented early (one message end-to-end)
ARCH-009: Module status shall be tracked in MEMORY.md and PROGRESS.md
ARCH-010: No modules shall be created for deferred features (only extension points)
ARCH-011: Documentation shall be version-controlled with code in docs/ directory
ARCH-012: Minimum documentation set shall be created as specified in section 2
ARCH-013: BRD and Architecture documents shall remain separate (not merged)
ARCH-014: Markdown shall be the canonical engineering document format
ARCH-015: Technical decision records shall be maintained as ADRs in docs/03-TECHNICAL-DECISIONS.md
ARCH-016: API specification shall be documented in docs/05-API-SPECIFICATION.md
ARCH-017: Security considerations shall be documented in docs/08-SECURITY.md
ARCH-018: UI architecture shall be documented in docs/09-UI-ARCHITECTURE.md
ARCH-019: Implementation plan shall be documented in docs/10-IMPLEMENTATION-PLAN.md
ARCH-020: Testing strategy shall be documented in docs/11-TESTING-STRATEGY.md
ARCH-021: Performance considerations shall be documented in docs/12-PERFORMANCE.md
ARCH-022: Provider verification shall be documented in docs/13-PROVIDERS.md
ARCH-023: The application shall follow the phased implementation approach (section 23)
ARCH-024: Each phase shall have acceptance criteria, tests, and completion verification
ARCH-025: MEMORY.md shall be created at implementation start and updated regularly
ARCH-026: PROGRESS.md shall be updated after every work session
ARCH-027: Code shall follow the established dependency direction and module map
ARCH-028: Architecture shall be designed for future extensibility points (section 28)