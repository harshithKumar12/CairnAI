# Business Requirements Document (BRD)

## Document Control
- **Version:** 1.0.0
- **Date:** $(date +%Y-%m-%d)
- **Author:** CairnAI Development Team
- **Status:** Draft
- **Project:** CairnAI - Personal AI Workspace

## Executive Summary
CairnAI is a personal AI workspace that acts as a persistent LLM client, connecting to local and remote AI providers through a unified interface. Unlike traditional chatbot interfaces, CairnAI provides a local-first experience with persistent sessions, file interaction, document retrieval, and tool integration capabilities.

The product addresses the friction of repeatedly accessing web-based LLM interfaces by providing a dedicated application that maintains conversation history, session state, and local workspace data. Built on top of 9Router (by default), CairnAI enables multi-model routing, streaming inference, and MCP-based tool integration while keeping all user data stored locally.

## Problem Statement
Users currently face several pain points when interacting with LLMs:
1. **Context Loss:** Conversations are ephemeral on web platforms, requiring repeated context setup
2. **Provider Lock-in:** Users must manage multiple browser tabs/API keys for different providers
3. **Data Privacy Concerns:** Conversation data may be stored on external servers without explicit consent
4. **Fragmented Experience:** File uploads, document querying, and tool usage require switching between different interfaces
5. **No Local Persistence:** Chat history and session data are typically not available across device restarts or browser sessions

## Product Vision
To create a personal AI workspace that feels like a native application for LLM interaction, where users can maintain persistent conversations across sessions, interact with their local files, leverage AI tools through MCP integration, and maintain complete control over their data and provider configurations.

## Product Goals
1. Provide a local-first AI workspace with persistent session storage
2. Enable seamless switching between different LLM providers and models
3. Support file uploads, document processing, and retrieval-augmented generation (RAG)
4. Integrate with Model Context Protocol (MCP) servers for tool usage
5. Ensure all user data remains stored locally by default
6. Deliver a responsive user interface with minimal perceived latency
7. Support multi-model routing through 9Router or direct provider connections
8. Allow runtime configuration of providers, models, and tools without application restarts

## Non-Goals
1. Building a competitive consumer chatbot platform
2. Providing hosted/cloud-based AI model inference
3. Creating proprietary LLMs or training infrastructure
4. Offering multi-user collaboration features in the initial release
5. Implementing billing, subscription, or usage tracking systems
6. Developing mobile applications as a primary target (desktop-first approach)

## Target User
The primary target user for CairnAI is:
- **Technical Professionals:** Developers, data scientists, researchers, and engineers who frequently interact with LLMs for coding, analysis, and research tasks
- **Power Users:** Individuals who require persistent AI conversations for ongoing projects or learning
- **Privacy-Conscious Users:** Those who prefer to keep their conversation data local rather than on external servers
- **Tool Integrators:** Users who want to leverage MCP servers and custom tools within their AI workflow

The secondary target includes:
- Students and educators using AI for learning assistance
- Writers and content creators using AI for ideation and drafting
- Business professionals using AI for analysis and decision support

## User Problems
CairnAI addresses the following specific user problems:
- **FRUSTRATION:** Having to re-explain context in every new chat session
- **INEFFICIENCY:** Switching between multiple AI provider websites or applications
- **ANXIETY:** Concerns about conversation data being stored or analyzed by third parties
- **LIMITATION:** Inability to easily reference or query personal documents during AI conversations
- **FRAGMENTATION:** Needing to use separate tools for file processing, code execution, or web search alongside AI chat
- **DISCONNECTION:** Losing conversation history when browser crashes, updates, or sessions expire

## Core User Journeys
1. **Persistent Conversation:** User starts a conversation about a complex topic, closes the application, returns days later, and continues the conversation with full context intact
2. **Multi-Provider Workflow:** User configures multiple AI providers (9Router, OpenAI, Anthropic), switches between them based on task requirements, and maintains separate conversation histories for each
3. **Document-Centric Inquiry:** User uploads research papers, asks questions about specific sections, receives cited answers, and follows up with additional queries
4. **Tool-Augmented Reasoning:** User connects to an MCP server providing calculator and web search capabilities, asks the AI to perform calculations and retrieve current information, and synthesizes the results
5. **Session Management:** User maintains separate sessions for different projects (e.g., "Web API Research", "Machine Learning Experiments"), switches between them instantly, and preserves all context in each

## Functional Requirements
### Persistent Workspace (FR-001)
- The application shall maintain a local workspace directory for storing all user data
- Workspace data shall persist across application restarts, system reboots, and updates
- Users shall be able to specify the workspace location or accept a default location

