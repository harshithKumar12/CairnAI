# Data Model

## Overview
This document defines the conceptual data model for CairnAI's local workspace. The model is designed for persistence, querying, backup/export, and future extensibility.

## Core Entities

### Workspace
Root container for all application data.

**Fields:**
- `id`: UUID
- `name`: string
- `created_at`: timestamp
- `updated_at`: timestamp
- `location`: path (configurable)
- `version`: schema version

### Session
First-class entity representing an independent conversation context.

**Fields:**
- `id`: UUID
- `title`: string
- `created_at`: timestamp
- `updated_at`: timestamp
- `archived`: boolean
- `default_model_id`: string (nullable)
- `active_skill_ids`: array of UUIDs
- `attachment_ids`: array of UUIDs

### Message
Individual conversation message within a session.

**Fields:**
- `id`: UUID
- `session_id`: foreign key
- `role`: enum (user, assistant, system, summary)
- `content`: string (markdown)
- `created_at`: timestamp
- `model_id`: string (nullable)
- `provider_id`: string (nullable)
- `requested_model`: string (nullable)
- `answered_model`: string (nullable)
- `tokens`: integer (nullable)
- `citations`: array of citation references
- `tool_calls`: array of tool call references

### Attachment
File uploaded and associated with a session.

**Fields:**
- `id`: UUID
- `session_id`: foreign key
- `file_name`: string
- `mime_type`: string
- `size_bytes`: integer
- `storage_path`: string
- `content_hash`: string
- `processing_status`: enum (uploading, processing, ready, failed)
- `created_at`: timestamp
- `use_in_answers`: boolean

### Document
Processed representation of an attachment.

**Fields:**
- `id`: UUID
- `attachment_id`: foreign key
- `content_text`: string (normalized)
- `page_count`: integer (nullable)
- `metadata`: JSON

### DocumentChunk
Chunked unit for retrieval.

**Fields:**
- `id`: UUID
- `document_id`: foreign key
- `chunk_index`: integer
- `content`: string
- `start_offset`: integer
- `end_offset`: integer
- `embedding_id`: UUID (nullable)

### Embedding
Vector representation of a chunk.

**Fields:**
- `id`: UUID
- `chunk_id`: foreign key
- `model_id`: string
- `vector`: array of floats
- `dimensions`: integer

### ProviderProfile
Configuration for an LLM provider.

**Fields:**
- `id`: UUID
- `display_name`: string
- `adapter_type`: enum (openai-compatible, anthropic, google-gemini)
- `base_url`: string
- `credential_ref`: string (never raw key)
- `headers`: JSON (optional)
- `enabled`: boolean
- `is_default`: boolean
- `metadata_overrides`: JSON (optional)

### ModelCatalogEntry
Cached model listing with user overrides.

**Fields:**
- `id`: UUID
- `provider_id`: foreign key
- `model_id`: string
- `display_name`: string (nullable)
- `context_window`: integer (nullable)
- `supports_tools`: boolean (nullable)
- `supports_vision`: boolean (nullable)
- `source`: enum (provider, manual, metadata)

### MCPServer
Configured Model Context Protocol server.

**Fields:**
- `id`: UUID
- `name`: string
- `transport`: enum (stdio, http, sse)
- `command`: string (nullable)
- `url`: string (nullable)
- `status`: enum (connected, disconnected, error)
- `enabled`: boolean

### MCPTool
Tool discovered from an MCP server.

**Fields:**
- `id`: UUID
- `server_id`: foreign key
- `name`: string
- `description`: string
- `input_schema`: JSON
- `permission`: enum (ask, allow, deny)

### ToolExecution
Record of a tool call.

**Fields:**
- `id`: UUID
- `session_id`: foreign key
- `tool_id`: foreign key
- `arguments`: JSON
- `result`: string (nullable)
- `status`: enum (running, succeeded, failed, denied)
- `created_at`: timestamp

### Skill
Skill definition (workspace or global).

**Fields:**
- `id`: UUID
- `name`: string
- `description`: string
- `body_path`: string
- `scope`: enum (workspace, global, starter)
- `hash`: string
- `enabled`: boolean
- `token_count`: integer (estimated)
- `body_token_count`: integer (estimated)

### SessionSkill
Association between session and activated skill.

**Fields:**
- `session_id`: foreign key
- `skill_id`: foreign key
- `activated_at`: timestamp
- `deactivated_at`: timestamp (nullable)

## Relationships

```
Workspace (1) ──< Session (1) ──< Message
                        │
                        └──< Attachment (1) ──< Document (1) ──< DocumentChunk (1) ──< Embedding
                        │
                        └──< ToolExecution (1) ──< MCPTool (1) ──< MCPServer
                        │
                        └──< SessionSkill (1) ──< Skill
```

## Indexes
- `sessions`: by `updated_at` (recency), by `title`
- `messages`: by `session_id` + `created_at`
- `attachments`: by `session_id`
- `documents`: by `attachment_id`
- `chunks`: by `document_id`
- `providers`: by `enabled`
- `skills`: by `enabled`

## Persistence
- **Database:** SQLite with WAL mode
- **Files:** Stored per-session in workspace directory
- **Secrets:** OS credential store or encrypted local file (never in database)
- **Backups:** Export excludes credentials

## Migration Strategy
- Schema versioned in workspace metadata
- Migration scripts run on startup
- Backward-compatible reads preferred
- No automatic destructive migrations