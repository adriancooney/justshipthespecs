# Style

## Philosophy

The page looks like you opened a markdown file in a beautiful, minimal text editor. Everything is monospace. Nothing is "rendered" — the `#`, `**`, `[]()` characters are all visible. But they're softly colored so your eye flows through the content naturally.

Think: source code aesthetic applied to prose. The content speaks. The styling whispers.

## Layout

- Centered single column, max-width ~680px
- Generous vertical padding (120px+ top, 80px+ bottom)
- Line height 1.7–1.8 for comfortable reading
- No visible chrome — no header, no sidebar, no toolbar

## Typography

- Monospace everywhere — Geist Mono (loaded via `next/font/google`) or system monospace fallback
- Base size ~15px
- No size variation — headings, body, everything is the same size
- Weight: regular (400) for body, medium (500) for emphasized syntax if needed
- Tab size / indent: 2 spaces

## Color Scheme

Two modes: light (default) and dark. Respects `prefers-color-scheme`. No toggle — the OS decides.

### Light Mode

| Role             | Hex       | Description                |
| ---------------- | --------- | -------------------------- |
| Background       | `#FAFAF8` | Warm off-white             |
| Text (body)      | `#3C3C3C` | Soft dark gray, never black |
| Selection        | `#E8E4DF` | Warm beige highlight       |

### Dark Mode

| Role             | Hex       | Description                  |
| ---------------- | --------- | ---------------------------- |
| Background       | `#1C1C1C` | Deep warm charcoal           |
| Text (body)      | `#C8C4BE` | Soft warm light gray         |
| Selection        | `#2E2C28` | Subtle warm dark highlight   |

## Syntax Highlighting

Markdown syntax characters get soft, muted color treatment. Content text stays the primary body color. The highlighting is subtle — more like a whisper than a shout.

One palette works across both modes. In dark mode, each color is lifted slightly in lightness to compensate for the dark background, but the hue and saturation stay the same.

### Token Colors

| Token              | Example chars     | Light         | Dark          | Shiki TextMate Scope                    |
| ------------------ | ----------------- | ------------- | ------------- | --------------------------------------- |
| Heading markers    | `#`, `##`, `###`  | `#B07D6E`     | `#C48A7B`     | `markup.heading`, `punctuation.definition.heading` |
| Bold markers       | `**`              | `#A48952`     | `#B89860`     | `punctuation.definition.bold`           |
| Bold text          | text inside `**`  | `#3C3C3C`     | `#C8C4BE`     | `markup.bold` (same as body text)       |
| Italic markers     | `_` or `*`        | `#A48952`     | `#B89860`     | `punctuation.definition.italic`         |
| Link brackets      | `[]`              | `#6E7F94`     | `#7B8FA8`     | `punctuation.definition.link`           |
| Link URL           | `(url)`           | `#6E7F94`     | `#7B8FA8`     | `markup.underline.link`                 |
| Link text          | text inside `[]`  | `#6E7F94`     | `#7B8FA8`     | `string.other.link.title`               |
| Code backticks     | `` ` ``           | `#5E8E78`     | `#6B9E8A`     | `punctuation.definition.raw`            |
| Code content       | inline code       | `#5E8E78`     | `#6B9E8A`     | `markup.inline.raw`                     |
| List markers       | `-`, `*`, `1.`    | `#B07D6E`     | `#C48A7B`     | `punctuation.definition.list`           |
| Blockquote markers | `>`               | `#8A7DB0`     | `#9B8EC4`     | `markup.quote`, `punctuation.definition.quote` |
| Horizontal rules   | `---`             | `#9C9890`     | `#6B6860`     | `meta.separator`                        |
| Pipe tables        | `\|`, `---`       | `#9C9890`     | `#6B6860`     | `punctuation.separator`                 |

### Palette Summary

The palette has five hues, all desaturated and earthy:

- **Rose/coral** — structural markers (headings, lists)
- **Amber** — emphasis markers (bold, italic)
- **Blue-gray** — links and references
- **Teal** — code and raw content
- **Lavender** — quotations

Everything else (horizontal rules, table pipes) gets neutral gray.

## Shiki Theme Definition

The custom theme is defined as a `ThemeRegistration` object in `lib/markdown/theme.ts`. Two themes are defined: `specs-light` and `specs-dark`. Shiki's `codeToHtml` is called with `themes: { light: specsLight, dark: specsDark }` and `defaultColor: false` so it outputs `class="shiki"` with CSS variables for both modes.

The page CSS then activates the correct set of variables:

```css
.shiki {
  background-color: #FAFAF8;
  color: #3C3C3C;
}

.shiki span {
  color: var(--shiki-light);
}

@media (prefers-color-scheme: dark) {
  .shiki {
    background-color: #1C1C1C;
    color: #C8C4BE;
  }

  .shiki span {
    color: var(--shiki-dark);
  }
}
```

This is the exact pattern Shiki recommends for dual-theme rendering with zero client JS.

## Blank Lines

Markdown blank lines are preserved literally — they create vertical rhythm the same way a writer uses them in a text file. No collapsing, no extra margin hacks.

## Selection & Cursor

- Text is selectable (it's a real page, not an image)
- Selection uses the mode-appropriate selection color from the color scheme table
- `::selection` is styled explicitly in CSS
- No visible cursor — this is not an editor

## Motion

None. Static page. No animations, no transitions, no scroll effects.
