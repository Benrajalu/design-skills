# AutoDoc for Designers

AutoDoc helps designers do three tasks faster using GitHub Copilot:

1. Fill in component documentation in Figma.
2. Annotate a feature screen for accessibility.
3. Generate documentation text for Supernova.

You do not need to be an engineer to use this repo.

## What You Need

Before you start:

1. A GitHub account with access to this repository.
2. VS Code installed.
3. GitHub Copilot access.
4. A Figma account with a full seat (editing rights).
5. A Supernova account (found in 1Password).

## Step 1: Clone This Repository

Cloning means "download a local copy of this project".

### Option A (Recommended): GitHub Desktop

1. Open GitHub Desktop.
2. Go to **File > Clone Repository**.
3. Open the **URL** tab.
4. Paste this URL:

```text
https://github.com/Benrajalu/pixar-agents.git
```

5. Choose a local folder where you want the project.
6. Click **Clone**.

### Option B: Terminal

1. Open Terminal.
2. Run:

```bash
git clone https://github.com/Benrajalu/pixar-agents.git
cd pixar-agents
```

## Step 2: Open the Project in VS Code

1. Open VS Code.
2. Click **File > Open Folder...**.
3. Select the cloned `pixar-agents` folder.

VS Code may ask if you trust this workspace.

4. Click **Trust**.

This is required so Copilot can use the project configuration.

## Step 3: Connect Copilot to Figma and Supernova

This repository already includes MCP setup for VS Code in `.vscode/mcp.json`.

On your first Copilot run:

1. Open Copilot Chat.
2. Send one of the example prompts from the "How to Use" section below.
3. When prompted, sign in to Figma and Supernova.

After sign-in, Copilot can read design context and generate or update content for you.

## Skills

Start a fresh Copilot chat for each task. Paste the relevant prompt with your Figma URL.

---

### Component Documentation

Fills in a component documentation frame in Figma. Given a frame URL, Copilot reads the component structure, then writes into each documentation section: primitives, main component showcase, component behaviour (text handling, screen sizes, partial content), and accessibility. It places interactive demo instances directly in the frame to illustrate good and bad usage examples.

**Before you start:** Use AleksAI to generate the documentation template for your component. Copy the URL of the frame it creates — it should include sections like Primitives, Main component, etc.

**Prompt:**

```text
Complete the documentation for this component: FRAME_URL_HERE
```

**What to expect:**

- Copilot may ask a few clarifying questions (sizing behaviour, interactive states, platform differences) before writing.
- It fills all sections directly in the Figma file and removes placeholder text.
- The accessibility section is completed last, informed by the rest of the documentation.

---

### Accessibility Annotations

Audits a single feature screen for WCAG AA requirements and places tooltip annotations beside the mockup in Figma. Covers web, native iOS, and native Android in one pass. Annotations are minimal and actionable — only what is ambiguous, missing, or specific to that screen.

**Prompt:**

```text
Annotate this screen for accessibility: FRAME_URL_HERE
```

**What to expect:**

- Copilot walks you through clarifying questions one at a time (imagery, interactive roles, focus order).
- It presents a proposed annotation plan for your approval before drawing anything.
- Tooltips are placed in a new container named `🔍 A11y — [Frame name]` beside your mockup.
- After drawing, you receive a written summary of designer actions not covered by the tooltips.

---

### Supernova Documentation

Generates copy-paste ready documentation for Supernova from a Figma component doc frame. Produces four pages — Overview, Usage, Content, and Accessibility — translated from implementation specs into usage guidance.

**Prompt:**

```text
Generate Supernova documentation for this component: FRAME_URL_HERE
```

**What to expect:**

- Copilot extracts component information from Figma, then asks a small set of targeted questions about real usage context.
- It generates all four pages in a single response, formatted for direct paste into Supernova's editor.
- You can ask it to save the output as a local markdown file in `supernova-doc/generated/`.

---

### Qualitative Research Analysis

Analyzes qualitative research transcripts and moderator observation notes to produce evidence-based findings. It keeps GTM asset feedback separate from interface usability findings, preserves native-language verbatims with translations, anonymizes participant data, and pauses for validation between analysis stages.

**Prompt:**

```text
Analyze this qualitative research study using the qual-research-analyzer skill. I will provide the research context, research questions, hypotheses, assets, participant profiles, transcript, and moderator notes.
```

**What to expect:**

- Copilot asks for only the missing study setup inputs before analysis.
- It presents cleaned participant metadata for validation before extracting evidence.
- It presents quote extractions for validation before clustering patterns.
- It presents draft patterns for validation before generating the final report.
- Findings use qualitative frequency labels, never percentages, and keep direct observation distinct from interpretation.

---

## Rules

- Use one skill per chat conversation.
- Always provide a Figma URL with a `node-id`.

## Troubleshooting

### Copilot cannot access Figma or Supernova

- Make sure you completed the login prompts in Copilot Chat.
- Retry the same prompt once after login.

### Nothing happens after prompt

- Check that your Figma URL is valid and includes `node-id`.
- Make sure the workspace is trusted in VS Code.

### I prefer Copilot CLI instead of VS Code

You can also configure MCP servers in Copilot CLI.

Option A (interactive):

```bash
copilot
/mcp add
```

Add:

- `figma` -> `https://mcp.figma.com/mcp`
- `supernova` -> `https://mcp.supernova.io/mcp/ds/23456`

Option B (manual):

- Merge settings from `mcp-config.copilot-cli.json` into `~/.copilot/mcp-config.json`.

## Repository Structure

- `.agents/skills/component-doc/` → component documentation skill
- `.agents/skills/supernova-doc/` → Supernova documentation skill
- `.agents/skills/a11y-check/` → accessibility annotation skill
- `.agents/skills/qual-research-analyzer/` → qualitative research analysis skill
- `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` → operating instructions per platform
