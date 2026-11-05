# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express REST API (users plus a health check) with an in-memory store. It is the starter repo for the Claude Code course.

## Commands

```sh
npm install
npm run dev                                      # API on http://localhost:3000 (node --watch, restarts on change)
npm test                                         # node:test runner, finds tests/*.test.js
npm run lint                                     # eslint .
node --test tests/users.test.js                  # one test file
node --test --test-name-pattern="404" tests/     # tests whose name matches
```

CI (`.github/workflows/ci.yml`) runs `npm install`, `npm run lint` and `npm test` on Node 22 for every push and PR. Make sure lint and tests pass before you commit.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`) and exports `app`. It calls `listen` only when run directly (`require.main === module`), so tests import `app` and drive it with `supertest` without opening a port.
- `routes/<resource>.js` holds one `express.Router()` per resource. To add a resource, create a file there and mount it in `server.js`.
- `db/store.js` is the only data-access layer: plain functions over a module-level array. Data resets on every restart. Because tests share one process, a test that creates a user changes the state that later tests see.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES `import`/`export`. ESLint sets `sourceType: "script"`.
- Routes never touch data directly. Add or extend functions in `db/store.js` and call them from the route.
- Send errors as JSON `{ error: "<message>" }` with the matching status: `400` for invalid input, `404` for a missing resource. A successful create returns `201` with the new object.
- Parse route params with `Number(req.params.id)`. The store compares ids with strict equality.
- Write tests with `node:test` + `node:assert` + `supertest` against the exported `app`. Don't use Jest or Mocha.
- ESLint ignores unused params named `req`, `res`, `next` or `_`.
- Config comes from environment variables (`PORT`, see `.env.example`). Don't read or edit `.env`.
- This is a course repo: the deliverables are `CLAUDE.md`, `.claude/settings.json` and `NOTES.md`. Don't change app code unless asked.
