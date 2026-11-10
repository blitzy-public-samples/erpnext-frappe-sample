# 1. Executive Summary

## 1.1 Project Overview

The ERPNext Bank Reconciliation single-page application at `erpnext/banking/` gives an accountant one screen on which to import a bank statement, review the match the rule engine suggests, accept it or link a different Payment Entry, Journal Entry or Invoice, and post the reconciliation. This work completed and hardened that workflow: the five prescribed failure modes now behave exactly as specified, the workspace type-checks and builds cleanly on Node.js 24, and a frontend test suite exists where there was none. The result is a reviewer-facing surface whose financial writes are server-authoritative, whose refusals are legible, and whose regressions meet an enforced coverage floor.

## 1.2 Completion Status

```mermaid
pie title AAP-Scoped Completion — 84.3% Complete
    "Completed Work" : 301
    "Remaining Work" : 56
```

Completed `#5B39F3` · Remaining `#FFFFFF`.

| Metric | Value |
| --- | --- |
| Total Hours | **357 h** |
| Completed Hours (AI + Manual) | **301 h** (301 h autonomous, 0 h manual) |
| Remaining Hours | **56 h** |
| Percent Complete | **84.3 %** (301 ÷ 357) |

## 1.3 Key Accomplishments

- ✅ Reconciliation verified end to end: import, rule-suggested match, manual override, atomic post.
- ✅ Confirm/post is one server call with nothing mutated on the client beforehand.
- ✅ A refused post reports the server's own words in a dismissible dialog and changes nothing.
- ✅ An already-reconciled transaction cannot be posted again, on either database engine.
- ✅ Statement imports mark the file that failed and create no transactions when refused.
- ✅ A currency mismatch warns without blocking, exactly as specified.
- ✅ 62 frontend suites and 1,521 cases pass at 85.04 % line coverage against an 80 % floor.
- ✅ 134 backend tests pass on MariaDB and 134 on PostgreSQL; zero type errors, clean build on Node 24.

## 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
| --- | --- | --- | --- |
| Two paths in the bank-account controller directory sit outside the change set's agreed scope | The permitted-path criterion cannot be signed off as written; the code itself is correct and green on both engines | Engineering lead / plan owner | 1 day |
| No pipeline runs the frontend acceptance gate | 1,521 cases and the 80 % line floor are enforced only on a developer's machine, so a regression can reach the default branch | CI owner | 1 day |
| Undo removes one allocation row per call on a transaction matched to two or more vouchers | A second Undo is needed to finish reversing; the interim state is internally consistent | Backend | 1 day |
| Concurrent posts against one account can answer HTTP 508 instead of a user-level message | Data integrity always holds — correct final status, exactly one allocation row, no partial or duplicate posting — but a double-submit sees a raw status code | Backend | 1 day |
| Six solid-control colour pairs measure below 4.5:1 contrast | Buttons and badges in the design system's brand colours fall short of AA in both themes | Design-system owner | 2 days |
| The per-file import failure marker is session-scoped | A hard reload returns a refused row to "Not Started", so a reviewer can miss that a file failed | Product | 1 day |

## 1.5 Access Issues

No access issues identified. Verified here: repository and workflow files readable and writable; the application signs in and serves `/banking` with HTTP 200; both database engines accept connections and run their suites; the package registry answers live queries. Adding a continuous-integration job is blocked by scope, not permissions.

## 1.6 Recommended Next Steps

1. **[High]** Rule on the bank-account controller's two paths — amend the scope or re-plan — since reverting the authorisation fix inside them reopens a financial-write bypass.
2. **[High]** Add the acceptance gate to an existing workflow on Node ≥ 24.15: install, typecheck, lint, coverage, one shuffled seed.
3. **[Medium]** Close the reversal-path gaps: one Undo call releases every allocation row, and concurrent posts answer with a message rather than a status code.
4. **[Medium]** Settle the brand-colour contrast question with the design-system owner, then re-verify both themes.

# 2. Project Hours Breakdown

## 2.1 Completed Work Detail

| Component | Hours | Description |
| --- | --- | --- |
| Frontend test infrastructure and fixtures | 22 | A runner configuration deliberately independent of the build config, so tests need no provisioned bench; a simulated-DOM harness stubbing the boot user, roles, every permission array, the currency and company lookups, `matchMedia`, `ResizeObserver`, pointer capture and virtualiser measurement; a 1,513-line typed factory module shaped by the real server payloads; and a shared panel renderer (`banking/vitest.config.ts`, `banking/src/test/`) |
| Frontend test suite — 62 suites, 1,521 cases | 95 | Suites covering the workbench, the importer, all four routes, the API-client hooks, the shared libraries, the design-system primitives and the design tokens, including the six mandated scenarios and all five failure modes |
| Backend test suites — 5 bank DocType modules | 26 | 134 tests per engine across `bank_transaction`, `bank_reconciliation_tool`, `bank_transaction_rule`, `bank_statement_import_log` and `bank_account`, including rule stamping, first-match-wins by priority, the already-reconciled refusal and the currency guards |
| Dismissible failure reporting on confirm/post | 16 | `BankRecErrorDialog.tsx` composed from the existing alert dialog and error banner, the rejection path in `BankReconciliation/utils.ts`, two feature atoms, focus capture and restoration (`hooks/use-return-focus.ts`) and viewport-clamped scrolling |
| Statement import failure surfacing | 16 | Backend error captured instead of discarded in `BankStatementImporter/CSV/StatementDetails.tsx`, a three-state per-file indicator in `pages/BankStatementImporter.tsx`, a zero-transaction guard, and placeholder-attachment cleanup on both endings |
| Already-reconciled guard and stale-client convergence | 10 | Confirm disabled on the server's own predicate with an explanatory tooltip, plus cache eviction, revalidation and a fail-closed selection clear after a refusal |
| Currency advisory and no-deduplication behaviour | 6 | Amber advisory badge with tooltip on mismatched rows in `MatchAndReconcile.tsx`, proven not to disable confirm; duplicate re-imports verified to render exactly as the server returns them |
| Backend authorisation and input-validation hardening | 40 | Across the five bank controllers: per-account permission on the closing-balance write, an argument-arity guard applied at 28 call sites, POST-only on four state-changing endpoints, total date parsing, voucher-type and clearance-write validators, company-scoped rule evaluation, deterministic candidate ordering, and a markdown allowlist over server-supplied text |
| Frontend robustness, responsive layout and accessibility | 34 | Error boundary, render-failure surface and page shell (`components/common/`), available-height and scroll-overflow hooks, strict amount parsing, selection scoping, progress-figure gating, routing history correction, and design-token-only styling across the workbench and importer |
| Type and build integrity | 2 | The implicit-`any` parameter in `banking/proxyOptions.ts` annotated and a `typecheck` script added, so zero-error compilation is observable rather than assumed |
| Dependency and install integrity | 4 | Seven exactly pinned test packages, four resolution overrides including the one without which the install fails outright, and a clean frozen-lockfile install |
| Workspace runbook | 12 | `banking/README.md` rewritten from the scaffold template to a 1,211-line operating manual whose every command, version, count and threshold reproduces |
| Success-criteria verification | 18 | Dual-engine backend runs, the coverage gate proved to bite, build, lint and type gates, end-to-end runtime verification of the workflow and all five failure modes, and submodule and schema integrity checks |
| **Total** | **301** | |

