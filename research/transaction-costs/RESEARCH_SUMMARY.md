---
layout: single
title: "Transaction Costs"
permalink: /research/transaction-costs/
---
## The goal

For the trading strategies I'm building, I need to estimate the round-trip cost of a trade — the
spread cost. For heavily traded names (AAPL, TSLA, etc.) it's close to negligible, but for
lower-liquidity stocks (low ADV) it can be meaningful. Either way these are real costs, and
including them is an important part of development. Because of the gap between models/theory and
reality, there are unknowns and errors all over the place — so I wanted an approach tied to some
logic, and in some cases deliberately overestimating. Along the way I went down the rabbit hole
of the different methods people use for this.

My biggest constraint is the data I decided to use: **daily OHLCV**. The best method would be to
source actual bid-ask data at my entry and exit times, but with how far back my historical data
goes, that database would get out of hand — and data quality becomes its own potential issue I
didn't want to take on.

**Why not just use a ballpark estimate?** Personal decision. I wanted to learn about spread
models, and I wanted my estimate tied to some logic — a numerical approximation that responds to
liquidity and volatility. That said, I understand it's an estimator and I'm not stating it as the
truth. It's a doing-the-best-I-can-with-what-I-have situation.

---

## First attempt: Corwin–Schultz (CS)

I began with an apparently common method, Corwin–Schultz, which uses only the high and low over
pairs of consecutive days. Plotting its estimates across ADV buckets — the black line is a
ballpark approximation of what these stocks really cost to trade, which I use as the reality
check throughout:

![Corwin-Schultz claimed costs stay near 50-70 bp across every ADV bucket while approximate real cost falls from ~90 bp to ~2 bp](figures/01-corwin-schultz-fails.png)

It overestimates the spread cost in almost every bucket — and, ironically, *underestimates* in
the one place spreads really are wide (microcaps). It basically outputs the same answer
everywhere.

The reasons come down to how short-sighted the model is:

- It only takes in **2 days of data** per estimate, using only the high and low. So if a stock
  moves a lot in a day, the range is wide and the model reads that as spread — even if the stock
  is very liquid and the true spread is a penny.
- Within 2 days you can also get extreme **overnight moves**, and since the model assumes
  continuous trading, those jumps get counted into the spread estimate too.

So you could state it plainly: **CS is sensitive to volatility and does not respond to
liquidity.** (On my panel: correlation with volatility +0.58, correlation with liquidity
−0.002 — essentially zero.) I like the simplicity of it, but it's easy to see how it's
short-sighted and doesn't represent what's actually going on.

---

## Second attempt: EDGE

Next I considered the EDGE method. It takes in all four OHLC prices and looks at how price
movements behave *across* consecutive days, rather than reading a single range. Plotting it
against the same ADV buckets:

![EDGE tracks approximate real costs well at the illiquid end, then flattens into a noise floor above ~$20M ADV](figures/02-edge-resolution-limit.png)

It does clearly better at the lower ADV levels when compared against the real approximations. The
main reasons it works better:

- It treats overnight jumps and daily movement as **random noise to be averaged away** rather
  than counting them as spread. Genuine price movement is random day to day, so multiplied across
  days and averaged over a long sample, it cancels toward zero — the law of large numbers. The
  spread pushes the open and close the same way every day, so it survives the averaging.
- It uses a much larger sample: I calibrate it on pooled 21-day windows across thousands of
  stocks, not a rolling 2-day read.

**In summary:**

- **CS** uses minimal data over a micro-horizon (2 days) → volatile and overestimated.
- **EDGE** uses richer data over a long, pooled horizon → stable, filtered, and accurate at the
  illiquid end.

---

## But EDGE has a ceiling — the sample-size problem

As the visual above shows, the estimates for higher ADV buckets are still heavily impacted by
daily movement, and that contributes error. To smooth the error well below the spread being
estimated (~0.01% for a megacap), the number of samples needed gets extreme. How many is governed
by one ratio — the stock's daily volatility **σ** (how much it typically moves in a day) against
its spread **S** (the thing being estimated):

```
days needed  ≈  (σ/S)⁴
```

Averaging shrinks noise slowly (with the square root of the sample size), and the estimator works
in squared terms, which is where the brutal fourth power comes from. Worked through for three
real cases:

|          | spread S | daily vol σ | σ/S | days needed = (σ/S)⁴                         |
| -------- | -------- | ------------ | ---- | ---------------------------------------------- |
| microcap | 100 bp   | 200 bp       | 2    | 2⁴ =**16**                              |
| $25M ADV | 25 bp    | 190 bp       | 7.6  | 7.6⁴ ≈**3,300** (~13 years)            |
| megacap  | 1.5 bp   | 180 bp       | 120  | 120⁴ ≈**207 million** (~830,000 years) |

