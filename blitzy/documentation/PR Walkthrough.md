# 1. PR Walkthrough — ERPNext Bank Reconciliation SPA

## 1.1 Purpose

This is a reviewer's guided tour of the pull request that completes and hardens the ERPNext Bank Reconciliation single-page application at `erpnext/banking/` and the bank DocType controllers it calls. For each area of the diff it states what changed, why, where to look, what deserves scrutiny and how to check the claim yourself.

- **Status, effort and open items** are in the [Project Guide](Project%20Guide.md). This walkthrough carries none of them.
- **The operating runbook** — install, test, build, run and troubleshoot — is the workspace README, `erpnext/banking/README.md`.
- **Requirement identifiers** come from the Agent Action Plan (AAP) that scoped the work. §1.3 defines them; §5 states every FM, TC and SC in full.

## 1.2 How to Use It

Read §2 for the shape of the change, follow the order in §3, use §4 area by area, check any single requirement end to end in §5, read the departures from the plan in §6, reproduce every gate with §7, and use §8 as the checklist that every changed path was looked at.

**Paths.** Every path here is relative to the superproject root. `git -C erpnext …` reports the same files without the leading `erpnext/`: its `banking/README.md` is this document's `erpnext/banking/README.md`.

## 1.3 Identifiers

| Identifier | Meaning | AAP section |
| --- | --- | --- |
| Gap A | No frontend test infrastructure: no runner, DOM environment, assertion library, `test` script or test file | §0.1.1.3 |
| Gap B | The build never type-checked, so one TypeScript error (`proxyOptions.ts`, TS7006 implicit `any`) stayed invisible, and no `typecheck` script existed | §0.1.1.3 |
| Gap C | The workspace README was the stock Vite scaffold | §0.1.1.3 |
| Gap D | Four missing failure-mode affordances: **D1** confirm not disabled for an already-reconciled transaction; **D2** no dismissible error dialog; **D3** the import failure handler discarded the backend error; **D4** no currency-mismatch indicator | §0.1.1.3 |
| FM1–FM5 | The five binding failure modes | §0.2.6 |
| TC1–TC6 | The six binding test scenarios | §0.2.8 |
| SC1–SC6 | The six success criteria | §0.2.9 |

# 2. The PR at a Glance

## 2.1 Repositories and Commit Ranges

| Repository | Range | Commits | What changes |
| --- | --- | --- | --- |
| Superproject (this repository) | `f8aa53cb2784bde42f0365d7d9fc5d63da028219..a49eee37d77a8837f831055b9141dbf021f8842b`, then the refinement commits on top (this walkthrough, and revisions to the Project Guide) | 35 in the range | The `erpnext` gitlink advances; the Project Guide is added; this walkthrough is added |
| `erpnext` submodule | `208a07e19ef4e50461a2a89c94378bd64649ff7e..3ed4ea28fbab13d7c77c2f813b4f99f6dbedb923` | 34 | Every code, test, configuration and README change |
| `frappe` submodule | pinned at `8a43de00d7f1e304598c910f0aa35aa384d474d7` | 0 | Nothing — pointer and contents unchanged |

The erpnext base, `208a07e19e`, is the upstream commit "Merge pull request #57452 from nabinhait/fix-child-qty-rate-type-hint". The tip, `3ed4ea28fb`, is "Resolve QA findings: data binding, importer durability, dialogs, a11y, perf, RPC hardening".

## 2.2 Superproject Changes

| Path | Status | Change |
| --- | --- | --- |
| `erpnext` | M (gitlink) | `208a07e19ef4e50461a2a89c94378bd64649ff7e` → `3ed4ea28fbab13d7c77c2f813b4f99f6dbedb923` |
| `blitzy/documentation/Project Guide.md` | A | Status, test results, compliance review, risks and development guide; its appendix H points here |
| `blitzy/documentation/PR Walkthrough.md` | A | This document |

The `frappe` gitlink does not move: `git diff f8aa53cb2784bde42f0365d7d9fc5d63da028219 HEAD -- frappe` prints nothing.

## 2.3 The erpnext Diff

`git -C erpnext diff --shortstat 208a07e19e 3ed4ea28fb` reports **159 files changed, 41,645 insertions, 938 deletions**.

| Status | Files |
| --- | --- |
| A — added | 73 |
| M — modified | 86 |
| D — deleted | 0 |
| R — renamed | 0 |
| **Total** | **159** |

| Area (subsection of §4) | Files | A | M | Lines added | Lines deleted |
| --- | --- | --- | --- | --- | --- |
| 4.1 Backend — bank DocType controllers and tests | 10 | 0 | 10 | 3,638 | 80 |
| 4.2 Reconciliation workbench | 27 | 1 | 26 | 2,413 | 471 |
| 4.3 Statement importer | 9 | 0 | 9 | 1,004 | 119 |
| 4.4 Shell, primitives, hooks, libraries and tokens | 39 | 6 | 33 | 2,560 | 165 |
| 4.5 Build, configuration and dependencies | 8 | 1 | 7 | 771 | 43 |
| 4.6 Test infrastructure | 3 | 3 | 0 | 2,089 | 0 |
| 4.7 Test suites | 62 | 62 | 0 | 27,972 | 0 |
| 4.8 Documentation | 1 | 0 | 1 | 1,198 | 60 |
| **Total** | **159** | **73** | **86** | **41,645** | **938** |

Areas 4.2–4.4 hold production source only: every frontend `*.test.ts` and `*.test.tsx` file, including `erpnext/banking/src/test/design-tokens.test.ts`, is counted once, under 4.7. Backend test modules stay in 4.1, beside their controllers.

# 3. Recommended Review Order

1. **Backend controllers (§4.1)** — `bank_transaction.py`, then `bank_reconciliation_tool.py`, `bank_statement_import_log.py`, `bank_transaction_rule.py`, and `bank_account.py` last. Financial writes and authorisation live here, and the Project Guide (§5.2 row 1) asks for this diff to be read as a security change set. `bank_transaction.py` comes first because it defines the argument guard the other controllers import; `bank_account.py` comes last because it is the open scope question (§5.2 row 2).
2. **Backend tests (§4.1)** — read each test module straight after its controller, while the guards are fresh. They pin the refusals and close TC2.
3. **The confirm/post failure path (§4.2)** — `bankRecAtoms.ts`, then `BankReconciliation/utils.ts`, `BankRecErrorDialog.tsx`, `MatchAndReconcile.tsx`. FM1, FM3, FM5 and TC3–TC6 converge here; reading state before its consumers makes the no-optimistic-mutation rule checkable.
4. **The statement importer (§4.3)** — `CSV/StatementDetails.tsx`, then `pages/BankStatementImporter.tsx`. FM2 and TC1 live here, and they reuse the atoms and dialog read in step 3.
5. **Shared UI, hooks, libraries and tokens (§4.4)** — `lib/frappe.ts`, `ui/error-banner.tsx`, `ui/markdown.tsx`, `ui/alert-dialog.tsx`, then `App.tsx` and `components/common/`. Every refusal is parsed and rendered through these, and the markdown sanitiser is the trust boundary for server-supplied text.
6. **The rest of the workbench (§4.2)** — layout, loading-state and accessibility edits across the remaining workbench components. Lower risk; read them once the core paths are understood.
7. **Build, configuration and dependencies (§4.5)** — small but load-bearing: the install fails without one of the `resolutions`, and the typecheck gate depends on two of these files.
8. **Test infrastructure (§4.6)** — every suite trusts the harness and the fixtures, so check that they model the real runtime and payloads before trusting a green run.
9. **Test suites (§4.7)** — sample by scenario label (`TC1`–`TC6`, `FM2`, `FM3`, `FM5`) rather than reading 27,972 lines in order.
10. **Documentation (§4.8)** — last, so you can check the README's claims against code you have already read.

# 4. Area-by-Area Walkthrough

Each subsection follows the same pattern: what changed, why, the key files, what to scrutinise, and how to verify. Frontend commands run from `erpnext/banking`; backend commands run from your bench root (`$BENCH_ROOT`) or the `erpnext` submodule root, as marked. §8 lists every file of each area.

## 4.1 Backend — Bank DocType Controllers and Tests (10 files)

**What changed**

