# Clean Room Build

## Definition

A clean room build is a project built entirely by an agent from specs alone. No existing codebase. No starter template. No human-written code to reference. The agent reads the specs directory and produces a working application from scratch.

The code is a build artifact — generated, disposable, regenerable. The specs are the source of truth.

## How It Works

1. Agent receives the specs directory as its only input.
2. Agent reads all spec files to understand the full picture: technology choices, architecture, style, content, behavior.
3. Agent scaffolds the project (package.json, tsconfig, Next.js config, etc.) based on the technology decisions in `overview.md`.
4. Agent implements each spec file as described — no interpretation, no improvisation.
5. Agent runs the build to verify the output compiles.
6. Done. The code exists to be deployed, not to be read.

## What the Specs Must Cover

For a clean room build to work, the specs must be complete enough that an agent makes zero judgment calls. Every decision is pre-made.

| Concern              | Must be specified                                      |
| -------------------- | ------------------------------------------------------ |
| Technology           | Framework, language, package manager, all dependencies |
| Architecture         | File structure, module boundaries, data flow           |
| Behavior             | What each route/component does, edge cases, non-goals  |
| Style                | Visual design, colors, typography, layout rules        |
| Content              | Literal copy, links, assets                            |
| Contracts            | API shapes, type definitions, function signatures      |
| SEO / Meta           | Titles, descriptions, OG tags                          |
| Deployment           | Platform, environment, build commands                  |

If a spec leaves a gap, the agent fills it with its own judgment — and that's a spec bug.

## Properties

### Reproducible

Two agents given the same specs should produce functionally identical output. The code won't be character-for-character identical, but the behavior, structure, and appearance will be.

### Disposable Code

If you delete the generated code and re-run the build, you get the same product. This means:

- No hand-editing generated code. If something is wrong, fix the spec.
- No accumulating tech debt — there's no debt if you regenerate from scratch.
- No refactoring — just re-spec and rebuild.

### Versionable

The specs are the thing you version control. Code diffs are meaningless — spec diffs are the real changelog. A PR is a spec change, not a code change.

### Portable

The specs are agent-agnostic. Any sufficiently capable agent (Claude, GPT, Gemini, future models) should be able to read the same specs and produce a working build. The specs are a contract with the machine, not a contract with a specific model.

## What Changes

| Before (code-first)         | After (spec-first)                    |
| --------------------------- | ------------------------------------- |
| you write code              | you write specs                       |
| you read code to understand | you read specs to understand          |
| you refactor code           | you re-spec and rebuild               |
| you review code in PRs      | you review specs in PRs               |
| code quality matters        | spec quality matters                  |
| tech debt accumulates       | tech debt doesn't exist               |
| onboarding = read the code  | onboarding = read the specs           |
| the code is the product     | the specs are the product             |

## Build Constraints

The build runs on Vercel via `@vercel/dream`. Optimize for speed and cost.

- Use `npm` — it's pre-installed in the Vercel build env. Do NOT install pnpm, yarn, or any other package manager.
- Do NOT install system-level packages (no `apt-get`, no `dnf`). If a tool needs system deps that aren't present, skip that step.
- Do NOT generate image assets (OG images, favicons). These are skipped for v1.
- Do NOT install or configure a linter. The build passing is sufficient.
- Minimize tool calls. Write all files first, then build, then verify. Target: under 50 tool calls.

## Build Command

The clean room build is triggered by pointing an agent at the specs directory with an instruction like:

```
build this project from the specs in ./specs/
```

No other context needed. The specs are self-contained.

## Verification

The agent must verify its work. An unverified build is a failed build.

### Required

1. `npm run build` completes with zero errors. This is the only hard gate.

### Optional

2. Start the dev/production server and `curl http://localhost:3000` — verify HTML output.
3. `curl -H "Accept: text/markdown" http://localhost:3000` — verify raw markdown response.
4. If agent-browser is available, take a screenshot to verify visual output. Do NOT install system packages to make it work — if it fails due to missing deps, skip visual verification.

### Failure protocol

If `npm run build` fails, fix and retry. Max 3 attempts before stopping.
