# Security Document

## Overview
CairnAI is a production-oriented personal AI workspace that must be treated as a real application with real security considerations. Unlike simple demo apps, this is a serious portfolio project where users may store personal data, API keys, and business documents.

## Threat Model
### Attacker Capabilities
1. **Local Access:** Attacker has physical or remote access to the user's machine
2. **Network Access:** Attacker can intercept network traffic (if deployed remotely)
3. **Application Access:** Attacker can open CairnAI on the user's machine (password, keychain, session)
4. **Code Injection:** Attacker can modify the application code
5. **Resource Access:** Attacker can monitor system resources and timing

### Security Objectives
- **Confidentiality:** Protect user credentials, API keys, and conversation data
- **Integrity:** Ensure data is not tampered with or corrupted
- **Availability:** Keep the application functioning and accessible
- **Privacy:** Do not collect unnecessary user data
- **Compliance:** Avoid violating laws and regulations

## Security Boundaries

### Front-End / Back-End
- **Front-end:** Next.js application running in browser. Never calls LLM providers directly.
- **Back-end:** Local server handling all provider communication, storage, tool execution. Holds all secrets.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Browser       │◄──►│  Local Server   │◄──►│  LLM Providers  │
│  (Next.js)      │    │  (Bun/Node)     │    │  (9Router, etc) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Local Storage
- **Configuration:** Local workspace directory (configurable)
- **Secrets:** OS credential store or encrypted local file
- **Data:** Local workspace directory contains all user data

## Implementation Requirements

### 1. API Key Management
**Requirements:**
- Keys stored in OS credential store (preferred) or encrypted local file
- Never in plain text in workspace
- Never in commits
- Never in logs
- Never exposed in UI
- Backend-only access to keys
- Keys sent only to base URL of their profile

**Implementation:**
- Provider profiles reference keys by ID only
- Backend fetches keys from credential store before each API call
- Fallback to encrypted file when OS store unavailable
- Keys cached in memory with short expiration

### 2. Workspace Isolation
**Requirements:**
- Each session has its own directory and data
- Files attached to Session A must not automatically become available to Session B
- Workspace location configurable by user
- Clear ownership/association relationships

**Implementation:**
- Workspace structure: workspace/session/{session-id}/
- Files stored per-session
- Import feature for explicit cross-session sharing
- All paths validated to stay within workspace

### 3. Tool Execution Security
**Requirements:**
- All tool executions require explicit user permission
- Tool results treated as untrusted
- Tools cannot change permissions or approve other tool calls
- Path traversal protection for file-based tools
- Supporting file paths resolved inside skill folders

**Implementation:**
- Three permission states: Ask, Allow (session), Deny
- Tool execution goes through permission prompt
- Results labeled "Untrusted tool output"
- File path validation for all tool inputs

### 4. Prompt Injection Protection
**Requirements:**
- Uploaded documents and MCP outputs never automatically trusted as instructions
- Explicit labeling of untrusted content
- System instructions separate from conversation context
- Retrieval citations clearly attributed

**Implementation:**
- "Untrusted document content" label on retrieved passages
- "Untrusted tool output" label on tool results
- Context builder separates system/instructions from external content
- Retrieval citations prevent direct injection via documents

### 5. Credential Storage Security
**Requirements:**
- Keys never in application memory longer than necessary
- Keys encrypted at rest
- No key recovery mechanism exposed
- Secure environment variable handling

**Implementation:**
- OS credential store with automatic lock
- Encrypted file fallback (age/argon2)
- Keys flushed from memory after use
- No credential export functionality

### 6. File Upload Security
**Requirements:**
- File type validation
- File size limits
- Malware scanning (basic)
- Path traversal prevention
- Content sniffing for Microsoft Office files

**Implementation:**
- Supported types: PDF, TXT, MD, DOCX, JSON, CSV, source code
- File extension validation
- MIME type verification
- Size limits per workspace configuration
- Path normalization before storage

### 7. Configuration Security
**Requirements:**
- Provider base URL validation (http/https only)
- Warn on non-local host with plain http
- No redirect following with auth headers attached
- Keys never in configuration files

**Implementation:**
- URL validation regex
- Warning for non-local http
- Modified fetch with no-redirect option
- Configuration encrypted

### 8. Logging Security
**Requirements:**
- Never log API keys
- Never log sensitive file contents
- Rate-limit endpoints if exposed remotely
- Sanitize error messages
- Audit logging of security events

**Implementation:**
- Structured logging with field filtering
- Redact keys from all error messages
- Log only necessary data
- Security events logged to separate channel

## Compliance

### Data Protection
- No personal data collection
- No analytics unless explicitly enabled
- Local storage only (no cloud services)
- Clear data export functionality
- Account deletion (workspace deletion)

