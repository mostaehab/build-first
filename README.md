# build-first

Stops Claude Code asking for API keys mid-session. It builds the whole thing against
environment variables, records every credential it needs in
`CONFIGURATION_REQUIREMENTS.md` as it goes, and configures at the end.

## Install

```
/plugin marketplace add mostaehab/build-first
/plugin install build-first@build-first
```

Then start a new session (the rule loads at session start).

## How it works

- `hooks/hooks.json` — SessionStart + SubagentStart hook that injects `rules/build-first.md`
  into every session and every subagent. This is what makes it always-on rather than
  something Claude has to remember.
- `rules/build-first.md` — the policy: read credentials from env, fail loud, log the
  requirement, keep building. Never ask mid-build.
- `skills/configuration-requirements/` — the format of `CONFIGURATION_REQUIREMENTS.md` and
  the end-of-build reconciliation (greps the code for every env read so nothing is missed).
  Invoke directly with `/configuration-requirements` any time.

Say "stop build-first" or "normal mode" to drop it for the rest of a session.
