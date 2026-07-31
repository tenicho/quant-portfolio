---
layout: single
title: "Transaction Costs"
permalink: /research/transaction-costs/
---
Every backtest has to charge something for trading. Get that number wrong and you're not testing a
strategy, you're testing an assumption.

I found this out the hard way. For five iterations of this project I was using a published,
well-cited estimator to charge trading costs, and it was overcharging by roughly 5×. The strategy
looked uneconomic. It wasn't — the cost model was broken. Fixing it recovered about 6% a year on
every result I'd produced up to that point.

This is what I use now, why the obvious approaches don't work, and where it's still weak.

---

## The problem

I have daily open, high, low, close and volume. I don't have bid/ask quotes.

The cost of trading is mostly the **spread** — the gap between what buyers offer and what sellers
ask. You buy at the ask and sell at the bid, so you lose the difference. On Apple that's about a
penny on a $135 stock. On a $4 microcap it might be four cents, which is 150× worse in percentage
terms.

So the number I need isn't in my data, and it has to be inferred.

---

## Why the standard approach fails

There's a family of estimators that infers the spread from **bar geometry**. The logic is: a day's
high-low range is wider than the stock's true movement, because the high is probably a buy printing
at the ask and the low is probably a sell printing at the bid. Measure that extra width and you've
measured the spread.

It's a clever idea and it falls apart the moment the stock actually moves.

Apple, 15 June 2022:

|                           |                                 |
| ------------------------- | ------------------------------- |
| high minus low that day   | **$5.08** — a 3.8% swing |
| the actual bid-ask spread | **about 1 cent**          |

The spread is 0.2% of the range. Anything trying to read it out of that bar is looking for a penny
inside five dollars, and what it finds instead is volatility.

**That matters more for me than for most people**, because my models select volatile stocks. An
estimator that confuses volatility with spread will systematically overcharge exactly the trades I
make.

### Corwin-Schultz

This is the one I'd been using. It compares one-day high-low ranges against two-day ranges, on the
logic that volatility grows with time while the spread doesn't, so the difference isolates the
spread.

![Corwin-Schultz claimed cost by liquidity](/assets/images/research/transaction-costs/01-corwin-schultz-fails.png)

The red bars are what it claims. The dashed line is roughly what these stocks really cost to trade.

It says a stock trading **a billion dollars a day** costs **52 bp** to round-trip. The real figure
is 1-2 bp. And it barely moves across a 500× span of liquidity — 73 bp for microcaps, 52 for
megacaps.

I ran the correlations to be sure:

|                | correlation with liquidity | correlation with volatility |
| -------------- | -------------------------- | --------------------------- |
| Corwin-Schultz | **−0.002**          | **+0.580**            |

Its relationship with liquidity is **zero**. It's a volatility estimator wearing a spread
estimator's name. Once you partial out volatility, the entire liquidity spectrum separates by 18 bp.

That's what was charging my book 113 bp a round-trip where about 20 bp was real.

### EDGE

There's a newer estimator — Ardia, Guidotti & Kroencke (2024) — that's genuinely better. It uses all
four prices instead of just the high and low, and critically it compares them **across consecutive
days**, which is what lets it separate spread from movement. Its correlation with liquidity is
**−0.420** against Corwin-Schultz's −0.002. Real signal.

But it only works in part of the market.

![EDGE resolution limit](/assets/images/research/transaction-costs/02-edge-resolution-limit.png)

The blue bars are EDGE's estimate. The green line is the thing worth watching: **inside each
liquidity bucket, is the estimator still responding to liquidity at all?**

Below $5M a day, yes — correlation −0.40. Above that it crosses to zero and stays there. Past that
point EDGE isn't measuring a spread any more; it's reporting about 30% of daily volatility, which is
a noise floor.

### Why there's a cutoff at all

I worked out the arithmetic, and it's simple enough to be worth stating.

