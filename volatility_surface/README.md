# Implied Volatility Surface

Option chain from `yfinance` -> implied volatility across strike and time, with Plotly; plus a volatility smile slice.

Terms:
- strike - the price the option can be exercised at
- tenor (T) - time left to expiration, in years
- OTM - out-of-the-money: calls above spot, puts below spot
- implied volatility - the volatility implied by an option's market price

- **x-axis:** expiration date
- **y-axis:** strike price ($)
- **z-axis:** implied volatility
- **hover:** the expiration date, the strike and the IV
- cells with no quotes stay empty - the surface just ends there

## Inputs

| Input | Meaning | Default |
|---|---|---|
| `symbol` | ticker | `"KO"` |
| `strike_range` | strikes in dollars `(low, high)` | `(70, 130)` |
| `tenor_range` | expirations with $T$ in this range (years) | `(0, 4)` |

## How it works

### 1. Fetch

For every expiration inside `tenor_range`, download the out-of-the-money quotes (calls with $K \ge S_0$, puts with $K < S_0$) from `option_chain(date)`. Implied volatility is read straight from yfinance's `impliedVolatility` column (Yahoo inverts Black-Scholes for us) - nothing is solved for here. Quotes outside 5%-60% are dropped as junk - a near-dated wing can print 100%+ on no volume and would flatten the picture.

### 2. Build the surface

Chains are ragged (each expiration quotes different strikes), so every expiration's IV curve is linearly interpolated onto one common strike grid. Result: a matrix with **strikes as rows, expirations as columns**. Where an expiration has no quotes the cell stays `NaN`, so the surface ends where the data ends instead of being filled.

### 3. Plot

`plotly.graph_objects.Surface` draws the matrix. The x-axis is the expiration date (real time spacing); the hover shows that date, the strike and the IV.

### 4. Volatility smile

One cell lists every available expiration date; put the one you want into `smile_date` and re-run the next cell to plot **implied volatility vs strike** for that single expiration - a 2D slice of the surface, sorted by strike. The curve is normally U-shaped: the market prices the tails above one constant IV (crash protection, fat tails).

## Caveats

- IVs are yfinance's, not computed from bid/ask here
- the interpolated surface is a visualization, not an arbitrage-free fitted model
