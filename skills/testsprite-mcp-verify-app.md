---
name: Verify an app with the TestSprite MCP server
description: >-
  Drive TestSprite's autonomous testing agent from inside an AI coding
  assistant (Cursor, Claude Code, VS Code) to generate, run, and review a test
  suite for the current project — the "no-code, no-prompt" verification loop.
api: mcp/testsprite-mcp.yml
method: generated
source: >-
  Grounded in the real TestSprite MCP tool names (mcp/testsprite-mcp.yml) and
  installation docs (https://docs.testsprite.com/mcp/getting-started/installation).
operations:
  - testsprite_bootstrap_tests
  - testsprite_generate_code_summary
  - testsprite_generate_standardized_prd
  - testsprite_generate_frontend_test_plan
  - testsprite_generate_backend_test_plan
  - testsprite_generate_code_and_execute
  - testsprite_open_test_result_dashboard
  - testsprite_rerun_tests
---

# Verify an app with the TestSprite MCP server

Use this when the user wants TestSprite to autonomously generate and run tests
against the project currently open in the IDE.

## Prerequisites
- The TestSprite MCP server is configured: `npx @testsprite/testsprite-mcp@latest`
  with a valid `API_KEY` env var (key from https://www.testsprite.com/dashboard/settings/apikey).
- Auth model and scopes: see `authentication/testsprite-authentication.yml`
  (needs `write:tests` and `run:tests`).

## Steps
1. **Bootstrap** — call `testsprite_bootstrap_tests` to initialize the testing
   environment and configuration for the repo.
2. **Understand the codebase** — call `testsprite_generate_code_summary` to
   produce an architectural summary, then `testsprite_generate_standardized_prd`
   to derive a standardized PRD the plan will be grounded in.
3. **Plan** — call `testsprite_generate_frontend_test_plan` for a web UI or
   `testsprite_generate_backend_test_plan` for a REST API (choose by project type).
4. **Generate & execute** — call `testsprite_generate_code_and_execute` to
   generate the runnable suite and run it.
5. **Review** — call `testsprite_open_test_result_dashboard` to inspect
   failures, recordings, and failure bundles; fix the code the agent flags.
6. **Re-verify** — after fixing, call `testsprite_rerun_tests` to replay the
   previously generated cases and confirm green.

## Rules
- This is a loop, not one-shot: create -> run -> read failures -> fix -> rerun;
  coverage compounds each pass.
- Respect rate limits (60 triggers/min/key) and honor `Retry-After` on backoff —
  see `conventions/testsprite-conventions.yml`.
- On errors, branch on the error/exit codes in `errors/testsprite-error-codes.yml`
  (e.g. `INSUFFICIENT_CREDITS` is non-retriable; `CLIENT_TOO_OLD` means upgrade).
