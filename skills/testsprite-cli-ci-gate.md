---
name: Gate CI on TestSprite results with the CLI
description: >-
  Use the TestSprite CLI (`testsprite`) in a CI/CD job to create, run, and wait
  on tests, then branch the pipeline on stable exit codes and JSON output.
api: cli/testsprite-cli.yml
method: generated
source: >-
  Grounded in the real TestSprite CLI command surface (cli/testsprite-cli.yml)
  and the CI/CD + exit-codes reference (https://docs.testsprite.com/cli/integrations/ci-cd).
operations:
  - setup
  - project list
  - test run
  - test wait
  - test result
  - test failure summary
  - usage
---

# Gate CI on TestSprite results with the CLI

Use this to fail a pipeline when TestSprite tests regress.

## Prerequisites
- Install: `npm install -g @testsprite/testsprite-cli` (binary `testsprite`).
- Authenticate non-interactively with an API key that has `run:tests` and
  `read:tests` scopes (`authentication/testsprite-authentication.yml`).

## Steps
1. **Authenticate** — `testsprite setup --no-agent` (credentials-only) using the
   API key from the CI secret store.
2. **Locate the project** — `testsprite project list` to find the project id.
3. **Run and block** — `testsprite test run --wait` to trigger a run and block
   until it reaches a terminal status.
4. **Read the verdict** — the process exit code IS the gate:
   `0` pass; `1` a non-passed run; `3` auth; `11` rate limited (honor Retry-After);
   `12` insufficient credits. Full table in `errors/testsprite-error-codes.yml`.
5. **On failure** — `testsprite test failure summary` for a one-screen triage
   card, or `testsprite test result` for run history; emit the stable JSON and
   pipe into `jq` for downstream steps.

## Rules
- Never rely on `Ctrl-C` to stop a run in CI — use `testsprite test cancel`.
- Rate limit is 60 triggers/min/key; batch and back off on exit code 11.
- Check `testsprite usage` if runs stop — exit code 12 means top up credits.
