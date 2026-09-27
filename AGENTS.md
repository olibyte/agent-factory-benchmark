# Agent instructions

This project uses Agent Skills from `.agents/skills/`. Claude Code also needs matching links under `.claude/skills/`.

Invoke `handoff` before you switch coding agents. Codex: `$handoff`. Claude Code, Cursor, and Antigravity: `/handoff`.

## Models

Subagents read `.agents/models.md` when that file exists. A missing file means inherit the current chat model. Values `inherit` and `auto` mean the same. Do not hardcode a vendor model name in a skill.

## Project

Add project-specific rules below this heading.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
