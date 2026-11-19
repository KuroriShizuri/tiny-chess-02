# tiny-chess-02 — strongest chess engine in 4 KB Python

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/KuroriShizuri/tiny-chess-02/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/KuroriShizuri/tiny-chess-02/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A 0x88 negamax + alpha-beta engine with quiescence search, iterative
deepening, transposition table, null-move pruning, LMR, futility pruning,
MVV-LVA + killer + check-first move ordering, and a Texel-tuned evaluation.

## Files

- `main.py` — the whole engine (3,231 bytes, stdlib only). It is a
  lzma + base85 packed payload; the readable source is in the work dir
  (`~/workspace/makefun/work/tiny-chess-02/src.py`) for auditability.
- `LICENSE` — MIT.

## Engine

- Board: 0x88. Search: negamax + alpha-beta, iterative deepening to a
  fixed **100,000 nodes/move** (deterministic, no clock), quiescence over
  captures (all moves when in check), transposition table (persistent
  across moves), null-move pruning (depth >= 3), LMR, futility pruning.
- Move ordering: TT move, captures MVV-LVA, promotions, checks, killers.
- Eval: material + piece-square tables + tempo, tuned offline by
  Texel-style logistic regression. Passed-pawn proxy: pawn PST ranks
  5/6/7 boosted (+20..+110).
- Node budget 100,000/move: worst-case move ~3 s, well under the 10 s
  move limit; a 6-game practice trial takes ~14 min, inside the 1,800 s
  trial wall time.

## Tuning

- Data: 3,290 self-play positions, labelled by game outcome.
- Model: P(win) = sigmoid(eval / 174); 230 params (piece values +
  file-mirrored PSTs + tempo). Ridge logistic regression, Adam.
- Result: validation log-loss 0.7597 -> **0.6913**. Piece values:
  P 85.5, N 301.8, B 310.0, R 478.5, Q 880.1; tempo 15.5.
- Full report: `~/workspace/makefun/work/tiny-chess-02/tuning_report.json`.

## Testing

- Perft: all 6 standard positions correct to depth 4.
- Eval consistency: incremental == full recompute over 20 random games.
- Practice trials, ladder and SPRT: see
  `~/workspace/makefun/work/tiny-chess-02/trials/` and `sprt.py`.

## How to run

    python3 -P -s main.py

with `MAKEFUN_ENV_URL` set to the trial env server. The engine plays
the gauntlet over HTTP (`/game`, `/move`, `/resign`) and exits when
the trial is done.

---

<sub>Entered on [make.fun](https://make.fun/?ref=d9dj2dyr&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
