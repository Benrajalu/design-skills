# AutoDoc — Agent instructions

AutoDoc automates four design team workflows: completing component documentation in Figma, migrating deprecated component instances, generating Supernova documentation copy, and analyzing qualitative research artifacts.

## Architecture

```
AI Agent ──┬──> Figma MCP ──> Figma
           └──> Supernova MCP ──> Supernova (design system docs)
```

Agents read component context from Figma via MCP tools and write changes back directly using `mcp_figma_use_figma`. They can also query the Supernova design system documentation for existing documentation examples.

## Actions

| Action | Purpose |
|--------|---------|
| `ComponentDoc` | Complete component documentation in a Figma frame (sections, accessibility, interactive demos) |
| `ItemReplace` | Scan a Figma frame for deprecated `DEPRECATED Item*` instances and replace them with their modern equivalents |
| `SupernovaDoc` | Generate copy-paste ready Supernova documentation (Overview, Usage, Content, Accessibility) from a Figma component-doc frame |

## Skills

Skills are in `.agents/skills/`. Each skill has its own directory with a `SKILL.md` file and a `references/` folder:

| Skill | Purpose |
|-------|---------|
| `.agents/skills/component-doc/` | Complete component documentation in Figma |
| `.agents/skills/item-replace/` | Migrate deprecated Item* component instances |
| `.agents/skills/supernova-doc/` | Generate Supernova documentation copy |
| `.agents/skills/a11y-check/` | Produce accessibility annotations for a feature screen |
| `.agents/skills/qual-research-analyzer/` | Analyze qualitative research transcripts and moderator notes |

## MCP dependency

Both actions require the **Figma MCP**. The key tools used are:

| Tool | Purpose |
|------|---------|
| `mcp_figma_get_design_context` | Read component structure, variants, and states |
| `mcp_figma_get_screenshot` | Capture visual of component or frame |
| `mcp_figma_get_metadata` | Get node IDs, layer types, and positions |
| `mcp_figma_use_figma` | Write changes to Figma (text, instances, properties) |

### Figma context retrieval default

When reading design context for AutoDoc actions, call `mcp_figma_get_design_context` with `disableCodeConnect: true` by default.

- Do not prompt to connect Code Connect mappings unless the user explicitly asks to set up or manage Code Connect.
- Prioritize direct design context retrieval to keep documentation workflows uninterrupted.

The **Supernova MCP** provides access to existing design system documentation:

| Tool | Purpose |
|------|---------|
| `mcp_supernova_get_documentation_page_list` | List all documentation pages (foundations, components, tokens) |
| `mcp_supernova_get_documentation_page_content` | Get content of specific documentation pages (max 5 at once) |
| `mcp_supernova_get_design_system_component_list` | List all design system components |
| `mcp_supernova_get_design_system_component_detail` | Get component details — Figma link, Storybook URL, statuses (max 10) |
| `mcp_supernova_get_figma_component_list` | List all Figma components from the design system file |
| `mcp_supernova_get_token_list` | Get all design tokens (IDs, paths, values) |
| `mcp_supernova_get_asset_list` | List all assets in the design system |
| `mcp_supernova_get_asset_detail` | Get asset details — name, description, thumbnail, SVG URL (max 10) |

## Setup

### Claude Desktop / Claude Code

MCP servers are configured in `.vscode/mcp.json` (for VS Code integration) or via Claude's MCP settings.

To configure manually, add Figma and Supernova as MCP servers:

- **Figma MCP:** HTTP, URL `https://mcp.figma.com/mcp`
- **Supernova MCP:** HTTP, URL `https://mcp.supernova.io/mcp/ds/23456`

Authenticate with each service when prompted on first tool use.

### VS Code

The workspace includes a `.vscode/mcp.json` config that auto-configures both MCP servers.

1. Open this workspace in VS Code
2. Accept the trust prompt when VS Code asks to start the MCP servers
3. Authenticate with Figma and Supernova when prompted on first tool use

## Running an action

Provide a Figma URL and describe what you need. Match the request to the appropriate skill:

```
Complete the documentation for this component: https://www.figma.com/design/abc123/...?node-id=100:200
```

```
Replace deprecated items in this frame: https://www.figma.com/design/abc123/...?node-id=100:200
```

```
Generate Supernova documentation for this component: https://www.figma.com/design/abc123/...?node-id=100:200
```

For qualitative research analysis, provide the study context, research questions, hypotheses, assets, participant profiles, transcript, and moderator notes:

```
Analyze this qualitative research study using the qual-research-analyzer skill: [study materials]
```

The qualitative research workflow is independent of the Figma MCP and pauses for validation after participant setup, quote extraction, and pattern identification.

## Constraints

- Start a fresh conversation for each action.
- Run one agent per Figma file at a time.
- Always ask the user to create a named version in Figma before making destructive changes (ItemReplace).
