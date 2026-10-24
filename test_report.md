# Thaura SQA Hiring Assessment Report

**Execution date:** 2026-09-16  
**Target:** https://thaura.ai/ and documented API at https://thaura.ai/api-platform  
**Scope:** Public website functional, data-correctness, basic security, performance reconnaissance, and API authentication boundary.  
**Environment:** Windows, VS Code integrated browser/Playwright, PowerShell HTTP checks.  
**Important limitation:** An authenticated Free-plan browser session became available during follow-up execution. Chat quota, Settings email validation, Memory UI, Incognito toggle, and upload accept-list checks were then executed. No API key or second account was supplied, so paid API calls, cross-account isolation, and server-side deletion remain blocked.

## Executive Summary

Seven issues or observations were recorded. The clearest user-facing defect is the Pricing page's Annual tab: it displays `$12/month`, `Billed $144/year`, and `Save 20%`; the first two values imply zero annual discount, not 20%. The public pricing page and authenticated quota modal also say Free users receive `5 messages every 5 hours`, which conflicts with the assessment requirement of `5 messages / 2 hours` and needs product-owner clarification.

The homepage at `/` exposes a canonical URL of `/home` without redirecting to it. Public route status checks returned 200 for all sampled primary and footer destinations. Browser evidence repeatedly showed an unauthenticated 401 request to `https://backend.thaura.ai/api/auth/me`, plus aborted Next.js route/data requests and slow or timed-out navigation under a 12-second budget.

Security headers were present on the sampled pages: CSP, X-Frame-Options: DENY, Strict-Transport-Security, and X-Content-Type-Options: nosniff. The CSP is permissive in places (`unsafe-inline`, broad `https:` sources, and localhost HTTP connections) and should be reviewed as a hardening observation, not treated as proof of exploitability.

## Findings

| ID | Severity | Area | Finding | Evidence |
|---|---|---|---|---|
| BUG-001 | Medium | Pricing/data correctness | Annual plan shows `$12/month`, `Billed $144/year`, and `Save 20%`. $12 x 12 = $144, so the shown annual price provides 0% savings. | Pricing DOM text captured at `/pricing`; mathematical check: `(144 - 144) / 144 = 0%`. |
| BUG-002 | High pending clarification | Product contract | Public pricing and authenticated quota modal state `5 messages every 5 hours`, while the assessment contract says `5 messages / 2 hours`. This makes quota expectations and test acceptance criteria inconsistent. | Pricing DOM text and authenticated modal at `/pricing`/chat UI. |
| BUG-006 | Medium | Quota enforcement | The sixth message is blocked after five successful messages, as expected for a five-message limit. The product's actual five-hour window differs from the requested two-hour acceptance criterion. | Authenticated session: messages 1-5 returned completion HTTP 200; message 6 made no completion request. Rate-limit response: `allowed:false`, `remaining:0`, `resetAt:2026-09-16T22:02:01.162Z`; modal said `Free messages reset in: 4h 59m`. |
| BUG-003 | Low | SEO/routing | `https://thaura.ai/` returned 200 but exposed canonical `https://thaura.ai/home`; the root did not redirect to `/home`. This creates two indexable entry URLs unless intentionally configured. | Browser DOM metadata on `/`; HEAD status checks for both `/` and `/home`. |
| BUG-004 | Medium | Frontend reliability | Public pages repeatedly logged `Failed to load resource: the server responded with a status of 401 ()`, corresponding to unauthenticated backend auth/session checks. A normal unauthenticated public page should either suppress expected 401s or handle them without console errors. | Browser console screenshot on `https://thaura.ai/home` shows the 401 error for `backend.thaura.ai/api/auth/me`; same issue was also observed in Playwright console events. |
| BUG-005 | Medium | Performance/reliability | Navigation was intermittently slow: `/story` and `/api-platform` exceeded a 12-second `domcontentloaded` budget during one sequential crawl; other pages took approximately 0.7-10.4 seconds in the same run. The initial homepage navigation also exceeded 30 seconds before becoming readable. | Playwright navigation timing/error records in execution notes. |
| BUG-007 | Medium | Lighthouse/performance | Desktop Lighthouse identified console errors and avoidable script/rendering cost. | Attached report captured 2026-09-16: console errors, 173 KiB unused JavaScript, 120 ms render-blocking savings, 12 KiB legacy JavaScript savings, and a network dependency-chain insight. |
| BUG-008 | Low | Contact form/input validation | Contact Subject has no client-side maximum length and accepted a 10,000-character value. | Chromium browser on `/contact`; field remained valid with length 10,000. No submission was made. |
| BUG-009 | Medium | Upload/data integrity | Known-content spreadsheet and image markers were extracted exactly; known PDF amounts/title were reproduced. Empty, corrupted, and unsupported files were rejected with explicit UI errors. Cross-account leakage and password-protected/oversized files remain unverified. | Authenticated second Free-plan session: CSV upload 200 and exact marker/value/amount response; SVG exact text response; PDF response reproduced invoice title and totals; empty PDF rejected locally; corrupted PDF returned 400; `.exe` rejected locally. |
| BUG-010 | Medium | Memory/Incognito | Normal memory persisted into a separate chat, but Incognito runtime persistence/leakage could not be fully verified because the account reached its message allowance before an Incognito message could be sent. | Normal chat stored `cobalt blue`; Memory panel displayed it; unrelated normal chat returned exactly `cobalt blue`; Incognito toggle displayed `You're incognito`, while existing normal history and the normal Memory entry remained visible. |
| BUG-011 | Low | Settings/input validation | Settings Email has no client-side maximum length and accepted a 10,012-character syntactically valid address. | Settings field had no `maxlength`; 10,000 `a` characters plus `@example.com` remained valid. Value was restored and not saved. |
| OBS-001 | Low/security hardening | CSP | CSP contains `script-src 'unsafe-inline'`, broad `https:` script/style/font sources, and `connect-src ... https: ... http://127.0.0.1:* http://localhost:*`. This increases attack surface and weakens policy precision. | HEAD response headers from `/`, `/home`, `/pricing`, `/api-platform`, `/faq`, `/contact`. |