- **A shared argument guard.** `bank_transaction.py` defines `refuse_missing_arguments`, a decorator placed directly beneath `@frappe.whitelist(...)`, plus `refuse_arguments` and `refuse_unknown_bank_transaction`. It is applied to 28 whitelisted methods: 18 in `bank_reconciliation_tool.py`, 6 in `bank_statement_import_log.py`, 3 in `bank_transaction.py` and 1 in `bank_transaction_rule.py`. A missing or surplus argument becomes a `frappe.throw` validation error, and an unknown transaction a `frappe.DoesNotExistError`, instead of a raw `TypeError`, `IndexError` or `AttributeError`.
- **`bank_transaction.py`.** `before_update_after_submit` now calls `validate_currency()`, so the reconcile path — `save()` on a submitted document — enforces the currency rule. `add_payment_entries` refuses a draft transaction before appending anything. The over-allocation message fills both of its placeholders. `unreconcile_transaction_entry` refuses ("Nothing to Undo") when the named voucher is not linked, instead of reporting success. `unreconcile_transaction` becomes POST-only.
- **`bank_reconciliation_tool.py`.** `reconcile_vouchers` runs three new steps in order: `parse_vouchers_to_reconcile` (malformed, non-list or empty payloads refused), `load_bank_transaction_for_reconciliation` (a `for_update=True` row lock, `docstatus` and `unallocated_amount` re-read through it, refusal when `modified` has moved), and `validate_vouchers_to_reconcile` (voucher type, existence, then read permission through `filter_permitted_vouchers`) — all before `add_payment_entries`. `get_linked_payments` checks read permission on the transaction and its Bank Account, refuses an absent `document_types`, and returns only candidates that pass `filter_permitted_vouchers`; the Payment Entry candidate query gains an `orderby(pe.name)` tiebreak. `parse_date_argument` normalises the module's client-supplied dates, refusing an unreadable, blank or overflowing value. `validate_reconciliation_voucher_type` and `validate_clearance_write_permission` guard clearance-date writes. `update_clearance_date`, `clear_clearing_date` and `auto_reconcile_vouchers` become POST-only.
- **`bank_statement_import_log.py`.** `read_statement_or_throw` wraps every third-party parser so an unreadable file is a titled refusal, logged with `frappe.log_error`. A file that parses to no rows, and a statement with no transactions to insert, are refused. Column-mapping and PDF table-region inputs are validated. The closing balance is written through the internal `_set_closing_balance_as_per_statement`.
- **`bank_transaction_rule.py`.** `run_rule_evaluation` requires write rather than read permission, scopes the pass to `get_permitted_companies_for_rule_evaluation()`, coerces a string force flag with `sbool`, and enqueues with a per-mode `job_id`, `deduplicate=True` and `enqueue_after_commit=True`. `_run_rule_evaluation` takes `companies` and skips a row locked by a concurrent reconciliation (`frappe.QueryDeadlockError`) instead of failing the pass.
- **`bank_account.py`.** `set_closing_balance_as_per_statement` authorises the caller against the named Bank Account, refuses an unknown account with `frappe.DoesNotExistError` and an unreadable date with a validation error, then delegates to `_set_closing_balance_as_per_statement`.
- **Tests.** Every new test is additive; no existing test was removed.

| Test module | Tests at `208a07e19e` | Tests at `3ed4ea28fb` |
| --- | --- | --- |
| `test_bank_transaction.py` | 8 | 21 |
| `test_bank_reconciliation_tool.py` | 4 | 46 |
| `test_bank_transaction_rule.py` | 20 | 31 |
| `test_bank_statement_import_log.py` | 13 | 26 |
| `test_bank_account.py` | 0 | 10 |
| **Total** | **45** | **134** |

**Why**

- **FM3 / TC5.** The authoritative already-reconciled guard is server-side: `add_payment_entries` throws "already fully reconciled" when `unallocated_amount <= 0`. The pre-existing `test_already_reconciled` covers it.
- **FM5.** "Behaviour is whatever the backend enforces": `validate_currency()` on the submitted-document path, pinned by `test_reconcile_refuses_a_currency_mismatch`.
- **FM1 / TC6.** The payload, lock and permission checks all run before the first child row is appended, so a refused post writes nothing.
- **TC2.** The AAP's only scenario with no backend coverage (§0.2.8) is closed by `test_run_rule_evaluation_stamps_matched_rule` and `test_run_rule_evaluation_lower_priority_number_wins`.
- **Divergences.** The controller hardening exceeds the plan's "one additive backend test module" — Project Guide §5.2 row 1. The `bank_account/` pair — row 2. The four POST-only endpoints — row 3.

**Key files**

- `erpnext/erpnext/accounts/doctype/bank_transaction/bank_transaction.py`
- `erpnext/erpnext/accounts/doctype/bank_reconciliation_tool/bank_reconciliation_tool.py`
- `erpnext/erpnext/accounts/doctype/bank_statement_import_log/bank_statement_import_log.py`
- `erpnext/erpnext/accounts/doctype/bank_transaction_rule/bank_transaction_rule.py`
- `erpnext/erpnext/accounts/doctype/bank_account/bank_account.py`
- the five `test_*.py` modules beside them

**Scrutinise**

- **Decorator order.** `@refuse_missing_arguments` must sit *under* `@frappe.whitelist(...)` at every one of the 28 sites, because `whitelist()` registers the object it decorates; the decorator's docstring sets out what breaks otherwise and why argument coercion is unaffected.
- **The reconcile sequence.** Confirm that `validate_vouchers_to_reconcile` runs after the transaction loads and before `add_payment_entries`, so every refusal leaves the transaction untouched.
- **Concurrency.** The Project Guide (§1.4, §6) records that concurrent posts can answer HTTP 508. Read that against `load_bank_transaction_for_reconciliation`, which takes a row lock and maps a `QueryDeadlockError` on that read to a "Refresh Required" refusal.
- **Permission source.** `filter_permitted_vouchers` uses `frappe.get_list`, which applies DocType and User Permissions; `frappe.get_all` would not.
- **Contract change.** The four POST-only endpoints refuse GET; confirm no external caller depends on GET (§5.2 row 3).
- **Scope.** `bank_account.py` and `test_bank_account.py` sit in a directory the plan classified reference-only (§5.2 row 2).
- **Test hygiene.** No `frappe.db.commit()`, no `frappe.db.truncate()` and no teardown override, per `erpnext/semgrep/test-correctness.yml`.

**Verify**

```bash
cd "$BENCH_ROOT"
for m in bank_transaction bank_reconciliation_tool bank_transaction_rule \
         bank_statement_import_log bank_account; do
  bench --site <site> run-tests --lightmode --module erpnext.accounts.doctype.$m.test_$m
done
# Measured: 21, 46, 31, 26, 10 — 134 tests, OK, on MariaDB and again on PostgreSQL

# From the erpnext submodule root: the guard count per controller (18, 6, 3, 1)
grep -c '^@refuse_missing_arguments' erpnext/accounts/doctype/bank_*/bank_*.py
```

## 4.2 Reconciliation Workbench (27 files)

**What changed**

- **`bankRecAtoms.ts` — feature state.**
  - `bankRecErrorDialogAtom`: the dialog payload, `FrappeError | null`, in memory.
  - `bankRecImportFailuresAtom`: per-import-log failure markers, persisted to `sessionStorage`.
  - `bankRecUploadRefusalsAtom`: refusals that happened before any import log existed, keyed by `uploadRefusalKey` and bounded by `MAX_UPLOAD_REFUSALS`, persisted to `sessionStorage`.
  - `bankRecLastRefusalAtom` and `bankRecReconcileInFlightAtom`: the last failed attempt (its error, its transaction, and whether the server answered) and the name of the transaction currently posting.
  - The account-keyed `bankRecSelectedTransactionAtom` family is replaced by `bankRecSelectedTransactionsAtom`, scoped by `selectedTransactionScopeKey` (account plus date range), so a selection cannot outlive the range it was made in.
  - The amount filter's display text starts empty instead of `'0.00'`.
- **`BankReconciliation/utils.ts` — the hook layer.** The rejection path of `useReconcileTransaction` raises a toast with a stable id, puts the raw error on `bankRecErrorDialogAtom`, revalidates the affected cache keys, and calls `convergeWithServer` to replace or clear a stale selection. `isServerRefusal` treats only a rejection carrying an HTTP status as a server refusal, so a request that never got an answer is not reported as one. `bankRecReconcileInFlightAtom` disables every candidate for the transaction while a post is outstanding and refuses re-entry. The toast's text is stripped of markup through an inert `DOMParser` document. `useSelectedBankAccountCurrency` reads the live `bank_account.get_list` row for FM5; `useRevalidateUnreconciledTransactions` invalidates the list without mounting a query.
- **`BankRecErrorDialog.tsx` — new.** A dismissible alert dialog composed from the existing `AlertDialog` and `ErrorBanner`. It passes the Frappe error by identity, so the banner shows the server's own words and severity, and it is dismissed with `AlertDialogCancel`. It is mounted in `MatchAndReconcile.tsx`, `CSV/StatementDetails.tsx` and `pages/BankStatementImporter.tsx`.
- **`MatchAndReconcile.tsx`.**
  - Mounts the dialog.
  - Computes `isAlreadyReconciled` (`status === 'Reconciled'` or `unallocated_amount <= 0`) and disables the Reconcile control with a tooltip reason, as it does while a post is in flight.
  - Computes `isCurrencyMismatch` and shows an advisory badge that never enters the `disabled` expression.
  - Adds `LastRefusalNotice`.
