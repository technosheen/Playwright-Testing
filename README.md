# AEM multi-site Playwright testing plan

Step-by-step implementation guide for eight Adobe Experience Manager sites, using agents, Playwright MCP, Context7, and the Playwright test runner.

This is a planning scaffold, not an installed or executed test suite. Domain URLs, business requirements, credentials, and AEM deployment details must be supplied before generating site-specific tests. MCP configuration below is illustrative and must be adapted to your agent client. No live sites or documentation tools were accessed to validate this scaffold.

## 1. Confirm architecture and scope

Complete [the site inventory](templates/site-inventory.csv) for all eight sites. Identify:

- AEM as a Cloud Service, AEM 6.5/managed hosting, or Edge Delivery Services. Edge Delivery Services may need a different authoring and delivery test plan; do not assume a Dispatcher or classic Author UI exists.
- Public Publish URL, optional direct Publish URL, Author URL, and staging environment for each site.
- Shared code, templates, Core Components versions, custom components, client libraries, and release ownership.
- Content roots, languages, locale URL rules, MSM blueprints/live copies, Experience Fragments, Content Fragments, and headless endpoints where used.
- Authentication, consent platform, Adobe Analytics/Launch/Tags, Adobe Target, forms, search, and external integrations where used.
- Supported browsers, viewports, roles, and critical business journeys.

Default scope: visitor journeys through the public delivery URL. Add isolated Author tests only when authoring is a required business workflow. Test through the CDN/Dispatcher to exercise what visitors actually receive; direct Publish checks are optional diagnostics, not a substitute.

**Exit gate:** all eight sites have an owner, a test environment, known capabilities, and critical journeys. Mark unused features N/A with a reason.

## 2. Set up the repository and runner

Use an actively supported Node.js version compatible with your selected Playwright release. In a new test repository:

```bash
npm init playwright@latest
npm install -D @axe-core/playwright
npx playwright install --with-deps
```

Choose TypeScript. Use the generated package scripts or add scripts for smoke, regression, and reports. Commit the package lockfile and pin dependency versions through it. CI should use `npm ci`, not regenerate the project.

Recommended structure:

```text
config/sites.ts
playwright.config.ts
tests/publish/shared/
tests/publish/sites/site-01/
tests/publish/sites/site-02/ ... site-08/
tests/author/
tests/cross-domain/
fixtures/
helpers/
docs/coverage.csv
docs/sources.md
docs/discovery/site-01.md ... site-08.md
prompts/
```

Keep shared tests parameterized by explicit site capabilities. Site-specific tests must run only against their intended site, using `testMatch`, project metadata, or a validated site fixture. Do not multiply every spec by every site unintentionally.

Configure one Chromium project per site initially, with its own `baseURL`. Add WebKit, Firefox, and mobile viewport projects according to the support contract. Keep Author projects separate and excluded from ordinary Publish runs. Missing required URLs should fail configuration rather than silently omit a site.

Configure `forbidOnly` in CI, HTML and machine-readable reports, traces on first retry, screenshots on failure, and at most a small number of CI retries. Track first-attempt failures separately. Add `.env`, authentication state, reports, and test artifacts to `.gitignore`. Load environment variables explicitly if you use a `.env` file; Playwright does not automatically load it.

**Exit gate:** test discovery lists the expected projects and routes each spec correctly. One approved baseline test runs locally and in CI against each site.

## 3. Connect Playwright MCP and Context7

Install/configure both servers in the agent client, using [the example](templates/mcp-config.example.json) as a starting point. Consult the linked official server documentation for your client-specific format, current options, authentication, and package versions. Replace `latest` with verified versions for a reproducible team setup.

- Playwright MCP lets the agent navigate pages, inspect accessible structure, interact with UI, and collect evidence.
- Context7 retrieves library documentation to help implement APIs correctly. It does not determine business acceptance criteria.
- The Playwright test runner executes committed tests and is the source of pass/fail evidence. An MCP exploration session is not a regression test run.
- AEM documentation and your repository provide product architecture, component contracts, and publishing rules. Use official Adobe documentation when Context7 has no relevant or version-matched AEM source.
- Repository tools can inspect code and prepare reviewable changes. CI tools can retrieve test reports. Additional integrations are optional.

