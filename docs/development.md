# Development

## Repository layout

- `client/`: Next.js browser application.
- `server/`: Express REST API and Prisma database layer.
- `docs/`: system, database, API, and development documentation.

The packages are not combined under an npm workspace; install and run each one separately.

## Local setup

Install dependencies:

```bash
npm --prefix client ci
npm --prefix server ci
```

Configure:

```bash
cp server/.env.example server/.env
cp client/.env.example client/.env.local
```

Create and migrate the database:

```bash
cd server
npx prisma migrate deploy
npm run seed
cd ..
```

Run both processes:

```bash
# Terminal 1
cd server
npm run dev
```

```bash
# Terminal 2
cd client
npm run dev
```

The client is expected at `http://localhost:3000`; the API is at `http://localhost:3001`.

## Scripts

### Client

| Command | Purpose |
| --- | --- |
| `npm run dev` | Starts Next.js development mode. |
| `npm run build` | Creates a production Next.js build. |
| `npm start` | Starts the built Next.js application. |
| `npm run lint` | Runs `next lint` with `next/core-web-vitals`. |

### Server

| Command | Purpose |
| --- | --- |
| `npm run dev` | Builds TypeScript, watches it, and runs `src/index.ts` through `ts-node`/Nodemon. |
| `npm run build` | Deletes `dist` and compiles TypeScript. |
| `npm start` | Runs build, then starts `dist/index.js`. |
| `npm run seed` | Destructively resets and seeds fixture data. |
| `npm test` | Placeholder; currently exits with an error. |

### PM2 example

`server/ecosystem.config.js` defines a process named `project-management` that runs `npm run dev`. It sets `NODE_ENV=development`. This is a process example, not a complete production deployment configuration.

## Environment

See [.env.example](../.env.example). Component-local templates are at [client/.env.example](../client/.env.example) and [server/.env.example](../server/.env.example).

The server reads:

- `DATABASE_URL`
- `PORT`, default `3001`

The client reads:

- `NEXT_PUBLIC_API_BASE_URL`
- `NEXT_PUBLIC_IMAGE_BASE_URL`
- `NEXT_PUBLIC_COGNITO_USER_POOL_ID`
- `NEXT_PUBLIC_COGNITO_USER_POOL_CLIENT_ID`

Next.js evaluates `NEXT_PUBLIC_*` values in browser-visible code. Do not place secrets in them.

## Coding conventions

The repository uses:

- TypeScript strict mode in both package configurations.
- App Router pages and `"use client"` components where browser state/hooks are needed.
- `@/` path aliases for `client/src`.
- Functional React components and React hooks.
- Redux Toolkit slices for small global UI state.
- RTK Query for API operations and cache tags.
- Express routers for route grouping and controllers for Prisma operations.
- Prettier; the client uses the Tailwind plugin and the server uses the Prisma plugin.

No ESLint server configuration, shared monorepo formatter, commit hook, or CI pipeline is present.

## Database development

Prisma schema changes should include a migration:

```bash
cd server
npx prisma migrate dev --name add_new_field
```

Use `npx prisma migrate status` to inspect migration state. Use `npx prisma studio` for a local browser database viewer.

The seed is destructive. Never run it before backing up a shared or production database.

## API development

Add:

1. A route in `server/src/routes`.
2. A controller in `server/src/controllers`.
3. A matching RTK Query endpoint in `client/src/state/api.ts`.
4. Documentation in [api.md](api.md).

Preserve HTTP response conventions unless intentionally changing them in a coordinated implementation and documentation update.

## Verification

No automated test suite is implemented. Recommended verification before a pull request:

```bash
npm --prefix server run build
npm --prefix client run lint
npm --prefix client run build
```

For API/database work, additionally:

```bash
cd server
npx prisma validate
npx prisma migrate status
```

Then manually exercise the changed endpoints against a disposable database and confirm the client behavior in a browser.
