BUILD-FIRST MODE ACTIVE — credentials are configured last, never mid-build.

# Build First, Configure Last

A missing credential is never a reason to stop building. When the code needs an API key,
secret, token, connection string, OAuth client, webhook URL, project/account id or
service-account file: do not ask the user for it, do not pause the task, do not switch to
something easier. Write the code that reads it from the environment, record the
requirement, keep building.

This holds for the whole session unless the user says "stop build-first" or "normal mode".

## When you hit a credential

1. Read it from an env var with an obvious name (`STRIPE_SECRET_KEY`, `RESEND_API_KEY`) —
   no inline default, no placeholder that could pass for a real value. If the project
   already has a config pattern (a config module, settings class, `env.ts`), use that one.
2. Fail loud at the boundary: one check where the value is first used, with a message that
   names the variable and points at `CONFIGURATION_REQUIREMENTS.md`. Not a silent
   `undefined` travelling three layers down.
3. Append the requirement to `CONFIGURATION_REQUIREMENTS.md` at the repo root the moment
   you hit it — create the file if it is not there. Use the format in the
   `configuration-requirements` skill. Append as you go; never plan to reconstruct the
   list from memory at the end.
4. Add the variable to `.env.example` with an empty value, and check `.env` / `.env.local`
   is gitignored. Add it if it is not.
5. Keep building the next thing.

## Never

- Never ask the user for a key, secret or account "so we can continue".
- Never invent, guess or scaffold a fake-looking credential value. Empty or absent only.
- Never put a real secret in source, in a committed file, or in
  `CONFIGURATION_REQUIREMENTS.md` — that file says how to get values, never the values.
- Never leave a feature half-written because its key is missing: finish the code path,
  success branch and failure branch both.
- Never silently swap in a different provider, or stub something out, without saying so in
  chat.

## Running it before the keys exist

Only when the user needs to run or see the feature before configuring: let the
unconfigured path return obviously-fake data behind the same env check — the smallest form
that works, marked `shortcut: stubbed until <VAR> is set, delete once configured` — and
say in chat that it is stubbed. Otherwise leave it failing loud. Do not build a mock layer
nobody asked for.

## Genuinely blocked

If a credential is needed to *discover* what to build — an undocumented API's real response
shape, a schema that exists only inside the live account — build every other part of the
task first, then ask for that one thing at the end, with the question already narrowed to
exactly what you need.

## End of the build

When the requested work is built, in the same turn, unasked:

- Reconcile: grep the code for environment reads and make sure
  `CONFIGURATION_REQUIREMENTS.md` lists every one, with nothing stale left in it.
- Hand the user the file, plus one line for each thing you stubbed or left failing loud.
- Offer to wire the values in and run it once they have them.

If the user volunteers a credential, take it and use it straight away. This is about never
blocking on secrets, not about refusing them.
