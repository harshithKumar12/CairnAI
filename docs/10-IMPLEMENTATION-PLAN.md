# Implementation Plan

## Overview
This document defines the phased implementation approach for CairnAI. Each phase has a clear goal, inputs, dependencies, tasks, acceptance criteria, and verification approach. The plan follows modular decomposition requirements and supports one-at-a-time development.

## Phase 0: Discovery and Specification

### Goal
Analyze repository, design.md, skills, and specifications. Create all engineering documents. Stop at review gate.

### Inputs
- repository inspection
- design.md analysis
- skill inspection
- BRD
- architecture
- technical decisions
- requirements
- acceptance criteria

### Dependencies
None (initial analysis phase)

### Tasks
1. Inspect repository structure and existing code
2. Read and analyze DESIGN.md design system
3. Locate and inspect Phase Executor skill (if available)
4. Create docs/00-BRD.md (Business Requirements Document)
5. Create docs/01-ARCHITECTURE.md (Technical Architecture)
6. Create docs/03-TECHNICAL-DECISIONS.md (ADR records)
7. Create docs/02-REQUIREMENTS.md (Requirement IDs)
8. Define requirement IDs for all features
9. Define architecture components and module map (section 1A)
10. Define data model
11. Define session/workspace boundaries
12. Define provider abstraction (profiles, adapters, model catalog)
13. Define streaming architecture and latency budgets
14. Define file/RAG architecture
15. Define MCP/tool architecture
16. Define security boundaries
17. Define phase dependency graph
18. Define acceptance criteria
19. STOP: summarize documents, conflicts, decisions, open assumptions, wait for approval

### Files Affected
All docs/ directory files, README, AGENTS.md

### Technical Decisions
- Modular decomposition only
- One module at a time
- Interface-first development
- No circular dependencies
- Small files (<300 lines)

### Acceptance Criteria
- All required engineering documents created
- BRD meets requirement specification
- Architecture documents align with design.md
- Module map covers all required functionality
- Requirement traceability matrix created
- Phase 0 review gate approval

### Tests
- Document verification (reading quality)
- Cross-document consistency checks
- Requirement coverage verification

### Risks
- Missing dependencies (DESIGN.md, skills)
- Scope creep from uncontrolled requirements
- Timeline delays from documentation complexity
- Approval delays from reviewer

### Verification
- Document completeness checked against SPEC.txt section 2
- Cross-references verified between documents
- README updated with project overview
- AGENTS.md synchronized with process requirements
- User approval of Phase 0 artifacts

### Completion Verification
- MEMORY.md created (deferred until Phase 1)
- PROGRESS.md updated with initial status
- All docs in repository synchronized with code requirements
- Module statuses set to "not started"

## Phase 1: Workspace/Foundation

### Goal
Create application foundation: local workspace, persistence, session model, basic UI, navigation/history, performance harness.

### Inputs
- Phase 0 documents approved
- design.md requirements
- module map from docs/01-ARCHITECTURE.md

### Dependencies
- Workspace storage implementation (Phase 1 task 1)
- Session service (Phase 1 task 2)
- Basic UI shell (Phase 1 task 3)

### Tasks
1. **Module: workspace and storage**
   - Create local workspace directory structure
   - Implement workspace persistence across restarts
   - Create session model (UUID, title, timestamps, archived)
   - Define session data relationships
   - Implement session CRUD operations

2. **Module: sessions, conversations and messages**
   - Create message storage with session indexing
   - Implement conversation history retrieval
   - Define message entity with metadata
   - Session switching logic
   - Message persistence on app startup

3. **Module: UI shell and UI components**
   - Implement sessions sidebar component
   - Create main canvas with landing state
   - Implement composer component
   - Add context drawer toggle
   - Basic navigation/history UI

4. **Module: observability and the performance harness**
   - Hot-path timestamp instrumentation
   - Local mock streaming provider
   - Latency measurement against budgets
   - Performance logging setup
   - Budget validation framework

### Files Affected
- src/ (frontend components)
- backend/ (if separate)
- workspace/ (created at runtime)
- docs/01-ARCHITECTURE.md (updated)
- docs/12-PERFORMANCE.md (updated)

### Technical Decisions
- Single-process architecture
- Local-first design
- URL-based session state
- Session isolation
- Performance-first implementation

### Acceptance Criteria
- Workspace persists across restarts
- Sessions can be created, switched, archived, deleted
- Session history visible and searchable
- Session data isolated between sessions
- Application opens quickly (under 2 seconds cold)
- Performance budgets measured against local mock

