# app/globals.css

All global styles in one file. Imported by `layout.tsx`.

## Full Contents

```css
@import "tailwindcss";

/* Override Tailwind's font-mono to use Geist Mono */
@theme {
  --font-mono: var(--font-geist-mono), ui-monospace, monospace;
}

/* Shiki output overrides */
.shiki {
  background-color: transparent !important;
  font-family: inherit;
  font-size: inherit;
  line-height: 1.75;
}

.shiki code {
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
  white-space: pre-wrap;
  word-break: break-word;
}

pre {
  margin: 0;
  padding: 0;
  background: transparent;
  border: none;
}

/* Shiki dual-theme: light mode uses --shiki-light, dark mode uses --shiki-dark */
.shiki span {
  color: var(--shiki-light);
}

@media (prefers-color-scheme: dark) {
  .shiki span {
    color: var(--shiki-dark);
  }
}

/* Links inside highlighted content */
.shiki a {
  text-decoration: none;
  color: inherit;
  cursor: pointer;
}

/* Selection */
::selection {
  background-color: #E8E4DF;
}

@media (prefers-color-scheme: dark) {
  ::selection {
    background-color: #2E2C28;
  }
}
```

## Notes

- `@import "tailwindcss"` is the Tailwind v4 CSS-based entry point. No `@tailwind base/components/utilities` directives.
- `@theme` block overrides Tailwind's default `--font-mono` to use Geist Mono loaded in the layout.
- Shiki's default `<pre>` styles (background, padding) are stripped. The `<article>` in `page.tsx` handles all layout.
- `white-space: pre-wrap` ensures long lines wrap instead of causing horizontal scroll.
- Link styles are minimal — `cursor: pointer` is the only hover affordance. No underlines, no color changes.
- This is the complete file. No other CSS files exist.
