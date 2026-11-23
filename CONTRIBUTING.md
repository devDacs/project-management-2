# Contributing

This guide describes the workflow visible in the repository and marks recommendations that are not established project policy.

## Development setup

Follow the repository [README](README.md#setup). Install dependencies separately because `client` and `server` are separate npm projects:

```bash
npm --prefix client ci
npm --prefix server ci
```

Create `server/.env` and `client/.env.local` from their examples. Apply migrations and seed only a disposable development database.

## Branching

No team-specific branching policy is established in the repository. A practical convention is feature branches from `main`, for example `feature/task-filters`, and one pull request per focused change.

## Commits

The recent history uses concise imperative commit subjects with conventional-type prefixes, such as `feat(client): ...`, `fix(server): ...`, and `chore(server): ...`.

This convention is observed in recent history but is not enforced by repository configuration. Contributors may use a conventional commit format for consistency:

```text
<type>(<scope>): <summary>
```

Common types include `feat`, `fix`, `docs`, `chore`, `test`, and `refactor`.

## Formatting and linting

Both applications contain Prettier configuration.

```bash
npx prettier --write client/src client/*.json client/*.mjs
npx prettier --write server/src server/prisma
```

Run the configured Next.js lint command:

```bash
npm --prefix client run lint
```

There is no server lint script.

TypeScript/Prisma conventions currently present in the codebase include strict client/server compiler settings, semicolon-terminated TS/TSX statements, and Prisma schema formatting with `prettier-plugin-prisma`.

## Testing expectations

No automated test framework or test suite is configured. A pull request should include manual verification where practical and any test command that exists in the final change.

At minimum, run:

```bash
npm --prefix server run build
npm --prefix client run lint
npm --prefix client run build
```

Before submitting API or database changes, also verify migrations against an isolated PostgreSQL database and state what manual endpoints or seed scenarios were exercised.

## Pull requests

For a maintainable pull request:

- Keep documentation and source changes scoped together.
- Explain user-visible behavior and API/database effects.
- Include environment variable changes in the appropriate `.env.example`.
- Include migration files for schema changes; do not edit existing migrations unless the project explicitly resets migration history.
- Include screenshots or a concise screen recording for significant UI changes.
- Document manual test steps and results.
- Call out breaking API, authentication, or seed-data changes.

## Code review expectations

No automated review policy is configured. Reviewers should check:

- Whether changes match the existing App Router, RTK Query, Express router/controller, and Prisma patterns.
- Whether public data shapes remain consistent with the API `{ success, data }` convention.
- Whether new environment variables are public-safe and documented.
- Whether database relations, deletion behavior, and required fields are preserved.
- Whether any authentication or authorization claim is backed by implementation. The server currently sends no 401/403 response and does not verify bearer tokens.
- Whether destructive seed or migration behavior is clearly isolated to development.

## Adding API endpoints

Follow the current structure:

1. Add the route in `server/src/routes/`.
2. Add controller logic in `server/src/controllers/`.
3. Use Prisma for database access.
4. Return success responses with `{ success: true, data: ... }` for successful reads/writes.
5. Add a matching RTK Query endpoint in `client/src/state/api.ts` when the client needs it.
6. Update [API documentation](docs/api.md).