### Session Management (FR-002)
- Users shall be able to create, name, and manage multiple independent chat sessions
- Each session shall maintain its own conversation history, file attachments, and configuration
- Switching between sessions shall be instantaneous and preserve all session state
- Users shall be able to search/filter sessions by name, content, or date
- Sessions shall support archiving and deletion with confirmation

### Conversation Persistence (FR-003)
- All user and assistant messages shall be stored locally and associated with their session
- Message history shall be available immediately upon session restoration
- The application shall support exporting and importing conversation histories
- Messages shall retain formatting (markdown, code blocks, etc.) in storage

### LLM Provider Abstraction (FR-004)
- The application shall provide a unified interface for interacting with multiple LLM providers
- Users shall be able to add, remove, enable, and disable provider profiles at runtime
- Each provider profile shall store connection information separately from sensitive credentials
- Provider switching shall not disrupt active conversations or require session restart

### Multi-Model Support (FR-005)
- Users shall be able to select from available models provided by configured providers
- Model selection shall be available at application default, per-session, and per-message levels
- Changing models shall not affect existing conversation history
- The application shall display both requested and actual responding model information when available

### Streaming Responses (FR-006)
- All LLM responses shall be streamed to the user interface in real-time
- Users shall be able to stop ongoing responses and regenerate if needed
- Streaming shall support proper rendering of markdown, code blocks, and other formatted content
- Error states during streaming shall be handled gracefully with user-visible indicators

### File Upload & Attachment (FR-007)
- Users shall be able to attach files to specific sessions for context and reference
- Uploaded files shall be stored locally and associated with the session they were uploaded to
- The application shall validate file types and sizes before processing
- File processing (parsing, chunking, etc.) shall occur in background workers without blocking UI

### Document Retrieval & RAG (FR-008)
- The application shall support retrieving relevant passages from attached documents
- Retrieval shall be configurable (off, selected files only, or automatic)
- Retrieved context shall be clearly attributed to source documents with citation mechanism
- The system shall distinguish between general conversation and retrieval-assisted responses

### MCP Tool Integration (FR-009)
- Users shall be able to configure and manage MCP servers through the application interface
- Available tools from connected MCP servers shall be discoverable and executable
- Tool execution shall require explicit user permission before running
- Tool results shall be incorporated into the conversation context with appropriate attribution

### Skills System (FR-010)
- Users shall be able to create, import, enable, and disable skills for token-efficient prompting
- Skills shall be stored locally and scoped to individual sessions or globally available
- Skill activation/deactivation shall not require application restart
- The system shall display estimated token costs for active skills

## Non-Functional Requirements
### Performance (NFR-001)
- Keypress to visible feedback: under 100ms
- App-added latency between send and request: under 20ms
- First-token overhead vs direct provider call: under 30ms
- Session switching: under 100ms for cached sessions, under 200ms for uncached
- Cold start to interactive UI: under 2 seconds

### Security (NFR-002)
- API keys and credentials shall never be stored in plain text in the workspace
- Credentials shall be stored using OS credential store or encrypted local storage
- API keys shall never be logged, exposed in UI, or included in exports
- Uploaded files and tool outputs shall be marked as untrusted and not executed as code
- Frontend shall never communicate directly with LLM providers (backend-only communication)

### Reliability (NFR-003)
- Application shall recover gracefully from backend/provider failures
- Partial message storage shall prevent data loss during crashes
- Workspace backups shall be possible without including sensitive credentials
- Application shall handle provider rate limits and quota exhaustion with user-visible warnings

### Usability (NFR-004)
- Interface shall follow the design system specified in DESIGN.md
- Keyboard shortcuts shall be provided for common actions
- Error messages shall be specific and actionable (no generic "something went wrong")
- The application shall respect user preferences for theme (light/dark/system)

### Scalability (NFR-005)
- Local storage shall efficiently handle thousands of messages per session
- Workspace shall support multiple concurrent sessions without significant performance degradation
- Background processing shall not block the main UI thread
- Memory usage shall remain bounded during extended sessions

## MVP Scope
For the minimum viable product, CairnAI shall include:
- Local workspace with persistent storage
- Single session management with conversation history
- 9Router integration as the default provider
- Basic model catalog and switching capabilities
- Streaming response rendering
- File upload and local storage
- Basic UI with sessions sidebar and main canvas
- Provider configuration through settings
- Session persistence across restarts

## Future Scope
Post-MVP features to be considered:
- Advanced session organization (tagging, folders, search)
- Document processing pipeline with multiple parser types
- Embedding-based semantic search and retrieval
- MCP server discovery and automatic tool registration
- Skills marketplace and sharing capabilities
- Local model inference integration
- Multi-modal support (image, audio input)
- Advanced analytics and token usage tracking
- Workspace synchronization across devices (opt-in, encrypted)

