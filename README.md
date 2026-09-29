# Should you wait to book?

What the same hotel stay cost agents who booked it **later**. The stay is fixed and only the
booking date moves, which is the question no supplier will quote because the answer does not
exist yet.

**[Open the page](https://kushajm.github.io/plexe-price-delay/)**

## What it measures

83,871 pairs of the *same* stay, keyed on hotel, stay date, room rate identifier, board,
refundability, provider and currency at one room, observed at two different booking dates, across
11,532 hotels between 2025-01 and 2026-09.

For each delay it reports the distribution of `price_later / price_earlier`, the share that went
up, down or did not move, and the expected cost of waiting with a 95% interval bootstrapped over
hotels rather than pairs.

## What it found

**Waiting costs money, and the typical outcome is nothing.** The median change is 0.0% at every
horizon under a month, so a median-led reading says nothing happens. The mean runs +1.7% at a day
to +9.4% at four months, because the distribution is skewed: at a month a rise of 10% or more ran
27% against 17% for a saving of that size. Every horizon's interval sits above zero.

**Nothing beat assuming no change.** A fitted booking curve and a delay-only curve both scored
negative skill against carrying the price forward. A quantile model over 7 rolling origins and 3
seeds beat the shipped lookup at all 7 origins, but by 0.0033 against a pre-registered bar of
0.005 with 4 of 7 intervals clear of zero, so it does not ship and the page says so.

**Hotel identity adds nothing on this axis.** Held-out pinball loss for delay by hotel was 0.11099
against 0.11090 for delay alone. A property we have never seen is drawn from the same curve as a
familiar one and loses +0.26% for it, measured leave-one-hotel-out.

## Cold start, and a number this page has withdrawn

A property with no quote has no anchor, so the price level itself has to be estimated. **The page
now runs a real model for this**: 400 boosted trees over thirteen inputs, exported to plain arrays
and evaluated client-side. Pick a market, star band, chain, property type and facility count and
it prices the property.

This section previously reported **0.1928** log-MAE with a median error of 8.6%. That was wrong.

Every neighbourhood feature was built leave-one-out, as `(sum - own) / (count - 1)`. That passes
the obvious test, which is that the feature equals what you would have got had the property never
existed, and the original assertion checked exactly that and passed. It is still not safe:
conditional on the cell total the value is a strictly decreasing function of the property's own
price, so a learner that can estimate the total recovers the price, and the feature set offered
two ways to estimate it.

The test that catches it is to shuffle the target within currency, rebuild every feature against
the shuffle and refit. A clean feature set then scores the currency baseline, because nothing
predictable is left.

| | real target | shuffled target |
|---|---|---|
| leave-one-out features | 0.2306 | **0.3166** (baseline is 0.6100) |
| tables built without the fold | 0.3816 | 0.6189 |

The ladder baseline was built the same way, so it is re-derived too. The honest picture, over
122,331 hotel-currency units and 88,282 hotels, with every table recomputed from hotels outside
the fold:

| | log-MAE |
|---|---|
| currency alone | 0.6100 |
| geography and star cell lookup | 0.4833 |
| **the model this page runs** | **0.4052** |
| the same model plus the 136k-hotel point set | 0.3816 |

Median error 30.3%, 42% within 25% of the true rate, 72% within 50%. The model beats the cell
lookup by 0.078, about a sixth. It is a real gain and a modest one.

Every control earns its place. Sweeping one input end to end with the other four fixed moves the
estimate by, at the median over 300 configurations, 147% for chain, 123% for market, 107% for
star band, 94% for property type and 11% for facility count.

Two checks run before the model is served: the flattened forest reproduces LightGBM exactly on
300 rows, and a harness slices the functions out of the built page, re-prices 40 cases in Node
and re-prices them again through the booster in Python, agreeing to 2e-16.

## Honesty notes

- Nothing here is a forecast. The curve reports what comparable bookings did, and the gate that
  would license a prediction was not met.
- The bands are shares of past bookings, not model confidence intervals.
- A hotel is only listed when it can answer all three horizons on 12 or more of its own pairs per
  window, which is why the list is 81 and not 124. A tenth percentile over fewer than eleven
  points is just the minimum.
- 3,157 hotels in the source carry coordinates of exactly (0,0) and are excluded from every
  geographic feature rather than treated as a place.

Built by Plexe for ZenIQ.
