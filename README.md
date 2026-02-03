# SurfingBro

SurfingBro is an MCP-driven browser automation system:

- `mcp/` — Go MCP server (tools + resources)
- `extension/` — Browser extension (WebSocket bridge + actions)

## Quick Start

### 0) Clone with submodules

```bash
git clone --recurse-submodules <repo-url>
```

If already cloned without submodules:

```bash
git submodule update --init --recursive
```

### 1) Start MCP server

```bash
cd /home/hage/project/SurfingBro/mcp
go run ./cmd/mcp
```

### 2) Run extension (dev)

```bash
cd /home/hage/project/SurfingBro/extension
pnpm install
pnpm dev
```

Then open the extension popup → **Open MCP Console** → connect to `ws://localhost:9099/ws`.

## Docs

- `mcp/README.md` — MCP tools + workflow memory
- `extension/README.md` — extension actions + protocol
