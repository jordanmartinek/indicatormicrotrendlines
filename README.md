# Brooks Microtrendline Engine (in development)

File: [`BrooksMicrotrendlineEngine.pine`](./BrooksMicrotrendlineEngine.pine) — the
professional-grade NQ price-action tool, built in phases.

| Phase | Scope | Status |
|---|---|---|
| 1 · Core engine | Microtrendline detection (pivot + microchannel), touches/age/slope/quality, break → failed-breakout state machine, signal-bar quality, LONG/SHORT markers | **This branch — needs TradingView compile check** |
| 2 · Price action intelligence | H1/H2/L1/L2, pullback continuation, failed-breakout reversal, market classification (trend/range/transition) | Next |
| 3 · Confluence | 15m/5m bias via `request.security` (non-repainting), VWAP, 0–100 confluence score | — |
| 4 · Trade management | Pending stop entry, stop, 1R/2R/structure targets, triggered/stopped/target/invalidated states | — |
| 5 · Dashboard & alerts | Full dashboard with score explanation, all 16 alerts | — |
| 6 · Historical validation | Signal stats, expectancy, segmentation, conservative same-bar stop/target handling | — |

### Phase 1 — how the engine works

- **Two detection paths per direction**
  - **PIV** — two adjacent confirmed swing lows (bull, higher low) or highs (bear, lower
    high), 2–10 bars apart, with slope between min/max ATR-per-bar. The line has to be
    **clean**: no close beyond it and no wick deeper than the penetration tolerance.
  - **CHN** — a separate **microchannel** detector for strong pushes that never form pivots:
    N consecutive bars of higher lows / lower highs, mostly trend bars, with a minimum ATR
    move. Every bar of the run sits on the correct side of the line.
- **State machine**: `ACTIVE → BREAK?` (a close beyond the line) `→` either **reclaimed
  within N bars = FAILED BREAKOUT**, or **BROKEN** (no reclaim in time, or price runs too far
  past the line). A wick through the line that closes back inside is also a failed breakout.
- **Signal** = failed breakout + quality signal bar (body, close position, opposing wick,
  relative size) + enough touches (or a CHN line) + EMA-slope context. A signal is a
  **pending stop entry** one tick beyond the signal bar. Tracking whether it actually
  triggers comes in Phase 4.
- **Non-repainting**: every state change and drawing happens only on `barstate.isconfirmed`
  bars. Pivots are only used after `ta.pivot*` has confirmed them.
- **Markers**: yellow ✕ = break attempt, orange ◆ = confirmed break, aqua/fuchsia **FB** =
  failed breakout, green **LONG** / red **SHORT** = signal (hover a label for entry, stop,
  and signal-bar quality).
- The small debug table (top-right) shows each line's state, source, touches, slope, quality,
  and failed-breakout count. The Phase 5 dashboard replaces it.

### Phase 1 test checklist (on TradingView, NQ 1-minute)

1. The script compiles with no errors. Warnings about calling functions inside local scope
   are expected and harmless.
2. Steady pushes with no pivots get a **CHN** line. Pivot-based pushes get a **PIV** line.
3. A close beyond the line followed by a quick reclaim prints **FB**. A strong bull/bear
   signal bar after it prints **LONG/SHORT**.
4. A real break prints ◆ and the line turns dotted grey but stays on the chart.
5. Let the replay bar advance: markers on closed bars must not move or disappear.

---

# (Legacy) Al Brooks Micro Trendlines & Trend Channels

A TradingView **Pine Script v6** indicator that automatically draws Al Brooks–style
**micro trendlines** and **trend channels** in a bull or bear trend, freezes each line
when price breaks it, and signals when successive lines are **getting shallower** —
the Brooks tell that counter-trend pushes are weakening and a bigger move may be near.

File: [`MicroTrendLines.pine`](./MicroTrendLines.pine)

## The idea

In a trend, price moves in pushes. You draw a micro trendline on the *trend side*:

- **Bull trend** → a line along the pullback **lows** (the higher lows).
- **Bear trend** → a line along the pullback **highs** (the lower highs).

As a trend matures, each new line tends to be **flatter than the last**. Each flattening
step means the counter-trend traders are getting trapped. When a shallow line finally
**breaks**, those trapped traders are flushed — which is the fuel for a reversal or a
strong continuation. This indicator makes that whole sequence visible and alertable.

## What it does

