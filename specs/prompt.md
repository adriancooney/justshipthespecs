# Project Prompt Overlay

This is a small, single-page site. It should be completable in a single iteration.

## Critical: Use npm

npm is pre-installed. Do NOT attempt to install pnpm, yarn, or corepack. Use `npm install` and `npm run build`.

## Critical: No linter

Do not install or configure Biome, ESLint, or any linter. The build passing is sufficient.

## Critical: No image generation

Do not generate OG images or favicons. These are explicitly skipped in the specs (see overview.md).

## Critical: No system packages

Do not run `apt-get`, `dnf`, `yum`, or any system package manager. If agent-browser fails due to missing system deps, skip visual verification and move on.

## Build order

1. Read all specs.
2. Write all files (package.json, configs, then source files).
3. `npm install`
4. `npm run build`
5. Verify with curl if the build passes.
6. Done.