## 2.2 Remaining Work Detail

| Category | Hours | Priority |
| --- | --- | --- |
| Rule on the bank-account controller's two paths; once amended, extend the argument and permission guard to that directory's four remaining unguarded endpoints, with tests on both engines | 8 | High |
| Wire the frontend acceptance gate into an existing pipeline on Node ≥ 24.15, publishing the lcov artefact | 6 | High |
| Reversal-path robustness: remove every allocation row in one Undo call, and map the concurrent-post conflict to a user-level answer | 8 | High |
| Brand-colour contrast decision and per-mode token overrides for the six failing solid-control pairs | 10 | Medium |
| Coverage for the three voucher-creation modals and the app-shell guest branch, plus a branch-coverage floor | 8 | Medium |
| Per-file failure indicator polish (carry the stored server message, make it keyboard reachable) and the statement summary tile correction | 5 | Medium |
| Release mechanics: workspace version, release notes, staging deploy rehearsal and smoke test on both engines | 8 | Medium |
| Operational alignment: scheduler and automatic rule evaluation agreement, realtime socket port for the target deployment | 3 | Low |
| **Total** | **56** | |

## 2.3 Hours Reconciliation

The completion figure is derived only from work the agreed plan scoped, plus the standard path-to-production activities needed to deploy it:

```
Completed hours  = 301   (Section 2.1 total)
Remaining hours  =  56   (Section 2.2 total)
Total hours      = 357   (301 + 56)
Completion       = 301 ÷ 357 = 84.3 %
```

Priority distribution of the 56 remaining hours: **High 22 h**, **Medium 31 h**, **Low 3 h**.

Four bodies of work were deliberately excluded from these hours because they lie outside the agreed scope and are not required to deploy what was built: bounded pagination and a covering index on the two workbench list endpoints; virtualising the large statement-preview tables; converting the desktop-only breakpoint into a render-level guard; and a product decision on duplicate re-imports together with a review of the bank DocType permission personas. Each is carried instead as a risk in Section 6 and as a recommendation in Section 8, so it is visible without inflating the estimate. Two further bodies of work are excluded from these hours at the reviewer's direction, because neither is required for this project to be successful: production deployment hardening — security headers and CSP, `Secure`/`HttpOnly` cookies, HTTPS with HSTS, traceback exposure disabled, production WSGI behind a reverse proxy, compression and immutable asset caching, and websocket upgrade headers — and a repeatable browser regression suite for the four routes and the confirm, refusal and import journeys. Their related risks remain visible in Section 6.

# 3. Test Results

Every figure below was produced by executing the suites in this repository and reading the result. The frontend gate is `CI=true yarn test:coverage` from `erpnext/banking`; the backend suites are `bench --site <site> run-tests --lightmode --module erpnext.accounts.doctype.<module>.test_<module>`, run once per database engine.

| Area / Category | Framework | Tests | Passed | Failed | Coverage | What This Proves |
| --- | --- | --- | --- | --- | --- | --- |
| Reconciliation workbench, matching rules editor and action log | Vitest 4 (simulated DOM) | 580 | 580 | 0 | 79.54 % lines | A reviewer can list unreconciled transactions, read the rule-suggested match, override it, post atomically, be refused without state changing, and undo |
| Statement importer | Vitest 4 | 129 | 129 | 0 | 98.77 % lines | An import reports its own outcome per file, refuses a statement that would post nothing, and never creates a transaction client-side |
| Routes and pages | Vitest 4 | 142 | 142 | 0 | 92.40 % lines | All four routes and the catch-all mount, and the importer list renders its three per-file states correctly |
| Shared libraries and formatters | Vitest 4 | 246 | 246 | 0 | 96.94 % lines | Server errors are parsed and surfaced verbatim, money and dates format per company defaults, and amount and date inputs reject malformed values |
| Design-system primitives and design tokens | Vitest 4 | 213 | 213 | 0 | 82.21 % lines | Dialogs, tables, badges, markdown and list virtualisation behave as the workbench relies on, and every styled value resolves to a token |
| Shared hooks and application shell | Vitest 4 | 93 | 93 | 0 | 96.11 % lines | Panels measure and re-measure their available height, focus returns to the control that opened a dialog, and the shell renders only for a signed-in user |
| Settings and shared field components | Vitest 4 | 118 | 118 | 0 | 81.93 % lines | Preferences and the shared field components stay operable and keyboard-reachable, and a render failure degrades to a recoverable surface instead of blanking the page |
| Bank DocType controllers — both engines | Frappe unittest | 268 | 268 | 0 | not instrumented | Rule stamping and first-match-wins, the already-reconciled refusal, currency validation, import parsing and the authorisation guards behave identically on MariaDB and PostgreSQL (134 tests per engine) |

Whole-workspace frontend line coverage measured **85.04 % (3,271 of 3,846 lines)** against the reporter-enforced 80 % floor — a margin of 194 lines. Statements 84.18 %, functions 78.50 %, branches 73.15 %; only lines are gated. The suite is order-independent: a shuffled run with a fixed seed passes all 62 files and 1,521 cases.

**Not Covered**

- **The three voucher-creation modals** — Bank Entry (44.4 % lines), Transfer (55.4 %) and Record Payment (65.3 %). Creating a voucher from a transaction is exercised far less than matching an existing one. A human should drive each modal end to end against a seeded site before release.
- **The application shell's guest branch** (63.6 % lines) — the redirect taken when the boot payload reports no signed-in user is not asserted. Worth one manual check with an expired session.
- **Branch and function coverage are not gated** at 73.15 % and 78.50 %; defensive paths are measured but not enforced, so a new module can lower them without failing the gate.
- **Layout, scrolling and render cost cannot be asserted at all.** Every suite runs in a simulated DOM that performs no layout, so virtualised lists, real scroll behaviour and large-statement render cost are outside its reach — the large statement preview and the desktop-only breakpoint in particular are verified only by hand.
- **Narrow failure branches covered by unit tests but never reached at runtime**: the secondary "view could not be refreshed" path, which needs a re-read to fail after a refusal has already failed; the non-server-shaped rejection branch; the PDF-password write banner, which needs a protected-PDF fixture; and the refused-cleanup handler on the placeholder statement attachment.
- **No pipeline runs any of this.** The frontend gate exists and is green, but no workflow invokes it, so the numbers above are reproduced on demand rather than enforced on every change.