1. **Push detection (consecutive candles)** — a bear micro line starts after the last
   **2–3 candles** make *lower highs*, and rides those stepping highs; a bull micro line
   starts after 2–3 candles make *higher lows* and rides those stepping lows. It reads the
   candles directly, so it draws on a **steady one-way push** (e.g. a grind down off the
   20 EMA) where no local pivot ever forms — the case the earlier pivot-based version
   missed. The EMA is an **optional** confirmation filter (off by default).
2. **Micro trendlines** — the line is anchored at the first candle of the run and
   re-pivoted onto each new lower high / higher low, so it always touches the extremes of
   the push and spans the whole move.
3. **Break handling** — when price closes through (or wicks through) the active line, the
   line **stops extending and is recolored/dotted**, but it stays on the chart so you can
   read the full progression of pushes.
4. **Trend channel** — a parallel dashed line projected from the furthest opposite extreme
   (high in a bull leg, low in a bear leg), reset for each new micro line.
5. **Flattening signal** — tracks each line's |slope|. When a configurable number of
   successive lines are each shallower than the previous one and then a line breaks, it
   prints a `⚠` label + triangle and fires an **alert**: *look for a bigger move*.

## Inputs

| Group | Input | Meaning |
|---|---|---|
| Trend Detection | **Trend EMA length** | EMA used for the optional slope filter. |
| | **Require EMA slope agreement** | **Off by default.** When on, only draw bear lines while the EMA slopes down and bull lines while it slopes up. |
| Micro Trendlines | **Candles to start a line (2 or 3)** | How many consecutive stepping candles begin a line. `2` is the most sensitive. |
| | **Allow equal highs/lows as a step** | Treat an equal high/low as a valid step (helps in fast pushes). Off = strictly lower-high / higher-low. |
| | **Min bars a line must survive** | Ignore 1-bar noise: a line must last this many bars before a break counts toward the flattening sequence. |
| Micro Trendlines | **Pivots per micro line (2 or 3)** | Fit each line through the last 2 or 3 pivots. |
| | **Extend active line forward** | Project the live line to the right. |
| | **Break confirmation** | `close` = break on a close beyond the line; `wick` = any penetration. |
| Trend Channel | **Show parallel channel line** | Toggle the channel line. |
| Flattening Signal | **Consecutive shallower lines to flag** | How many flatter lines before a break is a signal. |
| | **Flattening tolerance** | A line is "shallower" if `|slope| < |prevSlope| * (1 - tol)`. `0` = strictly flatter. |
| Colors | — | Bull line, bear line, broken/frozen line, channel, signal. |

## Alerts

Three `alertcondition`s are exposed in TradingView's **Add alert** dialog:

- **Bull micro trendline broken**
- **Bear micro trendline broken**
- **Flattening flush (bigger-move)** — the one you'll usually want.

## How to install

1. Open TradingView → **Pine Editor**.
2. Paste the contents of `MicroTrendLines.pine`.
3. **Add to chart**, then open settings to tune the inputs for your instrument/timeframe.

## Reading it on the chart

- Watch the **sequence of lines**: steep → less steep → flat is the classic Brooks setup.
- A **frozen (dotted/gray)** line marks where a push's line was broken.
- A `⚠` label after a run of flattening lines is the cue to hunt the larger entry
  (reversal against the trend, or a strong breakout continuation).

## Tuning tips

- **Candles to start a line** is the main knob. `2` draws on nearly every push (most
  sensitive); `3` waits for a cleaner three-candle run and draws fewer, more significant
  lines.
- Keep **Allow equal highs/lows** on for fast pushes where a bar matches the prior
  high/low exactly; turn it off if you want only strictly stepping candles.
- Leave **Require EMA slope agreement** off to draw lines on every push (including the
  first leg off a low/high). Turn it on only to filter to EMA-aligned pushes.
- Raise **Min bars a line must survive** to stop single-bar pokes from counting as breaks
  in the flattening sequence.
- Use **`close`** break confirmation to cut noise; **`wick`** to catch breaks earlier.
- Raise **Consecutive shallower lines to flag** on noisy, lower timeframes.

## Notes & limitations

- Pine redraws on the latest (real-time) bar; pivots confirm `Pivot strength` bars late by
  definition, so lines/anchors are placed on the confirmed pivot bar, not the live bar.
- This is a **discretionary aid**, not a mechanical system — it visualizes the Brooks
  structure and flags the flattening, but entries/stops are your call.
