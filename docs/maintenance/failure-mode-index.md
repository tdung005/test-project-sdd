# Failure Mode Index

## Purpose

Record failure modes that are worth keeping because they help prevent repeat incidents across tickets.
Keep entries concise and prefer reusable prevention/detection guidance over ticket-specific prose.

---

## Organization-Derived Failure Modes

| ID | Failure mode | Trigger | Prevention | Detection |
|---|---|---|---|---|
| FMI-ORG-001 | Migration IT fails on a runner without Docker/Testcontainers | Running `OrganizationMigrationIntegrationTest` where Docker socket access is unavailable | Require a Docker-enabled runner for DB migration IT | Test fails before PostgreSQL container startup |
| FMI-ORG-002 | Raw translation keys appear in the UI | Missing or mistyped `Pages.Organization.*` locale entry | Keep FE translations in `public/locales/{en,ja,vi}/locale.json` and test translated labels | UI shows untranslated key text instead of a localized label |
| FMI-ORG-003 | Stale update/delete overwrites newer organization data | `version` is omitted or not checked atomically in update/delete SQL | Require `version` on mutating requests and check affected row count | Concurrent edit/delete unexpectedly succeeds |
| FMI-ORG-004 | Soft-deleted organization code/name cannot be reused | Partial unique index is defined without `deleted_at IS NULL` | Scope uniqueness to active rows only | Create/update fails for a code/name that exists only in Deleted |
| FMI-ORG-005 | Non-admin users can reach Organization UI but get poor feedback | FE route guard is missing or backend returns 200 + error string instead of a real forbidden response | Keep FE admin gate and backend forbidden handling aligned | Non-admin sees the page but a mutation appears to succeed or fails silently |

---

## Customer-Derived Failure Modes

| ID | Failure mode | Trigger | Prevention | Detection |
|---|---|---|---|---|
| FMI-CUS-001 | Playwright locator strict-mode collision on duplicated labels | The same visible label appears in the list row, the open drawer, and the option popup | Scope option clicks to the active popup/container or use a stable `data-testid` | Playwright reports a strict-mode violation when clicking the option |
| FMI-CUS-002 | Soft-deleted master data cannot be reused | Partial unique index is defined without `deleted_at IS NULL` for code/alias reuse | Keep uniqueness scoped to active rows only | Reusing a value from a soft-deleted row fails unexpectedly |
| FMI-CUS-003 | Stale update/delete overwrites newer master data | `version` is omitted from the mutating `WHERE` clause | Require `version` and verify affected row count | Concurrent edit/delete unexpectedly succeeds |
| FMI-CUS-004 | Raw translation keys appear in the UI | Missing or mismatched locale key for a Customer/Organization message | Keep FE translation keys aligned with the backend message contract | UI shows untranslated `Pages.*` keys or fallback text |

---

## Team-Derived Failure Modes

| ID | Failure mode | Trigger | Prevention | Detection |
|---|---|---|---|---|
| FMI-TEAM-001 | Duplicate active Team Code slips through create/update | Service check or active-scope unique constraint is missing or outdated | Enforce duplicate-code validation in service and DB for active rows only | Create/update test sees two active Teams with the same code |
| FMI-TEAM-002 | Same member is added twice to one Team | Membership uniqueness is checked only in UI or only in service | Enforce one active `(team_id, member_key)` membership in service and schema | Add-member test creates duplicate active membership rows |
| FMI-TEAM-003 | Team delete leaves active memberships behind | Soft-delete path does not cascade to active TeamMember rows in one transaction | Make delete + membership inactivation transactional | After Team delete, active member list still returns rows |
| FMI-TEAM-004 | Team member lookup offers invalid selection choices | Member/role lookup is not filtered to active/valid records | Keep add-member lookups read-only and filtered to valid options | UI shows inactive member/role or add-member API accepts it |
| FMI-TEAM-005 | Legacy Team-Project coupling reappears in Team flow | New Team UI/API accidentally exposes Project assignment | Keep Team-Project assignment out of scope and review route/API contracts | Team screen shows project selector or assignment endpoint |
| FMI-TEAM-006 | Raw locale keys appear in Team UI | Missing `en`/`vi`/`ja` Team translation key | Keep Team messages in locale JSON and verify build/test coverage | UI renders untranslated Team labels or messages |

---

## Artifact Scanner-Derived Failure Modes

| ID | Failure mode | Trigger | Prevention | Detection |
|---|---|---|---|---|
| FMI-AS-001 | Scanner boundary drifts back into parser behavior | Markdown parsing or full content persistence is added inside the scanner flow | Keep scanner metadata-only and hand off parsing to the parser module | Review diff for parsing logic, full-content columns, or content persistence |
| FMI-AS-002 | Unknown ticket becomes a hard failure | A ticket path is not found in dimension and the scan exits early | Preserve auto-create minimal ticket behavior for unknown tickets | Missing-ticket test or run result fails before inventory is written |
| FMI-AS-003 | Legacy tables get reused accidentally | New persistence code follows old mapper patterns instead of V4-only path | Review persistence against V4 schema and current inventory view before merge | Integration test or code review finds legacy table references |
| FMI-AS-004 | Current inventory returns stale data | Query does not read from `vw_artifact_inventory_current` or latest snapshot semantics | Always read current inventory from the dedicated view | Latest-vs-oldest inventory comparison test disagrees |
| FMI-AS-005 | Phase0 seed or artifact-type mapping drifts | File scope changes without updating seed/config together | Keep phase0 file list and artifact type seed documented together | Full scan shows missing mapping or unexpected skipped items |

---

## Notes

- Add a new row only when the failure mode has a real chance to recur.
- Avoid turning one-off process issues into permanent general rules.
- Prefer a short link to a living doc when the prevention pattern is already documented elsewhere.
