---
layout: single
title: "Data Cleaning"
permalink: /research/data-cleaning/
---
Before any of the modelling was worth doing, I had to build a dataset I actually trusted. This is
what that took: 27.7 million daily bars of US equities from 2003 to 2026, and the five problems I
had to work through to get there.

Two of them I knew about going in. Two I found by accident. One I found late enough that it forced
me to re-read results I'd already accepted.

I'm writing this partly so I remember why the data is shaped the way it is, and partly because most
of these problems are invisible until they've already ruined a backtest.

---

## What's in it

|                           |                                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| Coverage                  | 2003-01-02 to 2026-06-24, 5,906 trading days                       |
| Rows                      | 27,674,679 daily bars                                              |
| Listings                  | 12,762                                                             |
| Clean operating companies | 7,892                                                              |
| Sources                   | Financial Modeling Prep for prices, SEC EDGAR for company identity |

![Listings trading per day](/assets/images/research/data-cleaning/01-listings-per-day.png)

That's the raw count before any filtering — every symbol the vendor returned a price for. Getting
from there to something tradable is most of the work below.

---

## Problem 1: dead companies

If you download "all US stocks" from a data vendor, you get the ones that still exist. Companies
that went bankrupt, got bought, or were kicked off the exchange are just gone.

Backtest on that and your strategy **never once buys a stock that went to zero**. It picks only from
companies that made it — which is information nobody had at the time. The effect is not subtle.
Failure is a real outcome, and if you delete it from the sample, everything looks better than it was.

The fix is to build the universe from **active plus delisted**, in that order, before anything else
runs. I pull the vendor's delisted list first and cache it, then combine it with the active list,
then pull prices for the whole thing. A dead company's history simply ends at its last trade.

Order matters here. Build the universe from the active list alone and every later stage inherits the
bias — no amount of cleaning gets it back.

![Survivorship: dead vs surviving listings](/assets/images/research/data-cleaning/02-survivorship.png)

6,222 listings stop trading before the panel ends — 48.8% of everything in it. The right panel is
the part I find most useful: names that died lasted a median of 2.8 years, names that survived 8.7.
That short-lived population is exactly what a biased dataset quietly removes.

### The catch, which I found late

Look at the left panel again. Almost nothing before 2015, then a wall of delistings from 2021.

That's not what markets do. 2008 and 2009 should be visible from here.

So I went and checked the vendor's delisted list directly:

| period     | companies in the list |
| ---------- | --------------------- |
| 2002–2015 | about 40              |
| 2016–2026 | about 9,100           |

The US had hundreds of delistings a year through the 2000s. The list basically starts in 2016.

I tested it a second way — of the names trading in a given year, how many are still trading at the
end of the panel? Roughly half of US listed companies disappear within a decade, so an old cohort
should show low survival:

| cohort | still trading in 2026 |
| ------ | --------------------- |
| 2004   | **71.5%**       |
| 2008   | 69.5%                 |
| 2012   | 67.3%                 |
| 2016   | 63.2%                 |
| 2022   | 56.1%                 |

71.5% survival over 22 years isn't believable. And the trend runs the wrong way — older cohorts
survive *more*, when they've had more time to die. That only happens if the deaths are missing.

Third check: the panel has 2,660 listings in 2004. The real market had roughly 5,000 operating
companies. I have about half of them.

**So survivorship is fixed from about 2016 onward, and not before.** I'd written "survivorship
clean" in my own notes for months before catching this.

One thing that isn't obvious: the direction of the bias depends on which universe you're in. Missing
*failures* makes returns look better. But among liquid names, most exits are *acquisitions*, and
those usually complete at a premium — missing those makes returns look worse. In small caps the
failure effect dominates and the bias is clearly optimistic. In a liquid universe the net sign is
genuinely unclear.

Closing this properly needs a delisting source with real historical depth. CRSP is the standard, and
this is exactly why.

---

## Problem 2: recycled ticker symbols

Ticker symbols get reused. A company delists, its symbol goes back in the pool, and years later
somebody else gets it.

Stitch those together naively and you have one price series that jumps between two unrelated
businesses. Any moving average, return, or forward label spanning the join is measuring nothing.

760 listings here are affected. The fix is a dormancy rule: more than 90 days with no trading and
what follows is treated as a different listing —

```
SYM      first company
SYM__1   second company
```

They're separate rows everywhere downstream, so nothing ever spans two companies.

The gotcha is that anything joined by ticker afterwards — splits, fundamentals, anything — has to
strip the suffix first. 12,762 listings, but only 12,064 distinct base symbols.

---

## Problem 3: things that aren't stocks