## Public Website Test Matrix

| Test | Result | Notes |
|---|---|---|
| Primary and footer internal route status | Pass for sampled routes | `/`, `/home`, `/story`, `/constitution`, `/api-platform`, `/download`, `/pricing`, `/faq`, `/contact`, `/community`, `/careers`, `/donate`, `/terms-of-service`, `/privacy-policy`, `/data-processing-addendum`, `/imprint` returned 200 to HEAD checks. |
| HTTPS enforcement | Pass for tested HTTPS URLs | All 16 sampled public routes returned 200 over HTTPS. No literal `http://` HTML `src`/`href` references were found; no mixed-content warning was observed in browser/Lighthouse evidence. |
| Blank contact submission | Pass, client-side validation | Empty required Name, Email, and Message fields were blocked by native required validation; no request was sent. Subject is optional. |
| Contact email-format validation | Partial pass | Public contact field is type `email`; authenticated Settings email field rejected `invalid-email` with native message `Please include an '@'...`. No profile save was performed. |
| Contact successful delivery | Blocked | No message was submitted to avoid sending unsolicited data; delivery cannot be verified from the client alone. |
| Special-character/reflection input | Partial pass/finding | Locally filled name, message, and 10,000-character subject with script-like, Unicode, RTL, and long-string values. Invalid email was rejected; script-like and Unicode/RTL values were not reflected into visible page text. Subject had no client-side length limit. No message was submitted, so server-side sanitization, status code, receipt, and persistence remain unverified. |
| Canonical/meta/OG/Twitter | Partial | Homepage canonical is `/home`; OG/Twitter tags were present and non-duplicated in the observed DOM. Repeat across every route and compare `og:url` to final canonical URL. |
| Image loading | Partial | Images include SVG/GIF assets; observed images did not demonstrate WebP/AVIF usage. Below-fold lazy loading was present on some images. Video optimization was not verified because no video resource was observed in the sampled DOM. |
| Console/network errors | Fail/observation | Repeated 401 console errors and aborted Next.js data/HEAD requests were observed. |
| Security headers | Pass with hardening note | CSP, X-Frame-Options DENY, HSTS, nosniff, and `Permissions-Policy: microphone=(self), camera=(), geolocation=()` were present. CSP policy is broad. |
| Cookies | Pass for public responses, blocked for authenticated cookies | No `Set-Cookie` header was returned by 16 sampled public GET responses. Authenticated cookie attributes were not inspected because session tokens are sensitive. |
| Sensitive data exposure | Pass for sampled HTML, partial overall | Across 16 route documents, no API-key/private-key pattern was found. The backend hostname is visible in CSP and browser network calls; this is an endpoint disclosure, not a credential. Full JavaScript-bundle and authenticated-response review remains out of scope. |
| Cross-browser/device compatibility | Blocked | Only the integrated Chromium browser was available in this execution. |
| Lighthouse/Core Web Vitals | Pass with findings | Attached desktop Lighthouse report captured 2026-09-16: Performance 98, Accessibility 100, Best Practices 96, SEO 100; FCP 0.3 s, LCP 1.0 s, CLS 0, TBT 50 ms, Speed Index 1.2 s, 90 requests, total size 972 KiB. Lighthouse also reported console errors, 173 KiB unused JavaScript, 120 ms render-blocking savings, 12 KiB legacy JavaScript savings, and a network dependency-chain insight. |
| JMeter/sustained load | Not run | Non-intrusive single-user checks only; no load test was run against a production site without authorization. |

