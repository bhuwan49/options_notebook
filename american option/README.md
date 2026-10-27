# American Option Pricing with Binomial Trees (Cox-Ross-Rubinstein)

**American options** can be exercised **any time up to expiry** (this is how most stock options work). Because of this early-exercise right, Black-Scholes does **not** apply - there is no simple closed-form formula. Instead we build a **binomial tree**.

All three notebooks here price on **KO (Coca-Cola)** with live data from `yfinance`, so their results can be compared. Risk-free rate comes from `^IRX`, volatility from KO's historical log returns.

**American vs European:** an **European** option can only be exercised at expiry. Every notebook prices both - the difference is the **early-exercise premium**:

```
early-exercise premium = American price − European price  (≥ 0)
```

## How the tree works

Cut the option's life into $N$ steps of length $\Delta t = T/N$. At every step the stock moves up or down:

$$\Delta t = \frac{T}{N}, \qquad u = e^{\sigma\sqrt{\Delta t}}, \qquad d = \frac{1}{u}$$

Risk-neutral probability of an up move:

$$p = \frac{e^{r\,\Delta t} - d}{u - d}$$

Pricing runs **backwards**: start from the payoff at expiry, then step back one node at a time:

$$\text{value} = e^{-r\,\Delta t}\Big[p\cdot\text{value}_{up} + (1-p)\cdot\text{value}_{down}\Big]$$

At every node, **American options** also compare the exercise payoff:

$$\text{value} = \max\Big(\text{exercise payoff},\ \text{continuation value}\Big)$$

European options skip this step (exercise only at expiry).

## The three notebooks

### 1. `binomial option pricing.ipynb` - no dividends

- CRR parameters $u$, $d$, $p$ as above
- prices American and European calls and puts
- **expected results:** American call = European call (premium 0, early exercise of a call is never worth it without dividends), American put > European put
- **convergence check:** the European tree prices must match the Black-Scholes formula as $N$ grows

### 2. `binomial option pricing (continuous dividend).ipynb` - continuous dividend yield $q$

Dividends are smoothed into a continuous stream $q$ (KO's trailing 12-month payments / price). Only the up-probability changes, because the stock's drift becomes $r - q$:

$$p = \frac{e^{(r - q)\,\Delta t} - d}{u - d}$$

- the stock has no price jumps, so payoffs use the real stock price directly
- **expected results:** American call = European call (premium 0), American put > European put
- **validation:** European tree prices must match the closed-form Black-Scholes-Merton formula with dividends:

$$C = S_0e^{-qT}N(d_1) - Ke^{-rT}N(d_2), \qquad d_1 = \frac{\ln(S_0/K) + (r - q + \tfrac12\sigma^2)T}{\sigma\sqrt{T}}$$

### 3. `binomial option pricing (discrete dividend).ipynb` - fixed cash dividends on known dates

KO pays about $0.53 per share every quarter. On the ex-date the stock jumps **down by exactly that amount** - which breaks the recombining tree. The fix is the **escrowed method**: split the stock into

$$S(t) = \underbrace{X(t)}_{\text{risky part}} + \underbrace{PV_{rem}(t)}_{\text{cash still to be paid out}}$$

Both parts drop by $D$ on the ex-date, so $X$ never jumps and a normal tree works on it. Start the tree at

$$X_0 = S_0 - PV(D), \qquad PV(D) = \sum_i D_i e^{-r t_i}$$

At any node with time $t$, the real stock price used for payoffs and exercise decisions is

$$S = X + PV_{rem}(t), \qquad PV_{rem}(t) = \sum_{t_j > t} D_j e^{-r (t_j - t)}$$

- exercising **just before an ex-date** becomes attractive automatically - the holder collects the dividend (exactly what real holders do)
- **validation:** the European tree price must match the escrowed Black-Scholes price ($S_0$ replaced by $S_0 - PV(D)$)

## Implied volatility

Every notebook ends with an **implied volatility** section: given a market price, find the $\sigma$ that makes the tree produce it. The same Brent root-finder (`scipy.optimize.brentq`) is used as in the European project - only the pricer inside is the binomial tree.

## Common settings

| Setting | Value |
|---|---|
| Underlying | KO |
| Strike | nearest-dollar (at the money) |
| Expiry | 2027-06-18 |
| Tree steps | $N = 500$ |
| Market prices for IV | call $4.50, put $3.50 |
