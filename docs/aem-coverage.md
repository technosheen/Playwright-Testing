# AEM coverage and acceptance guide

Apply each row only when the feature exists. Priority is a starting point; business risk determines the final priority. Record a requirement source and environment-specific thresholds before implementation.

| Area | Determine tests from | Expected acceptance | Layer / lane |
| --- | --- | --- | --- |
| Public smoke | Critical URLs and site identity | Expected response, final URL, main content, brand, and primary action work on each domain | Browser / PR and deployment |
| Navigation | Header/footer contracts and responsive breakpoints | Desktop/mobile navigation reaches correct destinations; menu works with keyboard and focus | Browser / smoke and regression |
| Templates and components | Core/custom component inventory and variants | Representative variants render and behave according to their contracts; no missing critical client libraries | Browser/component / affected PR |
| DAM assets | Representative images, renditions, downloads | Required assets load and image has natural dimensions; downloads have expected file type and content; decorative images follow accessibility rules | Browser/API / regression |
| Forms | Business validation and backend contract | Valid submission creates one record; invalid input creates none; errors are associated with fields; controlled failures recover | Browser/API / critical |
| Search | Indexed fixture content and indexing SLA | Known fixture appears, filtering works, zero-result state works; index readiness uses bounded polling | Browser/API / critical or nightly |
| Localization | Locale rules and content expectations | Language switch reaches expected page/fallback; lang, canonical, hreflang and locale navigation match requirements | Browser/API / regression |
| MSM/live copies | Blueprint inheritance and rollout rules | In isolated content, rollout updates inherited fields; cancelled inheritance preserves local override | Author/API + Publish / scheduled |
| Experience Fragments | Approved consumers and variants | Expected fragment variant appears on consuming pages after controlled update/publication | Browser + lifecycle / regression |
| Content Fragments/headless | Schema and supported endpoint contract | Required fields and references are returned; unauthorized data absent; UI renders known fixture | API + browser / regression |
| Consent and analytics | Consent policy and event specifications | Required event/payload occurs once for defined action; nonessential calls obey consent policy; reject/accept states persist as specified | Network assertions + browser / regression |
| Personalization | Target audiences/variants and default fallback | Controlled audience receives expected variant; fallback renders; avoid unstable marketing copy assertions | Browser/network / regression |
| Authentication and SSO | Origin boundaries and identity contract | Allowed roles succeed; denied users cannot retrieve protected payload; cross-domain login/logout follows documented policy | API + browser / critical |
| Redirects and SEO | Migration map and SEO rules | Expected status/target without loops; canonical and robots metadata match environment policy; sitemap samples resolve | API + browser / regression |
| Dispatcher/CDN | Host routing and cache contract | Correct brand/content for each hostname; agreed headers; personalized/private responses follow cache policy; updates appear within agreed propagation deadline | HTTP + browser / deployment and scheduled |
| Error pages | Known invalid URL and controlled service failure | Correct HTTP status and usable branded recovery; no sensitive implementation details | API + browser / regression |
| Author permissions | Role matrix and supported authoring UI | Required create/edit/preview actions work; forbidden actions denied through supported endpoints as well as UI | Isolated Author / nightly |
| Publishing lifecycle | Publish/unpublish workflow and propagation SLA | Unique content marker reaches public URL within bounded deadline; removal behavior meets policy; cleanup verified | Author/API + public browser / scheduled |
| Accessibility | Agreed WCAG target and user journeys | Automated violations meet policy; manual keyboard, focus, and screen-reader checks cover critical flows | axe + manual / PR and release |
| Visual | Approved designs and stable fixture pages | Baseline differences reviewed on pinned browser/platform; only approved dynamic regions masked | Screenshot / scheduled |

## Publishing lifecycle procedure

1. Use an isolated staging subtree and a run-specific page/content marker. Avoid editing shared baseline pages.
2. Create/update content through the supported API or authoring workflow for your AEM edition. Do not assume a generic JCR write endpoint is supported or permitted.
3. Verify Author preview where required; publish through the configured workflow with a role permitted to do so.
4. Poll the actual public URL for the unique marker until the agreed deadline. A successful publish request alone does not prove public delivery.
5. Check representative consumers if reusable content is involved. Record timings and relevant cache headers for diagnosis.
6. Unpublish/delete test content through the supported workflow and verify the required public removal state. Ensure teardown runs after failure, with a cleanup backlog for unsuccessful teardown.

Never disable caching or use a cache-busting query as the sole proof of visitor-visible propagation. Compare direct Publish only as a diagnostic when access is available. Do not flush caches globally as routine test setup.

## Test-data rules

Prefer versioned fixtures and supported test APIs. Each worker needs isolated records/accounts for mutable flows. Keep read-only production checks separate from staging writes. External email, CRM, commerce, and payment services need sandbox endpoints or controlled adapters. Use a small separately identified integration suite to validate real sandbox dependencies; mocks do not prove integration correctness.

Use stable baseline content with ownership and an update policy. Avoid hardcoding editorial copy that changes independently unless the copy is itself a requirement. For asynchronous publishing/indexing, document the maximum allowed delay and test it with bounded retries of the observation, not arbitrary sleep.