## API Contract Test Matrix

| Case | Result | Expected/observed |
|---|---|---|
| Missing Authorization header | Executed | POST to `https://backend.thaura.ai/v1/chat/completions` with one-token payload returned HTTP 401. No billed completion occurred. |
| Replacement-key authenticated probes | Blocked | Five minimal requests using the locally configured environment variable all returned HTTP 401, including a valid one-token payload, missing `messages`, `temperature:2.1`, `max_completion_tokens:32001`, and both token fields. A direct user-shell probe confirmed the endpoint response: `{"error":{"type":"authentication_error","code":"unauthorized"},"message":"Authentication required"}`. No successful completion or usage charge was observed. The key is invalid, revoked, or not present in the shell running the request. |
| Malformed/invalid API key | Blocked | Requires a key-shaped test value and owner approval. |
| Zero-balance key / 402 | Blocked | Requires a real key and billing state. Do not fabricate or repeatedly probe. |
| Required/optional parameter validation | Blocked | Requires authenticated API access. |
| Temperature boundaries | Blocked | Requires authenticated API access. |
| `max_tokens` vs `max_completion_tokens` precedence | Documentation observed | API page states `max_completion_tokens` wins; runtime confirmation blocked. |
| 32,000 token cap | Documentation observed | API page states both token parameters are capped at 32,000; runtime confirmation blocked. |
| Streaming/non-streaming schema and usage accounting | Blocked | Requires authenticated API access; use `max_completion_tokens: 1`. |
| 60 requests/minute and 8 concurrency | Not run | Production load/rate-limit testing requires explicit authorization and a test key. |
| Functions/function_call accepted/ignored/rejected | Documentation/runtime blocked | Runtime confirmation requires authenticated access. |

The API page documents: Bearer authentication, model `thaura`, streaming, temperature 0.0-2.0, attachments, `max_tokens`, `max_completion_tokens`, and prepaid billing of $0.50 per million input tokens and $2.00 per million output tokens. The unauthenticated backend probe returned 401 with standard JSON/API security headers.

## Authenticated Follow-up Results

The shared browser session was authenticated as a Free Plan user. The account identifier and password were intentionally not written to this report.

