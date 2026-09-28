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

## Cold start

A property with no quote has no anchor, so the price level itself has to be estimated. Sixteen
parameters, scored leave-one-hotel-out over 122,331 hotel-currency units with the whole property
withheld:

| | log-MAE |
|---|---|
| currency alone | 0.6100 |
| geography and star backoff ladder | 0.4524 |
| sixteen attribute parameters | 0.2306 |
| plus chain price level at full cardinality | **0.1928** |

Median error 8.6%, 75% within 25%. The lever is the price level of other hotels in the same chain,
which accounts for 92% of the improvement over the attribute-only model. The nearest-neighbour
price topped the gain chart and removing it costs 0.0012.

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
