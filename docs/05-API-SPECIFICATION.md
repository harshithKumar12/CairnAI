# API Specification

## Overview
This document defines application-level API boundaries between frontend and backend. All endpoints are served locally. The frontend never calls LLM providers directly.

## REST Endpoints

### Sessions
`GET /api/sessions` - List sessions (paginated, by recency)
`POST /api/sessions` - Create new session
`GET /api/sessions/:id` - Get session with messages
`PATCH /api/sessions/:id` - Rename session
`DELETE /api/sessions/:id` - Delete session (with confirmation)
`POST /api/sessions/:id/archive` - Archive session
`POST /api/sessions/:id/restore` - Restore archived session
`GET /api/sessions/search?q=` - Search sessions by title/message content

### Messages
`POST /api/sessions/:id/messages` - Send message (starts streaming)
`GET /api/sessions/:id/messages` - List messages (windowed)
`POST /api/messages/:id/stop` - Stop streaming response
`POST /api/messages/:id/regenerate` - Regenerate response

### Streaming
`GET /api/sessions/:id/stream` - Server-Sent Events stream for response

### Files
`POST /api/sessions/:id/files` - Upload file (multipart)
`GET /api/sessions/:id/files` - List session files
`GET /api/files/:id` - Get file metadata
`DELETE /api/files/:id` - Delete file
`POST /api/files/:id/reprocess` - Retry failed processing
`POST /api/files/:id/remove` - Remove from session

### RAG
`GET /api/sessions/:id/retrieval` - Get retrieval results for last answer
`POST /api/sessions/:id/retrieval` - Configure retrieval mode (off, selected, auto)
`GET /api/documents/:id` - Get document metadata
`GET /api/documents/:id/chunks` - Get chunks for document

### MCP
`GET /api/mcp/servers` - List MCP servers
`POST /api/mcp/servers` - Add MCP server
`PATCH /api/mcp/servers/:id` - Update MCP server
`DELETE /api/mcp/servers/:id` - Remove MCP server
`GET /api/mcp/servers/:id/tools` - List tools for server
`POST /api/mcp/tools/:id/execute` - Execute tool (requires permission)
`POST /api/mcp/tools/:id/permission` - Set tool permission

### Providers
`GET /api/providers` - List provider profiles
`POST /api/providers` - Add provider profile
`PATCH /api/providers/:id` - Update provider profile
`DELETE /api/providers/:id` - Remove provider profile
`POST /api/providers/:id/test` - Test connection
`GET /api/providers/:id/models` - Fetch model catalog
`POST /api/providers/:id/models` - Add manual model ID

### Models
`GET /api/models` - List all models (grouped by provider)
`GET /api/models/default` - Get default model
`PATCH /api/models/default` - Set default model
`PATCH /api/sessions/:id/model` - Set session model

### Skills
`GET /api/skills` - List skills (indexed)
`POST /api/skills` - Create skill
`PATCH /api/skills/:id` - Update skill
`DELETE /api/skills/:id` - Delete skill
`POST /api/skills/import` - Import skill (untrusted, disabled until reviewed)
`POST /api/sessions/:id/skills/:id/activate` - Activate skill for session
`POST /api/sessions/:id/skills/:id/deactivate` - Deactivate skill

### Workspace
`GET /api/workspace` - Get workspace metadata
`POST /api/workspace/export` - Export workspace (excludes keys)
`GET /api/workspace/location` - Get workspace location
`PATCH /api/workspace/location` - Change workspace location
`GET /api/workspace/status` - Get workspace health/status

### Settings
`GET /api/settings` - Get non-secret settings
`PATCH /api/settings` - Update non-secret settings
`GET /api/settings/credentials/:id` - Get credential status only (never key)

## SSE Event Types

### Chat Stream Events
`event: token` - Streamed text chunk
`event: tool_call` - Tool call requested (needs permission)
`event: tool_result` - Tool execution result
`event: error` - Normalized error state
`event: done` - Stream complete
`event: cancelled` - Stream cancelled

### Progress Events
`event: file_progress` - File processing progress
`event: provider_status` - Provider health change
`event: skill_index` - Skill index update

## Error Response Format
```json
{
  "code": "provider_unreachable",
  "message": "The provider is not responding at the configured address",
  "action": "Open Settings",
  "details": {}
}
```

## Normalized Error Codes
- `provider_unreachable`
- `unauthorized`
- `rate_limited`
- `quota_exhausted`
- `model_not_found`
- `malformed_stream`
- `timeout`

## Authentication
- Local-only application
- No sign-in/sign-up
- Session-based or none (depending on threat model)
- Frontend communicates only with local backend

## Rate Limiting
- Local-only, not applicable for single-user app
- Provider-level limits handled at provider layer

## Content Types
- `application/json` for REST
- `text/event-stream` for SSE
- `multipart/form-data` for file uploads

## CORS
- Same-origin only (local app)
- No external origins allowed

## Versioning
- API versioned via path prefix (`/api/v1/...`) or header
- Backward compatibility maintained across versions

## Security Notes
- Never expose API keys in responses
- Validate all inputs (base URLs, file paths, tool names)
- Rate-limit local endpoints if exposed remotely
- No arbitrary file paths accepted from frontend
- All file operations restricted to workspace directory