- **The remaining workbench components** carry layout, loading-state, accessibility and correctness edits, for example:
  - the `BankBalance.tsx` grid;
  - the `BankPicker.tsx` skeleton and invalidation-only revalidation;
  - the actions column pinned to the trailing edge in `BankTransactionList.tsx`;
  - the server's own summary-row strings in `BankReconciliationStatement.tsx`;
  - the party-name fallback in `RecordPaymentModalContent.tsx`;
  - a per-row `ErrorBoundary` in `ActionLog/ActionLogDialogBody.tsx`;
  - accessible names in `ActionLog/ActionLog.tsx`, `BankRecDateFilter.tsx`, `CompanySelector.tsx`, `Rules/RuleForm.tsx`, `Settings/Preferences.tsx` and `Settings/Rules/RuleList.tsx`.

**Why**

- **Gap D1 → FM3 / TC5:** the guarded Reconcile control.
- **Gap D2 → FM1 / FM3 / TC6:** the dialog and the rejection path.
- **Gap D4 → FM5:** the advisory badge.
- **TC3 / TC4:** the override and post paths through `useReconcileTransaction`.
- **FM4:** a deliberate no-op — no deduplication exists in the list and none was added.
- **Divergences.** The session-persisted failure map is Project Guide §5.2 row 4; the advisory chip's size is row 5; the edits to components the plan did not enumerate are part of row 1.

**Key files**

- `erpnext/banking/src/components/features/BankReconciliation/bankRecAtoms.ts`
- `erpnext/banking/src/components/features/BankReconciliation/utils.ts`
- `erpnext/banking/src/components/features/BankReconciliation/BankRecErrorDialog.tsx`
- `erpnext/banking/src/components/features/BankReconciliation/MatchAndReconcile.tsx`

**Scrutinise**

- **No optimistic mutation.** The rejection path changes no row; a reconciled row leaves the list only because the server stops returning it.
- **The guard mirrors the server.** `isAlreadyReconciled` must match `add_payment_entries`'s predicate, and remains a UX affordance — the server check is the control.
- **The advisory never blocks.** Unknown currency on either side means no badge, never a mismatch.
- **The selection atom.** The plan (AAP §0.2.5) expected two atoms appended and the sixteen existing atoms untouched. The selection atom family was replaced; check every consumer with `grep -rn bankRecSelectedTransactionsAtom src`.
- **What reaches the console.** A narrowed summary of a refusal, not the server traceback (README, "Workflow behaviour and the five failure modes", FM1).

**Verify**

```bash
cd erpnext/banking
CI=true npx vitest run src/components/features/BankReconciliation \
  src/components/features/ActionLog src/components/features/Settings
# Measured: 21 files, 656 tests passed
```

## 4.3 Statement Importer (9 files)

**What changed**

- **`CSV/StatementDetails.tsx`** — shared by the CSV and PDF flows.
  - The import's rejection handler now receives the `FrappeError`. It records it in `bankRecImportFailuresAtom` under the import log's name and opens `bankRecErrorDialogAtom`; a confirmed success retires the marker.
  - Attempt tracking ignores a stale response.
  - A statement with no readable transactions shows a refusal banner built with `makeClientRefusal` in place of the Import prompt.
  - Realtime progress is read as a percentage, and the row count is shown only from the terminal event.
  - The dialog is mounted here.
- **`pages/BankStatementImporter.tsx`.**
  - Each Previous Imports row now shows one of three states: green `Completed`, solid red `Failed` from the failure map, or grey otherwise. An authoritative `Completed` wins, and stale markers are pruned against the rows the list returns.
  - A refusal that happened before any import log existed is marked on the upload form through `bankRecUploadRefusalsAtom`.
  - The inline banner is suppressed while the dialog reports the same error object.
  - The dialog is mounted here too, because this page is a separate route tree.
- **Smaller edits:**
  - `import_utils.ts` reads the statement currency from `data.doc.currency`;
  - `CSV/CSVImport.tsx` and `PDF/PDFImport.tsx` stack their panes below `xl`;
  - `PDF/PDFTableEditor.tsx` names its page-stepping controls and announces the page with `role="status"`;
  - `RawTableGrid.tsx` moves its highlight colours onto design-system surface tokens;
  - `pages/ViewBankStatementImportLog.tsx` renders its loading and error states before the no-data guard;
  - `pages/BankStatementImporterContainer.tsx` matches the workbench's width cap.

**Why**

- **Gap D3 → FM2.** The handler used to take no argument and discard the backend error.
- **TC1.** The parsed statement and the import outcome are what the reviewer sees.
- **Failed import chip.** The three-state chip is the AAP's "Failed import chip" (§0.8.5.2).
- **Divergences.** Session persistence is Project Guide §5.2 row 4; the remaining edits are part of row 1.

**Key files**

- `erpnext/banking/src/components/features/BankStatementImporter/CSV/StatementDetails.tsx`
- `erpnext/banking/src/pages/BankStatementImporter.tsx`
- `erpnext/banking/src/components/features/BankStatementImporter/import_utils.ts`

**Scrutinise**

- **No client-side transaction creation.** The only write is the server's import call.
- **Server status outranks a marker.** `Completed` always wins.
- **Two refusal stages.** A refusal after the log exists marks its row; a refusal before any log exists marks the upload form.
- **Marker lifetime.** The Project Guide's §1.4 and §6 rows describe a hard reload returning a refused row to "Not Started", while §5.2 row 4 and `bankRecAtoms.ts` persist the marker map in `sessionStorage`. Confirm the behaviour in a browser.

**Verify**

```bash
cd erpnext/banking
CI=true npx vitest run src/components/features/BankStatementImporter \
  src/pages/BankStatementImporter src/pages/ViewBankStatementImportLog
# Measured: 8 files, 250 tests passed
```

## 4.4 Shell, Primitives, Hooks, Libraries and Tokens (39 files)

**What changed**

- **Application shell.**
  - `src/App.tsx` wraps the router in `ErrorBoundary` with an `AppRenderFailure` fallback, and mounts `SkipToContent`. The catch-all route becomes `<Navigate to="/" replace />`, so an unknown URL is replaced in history rather than left behind for Back to return to.
  - New components: `components/common/ErrorBoundary.tsx`, `components/common/AppRenderFailure.tsx` and `components/common/PageShell.tsx` (`SkipToContent`, `MainContent`).
  - `pages/BankReconciliation.tsx` measures its header to size the tab panels.
- **Design-system primitives.** 21 of the 43 modules in `src/components/ui/` are modified, and none is added:
  - `markdown.tsx` sanitises server markup after `rehypeRaw`, with an element allowlist, a `DROPPED_SUBTREES` set and an attribute allowlist;
  - `error-banner.tsx` gains an optional dismiss;
  - `alert-dialog.tsx` and `dialog.tsx` clamp their widths to the viewport;
  - `list-view.tsx` can pin a trailing column;
  - the rest carry focus, sizing and labelling fixes.
- **Hooks.**
  - New: `use-available-height.ts`, `use-return-focus.ts` (focus returns to the control that opened a dialog) and `use-scroll-overflow.ts`.
  - `useCurrentCompany.ts` hydrates its atom on init.
- **Libraries.**
  - `lib/frappe.ts` detects a transport failure while parsing messages and adds `makeClientRefusal`.
  - `lib/numbers.ts` adds `parseCurrencyInput` and `currencyInputValueToNumber` for strict amount parsing.
  - `lib/utils.ts` adds `TARGET_MIN_SIZE`.
- **Tokens.** `src/index.css` changes the values of four existing tokens in both themes — `--ink-gray-5`, `--ink-red-3`, `--ink-red-4` and `--focus-shadow-gray`. It introduces no new custom property.

**Why**

- **Gap D2.** The dialog composes the existing alert dialog and error banner, and refusals are parsed through the single shared path in `lib/frappe.ts` (AAP §0.2.10).
- **The sanitiser.** Every refusal renders server text through `markdown.tsx`, which makes the sanitiser the trust boundary.
- **Divergences.** The robustness, layout and accessibility edits here are part of Project Guide §5.2 row 1. The design system's solid brand colours are deliberately unchanged — row 8.

**Key files**

- `erpnext/banking/src/lib/frappe.ts`
- `erpnext/banking/src/components/ui/markdown.tsx`
- `erpnext/banking/src/components/ui/error-banner.tsx`
- `erpnext/banking/src/components/ui/alert-dialog.tsx`
- `erpnext/banking/src/App.tsx`
- `erpnext/banking/src/index.css`

