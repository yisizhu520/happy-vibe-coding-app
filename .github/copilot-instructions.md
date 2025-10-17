# copilot-instructions for happy-* repos

This brief guide helps AI coding agents become productive in this monorepo (happy-cli / happy-client / happy-server).

1) High-level architecture (what to read first)
- `happy-cli/` — CLI and Claude integration. Key dirs: `src/claude/`, `src/api/`, `bin/` (entry points `happy.mjs`, `happy-mcp.mjs`).
- `happy-client/` — React Native Expo client. Key dir: `sources/` (UI, `sync/`, `auth/`). Uses `@/*` path alias -> `sources/*`.
- `happy-server/` — Node/Fastify API + Prisma. Key dir: `sources/`, `prisma/`.

2) Essential commands (examples from package.json)
- CLI (development & daemon):
  - build then run: from `happy-cli/` run `yarn build` then `./bin/happy.mjs` (scripts: `build`, `start`).
  - dev (no build): `yarn dev` runs `tsx src/index.ts`.
  - tests: `yarn test` runs `yarn build && vitest` (note: build is required before tests in this repo).
- Client (React Native / Expo): from `happy-client/` use yarn (repo uses yarn@1):
  - `yarn start` / `yarn web` / `yarn ios` / `yarn android`
  - typecheck: `yarn typecheck`
  - test: `yarn test` (vitest/jest-expo depending on package.json)
- Server: from `happy-server/`:
  - dev: `yarn dev` (tsx with `.env`),
  - start: `yarn start`,
  - migrate: `yarn migrate` / `yarn migrate:reset` (Prisma),
  - run deps locally: `yarn db` (postgres docker), `yarn redis`, `yarn s3` (minio).

3) Project-specific conventions and gotchas
- Always build the CLI before running daemon-related scripts — many commands expect the compiled `bin/happy.mjs` produced by the build.
- Package manager: yarn v1 (package.json contains `packageManager: "yarn@1.22.22"`). Use `yarn` for client and repo tasks.
- TypeScript strictness: codebase prefers strict types. Use `yarn typecheck` / `tsc --noEmit` where available.
- Tests: Vitest is used across packages; some tests expect the build artifact (see `happy-cli` tests). Run `yarn build` if tests fail unexpectedly.
- Path aliases:
  - `happy-client` uses `@/*` -> `sources/*` (check `tsconfig.json` / bundler config).

4) Integration points & external dependencies to know
- Claude integration: `@anthropic-ai/claude-code` is used in `happy-cli` (`src/claude/`). Look at `src/claude/loop.ts`, `runClaude.ts`, `session.ts` for session lifecycle and resume behavior.
- Real-time sync: Socket.IO client/server used — see `src/api/apiSession.ts` (CLI) and `happy-server/sources/` (server-side socket handling).
- Encryption / auth: `tweetnacl` / `tweetnacl` wrappers used for E2E encryption and challenge-response (see `src/api/auth.ts`, `src/api/encryption.ts`).
- Persistence: `persistence.ts` in CLI stores keys and config in user home directory. Session files (claude) are under user profile (`~/.claude/...`) during interactive runs — code in `src/claude/*` expects file-based session JSONL output.
- Database/infra: server uses Prisma (`prisma/schema.prisma`) and scripts to run postgres/minio locally.

5) Common code patterns (examples)
- Dual Claude modes: CLI supports interactive PTY-based sessions (spawned `claude` process) and SDK mode (`@anthropic-ai/claude-code`) for remote control — check `src/claude/loop.ts` and `src/codex/runCodex.ts`.
- Session resume: resuming a Claude session creates a new session ID and a new file containing updated sessionId fields (see `happy-cli/CLAUDE.md` notes). When reasoning about history, reference `src/claude/utils/*` tests and fixtures.
- Logging: prefer file-based logs (to avoid interfering with Claude terminal output); look for `src/ui/logger.ts` and log file locations in `CLAUDE.md`.

6) What to inspect first for common tasks
- Fix a CLI bug: start at `happy-cli/src/index.ts` -> follow into `src/claude/` and `src/api/`.
- Add a mobile UI change: edit `happy-client/sources/` components; run `yarn start` for Expo.
- Backend API change: edit `happy-server/sources/`, run `yarn dev`, run prisma migrations when schema changes.

7) Security & safety notes for agents
- Sensitive keys: private keys and tokens are stored on-disk (see `persistence.ts`) — do not output secrets.
- Network calls and E2E encryption are enforced; prefer using existing helper functions in `src/api/` for signing and encrypting.

8) Quick references (files to open)
- `happy-cli/src/claude/loop.ts` — session lifecycle
- `happy-cli/src/api/api.ts` / `apiSession.ts` — server comms
- `happy-client/sources/` — client UI and `sync/` engine
- `happy-server/prisma/schema.prisma` and `happy-server/sources/main.ts`
- `happy-cli/CLAUDE.md`, `happy-client/CLAUDE.md` — repository-specific agent instructions and gotchas

If anything in this file is unclear or you want more examples (e.g., common PR changes, test harness commands, or more internal file maps), tell me which area to expand and I will iterate.