The vendor's "US equities" list contains a lot that isn't a company: ETFs, closed-end funds, ETNs,
SPACs, commodity trusts, warrants, units, preferred shares, exchange test tickers.

Leave them in and your stock-picking model is partly picking index funds.

I filter twice. First on symbol patterns, which catches the structural stuff — test tickers,
warrants, units, preferreds — while keeping legitimate class shares like BRK-A.

The second pass is the real one, and it uses SEC filings. Every ticker gets resolved to an SEC CIK,
and then the test is what that entity actually **files**:

```
operating company  if  files 10-K / 10-Q / 20-F / 40-F
                  and  files no N-series or 497/485 fund forms
                  and  SIC is not a fund, commodity ETP, or blank check
                  and  name isn't fund-like
                  and  ticker isn't on the vendor's ETF list
```

Five conditions looks paranoid until you hit the case that motivated it: commodity ETPs like GLD and
USO **do** file 10-Ks. The first test alone lets them straight through. It takes the SIC code, the
name check, and the ETF list to catch them.

Getting to a CIK at all is harder than it sounds, because the free ticker-to-CIK files only cover
*active* filers — and half my listings are dead. So it's a three-rung ladder: exact ticker match,
then exact company-name match against SEC's all-filers-ever file, then a fuzzy name match. Whatever
doesn't resolve stays unresolved rather than getting a guess.

| method               | listings        |
| -------------------- | --------------- |
| exact ticker         | 6,251           |
| exact name           | 4,363           |
| fuzzy name           | 73              |
| **unresolved** | **2,075** |

![Universe funnel](/assets/images/research/data-cleaning/03-universe-funnel.png)

12,762 listings down to 7,892 operating companies. Roughly 38% of what the vendor called US equities
is something I wouldn't want to trade as a stock.

The same EDGAR crawl gives me SIC codes, which map to sectors — that's the right panel, and it's a
free byproduct rather than a separate job.

---

## Problem 4: the adjusted price trap

This is the subtle one, and the one that actually changed my results.

Every vendor's "adjusted close" is **back-adjusted**. When a stock splits 2-for-1, the price halves
overnight — you didn't lose anything, you own twice as many shares, but a raw chart would show a 50%
crash that never happened. So vendors go back and rewrite all the earlier prices. Same for
dividends.

This is necessary. Without it your returns are wrong.

**But the rewriting uses the future.** To know what Apple's 2003 price should be adjusted to, you
need every split and dividend from 2003 until today. So the number stored for 2003 could not have
been known in 2003.

That's harmless when you take **differences**, because a return is a ratio and the adjustment factor
sits in both the top and the bottom:

|                             | Jan 2             | Jan 3  | return |
| --------------------------- | ----------------- | ------ | ------ |
| what Apple really traded at | $14.80 | $15.10   | +2.03% |        |
| adjusted (both ÷ 67)       | $0.2216 | $0.2261 | +2.03% |        |

Same answer. The 67 divides out.

It is **not** harmless when you use the price on its own, because every stock carries a different
future factor.

> Apple on 2 January 2003 reads **$0.22** in an adjusted series. It traded at **$14.80**.

Sort your universe by price and 2003 Apple sits down among the penny stocks. And ask *why* its number
is so small — because it split 67× afterwards. Splits happen to stocks that went up.

So "cheap" quietly becomes **"this stock was about to do well."** That's future information sitting
in a column that looks like an ordinary price.

![Price level distortion by year](/assets/images/research/data-cleaning/04-price-level-distortion.png)

The left panel is the measurement. 2003 rows are priced about 2× below reality, decaying to exactly
1.00 by 2021 — which is the giveaway, because a row at the end of the panel has no future left to be
adjusted for. Across the whole panel, 41.9% of rows differ by more than 10% between the two prices.

The right panel is the part that annoyed me most. I had a $5 minimum price filter, meant to keep
genuinely cheap stocks out. Run on adjusted prices, **it does the opposite of its job**: it let in
6.62% of rows that were really trading under $5, and threw out 4.63% that were really above it. It
excluded 2003 Apple for the crime of splitting later, while admitting real penny stocks whose
adjusted history got inflated by later reverse splits.

### Fixing it

Two distortions, and they come apart separately.

**Dividends were free.** The vendor returns *two* price fields and I'd been keeping the wrong one:

| field        | adjusted for         | AAPL 2003-01-02 |
| ------------ | -------------------- | --------------- |
| `close`    | splits only          | 0.26429         |
| `adjClose` | splits and dividends | 0.22155         |

Switching to the first removes the dividend half at zero cost.

**Splits needed one download.** 11,302 split events across 3,760 symbols. Then for any date, multiply
by every split ratio dated after it:

