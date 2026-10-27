# European Option Pricing - 3 Notebooks

Three Jupyter notebooks that price **European call and put options** with real market data
(fetched from Yahoo Finance with `yfinance`). Every notebook explains the theory and the
formula in simple words right before the code that uses it.

---

## The 3 notebooks

### 1. `black scholes option pricing.ipynb` - the basic model
- Prices European options on the **S&P 500 index (^SPX)** with the classic
  **Black-Scholes model** (no dividends).
- Gets the stock price, risk-free rate and 1 year of price history from Yahoo Finance,
  calculates historical volatility, prints call and put prices, and solves for
  **implied volatility**.
- Key formula:

```
d1 = [ln(S/K) + (r + σ²/2)·T] / (σ·√T)     d2 = d1 - σ·√T

Call = S·N(d1) - K·e^(-rT)·N(d2)
Put  = K·e^(-rT)·N(-d2) - S·N(-d1)
```

### 2. `black scholes merton option pricing (dividends).ipynb` - continuous dividends
- Same idea, but for a stock that pays dividends: **KO (Coca-Cola)**.
- Dividends are smoothed into one constant number: the **continuous dividend
  yield q** (calculated from KO's real dividend payments of the last 12 months:

```
q = (sum of dividends paid in last 12 months) / current price
```
- The formula only changes in two places: `r` becomes `r - q`, and `S` becomes `S·e^(-qT)`:

```
d1 = [ln(S/K) + (r - q + σ²/2)·T] / (σ·√T)     d2 = d1 - σ·√T

Call = S·e^(-qT)·N(d1) - K·e^(-rT)·N(d2)
Put  = K·e^(-rT)·N(-d2) - S·e^(-qT)·N(-d1)
```
- Also checks **put-call parity**: `C - P = S·e^(-qT) - K·e^(-rT)`.

### 3. `black scholes discrete dividends option pricing.ipynb` - one dividend at a time
- Real dividends are paid as **fixed cash amounts on fixed dates** (e.g. KO pays
  ~$0.46 every quarter). On the ex-date the stock price jumps **down** by that amount,
  so plain Black-Scholes no longer works exactly.
- The notebook builds the dividend schedule from KO's payment history (projected
  forward until the option expires), then prices the option in **two ways**
  with one switch:

```python
dividend_model = 'escrowed'   # or 'forward'
```

| Model | Simple meaning | Adjustment |
|---|---|---|
| `'escrowed'` | take the "cash waiting to be paid out" out of today's price | `S* = S - PV(D)` |
| `'forward'` | the buyer also collects the dividends, so the effective strike is higher | `K* = K + FV(D)` |

where

```
PV(D) = Σ D_i · e^(-r·t_i)        (value today of each dividend)
FV(D) = Σ D_i · e^(r·(T - t_i))   (same dividends grown to expiry)
```
- Both models satisfy the same put-call parity
  `C - P = S - PV(D) - K·e^(-rT)`, but they give slightly different prices.
- The notebook prints a comparison of both models, the price with no dividends,
  and the implied volatility under each model.

---

## What is put-call parity?

A call and a put with the **same strike and the same expiry** are linked together -
you can build one from the other plus some cash or stock. So their prices can never
move independently. This is not a model guess - it follows from simple no-arbitrage
logic.

**why it must be true:**

- **Portfolio A:** buy a call + keep `K·e^(-rT)` in the bank.
  At expiry you own the stock (if it is worth more than K) or the cash (if it is not) -
  either way you end up with `max(stock price, K)`.
- **Portfolio B:** buy a put + buy the stock (minus whatever cash dividends it pays
  before expiry).
  At expiry the put protects you if the stock falls - you also end up with
  `max(stock price, K)`.

Both portfolios pay the same, so they must cost the same today. That gives one
formula per notebook:

| Notebook | Put-call parity formula |
|---|---|
| 1 - no dividends | `C - P = S - K·e^(-rT)` |
| 2 - continuous dividend yield q | `C - P = S·e^(-qT) - K·e^(-rT)` |
| 3 - discrete cash dividends | `C - P = S - PV(D) - K·e^(-rT)` |

(In words: call minus put equals the stock minus the present value of the strike -
shrunk by the value of any dividends the stock pays before expiry.)

**Why every notebook prints a parity check:**

- Unlike the pricing formula, parity does **not** use volatility, dividends or any
  model assumption - it only uses the no-arbitrage logic shown above. So it must be
  true *exactly*, no matter what the market does.
- Each notebook therefore prints three numbers: `C - P`, the value it *should* be,
  and the difference (the residual). Since both sides come from the same code, a
  residual close to `0.00000000` confirms the call and put formulas were coded
  correctly. A big residual would mean there is a bug in the code.
- The same check works on real market prices too. If a live market ever violated
  parity, you could make a risk-free profit: buy the side that is underpriced and
  sell the side that is overpriced. Trades like this - where you profit with no risk -
  are called *arbitrage*, and they disappear almost instantly because everyone
  rushes to take them. That is exactly why parity holds in practice.


