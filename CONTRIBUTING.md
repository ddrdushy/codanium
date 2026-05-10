# Contributing to Codanium

Thanks for your interest in contributing! Codanium is an AI software-delivery platform. This repo is a monorepo containing the web app, desktop app, VS Code extension, and architecture docs.

## Repository layout

| Directory | What it is |
|-----------|------------|
| [`ai-team-studio/`](ai-team-studio/) | Web application (Next.js 16, Prisma, PostgreSQL, BullMQ) |
| [`codanium-desktop/`](codanium-desktop/) | Desktop app (Tauri v2, React, TypeScript) |
| [`ai-team-studio-vscode/`](ai-team-studio-vscode/) | VS Code extension (deprecated — being replaced by `codanium-desktop`) |
| [`AI_Team_Studio_v4/`](AI_Team_Studio_v4/) | Enterprise architecture documentation |

## Code of Conduct

By participating, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md). Be respectful and constructive.

## How to contribute

### Reporting bugs

- Search [existing issues](../../issues) first.
- If new, open an issue with: title, repro steps, expected vs. actual behavior, environment (OS, Node version, etc.), and which subproject it affects.

### Suggesting enhancements

- Open an issue labelled `enhancement`.
- Describe the use case and why it matters before proposing a specific implementation.

### Pull requests

1. Fork the repo and branch from `main`.
2. Scope the PR to one subproject when possible — note which one in the PR title (e.g. `[web] add foo`, `[desktop] fix bar`).
3. Add tests for new behavior.
4. Update relevant documentation.
5. Make sure lint, type check, and tests pass for the affected subproject.
6. Open the PR with a clear description and link to any related issue.

## Development setup

Each subproject has its own toolchain. Pick the one you're working on:

### Web app — `ai-team-studio/`

```bash
cd ai-team-studio
docker compose up -d db redis        # Postgres + Redis
npm install
cp .env.example .env.local           # then edit values
npx prisma db push
npx prisma db seed
npm run dev                          # http://localhost:3000
```

### Desktop app — `codanium-desktop/`

```bash
cd codanium-desktop
npm install
npm run tauri dev                    # launches the desktop window
```

Requires Rust + Tauri prerequisites — see [tauri.app/start/prerequisites](https://tauri.app/start/prerequisites/).

### VS Code extension — `ai-team-studio-vscode/`

```bash
cd ai-team-studio-vscode
npm install
npm run package                      # produces .vsix
```

> Note: The VS Code extension is being phased out in favor of the standalone desktop app. New feature work should target `codanium-desktop/`.

## Commit style

- Concise imperative subject (≤ 72 chars): `Fix BRD persistence in PM gate`
- Body explains *why*, not *what*. Reference issues with `Closes #123` where relevant.

## Questions?

Open a [discussion](../../discussions) or comment on an issue.
