# Memory — Prisma Postgres + NestJS setup

Last updated: 2026-07-27, 1:30 PM

## What was built

- Linked Prisma Postgres database `db_cms34b5if0zyjy6g3euij0j6c` (project "Hackathon", eu-central-1) to the app. DATABASE_URL lives in `.env` (gitignored, value never committed).
- Installed deps: `@prisma/client`, `@prisma/adapter-pg`, `pg`, `dotenv`, `@nestjs/config`; dev: `prisma` 7.9.0, `@types/pg`.
- `prisma/schema.prisma` (empty — no models yet), `prisma.config.ts`, generated client at `src/generated/prisma` (via `npx prisma generate`).
- `src/lib/database/prisma.service.ts` (extends PrismaClient, PrismaPg adapter, ConfigService-injected DATABASE_URL, $connect/$disconnect lifecycle hooks) and `src/lib/database/prisma.module.ts` (`@Global()`, exports PrismaService), imported once in `AppModule` alongside `ConfigModule.forRoot({ isGlobal: true })` — per AGENTS.md structure.
- `tsconfig.build.json`: added `"include": ["src/**/*"]`.

## Decisions made

- Used the **direct** `postgres://` connection string (not the `prisma+postgres://` Accelerate one) because the NestJS guide's `PrismaPg` pg-driver-adapter setup requires a standard Postgres URL.
- Generator `moduleFormat = "cjs"` in schema.prisma — app is CommonJS (no `"type": "module"` in package.json); ESM client output breaks under tsc CJS emit.
- Did NOT add `"type": "module"` to package.json (Prisma guide suggests it, but it would break the default NestJS CJS scaffold).

## Problems solved

- `prisma postgres link` (CLI, both env-var and `--api-key` forms) rejected the provided API key as invalid → switched to **Prisma MCP tools** (`create_prisma_postgres_connection_string`) which worked.
- Server crashed on boot: `ReferenceError: exports is not defined in ES module scope` — generated client used `import.meta.url`, TS emitted it into CJS output, Node 24 syntax detection misclassified the file. Fixed with `moduleFormat = "cjs"` + regenerate.
- `prisma.config.ts` was being compiled into `dist/` (no `include` in tsconfig.build.json), producing `dist/src/main.js` instead of `dist/main.js`. Fixed with the `include` above.
- `npx prisma migrate dev --name init` reported "Already in sync" (empty schema, fresh DB) and created no migration; client was generated separately with `npx prisma generate`.

## Current state

- Server runs fine (`npm start`, port 3000, `GET /` → 200). DB connectivity verified (`SELECT 1` via `prisma db execute`).
- Schema has **no models** — migration history not yet initialized (nothing to migrate).
- Note: `prisma init` auto-installed Prisma agent-skills into `.claude/skills/`, `.windsurf/skills/`, `.agents/skills/` and updated `skills-lock.json`.

## Next session starts with

Define the first models in `prisma/schema.prisma` (hackathon domain TBD), then `npx prisma migrate dev --name <name>` — this creates the real initial migration and regenerates the client. Feature modules go in `src/module/<name>/` per AGENTS.md; inject `PrismaService` via constructor (it's globally available).

## Open questions

- What domain models does the hackathon app need?
- Should `src/generated/` be gitignored or committed? (Currently untracked either way — decision pending.)
