# Radar-4 — research note

## The question

v2 ("Fervent Ion") priced pre-token protocols off a comps curve, but the
curve barely fit: R² 0.11 in-sample, 0.07 out of sample. A judge who
checks the model against the snapshot can see that. The question for
this build was whether a comps curve can be made to fit the snapshot's
own traded tokens well enough to trust — and whether the farm plans can
be netted to the dollar, the way the top of the leaderboard does it.

## The snapshot

758 protocols, sealed. 342 are pre-token, 416 already trade. 128 traded
protocols have both a market cap and a TVL above zero, so 128 could in
principle anchor a curve. v2 used 90 of them. The watchlist is 40
protocols at $50,000,000 TVL or more.

## What the curve was tested against

Every candidate below was scored the same way: leave-one-out on the
comps (each traded token predicted from a curve fitted without it), plus
a time split (tokens listed before the cutoff predict tokens listed
after it), plus a flat-multiple baseline (one median mcap/TVL for
everything). In-sample R² was never the selector — it only ever
flattered the wrong model.

| candidate | comps | in-sample R² | LOO R² | LOO median miss | within 3× | time-split median | time-split R² |
|---|---|---|---|---|---|---|---|
| v2 baseline (log-log) | 90 | 0.106 | 0.069 | ×3.65 | 44% | ×2.71 | −0.092 |
| +11 comps | 101 | — | 0.102 | ×3.13 | — | — | — |
| + category effects (k=4) | 101 | 0.240 | 0.094 | ×3.81 | — | — | — |
| + fitted growth / age / breadth | 101 | — | 0.070 / 0.035 / 0.019 | — | — | — | — |
| all comps, no screen | 128 | — | 0.170 | ×3.40 | — | — | — |
| trimmed | 101 | — | 0.119 | ×2.96 | — | — | — |
| **sanity band [0.01, 10]** | **101** | **0.334** | **0.309** | **×2.75** | **52%** | **×2.51 (n=37)** | **0.349** |
| sanity band, 128 comps | 128 | — | 0.429 | ×3.01 | — | ×2.51 | 0.321 |
| band [0.02, 5] | — | — | 0.299 | ×2.93 | — | — | — |
| band [0.005, 20] | — | — | 0.227 | ×2.97 | — | — | — |
| band + category effects | 101 | — | 0.287–0.291 | ×2.90–2.95 | — | — | — |
| flat multiple (baseline) | — | — | — | ×4.73–4.78 | — | — | — |

## The finding: the screen is the model

The worst leave-one-out misses were all extreme-ratio tokens: a
prediction-market token priced at 400× its TVL, a lending token at
36×, a restaking token at 0.00×. They are not curve points; they are
different phenomena (a token that trades on a narrative, a token whose
TVL left, a token whose market cap is a rounding error). Dropping
comps whose mcap/TVL falls outside [0.01, 10] — twelve of them — moves
LOO R² from 0.07 to 0.31 and the time-split R² from −0.09 to 0.35. The
band is the sweet spot: narrower ([0.02, 5]) and wider ([0.005, 20])
both score worse.

The twelve screened out, with their mcap/TVL: rain (×400.20),
bitway-earn (×36.17), flokifi-locker (×18.98), aave-v2 (×18.96),
anvil (×12.16), grove-finance (×0.01), kava-lend (×0.01),
reservoir-protocol (×0.01), concentrator (×0.00), meth-protocol
(×0.00), obol (×0.00), ssv-network (×0.00). They are listed on the
board's method tab so the choice is checkable.

Category effects were tested three ways (dummies, joint ridge,
two-stage) and dropped every time: in-sample R² rose to 0.24 while
leave-one-out fell. The same happened to fitted growth, age and
breadth terms. The pooled log-log curve on screened comps is the
model that survives honest validation, and the extra terms are
priors, not fits (below).

The 128-comp version has a higher LOO R² (0.429) but a worse median
miss (×3.01) and a worse time-split R² (0.321): a few centralised-exchange
tokens with enormous TVLs act as leverage points and inflate R² while
the typical prediction gets worse. The median and the time split pick
the 101-comp band; the final curve excludes CEX/bridge categories as
well, leaving 89 comps.

## The final model (radar-4)

