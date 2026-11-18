# radar-2

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/aqqaaqqa/radar-2/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/aqqaaqqa/radar-2/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

RADAR-2 is a token launch radar: it ranks 198 pre-token protocols by implied value and gives each one a concrete farm plan. It reads a sealed DefiLlama snapshot, covers the whole 40-protocol watchlist, and adds the 158 rankable pre-token protocols below the $50M floor — the early launches the watchlist leaves out. The dashboard is a static page that runs with no network: sortable and filterable board, a detail view per protocol with the model's own breakdown, a top-movers strip, and the calibration table the model is built from.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Then open `out/dashboard/index.html` in a browser. No network request is made while viewing it: the board's data is written beside the page as `data.js`.

## How the model works

Implied value follows only from the snapshot's own figures — no protocol name is ever special-cased:

```
implied = TVL x categoryMultiple x sizeFactor x growthMult x maturityMult
```

- **categoryMultiple** — the category's median market cap per dollar of TVL among the snapshot's own live-token protocols (market cap greater than zero, CEX excluded), shrunk toward the global median of 0.241× with k = 10, then clamped to [0.05, 0.75]. A category with no live comps falls back to the global median. The shrinkage is the point: one live comp cannot set a multiple.
- **sizeFactor** — `clamp((TVL / 1e9) ^ -0.15, 0.7, 1.6)`: the multiple fades as size grows.
- **growthMult** — `1 + 0.5 x clamp(change_7d, -40, +60) / 100`: seven-day growth, capped so a one-week spike cannot dominate the board.
- **maturityMult** — `0.55 + 0.45 x ln(1 + age_days) / ln(1 + 1095)`: log-age, so a protocol that has run three years with no token on record scores near 1.0. Unknown age counts as 365 days.

Wrapped, staked and bridged products of tokens that already trade are kept off the board entirely, as are exchange-named and versioned names: a wrapped product is not a launch. The board separates protocols whose token already trades from those still to come.

## Farm plans

Every protocol gets a 7-step plan from one of twelve category templates (Lending/CDP, Dexs, Yield, RWA, Risk Curators, Onchain Capital Allocator, Restaking, Perps/Derivatives, Stablecoin, Insurance, Chain, Payments), each step naming the protocol, its first chain, a capital tier from its own TVL ($250 at $5B+, $150 at $1B+, $100 at $100M+, $50 below) and a hold period from its own age (60 days under 180 days old, 45 under a year, 30 otherwise), with a cost and a time on every step. One step in every plan is safety: claim only from the link the protocol itself publishes, never pay to claim, and points are not a promised token. Where a step needs a human — some RWA mints require identity verification — the plan says so honestly.

## Structure

`main.ts` is the make.fun entrypoint: it reads the snapshot, scores the board, writes `board.json` and `data.js`, copies `dashboard/` and `screenshots/` to the output directory, and writes `dossier.json`. `lib/model.ts` holds the radar-2 model (`calibrateMultiples`, `categoryMultiple`, `impliedValue`, `isRankable`, `farmSteps`). `dashboard/` holds the static page (`index.html`, `app.css`, `app.js`). `vendor/makefun-board.ts` is the kit's vendored dossier library, copied byte-for-byte.

---

<sub>Entered on [make.fun](https://make.fun/?ref=mv94neex&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