```
real price(t) = vendor close(t) × every split ratio after t
```

![AAPL: reconstructed price vs adjusted close](/assets/images/research/data-cleaning/05-aapl-nominal-vs-adjusted.png)

Apple is a good test because its history is well known and it has 56× of splits after 2003. The
reconstruction gives **$14.80** for 2 January 2003, which is right. The two lines converge at the
right edge exactly as they should — no future left to adjust for.

I checked it four ways before trusting it: against known prices (Apple $14.80, Microsoft $53.72),
against known corporate actions (five cumulative split factors, all exact), against structural
invariants that must hold by construction, and with a falsification test on the core assumption. The
build script refuses to write a file if that last one fails.

### The rule I now follow

| what you're doing                                               | which price              |
| --------------------------------------------------------------- | ------------------------ |
| returns, volatility, labels — anything that's a**ratio** | adjusted close           |
| price**levels** — filters, rankings, factors             | reconstructed real price |

Or: **differences are safe, levels are not.**

There's a sting in the tail here. Share volume is back-adjusted too — Apple's 2003 volume reads 182
million shares when 3.26 million actually traded. Nobody thinks of a share count as "adjusted," which
is exactly why I missed it for weeks after fixing the price. Same mechanism, same fix, and it's why
the rule above is written about *fields* and not about *prices*.

---

## Problem 5: data that's simply wrong

Price feeds contain impossible values. Prices of 9.75e15. An adjusted close 10,000× off the real one.
A split ratio of 1-for-15,464.

Left alone these manufacture enormous fake returns, so there are filters during the price pull —
non-positive prices, absurd magnitudes, listings with physically impossible daily moves, dates where
fewer than 50 names traded.

Then during the price reconstruction, two classes get **no reconstructed price at all**, which means
they fail any price filter and drop out of everything. That's deliberate: a missing price can never
accidentally become a traded position.

The threshold for "this vendor data contradicts itself" is 5%, and I set it by looking at the
distribution rather than picking a round number. Benign rounding noise sits under 1%; genuine
corruption jumps by factors of 100 to 10,000. 5% is the empty gap between the two populations.

I mention that because my first attempt used 0.1%, which quarantined 62% of the data **including
Apple**. The threshold was cutting into the healthy population, not the broken one. Worth measuring
before choosing.

About 4.4% of rows end up without a reconstructed price. That's the intended cost.

### The one I haven't solved

The corrupt-data filter drops listings with a single-day move beyond ±500%, or more than 20 days
beyond ±50%.

**Companies that actually collapse look exactly like that.**

So this filter may be removing real disasters along with real corruption — which is the survivorship
problem from Problem 1, sneaking back in through a side door. The price and liquidity filters exclude
most such names anyway, so the exposure is probably small, but I haven't measured it. It's on the
list.

---

## What you end up with

![Tradable universe](/assets/images/research/data-cleaning/06-tradable-universe.png)

Clean operating companies, above $5, at a few liquidity floors.

The thing worth noticing is the drift. A **fixed dollar** volume floor doesn't hold still — dollar
volumes inflate, so $25M a day admits about 340 names in 2004 and 1,660 by 2026. Nearly five times
as many. That's not the market getting five times bigger; it's the yardstick shrinking. Either report
it or switch to a percentile rank.

---

## What this dataset still can't do

**No shares outstanding**, anywhere. Which means **market cap cannot be computed at all**. Daily
volume and share price are the only size proxies available, and they're not the same thing.

**No fundamentals.** EDGAR is used here purely for company identity — no earnings, revenue, or book
value.

**No bid/ask quotes**, so trading costs have to be modelled rather than measured. That turned into
its own project.

**Survivorship is only clean from about 2016.** Repeating it because it's the one I'd most want a
reader to take away.

---

## What I'd tell someone starting this

The problems that cost me the most weren't the ones I planned for. Survivorship bias and junk
securities I knew about going in, and they were mostly a matter of doing the work.

The two that actually changed results were the ones where **the data was correct and I was using it
wrong**. Adjusted close isn't a bug — it's the right tool for returns and the wrong tool for levels,
and nothing warns you when you cross that line. Same with share volume. Same, probably, with things
I haven't found yet.

And the delisted-list gap is the one that stings, because I'd checked survivorship, satisfied myself
it was handled, and moved on. I only caught it because a chart looked wrong — not because a test
failed. There wasn't a test.

So the habit I'd keep: when a number looks fine, ask what it would look like if it were broken. If
you can't tell the two apart, you haven't checked it yet.

## Code Repository

[View on GitHub →](https://github.com/tenicho/data-cleaning)