### Legal Compliance
- Privacy policy explains local-only operation
- Terms of service defines local workspace ownership
- No data sharing with third parties
- Terms include liability limitations
- Real contact information in terms

### Open Source
- MIT license (see repository)
- No dependencies with restrictive licenses
- Third-party attributions maintained
- Compliance checks in CI/CD

## Configuration Requirements

### Environment Variables
`.env.example` with placeholder names:
- `CAIRNAI_WORKSPACE_PATH`: Workspace location
- `CAIRNAI_SECRET_STORE`: Credential store type

### Workspace Structure
```
workspace/
├── sessions/           # Session data
│   └── {session-id}/   # Per-session data
├── files/             # Uploaded files
├── config/            # Configuration
│   ├── providers.json
│   └── settings.json
├── skills/            # Skills
└── secrets/           # Encrypted credential storage
```

### File Permissions
- Workspace directory: user read/write/execute
- Files: user read/write
- Secrets: user read only

## Security Tests

### Provider Profile Security
- Adding provider with exposed keys in form
- Keys appear in logs or error messages
- Keys exposed in browser network tab
- Keys stored in plain text files
- Keys logged in API responses

### Session Isolation Security
- Session A files available in Session B
- Cross-session data leakage
- File path traversal from tool execution

### Credential Security
- Keys readable by other users on same machine
- Keys stored in application database
- Keys backed up to cloud storage
- Keys visible in application UI

### Tool Security
- Tool executes without user permission
- Tool modifies provider settings
- Tool changes skill permissions
- Tool path escapes skill folder

### Architecture Security
- Frontend calls provider directly
- API key in URL query parameters
- Keys in application configuration files
- Weak encryption for credential storage

## Error Handling

### Error Format
```json
{
  "code": "provider_unauthorized",
  "message": "The provider refused the API key",
  "action": "Open Settings, Providers",
  "details": {"provider": "OpenAI"}
}
```

### Error States
- **Unreachable:** Provider not responding
- **Unauthorized:** API key rejected
- **Rate Limited:** Upstream provider limit hit
- **Model Not Found:** Requested model unavailable
- **Malformed Stream:** Provider response malformed
- **Timeout:** Request timed out
- **Config Error:** Invalid configuration
- **Permission Error:** Tool execution denied

### Error Handling Rules
- Always user-visible error message with action
- Never generic "something went wrong"
- Never leak keys or file contents in errors
- Specific error states mapped to UI banners
- All errors logged with sanitized data

## Security Testing

### Manual Testing
- Attempt to access provider keys via browser developer tools
- Try to upload executable files
- Attempt session isolation bypass
- Test credential storage extraction
- Verify API key exposure in error messages

### Automated Testing
- Secret scanning pre-commit hook
- CI security checks
- Unit tests for path validation
- Integration tests for session isolation
- End-to-end tests for tool permissions

## Future Security Considerations

### Authentication Extension
- Local account support (password manager)
- Multi-device sync (encrypted)
- Audit logging

### Advanced Protection
- Rate limiting per provider
- Anomaly detection
- Sandbox for tool execution
- Network traffic encryption
- Host-based access control

## Compliance Checklist

### Public Launch Gates (Design.md section 17)
1. **Custom Domain:** Connected, HTTPS, canonical hostname
2. **Favicon:** Custom cairn mark, theme support, touch icon
3. **Builder Tags Removed:** No "Made with AI" or similar tags
4. **Privacy Policy:** Live at /privacy, links from footer and Settings
5. **Terms:** Live at /terms, links from footer and Settings
6. **Page Metadata:** Real title and description
7. **No Placeholder Text:** All content filled
8. **Content Check:** Sections 0.2 and 0.3 checked against every page

### Security Checklist
- [ ] Keys never in commits
- [ ] Keys never in logs
- [ ] Keys never exposed in UI
- [ ] Frontend never calls providers
- [ ] File validation implemented
- [ ] Path traversal protection
- [ ] Tool permissions enforced
- [ ] Upload documents marked untrusted
- [ ] Tool outputs labeled untrusted
- [ ] Retrieved passages labeled untrusted
- [ ] No secrets in configuration files
- [ ] Environment variables encrypted
- [ ] API keys use OS credential store
- [ ] Error messages sanitized
- [ ] Local storage only
- [ ] Data export available
- [ ] Clear data ownership

## Security Training

### For Developers
- Never commit keys to repository
- Validate all user inputs
- Use principle of least privilege
- Assume all external content is untrusted
- Keep secrets out of version control
- Use secure credential storage

### For Users
- Configure strong workspace location
- Use system credential manager
- Regular workspace backups
- Review installed MCP servers
- Check skill imports before enabling
- Understand permission prompts