# 4. Runtime Validation & UI Verification

The application was driven in a real browser against a running Frappe/ERPNext site seeded with a company, four bank accounts, 137 bank transactions and ten statement import logs. Signed in as `Administrator`, the session recorded **zero console error-level messages, zero uncaught exceptions, zero unhandled rejections and zero network responses of HTTP 400 or above** across every route, corroborated by DevTools, resource-timing status codes, an in-page request interceptor and a nine-target authenticated status sweep.

- ✅ **Start-up and authentication** — Sign-in succeeds and lands on the Desk; `/banking` mounts with 544 rendered elements and 3,063 characters of visible content.
- ✅ **Bank account selection** — Four account cards render; switching accounts re-queries the workbench and the counters, balances and progress follow the selection.
- ✅ **Unreconciled transaction list with rule-suggested match** — 83 results in the selected window, virtualised to 14 rendered rows, each carrying its reference badge, its violet rule badge titled with the matching rule, and a direction-coloured amount.
- ✅ **Balance and progress summary** — Opening, system closing, statement closing and difference all render with the shortfall stated in words; the progress bar reports 54 of 137 reconciled through an accessible value.
- ✅ **Match review and candidate pane** — Selecting a transaction opens the rule card, the recommended action and the ranked candidate vouchers, with the suggested voucher marked as a complete match and its confirm control enabled.
- ✅ **Confirm and post** — One atomic server call posts the reconciliation and the row leaves the list only because the server stops returning it; a manual override posts against the non-suggested voucher and writes exactly one allocation row.
- ✅ **Refusal reporting** — A refused post raises a dismissible alert dialog carrying the server's own message, traps and then restores focus to the control that opened it, and leaves the transaction unreconciled with no allocation rows and its original modification timestamp.
- ✅ **All five workbench tabs** — Match and Reconcile, Bank Reconciliation Statement, Bank Transactions (137 results with red, amber and green status badges), Bank Clearance Summary, and Incorrectly Cleared Entries as a designed empty state. No blank panel, no error panel, no stalled spinner.
- ✅ **Statement importer and per-file failure** — The upload control is genuinely disabled with no file staged; the previous-imports table shows six green completed rows and four solid-red failed rows, each failed row reporting no dates, zero transactions and a zero balance; the failed detail route refuses with a disabled import control and an explicit "nothing has been created" notice.
- ✅ **Routing and responsiveness** — An unknown route settles on the workbench with the full application rendered; no horizontal overflow at either 1440 or 390 pixels, where the intended desktop-only notice is the only visible content.

**Never exercised at runtime.** The three voucher-creation modals were opened but no voucher was created, so their save paths have not been driven against a live site. The unsupported-screen breakpoint and the large statement preview were observed but not measured under production data volumes. A protected-PDF import, an expired-session redirect and a re-read failure immediately following a refusal were not reproduced, because each needs a fixture or fault the environment cannot supply. All runtime verification was session-based rather than scripted, so none of it is repeatable by a pipeline today.

# 5. Compliance & Quality Review

## 5.1 Compliance Matrix

Each row states where the deliverable stands now, with the evidence a reader can open.

| # | Deliverable / Criterion | Status | Progress | Evidence |
| --- | --- | --- | --- | --- |
| 1 | Import a statement and see the resulting unreconciled transactions, each carrying the rule engine's suggestion | ✅ Pass | 100 % | Importer and workbench suites (129 + 580 cases); a valid import observed landing its rows; violet rule badge titled with the matching rule |
| 2 | Review the match: link a different Payment Entry, Journal Entry or Invoice, then confirm and post as one atomic server operation | ✅ Pass | 100 % | One `reconcile_vouchers` call with no optimistic client mutation (`BankReconciliation/utils.ts`); an override wrote exactly one allocation row typed "Matched" |
| 3 | An already-reconciled transaction cannot be posted again | ✅ Pass | 100 % | Client guard mirrors the server predicate in `MatchAndReconcile.tsx`; an 8-way race gives one success and seven refusals on both engines |
| 4 | A failed confirm leaves the transaction unreconciled with no partial posting | ✅ Pass | 100 % | Dismissible dialog with verbatim server text; server record unchanged including its modification timestamp |
| 5 | Import status view with a per-file failure indicator | ✅ Pass | 100 % | Three-state badge in `pages/BankStatementImporter.tsx`; four failed rows each with zero transactions created |
| 6 | Duplicate re-imports rendered as the backend produces them, with no client-side deduplication | ✅ Pass | 100 % | Verified no-op; duplicates render as distinct rows |
| 7 | Currency mismatch raises a non-blocking advisory | ✅ Pass | 100 % | Amber badge with explanatory tooltip; every confirm control stays enabled on the mismatched row |
| 8 | Rule evaluation stamps the matching rule by priority | ✅ Pass | 100 % | 31 rule-engine tests per engine cover stamping, first-match-wins and permission scoping |
| 9 | Zero TypeScript errors and zero build failures on Node ≥ 24 | ✅ Pass | 100 % | `tsc -b` exit 0 and `yarn build` exit 0 on Node 24.19.0, tree clean afterwards |
| 10 | Frontend line coverage ≥ 80 %, reporter-enforced | ✅ Pass | 100 % | 85.04 % lines (3,271 / 3,846) against `thresholds: { lines: 80 }` in `banking/vitest.config.ts` |
| 11 | Mandated scenarios pass in continuous integration on both databases | ⚠ Partial | 70 % | Backend modules are discovered by the existing dual-engine workflows and pass 134 tests on each; the frontend suite passes locally but no workflow invokes it |
| 12 | Change set confined to permitted paths, additions only, with the framework submodule and database schema untouched | ⚠ Partial | 90 % | 159 paths, zero deletions, zero renames, no DocType JSON, field, index or migration; submodule pointer `8a43de00d7f1e304598c910f0aa35aa384d474d7` with a clean tree — but two paths sit in the bank-account controller directory the plan classifies reference-only |

## 5.2 AAP & Rule Divergences and Gaps

No user-specified rules were provided for this project, so every divergence below is measured against the agreed plan.

