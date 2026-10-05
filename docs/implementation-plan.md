# Implementation milestones

Estimate effort after discovery. A domain count alone does not capture journey count, AEM customizations, or test-data complexity.

| Milestone | Work | Deliverables | Acceptance |
| --- | --- | --- | --- |
| M1: Scope | Inventory all eight sites; establish environments, requirements, roles, dependencies | Completed inventory; risk register; P0 list | All sites owned; unknown requirements tracked |
| M2: Foundation | Install runner; connect MCPs; add projects, fixtures, auth, reporting, CI | Configuration; source log; baseline tests | Correct routing; local and CI execution; tool connections verified |
| M3: Pilot | Explore representative site; generate and review critical tests | Discovery report; coverage mapping; pilot specs | Outcome assertions; repeatable execution; cleanup works |
| M4: Eight-site smoke | Discover remaining sites; parameterize shared contracts | Eight reports; site capability mapping; smoke specs | Every domain covered with no accidental cross-site execution |
| M5: Critical coverage | Implement P0 journeys, permissions, integration outcomes | Reviewed P0 specs and alternative evidence for exceptions | Required P0 checks pass; no unresolved P0 defects |
| M6: AEM regression | Add applicable lifecycle, fragments, MSM, localization, cache and analytics checks | P1 suite; isolated content fixtures; broader browsers | AEM contracts covered and propagation deadlines explicit |
| M7: Operations | Add change-impact routing, dashboards, triage, ownership and handover | CI lanes; runbook; maintenance backlog | Deliberate failure gates release; reports available; owners can diagnose |

## Operating responsibilities

- Product/content owners define correct behavior, critical journeys, and editorial fixtures.
- AEM developers provide component contracts, supported setup APIs, hosting details, and publishing/cache rules.
- Test engineers maintain fixtures, runner configuration, reporting, and coverage.
- Agents explore and implement within the supplied scope, record sources, and execute verification.
- Reviewers check requirements and test evidence before accepting generated changes.
- Release owners resolve gating failures and documented exceptions.

## Failure triage

1. Identify project/site, deployed version, content state, first-attempt result, and trace.
2. Classify the failure: product defect, unavailable environment, changed fixture, integration issue, or test defect.
3. Reproduce using the same scenario and data. Inspect network and public-vs-Author timing where applicable.
4. Fix the cause. Do not use retries or weaker assertions as a substitute.
5. If quarantined, retain reporting, assign an owner/deadline, and record the lost coverage. Critical coverage needs explicit release disposition and alternative verification.

## Completion checklist

- [ ] All eight inventory rows complete and owners assigned.
- [ ] AEM hosting model and applicable features confirmed.
- [ ] Browser support, environments, credentials, data setup/cleanup established.
- [ ] Playwright MCP and Context7 verified in the implementation client.
- [ ] Version-matched documentation sources recorded.
- [ ] Approved coverage matrix links scenarios to requirements.
- [ ] Projects route shared and site-specific specs correctly.
- [ ] Every site has smoke; every applicable P0 has verification.
- [ ] New gating tests verified for independence and execution reliability.
- [ ] Author writes and lifecycle tests use isolated staging content.
- [ ] CI lanes, artifacts, readiness, and release gates validated.
- [ ] Accessibility/manual review and visual baseline processes defined.
- [ ] Maintenance owners and failure triage process established.
