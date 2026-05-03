---
last_modified: 2026-05-03T14:43:35+05:30
layout: guides
title: Quant Finance Roadmap
description: Phased learning roadmap for quantitative finance
---
# First Phase: Stationarity and co-integration
- try auto regressive models like AR, MA, ARIMA and GARCH
- factor models like fama french 3 and 5 factor models

Prove why a standard XGBoost model mapping features to price would fail, and why using 'log returns' as a target might work better ?

Build a simple linear regressor for a stock's return, are the residuals white noise ? Why or why not

# Second Phase: 

- no arbitrage principle
- binomial trees and black scholes
- bond pricing, yield curve and duration in context of fixed income

Implement the black scholes formula in python
Then write a root finding algorithm (like newton raphson) to calculate implied volatility from market prices.

Simular a stock price using Geometric Brownian Motion and hedge an option possible.

# Third Phase
Derive the SDE for a 'Lookback Option'

Build a monte carlo engine that handles 'barrier option', compare speed and accuracy against a closed form solution.

