# SiYuan MCP Workflows

A lean, MCP-first operational skill for SiYuan Note.

It combines the durable workflow ideas from:

- https://github.com/dazexcl/siyuan-skill
- https://github.com/purefkh/siyuan-skills

It intentionally does not include a separate SiYuan CLI, API client, embedding
index, vector database integration, or credentials layer. Those are the MCP
server's responsibility. The skill supplies the agent behavior that makes the
MCP safe and predictable: discovery, search-before-edit, narrow block changes,
write verification, and explicit destructive-operation confirmation.

## Contents

- `SKILL.md`: the portable skill definition.

## Design Decisions

| Retained | Reason |
|---|---|
| Search before edit/delete | Prevents changes to a similarly named note or wrong block. |
| Block-first editing | Preserves document identity, references, and unrelated content. |
| Explicit deletion confirmation | Destructive effects must be scoped and visible. |
| Read-after-write verification | MCP write success alone does not prove the intended result. |
| SiYuan link and metadata conventions | Keeps the knowledge base internally consistent. |
| Bulk-operation preview and reconciliation | Avoids silently changing or skipping notes. |

| Excluded | Reason |
|---|---|
| Node/Python command wrappers | Duplicate the connected MCP server. |
| Vendor-specific tool names | MCP implementations expose different schemas and names. |
| Qdrant, Ollama, FAISS, or OpenAI setup | Semantic search is optional server infrastructure, not workflow policy. |
| Long endpoint and CLI references | Tool schemas are available dynamically through MCP discovery. |
| Auto-sync, auto-format, auto-icon rules | These are preference-sensitive state changes. |

## Installation

Install the directory with your agent's skill mechanism, then start a new agent
session so its skill index refreshes. The SiYuan MCP server must already be
configured and connected.

## Scope

This project is intentionally a Skill-only project. It assumes a capable
SiYuan MCP server, and it does not replace that server.

## License

MIT