Volatility is roughly the same for every stock (~2% a day), but the spread collapses 100× as
liquidity rises — so the ratio explodes, and the fourth power turns it into a data requirement
that no amount of history can meet. I verified this rule with simulations (fake price data with a
spread I chose, checking whether EDGE recovers it): with 3,000 days of data it nails spreads down
to ~50 bp and fails below, exactly as predicted. Worse, below its limit it doesn't degrade
gracefully — it makes numbers up.

**So EDGE gets used only where the sample-size rule says it can work: below $10M ADV**, where my
last measured anchor lands at $7.08M.

---

## Bridging the gap to the liquid end

For the very liquid end (≥ $100M ADV), no daily-bar estimator can measure spreads that small —
but nobody needs one to. What these stocks cost to trade is directly observable market structure,
the same common knowledge that made CS's ~50 bp megacap claim obviously wrong on its face. So I
pin fixed anchors there: **20 / 8 / 4 / 2 bp at $100M / $500M / $2B / $10B ADV**.

That leaves the stretch between **$7M and $100M ADV**, where I have no trustworthy measurement on
either method. There I **interpolate** between the last EDGE anchor and the first fixed one
(log-linearly, which keeps the curve smooth and always decreasing). So yeah — interpolating
between the two seems lazy, but assuming liquidity is the main driver of cost falling from $7M to
$100M, it's the best estimate I can make without things getting out of hand. Not the strongest
logic in the model, but it's where I landed without spending a lot more time on it.

![The calibrated cost curve: EDGE-measured anchors below $10M ADV, fixed market-structure anchors above $100M, and an interpolated stretch between](figures/03-cost-curve.png)

---

## Putting it together: the final formula

```
vol_mult   = clip( σ_20 / σ_peers ,  0.75 ,  2.5 )

tick_floor = 10,000 × $0.01 / price           # one tick, expressed in bp

cost_bps   = max( base_spread(ADV) × vol_mult ,  tick_floor ) × 0.85
```

Reading it inside-out:

**`base_spread(ADV)`** — the stock's baseline, looked up on the calibrated curve above.

**`× vol_mult` — the volatility multiplier.** Volatile names genuinely trade wider, so the
baseline gets multiplied by how volatile the stock is *relative to peers of similar liquidity*:
σ_20 is its trailing 20-day volatility, σ_peers is the median among names in the same ADV decile
that day. A ratio of 1.0 means "typical for its tier." The ratio is **clipped**, not rescaled —
1.6 stays 1.6, but 4.0 becomes 2.5 and 0.5 becomes 0.75. The cap is the important part: it's what
stops volatility from taking over the estimate, which is exactly the failure mode CS showed.

![How much volatility can move the cost at each liquidity level: from 0.75x for calm names to the 2.5x cap for very volatile ones](figures/04-volatility-multiplier.png)

**`max( … , tick_floor)`** — a quoted spread can't be narrower than one tick ($0.01), so the cost
is floored at a penny expressed in bp of the stock's price: 1 bp for a $100 stock, 20 bp for a $5
stock. It only kicks in for cheap and/or very liquid names.

**`× 0.85` — the effective-spread haircut.** According to published market-microstructure
studies, the spread you actually pay (the *effective* spread) typically runs about **50–80% of
the quoted spread**, because real executions capture price improvement — midpoint fills, hidden
liquidity, resting limit orders. My entries are market-on-open auction fills, where none of that
improvement is available, so I deliberately set the ratio *above* the published range at
**0.85**. Overestimating on purpose, again.

---

## What the model produces

![What the model charges: median round-trip cost by ADV bucket, with the middle 50% shaded](figures/05-cost-by-liquidity.png)

Each dot is the median round-trip cost for that liquidity tier; the gray band is the middle 50%
within the tier, which mostly reflects the volatility multiplier spreading otherwise-similar
stocks apart.

**Two things I stay suspicious of:**

1. **The $7M–$100M stretch is interpolated, not measured** — and a majority of my trades at
   ADV ≥ $25M land in it. It can't be improved with this data; it would take broker execution
   reports or published effective-spread statistics.
2. **The EDGE anchors are probably biased upward.** My three anchors came out at
   80.4 / 75.1 / 70.4 bp across a 4.4× range of ADV — implausibly flat. Simulation shows the
   estimator develops a floor around ~40 bp for small true spreads, and compression toward that
   floor would produce exactly this pattern. Conservative direction (overcharging, not under),
   but worth remembering.

---

## Code Repository

[View on GitHub →](https://github.com/tenicho/transaction-costs)
