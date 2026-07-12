# Contributing to Seiche

Thank you for looking under the hood. Seiche is a free, open source terminal
for dollar funding stress, and the part that matters most is the honesty
machinery: the sealed record, the walk forward validation, and the tests that
forbid look ahead. Contributions that strengthen that machinery are the most
valuable ones.

## Ground rules

- **The record is sacred.** Nothing may edit, reorder, or regenerate past
  entries in the notary ledger or the PIT record. If your change touches
  `notary.py`, the PIT keys, or the backtest, expect close review.
- **No look ahead.** The test suite mechanically forbids look ahead leakage in
  backtests. If your change breaks `test_backtest_split.py` or the leak audit,
  the change is wrong, not the test.
- **Fail loud.** Engines declare missing or stale data instead of silently
  interpolating. Follow the existing provenance patterns.
- **Free data only.** Collectors must use free, keyless public APIs. A data
  source that needs an API key or a subscription does not belong in core.

## Practical notes

- Backend is Python (FastAPI) under `backend/`, frontend is Vite/React under
  `frontend/`. Run the tests with `pytest backend/tests` from the repo root.
- Every engine ships with tests. A new engine without tests will not be
  merged.
- Keep pull requests small and single purpose. A good PR message explains what
  the change does to the published record, if anything.
- If you find a way to make the record lie, that is a security issue: please
  report it privately to desk@seiche.info before opening a public issue. You
  will be credited, loudly, unless you prefer otherwise.

## Decision making

Seiche currently has one maintainer (Mrinal Singh Meena). Decisions about the
published methodology, the composite weights, and anything that touches the
sealed record are made by the maintainer, in public, with reasons written
down. If the project grows more maintainers, this file will be updated with a
proper shared process before that happens, not after.

## License

Seiche is AGPL-3.0. By contributing you agree your contribution is licensed
under the same terms. The AGPL is a deliberate choice: anyone may run a fork,
but a fork that serves the public must also publish its source, so the record
stays checkable wherever it runs.
