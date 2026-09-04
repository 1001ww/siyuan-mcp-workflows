---
name: siyuan-mcp-workflows
description: Use SiYuan MCP safely for search and block editing.
version: 0.2.0
author: weish, Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [siyuan, mcp, notes, knowledge-management, blocks]
    related_skills: []
---

# SiYuan MCP Workflows

Use a connected SiYuan MCP server to find, create, and maintain notes without
reimplementing the SiYuan API in a separate CLI. This skill defines operating
discipline; the MCP server provides the concrete tools and their schemas.

## When to Use

- The user asks to search, read, organize, create, or edit SiYuan notes.
- The task concerns SiYuan notebooks, documents, blocks, tags, attributes,
  assets, backlinks, or exports.
- The task requires reliable multi-step work in a local SiYuan knowledge base.

Do not use for ordinary workspace files or for a note app other than SiYuan.

## Prerequisites

- A SiYuan MCP server is connected and its tools are available in the session.
- SiYuan is running and the MCP server can authenticate to it.
- Discover the server's actual tool names and argument schemas before acting.
  Do not assume a particular implementation's names or parameters.

If connection or authentication fails, report the exact failure. Do not fall
back to direct database or filesystem edits unless the user explicitly asks.

## Tool Discovery

1. Locate the connected SiYuan MCP tools with `tool_search` or the tool catalog.
2. Load the schemas needed for this task using `tool_describe`.
3. Prefer the server's structured document/block APIs over raw SQL or HTTP.
4. Use read-only tools first. Completion criterion: the target notebook,
   document, or block has a verified ID and current content.

## Search and Read

1. Choose search based on intent:
   - Exact text, title, tag, or known wording: keyword/full-text search.
   - Conceptual recall: semantic or hybrid search, if the MCP exposes it.
   - Recent activity: recent-document query.
   - Custom reporting: read-only SQL only when structured tools cannot express it.
2. Limit broad searches and inspect the returned path, type, ID, and snippet.
3. Read the candidate document or block before modifying it.
4. When the result is ambiguous, present the candidates and ask the user to
   choose. Completion criterion: one target is selected unambiguously.

## Create Documents

1. Resolve the destination notebook and parent/path. If unknown, list notebooks
   and inspect the document tree.
2. Check for an existing document with the same intended title or path.
3. Create the document with a meaningful title and content. Do not add YAML
   front matter solely to duplicate SiYuan metadata.
4. Add tags, custom attributes, or an icon only when requested or established by
   the user's notebook conventions.
5. Read back the created document and retain its returned ID. Completion
   criterion: title, location, and written content match the request.

## Edit Documents and Blocks

1. Read the current document and identify the narrowest correct edit scope.
   When adding new content rather than changing existing text, follow
   Inserting into Existing Documents for placement and heading level.
2. Use a block-level update, insert, move, or attribute operation for a local
   change. Use document-level replacement only when the user intends to replace
   the whole document.
3. Never delete and recreate a document merely to edit it. That can break block
   IDs, backlinks, attributes, and references.
4. Use IDs returned by SiYuan tools. Never construct an ID from a date, title,
   path, or guessed pattern.
5. Read the changed block or document after the write. Completion criterion: the
   requested change is present and adjacent content remains intact.

## Inserting into Existing Documents

Adding content to a document the user has already written is a placement
decision derived from that document's outline, not an append to whatever
section is convenient:

1. Read the heading outline first: use the outline tool if the MCP exposes
   one, otherwise list every heading with its level from the document content.
   Do not write anything until you can see the full outline and the content of
   the section you plan to touch.
2. Match the new content to the outline by meaning, then classify it:
   - It supplements, details, or expands the content of an existing section:
     it belongs under that section's heading as a child. Level = that
     heading's level + 1.
   - It is a point of the same rank as an existing subheading: it is a
     sibling. Level = that subheading's level.
   - It is an independent new topic: level = the level of the headings it
     will sit beside.
   The user's wording signals the relationship: "supplement" or "expand this
   point" means a child; "another point" or "on the same topic level" means a
   sibling.
3. Keep sibling levels uniform: all headings directly under one parent use the
   same level. If an H2 already has H3 children, content parallel to those
   children is H3, never H4. Never pick a deeper level because the content
   comes later in the document, and never flatten to the parent's own level
   when the content only makes sense as a supplement to that parent.
4. Insert at the boundary of the chosen section, not wherever the tool makes
   it easy. A section ends right before the next heading whose level is less
   than or equal to the section heading's own level; anchor the insert to the
   block before that heading using the tool's explicit position parameters
   (previous/next block ID). Do not blindly prepend or append to the document.
5. Read back the outline after writing. Completion criterion: the new heading
   appears at the intended level and position and no existing heading moved.
   If the level is wrong, update that block in place rather than leaving it.

## Links, Tags, and Attributes

- Use SiYuan block references for internal links: `((block-id "anchor text"))`.
- Preserve existing link style when editing a document. Do not replace block
  references with ordinary Markdown links to internal IDs.
- Prefer dedicated MCP tag and attribute tools over embedding management data in
  prose.
- Use hierarchical tags only when that taxonomy already exists or the user
  specifies it.

## Destructive Operations

1. Before deleting a notebook, document, block, tag, file, or asset, read and
   display the target's title/path/ID and a concise content preview.
2. For documents and blocks, check backlinks or references when that capability
   is available.
3. State the irreversible effect and ask for explicit confirmation in the same
   conversation turn. A prior general request to "clean up" is not confirmation.
4. Execute only the confirmed target, then verify it no longer exists.

Never broaden a deletion from a block to a document, document to a notebook, or
single item to a batch without a new explicit confirmation.

## Bulk Changes

1. Search and collect the complete candidate set using stable IDs.
2. Produce a compact preview: total count, paths, and the intended mutation.
3. Require explicit confirmation before any bulk write, move, tag replacement,
   formatting pass, or deletion.
4. Process the confirmed IDs only, recording failures instead of silently
   skipping them.
5. Read back a representative sample and report completed, failed, and skipped
   counts. Completion criterion: every confirmed ID is accounted for.

## Formatting and Synchronization

- Preserve the document's existing Markdown/Kramdown conventions and block
  structure.
- Treat automatic formatting as a content-changing write; request approval
  before applying it to existing material.
- Do not trigger cloud synchronization or backup unless the user requests it.
  If the MCP exposes sync status, use it only to verify a requested sync.

## Pitfalls

- Document IDs and block IDs are different identifiers. Use the ID type accepted
  by the selected MCP tool.
- Search snippets can be stale or incomplete. Read the target immediately before
  editing.
- Semantic search may be unavailable or have its own index freshness limits.
  Fall back to keyword/full-text search; do not claim semantic coverage.
- Raw SQL may bypass server-level affordances. Keep it read-only unless the user
  explicitly requests otherwise and the MCP server documents write support.
- A heading's level expresses its relationship in the outline — a child of the
  section it supplements, a sibling of the subheadings it parallels — never its
  position in the file. Inserting without reading the outline first produces
  wrongly leveled headings.

## Verification

After each state-changing operation, re-read the exact affected document, block,
or metadata object through the MCP server. Report the verified ID/path and any
items that could not be completed. For a batch, reconcile the reported total
against the confirmed input IDs before finalizing.