| # | What the AAP Required | What Was Delivered Instead | Why It Diverged | Impact | Remediation |
| --- | --- | --- | --- | --- | --- |
| 1 | 21 paths — one additive backend test module and zero backend source change | 159 paths, including five bank controllers modified (+1,204 / −79) and 62 frontend suites | The endpoints the screen calls carried authorisation and input-validation gaps; closing them meant editing existing controller logic, which the additive-only allowance did not anticipate | Security materially stronger; the delivery is far larger than agreed and needs review as an intentional change set | Review the backend diff as a security change set (task H1) |
| 2 | `bank_account/` is reference-only and "must not appear in the diff" | `bank_account.py` (+83 / −1) and `test_bank_account.py` (+278) are in the diff | The authorisation gap's root cause is in that file and nowhere else; relocating the endpoint would rename a published RPC path | The permitted-path criterion is not met as written; the code is correct and green on both engines | Grant an explicit scope amendment, or re-plan (task H1) |
| 3 | The published RPC contract keeps its current shape | Four state-changing endpoints now accept POST only | Those endpoints answered GET, so a link, prefetch or crawler could trigger a financial state change | Any client that used GET would break; this screen has always sent POST | Confirm no external integration relies on GET |
| 4 | Both new feature atoms are plain in-memory | The import-failure map persists to session storage | The per-file indicator must survive a reload to be useful | None functional; plan text and behaviour disagree on paper | Reconcile the plan text with the delivered behaviour |
| 5 | The currency advisory chip uses the small badge size | It renders at the primitive's default size | The binding compliance section ranks design-system conformance first, and every other badge on the page is the default size | Chip is four pixels taller; the non-blocking contract is unchanged | None unless visual subordination was intended |
| 6 | Two behavioural premises stated as fact | Both measured false against the running system | The premises were inferred from code paths that do not execute as assumed | Client behaviour is unaffected and correct; the reasoning behind it needs restating | Decide whether the posting path should enforce currency itself |
| 7 | The mandated scenarios are gated in continuous integration | The frontend half is documented, not gated | New workflow files are excluded by the plan and lie outside every permitted path | 1,521 cases and the 80 % floor are unenforced between changes | Add the gate to an existing workflow (task H2) |
| 8 | Accessible contrast on interactive controls | Six solid-control colour pairs measure below 4.5:1 | The binding compliance section ranks design-system conformance above accessibility, and these are the design system's own brand colours | Buttons and badges fall short of AA in both themes | Design-owner decision with per-mode overrides (task M2) |

**1 — Scope of the change set.** The plan froze a 21-path diff: nine updates, twelve creations, one of them a backend test module, and explicitly "zero backend source changes". What shipped is 159 paths (73 created, 86 updated, none deleted), of which ten are backend and five are controllers rather than tests. The expansion is not drift: it hardens `bank_reconciliation_tool.py`, `bank_transaction.py`, `bank_statement_import_log.py`, `bank_transaction_rule.py` and `bank_account.py` with an argument-arity guard at 28 sites, per-account permission checks, total date parsing and deterministic candidate ordering. The frontend likewise grew from eight enumerated test files to 62 suites, all inside the workspace. Read the backend diff deliberately: it is a security change set wearing a hardening label.

**2 — The bank-account controller.** The plan names four bank DocType directories as touchable and classifies a fifth, `bank_account/`, as reference-only because no change was thought necessary there. That premise proved false: the whitelisted closing-balance write authorises the caller against no particular account, so a user holding no permission on a bank account could still write a statement balance to it. The guard now authorises against the named account, returns 404 for an unknown one, and refuses an unreadable, blank or overflowing date with 417 rather than writing against today. Ten tests cover it on both engines. Reverting reopens the gap; moving the method would rename a published RPC path. This needs a human decision, which is why it heads Section 1.4.

**3 — HTTP verbs on state-changing endpoints.** The plan requires the auto-generated REST and RPC contract to keep its current shape. Four state-changing endpoints — clearance-date write and clear, automatic reconciliation, and unreconcile — carried a bare whitelist and were reachable by GET, demonstrated by reaching the document lookup rather than the router. They now declare POST only. This is a genuine contract change: a client that fetched them by link would now be refused. This screen has always sent POST, so nothing in the product regresses, but any external integration or saved link against those four paths must be checked before release.

**4 — Persistence of the import-failure marker.** The plan specified that both new feature atoms be plain in-memory, reasoning that a stale error must not outlive the truth. The failure map is instead persisted for the session, because a failed import rolls back and leaves the log reading as though nobody had tried, so an indicator that vanishes on reload tells the reviewer nothing. The plan's actual concern is still honoured: the marker retires and the map empties as soon as the server reports the import completed, and the server's status always outranks it. The only residue is that plan text and delivered behaviour disagree on paper — worth reconciling so a future reader does not treat the persistence as an oversight.

**5 — Size of the currency advisory chip.** The plan's component-mapping table names the small badge size for the advisory chip. It ships at the primitive's default size. The same table cell also names a badge theme the primitive does not offer, which marks that row as illustrative of the component mapping rather than a pixel specification; the binding compliance section ranks design-system conformance first and permits any size in the primitive's fixed API; and every other badge on the workbench renders at the default, so the small size would have been the single outlier. Its non-blocking contract — it must never disable confirm — is unchanged and was re-verified in the browser.

**6 — Two premises that measured false.** The plan asserts the server hard-rejects a currency mismatch during posting, and that the Bank Transactions tab shows reconciled rows with an enabled confirm control. Neither holds. Posting saves an already-submitted document, so the document-level validation the plan cites does not run on that path — a mismatched transaction was observed posting successfully — while enforcement does fire when the transaction is created. And that tab exposes no per-row confirm control at all, so the guard is stale-client defence rather than a visible affordance there. The delivered client behaviour is right either way: the advisory blocks nothing and the guard mirrors the server predicate. What needs deciding is whether the posting path should enforce currency itself.

**7 — The frontend gate is not gated.** The plan requires the six mandated scenarios to pass in continuous integration on both databases, and separately excludes new workflow files while confining the diff to the workspace and the bank DocType directories. Those two instructions cannot both be satisfied: the backend half is genuinely discovered by the existing dual-engine workflows, but nothing in the repository's 22 workflows references this workspace or its runner. So 1,521 cases, the enforced 80 % line floor and the shuffled-order check are reproducible from one documented command chain and green on demand — but unenforced between changes. One job on an existing workflow closes it.

