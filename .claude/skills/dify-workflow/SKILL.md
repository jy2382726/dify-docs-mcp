---
name: dify-workflow-dsl
description: >
  Create, modify, review, and debug Dify Workflow/Chatflow DSL YAML files that can
  be imported into Dify. Use for Dify app DSL, workflow YAML, advanced-chat YAML,
  graph nodes/edges, variables, tool nodes, plugin dependencies, database read/write
  tools, and import/export compatibility questions.
---

# Dify Workflow DSL

Use this skill to produce import-ready Dify DSL YAML. Dify calls the exported
workflow file a DSL; it is a YAML app definition with app metadata, dependencies,
workflow variables/features, and a ReactFlow-like graph of nodes and edges.

## Core Workflow

1. Start with mode intake. For new DSL, default to `workflow`. Use or offer
   `advanced-chat` only when the user needs Chatflow behavior: multi-turn chat,
   memory, `sys.query`, `sys.files`, streaming answers, or `answer` nodes.
2. Clarify only import-blocking requirements: app mode (`workflow` or `advanced-chat`),
   required inputs, model/provider, installed plugins, knowledge bases, secrets,
   trigger source, and expected outputs. If the user has not chosen a mode, say
   that you will proceed with `workflow` by default unless they prefer Chatflow.
3. Choose the DSL version. For new DSL, target official Dify app DSL
   `version: "0.6.0"` unless the user explicitly asks for old-version
   compatibility. Always write `version` as a YAML string.
4. Sketch the graph before writing YAML: start/input or trigger, transform/reasoning nodes,
   tools, branches/loops, final `end` or `answer`.
5. Use stable string node IDs and connect every edge with matching `sourceType`,
   `targetType`, `sourceHandle`, and `targetHandle`.
6. Add `dependencies` for every plugin-backed LLM provider, tool, agent, knowledge
   feature, and model config. Support marketplace, package, and GitHub dependency
   entries; use exact exported plugin identifiers when available.
7. For any plugin/tool not covered by existing examples, use Mintlify MCP
   (`references/mcp-usage-guide.md`) to query Dify docs. Prefer a minimal exported
   DSL from the user's Dify workspace, then plugin source/package metadata, then
   marketplace pages. Be explicit about reliability when exact tool schemas are
   unavailable.
8. Validate locally with `python3 .claude/skills/dify-workflow/scripts/validate_dsl.py <file.yaml>` before giving
   the user the YAML path.

## New DSL Intake

Use this compact intake when creating a workflow from a plain-language request:

- **Mode**: default `workflow`; choose `advanced-chat` for Chatflow, memory, or
  conversational answer nodes.
- **Trigger**: manual start variables, chat input, schedule, webhook, plugin event,
  or another workflow calling this one as a tool.
- **Inputs**: text, files, structured JSON, form fields, dataset IDs, external event
  payload, or tool credentials.
- **Output**: returned `end` values, chat `answer`, side-effect tool action
  (Slack/Feishu/email/DB/API), or generated file.
- **Shape**: straight-line transform, branch classifier, extractor/validator,
  retrieval-augmented answer, loop/iteration over records, or agent with tools.

## Required Decisions

- **Mode**: default to `workflow` for one-shot, batch, triggered, integration,
  and side-effect automations; use `advanced-chat` for Chatflow, `sys.query`,
  `sys.files`, memory, and `answer` nodes.
- **Inputs**: in `workflow`, define start `variables`; in `advanced-chat`, keep
  start variables empty unless the app needs explicit form inputs.
- **Secrets**: do not hardcode real API keys, DB passwords, or webhook secrets.
  Prefer Dify plugin authorization, `env` variables, or clear placeholders.
- **Database access**: prefer parameterized tool calls (`$arg0`, `$arg1`, ...)
  over interpolated SQL. For LLM-generated SQL, restrict to SELECT unless the
  user explicitly asks for writes and accepts the risk.
- **Model/provider**: keep provider names exactly as Dify exports them, for example
  `langgenius/tongyi/tongyi`, `openai`, or a marketplace provider path.
- **New plugin tools**: do not promise import-and-run reliability from a tool name
  alone. Ask for a minimal exported DSL or plugin source/package when exact
  `provider_id`, `tool_name`, parameters, and authorization schema are unknown.

## Authoring Rules

- For newly generated DSL, use `version: "0.6.0"` and top-level `kind: app`.
- `workflow.graph.nodes` and `workflow.graph.edges` must both exist.
- Node wrapper `type` is normally `custom`; `data.type` is the real node kind.
- Every node `id` should be a string. Do not reuse IDs.
- Edges must reference existing node IDs.
- Branch edges from `if-else` use source handles from case IDs, commonly `"true"`,
  `"false"`, or a UUID custom case ID.
- Question classifier source handles use class IDs such as `"1"`, `"2"`.
- Iteration/loop internals need `isInIteration`/`isInLoop`, parent IDs, and start
  helper nodes when exported by Dify.
- `value_selector` and `variable_selector` are arrays, for example
  `["node_id", "text"]`; prompt interpolation is `{{#node_id.field#}}`.
- Code nodes must define the runtime entrypoint: Python uses `def main(...)`,
  JavaScript/TypeScript uses `function main(...)` or an equivalent `main`
  function. Return keys must match `outputs`.
- Tool nodes must include `provider_id`, `provider_name`, `provider_type`,
  `tool_name`, `tool_label`, and `tool_parameters`. `plugin_id`,
  `plugin_unique_identifier`, and `tool_node_version` are common but not universal;
  preserve them when copied from an export.
