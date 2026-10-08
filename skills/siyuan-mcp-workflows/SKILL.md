---
name: siyuan-mcp-workflows
description: Use SiYuan MCP safely for search and block editing.
version: 0.3.0
author: weish, Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [siyuan, mcp, notes, knowledge-management, blocks]
    related_skills: []
---

# SiYuan MCP Workflows

Operating discipline for a connected SiYuan MCP server: which tools serve
which task, which are off-limits by default, and the exact call shapes that
are known to work. Follow these boundaries instead of improvising calls —
most failures come from calling the wrong tool, or the right tool with a
payload or position parameter it does not accept.

## When to Use

- The user asks to search, read, organize, create, or edit SiYuan notes.
- The task concerns SiYuan notebooks, documents, blocks, tags, attributes,
  databases, assets, backlinks, or exports.

Do not use for ordinary workspace files or a note app other than SiYuan.
Requires a connected SiYuan MCP server; if calls fail with connection or
authentication errors, report the exact failure and stop. Never fall back to
editing SiYuan data files or its database directly.

## Tool Naming

Calls below are written `tool action` — e.g. `block append` means the `block`
tool with `action: "append"`. Clients prefix tool names (ZCode/Claude Code:
`mcp__siyuan__block`); match by the suffix if your session differs. There is
no `tool_search` or `tool_describe` step in this environment — do not call
for one.

## Scope and Boundaries

Read-only actions (get/list/search/render/outline/export-preview and similar)
of any SiYuan tool may be used when a task needs them. Writes are limited to
the tools in the task table below.

**Locked by default — do not call unless the user explicitly asks for that
exact operation in this conversation:**

- `sync` perform/upload/download — cloud synchronization
- `repo` checkout/rollback/purge and `history` rollback/clear — data rollback
- `import` data / `export` data — full workspace backup and restore
- `bazaar` install/update/uninstall/enable — third-party code
- `file` write/delete/rename/copy — debug and log reading only; never use on
  workspace note data
- `unzip`, `http_request` — not note operations

**Destructive — confirmation required first** (see Destructive Operations):
`notebook` remove, `document` delete, `block` delete, `tag` remove,
`database` item_remove/clean, `asset` clean, `bookmark` remove.

**Task table:**

| Task | Use |
|------|-----|
| Find notebooks | `notebook list` |
| Find documents by title | `document search_docs(keyword)` |
| Find blocks by content | `search fulltext(query)` |
| Custom query | `sql query` (SELECT only) |
| Read document structure | `outline get(docID)`, `block get_children(id)` |
| Read content | `block get_kramdown(id)`, `document info(id)` |
| Where does this block live | `block breadcrumb(id)` |
| Create a document | `document create` |
| Write blocks | `block` append / insert / update / move / delete |
| Backlinks and mentions | `ref backlinks(id)` / `ref mentions(id)` |
| Tags | `tag` list / rename / remove |
| Block attributes | `attr` get / set / batch-get |
| Databases (attribute views) | `database` actions |
| Daily note | `dailynote` create / append / prepend |
| Export one document | `export` md / html / docx / sy / md-zip |

If the task needs a write not in this table, say so instead of experimenting
with unlisted tools.

## Identifiers

Three ID kinds exist and never substitute for one another: notebook ID (from
`notebook list`), document ID (from `document search_docs`, `document list`,
or `document info`), block ID (from `outline get`, `block get_children`,
`search fulltext`). A document ID also serves as the container/parent block
for `block append` and `block insert`. Check which kind the target parameter
takes before calling. Never construct an ID from a date, title, or path.

## Read Before Write

1. Resolve the target through the task table and keep its ID.
2. Read current content: `block get_kramdown(id)` for one block;
   `outline get(docID)` + `block get_children(docID)` for a whole document.
   Search snippets can be stale — always re-read the exact target before
   editing it.

## Create Documents

1. `notebook list` → pick the notebook ID (the notebook must be open).
2. Duplicate check: `document list(notebook, path)` for the intended hPath.
3. `document create` requires BOTH `notebook` (ID) and `path` (hPath such as
   `/Folder/Title`). A title-only call fails with "path is required".
4. Do not start the `markdown` payload with `# Title` — the document already
   has a title, and a leading H1 becomes a duplicate heading block inside it.
   Start with an introductory paragraph or `##` sections.
5. Read back with `document info(id)` and `block get_kramdown(id)`.

## Write Blocks

- `block append(data, dataType:"markdown", parentID)` — adds new block(s) as
  the LAST children of parentID. Use the document ID as parentID to add at
  document end. A multi-block payload (blank-line separated) is allowed, but
  the call returns only the FIRST new block's ID — verify with
  `block get_children`.
- `block insert(data, previousID)` — the precise placement call: inserts as a
  sibling immediately AFTER previousID (`nextID` inserts before). This is the
  only call that anchors content into the middle of a document.
