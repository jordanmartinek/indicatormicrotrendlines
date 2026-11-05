# Al Brooks Micro Trendlines & Trend Channels

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
