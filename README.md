# figma-token-comparison

A Claude Code / Agent Skills skill that audits a local React/Tailwind component against its Figma source of truth.

It connects to the [Figma MCP server](https://developers.figma.com/docs/figma-mcp-server/), fetches the design token variables for a given node, and compares them against the named design-token classes used in the component file. Arbitrary values (`h-[48px]`, raw utilities like `flex` or `w-full`, and inline literals) are intentionally ignored — the skill focuses on whether the component is using the **right tokens with the right names and values**.

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
2. Calls the Figma MCP server to fetch variable definitions and design context (screenshot, Code Connect hints, designer annotations).
3. Reads the component file and collects **named design-token classes only** (e.g. `bg-comment-field-background`, `rounded-base-fixed-s`, `p-base-fixed-400`, `font-preset-base-body-03`).
4. Compares them token-by-token against the Figma variables.
5. Produces a compact report grouped into **Matches**, **Discrepancies**, and **Unverified**, with `file.tsx:L123` references.

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
