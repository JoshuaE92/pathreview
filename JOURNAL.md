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