### Tests
- Workspace persistence tests
- Session CRUD tests
- Session isolation tests
- Performance budget validation
- Navigation and switching tests

### Risks
- Workspace location conflicts
- Session data corruption
- Performance bottlenecks in hot path
- Cross-session context leakage

### Verification
- Run all tests
- Check workspace persistence
- Test session operations
- Verify performance measurements
- Update PROGRESS.md
- Update MEMORY.md

## Phase 2: Provider Chat

### Goal
Implement provider abstraction and provider profiles, OpenAI-compatible adapter, secure key storage, model catalog, streaming with latency budgets, persistent messages, error handling, anthropic and google-gemini adapters.

### Inputs
- Phase 1 completed
- Provider interface from docs/01-ARCHITECTURE.md
- design.md requirements
- VERIFIED 9Router findings (from Phase 0)

### Dependencies
- Provider abstraction module (Phase 2 task 1)
- OpenAI-compatible adapter (Phase 2 task 2)
- Anthropic adapter (Phase 2 task 5)
- Google Gemini adapter (Phase 2 task 5)
- Key storage module (Phase 2 task 3)
- Model catalog (Phase 2 task 4)
- Streaming module (Phase 2 task 2)
- Error handling module (Phase 2 task 6)

### Tasks
1. **Module: provider gateway (adapters, profiles, model catalog)**
   - Define provider profile structure (id, display name, adapter type, base URL, credential reference)
   - Implement provider CRUD (add, edit, test, disable, remove at runtime)
   - Create generic OpenAI-compatible adapter
   - Implement provider connection testing
   - Create model catalog with manual model ID support
   - Add provider fallback and error normalization
   - 9Router adapter (OpenAI-compatible)
   - Anthropic adapter (messages API)
   - Google Gemini adapter (Gemini API)
   - Each adapter verified against official documentation

2. **Module: streaming**
   - Implement streaming client with cancellation support
   - Manage streaming state and retries
   - Forward chunks immediately with explicit flush
   - Implement stop streaming and regenerate
   - Stream error handling and recovery

3. **Module: settings and secrets**
   - Secure key storage (OS credential store + encrypted file)
   - .env.example and .gitignore
   - Automated secret scan
   - Keys never logged or exposed
   - Provider profile with credential reference

4. **Module: sessions, conversations and messages**
   - Extend to store model selection per message
   - Record provider and model metadata for each message
   - Track streaming status in messages

5. **Module: settings and secrets**
   - Settings dialog with provider management
   - Presets: 9Router, OpenAI-compatible, Anthropic, Google Gemini
   - Provider testing with explicit results

### Files Affected
- backend/ (provider implementations)
- src/components/ (provider configuration UI)
- workspace/ (provider profiles)
- docs/01-ARCHITECTURE.md (updated)
- docs/13-PROVIDERS.md (updated)

### Technical Decisions
- Provider abstraction layer
- Runtime profile management
- Secure credential handling
- Model discovery and selection
- Streaming-first implementation
- Error normalization

### Acceptance Criteria
- 9Router integration works
- Other providers can be added/edited/disabled/removed at runtime
- Model can be changed for default, per session, and mid-session
- All listed providers have integration test or documented verification
- API credentials handled securely
- Responses stream (no artificial delays)
- Conversations and sessions persist
- Sessions can be switched
- Session history is visible
- Session data remains isolated
- Error handling is robust
- Security considerations implemented
- Tests exist for critical paths

### Tests
- Provider integration tests
- Streaming tests
- Model switching tests
- Key security tests
- Session isolation tests
- Error handling tests

### Risks
- 9Router API changes
- Provider credential security
- Model metadata accuracy
- Streaming reliability
- Cross-provider compatibility

### Verification
- Run all tests
- Verify 9Router integration
- Test provider operations
- Validate model switching
- Check security controls
- Update PROGRESS.md
- Update MEMORY.md

## Phase 3: File System

### Goal
File upload, validation, storage, metadata, session/file association.

### Inputs
- Phase 2 completed
- File processing requirements from docs/04-DATA-MODEL.md
- Workspace structure from docs/01-ARCHITECTURE.md

### Dependencies
- File processing module (Phase 3)
- File validation module (Phase 3)
- File storage module (Phase 3)
- File metadata module (Phase 3)
- Session/file association module (Phase 3)

### Tasks
1. **Module: file upload**
   - File picker and drag-and-drop support
   - Dashed accent drop overlay
   - File validation (type, size)
   - Upload progress tracking