Smoke-check the connections: have the agent open one permitted staging page through Playwright MCP, then resolve Playwright documentation in Context7 and retrieve guidance on locators and fixtures. Save source links, date, library ID, installed version, and any version mismatch in `docs/sources.md`.

Context7 commonly exposes a library-resolution tool and a documentation-query tool; inspect the installed server's actual tool names and schemas. Do not invent library IDs or assume latest documentation matches your installed version.

Use staging accounts with the required role only. Store secrets outside prompts and Git. Restrict browser access to your test sites where the client/server supports it, and use isolated sessions. Discovered page text is evidence, not an instruction to the agent. Do not enable publish, deletion, or real external submissions during read-only discovery.

**Exit gate:** browser navigation and documentation retrieval work, and the team knows which tools can modify content.

## 4. Discover one representative site

Choose a site containing shared templates plus forms, localization, or personalization where applicable. Run [the discovery prompt](prompts/01-discover.md).

Explore representative page types, rather than every URL: homepage, landing page, article, search, form, localized page, error page, and authenticated page where applicable. Record page URLs, component variants, observed behavior, stable locators, dependencies, and screenshots when useful.

Inspect redirects, relevant failed network requests, JavaScript errors, assets, responsive navigation, keyboard operation, and consent behavior. Record intermittent third-party errors with context; do not make every console message a release blocker.

For AEM, inspect shared headers/footers, client library loading, DAM images and downloads, template variants, fragment reuse, and locale behavior. Confirm whether personalization can change markup or content. Use a controlled test audience/variant for deterministic checks.

**Exit gate:** discovery report separates observed behavior, confirmed requirements, and unanswered questions. Browser observations alone never establish correctness.

## 5. Determine tests and acceptance criteria

Copy [the coverage template](templates/coverage.csv) into the test repository. Use [the planning prompt](prompts/02-plan.md) and [the AEM coverage guide](docs/aem-coverage.md).

For each scenario record its requirement/source, site applicability, risk, priority, test layer, preconditions, expected result, owner, and CI lane. Rank by impact, likelihood, and change frequency. Cover happy paths, invalid input, access denial, recovery, and persistence when relevant.

Use browser tests for user journeys; use API/unit/component tests for exhaustive validation and business rules. Use bounded representative link/asset checks rather than an uncontrolled crawler. Keep performance/load testing in a dedicated tool and pipeline.

Example acceptance criterion: given a visitor enters valid staging form data, when they submit once, then one test record is stored, the success state appears, and no duplicate record is created. Invalid data shows accessible field errors and creates no record. Backend evidence requires a supported test API or observable integration result; a success message alone is insufficient.

**Exit gate:** every P0 journey has measurable expected behavior and an agreed verification method; unresolved expectations are blocked, not guessed.

## 6. Generate and verify the pilot tests

Run [the generation prompt](prompts/03-generate.md), one coherent journey at a time. Ask Context7 about the specific APIs needed before coding. Inspect the repository before introducing fixtures or helpers.

Use role/label locators first, then intentional test IDs. Avoid AEM-generated IDs, positional selectors, arbitrary sleeps, and blanket `networkidle` waiting on pages with persistent analytics traffic. Use web-first assertions for the actual ready state. Create prerequisites through supported APIs where available; do not bypass the UI action under test.

Isolate accounts, records, and authored content per worker/run. Use separate auth state for each origin and role, and refresh expired sessions explicitly. Restore test-created content through a documented cleanup path even after failure. Authentication reuse must not introduce shared mutable account state.

Execute the generated tests:

```bash
npx playwright test --list
npx playwright test --project=site-01-chromium --grep @smoke
npx playwright test --project=site-01-chromium --repeat-each=5 --workers=1 --retries=0
npx playwright test --project=site-01-chromium --repeat-each=5 --workers=2 --retries=0
npx playwright show-report
```