**Scrutinise**

- **The sanitiser's lists.** It drops `class`, `id`, `style` and every `on*` attribute, and discards whole subtrees such as `script`, `style`, `iframe`, `svg`, form controls and media.
- **Primitives are shared.** Each primitive edit reaches every consumer, so read it as a change to all of them.
- **Token values.** These changes recolour existing text and focus rings everywhere. `erpnext/banking/src/test/design-tokens.test.ts` reads the real stylesheet and asserts WCAG contrast ratios for them.
- **The README's frozen list.** Its "Intentionally unchanged files" list names `index.html`, `src/index.css`, everything under `src/components/ui/`, and `src/lib/frappe.ts` as frozen, yet all of them appear in this diff. The README acknowledges the primitive and stylesheet edits in its "Scope boundaries" row and marks `markdown.tsx` as modified; the `index.html` and `src/lib/frappe.ts` edits are not acknowledged there.

**Verify**

```bash
cd erpnext/banking
CI=true npx vitest run src/App.test.tsx src/components/common src/components/ui \
  src/hooks src/lib src/pages/BankReconciliation.test.tsx
# Measured: 32 files, 599 tests passed
```

## 4.5 Build, Configuration and Dependencies (8 files)

**What changed**

| File | Change |
| --- | --- |
| `erpnext/banking/package.json` | Scripts `test` (`vitest run`), `test:coverage` (`vitest run --coverage`) and `typecheck` (`tsc -b`). Seven exactly pinned dev dependencies: `vitest` 4.1.10, `@vitest/coverage-v8` 4.1.10, `jsdom` 30.0.1, `@testing-library/react` 16.3.2, `@testing-library/dom` 10.4.1, `@testing-library/jest-dom` 7.0.0, `@testing-library/user-event` 14.6.1. Four `resolutions`: `vitest/vite` 8.1.2, `ws` 8.21.2, `nanoid` 3.3.17, `js-yaml` 4.3.1. No existing dependency version changes |
| `erpnext/banking/yarn.lock` | Regenerated for those packages and overrides (+680 / −36) |
| `erpnext/banking/vitest.config.ts` | New. jsdom, globals, `src/test/setup.ts`, the `src/**/*.{test,spec}.{ts,tsx}` glob, the `@` alias, 30-second test and hook timeouts, V8 coverage over `src/**/*.{ts,tsx}` (excluding `src/types/**`, `src/test/**`, `**/*.d.ts` and `src/main.tsx`) with `thresholds: { lines: 80 }`. It deliberately does not import `vite.config.ts` |
| `erpnext/banking/proxyOptions.ts` | `import type { IncomingMessage }` and the `req: IncomingMessage` annotation that removes the one TS7006 error |
| `erpnext/banking/tsconfig.node.json` | `vitest.config.ts` added to `include`, so the typecheck covers the runner configuration |
| `erpnext/banking/eslint.config.js` | `globalIgnores(["dist", "coverage"])`, so generated coverage output is not linted |
| `erpnext/banking/.gitignore` | `coverage` |
| `erpnext/banking/index.html` | The duplicate, restrictive viewport `<meta>` (`user-scalable=no`, locked scale) removed, restoring pinch-zoom |

**Why**

- **Gap A.** The runner, the DOM environment, the testing libraries and the `test` scripts.
- **Gap B.** The `typecheck` script and the `proxyOptions.ts` annotation.
- **Success criteria.** SC2 (zero TypeScript errors, clean build on Node ≥ 24) and SC3 (the reporter-enforced 80 % line floor).
- **The `vitest/vite` resolution.** It is the one the AAP calls mandatory (§0.5.2): without it, installing the runner under Yarn Classic fails outright.
- **The other three resolutions.** `ws`, `nanoid` and `js-yaml` close dependency advisories (README, "The four `resolutions` overrides — do not remove them").
- **Divergences.** The `.gitignore`, `eslint.config.js`, `tsconfig.node.json` and `index.html` edits lie outside the plan's enumerated paths, and are part of Project Guide §5.2 row 1.

**Key files** — all eight above; start with `erpnext/banking/package.json` and `erpnext/banking/vitest.config.ts`.

**Scrutinise**

- **Pins.** Every added version is exact, with no caret.
- **`resolutions`.** Each of the four must be justified before anyone removes it.
- **Independence from the bench.** `vitest.config.ts` must stay independent of `vite.config.ts`, which reads a bench file at module load.
- **Coverage scope.** The coverage `include` spans the whole `src` tree, so the floor is a statement about the application rather than a chosen subset.
- **The lockfile delta.** Check the added and re-resolved entries against the manifest.

**Verify**

```bash
cd erpnext/banking
yarn install --frozen-lockfile   # Measured: exit 0
CI=true yarn typecheck           # Measured: exit 0, zero errors
CI=true yarn lint                # Measured: exit 0, zero violations
CI=true yarn build               # Needs a bench-style sites/common_site_config.json above the checkout; measured: exit 0, "✓ built in 692ms"
```

## 4.6 Test Infrastructure (3 files)

**What changed**

- **`erpnext/banking/src/test/setup.ts`** is the global harness, with no exports. It installs everything at module load, before any suite imports application code, and stubs:
  - the Desk boot user (non-`Guest`), with the roles `System Manager`, `Accounts Manager` and `Accounts User`;
  - the permission arrays the screens read;
  - the `locals` currency and company lookups;
  - `matchMedia`, `ResizeObserver`, the Pointer Capture methods and `scrollIntoView`, none of which jsdom provides.
- **`erpnext/banking/src/test/factories.ts`** holds typed fixtures shaped from real server payloads, and the single `frappe-react-sdk` module mock, `createFrappeSDKMock`, with its `resetFrappeSDKMock`. Its fixtures include `makeUnreconciledTransaction`, `makeRuleMatchedTransaction`, `makeReconciledTransaction`, `makeCurrencyMismatchTransaction`, `makeBankStatementImportLog` and `makeAlreadyReconciledError`.
- **`erpnext/banking/src/test/renderPanel.tsx`** provides `renderPanel` (a per-render jotai store, router and tooltip provider) and `stubViewportMeasurement`. A virtualised list renders no rows when jsdom reports zero height, which is why the stub exists.

**Why**

- **Gap A.**
- **The AAP's implicit harness requirements (§0.1.2).**
  - `System Manager` must be among the roles, because `Bank Statement Import Log` grants permissions to that role only.
  - `matchMedia` and `ResizeObserver` must be stubbed.

**Key files** — the three above.

**Scrutinise**

- **Fixture shapes.** Check them against the endpoints they model, for example the already-reconciled message template `ALREADY_RECONCILED_MESSAGE_TEMPLATE`, which must match the server's own wording.
- **Mock reset.** The harness's `afterEach` unmounts the rendered tree, then calls `resetFrappeSDKMock()`, so no test inherits another's spies.
- **Coverage.** The harness is excluded from coverage (`src/test/**`), so its own size does not lift the figure.

**Verify**

```bash
cd erpnext/banking
CI=true npx vitest run --sequence.shuffle --sequence.seed=1   # Measured: 62 files, 1,521 tests passed — order-independent
```

## 4.7 Test Suites (62 files, 1,521 tests)

**What changed** — 62 new suites, all passing:

| Directory under `erpnext/banking/` | Suites | Tests |
| --- | --- | --- |
| `src` (`App.test.tsx`) | 1 | 9 |
| `src/components/common` | 3 | 42 |
| `src/components/features/ActionLog` | 1 | 29 |
| `src/components/features/BankReconciliation` | 17 | 551 |
| `src/components/features/BankReconciliation/Rules` | 1 | 22 |
| `src/components/features/BankStatementImporter` | 1 | 18 |
| `src/components/features/BankStatementImporter/CSV` | 2 | 72 |
| `src/components/features/BankStatementImporter/PDF` | 2 | 39 |
| `src/components/features/Settings` | 2 | 54 |
| `src/components/ui` | 13 | 197 |
| `src/hooks` | 6 | 84 |
| `src/lib` | 8 | 246 |
| `src/pages` | 4 | 142 |
| `src/test` (`design-tokens.test.ts`) | 1 | 16 |
| **Total** | **62** | **1,521** |

The mandated scenarios and failure modes are labelled in the `describe` names; `grep -rnE "describe\(.*(TC[1-6]|FM[1-5])" src` lists all 13 labelled blocks:

