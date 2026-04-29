# figma-token-comparison

A Claude Code / Agent Skills skill that audits a local React/Tailwind component against its Figma source of truth.

It connects to the [Figma MCP server](https://developers.figma.com/docs/figma-mcp-server/), fetches **Code Syntax class names** (preferred) and/or **design token variables** for a given node, and compares them against the named design-token classes used in the component file. When Code Syntax is configured in the Figma file, the comparison uses exact string matching for maximum accuracy. When only variable definitions are available, it falls back to heuristic token-name reconciliation. Arbitrary values (`h-[48px]`, raw utilities like `flex` or `w-full`, and inline literals) are intentionally ignored — the skill focuses on whether the component is using the **right tokens with the right names and values**.

> **Read-only by design.** This skill must never modify your Figma files. It only calls read-only MCP tools (`get_code_connect_suggestions`, `get_code_connect_map`, `get_variable_defs`, `get_design_context`, etc.). The `SKILL.md` documents the full contract and includes a `settings.json` snippet that denies write-capable Figma tools at the harness level — applying it is strongly recommended, since skill text alone cannot enforce tool access.

## Install

```bash
npx skills add SubliminalGuy/figma-token-comparison
```

## Usage

Invoke the skill with two arguments:

1. `figma_url` — a Figma design URL such as `https://www.figma.com/design/<fileKey>/<fileName>?node-id=<a>-<b>`
2. `component_path` — a path to the local component file to audit

Example:

```
/figma-token-comparison https://www.figma.com/design/kxO1Zkqt2mTTLmqIM52zZq/Article-Components?node-id=13349-14663 packages/components/src/components/CommentField/CommentField.tsx
```

## What it does

For the given Figma node, the skill:

1. Extracts `fileKey` and `nodeId` from the URL.
2. Fetches **Code Syntax class names** via `get_code_connect_suggestions` / `get_code_connect_map` (preferred source — enables exact string matching).
3. Fetches **variable definitions** via `get_variable_defs` (fallback when Code Syntax is unavailable, or supplementary cross-check when both are present).
4. Fetches **design context** — screenshot, Code Connect hints, designer annotations.
5. Reads the component file and collects **named design-token classes only** (e.g. `bg-comment-field-background`, `rounded-base-fixed-s`, `p-base-fixed-400`, `font-preset-base-body-03`).
6. Compares them against the Figma data — exact string matching when Code Syntax is available, heuristic token-name reconciliation otherwise.
7. Produces a compact report grouped into **Matches**, **Discrepancies**, and **Unverified**, with `file.tsx:L123` references and a note on which comparison method was used.

## Prerequisites

You need the Figma MCP server connected in Claude Code. Either variant works:

- **Remote (recommended)** — hosted at `https://mcp.figma.com/mcp`, OAuth-based:

  ```bash
  claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
  ```

  Then authenticate via `/mcp` → **figma** → **Authenticate**.

- **Desktop (local)** — requires the Figma desktop app open with Dev Mode MCP enabled:

  ```bash
  claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp
  ```

See Figma's own [MCP server docs](https://developers.figma.com/docs/figma-mcp-server/) for the full walkthrough.

## License

MIT — see [LICENSE](./LICENSE).