| Test | Result | Evidence |
|---|---|---|
| Login/session presence | Pass | UI displayed the signed-in profile and `Free Plan`; authenticated backend calls returned successful statuses. |
| Messages 1-5 | Pass | Five minimal prompts returned HTTP 200 from `/v1/chat/completions` and rendered `OK`. |
| Message 6 | Pass, limit enforced | `/api/chats/rate-limit/check` returned 200 with `allowed:false`, `remaining:0`; no completion request was issued. UI displayed `Out of messages`. |
| Quota reset duration | Pass against product UI, contract mismatch | Modal displayed `Free messages reset in: 4h 59m`; endpoint returned a concrete `resetAt`. This confirms a five-hour reset, not the requested two-hour window. |
| Quota rolling versus fixed | Blocked | A five-hour wait or a second controlled account/time-series run is required. Current evidence only establishes the next reset timestamp. |
| Failed response quota consumption | Blocked | No safe, deterministic failure case was available after quota exhaustion; do not intentionally disrupt the service. |
| Refresh/multiple-tab quota bypass | Partial | The sixth send was blocked at the server-backed rate-limit check in the current session. A second browser context and direct authenticated replay were not run. |
| Settings invalid email | Pass | `invalid-email` was rejected client-side with native HTML email validation; value was restored and not saved. |
| Memory UI | Pass/observation | Memory panel opened and displayed `Nothing remembered yet`. Persistence of a newly learned fact requires available quota and a later unrelated chat. |
| Incognito toggle | Pass/observation | New Chat exposed `Incognito mode`; toggling changed the control to `Exit incognito` and heading to `You're incognito`. No message was sent because quota was exhausted. |
| Upload accept list | Pass | File input accepted PDF, office/spreadsheet, image, audio, code, and text extensions; the primary input allowed multiple files. |
| Spreadsheet extraction integrity | Pass | Synthetic CSV upload returned HTTP 200 and the assistant reproduced `ISOLATION_MARKER_20260916`, `SpreadsheetValue-42`, and `12345.67` exactly. |
| PDF extraction integrity | Partial pass | `Color Bill.pdf` was processed and the assistant reproduced the invoice title and monetary values including Grand Total `2,850.00`; exact source-text comparison and page count could not be independently established locally. |
| Image extraction integrity | Pass | Synthetic SVG upload returned both exact visible strings: `ISOLATION_MARKER_20260916` and `ImageValue-99`. |
| Empty/corrupted/unsupported uploads | Pass | Empty PDF showed `Cannot upload empty file`; corrupted PDF returned HTTP 400 with `Could not extract text from PDF "corrupted.pdf"`; `.exe` was rejected with `File type .exe is not allowed.` No silent failure observed. |
| Password-protected/oversized uploads | Blocked | No password-protected fixture or authorized oversized upload was available. |
| Upload data isolation | Blocked/partial | The second authenticated account had a separate empty chat history before testing and its own uploaded marker chats. A second independent account/session was not available to attempt cross-account retrieval. Full server-side deletion cannot be verified from the client. |
| Normal memory persistence | Pass | Stored synthetic preference `my QA test color is cobalt blue`; Memory panel displayed it; a new unrelated normal chat returned exactly `cobalt blue`. |
| Incognito mode visibility | Partial pass | New Chat changed to `You're incognito`/`Exit incognito`. Existing normal chats remained visible in the sidebar and the normal Memory preference remained visible, which is expected for account-level memory/history UI. |
| Incognito chat non-persistence and memory leakage | Blocked | No message was sent in Incognito because the Free account had exhausted its five-message allowance. A fresh quota window and a unique Incognito-only marker are required. |
| Server-side deletion claims | Not client-verifiable | Client observation cannot prove database deletion, backups, vector-index removal, retention expiry, or deletion from provider logs. |
| Negative/boundary: blank Settings fields | Pass | Required Name and Email fields failed native validation with `Please fill out this field.`; no save was performed. |
| Negative/boundary: Settings invalid email | Pass | `invalid-email` failed native email validation; original value was restored. |
| Negative/boundary: Settings length | Finding | Name input enforced `maxlength=80`; Email had no maxlength and accepted 10,012 characters. |
| Negative/boundary: Billing/API inputs | Observation | Billing showed current balance `$0.00`, preset Add Balance buttons from `$10.00` to `$250.00`, and Add funds & save card; no text/input fields were exposed. No payment action was clicked. |
| Negative/boundary: password formats | Blocked | Login/password form was not opened to avoid logging out the authenticated test session. |

### Authenticated evidence details

The successful chat flow observed: `GET /api/chats/rate-limit/check` 200, `POST /api/chats` 201 for the first chat, `POST /v1/chat/completions` 200 for messages 1-5, and `POST /api/title/generate` 200. The sixth attempt observed only the rate-limit check and no completion request. The UI displayed approximately `~30.4g CO₂ saved` after message 5.

The quota modal explicitly stated: `free accounts get 5 messages every 5 hours` and `Free messages reset in: 4h 59m`. This is direct authenticated evidence of the five-hour behavior and should replace the earlier blocked-quota wording.

## Blocked Authenticated Test Plan

