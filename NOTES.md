# Notes

## What's in CLAUDE.md, and what I left out

I kept it to four parts: a one-line description, the commands (dev, test, lint, plus how to run a single test file or filter tests by name), a short architecture note, and conventions. The architecture note covers things that only become clear after reading several files. `server.js` exports `app` and only listens when run directly, which is how the tests use `supertest` without opening a port. Routes reach data only through `db/store.js`. The store is module-level state, so tests that create users affect later tests. The conventions are concrete "use X, not Y" rules taken from the code and the lint config: CommonJS rather than ES modules, `{ error }` JSON with 400/404/201, and `node:test` + `supertest` rather than Jest.

I left out the file-by-file listing and endpoint details because Claude can read those from the code in seconds. Generic advice like "write tests" or "handle errors" didn't go in either, since it doesn't change what Claude does. There's nothing sensitive in the file: no values from `.env`, only a pointer to `.env.example`.

## Permission rules, and why the deny rules matter

- **Allow:** `npm test`, `npm run lint`, `node --test`. These are safe and run constantly, so prompting for them would just be noise.
- **Ask:** `git push` (it changes the shared remote) and `npm install` (it runs third-party install scripts).
- **Deny:** reading or editing `.env`, `git push --force` / `-f`, and `rm -rf`.

Without the `.env` deny rule, Claude could read real secrets into its context while "checking the config", and from there they could end up in output, logs or a commit. Without the force-push deny, one mistaken command could overwrite other people's history on the remote. Deny takes priority over ask, so force-push is blocked even though plain `git push` only asks.

I also found that the `.gitignore` lines started with spaces, so Git didn't actually ignore `.env` or `.claude/settings.local.json`. I removed the spaces and added `.DS_Store`. It's a reminder that one safeguard can quietly fail, which is why the deny rule is worth having alongside it.

## Verification

- `/memory`: TODO, confirm `CLAUDE.md` shows as loaded
- `/permissions`: TODO, confirm the allow / ask / deny rules are listed