2. **Module: file processing**
   - Background worker architecture
   - File parsing (PDF, TXT, MD, DOCX, JSON, CSV, source code)
   - Content hash to skip unchanged files
   - Chunking with configurable size
   - Progress tracking

3. **Module: file storage**
   - Per-session file storage
   - File metadata storage
   - Storage path validation
   - File access control

4. **Module: file processing and RAG**
   - File processing coordination
   - Chunk metadata storage
   - Processing status tracking

5. **Module: session/file association**
   - Explicit file-session mapping
   - No automatic cross-session sharing
   - File association management
   - Import/export functionality

### Files Affected
- src/components/ (file upload UI)
- backend/ (file processing)
- workspace/files/ (file storage)
- docs/04-DATA-MODEL.md (updated)
- docs/06-RAG-ARCHITECTURE.md (updated)

### Technical Decisions
- Background processing
- Content hash validation
- Per-session file ownership
- Lazy parsing
- Progress tracking

### Acceptance Criteria
- Files can be uploaded
- Files are validated and stored
- Files associated with sessions
- Processing runs in background
- Processing progress tracked
- File search available
- File isolation between sessions
- File metadata managed

### Tests
- File upload tests
- File validation tests
- Background processing tests
- File storage tests
- Session isolation tests

### Risks
- File parsing errors
- Large file handling
- Processing performance
- Storage limits
- Cross-session access

### Verification
- Run all tests
- Test file upload scenarios
- Test background processing
- Test file associations
- Test isolation
- Update PROGRESS.md
- Update MEMORY.md

## Phase 4: Document Intelligence

### Goal
Parsing, chunking, embeddings, vector storage, retrieval, RAG.

### Inputs
- Phase 3 completed
- Document processing architecture from docs/06-RAG-ARCHITECTURE.md
- Embedding approach from ADR
- Vector storage choice

### Dependencies
- Document parser module (Phase 4)
- Chunker module (Phase 4)
- Embedding module (Phase 4)
- Vector store module (Phase 4)
- Retriever module (Phase 4)
- RAG context builder (Phase 4)

### Tasks
1. **Module: document parser**
   - Modular parser architecture
   - Parser for each supported type
   - Content extraction
   - Error handling

2. **Module: chunker**
   - Configurable chunking strategy
   - Overlap management
   - Chunk metadata storage

3. **Module: embedding**
   - Embedding model interface
   - Embedding cache per content hash
   - Fallback embedding models

4. **Module: vector store**
   - Vector storage implementation
   - Similarity search
   - Vector metadata storage

5. **Module: retriever**
   - Query processing
   - Vector search integration
   - Relevance ranking
   - Passag extraction

6. **Module: context builder**
   - RAG context assembly
   - Citation formatting
   - Integration with LLM request

### Files Affected
- backend/ (RAG implementation)
- docs/06-RAG-ARCHITECTURE.md (updated)
- docs/04-DATA-MODEL.md (updated)

### Technical Decisions
- Modular document processing
- Configurable chunking
- Embedding approach (Phase 0)
- Vector storage choice
- Retrieval performance

### Acceptance Criteria
- Document parsing works
- Chunking produces correct chunks
- Embeddings generated
- Vector storage works
- Retrieval returns relevant passages
- Citations formatted correctly
- RAG context built correctly
- Integration with conversation

### Tests
- Parser tests
- Chunker tests
- Embedding tests
- Vector store tests
- Retrieval tests
- End-to-end RAG tests

### Risks
- Embedding model availability
- Large document handling
- Processing performance
- Vector search accuracy

### Verification
- Run all tests
- Test parsing scenarios
- Test embedding generation
- Test retrieval accuracy
- Test end-to-end RAG
- Update PROGRESS.md
- Update MEMORY.md

## Phase 5: MCP

### Goal
MCP configuration, discovery, tool execution, tool results, permissions.

### Inputs
- Phase 4 completed
- MCP architecture from docs/07-MCP-ARCHITECTURE.md

### Dependencies
- MCP client module (Phase 5)
- Server discovery module (Phase 5)
- Tool execution module (Phase 5)
- Permission management module (Phase 5)
- Tool result processing module (Phase 5)

### Tasks
1. **Module: MCP configuration**
   - Server configuration UI
   - Transport support (stdio, http, sse)
   - Server status monitoring

2. **Module: discovery**
   - Server availability checking
   - Tool discovery per server
   - Tool metadata collection

3. **Module: tool execution**
   - Tool execution with permission prompts
   - Result capture and processing
   - Tool execution history

4. **Module: permissions**
   - Permission state management
   - Per-session/tool permissions
   - Permission UI

5. **Module: tool results**
   - Result formatting
   - Result storage
   - Result display in conversation

