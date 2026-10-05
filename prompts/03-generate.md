# Generation agent prompt

Implement one approved scenario from docs/coverage.csv. Inspect repository conventions and installed versions first. Use Context7 for the exact Playwright APIs needed, resolving the library before querying documentation; record sources. Use Playwright MCP to confirm the staging UI and locators if available.

Write TypeScript tests with meaningful assertions, role/label/test-ID locators, isolated fixtures, and supported test-data setup/cleanup. Route tests to the intended site/project. Avoid arbitrary sleeps, generated AEM IDs, and assertion weakening. Do not introduce writes outside the scenario's approved staging scope.

Run Playwright test discovery and the relevant tests. Report exact commands, actual results, changed files, coverage mapping, and unresolved limitations. Browser exploration is not proof that committed tests pass. If access or tooling is missing, retain a clearly marked draft and report the blocker.