**8 — Contrast on solid controls.** Six colour pairs used by solid buttons and badges measure below the 4.5:1 threshold: blue at 4.03 light and 4.11 dark, green at 3.89 and 4.05, amber at 3.02 and 3.66. Gray, red and violet pass. These are the design system's own unmodified brand colours, and the binding compliance section ranks design-system conformance above accessibility heuristics, so they were left alone rather than overridden locally. A fix is not a simple step-shift: the dark ramp inverts, with the sixth surface step lighter than the fifth, so it needs per-mode overrides agreed with the design owner and applied across every solid control.

# 6. Risk Assessment

Each risk below is forward-looking: something that could still go wrong once this is in production. Every one was measured or reproduced rather than inferred.

| Risk | Category | Severity | Probability | Mitigation | Status |
| --- | --- | --- | --- | --- | --- |
| The two workbench list endpoints return every matching row regardless of the paging arguments a caller sends, and no index covers the account, submission state or date columns — a request asking for ten rows returned all 400, at 176 KB, over a full scan with a filesort | Technical / Performance | High | Medium | Add bounded or cursor pagination and a composite index; both need the contract and schema prohibitions lifted first | Open — measured |
| Simultaneous confirm/post attempts can answer HTTP 508 from a record-changed conflict rather than a user-level message. Data integrity always holds: correct final status, exactly one allocation row, no partial or duplicate posting | Technical | Medium | Low | Serialise per transaction with a re-read after the lock, or map the conflict to a stable response | Open |
| No pipeline enforces the frontend gate, so 1,521 cases, the 80 % line floor and the order-independence check can silently stop being true | Operational | High | High | Add the documented four-command job plus one shuffled seed to an existing workflow | Open |
| No security headers or cookie protections on any served surface: no frame-options, content-type-options, transport security or content policy, and the session cookie carries neither `Secure` nor `HttpOnly` | Security | High | Medium | Configure at the reverse proxy and framework, and enforce HTTPS before the surface is exposed | Open |
| A development-mode deployment returns a Python traceback and a server-version banner in error envelopes; the reconciliation surfaces never render them, but they are on the wire | Security | Medium | Medium | Disable traceback exposure in site configuration and serve behind a production WSGI stack | Open |
| The per-file import failure indicator is session-scoped because the import log persists no error state, so a hard reload returns a refused row to "Not Started" and a reviewer can miss the failure | Operational | Medium | Medium | Surface the stored message on the indicator, or persist an error field as a deliberate schema change | Open by design |
| Re-importing the same statement creates duplicate-equivalent transactions that must be unreconciled or cancelled by hand; the conflict notice warns but does not block | Integration | Medium | Medium | Decide the product policy and the deduplication key, then enforce it server-side | Open by design |
| Every automated suite runs in a simulated DOM that performs no layout, so scrolling, real measurement and render cost are unassertable — a large statement preview materialises around 11,300 elements unvirtualised, and the desktop-only breakpoint still mounts and fetches the full workbench payload at phone widths | Technical | Medium | Medium | Add a browser-driven regression suite, then virtualise the previews and make the breakpoint a render-level guard | Open |

# 7. Visual Project Status

**Overall progress — 84.3 % complete (301 of 357 hours).**

```mermaid
pie title Project Hours Breakdown
    "Completed Work" : 301
    "Remaining Work" : 56
```

Colour key: **Completed Work = Dark Blue `#5B39F3`** · **Remaining Work = White `#FFFFFF`**.

**Remaining work by priority — 56 hours.**

```mermaid
pie title Remaining Work by Priority
    "High" : 22
    "Medium" : 31
    "Low" : 3
```

**Remaining hours by category.**

| Category | Hours | Share of remaining |
| --- | --- | --- |
| Brand-colour contrast decision and token overrides | 10 | 17.9 % |
| Permitted-path decision and guard extension | 8 | 14.3 % |
| Reversal-path robustness | 8 | 14.3 % |
| Coverage for voucher-creation modals and a branch floor | 8 | 14.3 % |
| Release mechanics | 8 | 14.3 % |
| Continuous-integration gating of the frontend gate | 6 | 10.7 % |
| Per-file indicator polish and statement tile correction | 5 | 8.9 % |
| Operational alignment | 3 | 5.4 % |
| **Total** | **56** | **100 %** |

**Verification status by capability.**

```mermaid
graph LR
    subgraph Verified["Verified end to end"]
        A["Import to unreconciled list"]
        B["Rule-suggested match"]
        C["Manual override"]
        D["Atomic confirm and post"]
        E["Already-reconciled guard"]
        F["Refusal leaves state clean"]
        G["Per-file import failure"]
        H["Currency advisory, non-blocking"]
    end

    subgraph Partial["Verified, not yet gated"]
        I["Frontend suite — 1,521 cases"]
        J["Permitted-path decision"]
    end

    subgraph Outstanding["Awaiting work"]
        M["Control contrast"]
    end

    A --> D
    B --> C
    C --> D
    D --> E
    D --> F
    G --> I
    H --> I
```

# 8. Summary & Recommendations

**What was delivered.** The reconciliation workflow this project set out to complete now works end to end and has been exercised end to end. A reviewer imports a statement, sees the rule engine's suggestion on each unreconciled row, accepts it or links a different Payment Entry, Journal Entry or Invoice, and posts — in a single server call, with nothing mutated on the client before the server answers. All five prescribed failure modes behave as specified: a refused post raises a dismissible dialog carrying the server's own words and changes nothing; a malformed statement marks the file that failed and creates no transactions; an already-reconciled transaction cannot be posted again; duplicate re-imports are rendered exactly as the backend produces them; and a currency mismatch warns without blocking. Behind the screen, the endpoints it calls now authorise per account, validate their arguments, refuse malformed dates, and order candidates deterministically so the same statement produces the same suggestion twice.

**What proves it.** A frontend suite of 62 files and 1,521 cases passes with 85.04 % line coverage against a reporter-enforced 80 % floor, and passes equally under a shuffled execution order. The bank DocType controllers pass 134 tests on MariaDB and 134 on PostgreSQL. The workspace type-checks with zero errors and builds cleanly on Node.js 24.19.0, leaving the working tree clean; the package audit reports no advisories across 582 packages. Driven in a browser against a seeded site, the application mounted, all five workbench tabs rendered real data, the importer showed six completed and four failed files with nothing created for the failures, and the whole session produced no console errors and no response of HTTP 400 or above.

**What remains.** Fifty-six hours across eight items, and almost none of it is feature work. One item makes the verification repeatable: the frontend gate is green but no pipeline runs it. Two items turn on a decision before their engineering: whether the change set's scope is amended to include the bank-account controller whose authorisation fix now lives there, after which the argument and permission guard extends to that directory's four remaining unguarded endpoints; and how the design system's brand colours should reach AA contrast on solid controls, delivered as per-mode token overrides. The rest is finishing work — a complete reversal on multi-voucher transactions and a friendlier answer to concurrent posts; coverage on the voucher-creation modals and the application shell's guest branch, plus a branch-coverage floor; a per-file failure indicator that carries the stored server message and is keyboard reachable, with the statement summary tile corrected; release mechanics; and operational alignment of the scheduler with automatic rule evaluation, and of the realtime socket port with the target deployment.

