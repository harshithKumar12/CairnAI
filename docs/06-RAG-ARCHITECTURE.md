# RAG Architecture

## Overview
Retrieval-Augmented Generation (RAG) provides contextual answers from attached documents without injecting every document into every request.

## RAG Modes
1. **Off:** No retrieval
2. **Selected Files:** Only explicitly attached files
3. **Auto:** Automatic retrieval based on query relevance

## Pipeline
```
Uploaded Document
    │
    v
Parser (PDF/TXT/MD/DOCX/JSON/CSV/code)
    │
    v
Chunker (configurable size + overlap)
    │
    v
Embedding Model (9Router endpoint or local model)
    │
    v
Vector Store (sqlite-vec or equivalent)
    │
    v
Retriever (similarity search + relevance ranking)
    │
    v
Context Builder (assemble prompt with citations)
    │
    v
LLM Gateway → 9Router/Provider → Response
```

## Document Processing
### Upload
- Drag-and-drop or file picker
- Validates file type and size
- Dashed accent drop overlay
- Stored per-session in workspace

### Processing
- Runs in background workers
- Content hash used to skip unchanged files
- Lazy-load heavy parsers
- Progress tracking in context drawer

### Chunking
- Configurable chunk size and overlap
- Metadata preserved (file name, page/section)
- Chunks indexed with document references

## Embedding Strategy
**Decision:** Evaluated in Phase 0 (ADR-008). Options:
1. 9Router embeddings endpoint (if available and functional)
2. Local embedding model (e.g., BGE, E5, nomic-embed-text)

### Constraints
- Never assume 9Router supports embeddings
- Local model preferred for privacy if 9Router endpoint unavailable
- Embedding model choice documented in ADR

## Vector Storage
**Decision:** sqlite-vec (SQLite extension) or equivalent embedded vector store. Decision pending database choice.

### Requirements
- Local-only
- Efficient similarity search
- Persistence across restarts
- Indexes for hot queries

## Retrieval
### Query Processing
1. User query → context builder determines retrieval need
2. Retriever queries vector store for relevant passages
3. Passages ranked by relevance
4. Context builder assembles prompt with cited passages
5. LLM generates response with citations visible to user

### Performance
- Hard time budget (typically under 100ms)
- Runs in parallel with other request preparation
- If exceeds budget, inform user it was skipped
- Never silently answer without documents when retrieval requested

## Citation Display
- Numbered citation chips inline in responses
- Sand tint, small radius, label text
- Clicking opens popover with:
  - File name
  - Location (page or section)
  - Retrieved passage
  - "Untrusted document content" label with shield icon

## Contextual Modes
### Normal Conversation
No retrieval. Pure conversation history + system instructions.

### Selected-File Interaction
User explicitly selects files for the current question.

### Retrieval-Assisted Conversation
Automatic retrieval based on query relevance and configured mode.

### Future Tool-Assisted Retrieval
MCP-backed retrieval tools (not built into core).

## Security
- Retrieved passages labeled as untrusted
- Prompt injection from documents must not override system instructions
- File paths restricted to workspace
- No automatic cross-session file sharing

## Testing
### Unit
- Chunking algorithm
- Retrieval ranking
- Citation formatting
- Embedding interface

### Integration
- End-to-end retrieval with mock documents
- 9Router embeddings endpoint (if available)
- Local embedding model (if chosen)

### E2E
Flow B: Create session → upload document → process → ask question → retrieve context → generate response

## Performance Budgets
- Retrieval: under 100ms (local)
- No blocking on send path
- Background processing only
- Cached embeddings per content hash