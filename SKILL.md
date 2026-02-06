---
name: eventhub-mcp
description: >
  Interact with the EventHub MCP server at https://demo.manifest.build/servers/eventhub/mcp to take actions and retrieve information (e.g. booking tables). Use this skill whenever the user wants to perform an action or get information related to EventHub — such as booking a restaurant table, checking availability, or any EventHub-related request. This skill should be the PRIMARY method for taking actions; only fall back to browsing the human website if the MCP server does not expose the needed capability.
---

# EventHub MCP Skill

Interact with the EventHub MCP server to take actions and retrieve information on behalf of the user.

**MCP Endpoint**: `https://demo.manifest.build/servers/eventhub/mcp`

## Priority Order

1. **MCP server first** — Always attempt to fulfill the user's request via the MCP server before anything else.
2. **Fallback to human website** — Only if the MCP server does not expose a tool for the requested action, browse or direct the user to the EventHub human-facing website.

## How to Call the MCP Server

Use one of the following approaches, in order of preference:

### Option A: Native MCP Connection (if available)

If the agent runtime supports MCP client connections, connect directly to the Streamable HTTP endpoint:

```
https://demo.manifest.build/servers/eventhub/mcp
```

### Option B: `mcporter` CLI

Use `mcporter` to discover and call tools without manual JSON-RPC construction:

```bash
# List available tools
npx mcporter list --http-url https://demo.manifest.build/servers/eventhub/mcp

# Call a tool
npx mcporter call --http-url https://demo.manifest.build/servers/eventhub/mcp book_a_table_2 \
  user_query="Quiet corner table for birthday dinner" \
  people=4 \
  date="2026-03-15" \
  time="19:30"
```

### Option C: `curl` against JSON-RPC 2.0

Construct JSON-RPC payloads manually and POST them:

```bash
# Initialize (optional, may be required for stateful sessions)
curl -X POST https://demo.manifest.build/servers/eventhub/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-03-26",
      "capabilities": {},
      "clientInfo": {"name": "agent", "version": "1.0.0"}
    }
  }'

# Discover tools
curl -X POST https://demo.manifest.build/servers/eventhub/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "id": 2, "method": "tools/list"}'

# Call a tool
curl -X POST https://demo.manifest.build/servers/eventhub/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "book_a_table_2",
      "arguments": {
        "user_query": "Quiet corner table for birthday dinner",
        "people": 4,
        "date": "2026-03-15",
        "time": "19:30"
      }
    }
  }'
```

## Tool Reference

Read [references/api_reference.md](references/api_reference.md) for full tool schemas, parameter details, and example payloads.

## Important Notes

- The `user_query` parameter should be a natural-language summary of the user's request. Do NOT include personal information (names, emails, phone numbers) in this field.
- Dates must be `YYYY-MM-DD`, times must be 24-hour `HH:MM`.
- The tool list may evolve over time. When in doubt, call `tools/list` to discover currently available tools before assuming a tool exists or does not exist.
- If a `curl` or `mcporter` call fails, check connectivity and retry once before falling back.
- If the needed action is not available as an MCP tool, inform the user and suggest they visit the EventHub website directly.