## Constraints
1. **Technical:** Must be built as a web application using modern browser technologies
2. **Performance:** Must meet specified latency budgets for hot paths
3. **Security:** Must protect user credentials and sensitive data
4. **Privacy:** User data must remain local by default with clear opt-in for any external sharing
5. **Maintainability:** Code must follow modular architecture principles for long-term maintainability
6. **Compatibility:** Must work across major desktop browsers (Chrome, Firefox, Safari, Edge)

## Assumptions
1. Users will have access to at least one LLM provider (local or remote)
2. 9Router will be available as a recommended default provider option
3. Users possess basic technical literacy to configure API keys and providers
4. Local storage space will be sufficient for typical usage patterns
5. Target users primarily operate on desktop/laptop form factors
6. Network connectivity will be available for provider communication (with offline awareness)

## Risks
1. **Provider API Changes:** Breaking changes to provider APIs could require application updates
   - Mitigation: Abstract provider interfaces and maintain version compatibility layers
2. **Storage Limitations:** Local storage constraints for extensive document collections
   - Mitigation: Implement storage cleanup policies and archive capabilities
3. **Security Vulnerabilities:** Potential exposure of credentials or data leakage
   - Mitigation: Follow security best practices, implement credential protection, and conduct audits
4. **Performance Degradation:** Accumulation of data over time impacting responsiveness
   - Mitigation: Implement data pruning, indexing, and background optimization strategies
5. **User Adoption:** Learning curve for new interface compared to familiar chat platforms
   - Mitigation: Focus on familiarity with proven UI patterns and clear onboarding

## Success Criteria
1. **Technical:** All latency budgets in section 27 of SPEC.txt are met or exceeded
2. **Functional:** Users can complete all core user journeys without workflow interruption
3. **Security:** No credential leakage in logs, exports, or UI; secure key storage implemented
4. **Performance:** Application feels responsive with no noticeable lag from the application layer
5. **Reliability:** Workspace data persists correctly across restarts and survives common failure scenarios
6. **Usability:** Users can discover and utilize core features without external documentation
7. **Extensibility:** New providers, models, and tools can be added without modifying core code

## Acceptance Criteria
### Persistent Workspace
- [ ] Workspace directory is created on first run and maintains data across sessions
- [ ] Application restores to previous state after system reboot
- [ ] Workspace location is configurable through settings

### Session Management
- [ ] Users can create multiple named sessions with independent histories
- [ ] Session switching preserves all context and happens instantly
- [ ] Deleted sessions cannot be recovered without explicit backup

### LLM Integration
- [ ] 9Router integration works with default configuration
- [ ] Users can add custom provider profiles with API keys
- [ ] Model switching works at all levels (default, session, per-message)
- [ ] Streaming responses render correctly with proper formatting

### File & Document Handling
- [ ] Users can upload various file types (PDF, TXT, MD, etc.)
- [ ] Uploaded files are associated with the uploading session only
- [ ] Document retrieval provides cited answers when enabled

### Tool & Skill Integration
- [ ] MCP servers can be configured and tools executed with user permission
- [ ] Skills can be created, imported, and activated per session
- [ ] Only skill names/descriptions are sent until activation (token efficiency)

### Security & Privacy
- [ ] API keys are never visible in UI after initial entry
- [ ] No sensitive data appears in logs, error messages, or exports
- [ ] Frontend makes no direct calls to LLM providers

## Requirement Traceability
| Requirement ID | Description | Related Sections | Verification Method |
|----------------|-------------|------------------|---------------------|
| FR-001 | Persistent Workspace | Section 5, 17 | Workspace persistence test |
| FR-002 | Session Management | Section 6, 7 | Session creation/switching test |
| FR-003 | Conversation Persistence | Section 6, 7 | Message history preservation |
| FR-004 | LLM Provider Abstraction | Section 9 | Provider add/remove/test |
| FR-005 | Multi-Model Support | Section 10 | Model switching validation |
| FR-006 | Streaming Responses | Section 11 | Streaming latency and rendering |
| FR-007 | File Upload & Attachment | Section 12 | File association and storage |
| FR-008 | Document Retrieval & RAG | Section 13 | Retrieval accuracy and citation |
| FR-009 | MCP Tool Integration | Section 14 | Tool execution with permissions |
| FR-010 | Skills System | Section 14A | Skill activation and token display |
| NFR-001 | Performance | Section 27 | Latency budget validation |
| NFR-002 | Security | Section 16 accelerates | Credential handling verification |
| NFR-003 | Reliability | Section 20, 30 | Failure recovery testing |
| NFR-004 | Usability | Section 15, 22 | UI/UX evaluation |
| NFR-005 | Scalability | Section 5, 7, 17 | Load testing with large datasets |