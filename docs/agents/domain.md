# Domain Docs

How all AI agents and skills must consume this repository's domain documentation.

## Before Exploring or Writing Code, Read These

1. **`GLOSSARY.md`** at the repo root:
   - Contains quantitative definitions, domain language, and formula specifications.
   - Use the glossary's vocabulary strictly: avoid synonyms the glossary avoids (e.g. use "Historical Sample Positive Frequency", never "Win Rate").
2. **`BRD.md`** at the repo root:
   - Business requirements, quantitative engine expectations, SEBI guardrails (§8.2D), and UI constraints.
3. **`specs/changes/`** (Architecture & Change Records):
   - Historical decision records explaining previous technical choices, ceilings, and trade-offs.

If any of these files do not yet exist, proceed without failing, but never invent conflicting domain jargon. Use `/domain-modeling` or `/grill-with-docs` to formalize terminology when new concepts are defined.

## File Structure

Single-context repository layout:

```
/
├── GLOSSARY.md                  ← Quantitative definitions & safe-harbor terms
├── BRD.md                        ← Business requirements & SEBI compliance rules
├── specs/
│   ├── requirements/            ← Gate 1: Monotonic specs (0001-*.md)
│   ├── tests/                   ← Gate 2: Frozen test specs (0001-*.md)
│   └── changes/                 ← Gate 4: Change & decision records
└── src/                         ← Implementation (pure functions, minimal classes)
```

## Quantitative Triumvirate Audit Standard

Every feature, model, and decision recorded in `specs/requirements/` and `specs/changes/` must pass:
1. **Dr. Ernest P. Chan Lens:** Real-world transaction costs (STT, exchange fees, GST, stamp duty), bid-ask half-spread crossing, stationary features, and fractional Kelly / volatility parity sizing.
2. **Euan Sinclair Lens:** True mathematical expectancy, negative skewness / tail risk rejection, dynamic ATR-scaled volatility barriers, and order-flow microstructure (Volume Delta).
3. **Dr. Marcos López de Prado Lens:** Zero lookahead leakage, Purged & Embargoed Walk-Forward CV, path-dependent Triple Barrier Method ($y \in \{+1, 0, -1\}$), shallow regularized trees (depth 3-5), Deflated Sharpe Ratio (DSR), and Graft structural verification.

## Anti-Bloat & Simplicity Directives

- **Kailash Nadh Minimalism:** Flat files, Parquet, SQLite, vanilla Python/HTML/JS over bloated dependencies and heavy frameworks.
- **Strict Frontend/Backend Isolation:** Backend Python never renders HTML/CSS; Frontend static UI communicates exclusively via frozen JSON contracts under `data/static/api/vX/`.
- **Flag Conflicts Early:** If a proposed change contradicts `BRD.md`, `GLOSSARY.md`, or the Quantitative Triumvirate standard, pause and surface it immediately to the user.