### Files Affected
- backend/ (MCP client)
- src/components/ (MCP UI)
- workspace/ (MCP configuration)
- docs/07-MCP-ARCHITECTURE.md (updated)

### Technical Decisions
- Permission-based execution
- Server discovery and monitoring
- Tool execution isolation
- Result security labeling

### Acceptance Criteria
- MCP servers can be configured
- Tools can be discovered
- Tools can be executed with permission
- Results incorporated into conversation
- Server availability monitored
- Permissions respected
- Security boundaries maintained

### Tests
- MCP connection tests
- Tool discovery tests
- Tool execution tests
- Permission tests
- End-to-end MCP tests

### Risks
- Server availability
- Tool permission complexity
- Tool execution security
- Performance impact

### Verification
- Run all tests
- Test MCP configuration
- Test tool execution
- Test permissions
- Test end-to-end MCP
- Update PROGRESS.md
- Update MEMORY.md

## Phase 5B: Skills

### Goal
Skill format, folders and background index, progressive loading (index, on-demand body, supporting files), activation by me (/ picker) and by model (load_skill, when supported), per-session activation and token cost display, Compact action, starter skills, each verified against at least one provider, import review and skill security checks.

### Inputs
- Phase 5 completed
- Skills architecture from section 14A
- Starter skills specification

### Dependencies
- Skills index module (Phase 5B)
- Skill loading module (Phase 5B)
- Progressive loading module (Phase 5B)
- Activation management module (Phase 5B)
- Compact action module (Phase 5B)
- Skill security module (Phase 5B)

### Tasks
1. **Module: skills index**
   - Background skills indexing
   - File hash-based cache
   - Index refresh strategy
   - Skills discovery (workspace, global, starter)

2. **Module: progressive loading**
   - Level 1: names and descriptions (always)
   - Level 2: full SKILL.md body (on activation)
   - Level 3: supporting files (on demand)
   - Token cost calculation

3. **Module: activation**
   - Me activation (type / in composer)
   - Model activation (load_skill tool when supported)
   - Per-session activation
   - Activation UI

4. **Module: compact action**
   - Compact context skill (verify against providers)
   - Context summarization
   - Undo capability
   - Token cost display before/after

5. **Module: starter skills**
   - Verify each starter skill against at least one provider
   - Skip unverified skills
   - Security review of starter skills

6. **Module: security**
   - Import review and skill security checks
   - Skills cannot change permissions
   - Skills cannot approve tool calls
   - Skills follow safety rules

### Files Affected
- src/components/ (skills UI)
- backend/ (skills service)
- workspace/skills/ (skills storage)
- docs/14A-skills.md (if created)
- skills/starter/ (starter skills)

### Technical Decisions
- Progressive loading
- Background indexing
- Security sandbox
- Token-efficient prompts

### Acceptance Criteria
- Skills can be indexed
- Skills load progressively
- Skills can be activated by me and model
- Skills active per session only
- Compact action works
- Token costs visible
- Starter skills verified
- Security checks implemented

### Tests
- Skills indexing tests
- Progressive loading tests
- Activation tests
- Compact tests
- Security tests
- End-to-end skills tests

### Risks
- Skill body parsing
- Model support variability
- Performance of progressive loading
- Security vulnerabilities

### Verification
- Run all tests
- Test skills discovery
- Test activation
- Test security
- Test end-to-end skills
- Update PROGRESS.md
- Update MEMORY.md

## Phase 6: AI Workspace

### Goal
Polished session management, model selector, context panel, file panel, MCP panel, improved UX.

### Inputs
- Phase 5B completed
- Existing component architecture
- design.md improvements

### Dependencies
- Enhanced session management
- Model selector component
- Context drawer improvements
- File panel enhancements
- MCP panel enhancements
- UX improvements

### Tasks
1. **Enhanced session management**
   - Session renaming
   - Session tagging
   - Bulk operations
   - Session organization UI

2. **Model selector**
   - Enhanced model picker
   - Model information display
   - Quick model switching

3. **Context drawer improvements**
   - Better file organization
   - Enhanced retrieval UI
   - Improved tool management
   - Skills panel enhancements

4. **UX improvements**
   - Loading states
   - Error states
   - Empty states
   - Help texts
   - Onboarding tour

### Files Affected
- src/components/ (UI enhancements)
- docs/09-UI-ARCHITECTURE.md (updated)

### Technical Decisions
- UX-first improvements
- Accessibility enhancements
- Performance optimizations
- Component consistency

