# SurfingBro: MCP Web Architecture Plan

## Goals
- Replace stdio with scalable network transports (SSE + Streamable HTTP).
- Keep browser bridge stable and reliable.
- Support multiple MCP clients (local, web, remote) with auth.
- Support multiple sessions and isolated tabs per client/agent.
- Add sidepanel client with multi-LLM support, system prompts, agentic flows.
- Provide admin visibility: connected clients, browser sessions, tabs.
- Create an extensible plugin model for workflows, RAG, embeddings, vision, TTS, local actions.

## Non-Goals (for v1)
- Custom MCP transport over WebSocket.
- Public multi-tenant hosting.
- Fully distributed orchestration across multiple MCP servers.

## Architecture (v1)
### Daemon (server)
- New binary: `cmd/mcpd`.
- HTTP server exposes:
  - `GET /mcp/sse` (MCP SSE transport, SDK handler)
  - `POST /mcp/stream` (MCP streamable transport, SDK handler)
  - `GET /ws` (browser extension WebSocket bridge)
  - `GET /admin/status` (protected status and counts)
  - `GET /admin/clients` (protected list of MCP clients)
  - `GET /admin/browsers` (protected list of browser sessions/tabs)
- Auth:
  - MCP auth token (env: `MCPD_AUTH_TOKEN`) on `/mcp/*`.
  - Admin auth token (env: `MCPD_ADMIN_TOKEN`) on `/admin/*`.
  - Optional auth for `/ws` if needed later.

### Session model
- MCP Client Session
  - `client_id`, `ip`, `user_agent`, `connected_at`, `last_seen`
- Browser Session (extension bridge)
  - `browser_session_id`, `connected_at`, `last_seen`
- Tab Session (content script)
  - `tab_id`, `url`, `title`, `active`, `browser_session_id`
- Routing
  - MCP requests include a `target_session_id` or `tab_id` so multiple agents can operate independently.

### Progress / "What it's doing"
- Server emits MCP notifications with structured progress events:
  - `progress.start`, `progress.update`, `progress.done`
  - `reasoning.summary` (brief, user-friendly)

## Plugin Model
### Plugin Types
1. Server tools (MCP plugins)
   - Best for shared capabilities: workflows, RAG, embeddings, vision, TTS, filesystem ops, integrations.
   - Exposed to all clients via MCP tool registry.
2. Client adapters (sidepanel plugins)
   - Best for LLM providers, UI features, prompt templates, model selection.
3. Orchestration/agent plugins
   - Best for agentic strategies, task decomposition, multi-session planning.

### Recommendation
- Default to server tools for reusable capabilities.
- Keep client adapters lightweight and focused on model UX.
- Orchestration can live in the client initially, and migrate to server if it needs shared state or cross-client access.

## Phased TODO
### Phase 1: Daemon + MCP HTTP
Status: done.
Done items:
- `cmd/mcpd` with SSE + Streamable HTTP handlers.
- Auth middleware.
- Session registry.
- Admin endpoints.

### Phase 2: Session routing
Status: in progress.
Goals:
- Add session-aware routing in MCP calls (client provides target session or tab). Done.
- Extension: support per-command tab targeting. Done.
- Admin endpoints show tabs per browser session.
- Define client/session metadata contract (headers + IDs).
- Provide tab discovery and matching by label/URL.
- Enforce tab ownership and isolation (open/claim/release flows).

### Phase 3: Sidepanel client
- New sidepanel UI.
- Connect over SSE/streamable.
- Multi-LLM adapters (OpenAI, Anthropic, local, etc.).
- System prompt + user chat + tool calls.

### Phase 4: Plugins
- Define plugin API and SDK.
- Add workflows plugin.
- Add RAG / embeddings plugin.

## Open Questions
- Exact session routing identifiers (`tab_id` vs `session_id`).
- Whether `/ws` should be protected for remote usage.
- How to store long-lived session metadata (in-memory vs persisted).
