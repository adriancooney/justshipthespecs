# app/layout.tsx

Root layout. Handles font loading, global CSS, meta tags, and the `<html>`/`<body>` shell.

## Font Loading

Geist Mono is loaded via `next/font/google`:

```tsx
import { Geist_Mono } from "next/font/google";

const geistMono = Geist_Mono({
  subsets: ["latin"],
  variable: "--font-geist-mono",
});
```

The font variable is applied to `<html>` so it cascades everywhere:

```tsx
<html lang="en" className={geistMono.variable}>
```

## Body

```tsx
<body className="font-mono bg-[#FAFAF8] text-[#3C3C3C] dark:bg-[#1C1C1C] dark:text-[#C8C4BE]">
  {children}
</body>
```

- `font-mono` maps to the `--font-geist-mono` variable (configured in `globals.css` via Tailwind's `@theme` directive or the font-mono override)
- Background and text colors set for both light and dark mode
- Tailwind's `dark:` variant uses `prefers-color-scheme` by default in v4

## Metadata

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "just ship the specs",
  description: "code is dead. just ship the specs.",
  openGraph: {
    title: "just ship the specs",
    description: "code is dead. just ship the specs.",
  },
  twitter: {
    card: "summary",
    title: "just ship the specs",
    description: "code is dead. just ship the specs.",
  },
};
```

## Global CSS

`globals.css` is imported in the layout:

```tsx
import "./globals.css";
```

See `app/globals.md` for the full CSS spec.