| Suite | Labelled blocks |
| --- | --- |
| `MatchAndReconcile.test.tsx` (116 tests) | `TC1 - the unreconciled list` (including the FM4 case "renders each row exactly as the backend produced it, with no client-side deduplication"), `TC2 - the rule-suggested match`, `TC3 - a manual override to a different voucher`, `TC4 - confirming a match posts the reconciliation`, `TC5 and FM3 - an already-reconciled transaction cannot be reconciled again`, `FM5 - a currency mismatch warns without blocking` |
| `utils.test.ts` (115 tests) | `useReconcileTransaction — the accepted post (TC4)`, `a refused post leaves the client's state exactly as it was` |
| `BankRecErrorDialog.test.tsx` (49 tests) | The server's own words across every envelope path, severity, structure, focus management, dismissal |
| `CSV/StatementDetails.test.tsx` (59 tests) | `TC1 — the parsed statement is presented for review`, `TC1 — importing posts one server-side operation and follows its answer`, `FM2 — a refused import surfaces the backend error and creates nothing` |
| `pages/BankStatementImporter.test.tsx` (101 tests) | `previous imports list (TC1)`, `per-row status chip (FM2)`, `a statement the server cannot read is refused before any log exists (FM2)` |

**Why**

- **Gap A.**
- **The six scenarios and five failure modes.** TC1–TC6 and FM1–FM5 (§5).
- **Success criteria.** SC1 for the frontend half, and SC3.
- **Divergences.** The plan enumerated eight frontend test files; 62 is part of Project Guide §5.2 row 1. That no workflow runs these suites is row 7.

**Key files** — the five suites in the table above.

**Scrutinise**

- **What is asserted.** Check assertions on server-authoritative behaviour: no row changes state after a refusal, and the server message renders verbatim.
- **`QUIRK` labels.** 33 of the 62 suites mark pinned pre-existing behaviour with a `QUIRK` label (`grep -rl QUIRK src --include='*.test.*'`). They specify surprising behaviour, including latent defects, rather than fixing it (README, "Known limits of this change set", row 8).
- **jsdom limits.** jsdom performs no layout, so layout, scrolling and render cost are outside every suite's reach (Project Guide §3, "Not Covered").

**Verify**

```bash
cd erpnext/banking
CI=true yarn test:coverage
# Measured: 62 files, 1,521 tests passed; lines 85.04 % (3,271 / 3,846) against the 80 % floor;
# statements 84.18 %, functions 78.5 %, branches 73.15 % (not gated)
```

## 4.8 Documentation (1 file)

**What changed** — `erpnext/banking/README.md` (+1,198 / −60, 1,211 lines) replaces the stock Vite scaffold text with the workspace runbook. Its sections:

- Stack and application shell; how the package reaches the host app;
- Requirements, quick start and scripts;
- The bench-config prerequisite and why `test` does not share it;
- Testing, the coverage gate and the four `resolutions` overrides;
- TypeScript layout, linting, source layout and conventions;
- Workflow behaviour and the five failure modes;
- Scope boundaries and intentionally unchanged files;
- What this change set contains, and its known limits.

**Why**

- **Gap C.**
- **AAP §0.10.1.2.** The README must document the bench-config prerequisite for `build`, and must state that `test` does not need it.

**Key files** — `erpnext/banking/README.md`.

**Scrutinise** — the README's figures against the diff. Several do not match the counts measured for this walkthrough:

- **"What this change set contains"** quotes per-file line counts that do not match `--numstat`. For example, `package.json` +16 / −2 against a measured +18 / −2; `MatchAndReconcile.tsx` +465 / −103 against +513 / −117; `pages/BankStatementImporter.tsx` +301 / −23 against +607 / −39.
- **Its backend table** is headed "four controllers and their four test modules" while five of each are in the diff, and gives per-module test counts (8 → 14, 4 → 29, 20 → 29, 13 → 20) that differ from the measured 21, 46, 31 and 26.
- **Its `bankRecAtoms.ts` row** says neither new atom is persisted, while `bankRecImportFailuresAtom` is stored in `sessionStorage`.
- **The "Scope boundaries" row** says no endpoint decorator changed, while four endpoints became POST-only (Project Guide §5.2 row 3).

**Verify**

```bash
git -C erpnext diff --numstat 208a07e19e 3ed4ea28fb -- banking/
```

# 5. Requirement Traceability

One row per binding requirement. Requirement wording is condensed from the AAP sections named in §1. Short file names refer to the key files in §4; each resolves to exactly one file under `erpnext/banking/src/` or `erpnext/erpnext/accounts/doctype/`.

| ID | Requirement | Implemented in | Proven by |
| --- | --- | --- | --- |
| FM1 | An API or network failure during confirm/post produces a dismissible error dialog; the transaction stays unreconciled with its state unchanged; no partial or duplicate posting | `useReconcileTransaction` rejection path in `BankReconciliation/utils.ts`; `BankRecErrorDialog.tsx`; `bankRecErrorDialogAtom` and `bankRecReconcileInFlightAtom` in `bankRecAtoms.ts`; server-side checks in `bank_reconciliation_tool.py` before any write | `BankRecErrorDialog.test.tsx`; `utils.test.ts` "a refused post leaves the client's state exactly as it was"; `MatchAndReconcile.test.tsx` "does not retry a refused post, so a refusal cannot become a duplicate posting"; `test_bank_reconciliation_tool.py` `test_reconcile_vouchers_refuses_*` |
| FM2 | A malformed or empty statement shows the backend's import error for that record with a per-file failure indicator; no client-side transaction creation | `CSV/StatementDetails.tsx`; `pages/BankStatementImporter.tsx`; `bankRecImportFailuresAtom` and `bankRecUploadRefusalsAtom`; `read_statement_or_throw` and the empty-statement refusals in `bank_statement_import_log.py` | `StatementDetails.test.tsx` "FM2 — a refused import surfaces the backend error and creates nothing"; `BankStatementImporter.test.tsx` "per-row status chip (FM2)"; `test_bank_statement_import_log.py` `test_get_data_refuses_a_file_that_parses_to_no_rows`, `test_insert_transactions_refuses_a_statement_with_no_transactions` |
| FM3 | An already-reconciled transaction has confirm disabled; a stale-client attempt surfaces the backend rejection in the same dialog and refreshes status | `isAlreadyReconciled` in `MatchAndReconcile.tsx`; `convergeWithServer` and cache revalidation in `BankReconciliation/utils.ts`; the server guard in `add_payment_entries` (`bank_transaction.py`) | `MatchAndReconcile.test.tsx` "TC5 and FM3 - an already-reconciled transaction cannot be reconciled again"; `test_bank_transaction.py` `test_already_reconciled` |
| FM4 | Duplicate transactions from a re-import render as the backend produces them; no client-side deduplication | Deliberate no-op: the list renders the server's rows unchanged | `MatchAndReconcile.test.tsx` "renders each row exactly as the backend produced it, with no client-side deduplication" |
| FM5 | A currency mismatch produces a non-blocking warning; behaviour is whatever the backend enforces | `isCurrencyMismatch` advisory in `MatchAndReconcile.tsx`; `useSelectedBankAccountCurrency` in `BankReconciliation/utils.ts`; `validate_currency()` in `before_update_after_submit` (`bank_transaction.py`) | `MatchAndReconcile.test.tsx` "FM5 - a currency mismatch warns without blocking"; `test_bank_transaction.py` `test_reconcile_refuses_a_currency_mismatch` |
| TC1 | Statement import produces the expected transaction list | `CSV/StatementDetails.tsx`; `pages/BankStatementImporter.tsx`; `MatchAndReconcile.tsx` | `StatementDetails.test.tsx` "TC1 — the parsed statement is presented for review" and "TC1 — importing posts one server-side operation and follows its answer"; `BankStatementImporter.test.tsx` "previous imports list (TC1)"; `MatchAndReconcile.test.tsx` "TC1 - the unreconciled list" |
| TC2 | Rule-based auto-match surfaces the correct suggested match | `_run_rule_evaluation` in `bank_transaction_rule.py`; the rule badge in `MatchAndReconcile.tsx` | `test_bank_transaction_rule.py` `test_run_rule_evaluation_stamps_matched_rule`, `test_run_rule_evaluation_lower_priority_number_wins`; `MatchAndReconcile.test.tsx` "TC2 - the rule-suggested match" |
| TC3 | Manual override links to a different ledger entry | Candidate vouchers in `MatchAndReconcile.tsx`; `useReconcileTransaction` in `BankReconciliation/utils.ts` | `MatchAndReconcile.test.tsx` "TC3 - a manual override to a different voucher" |
| TC4 | Confirming a match posts the reconciliation and updates status | `useReconcileTransaction` in `BankReconciliation/utils.ts`; `reconcile_vouchers` in `bank_reconciliation_tool.py` | `utils.test.ts` "useReconcileTransaction — the accepted post (TC4)"; `MatchAndReconcile.test.tsx` "TC4 - confirming a match posts the reconciliation"; `test_bank_reconciliation_tool.py` `test_reconcile_vouchers_posts_when_the_request_is_well_formed` |
| TC5 | An already-reconciled transaction cannot be reconciled again | `isAlreadyReconciled` in `MatchAndReconcile.tsx`; `add_payment_entries` in `bank_transaction.py` | `MatchAndReconcile.test.tsx` "TC5 and FM3 - …"; `test_bank_transaction.py` `test_already_reconciled` |
| TC6 | A failed confirm/post leaves the transaction unreconciled with no partial posting | The rejection path in `BankReconciliation/utils.ts`; `parse_vouchers_to_reconcile`, `load_bank_transaction_for_reconciliation` and `validate_vouchers_to_reconcile` in `bank_reconciliation_tool.py` | `utils.test.ts` "a refused post leaves the client's state exactly as it was"; `BankRecErrorDialog.test.tsx`; `test_bank_reconciliation_tool.py` `test_reconcile_vouchers_refuses_the_whole_request_when_one_voucher_is_unpermitted`; `test_bank_transaction.py` `test_reconcile_refuses_a_currency_mismatch`, which asserts no allocation row persists |
| SC1 | All six scenarios pass in continuous integration on both databases | The suites above; the existing workflows `erpnext/.github/workflows/server-tests-mariadb.yml` and `erpnext/.github/workflows/server-tests-postgres.yml` discover the backend modules | Measured: 134 backend tests per engine and 1,521 frontend tests pass. No workflow runs the frontend suite (Project Guide §5.2 row 7) |
| SC2 | `erpnext/banking/` builds with zero TypeScript errors and zero build failures through its own scripts on Node.js ≥ 24 | `typecheck` script in `erpnext/banking/package.json`; `erpnext/banking/proxyOptions.ts`; `erpnext/banking/tsconfig.node.json` | `CI=true yarn typecheck` and `CI=true yarn build`, both exit 0 on Node v24.22.2 |
| SC3 | Frontend line coverage ≥ 80 %, verified by the configured reporter | `thresholds: { lines: 80 }` in `erpnext/banking/vitest.config.ts` | `CI=true yarn test:coverage`: 85.04 % lines (3,271 / 3,846) |
| SC4 | The diff touches only `erpnext/banking/` and, if needed, the four bank DocType controller directories — additions only | — | `git -C erpnext diff --name-status 208a07e19e 3ed4ea28fb`: 159 paths, 0 deleted; 149 under `erpnext/banking/`, 8 in the four permitted directories, 2 in `bank_account/` (Project Guide §5.2 rows 1–2, §5.1 row 12) |
| SC5 | The `frappe/` submodule pointer and contents are unchanged | — | `git submodule status` shows `frappe` at `8a43de00d7f1e304598c910f0aa35aa384d474d7`; `git -C frappe status --porcelain` and `git diff f8aa53cb2784bde42f0365d7d9fc5d63da028219 HEAD -- frappe` print nothing |
| SC6 | The workflow completes end to end against a seeded test site, and each failure mode behaves as specified | The whole SPA and the five controllers | Project Guide §4, a browser session against a seeded site; not re-run for this walkthrough |

