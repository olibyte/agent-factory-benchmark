# agent-factory-benchmark

This is the benchmark app for Factory v0 in [olibyte/agent-toolkit](https://github.com/olibyte/agent-toolkit): a deliberately small task tracker whose product brief comes next.

## Stack

- Next.js 16.3.6
- React
- TypeScript
- Tailwind CSS
- Node 24
- npm
- Vitest

## Commands

```bash
npm ci
npm run dev
npm run lint
npm run build
npm test
```

## Agent skills

Skills are installed under `.agents/skills/`, with symlinks for Claude Code under `.claude/skills/`, and pinned in `skills-lock.json`. They come from three sources:

- [olibyte/agent-toolkit](https://github.com/olibyte/agent-toolkit) @ `3a69c94ae8a0b5990a1c13aeb0f8e978269f6756`
- [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) @ `063bee94c3f4df8453406c830b0a7df0f2860278`
- [vercel/next.js](https://github.com/vercel/next.js) @ `a758ffcf501f6f1ddb03175bd1033508424c261e` (`skills/` subdirectory)

To update a source, rerun its `skills add` command with a new commit.