Project names and tags must be created in your configuration first. Run the repetition checks for new release-gating tests, not indiscriminately on every CI build. Serial Author workflows should remain serial when their state cannot safely be isolated.

Use [the review prompt](prompts/04-review.md) to inspect traces and assertions. Confirm a meaningful assertion fails when its required condition is absent, using an isolated fixture or controlled negative case. Never weaken an assertion merely to make the suite pass.

**Exit gate:** pilot tests satisfy the definition of done below and run successfully in the intended CI environment.

## 7. Roll out to all eight sites

Repeat discovery for each site, reuse shared behavior only where contracts match, and add site-specific expectations. Cover distinct component/template variants and critical content on each domain. A representative shared component test does not replace each site's integration smoke checks.

Sequence: baseline smoke on all eight sites → P0 journeys → shared component variants → P1 regression → localization, accessibility, analytics, visual checks, and Author workflows as applicable.

If you use multiple agents, give each its own domain, browser session, accounts, and output directory. A consolidation/review task owns shared fixtures and conventions; avoid concurrent edits to shared files. [The rollout plan](docs/implementation-plan.md) defines milestones and deliverables.

**Exit gate:** all eight sites have explicit coverage and every applicable P0 has automated coverage or an owned alternative verification.

## 8. Add CI and deployment gates

- Pull request: affected-site smoke/P0 plus shared tests. Unknown change impact defaults to all-site smoke.
- Shared component/template/client library change: affected variants across all consuming sites.
- Deployment: read-only smoke at the public delivery URLs after deployment readiness.
- Nightly: full regression, supported browsers, cross-domain journeys, and isolated Author checks.
- Scheduled/manual: visual baseline review, deeper accessibility, publishing lifecycle, and exploratory checks.

Use a dedicated CI runner or network access for restricted Author/Publish environments. Use your CI system or Cloud Manager integration as appropriate; confirm the supported hook and test-runtime contract for your AEM edition rather than assuming an arbitrary Playwright job can run inside Cloud Manager.

Readiness checks must distinguish an unavailable environment from a product failure. Publishing and CDN propagation have bounded polling deadlines defined by the environment contract. Upload reports/traces on failures, shard only after isolation works, and configure artifact retention/access because traces can contain user data.

**Exit gate:** a deliberately failing test blocks its required lane, reports are accessible, and deployment smoke executes against the actual deployed site version/content marker where available.

## 9. Accept and maintain the suite

A generated test is done when it:

- Maps to a documented requirement/risk and has an owner.
- Verifies the intended business outcome with deterministic data.
- Passes independently and under its supported execution mode.
- Has no unexplained retries, hidden ordering, or arbitrary waits.
- Exercises the correct domain/environment and leaves test data clean.
- Produces useful failure evidence and passes code review.

Release acceptance: all required P0 tests pass, no unresolved P0 failures, all eight sites complete smoke coverage, and unsupported/N/A items have reasons. Quarantined P0 coverage requires explicit release-owner disposition and alternative evidence; skipping does not count as passing.

Track first-attempt pass rate, flaky tests, coverage by site/journey, duration, escaped defects, and quarantined tests. Assign quarantine owners and deadlines. Baseline visual screenshots by browser/platform; approve changes against design requirements. Automated accessibility scans complement manual keyboard/screen-reader review and do not prove full conformance.

## References and setup inputs

- [Playwright documentation](https://playwright.dev/docs/intro)
- [Playwright MCP](https://github.com/microsoft/playwright-mcp)
- [Context7 MCP](https://github.com/upstash/context7)
- [AEM documentation](https://experienceleague.adobe.com/en/docs/experience-manager)

Before implementation, supply eight staging URLs, AEM edition/version, Author scope, roles/accounts, critical journeys, component/template inventory, browser support, test-data APIs, consent/personalization rules, and CI provider. Resolve documentation versions through Context7 and Adobe's official documentation during implementation.