# 6. Where This PR Departs from the Plan

The table restates Project Guide §5.2 rows 1–8 and points to where each shows in the diff. Impact and remediation for every row are in Project Guide §5.2; this table adds no divergence of its own.

| # | What the AAP required | What the diff contains | Where to look |
| --- | --- | --- | --- |
| 1 | 21 paths — one additive backend test module and zero backend source change | 159 paths, including five bank controllers modified (+1,204 / −79) and 62 frontend suites | §4.1, §4.7, §8 |
| 2 | `bank_account/` is reference-only and does not appear in the diff | `bank_account.py` (+83 / −1) and `test_bank_account.py` (+278) | §4.1 |
| 3 | The published RPC contract keeps its current shape | `update_clearance_date`, `clear_clearing_date`, `auto_reconcile_vouchers` and `unreconcile_transaction` accept POST only | §4.1 |
| 4 | Both new feature atoms are plain in-memory | The import-failure map, `bankRecImportFailuresAtom`, persists to session storage | §4.2, §4.3 |
| 5 | The currency advisory chip uses the small badge size | It renders at the badge primitive's default size | `MatchAndReconcile.tsx` |
| 6 | Two behavioural premises stated as fact | Both measured false against the running system | Project Guide §5.2, narrative 6 |
| 7 | The mandated scenarios are gated in continuous integration | The frontend half is documented, not gated: none of the 22 files in `erpnext/.github/workflows/` mentions Vitest or the workspace | §4.7 |
| 8 | Accessible contrast on interactive controls | Six solid-control colour pairs measure below 4.5:1; the design system's brand colours are unchanged | §4.4, `erpnext/banking/src/index.css` |

**Read row 6 alongside the code.** The guide's narrative states that posting saves an already-submitted document, so the currency validation does not run on that path. At `3ed4ea28fb`, however, `before_update_after_submit` in `bank_transaction.py` calls `validate_currency()`, and `test_reconcile_refuses_a_currency_mismatch` asserts that `reconcile_vouchers` refuses a mismatch; the README's FM5 row describes the same enforcement. Settle which statement holds before relying on either.

# 7. How to Verify Locally

The commands are host-independent: substitute your own bench root and site names. The workspace README, `erpnext/banking/README.md`, and Project Guide §9 carry the longer runbook.

## 7.1 Frontend Gate — No Bench Required

```bash
cd erpnext/banking
yarn install --frozen-lockfile
CI=true yarn typecheck
CI=true yarn lint
CI=true yarn test:coverage
```

Every install prints the `Resolution field "ws@8.21.2" is incompatible…` warning, and Yarn Classic on Node 24 prints `[DEP0169] url.parse()` notices. Both are expected, and the README explains them.

## 7.2 Backend Tests — Both Engines

```bash
cd "$BENCH_ROOT"
for site in <mariadb-site> <postgres-site>; do
  for m in bank_transaction bank_reconciliation_tool bank_transaction_rule \
           bank_statement_import_log bank_account; do
    bench --site "$site" run-tests --lightmode --module erpnext.accounts.doctype.$m.test_$m
  done
done
```

Run the modules one at a time per site; concurrent runs against one site hit table locks.

## 7.3 Backend Static Gates — From the `erpnext` Submodule Root

```bash
cd erpnext
ruff check erpnext/accounts/doctype/bank_*
ruff check --select=I erpnext/accounts/doctype/bank_*
ruff format --check erpnext/accounts/doctype/bank_*
semgrep --metrics=off --error --config=semgrep/test-correctness.yml erpnext/accounts/doctype/bank_*
semgrep --metrics=off --error --config <frappe-semgrep-rules>/rules --config r/python.lang.correctness \
  --baseline-commit 208a07e19ef4e50461a2a89c94378bd64649ff7e erpnext/accounts/doctype/bank_*
python .github/helper/postgres_compat.py $(git diff --name-only 208a07e19e 3ed4ea28fb -- '*.py')
```

`<frappe-semgrep-rules>` is a checkout of Frappe's semgrep rule set.

## 7.4 Confinement Checks — From the Superproject Root

```bash
git diff --stat f8aa53cb2784bde42f0365d7d9fc5d63da028219 HEAD
git submodule status
git -C frappe status --porcelain
git -C erpnext diff --name-status 208a07e19e 3ed4ea28fb | cut -f1 | sort | uniq -c
```

## 7.5 Results Measured for This Walkthrough

Environment: Node v24.22.2, Yarn 1.22.22, Python 3.14.4, MariaDB 11.8.9, PostgreSQL 17.11, ruff 0.2.0, semgrep 1.184.0.

| Check | Result |
| --- | --- |
| `yarn install --frozen-lockfile` | exit 0 |
| `CI=true yarn typecheck` | exit 0, zero errors |
| `CI=true yarn lint` | exit 0, zero violations |
| `CI=true yarn test:coverage` | exit 0; 62 files, 1,521 tests passed; lines 85.04 % (3,271 / 3,846) against the 80 % floor; statements 84.18 %, functions 78.5 %, branches 73.15 % |
| `CI=true npx vitest run --sequence.shuffle --sequence.seed=1` | 62 files, 1,521 tests passed |
| `CI=true yarn build` | exit 0, "✓ built in 692ms"; working trees clean afterwards |
| Backend modules, MariaDB | `bank_transaction` 21, `bank_reconciliation_tool` 46, `bank_transaction_rule` 31, `bank_statement_import_log` 26, `bank_account` 10 — 134 tests, OK |
| Backend modules, PostgreSQL | The same five modules and counts — 134 tests, OK |
| `ruff check`, `ruff check --select=I`, `ruff format --check` | exit 0; 49 files already formatted |
| `semgrep` with `semgrep/test-correctness.yml` | 0 findings (3 rules, 49 files) |
| `semgrep` with Frappe's rules and `r/python.lang.correctness`, baseline `208a07e19e` | 0 findings (65 rules, 10 files) |
| `postgres_compat.py` on the 10 changed Python files | exit 0 |
| `git -C erpnext diff --name-status 208a07e19e 3ed4ea28fb` | 159 paths: 73 A, 86 M, 0 D |
| `git submodule status` | `erpnext` at `3ed4ea28fbab13d7c77c2f813b4f99f6dbedb923`, `frappe` at `8a43de00d7f1e304598c910f0aa35aa384d474d7` |

