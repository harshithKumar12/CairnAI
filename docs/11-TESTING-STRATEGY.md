# Testing Strategy

## Overview
Testing for CairnAI covers unit, integration, and end-to-end scenarios. Testing must cover provider layer, context builder, document processing, chunking, retrieval, tool handling, session management, plus integration tests for 9Router, streaming, persistence, file upload, RAG, MCP.

## Testing Framework
Tests are written for Bun runtime using built-in test runner or Jest-compatible framework.

## Unit Tests

### Provider Layer
Tests for:
- Provider adapter interface
- OpenAI-compatible adapter
- Anthropic adapter
- Google Gemini adapter
- Provider profile validation
- Base URL validation
- Error normalization

Test cases:
- Valid and invalid base URLs
- Model listing and caching
- Streaming chunk parsing
- Error state mapping
- Connection testing

### Context Builder
Tests for:
- Prompt assembly from contexts
- Token counting and estimation
- Incremental caching
- Retrieval context integration
- Skill injection
- Tool definition assembly

Test cases:
- Empty context
- Full conversation history
- Retrieval context inclusion
- Skill body injection
- Token budget enforcement
- Cache validity

### Document Processing
Tests for:
- Parser interface
- Text extraction
- Metadata parsing
- Content hash computation

Test cases:
- Valid and invalid file types
- Corrupted files
- Large files
- Unsupported formats
- Content hash consistency

### Chunking
Tests for:
- Chunk size and overlap
- Chunk metadata
- Chunk boundaries

Test cases:
- Small documents
- Large documents
- Special characters
- Code blocks
- Markdown formatting

### Retrieval
Tests for:
- Similarity search
- Relevance ranking
- Passage extraction

Test cases:
- No matches
- Exact matches
- Partial matches
- Cross-document search
- Performance timing

### Tool Handling
Tests for:
- Tool call formatting
- Result processing
- Permission checks

Test cases:
- Valid tool arguments
- Invalid tool arguments
- Tool execution timeout
- Permission denial
- Result formatting

### Session Management
Tests for:
- Session CRUD operations
- Session switching
- Session isolation
- Session archival

Test cases:
- Create session
- Rename session
- Archive session
- Delete session
- Switch sessions
- Session data isolation

## Integration Tests

### 9Router
Tests for:
- Connection establishment
- Model listing and caching
- Chat streaming interface
- Error handling
- Fallback behavior (if combos used)

Test setup:
- Running 9Router instance (verification required)
- Test with default and custom provider profiles
- Test with mocked and real 9Router

### Streaming
Tests for:
- Complete streaming flow
- Chunk delivery
- Cancellation
- Resume capability
- Error handling during streaming

Test cases:
- Normal streaming
- Streaming interruption
- Cancellation mid-stream
- Error during streaming
- Malformed chunks

### Persistence
Tests for:
- Workspace persistence
- Session persistence
- Message persistence
- Settings persistence

Test cases:
- Application restart
- Backend restart
- System reboot
- Crash recovery
- Data integrity

### File Upload
Tests for:
- Upload workflow
- File validation
- Storage isolation
- Processing progress

Test cases:
- Valid file uploads
- Invalid file types
- Large files
- Cross-session isolation
- Processing failure

### RAG
Tests for:
- Full RAG workflow
- Retrieval accuracy
- Citation display
- Context assembly

Test cases:
- Document upload and processing
- Query and retrieval
- Response with citations
- No match scenario
- Performance timing

### MCP
Tests for:
- Server connection
- Tool discovery
- Tool execution
- Permission handling
- Result processing

Test cases:
- Valid MCP server
- Unavailable server
- Tool execution
- Permission prompt
- Tool error handling

## End-to-End Tests

### Flow A: Session Persistence
```
1. Create session
2. Send message
3. Receive streamed response
4. Restart
5. Reopen session
6. Verify history remains
```

### Flow B: Document RAG
```
1. Create session
2. Upload document
3. Process document
4. Ask question
5. Retrieve context
6. Generate response
```

### Flow C: MCP Tool Flow
```
1. Configure MCP
2. Discover tool
3. Request tool
4. Execute tool
5. Receive result
6. Continue response
```

### Session Isolation Test
```
1. Create Session A
2. Upload files to Session A
3. Create Session B
4. Attempt to access Session A files from Session B
5. Verify files NOT accessible in Session B
```

### Provider and Model Tests
- Adding, editing, disabling and removing a provider profile at runtime
- Changing the model mid-session keeps history intact
- Keys never appear in logs, exports, error messages or frontend responses
- The same model ID on two providers stays distinct
- Every provider listed as supported has an integration test or documented manual verification

### Skills Tests
```
1. Only names and descriptions are in the request until a skill is activated
2. An activated skill's body appears in the request, and only in that session
3. An imported skill is not enabled until reviewed
4. A skill cannot change permissions or approve a tool call
5. Supporting-file paths cannot escape the skill folder
6. Compact keeps history visible and sends summary instead of older messages
7. Without tool-calling support, only manual activation is offered
```

### Performance Tests
- Measure app overhead against local mock streaming provider
- Record numbers against real provider profile
- Validate latency budgets from section 27:
  - Keypress feedback: < 100ms
  - Send to dispatch: < 20ms
  - Per chunk render: < 1 frame
  - First-token overhead: < 30ms
  - Session switch (loaded): < 100ms
  - Cold start: < 2 seconds

## Testing Infrastructure

### Test Environment
- Local mock streaming provider for deterministic tests
- Real provider profiles for integration tests
- CI environment with test credentials

### Test Data
- Sample documents for RAG tests
- Mock MCP servers for tool tests
- Test provider profiles

### Test Automation
- Tests run automatically on CI
- Performance tests run on PR
- Security tests in CI pipeline
- Secret scanning in pre-commit and CI

## Mocking Strategy

### Provider Mocks
- Mock OpenAI-compatible endpoint
- Mock Anthropic endpoint
- Mock Google Gemini endpoint
- Mock streaming provider

### MCP Mocks
- Mock MCP server
- Mock tool implementations

### File System Mocks
- Mock uploads
- Mock file storage

## Test Coverage Goals
- Provider adapters: 90%+
- Context builder: 90%+
- Document processing: 85%+
- Chunking: 95%+
- Retrieval: 85%+
- Tool handling: 90%+
- Session management: 90%+

## Test Running Commands
```
bun test                     # Run unit tests
bun test:integration         # Run integration tests
bun test:e2e                 # Run E2E tests
bun test:performance         # Run performance tests
bun test:all                 # Run all tests
bun run typecheck            # Type check
```

## Continuous Integration
- Tests run on every push
- PR gating required
- Performance regression alerts
- Security scan integration
- Coverage reporting