### Acceptance Criteria
- Session management polished
- Model selector functional
- Context drawer improved
- UX quality meets standards
- All components consistent

### Tests
- UI component tests
- UX testing
- Accessibility tests
- Performance tests

### Risks
- Breaking changes
- UX regression
- Performance impact

### Verification
- Run all tests
- UI testing
- Accessibility validation
- Performance checks
- Update PROGRESS.md
- Update MEMORY.md

## Phase 7: Hardening

### Goal
Testing, security, performance, observability, recovery, documentation, production cleanup.

### Inputs
- Phase 6 completed
- All phases complete
- Performance budgets from section 27
- Security requirements from docs/08-SECURITY.md

### Dependencies
- Testing framework setup
- Security audit
- Performance verification
- Observability implementation
- Recovery procedures
- Documentation updates
- Production cleanup

### Tasks
1. **Testing**
   - Unit tests for all modules
   - Integration tests
   - E2E tests
   - Performance tests
   - Security tests

2. **Security**
   - Security audit
   - Penetration testing
   - Vulnerability scanning
   - Fix security issues

3. **Performance**
   - Verify all budgets in section 27
   - Performance optimization
   - Load testing
   - Monitor performance over time

4. **Observability**
   - Structured logging
   - Metrics collection
   - Error tracking
   - Performance monitoring

5. **Recovery**
   - Crash recovery
   - Data corruption recovery
   - Service recovery
   - Backup and restore

6. **Documentation**
   - Code documentation
   - API documentation
   - Architecture documentation
   - User documentation

7. **Production cleanup**
   - Remove development artifacts
   - Optimize for production
   - Clean up temporary files
   - Finalize configuration

### Files Affected
- All source files
- Configuration files
- Documentation files
- README

### Technical Decisions
- Comprehensive testing
- Security-first approach
- Performance validation
- Observability ready
- Production-ready code

### Acceptance Criteria
- All tests pass
- Security audit passes
- All performance budgets met
- Observability implemented
- Recovery procedures tested
- Documentation complete
- Production cleanup complete

### Tests
- All existing tests
- New security tests
- Performance regression tests
- Integration tests
- End-to-end tests

### Risks
- Test coverage gaps
- Security vulnerabilities
- Performance regressions
- Deployment issues

### Verification
- Run entire test suite
- Security audit
- Performance validation
- Documentation review
- Production deployment simulation
- Update PROGRESS.md
- Update MEMORY.md

## Phased Implementation Status

### Phase 0: Discovery and Specification
Status: **NOT STARTED** (to be started after user approval)

### Phase 1: Workspace/Foundation
Status: **NOT STARTED** (depends on Phase 0)

### Phase 2: Provider Chat
Status: **NOT STARTED** (depends on Phase 1)

### Phase 3: File System
Status: **NOT STARTED** (depends on Phase 2)

### Phase 4: Document Intelligence
Status: **NOT STARTED** (depends on Phase 3)

### Phase 5: MCP
Status: **NOT STARTED** (depends on Phase 4)

### Phase 5B: Skills
Status: **NOT STARTED** (depends on Phase 5)

### Phase 6: AI Workspace
Status: **NOT STARTED** (depends on Phase 5B)

### Phase 7: Hardening
Status: **NOT STARTED** (depends on Phase 6)

## Module Map

See docs/01-ARCHITECTURE.md section 33 for detailed module map:

### Current Status
- **Module:** **Status:** (Not started after Phase 0)
- **Size:** Small, Medium, Large
- **Dependencies:** None for Phase 0

### Modules by Phase
**Phase 0:** Documentation only (no implementation)
**Phase 1:** workspace and storage, sessions/conversations/messages, UI shell, observability
**Phase 2:** provider gateway, streaming, settings/secrets
**Phase 3:** file upload, file processing, file storage
**Phase 4:** document parser, chunker, embedding, vector store, retriever, context builder
**Phase 5:** MCP configuration, discovery, tool execution, permissions, tool results
**Phase 5B:** skills index, progressive loading, activation, compact action, starter skills, security
**Phase 6:** enhanced session management, model selector, context drawer, UX improvements
**Phase 7:** comprehensive testing, security, performance, observability, recovery

## Starting Instructions

1. Review all Phase 0 documentation
2. Verify all requirements are met
3. Create MEMORY.md (at start of implementation)
4. Start Phase 1 implementation
5. Update PROGRESS.md after each session
6. Follow modular decomposition rules
7. Run tests after each module

## Current Progress
- **Phase 0:** Not started (awaiting approval)
- **Modules:** All documentation created, implementation pending
- **Overall:** 0% complete