The estimator finds the spread by **averaging away the volatility**. The signal is the size of the
spread squared; the noise is volatility squared. Averaging N days shrinks noise by √N. So you can
see the spread when:

```
N  >  (volatility ÷ spread)⁴
```

A **fourth power**. Double that ratio and you need 16× the data.

|          | spread | daily vol | ratio | days needed           |
| -------- | ------ | --------- | ----- | --------------------- |
| microcap | 100 bp | 200 bp    | 2     | **16**          |
| mid-cap  | 25 bp  | 190 bp    | 7.6   | **3,300**       |
| megacap  | 1.5 bp | 180 bp    | 120   | **207 million** |

Nothing about the estimator changes across those rows. What changes is whether you could ever have
enough data.

I checked it by simulation — generate fake prices containing a spread I chose, hand EDGE only the
four bar prices, see if it finds the answer. It recovered a 300 bp spread to within 1%, a 100 bp
spread to within 1%, and when I gave it a 1 bp spread it answered 13 bp.

The version of that test I found most useful: hold the true spread **fixed** at 100 bp and only
raise volatility. At 1% daily vol it says 103 bp. At 15% daily vol it says **141 bp**. The spread
never changed. That's the overcharging-volatile-names problem, isolated.

---

## What I do instead

Use the estimator where it demonstrably works, and pin the rest to market structure I can observe
directly.

![The calibrated cost curve](/assets/images/research/transaction-costs/03-cost-curve.png)

Green dots are anchored by EDGE, each one the median of about 2 million estimates from stocks below
$10M a day. Black squares are fixed from observable market structure — I don't need an estimator to
tell me a megacap trades at 1-2 bp, I can read that off a quote screen. Between the anchors it's
log-linear interpolation, which gives a smooth curve instead of tier cliffs.

Then three adjustments on top.

**A volatility multiplier**, bounded between 0.75× and 2.5×. Volatile stocks genuinely do trade
wider, so the effect is real and worth keeping. Bounding it is what stops volatility from taking
over the estimate the way it does in Corwin-Schultz — the liquidity curve stays in charge.

![Volatility multiplier range](/assets/images/research/transaction-costs/04-volatility-multiplier.png)

**A tick floor.** A quoted spread can't be narrower than one cent, so the floor is one cent as a
percentage of price. This has to use the price the stock **actually traded at** — on a back-adjusted
series, 2003 Apple reads $0.22 instead of $14.80 and would get a 454 bp floor instead of 6.8 bp.
That's a whole separate problem I wrote up under [data cleaning](/research/data-cleaning/).

**An effective-spread haircut of 0.85.** You don't always pay the full quoted spread — brokers fill
inside the quote, there's hidden liquidity, limit orders can earn the spread instead of paying it.
Published effective-to-quoted ratios run around 0.5-0.8.

I use 0.85, deliberately above that range, because **I enter at the opening auction**. An auction
has one clearing price. There's no resting quote to be improved against, no midpoint peg, no passive
option. Most of the mechanisms that justify a discount don't operate there.

That single number multiplies every cost in the model, which makes it the most consequential
assumption in the whole thing.

---

## What it charges

![Cost by liquidity](/assets/images/research/transaction-costs/05-cost-by-liquidity.png)

| average daily volume | median round-trip |
| -------------------- | ----------------- |
| under $5M            | 69 bp             |
| $5-20M               | 50 bp             |
| $20-50M              | 30 bp             |
| $50-100M             | 20 bp             |
| $100-200M            | 14 bp             |
| $200-500M            | 9 bp              |
| over $500M           | 5 bp              |

One thing that's easy to overlook: **the holding period matters more than the cost itself**. A 20-day
hold turns over 12.6 times a year, so 25 bp becomes about 3.1% a year. A 5-day hold turns 50 times,
and the same 25 bp becomes 12.5% a year. Short holding periods are where cost assumptions decide
whether a strategy exists.

---

## Shorting is a different animal

