# Phases

## Phase 0: Discovery and Specification
**Goal:** Analyze repository, design.md, skills, and specifications. Create all engineering documents. Stop at review gate.
**Status:** NOT STARTED (awaiting approval)
**Acceptance Criteria:** All required engineering documents created, Phase 0 review gate approval

## Phase 1: Workspace/Foundation
**Goal:** Create application foundation: local workspace, persistence, session model, basic UI, navigation/history, performance harness.
**Status:** NOT STARTED
**Acceptance Criteria:** Workspace persists across restarts, sessions can be created, switched, archived, deleted, session history visible and searchable, performance harness working

## Phase 2: Provider Chat
**Goal:** Implement provider abstraction and provider profiles, OpenAI-compatible adapter, secure key storage, model catalog, streaming with latency budgets, persistent messages, error handling, anthropic and google-gemini adapters.
**Status:** NOT STARTED
**Acceptance Criteria:** 9Router integration works, other providers can be added/edited/disabled/removed at runtime, model can be changed for default/per session/mid-session, credentials handled securely, responses stream

## Phase 3: File System
**Goal:** File upload, validation, storage, metadata, session/file association.
**Status:** NOT STARTED
**Acceptance Criteria:** Files can be uploaded and validated, files associated with sessions, processing runs in background, file isolation between sessions, file metadata managed

## Phase 4: Document Intelligence
**Goal:** Parsing, chunking, embeddings, vector storage, retrieval, RAG.
**Status:** NOT STARTED
**Acceptance Criteria:** Document parsing works, chunking produces correct chunks, embeddings generated, vector storage works, retrieval returns relevant passages, RAG context built correctly

## Phase 5: MCP
**Goal:** MCP configuration, discovery, tool execution, tool results, permissions.
**Status:** NOT STARTED
**Acceptance Criteria:** MCP servers can be configured, tools can be discovered, tools can be executed with permission, results incorporated into conversation, server availability monitored, permissions respected

## Phase 5B: Skills
**Goal:** Skill format, folders and background index, progressive loading, activation by me (/ picker) and by model (load_skill, when supported), per-session activation and token cost display, Compact action, starter skills, each verified against at least one provider, import review and skill security checks.
**Status:** NOT STARTED
**Acceptance Criteria:** Skills can be indexed, skills load progressively, skills can be activated by me and model, skills active per session only, Compact action works, token costs visible, starter skills verified, security checks implemented

## Phase 6: AI Workspace
**Goal:** Polished session management, model selector, context panel, file panel, MCP panel, improved UX.
**Status:** NOT STARTED
**Acceptance Criteria:** Session management polished, model selector functional, context drawer improved, UX quality meets standards, all components consistent

## Phase 7: Hardening
**Goal:** Testing, security, performance, observability, recovery, documentation, production cleanup.
**Status:** NOT STARTED
**Acceptance Criteria:** All tests pass, security audit passes, all performance budgets met, observability implemented, recovery procedures tested, documentation complete, production cleanup complete

## Current Status
**Overall Progress:** 0%
**Current Phase:** Phase 0 (Discovery and Specification) - Awaiting user approval