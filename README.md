[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/bisabisanya/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/bisabisanya/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Token Radar — model radar-2

A token launch radar: it reads a sealed DefiLlama protocol snapshot, finds every
protocol that records no token and is not a wrapped, staked or bridged product
of one that trades, ranks them by an implied value, and hands in a web dashboard
with the board, a detail view for each protocol, and the steps to farm it.

## Run

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

Node 22.18 or later, standard library only. No network at run time: the program
reads the snapshot file and writes everything under the output directory.

Outputs: `board.json` (the ranked board), `data.js` (the dashboard's dataset),
`dashboard/` (the static page), `screenshots/` (board, detail, filtered views),
`dossier.json` (built with the vendored make.fun library).

## Model radar-2

Implied value follows only from the snapshot's own figures:

```
implied = round( tvl_usd x 0.25
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + min(chains - 1, 5) x 0.04) )
```

- **Base**: TVL times 0.25 — what a launch would be worth if the protocol's
  locked value stayed as it is.
- **Growth**: the seven-day TVL change, clamped, half-weighted. An unreported
  change counts as zero.
- **Maturity**: how long the protocol has run with no token; an unknown age
  uses 365 days.
- **Breadth** (new in radar-2): one step per extra chain, up to five — a
  protocol live on several chains has more distribution surface for a launch.

Traction (TVL per day alive) is context in each detail view, never a ranking
input. Risk flags (falling TVL, hypergrowth, single chain, very young, missing
growth data, unknown age) are warnings from the same figures; they never change
a row's implied value. No protocol name is special-cased: the same functions
score every row.

## Coverage

The board is the whole rankable set: the 40 watchlist slugs plus every other
pre-token protocol the watchlist rule keeps (198 rows on this snapshot), not a
sample. Live-token protocols are set aside in their own view with their market
caps and TVL / market-cap ratios.

## Farm plans

Eight steps per protocol, derived from its category, chains, TVL and age:
a fresh wallet, the category's main action with a capital tier, a second
qualifying action, a weekly cadence, a hold window, a claim-safety check
(never pay to claim), a farm log, and a staged exit. Each step states its cost
and time.

## Dashboard

Static page, no framework, no network. Search, category / chain / growth
filters, a watchlist-only scope, sortable columns, a live-token view, a model
panel with a sandbox (change a constant and the board re-ranks live), and a
detail view per protocol with the model's full breakdown and its farm plan.
Deep-linkable via the URL hash (`#p=<slug>`, `#category=`, `#growth=` ...).

Implied value is this entry's own claim from the snapshot's figures. It is not
a prediction that a token will launch, when it will launch, or what it will be
worth.
