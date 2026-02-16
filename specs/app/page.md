# app/page.tsx

The homepage. Server component. Single route.

## Rendering Pipeline

```
content.ts (raw string)
  → highlight.ts (Shiki codeToHtml, lang: "markdown", custom theme)
  → links.ts transformer (wrap [text](url) in <a> tags)
  → page.tsx (dangerouslySetInnerHTML into centered <article>)
```

## Component Structure

`page.tsx` is a server component. No client JS. The full pipeline runs at build time.

```tsx
import { highlight } from "@/lib/markdown/highlight";
import { content } from "@/lib/markdown/content";

export default async function Page() {
  const html = await highlight(content);

  return (
    <article
      className="mx-auto max-w-[680px] px-6 py-[120px] pb-[80px]"
      dangerouslySetInnerHTML={{ __html: html }}
    />
  );
}
```

## Shiki Output Overrides

Shiki wraps output in `<pre><code>`. Global CSS overrides ensure:

- `pre` has no background, no border, no padding (the `<article>` handles layout)
- `code` inherits the page font (Geist Mono) and line-height
- `white-space: pre-wrap` so long lines wrap instead of scrolling horizontally
- `word-break: break-word` as fallback for very long URLs

## Link Behavior

The Shiki HAST-level transformer in `links.ts` wraps link tokens in `<a>` elements at the tree level (not string manipulation). See the Link Transformer section in `overview.md` for the full approach.

- `<a>` inherits all colors from child spans (no additional styling)
- `target="_blank"` and `rel="noopener noreferrer"` for external URLs
- Cursor changes to pointer on hover (only visual affordance, styled in `globals.css`)

## Content Notes

- The ASCII diagram under "specs are the new abstraction" uses 4-space indentation. Shiki's markdown grammar tokenizes this as an indented code block, so it will pick up the teal/code color from the theme. This is intentional — it visually distinguishes the diagram from body text.
- The `↓` and `←` Unicode arrows are plain text content within the code block and render as-is.

## Content

The raw markdown content displayed on the page:

```markdown
# just ship the specs

**code is dead**

you don't need to read code anymore. you don't need to write it.
you need a comprehensive spec and an agent to build it.

lets stop shipping code to Github. just ship the specs.

## specs are the new abstraction

    machine code
      ↓ abstracted by
    code
      ↓ abstracted by
    specs ← you are here

---

## clean room builds

a clean room build means an agent builds the entire project
from specs alone. no existing codebase. no prior context.
no code to reference. just the specs.

every build is fresh. the code is a disposable build artifact.
the specs are the source of truth. markdown is king.

"my specs are building" is the new "my code is compiling"

---

## determinism

> but my build will be non-deterministic

skill issue. tighten your specs. agents can produce near identical builds.

---

## this is a new chapter

the movement is young. we need faster models. `gpt-5.3-codex-spark` is a glimpse
of the future. builds that take many hours today will take minutes tomorrow. ready your brain.

---

## work to do

- make builds cheap
- solve fast iteration
- build out a canonical spec repository + build cache

---

this site is a clean room build. no code was written by a human.
no code was read by a human. the specs are the entire source of truth.

[see the specs](https://github.com/adriancooney/justshipthespecs)

---

just ship the specs.

a psa from [agent cooney](https://x.com/adrian_cooney)
```