- `block insert(data, parentID)` WITHOUT an anchor inserts at the TOP of the
  parent. That is rarely intended — for document end use `append`, for
  mid-document use `previousID`/`nextID`.
- `block update(id, data)` — replaces exactly ONE block. A multi-block
  payload silently drops everything after the first block and still reports
  success. Keep the payload a single block; use insert/append to add content.
  Heading level errors are fixed in place this way (see below).
- `block move(id, parentID, previousID?)`, `block delete(id)`.

Kramdown payload rules: separate blocks with blank lines; write headings as
`## Title`; internal links are block references `((blockID "anchor text"))`,
never `[[blockID]]`; a horizontal super block is `{{{col` … `}}}` with
blank-line-separated children; never paste back the `{: id="…" updated="…"}`
attribute tails that `get_kramdown` output shows.

## Inserting into Existing Documents

Adding content to a document the user already wrote is a placement decision
derived from that document's outline, not an append to whatever section is
convenient:

1. Read the outline first (`outline get(docID)`) plus the content of the
   section you will touch (`block get_children`). Do not write anything until
   you can see the full outline and the target section's blocks.
2. Classify the new content by meaning:
   - It supplements or expands an existing section → child of that section:
     level = section heading's level + 1.
   - It is a point of the same rank as existing subheadings → sibling:
     level = those subheadings' level.
   - It is an independent new topic → level of the headings it sits beside.
   The user's wording signals the relationship: "supplement/expand this
   point" = child; "another point / same level" = sibling.
3. Keep sibling levels uniform: all headings directly under one parent share
   one level. Content parallel to the H3 children of an H2 is H3, never H4.
   Never go deeper merely because the content comes later in the document,
   and never flatten to the parent's own level for content that only makes
   sense as a supplement to that parent.
4. Anchor the insert: a section ends right before the next heading whose
   level is ≤ the section heading's own level. `block get_children` on the
   document shows that boundary block; call `block insert(data,
   previousID=<last block of the section>)`. Do not blindly append to or
   prepend to the document.
5. Verify by re-reading the heading's literal marker (`###`) via
   `block get_kramdown` — outline depth does NOT reveal a wrong level,
   because an H4 following an H2 still nests at depth 2. If the level is
   wrong, fix it in place with `block update` using the corrected marker.

## Tags, Attributes, Databases

- Prefer `tag` and `attr` tools over embedding management data in prose. In
  `attr set`, a null or empty-string value deletes the attribute.
- `database key_update`: change exactly ONE config setting per call, then
  re-render the view to verify. Never write calculated template cells with
  `item_update`.
- Use hierarchical tags only where that taxonomy already exists or the user
  asked for it.

## Destructive Operations

1. Before deleting, read and display the target's title/path/ID plus a
   concise content preview (`block get_kramdown`); for documents and blocks,
   check `ref backlinks(id)` when references matter.
2. State the irreversible effect and ask for explicit confirmation in the
   same conversation turn. A prior general request to "clean up" is not
   confirmation.
3. Execute only the confirmed target, then verify it no longer exists.
4. Never broaden scope — block to document, document to notebook, single item
   to batch — without a new explicit confirmation.

## Bulk Changes

1. Collect the complete candidate set with stable IDs.
2. Preview compactly: total count, paths, intended mutation. Require explicit
   confirmation before any bulk write, move, tag replacement, formatting
   pass, or deletion.
3. Process confirmed IDs only; record failures instead of skipping silently.
4. Read back a representative sample and report completed/failed/skipped
   counts; every confirmed ID must be accounted for.

## Common Errors and Fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `path is required` on create | `document create` called with title only | pass `notebook` (ID) + `path` (hPath) |
| tool not found: `tool_search`/`tool_describe` | no such tools exist here | use the task table; there is no discovery step |
| content after the first block vanished after `update` | `update` replaces ONE block; extra blocks are silently dropped | single-block payload; add content with `insert`/`append` |
| new content landed at the top of the document | `insert` with parentID only inserts FIRST | `append` for document end; `previousID`/`nextID` to anchor mid-document |
| `append` returned one ID for a multi-block payload | only the first new block's ID is returned | expected; verify with `block get_children` |
| outline depth looks right but the heading level is wrong | depth ≠ marker; H4 after H2 still nests at depth 2 | verify via `block get_kramdown`; fix with `block update` |
| SQL rejected or write via SQL failed | `sql` is SELECT-only with a 100-row default | writes go through block/document tools; add explicit LIMIT |
| auth error mentioning encrypted notebook | notebook parameter missing | pass the notebook ID where the tool accepts one |
| semantic search returns nothing | AI embedding may be unconfigured | fall back to `search fulltext`; do not claim semantic coverage |
| duplicate H1 title inside a new document | create payload started with `# Title` | omit the leading H1 in `document create` |

## Verification

After every state-changing call, re-read the affected object through the MCP
server — never trust the call's success message alone (`update` reports
success even when it dropped content). Report the verified ID/path and
anything that could not be completed; for batches, reconcile counts against
the confirmed input.
