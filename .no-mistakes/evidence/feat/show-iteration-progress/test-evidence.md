# Iteration progress: test evidence

Built target commit 3ce692557d5a5c79368113057b907a8d49096e7b and drove the normal CLI with authenticated Codex, an 83-column by 30-row PTY configured before process startup, and continuous master draining. No mock agent was used for the live checks. Telemetry and sleep inhibition were disabled. All disposable repositories were inside the assigned worktree.

The runs exited with status 130 because the driver used Ctrl+C to dismiss capped done screens or request graceful shutdown. Product logs show the expected cap or graceful-stop reason, without provider errors.

## Capped run counts only finished iterations, including a failed iteration

0/2 during iteration 1; 1/2 during iteration 2; 2/2 with one commit after the intentional failure.

## Resume restores previously finished iterations

Existing real run resumed at 2/3; running iteration 3 stayed at 2/3, then finished at 3/3.

## Uncapped run omits progress

Timer is followed directly by token totals, both before and after token usage arrives.

## Zero iteration cap finishes without launching an agent

0/0 and max iterations reached (0); no agent start in the product log.

## High-token boundary (synthetic supplemental check)

Executed the original and fixed renderer with elapsed=08:07:17, completed=12, max=20, input=87,300,000, output=860,000, commits=11. Original exact/estimated rows required 65/68 columns and visibly clipped the commit label. Fixed rows require 55/58 columns and preserve all values and the full label inside the 63-column content area. Existing focused tests also cover 99/100, 999/1000, cache-token totals, and 9999/10000 with compact labels. These controlled-state checks are not live provider runs.

## Commands and capture steps

1. `pnpm run build`
2. `pnpm exec vitest run src/renderer.test.ts src/core/orchestrator.test.ts -t 'renderStats|complete crowded stats|shows finished iterations|counts an iteration only after|rate.limit.*retry|retries the same iteration'`
3. `python3 .local-test/drive.py`: launch `node dist/cli.mjs` with `--agent codex --current-branch --prevent-sleep off --meteor-frequency 0`; cap 2, resume with cap 3, no cap plus graceful interrupt, and cap 0. The temporary PATH wrapper adds only Codex `--ephemeral`.
4. `pnpm exec vitest run .local-test/width-visual.test.ts`: execute original/fixed renderers and assert complete rendered stats plus width.
5. `python3 .local-test/verify-evidence.py`: replay captured ANSI bytes with pyte, assert progress sequences against product logs, and create rendered terminal HTML.
6. Serve evidence with `python3 -m http.server 8769 --bind 127.0.0.1 --directory <evidence-directory>`; capture full-page PNGs with Chrome DevTools and inspect them. Browser direct file output did not allow the external evidence directory; returned screenshot bytes were saved there using Python.

No linter, formatter, static analyzer, complete repository test suite, push, PR, or CI phase was run. Temporary drivers, sandbox repositories, Python test dependencies, historical-renderer copy, and the generated dist build are removed from the worktree after validation. Existing node_modules dependencies are retained.
