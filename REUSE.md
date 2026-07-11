# Reusing parts of Seiche

Seiche is a whole terminal, but the parts that make it trustworthy are not
specific to money markets. This page shows how to lift the two most reusable
pieces, the skill test and the free data, and points at the rest. Everything
here is AGPL, so what you build on it stays open the same way.

## 1. Test whether your own signal actually has skill, the honest way

The core question behind every engine in Seiche is: does this signal really
tend to rise *before* the events I care about, or am I fooling myself? Two
tested helpers answer it with no threshold to cherry-pick and no way to smuggle
the future into the past.

```python
import pandas as pd
from seiche.engines.backtest import _event_auroc, _significance

# Your signal, one value per business day. Higher should mean "trouble ahead".
# Works for anything: an outage risk score, a disease surveillance index,
# a fraud pressure gauge, a churn signal.
signal = pd.Series(my_daily_scores, index=pd.bdate_range("2020-01-01", periods=500))

# The dates the thing you were trying to see coming actually happened.
events = pd.DatetimeIndex(["2020-03-16", "2020-09-16", "2021-06-30"])

auroc = _event_auroc(signal, events)              # 0.5 is a coin flip, 1.0 is perfect
verdict = _significance(signal, events, n_perm=2000)

print("threshold-free skill (AUROC):", auroc)
print("beats chance placement:", verdict["ok"], "p =", verdict["p_value"])
```

What each one does:

- `_event_auroc` asks whether the signal tends to sit high in the run up to your
  events, scored across every possible threshold at once. There is nothing to
  tune, so there is nothing to overfit.
- `_significance` keeps the exact same alert runs (same count, same lengths) and
  relocates them at random a couple of thousand times. The p value is how often
  blind luck places them at least as well as your real signal did. A small p
  means the timing was not a coincidence.

If you want to go further and measure how much a *specific* form of cheating
would have flattered your backtest, the one-switch leakage protocol in
[`backend/seiche/engines/leakaudit.py`](backend/seiche/engines/leakaudit.py)
rebuilds the same index with exactly one discipline broken (full-sample
normalization, a forward-peeking smoother, an in-sample fitted threshold) and
prints the Leakage Gain each break would have bought. A clean pipeline sits at
the bottom of its own audit table.

## 2. Pull the same free, keyless public data

Every source Seiche reads is free and needs no API key. The simplest one, FRED,
is a plain CSV endpoint you can hit in one line:

```python
import pandas as pd

# The same keyless endpoint Seiche uses, no key, no account.
sofr = pd.read_csv("https://fred.stlouisfed.org/graph/fredgraph.csv?id=SOFR")
print(sofr.tail())   # a date column and a SOFR value column
```

The hardened versions, with caching, retries and backoff, live in
[`backend/seiche/sources/`](backend/seiche/sources/): `fred`, `nyfed` (NY Fed
Markets), `ofr` (OFR short-term funding monitor), `fiscaldata` (US Treasury),
`cftc`, `ecb`, `bis` and `crypto` (DeFiLlama and Coinbase). Each one wraps a
keyless public endpoint like the FRED example above, so if your project touches
any of these sources the client is already written and tested.

## 3. The as-published ledger pattern

If you publish predictions, the honest thing is to seal each one the day you
make it so nobody, including you, can quietly rewrite a bad call later.
[`backend/seiche/notary.py`](backend/seiche/notary.py) does this: each daily
board is hashed and chained to the one before it, and the point-in-time record
is served at `/api/pit`. The idea travels to any project that wants its track
record to be a matter of record rather than a matter of trust. Lift the pattern,
or the file.

## 4. The fusion scaffolding

The habit that keeps Seiche honest is that no judgment call is buried in the
math. Every weight, threshold and rule sits in one place,
[`backend/seiche/config.py`](backend/seiche/config.py), and the engines in
[`backend/seiche/engines/`](backend/seiche/engines/) each answer one narrow
question and are combined out in the open. If you are building your own
monitoring board on a different subject, that separation is a good starting
shape to copy.

---

Questions or a project that reuses a piece? Details are at
[seiche.info](https://seiche.info). A note back is welcome but never required.
