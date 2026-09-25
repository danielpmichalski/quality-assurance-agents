---
name: qa-id-scheme
description: >
  Shared ID scheme and document conventions for the quality-assurance-agents test role agents
  (test-analyst, security-tester, acceptance-tester, usability-tester, model-based-tester,
  reviewer, performance-tester, technical-test-analyst, test-automation-engineer). Defines ID
  prefixes, role qualifiers, role-specific IDs and output file names so traceability works
  across all their documents. Preloaded into every agent of this plugin; also useful when a
  person or another agent needs to read, cross-reference or extend those documents.
user-invocable: false
---

# QA ID scheme

All test role agents in this plugin share one ID scheme so their documents can reference each
other and traceability holds end to end — from acceptance criteria through risks, test
conditions and test cases down to automated tests.

## Core prefixes

| Prefix | Meaning |
|---|---|
| `AC-` | Acceptance criterion / requirement |
| `F-` | Finding (test basis review finding, review anomaly, static analysis or coverage gap, suspected defect) |
| `ASM-` | Assumption |
| `R-` | Product risk |
| `TCO-` | Test condition |
| `TC-` | Test case |

## Role qualifiers

Every role except the base role inserts its qualifier after the prefix, so several roles can
work on the same test basis without ID collisions: `TC-SEC-01`, `R-PERF-03`, `F-REV-12`.
The **test-analyst is the base role and uses plain IDs** (`TC-01`, `R-02`).

| Role agent | Qualifier | Default output file |
|---|---|---|
| test-analyst | *(none)* | `<input>.test-design.md` |
| security-tester | `SEC` | `<input>.security-test-design.md` |
| acceptance-tester | `ACC` | `<input>.acceptance.md` |
| usability-tester | `UX` | `<input>.usability-test-design.md` |
| model-based-tester | `MBT` | `<input>.mbt.md` |
| reviewer | `REV` | `<reviewed-file>.review.md` |
| performance-tester | `PERF` | `<input>.performance-test-design.md` |
| technical-test-analyst | `TECH` | `technical-test-design.md` (scope root) |
| test-automation-engineer | `AUTO` | `<input>.automation.md` + test code |

## Role-specific IDs

These always carry the role's qualifier.

| Role | IDs |
|---|---|
| security-tester | `AST-SEC-` asset · `TB-SEC-` trust boundary · `THR-SEC-` threat |
| acceptance-tester | `US-ACC-` user story (only if the source has no story ID) · `RULE-ACC-` business rule · `EX-ACC-` example · `Q-ACC-` open question |
| usability-tester | `UP-UX-` user profile · `HE-UX-` heuristic evaluation finding · `A11Y-UX-` accessibility finding |
| model-based-tester | `MDL-MBT-` model · `S-MBT-` state · `T-MBT-` transition / flow edge · `DT-MBT-` decision table · `CT-MBT-` classification tree |
| performance-tester | `NFR-PERF-` derived/reformulated performance requirement · `OP-PERF-` operational profile · `LP-PERF-` load profile · `M-PERF-` metric |
| test-automation-engineer | `TC-AUTO-` low-level / automated test case, each refining one (or a listed few) source test cases |

## Rules

1. **`AC-` IDs are never qualified** — they belong to the test basis, not to a role.
   Reuse source IDs exactly as written (e.g. `AC-3`, `PROJ-123 AC2`). If the source has none,
   assign `AC-01`, `AC-02`, … in document order and say so.
2. **Only the acceptance-tester creates new `AC-` IDs** (numbering after the highest existing
   one for that story). A revised AC keeps its ID; a split AC keeps the original ID on the
   first part.
3. **Never renumber IDs owned by another document.** Reference them exactly as they appear
   (`R-03`, `TC-SEC-04`, `THR-SEC-02`).
4. **IDs are stable.** When updating an existing output document, keep every existing ID;
   add new ones after the highest in use; never reuse a retired ID — mark it *withdrawn*.
5. **Numbering** is two-digit zero-padded (`-01` … `-99`), extended to three digits only when a
   document exceeds 99 items of one kind.
6. **Reuse before duplicating.** If another role's document already has a risk or test case
   for the same concern, reference it instead of creating a near-duplicate; add your own only
   when your role adds a distinct angle.
7. **Traceability links use IDs only** — in tables write `AC-02, R-SEC-01`, never prose
   paraphrases of the item.
8. **In code**, automated tests carry the source test case ID in their display name or
   description (e.g. `TC-SEC-04: non-owner cannot delete item`), never in production code.

## Reading another role's document

- The qualifier tells you which role produced an ID and therefore which file to look in
  (see the output file table).
- Plain `TC-`/`TCO-`/`R-`/`F-`/`ASM-` IDs come from the test-analyst document.
- If a referenced ID can't be found in the documents you were given, don't invent its
  meaning — note it as an unresolved reference.