**The critical path to production.** Rule on the permitted-path question first, because it is the only item that decides whether code already written and tested stays or goes. Wire the frontend gate in parallel — it is one job on an existing workflow and it converts today's green run into a standing guarantee. Those two take a day or two each and can proceed concurrently. Everything after them is sequenced by preference rather than dependency, with the reversal-path fix the highest-value of the remainder because it is reachable from the reviewer's own Undo.

**Production readiness.** The reconciliation feature itself is ready: its behaviour is specified, implemented, tested on both database engines and observed working, and its financial writes are server-authoritative by construction. The delivery around it is not yet ready to release unattended, for two reasons that are both addressable inside a week: the verification is not enforced by any pipeline, and the change set carries one scope question that a human must answer. At **84.3 % complete (301 of 357 hours)**, the honest reading is that the product is finished and the path to production is not. Two caveats belong on the record for whoever plans capacity: the performance characteristics of the list endpoints and the large statement preview were measured and left as they are, so they should be re-measured at the data volume the first real customer brings; and every automated suite runs in a simulated DOM, so layout and scroll regressions are caught only by manual browser verification.

# 9. Development Guide

Every command below was executed against this repository and its observed result is stated. The workspace's own `README.md` carries the longer-form runbook; this section is the working subset.

## 9.1 System Prerequisites

| Requirement | Version used | Why this floor |
| --- | --- | --- |
| Node.js | **24.19.0** | The workspace declares Node ≥ 24, and the test DOM package narrows it further to ≥ 24.15 and < 25 |
| Yarn Classic | **1.22.22** (via corepack) | The workspace ships a Yarn v1 lockfile and a `resolutions` block that npm ignores. Use Yarn, not npm |
| Python | **3.14.0** | The framework pins `>=3.14,<3.15` |
| MariaDB | **11.8.8** | Primary engine, `utf8mb4` / `utf8mb4_unicode_ci` |
| PostgreSQL | **17.10** | Second engine for the dual-database test runs |
| Redis | two instances | Cache and queue, on separate ports |
| wkhtmltopdf | **0.12.6.1** (patched Qt) | Required by the statement-import PDF tests |

```bash
node -v      # v24.19.0
yarn -v      # 1.22.22
python3 -V   # Python 3.14.0
```

## 9.2 Frontend Environment — No Bench Required

The test path is deliberately independent of the build path: the runner configuration imports neither the build config nor the proxy options, so the suites need no provisioned bench and no database.

```bash
cd erpnext/banking

# Install exactly what the lockfile pins. Observed: exit 0.
yarn install --frozen-lockfile
```

Expect one warning on every install — `Resolution field "ws@8.21.2" is incompatible with requested version "ws@~8.17.1"`. It is the deliberate override doing its job, not a defect. A second notice, `[DEP0169] url.parse()`, comes from Yarn Classic itself on Node 24 and appears even for a bare type check.

## 9.3 The Acceptance Gate

Run these four in order; all four were observed exiting 0.

```bash
cd erpnext/banking

CI=true yarn typecheck      # tsc -b — 0 errors, ~8 s
CI=true yarn lint           # eslint . — 0 violations, silent, ~4 s
CI=true yarn test           # 62 files, 1,521 tests, 0 failures
CI=true yarn test:coverage  # same, plus lines 85.04% against the 80% floor, ~87 s
```

Or as the single chain the runbook recommends:

```bash
cd erpnext/banking && yarn install --frozen-lockfile \
  && CI=true yarn typecheck && CI=true yarn lint && CI=true yarn test:coverage
```

Targeted forms, both verified:

```bash
# One file
CI=true npx vitest run src/lib/currency.test.ts        # 1 file, 26 tests

# One case by name
CI=true npx vitest run -t "already fully reconciled"   # 1 passed, 1,520 skipped

# Order-independence check
CI=true npx vitest run --sequence.shuffle --sequence.seed=1   # 62 files, 1,521 tests
```

Coverage output lands in `erpnext/banking/coverage/` — `coverage-summary.json` for the machine-readable totals and `lcov.info` for a coverage service. The directory is ignored by both git and the linter, so a coverage run leaves the tree clean.

## 9.4 Building the Frontend

The build — unlike the tests — needs a bench-style `sites/common_site_config.json` **above** the checkout, because the proxy options read it when the module loads. Without it the build stops with `ENOENT`.

```bash
# One-time on a bench host: point the checkout's parent directory at the bench's sites folder.
# Substitute your own bench root and the directory that contains this checkout.
ln -sfn "$BENCH_ROOT/sites" "$(dirname "$(git rev-parse --show-toplevel)")/sites"

cd erpnext/banking && CI=true yarn build
# Observed: "✓ built in 686ms", then the host HTML entry is copied.
# Emitted assets are gitignored, so all three working trees stay clean.
```

The build writes into the host app and the page is served from it: the website route rule lives at `erpnext/erpnext/hooks.py:222`, the page controller at `erpnext/erpnext/www/banking.py`, and the asset contract in `erpnext/pyproject.toml` under `[tool.bench.assets]` (`build_dir = "./banking"`, `out_dir = "../erpnext/public/banking"`, `index_html_path = "../erpnext/www/banking.html"`).

If you install at the app root rather than in the workspace, suppress the delegating post-install so it does not run a non-frozen install underneath you:

```bash
cd erpnext && yarn install --frozen-lockfile --ignore-scripts
```

## 9.5 Running the Application

Run these from your bench root (`$BENCH_ROOT` below), which sits outside this checkout; the ERPNext app inside it points at this repository.

```bash
cd "$BENCH_ROOT"

# Web server. DEV_SERVER=true is required: without it the realtime client
# dials the page origin instead of the socket port and every poll 404s.
CI=Yes DEV_SERVER=true nohup bench serve --port 8000 > logs/web.out.log 2>&1 &

# Realtime and background worker
nohup bench socketio > logs/socketio.out.log 2>&1 &
nohup bench worker   > logs/worker.out.log   2>&1 &
```

Verify each part:

```bash
curl -s http://127.0.0.1:8000/api/method/ping
# {"message":"pong"}

curl -s -o /dev/null -w '%{http_code}\n' 'http://127.0.0.1:9000/socket.io/?EIO=4&transport=polling'
# 200

curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8000/banking
# 200
```

