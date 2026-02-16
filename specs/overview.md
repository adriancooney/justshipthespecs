# Overview

Single-page marketing site for an esoteric concept. The entire page is a markdown document rendered literally — not converted to HTML semantics, but displayed as raw markdown text with soft syntax highlighting applied to the markdown syntax characters themselves.

The feel is a calm, focused text editor (iA Writer, Typora, Obsidian's source mode) — but read-only. Centered column, monospace type, quiet colors. The markdown source _is_ the design.

## Technology

| Layer        | Choice                                      |
| ------------ | ------------------------------------------- |
| Framework    | Next.js 15 (App Router)                     |
| Language     | TypeScript                                  |
| Styling      | Tailwind CSS v4                             |
| Font         | Geist Mono via `next/font/google`           |
| Highlighting | Shiki with custom theme (see below)         |
| Deployment   | Vercel                                      |
| Package mgr  | npm (default in Vercel build env)           |

### Dependencies

Exact packages to install:

| Package              | Purpose                        |
| -------------------- | ------------------------------ |
| `next`               | framework                      |
| `react`              | peer dep                       |
| `react-dom`          | peer dep                       |
| `shiki`              | syntax highlighting            |

Dev dependencies:

| Package              | Purpose                        |
| -------------------- | ------------------------------ |
| `typescript`         | language                       |
| `@types/react`       | types                          |
| `@types/react-dom`   | types                          |
| `@tailwindcss/postcss` | Tailwind v4 PostCSS plugin   |
| `tailwindcss`        | styling                        |
| `postcss`            | required by Tailwind v4        |

Use latest stable versions. No version pinning — `npm install` gets current.

### Scripts

`package.json` scripts:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

No linter. This is an 8-file project — the build passing is sufficient.

### Tailwind v4 Configuration

Tailwind v4 uses CSS-based configuration, not `tailwind.config.ts`. Setup:

1. `postcss.config.mjs` exports `@tailwindcss/postcss` as a plugin.
2. `globals.css` imports Tailwind via `@import "tailwindcss";`
3. No `tailwind.config.ts` file. Custom values use Tailwind's inline arbitrary value syntax (e.g. `max-w-[680px]`, `text-[#3C3C3C]`).

### Next.js Configuration

`next.config.ts` is minimal:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {};

export default nextConfig;
```

No custom webpack config. No experimental flags. No rewrites or redirects.

### TypeScript Configuration

Standard Next.js `tsconfig.json` with path alias:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

Use `npx create-next-app` defaults for the rest. The `@/` alias is used in all imports throughout the codebase.

## Syntax Highlighting: Shiki

Shiki is the highlighter. It ships a TextMate grammar for markdown that already tokenizes the exact syntax characters we need (`#`, `**`, `` ` ``, `[]()`, `-`, `>`, etc.).

### How it works

1. The markdown content string is passed to Shiki's `codeToHtml()` at build time (server component, zero client JS).
2. Shiki uses its built-in `markdown` language grammar to tokenize the source.
3. A custom Shiki theme maps TextMate scopes to the muted color palette defined in `style.md`.
4. Output is a `<pre><code>` block of `<span>` elements with CSS variables for dual-theme colors.
5. A `postprocess` Shiki transformer runs on the final HTML string to wrap link patterns in real `<a>` tags.

### Why Shiki

- Battle-tested TextMate grammar handles markdown edge cases we'd miss with regex.
- Runs server-side — zero JS shipped to the client.
- Custom themes are a simple JSON object mapping scopes to colors.
- Already in the Next.js/Vercel ecosystem.
- No custom tokenizer to build or maintain.

### Custom Theme

The theme file (`lib/markdown/theme.ts`) exports two `ThemeRegistration` objects: `specsLight` and `specsDark`. Scope-to-color mappings are derived directly from the token table in `style.md`.

### Link Transformer

`links.ts` exports a Shiki transformer that makes markdown links clickable.

**The problem:** Shiki fragments `[text](url)` into multiple `<span>` elements (one per token — bracket, text, bracket, paren, url, paren). A naive regex on the HTML won't match `[text](url)` because it's broken across spans.

**The approach:** Use Shiki's HAST-level transformer hooks, not string manipulation.

1. Pre-scan the raw markdown content with a regex (`\[([^\]]+)\]\(([^)]+)\)`) to build a map of `lineNumber → { text, url, fullMatch }` for every link.
2. Use Shiki's `line(node, lineNumber)` transformer hook. For each line that contains a link (per the pre-scan map), restructure the line's HAST children: find the consecutive child spans that make up the link tokens (identified by their TextMate scopes — `punctuation.definition.link`, `string.other.link.title`, `markup.underline.link`) and wrap them in an `<a>` element node.
3. The `<a>` element has `href` set to the URL, `target="_blank"`, and `rel="noopener noreferrer"`.

**Key detail:** The transformer receives HAST nodes (not HTML strings), so wrapping spans in an `<a>` is a tree operation — create a new `{ type: "element", tagName: "a", properties: { href, target, rel }, children: [...linkSpans] }` node and replace the original spans with it.

The `<a>` tag inherits all colors from the child spans. No additional styling. Cursor changes to pointer on hover (handled in `globals.css`). See `app/globals.md`.

## Architecture

```
app/
  layout.tsx       — root layout, font, meta, globals.css import (see app/layout.md)
  page.tsx         — single route, renders highlighted markdown (see app/page.md)
  globals.css      — all global styles (see app/globals.md)
lib/
  markdown/
    content.ts     — raw markdown string (the page copy)
    theme.ts       — two custom Shiki themes (specs-light, specs-dark)
    highlight.ts   — codeToHtml wrapper (see contract below)
    links.ts       — Shiki HAST transformer to wrap links in <a> tags

middleware.ts      — content negotiation (see Content Negotiation section)
postcss.config.mjs — Tailwind v4 PostCSS plugin
next.config.ts     — minimal, empty config
tsconfig.json      — standard Next.js with @/ alias
```

### `highlight.ts` Contract

```ts
export async function highlight(content: string): Promise<string>
```

Internally:

1. Creates a Shiki highlighter (cached — only instantiated once).
2. Calls `codeToHtml(content, { lang: "markdown", themes: { light: specsLight, dark: specsDark }, defaultColor: false, transformers: [linkTransformer(content)] })`.
3. Returns the HTML string.

The `linkTransformer` is a function that takes the raw content string (to pre-scan for link positions) and returns a Shiki transformer object with a `line` hook.

### Content Storage

The markdown copy lives in `lib/markdown/content.ts` as an exported template literal string. No file system reads, no webpack loaders, no raw imports. Just a string constant. Easy to edit, works everywhere Vercel runs.

```ts
export const content = `
# just ship the specs
...
`;
```

## SEO

Full meta tags via Next.js `metadata` export in `layout.tsx`:

| Tag                  | Value                                                        |
| -------------------- | ------------------------------------------------------------ |
| `title`              | just ship the specs                                          |
| `description`        | code is dead. just ship the specs.                           |
| `og:title`           | just ship the specs                                          |
| `og:description`     | code is dead. just ship the specs.                           |
| `og:url`             | (production URL, set after first deploy)                     |
| `twitter:card`       | `summary`                                                    |
| `twitter:title`      | just ship the specs                                          |
| `twitter:description`| code is dead. just ship the specs.                           |

### OG Image

Skip for v1. Remove the `og:image` and `twitter:image` meta tags. Add later when a static asset is provided.

### Favicon

Skip for v1. Use the Next.js default or omit. Not worth agent time.

## Content Negotiation

The root URL (`/`) supports content negotiation via the `Accept` header. This is handled by Next.js middleware (`middleware.ts`).

| Accept header includes | Response                                     |
| ---------------------- | -------------------------------------------- |
| `text/markdown`        | Raw markdown string, `Content-Type: text/markdown; charset=utf-8` |
| anything else          | Normal HTML page via `page.tsx`              |

### Implementation

`middleware.ts` at the project root:

- Imports the content string from `lib/markdown/content.ts`
- Checks the `Accept` header on requests to `/`
- If `text/markdown` is present, returns a `new Response()` with the raw markdown content
- Otherwise, calls `NextResponse.next()` to pass through to the page

```ts
import { NextRequest, NextResponse } from "next/server";
import { content } from "./lib/markdown/content";

export function middleware(request: NextRequest) {
  if (request.headers.get("accept")?.includes("text/markdown")) {
    return new Response(content, {
      headers: { "Content-Type": "text/markdown; charset=utf-8" },
    });
  }
  return NextResponse.next();
}

export const config = {
  matcher: "/",
};
```

The content string is a plain constant with no Node.js dependencies, so it works on the Edge runtime. No `runtime: 'nodejs'` needed.

### Usage

```bash
# get the raw markdown
curl -H "Accept: text/markdown" https://justshipthespecs.com

# get the HTML page (default)
curl https://justshipthespecs.com
```

This makes the site machine-readable. Agents, scrapers, and CLI tools can fetch the raw content directly.

## Repository

[github.com/adriancooney/justshipthespecs](https://github.com/adriancooney/justshipthespecs)

The repo contains only specs. No application code. An agent reads the specs and builds the site. The repo itself is the proof of concept.

## Non-Goals

- No markdown-to-HTML conversion (no `remark`, no `rehype`)
- No rich text rendering — headings don't get bigger, everything is same-size monospace
- No blog, no navigation, no footer links
- No animations or scroll effects