- `provider_type` may be `builtin`, `api`, `workflow`, or `mcp`.
- Dependencies may use `type: marketplace` with `marketplace_plugin_unique_identifier`,
  `type: package` with `plugin_unique_identifier`, or `type: github` with
  `github_plugin_unique_identifier` plus repo/package metadata.
- `custom-note` nodes are valid canvas annotations and may have empty `data.type`.
- `agent-chat`, `chat`, and `completion` apps may be top-level `model_config`
  apps with no `workflow.graph`; do not force graph rules onto them when reviewing
  legacy exports.
- For public examples, replace tenant-specific icon URLs and credentials with
  placeholders unless they are harmless exported metadata.

## Validation Checklist

Before finalizing a DSL:

- YAML parses cleanly.
- `version` is a string and `app.mode` matches terminal node type:
  non-trigger `workflow` uses `end`, `advanced-chat` uses `answer`, and
  trigger/side-effect workflows document why they may finish at a tool.
- Dependencies cover all plugin-backed nodes.
- All graph edges resolve to existing nodes and matching data types.
- Start variables, conversation variables, and environment variables have unique
  names. Include selectors for new variables; tolerate missing conversation
  selectors when reviewing older exports.
- Every LLM has a model and prompt template.
- Every tool has required provider/tool fields and parameter values.
- New or rare plugin tools are backed by an exported node, plugin package/source,
  or clearly labeled as a best-effort draft that still needs Dify import testing.
- SQL has no trailing comma before `)` and uses bound parameters for dynamic values.
- The final answer tells the user which file was written and whether validation
  passed.

## Reference Map

All references live under `.claude/skills/dify-workflow/references/`. Use these
to resolve node schemas, variable syntax, output fields, validation rules, and
MCP usage patterns.

| File | Contents |
|---|---|
| `references/node-schemas.md` | Node-specific schemas (26 node types) |
| `references/dsl-structure.md` | Top-level YAML structure, variables, dependencies, edges |
| `references/variable-syntax.md` | Variable reference syntax and `rag` prefix |
| `references/node-output-fields.md` | Node output fields for variable references |
| `references/validation-rules.md` | Human-readable validation rules |
| `references/mcp-usage-guide.md` | Mintlify MCP usage guide for querying Dify docs |
| `references/templates/` | 7 workflow templates |

## Mintlify MCP 文档查询

The project integrates with Mintlify MCP for querying Dify documentation. Use
MCP when local references are insufficient.

### When to Call MCP

Call the MCP tools when any of the following apply:

- A node type is **not in the 26 schema list** in `references/node-schemas.md`.
  The user's Dify instance may have a custom or newer node type not yet documented locally.
- The user's Dify version **differs from v0.6.0**. Query for version-specific differences.
- Querying **Context RAG, Jinja2, or Memory** features that may have changed
  between Dify releases.
- Confirming **latest node changes** or deprecations before finalizing DSL.

### Available Tools

Two MCP tools are available for documentation queries:

- **`search_dify_docs(query, version?, language?)`** — Semantic search across the
  Dify documentation knowledge base. Returns relevant content with titles and
  direct links to documentation pages. Use for conceptual queries like "how does
  Context RAG work" or "what are the latest changes to code nodes."
- **`query_docs_filesystem_dify_docs(command)`** — Read-only filesystem query against
  the documentation. Supports `rg`, `grep`, `find`, `cat`, `head`, `tail`, etc.
  Use for exact keyword matching and structural exploration. Pass `.mdx` paths
  (e.g., `/api-reference/create-customer.mdx`) to read full pages.

### MCP Usage Notes

- MCP queries are **read-only** — they never modify local files or state.
- Results are cached for 15 minutes to avoid redundant network calls.
- When MCP is unavailable (network issues, service downtime), fall back to local
  references — see the **降级策略** section below.

## 降级策略

When Mintlify MCP is unavailable (network issues, service downtime, or
unconfigured environment), the skill degrades gracefully to local references:

1. **Node schemas**: Use `references/node-schemas.md` which covers 26 common
   node types. If a node type is not listed, inform the user that the schema
   is a best-effort estimate based on known patterns and may need import
   testing in Dify.
2. **Variable syntax**: Use `references/variable-syntax.md` for standard
   `{{#node_id.field#}}` patterns. For complex or unusual references, note
   that the pattern should be verified against Dify's actual export.
3. **DSL structure**: Use `references/dsl-structure.md` for the top-level
   structure, variables, dependencies, and edges. This covers the stable
   core of the DSL format.
4. **Validation**: Use `references/validation-rules.md` and the validation
   script for structural checks. Note that semantic correctness (e.g., does
   this prompt template actually work with this model) cannot be verified
   without MCP.
5. **Templates**: Use `references/templates/` for common workflow patterns.
   Templates are always available regardless of MCP status.

When degraded, always inform the user: "MCP is currently unavailable; I'm
using local references which may not reflect the latest Dify changes. The
generated DSL should be tested with Dify import before production use."

## Useful Commands

- `python3 .claude/skills/dify-workflow/scripts/validate_dsl.py <file.yaml>` —
  Validate a DSL YAML file against structural and schema rules.
- `rg -i "keyword" .claude/skills/dify-workflow/references/` —
  Search across all reference files for a keyword or pattern.
- `cat .claude/skills/dify-workflow/references/templates/<template>.yaml` —
  View a workflow template for reference.