If you short, you're borrowing the shares and paying a fee for every day the loan is open. That fee
is set by supply and demand in the stock-loan market — which is exactly the information a price
panel doesn't contain.

So borrow can't be estimated. It's a parameter, and the honest thing is to sweep it and show the
range.

![Borrow sensitivity](/assets/images/research/transaction-costs/06-borrow-sensitivity.png)

Two things this makes obvious.

**It usually dominates.** At 8% a year on a dollar-neutral book, borrow costs 400 bp a year — more
than all the spread costs combined at a 20-day holding period.

**The exposure scales with how much you hold short**, which is the slope of those two lines. Going
from 8% to 15% costs a dollar-neutral book 350 bp a year and an 80/20 net-long book 140. So a tilted
book depends far less on a number nobody can verify. That's an argument from robustness rather than
performance, and I find it more persuasive than a backtest comparison.

There's also a nasty asymmetry: the short leg of a ranking model picks its **worst-scored** names,
which skew distressed and crowded — exactly the population where borrow is most expensive and most
volatile. A flat assumed rate is least trustworthy precisely where it's being applied.

And several real frictions aren't modelled at all: **locate availability** (you can't always short
what you want), **recall risk** (the lender can force you to buy back at the worst moment), and
**Rule 201** (after a 10% intraday fall, shorting gets restricted to at-or-above the bid for the rest
of that day and the next — which is exactly when a reversal model wants in). Every one of those hurts.
The only omission that helps is interest earned on short proceeds, which was negligible at zero rates
and matters again from 2022.

So I treat any long-short backtest as an upper bound.

---

## Where this is still weak

**There's a gap between $7M and $100M a day with no anchor in it.** EDGE stops being trustworthy at
$10M; my first fixed anchor is at $100M. Everything between is interpolation — and for a strategy
trading above $25M a day, that's roughly two-thirds of the trades.

Worse, **I can't close it with this data.** Spreads in that range are 20-50 bp, and the fourth-power
rule says a 21-day window resolves nothing below about 93 bp. A full year of data only gets you to
53 bp. That's not a calibration choice I made, it's a hard limit. Closing it needs real execution
data — broker TCA reports, or published effective-spread statistics by liquidity bucket.

**The illiquid anchors are probably too high.** They came out at 80, 75 and 70 bp across a 4.4× range
of liquidity, which is implausibly flat — real spreads should vary more than 12% over that span. When
I simulated the actual calibration procedure, pooling stops helping below about a 50 bp true spread
and the estimator develops a floor around 36-41 bp. If the true values are more like 100, 70 and 45,
the estimator would compress them upward into exactly the pattern I'm seeing. Conservative direction,
but wrong.

**The 0.85 haircut is a judgment call**, and it scales everything.

**No market impact.** Fine for small orders, wrong for size. Though the binding constraint is usually
the entry auction rather than order size — at 10% of a 2% opening auction you can only take 0.2% of a
stock's daily volume, which is about 5× tighter than the rule of thumb people use.

---

## What I'd tell someone starting this

**Sanity-check any estimator on a stock you know.** Run it on a megacap. If it says 50 bp, it's
measuring volatility. That one check would have saved me five iterations.

**Never report a single cost number.** Report a ladder of assumptions. If a conclusion flips across
the range, you've found an assumption, not a finding. This is the rule I'd keep above all the others,
and it's the reason I caught the original problem at all.

**Watch the holding period.** It multiplies everything.

**Be suspicious of a change that helps.** When I first rebuilt this, costs came out lower in the range
I actually trade. That's exactly the kind of result to check hardest, because it's the kind you want
to believe.

The general shape of the mistake I made is worth naming: I took a published, well-cited tool and used
it for a job it wasn't built for. Corwin-Schultz is a fine instrument for what it's designed to do —
comparing liquidity across stocks, in markets where no quote data exists at all. It's just not a cost
model. Nobody wrote that on the label, and I didn't check.

## Code Repository

[View on GitHub →](https://github.com/tenicho/transaction-costs)