# 8. Appendix — Complete Changed-Path Inventory

Every path the PR changes, each listed exactly once: the superproject's three, then the erpnext submodule's 159, grouped by the areas of §4. Line counts are `git -C erpnext diff --numstat 208a07e19e 3ed4ea28fb`.

## 8.1 Superproject — 3 paths

| Path | Status | Area |
| --- | --- | --- |
| `erpnext` | M (gitlink) | Submodule pointer |
| `blitzy/documentation/Project Guide.md` | A | Project documentation |
| `blitzy/documentation/PR Walkthrough.md` | A | Project documentation |

## 8.2 erpnext Submodule — 159 paths

### 8.2.1 Backend — bank DocType controllers and tests — 10 files (0 A, 10 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/erpnext/accounts/doctype/bank_account/bank_account.py` | M | 83 | 1 |
| `erpnext/erpnext/accounts/doctype/bank_account/test_bank_account.py` | M | 278 | 0 |
| `erpnext/erpnext/accounts/doctype/bank_reconciliation_tool/bank_reconciliation_tool.py` | M | 600 | 49 |
| `erpnext/erpnext/accounts/doctype/bank_reconciliation_tool/test_bank_reconciliation_tool.py` | M | 983 | 1 |
| `erpnext/erpnext/accounts/doctype/bank_statement_import_log/bank_statement_import_log.py` | M | 215 | 18 |
| `erpnext/erpnext/accounts/doctype/bank_statement_import_log/test_bank_statement_import_log.py` | M | 432 | 0 |
| `erpnext/erpnext/accounts/doctype/bank_transaction/bank_transaction.py` | M | 202 | 3 |
| `erpnext/erpnext/accounts/doctype/bank_transaction/test_bank_transaction.py` | M | 366 | 0 |
| `erpnext/erpnext/accounts/doctype/bank_transaction_rule/bank_transaction_rule.py` | M | 104 | 8 |
| `erpnext/erpnext/accounts/doctype/bank_transaction_rule/test_bank_transaction_rule.py` | M | 375 | 0 |

### 8.2.2 Reconciliation workbench — 27 files (1 A, 26 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/src/components/features/ActionLog/ActionLog.tsx` | M | 6 | 1 |
| `erpnext/banking/src/components/features/ActionLog/ActionLogDialogBody.tsx` | M | 146 | 30 |
| `erpnext/banking/src/components/features/BankReconciliation/BankBalance.tsx` | M | 214 | 61 |
| `erpnext/banking/src/components/features/BankReconciliation/BankClearanceSummary.tsx` | M | 5 | 2 |
| `erpnext/banking/src/components/features/BankReconciliation/BankEntryModal.tsx` | M | 1 | 1 |
| `erpnext/banking/src/components/features/BankReconciliation/BankEntryModalContent.tsx` | M | 19 | 8 |
| `erpnext/banking/src/components/features/BankReconciliation/BankPicker.tsx` | M | 87 | 7 |
| `erpnext/banking/src/components/features/BankReconciliation/BankRecDateFilter.tsx` | M | 16 | 3 |
| `erpnext/banking/src/components/features/BankReconciliation/BankRecErrorDialog.tsx` | A | 282 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankReconciliationStatement.tsx` | M | 75 | 21 |
| `erpnext/banking/src/components/features/BankReconciliation/BankTransactionList.tsx` | M | 107 | 22 |
| `erpnext/banking/src/components/features/BankReconciliation/BankTransactionUnreconcileModal.tsx` | M | 21 | 3 |
| `erpnext/banking/src/components/features/BankReconciliation/CompanySelector.tsx` | M | 10 | 2 |
| `erpnext/banking/src/components/features/BankReconciliation/IncorrectlyClearedEntries.tsx` | M | 3 | 1 |
| `erpnext/banking/src/components/features/BankReconciliation/MatchAndReconcile.tsx` | M | 513 | 117 |
| `erpnext/banking/src/components/features/BankReconciliation/MissingFiltersBanner.tsx` | M | 16 | 4 |
| `erpnext/banking/src/components/features/BankReconciliation/RecordPaymentModal.tsx` | M | 1 | 1 |
| `erpnext/banking/src/components/features/BankReconciliation/RecordPaymentModalContent.tsx` | M | 111 | 73 |
| `erpnext/banking/src/components/features/BankReconciliation/Rules/RuleForm.tsx` | M | 42 | 28 |
| `erpnext/banking/src/components/features/BankReconciliation/SelectedTransactionsTable.tsx` | M | 2 | 4 |
| `erpnext/banking/src/components/features/BankReconciliation/TransferModal.tsx` | M | 1 | 1 |
| `erpnext/banking/src/components/features/BankReconciliation/TransferModalContent.tsx` | M | 29 | 29 |
| `erpnext/banking/src/components/features/BankReconciliation/bankRecAtoms.ts` | M | 232 | 6 |
| `erpnext/banking/src/components/features/BankReconciliation/utils.ts` | M | 424 | 24 |
| `erpnext/banking/src/components/features/Settings/KeyboardShortcuts.tsx` | M | 5 | 5 |
| `erpnext/banking/src/components/features/Settings/Preferences.tsx` | M | 18 | 4 |
| `erpnext/banking/src/components/features/Settings/Rules/RuleList.tsx` | M | 27 | 13 |

### 8.2.3 Statement importer — 9 files (0 A, 9 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/src/components/features/BankStatementImporter/CSV/CSVImport.tsx` | M | 17 | 3 |
| `erpnext/banking/src/components/features/BankStatementImporter/CSV/StatementDetails.tsx` | M | 216 | 24 |
| `erpnext/banking/src/components/features/BankStatementImporter/PDF/PDFImport.tsx` | M | 6 | 3 |
| `erpnext/banking/src/components/features/BankStatementImporter/PDF/PDFTableEditor.tsx` | M | 18 | 5 |
| `erpnext/banking/src/components/features/BankStatementImporter/RawTableGrid.tsx` | M | 31 | 6 |
| `erpnext/banking/src/components/features/BankStatementImporter/import_utils.ts` | M | 12 | 1 |
| `erpnext/banking/src/pages/BankStatementImporter.tsx` | M | 607 | 39 |
| `erpnext/banking/src/pages/BankStatementImporterContainer.tsx` | M | 49 | 32 |
| `erpnext/banking/src/pages/ViewBankStatementImportLog.tsx` | M | 48 | 6 |

### 8.2.4 Shell, primitives, hooks, libraries and tokens — 39 files (6 A, 33 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/src/App.tsx` | M | 30 | 10 |
| `erpnext/banking/src/components/common/AccountsDropdown.tsx` | M | 5 | 1 |
| `erpnext/banking/src/components/common/AppRenderFailure.tsx` | A | 38 | 0 |
| `erpnext/banking/src/components/common/BankLogo.tsx` | M | 10 | 3 |
| `erpnext/banking/src/components/common/ErrorBoundary.tsx` | A | 82 | 0 |
| `erpnext/banking/src/components/common/FileUploadBanner.tsx` | M | 7 | 1 |
| `erpnext/banking/src/components/common/LinkFieldCombobox.tsx` | M | 41 | 8 |
| `erpnext/banking/src/components/common/PageShell.tsx` | A | 74 | 0 |
| `erpnext/banking/src/components/common/PartyTypeDropdown.tsx` | M | 6 | 3 |
| `erpnext/banking/src/components/ui/alert-dialog.tsx` | M | 76 | 2 |
| `erpnext/banking/src/components/ui/calendar.tsx` | M | 63 | 2 |
| `erpnext/banking/src/components/ui/checkbox.tsx` | M | 7 | 1 |
| `erpnext/banking/src/components/ui/command.tsx` | M | 29 | 2 |
| `erpnext/banking/src/components/ui/dialog.tsx` | M | 120 | 5 |
| `erpnext/banking/src/components/ui/dropdown-menu.tsx` | M | 1 | 1 |
| `erpnext/banking/src/components/ui/error-banner.tsx` | M | 36 | 3 |
| `erpnext/banking/src/components/ui/file-dropzone.tsx` | M | 126 | 9 |
| `erpnext/banking/src/components/ui/form-elements.tsx` | M | 110 | 32 |
| `erpnext/banking/src/components/ui/form.tsx` | M | 74 | 10 |
| `erpnext/banking/src/components/ui/keyboard-keys.tsx` | M | 82 | 5 |
| `erpnext/banking/src/components/ui/list-view.tsx` | M | 167 | 7 |
| `erpnext/banking/src/components/ui/markdown.tsx` | M | 230 | 2 |
| `erpnext/banking/src/components/ui/progress.tsx` | M | 26 | 0 |
| `erpnext/banking/src/components/ui/radio-group.tsx` | M | 11 | 2 |
| `erpnext/banking/src/components/ui/select.tsx` | M | 1 | 1 |
| `erpnext/banking/src/components/ui/settings-dialog.tsx` | M | 66 | 8 |
| `erpnext/banking/src/components/ui/stats.tsx` | M | 3 | 3 |
| `erpnext/banking/src/components/ui/switch.tsx` | M | 16 | 1 |
| `erpnext/banking/src/components/ui/table.tsx` | M | 44 | 2 |
| `erpnext/banking/src/components/ui/tabs.tsx` | M | 15 | 1 |
| `erpnext/banking/src/hooks/use-available-height.ts` | A | 250 | 0 |
| `erpnext/banking/src/hooks/use-return-focus.ts` | A | 69 | 0 |
| `erpnext/banking/src/hooks/use-scroll-overflow.ts` | A | 152 | 0 |
| `erpnext/banking/src/hooks/useCurrentCompany.ts` | M | 15 | 1 |
| `erpnext/banking/src/index.css` | M | 67 | 7 |
| `erpnext/banking/src/lib/frappe.ts` | M | 69 | 0 |
| `erpnext/banking/src/lib/numbers.ts` | M | 131 | 1 |
| `erpnext/banking/src/lib/utils.ts` | M | 17 | 0 |
| `erpnext/banking/src/pages/BankReconciliation.tsx` | M | 194 | 31 |