Use a dedicated test mailbox and a newly created Free account. Record the account identifier and creation time in the final version of this report; never place the password or API secret in the bug report. Capture screenshots and sanitized request/response evidence.

1. Register and log in; record session cookie/token names and expiry. Logout, replay the last authenticated request, and verify rejection. Test two browser contexts for concurrent-session behavior.
2. Send five minimal messages and record the exact response/status at messages 1-6. Compare the observed window with both the public 5-hour statement and the assessment's 2-hour requirement. Test reset after the documented window, not by changing system time.
3. Force a client-visible failed response if the UI supports a safe invalid request. Compare quota before/after; do not induce service disruption.
4. Upload small known-good PDF, XLSX/CSV, and PNG fixtures containing unique markers. Verify extracted values against the fixture. Then test empty, corrupted, password-protected, unsupported, and size-boundary files.
5. Use a second account/context to verify that unique uploaded markers and memories cannot be retrieved across accounts. Test normal memory in a new chat and Incognito history/memory behavior. Mark server-side deletion as unverifiable from the client alone.
6. For API runtime tests, use a one-token prompt, `max_completion_tokens: 1`, low request count, and stop immediately on unexpected billing or rate-limit behavior.

## Reproduction Details

### BUG-001
1. Open `https://thaura.ai/pricing`.
2. Select the `Annual` tab.
3. Observe `$12/month`, `Billed $144/year`, and `Save 20%`.
4. Calculate monthly equivalent: `$144 / 12 = $12`; actual displayed saving is 0%.

### BUG-002
1. Open `https://thaura.ai/pricing`.
2. Read the Free plan feature list.
3. Observe `5 messages every 5 hours`.
4. Compare with the assessment acceptance criterion `5 messages / 2 hours`; resolve the source-of-truth discrepancy before sign-off.

### BUG-003
1. Open `https://thaura.ai/`.
2. Inspect `link[rel=canonical]` in the DOM/view-source.
3. Observe `https://thaura.ai/home` while the response URL remains `/` and status is 200.
4. Decide whether `/` should redirect or canonical should remain `/`.

### BUG-004
1. Open any public page in a clean browser context.
2. Monitor console and network requests.
3. Observe an unauthenticated request to `backend.thaura.ai/api/auth/me` returning 401 and surfaced as a console error.

## Reflection Questions

### 1. How will AI impact software testing in the next 2-3 years?

AI will reduce the cost of creating and maintaining test coverage, especially for exploratory scenarios, API contract generation, log clustering, and data variation. The strongest change will be faster feedback: a tester can ask for a focused test matrix from a requirement, execute it, and get a first-pass explanation of failures in minutes. It will not remove the need for testers because the difficult parts are still product intent, risk judgment, observability, and deciding whether a result is meaningful. AI can confidently generate a plausible but incorrect assertion, so evidence and human review become more important, not less.

### 2. Do I use AI, and how does it help testing?

Yes. I use AI to turn requirements into boundary-value and negative-test candidates, draft API payload matrices, summarize console/network evidence, identify inconsistencies between UI copy and behavior, and produce a first version of a bug report. I still reproduce the behavior myself and keep the raw request, response, timestamp, environment, and exact expected/actual result. AI is most useful as a multiplier for breadth and triage, not as the source of truth.

### 3. Which AI tools do I use?

I use GitHub Copilot for test design, coding small automation helpers, debugging, and documentation; Playwright/browser automation for repeatable UI and network checks; and general-purpose language models for requirement analysis and review. The tools are useful in different roles: one generates possibilities, while the browser and HTTP client provide evidence.

### 4. Which is most helpful for testing and why?

GitHub Copilot is the most helpful for day-to-day testing because it sits next to the test code and the product context. I can turn a requirement into a Playwright or API test, ask it to add boundary cases, and have it explain a failure against the actual code and logs. Its value is highest when I provide precise evidence and constrain the task; vague prompts produce generic tests, while a concrete failing request or trace produces useful analysis.

## Evidence and Reproducibility

- Public route status and header checks were run on 2026-09-16 from Windows PowerShell.
- Browser evidence was captured from Playwright page ID `90a97044-ac74-4d11-9b35-3e4bf27e7616`.
- No password, API key, or personal mailbox credentials were created or stored.
- `bug_report.csv` is included in this workspace and opens directly in Excel.
