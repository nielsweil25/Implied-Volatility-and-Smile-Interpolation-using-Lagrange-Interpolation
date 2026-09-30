# Implied Volatility & Smile Interpolation

A Python project implementing numerical methods to recover implied volatility from European call prices and reconstruct a synthetic volatility smile.

## Overview

The project follows four steps:

1. Price European calls using the Black–Scholes model.
2. Recover implied volatility using bisection and Newton's method.
3. Generate synthetic option prices across nine strikes and recover their implied volatilities.
4. Interpolate the recovered smile using Lagrange polynomials.

The numerical algorithms are implemented from scratch.

## Mathematical Framework

### Black–Scholes Call Pricing

For a European call on a non-dividend-paying underlying:

$$
C(S,K,T,r,\sigma)=S\Phi(d_1)-Ke^{-rT}\Phi(d_2)
$$

where:

$$
d_1=\frac{\ln(S/K)+(r+\sigma^2/2)T}{\sigma\sqrt{T}},
\qquad
d_2=d_1-\sigma\sqrt{T}.
$$

Here, S is the spot price, K the strike, T the time to maturity, r the continuously compounded risk-free rate, and σ the annualized volatility. Φ denotes the standard normal cumulative distribution function.

### Implied Volatility

Implied volatility solves:

$$
C(S,K,T,r,\sigma)-C_{\mathrm{market}}=0.
$$

**Bisection** repeatedly halves an interval containing the root.

**Newton's method** updates the volatility estimate using vega:


$$
\sigma_{n+1}
$$
=
$$
\sigma_n-
\frac{C(S,K,T,r,\sigma_n)-C_{\mathrm{market}}}
{\mathrm{Vega}(\sigma_n)}
$$

with:

$$
\mathrm{Vega}=S\phi(d_1)\sqrt{T},
$$

where φ is the standard normal probability density function.

The Newton implementation checks for a near-zero vega and raises an error if the pricing tolerance is not reached within the iteration limit.

### Smile Interpolation

Given recovered strike–volatility pairs, the Lagrange interpolant is:

$$
P(K)=\sum_{i=0}^{n-1}\sigma_iL_i(K)
$$

where:

$$
L_i(K)=
\prod_{\substack{j=0\\j\ne i}}^{n-1}
\frac{K-K_j}{K_i-K_j}.
$$

This polynomial estimates volatility between the sampled strikes.

## Synthetic Experiment

| Parameter | Value |
|---|---|
| Spot price | 100 |
| Time to maturity | 1 year |
| Risk-free rate | 3% |
| Strikes | 9 equally spaced points from 70 to 130 |

The reference smile is:

$$
\sigma_{\mathrm{ref}}(K)
$$
=
$$
0.20+0.8\left(\frac{K}{S}-1\right)^2.
$$

Its volatility is 20% at the money and 27.2% at strikes 70 and 130.

Call prices are generated from this reference curve. Newton's method then recovers the implied volatilities from those prices, and Lagrange interpolation reconstructs the curve on a grid of 300 strikes.

## Results

- In the initial root-finding example, both methods recovered an implied volatility of approximately **22.77%**.
- Bisection required **20 iterations**, compared with **2 for Newton**.
- Across the synthetic smile, the recovered volatilities closely matched the reference values.
- The notebook plots the reference curve, recovered points and interpolated curve, and calculates the maximum absolute volatility error.

The iteration counts apply to the initial example. The stopping criteria differ: interval width for bisection and absolute pricing error for Newton.

## Limitations

The reference smile is quadratic. Lagrange interpolation with at least three distinct nodes therefore reproduces it exactly in exact arithmetic. The remaining error comes from root-finding and floating-point calculations.

This project validates the methods on synthetic data. It does not demonstrate interpolation accuracy on real market data or enforce arbitrage-free option prices.

Newton's method is sensitive to its starting value and can fail if vega is too small or an iteration produces a nonpositive volatility.

