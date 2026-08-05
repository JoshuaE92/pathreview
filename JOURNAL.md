## Week 7 — Issue selection

**Issue link:** https://github.com/jamjamgobambam/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has text: None

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The FaithfulnessChecker's `check()` method builds its context by reading each
chunk's text with `chunk.get("text", "")`. That empty-string default only
applies when the `"text"` key is missing — if the key exists but holds `None`,
`.get()` returns `None`, which then gets passed into `" ".join(...)`. Since
`join` only accepts strings, it raises a `TypeError` instead of handling the
chunk. So any retrieved context chunk with a null text value crashes the
faithfulness evaluation in `rag/evaluator/faithfulness_checker.py`. A successful
fix treats a `None` text the same as empty/missing (coerce to `""` or skip it)
so the join succeeds and the existing `test_none_context_chunk_text` passes.

**Branch name:** fix/153-faithfulness-checker-none-text

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

**Selection notes ("Is this right for me?"):**
Tier 1, tightly scoped. The bug, its cause, a reproduction, and the exact
failing test are all named in the issue, so scope is clear and small — a
one-line-ish fix in a single file plus verifying one existing test. It needs no
Chroma or external services to reproduce or fix, which keeps the setup burden
low. Good first contribution to a large codebase.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/JoshuaE92/pathreview/commit/a4365836decf9ccece65d182affc7902ec6dc1cb

**Reproduction summary:**
I ran the existing failing test with
`python -m pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_none_context_chunk_text -v`.
It fails at `rag/evaluator/faithfulness_checker.py:34` with
`TypeError: sequence item 0: expected str instance, NoneType found`, confirming
that a context chunk with `text: None` reaches `" ".join(...)` as `None` and
crashes the faithfulness evaluation exactly as issue #153 describes.

**PLAN.md link:** https://github.com/JoshuaE92/pathreview/blob/fix/153-faithfulness-checker-none-text/PLAN.md

**Walkthrough video (recommended):** [optional — add Loom link if recorded]

**Blockers or open questions:**
None blocking. Open question for Week 9: whether to coerce `None` with
`chunk.get("text") or ""` (simplest) vs. an explicit `is not None` check, and
whether to add a mixed None/valid multi-chunk test to strengthen coverage.

## Week 9 — Implementation & PR submission

### Mid-week check-in

**Progress:**
Implemented the fix in `rag/evaluator/faithfulness_checker.py`: the context
concatenation now uses `chunk.get("text") or ""`, so a chunk whose `text` is
`None` (or missing) is coerced to an empty string before `" ".join(...)`. Added
two edge-case tests following the existing patterns in
`tests/unit/test_faithfulness_checker.py`: `test_none_text_mixed_with_valid_chunks`
(a `None` chunk must not discard valid sibling chunks) and
`test_all_chunks_none_text` (all-`None` context returns a valid float, no crash).

**Verification:** `test_none_context_chunk_text` now passes, as do the two new
tests. The faithfulness module went from 18 passed / 4 failed to 21 passed /
3 failed.

**Scoping note (honest self-assessment):** The 3 still-failing tests
(`test_partial_support_returns_middle_score`, `test_multiple_context_chunks`,
`test_multiple_claims_varying_support`) are **pre-existing and unrelated** to
issue #153 — they fail because of the scoring threshold in `_is_supported`
(requires 2+ meaningful token overlaps), not the `None` crash. They fail on the
base branch before my change, so I am intentionally leaving them out of scope.
Likewise, CI runs `black --check .` and `ruff check .` repo-wide, and the
existing codebase is not black-formatted, so those jobs are already red on
`main`. I kept my diff minimal and matched the file's existing style rather than
reformatting unrelated code.

### Submission check-in

**PR link:** https://github.com/ascherj/pathreview/pull/854

**What I built:** A scoped fix for issue #153 plus two edge-case tests and inline
documentation explaining why `None` text is coerced to `""`.

**Files changed:**
- `rag/evaluator/faithfulness_checker.py` — coerce `None`/missing text to `""`
- `tests/unit/test_faithfulness_checker.py` — two new edge-case tests

**How to test:**
`python -m pytest tests/unit/test_faithfulness_checker.py -k "none" -v`

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
(In this codebase "passes" means my changes introduce no new failures. `make check`
is already red repo-wide on pre-existing `ruff`/`black` formatting, and 3 pre-existing
scoring tests in this module fail on `main`; my change adds no new failures and removes
one — `test_none_context_chunk_text` — see the scoping note in the mid-week check-in.)

**Draft PR feedback received from:** none (cohort was told peer review is not required)

**Blockers or open questions:**
None. Pre-existing scoring failures and repo-wide formatting are documented above
as out of scope for this issue.