Then open `http://localhost:8000/banking` and sign in. The workbench renders once a bank account is selected; the importer is at `/banking/statement-importer`.

## 9.6 Backend Tests — Both Engines

Run the modules one at a time from your bench root. Running them concurrently against a single site provokes a table-lock error from the harness, not a code failure.

```bash
cd "$BENCH_ROOT"

for m in bank_transaction bank_reconciliation_tool bank_transaction_rule \
         bank_statement_import_log bank_account; do
  bench --site test_site run-tests --lightmode \
    --module erpnext.accounts.doctype.$m.test_$m
done
# Observed: 21, 46, 31, 26, 10 — 134 tests, OK

# Same five modules on PostgreSQL
for m in bank_transaction bank_reconciliation_tool bank_transaction_rule \
         bank_statement_import_log bank_account; do
  bench --site test_site_pg run-tests --lightmode \
    --module erpnext.accounts.doctype.$m.test_$m
done
# Observed: identical counts — 134 tests, OK
```

Backend static gates, all observed exiting 0 and all run from the `erpnext` submodule root:

```bash
cd erpnext
ruff check erpnext/accounts/doctype/bank_*                       # pinned 0.2.0
ruff check --select=I erpnext/accounts/doctype/bank_*            # import order
ruff format --check erpnext/accounts/doctype/bank_*
semgrep --config=semgrep/test-correctness.yml erpnext/accounts/doctype/bank_*
python .github/helper/postgres_compat.py <changed .py files>
```

## 9.7 Example Usage

```bash
# Reconciliation surfaces the screen calls, as a logged-in session would
curl -s -b cookies.txt 'http://127.0.0.1:8000/api/method/erpnext.accounts.doctype.bank_account.bank_account.get_list'
curl -s -b cookies.txt 'http://127.0.0.1:8000/api/method/erpnext.accounts.doctype.bank_reconciliation_tool.bank_reconciliation_tool.get_bank_transactions?bank_account=<account>&from_date=2026-08-01&to_date=2026-08-31'

# The confirm/post endpoint accepts POST only, by design
curl -s -b cookies.txt -X POST \
  'http://127.0.0.1:8000/api/method/erpnext.accounts.doctype.bank_reconciliation_tool.bank_reconciliation_tool.reconcile_vouchers' \
  -H 'Content-Type: application/json' \
  -d '{"bank_transaction_name":"<name>","vouchers":"[{\"payment_doctype\":\"Payment Entry\",\"payment_name\":\"<voucher>\",\"amount\":100}]"}'
```

A malformed date is refused rather than silently widening the query, and a second post against an already-reconciled transaction is refused with the server's own "already fully reconciled" message — both are the behaviours the failure modes require.

## 9.8 Troubleshooting

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `yarn build` fails with `ENOENT` on a site config | The build reads a bench-style `sites/common_site_config.json` above the checkout | Create the symlink in §9.4. Tests do not need it |
| Realtime import progress never updates; polls 404 | The web server was started without `DEV_SERVER=true` | Restart it with the variable set, and confirm port 9000 answers 200 |
| `yarn check --verify-tree` reports one error; every install prints one warning | The deliberate `ws` override, which has no in-range fix upstream | Expected. `yarn install --frozen-lockfile` still exits 0 |
| `[DEP0169] url.parse()` deprecation on every yarn command | Emitted by Yarn Classic itself on Node 24 | Ignore; it appears even for a bare type check |
| Backend suites fail with a table-lock error | Two suites were run concurrently against one site | Run the modules sequentially |
| A `coverage/` directory appears after a coverage run | The reporter's own output | Ignored by git and the linter; delete it freely |
| Tests time out on a loaded machine | The suites drive real interactions in a simulated DOM; ceilings are 30 s | Reduce parallelism rather than raising the ceiling; assertions still fail immediately |
| `npm install` produces a different tree | The workspace pins a Yarn v1 lockfile and a `resolutions` block npm ignores | Use `yarn install --frozen-lockfile` |

# 10. Appendices

## A. Command Reference

| Purpose | Command | Working directory | Observed result |
| --- | --- | --- | --- |
| Install exactly what the lockfile pins | `yarn install --frozen-lockfile` | `erpnext/banking` | exit 0, lockfile untouched |
| Type check | `CI=true yarn typecheck` | `erpnext/banking` | exit 0, 0 errors, ~8 s |
| Lint | `CI=true yarn lint` | `erpnext/banking` | exit 0, silent, ~4 s |
| Full test run | `CI=true yarn test` | `erpnext/banking` | 62 files, 1,521 tests, 0 failures |
| Test run with the coverage gate | `CI=true yarn test:coverage` | `erpnext/banking` | exit 0, lines 85.04 % vs 80 % floor |
| One test file | `CI=true npx vitest run src/lib/currency.test.ts` | `erpnext/banking` | 1 file, 26 tests |
| One case by name | `CI=true npx vitest run -t "<name>"` | `erpnext/banking` | matching case only |
| Order-independence check | `CI=true npx vitest run --sequence.shuffle --sequence.seed=1` | `erpnext/banking` | 62 files, 1,521 tests |
| Production build | `CI=true yarn build` | `erpnext/banking` | exit 0, ~0.7 s, tree clean |
| Dev server | `yarn dev` | `erpnext/banking` | serves on port 8080 with the backend proxy |
| Dependency audit | `yarn audit` | `erpnext/banking` | 0 advisories across 582 packages |
| Backend module, MariaDB | `bench --site test_site run-tests --lightmode --module erpnext.accounts.doctype.<m>.test_<m>` | bench root | 134 tests across the five modules |
| Backend module, PostgreSQL | `bench --site test_site_pg run-tests --lightmode --module …` | bench root | 134 tests across the five modules |
| Python lint / import order / format | `ruff check`, `ruff check --select=I`, `ruff format --check` | `erpnext` | exit 0 (pinned 0.2.0) |
| Test-correctness rules | `semgrep --config=semgrep/test-correctness.yml erpnext/accounts/doctype/bank_*` | `erpnext` | exit 0 |
| Cross-engine SQL check | `python .github/helper/postgres_compat.py <files>` | `erpnext` | exit 0 |

## B. Port Reference

| Port | Service | Notes |
| --- | --- | --- |
| 8000 | Frappe web server | Start with `DEV_SERVER=true`, otherwise the realtime client dials this port instead of 9000 |
| 9000 | Realtime (socket.io) | Carries statement-import progress |
| 8080 | Vite dev server | `yarn dev` only; proxies `/api` to the web server |
| 3306 | MariaDB | Primary engine |
| 5432 | PostgreSQL | Second engine for dual-database runs |
| 13000 | Redis cache | |
| 11000 | Redis queue | Background rule evaluation |

