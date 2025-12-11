# Model Context Protocol (MCP) Architecture Summary

## Scope
- **Specification** – defines protocol requirements.
- **SDKs** – language‑specific implementations.
- **Development tools** – e.g., MCP Inspector.
- **Reference servers** – example server implementations.

## Participants
| Role | Description |
|------|-------------|
| **MCP Host** | AI application (e.g., VS Code) that orchestrates one or more clients. |
| **MCP Client** | Maintains a dedicated 1:1 connection to an MCP server and forwards context to the host. |
| **MCP Server** | Provides context, tools, resources, prompts, etc. Can run locally (std‑io) or remotely (HTTP). |

## Layers
- **Data Layer** – JSON‑RPC 2.0 based protocol that defines lifecycle, primitives (tools, resources, prompts), and notifications.
- **Transport Layer** – Handles connection establishment, framing, authentication. Two mechanisms:
  - *Stdio* for local processes.
  - *Streamable HTTP* for remote servers with SSE support.

## Core Primitives (Server → Client)
| Primitive | Purpose |
|-----------|---------|
| **Tools** | Executable functions the host can invoke. |
| **Resources** | Static data sources (files, DB rows). |
| **Prompts** | Reusable prompt templates for LLM interactions. |

## Core Primitives (Client → Server)
| Primitive | Purpose |
|-----------|---------|
| **Sampling** | Request a language‑model completion from the host. |
| **Elicitation** | Ask the user for additional input. |
| **Logging** | Send debug/monitoring messages to the server. |

## Notifications
Real‑time updates (e.g., `tools/list_changed`) allow servers to push changes without polling.

## Lifecycle Flow
1. **Initialize** – negotiate protocol version & capabilities.
2. **Discover** – client calls `tools/list`, `resources/list`, etc.
3. **Use** – client invokes `tools/call` or accesses resources.
4. **Notify** – server pushes updates via notifications.
5. **Terminate** – graceful shutdown.

## Example Flow (simplified)
```
Client → initialize() → Server
Server ↔ capabilities negotiation
Client → tools/list() → Server returns list
Client → tools/call("open_file", {path:"/tmp/foo.txt"})
Server → execute, return result
```