# MCP Architecture

## Overview
Model Context Protocol (MCP) integration enables the AI workspace to use external tools through configured servers. All tool execution requires explicit user permission.

## MCP Layer
```
AI Workspace
    │
    v
MCP / Tool Layer
    │
    ├── MCP Server A
    ├── MCP Server B
    ├── MCP Server C
    └── [Future MCP Servers]
```

## Core Capabilities
- MCP server configuration
- Server discovery
- Tool discovery
- Tool invocation
- Tool results
- Tool errors
- Server availability
- Permissions
- Configurable servers
- Adding/removing servers

## MCP Server Model
### Server Entity
- `id`: UUID
- `name`: string
- `transport`: enum (stdio, http, sse)
- `command`: string (nullable for stdio)
- `url`: string (nullable for http/sse)
- `status`: enum (connected, disconnected, error)
- `enabled`: boolean

### Tool Entity
- `id`: UUID
- `server_id`: foreign key
- `name`: string
- `description`: string
- `input_schema`: JSON
- `permission`: enum (ask, allow, deny)

## Tool Execution Flow
1. Model requests tool execution
2. Backend receives tool call via adapter
3. Permission prompt displayed to user
   - Tool name
   - Server name
   - Arguments
   - Three buttons: Allow once, Allow for this session, Deny
4. If allowed: tool executed via MCP client
5. Result captured, labeled "Untrusted tool output"
6. Result incorporated into conversation context
7. Errors handled gracefully with user-visible status

## Permission Model
### Ask
Prompt user before each execution.

### Allow
Allow for this session only. Does not persist beyond session.

### Deny
Block execution. Tool remains unavailable for current session.

### Constraints
- Skills cannot change tool permissions
- Tool permissions cannot be approved by skills
- Tool calls always go through permission prompts
- No arbitrary tool execution without user consent

## Tool Result Handling
- Results labeled "Untrusted tool output"
- Formatted JSON in code block
- Collapsed by default
- Errors shown with specific message and action
- Results incorporated into conversation context only when permitted

## Security Boundaries
- No arbitrary code execution
- Path traversal protection
- Tool input validation
- Server availability checks
- Tool output treated as untrusted
- Tool permissions explicit and user-controlled

## UI Integration
### Context Drawer → Tools Tab
- MCP servers as rows: name, transport, status dot with text
- Expanding server lists discovered tools
- Per-tool permission control (Ask, Allow, Deny)
- Add server opens small dialog
- Remove asks for confirmation

### Tool Call Cards
- `lg` radius, surface fill, hairline border
- Header: tool name (label style), server name (muted), status badge
- Body collapsed by default
- Expanded: arguments and result as formatted JSON
- "Untrusted tool output" label

## Testing
### Unit
- Permission state machine
- Tool input validation
- Tool result formatting

### Integration
- MCP server connection/discovery
- Tool execution with mock server
- Permission persistence per session

### E2E
Flow C: Configure MCP → discover tool → request tool → execute tool → receive result → continue response

## Future Extension Points
- Additional MCP server types
- Tool benchmarking
- Tool result caching
- Advanced permission policies