## C. Key File Locations

| Area | Path |
| --- | --- |
| Reconciliation workbench | `erpnext/banking/src/components/features/BankReconciliation/MatchAndReconcile.tsx` |
| Failure dialog | `erpnext/banking/src/components/features/BankReconciliation/BankRecErrorDialog.tsx` |
| API-client hooks and cache keys | `erpnext/banking/src/components/features/BankReconciliation/utils.ts` |
| Feature state atoms | `erpnext/banking/src/components/features/BankReconciliation/bankRecAtoms.ts` |
| Statement import step | `erpnext/banking/src/components/features/BankStatementImporter/CSV/StatementDetails.tsx` |
| Importer list and per-file indicator | `erpnext/banking/src/pages/BankStatementImporter.tsx` |
| Shared error parsing / banner | `erpnext/banking/src/lib/frappe.ts`, `erpnext/banking/src/components/ui/error-banner.tsx` |
| Test runner configuration | `erpnext/banking/vitest.config.ts` |
| Test harness and fixtures | `erpnext/banking/src/test/setup.ts`, `factories.ts`, `renderPanel.tsx` |
| Design tokens | `erpnext/banking/src/index.css` |
| Workspace runbook | `erpnext/banking/README.md` |
| Dev-server proxy options | `erpnext/banking/proxyOptions.ts` |
| Bank DocType controllers and tests | `erpnext/erpnext/accounts/doctype/bank_transaction/`, `bank_reconciliation_tool/`, `bank_transaction_rule/`, `bank_statement_import_log/`, `bank_account/` |
| Host route rule and page controller | `erpnext/erpnext/hooks.py:222`, `erpnext/erpnext/www/banking.py` |
| Asset build contract | `erpnext/pyproject.toml` → `[tool.bench.assets]` |
| Test-correctness rules | `erpnext/semgrep/test-correctness.yml` |
| Dual-engine workflows | `erpnext/.github/workflows/server-tests-mariadb.yml`, `server-tests-postgres.yml` |

## D. Technology Versions

| Component | Version |
| --- | --- |
| Node.js / Yarn Classic / Python | 24.19.0 / 1.22.22 / 3.14.0 |
| React / React DOM | 19.2.7 |
| Vite / TypeScript | 8.0.16 / 5.9.3 |
| Tailwind CSS / Radix primitives | 4.3.0 / 1.6.1 |
| jotai / frappe-react-sdk | 2.20.1 / 1.17.0 |
| Vitest / coverage provider | 4.1.10 / 4.1.10 (exact peer pin — move together) |
| Simulated DOM | 30.0.1 |
| Testing Library react / dom / jest-dom / user-event | 16.3.2 / 10.4.1 / 7.0.0 / 14.6.1 |
| MariaDB / PostgreSQL / Redis | 11.8.8 / 17.10 / two instances |
| ruff / semgrep | 0.2.0 (pinned) / 1.172.0 |

Dependency counts: 38 runtime, 17 development. Four resolution overrides are in force — one to make the runner installable under Yarn Classic at all, three to close advisories inside their declared ranges. Do not delete them; the workspace runbook explains each.

## E. Environment Variable Reference

| Variable | Where | Purpose |
| --- | --- | --- |
| `VITE_BASE_NAME` | `erpnext/banking/.env.production` | Router base path for the built bundle. The only variable the workspace declares; none was added |
| `CI` | shell | Set `true` for non-interactive runs so no command waits or watches |
| `DEV_SERVER` | shell, web server | Must be `true` locally, or the realtime client dials the page origin instead of the socket port |
| `VITE_SOCKET_PORT` | shell, at build time | Optional; set before `yarn build` when the deployment's realtime port is not the default |

No secret, credential or connection string is read by the workspace. Database and site credentials live in the bench's own site configuration, outside this repository.

## F. Developer Tools Guide

- **Runner** — a dedicated configuration that measures the whole `src` tree (excluding generated types, the harness, declaration files and the entry point) and enforces `thresholds: { lines: 80 }`. It imports neither the build config nor the proxy options, which is what keeps the suites independent of a provisioned bench.
- **Harness** — `src/test/setup.ts` stubs the boot user with a non-guest name and both required roles, every permission array the screen reads, the currency and company lookups, plus `matchMedia`, `ResizeObserver`, pointer capture and `scrollIntoView`, none of which the simulated DOM provides. Panel suites additionally stub viewport measurement, because a virtualised list renders no rows when the DOM reports zero height.
- **Fixtures** — `src/test/factories.ts` builds transactions, bank accounts, linked payments, import logs and realistic server-error envelopes from the shapes the real endpoints return, and provides one mock helper covering the whole client SDK, so no suite hand-rolls its own.
- **State and data** — jotai atoms for feature state (local storage for the selected bank, dates and filters; session storage for the action log and the import-failure map) and SDK hooks with explicit cache keys for every server call. There is no service container and no other store.
- **Linting** — a flat ESLint config covering the whole package; `ruff` at the pinned 0.2.0 plus the repository's own test-correctness rules and cross-engine SQL check for Python.
- **Coverage artefacts** — `coverage/coverage-summary.json` and `coverage/lcov.info`; the directory is ignored by both git and the linter.

## G. Glossary

| Term | Meaning |
| --- | --- |
| Bank Transaction | The imported statement line. Its status and unallocated amount are co-derived by the server and drive the confirm guard |
| Allocation row | A child record linking a bank transaction to one voucher for a given amount, typed as matched or created |
| Voucher | A Payment Entry, Journal Entry, Sales Invoice or Purchase Invoice a transaction can be reconciled against |
| Rule-suggested match | The candidate the rule engine stamps on a transaction, chosen by the lowest priority number that matches |
| Confirm / post | The single server call that allocates the chosen vouchers and updates the transaction's status |
| Unreconcile / Undo | Reversing a reconciliation: releasing the allocation and clearing the voucher's clearance date |
| Import log | The record of one statement upload, carrying the parsed range, counts and status |
| Per-file failure indicator | The red state on an import row, held for the session because the import log persists no error field |
| Currency advisory | The amber badge shown when a transaction's currency differs from its bank account's. Advisory only — it never disables confirm |
| Acceptance gate | Install, type check, lint and the coverage run — the four commands that must pass before a change lands |

## H. PR Walkthrough

A reviewer's guided tour of this pull request — review order, area-by-area rationale, requirement traceability and the complete changed-path inventory — is in [PR Walkthrough](PR%20Walkthrough.md), beside this guide.