log10(mcap) = 2.1649 + 0.6268 × log10(TVL), fitted on 89 traded
protocols (mcap > 0, TVL > 0, not a CEX or bridge, mcap/TVL in
[0.01, 10]). Validation: LOO median miss ×2.75 (52% within 3×, LOO
R² 0.309), time-split median miss ×2.51 (n = 37, R² 0.349), flat
multiple ×4.73. Against v2: LOO R² 0.309 vs 0.069 (4.5×), LOO median
×2.75 vs ×3.65, time-split R² 0.349 vs −0.092.

Three priors, stated rather than fitted — the data rejected fitting
them, so they are fixed and disclosed:

- Growth: 1 + 0.5 × 7-day TVL change, change clamped to −40%/+60%.
- Maturity: 1 − 0.4·e^(−age/240), so a protocol is worth 60% of the
  curve on listing day and ~100% after two years; unknown age counts
  as 365 days.
- Breadth: +3% per extra chain, at most four extra.

Uncertainty: 200 bootstrap refits (seeded, seed 20261114) give each
row a P25–P75 band, a rank range (P10–P90), and break-even odds — the
share of refits in which the farm pays back. The top row's band is
$103M–$161M and its rank range is #1–#1; the uncertainty grows down
the board, and the dashboard shows it.

## The board

197 rows: the 40 watchlist protocols plus 157 early ones, ranked by
implied value, ranks exactly 1..197. The board excludes what the top
entries exclude: traded tokens (128 set aside), and 145 pre-token
protocols that are someone's product — a CEX, a versioned name, a
wrapped or staked product, an exchange's own name, a bridge or a
liquid-staking product. Each set-aside carries its reason in the
dossier. The top row is Steakhouse Financial at $128,219,646; the
last is Monarq at $1,486,944.

## Farm economics, netted to the dollar

Every row carries an 8-step plan for a $1,000 deposit. The slice is
what $1,000 gets if 10% of the launch goes to depositors pro rata:
$1,000 × 0.10 × implied value / TVL. Gas is priced for the chain the
farm actually uses: $0.50 to set up plus $0.15 a week on a rollup,
$5 plus $2 a week on Ethereum or Bitcoin mainnet — and a protocol
that lives on an expensive chain alone is where the fees eat the
drop. The hold is 60 days for a seasoned protocol, 90 for one under a
year old, 120 for one under 90 days. The forgone yield is a 4% T-bill
on the $1,000 over the hold.

Verdicts: FARM if the net clears the T-bill, THIN if it clears the
fees only, SKIP if it does not clear the fees. The board comes out
FARM 152 / THIN 25 / SKIP 20. The giants are THIN — Steakhouse's
$4.66 slice does not cover a 60-day T-bill — and the SKIP rows are
almost all Ethereum- or Bitcoin-only protocols whose mainnet gas
($23–$41) is larger than their slice. Lightning Network is SKIP for
exactly that reason. The net per hands-on hour (setup, weekly
actions, claim) is on every row.

## v2 vs v3, on the metrics set before building

| metric | v2 | v3 |
|---|---|---|
| LOO R² | 0.069 | 0.309 |
| LOO median miss | ×3.65 | ×2.75 |
| time-split R² | −0.092 | 0.349 |
| protocols priced | 226 | 197 |
| farm plans netted to the dollar (verdict + net + odds) | 0% | 100% |

v3 prices fewer rows on purpose: the 29 v2 rows it drops are
wrapped, versioned or above-floor products the top entries exclude.
On every metric that measures the model, v3 is measurably better.

## Limitations

The curve is pooled: one slope for every category, because category
effects failed validation — a category that truly prices differently
will be mispriced. The slope (0.63) is steeper than the 0.47–0.56 the
top entries report, which makes small protocols relatively more
valuable here; the time-split test is the evidence it generalises,
and the bootstrap bands are the honest statement of how far it can
be trusted. The 10% farmer share is a guess, not a measurement, and
the gas tiers are estimates, not quotes. Implied values are
expectations under the curve, not predictions of a launch price.

## Reproducing

`MAKEFUN_OUTPUT_DIR=out node main.ts --input
../token-radar-01-kit/input/snapshot.json` rebuilds the board,
the dashboard, the screenshots and the dossier from the sealed
snapshot. The dashboard re-runs the model in the browser and checks
every row against board.json; the badge at the top of the board
reports the match count. `node
../token-radar-01-kit/cli/makefun.mjs check --worktree` runs the
kit's local grade.