### 8.2.5 Build, configuration and dependencies — 8 files (1 A, 7 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/.gitignore` | M | 4 | 0 |
| `erpnext/banking/eslint.config.js` | M | 2 | 1 |
| `erpnext/banking/index.html` | M | 12 | 2 |
| `erpnext/banking/package.json` | M | 18 | 2 |
| `erpnext/banking/proxyOptions.ts` | M | 4 | 1 |
| `erpnext/banking/tsconfig.node.json` | M | 2 | 1 |
| `erpnext/banking/vitest.config.ts` | A | 49 | 0 |
| `erpnext/banking/yarn.lock` | M | 680 | 36 |

### 8.2.6 Test infrastructure — 3 files (3 A, 0 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/src/test/factories.ts` | A | 1,513 | 0 |
| `erpnext/banking/src/test/renderPanel.tsx` | A | 136 | 0 |
| `erpnext/banking/src/test/setup.ts` | A | 440 | 0 |

### 8.2.7 Test suites — 62 files (62 A, 0 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/src/App.test.tsx` | A | 213 | 0 |
| `erpnext/banking/src/components/common/ErrorBoundary.test.tsx` | A | 189 | 0 |
| `erpnext/banking/src/components/common/LinkFieldCombobox.test.tsx` | A | 458 | 0 |
| `erpnext/banking/src/components/common/PageShell.test.tsx` | A | 132 | 0 |
| `erpnext/banking/src/components/features/ActionLog/ActionLogDialogBody.test.tsx` | A | 545 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankBalance.test.tsx` | A | 702 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankClearanceSummary.test.tsx` | A | 233 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankEntryModalContent.test.tsx` | A | 410 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankPicker.test.tsx` | A | 522 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankRecDateFilter.test.tsx` | A | 369 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankRecErrorDialog.test.tsx` | A | 939 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankReconciliationStatement.test.tsx` | A | 386 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankTransactionList.test.tsx` | A | 545 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/BankTransactionUnreconcileModalBody.test.tsx` | A | 371 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/IncorrectlyClearedEntries.test.tsx` | A | 209 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/MatchAndReconcile.test.tsx` | A | 3,068 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/MatchFilters.test.tsx` | A | 205 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/MissingFiltersBanner.test.tsx` | A | 107 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/RecordPaymentInvoices.test.tsx` | A | 790 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/RecordPaymentModalContent.test.tsx` | A | 274 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/Rules/RuleForm.test.tsx` | A | 433 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/TransferModalContent.test.tsx` | A | 346 | 0 |
| `erpnext/banking/src/components/features/BankReconciliation/utils.test.ts` | A | 1,979 | 0 |
| `erpnext/banking/src/components/features/BankStatementImporter/CSV/CSVRawDataPreview.test.tsx` | A | 498 | 0 |
| `erpnext/banking/src/components/features/BankStatementImporter/CSV/StatementDetails.test.tsx` | A | 1,349 | 0 |
| `erpnext/banking/src/components/features/BankStatementImporter/PDF/BBoxOverlay.test.tsx` | A | 353 | 0 |
| `erpnext/banking/src/components/features/BankStatementImporter/PDF/PDFTableEditor.test.tsx` | A | 528 | 0 |
| `erpnext/banking/src/components/features/BankStatementImporter/RawTableGrid.test.tsx` | A | 280 | 0 |
| `erpnext/banking/src/components/features/Settings/KeyboardShortcuts.test.tsx` | A | 264 | 0 |
| `erpnext/banking/src/components/features/Settings/Settings.test.tsx` | A | 866 | 0 |
| `erpnext/banking/src/components/ui/alert-dialog.test.tsx` | A | 327 | 0 |
| `erpnext/banking/src/components/ui/calendar.test.tsx` | A | 113 | 0 |
| `erpnext/banking/src/components/ui/command.test.tsx` | A | 79 | 0 |
| `erpnext/banking/src/components/ui/dialog.test.tsx` | A | 228 | 0 |
| `erpnext/banking/src/components/ui/file-dropzone.test.tsx` | A | 280 | 0 |
| `erpnext/banking/src/components/ui/focus-and-target.test.tsx` | A | 193 | 0 |
| `erpnext/banking/src/components/ui/form-elements.test.tsx` | A | 265 | 0 |
| `erpnext/banking/src/components/ui/form.test.tsx` | A | 188 | 0 |
| `erpnext/banking/src/components/ui/keyboard-keys.test.tsx` | A | 177 | 0 |
| `erpnext/banking/src/components/ui/list-view.test.tsx` | A | 318 | 0 |
| `erpnext/banking/src/components/ui/markdown.test.tsx` | A | 400 | 0 |
| `erpnext/banking/src/components/ui/progress.test.tsx` | A | 157 | 0 |
| `erpnext/banking/src/components/ui/table.test.tsx` | A | 119 | 0 |
| `erpnext/banking/src/hooks/use-available-height.test.tsx` | A | 447 | 0 |
| `erpnext/banking/src/hooks/use-mobile.test.ts` | A | 135 | 0 |
| `erpnext/banking/src/hooks/use-scroll-overflow.test.ts` | A | 381 | 0 |
| `erpnext/banking/src/hooks/useCurrentCompany.test.ts` | A | 116 | 0 |
| `erpnext/banking/src/hooks/useMultiFileUploadProgress.test.ts` | A | 239 | 0 |
| `erpnext/banking/src/hooks/usePaymentEntryCalculations.test.tsx` | A | 536 | 0 |
| `erpnext/banking/src/lib/amountFormula.test.ts` | A | 164 | 0 |
| `erpnext/banking/src/lib/checks.test.ts` | A | 91 | 0 |
| `erpnext/banking/src/lib/company.test.ts` | A | 139 | 0 |
| `erpnext/banking/src/lib/currency.test.ts` | A | 285 | 0 |
| `erpnext/banking/src/lib/date.test.ts` | A | 346 | 0 |
| `erpnext/banking/src/lib/file.test.ts` | A | 161 | 0 |
| `erpnext/banking/src/lib/frappe.test.ts` | A | 621 | 0 |
| `erpnext/banking/src/lib/numbers.test.ts` | A | 639 | 0 |
| `erpnext/banking/src/pages/BankReconciliation.test.tsx` | A | 474 | 0 |
| `erpnext/banking/src/pages/BankStatementImporter.test.tsx` | A | 2,184 | 0 |
| `erpnext/banking/src/pages/BankStatementImporterContainer.test.tsx` | A | 90 | 0 |
| `erpnext/banking/src/pages/ViewBankStatementImportLog.test.tsx` | A | 277 | 0 |
| `erpnext/banking/src/test/design-tokens.test.ts` | A | 240 | 0 |

### 8.2.8 Documentation — 1 file (0 A, 1 M)

| Path | Status | Added | Deleted |
| --- | --- | --- | --- |
| `erpnext/banking/README.md` | M | 1,198 | 60 |
