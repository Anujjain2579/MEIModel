## 1. Data Preparation  
We assembled two time series:  
- **Multivariate ENSO Index (MEI).**  
  - Loaded monthly MEI from 1979 onward, parsed dates, and forced a consistent month‐start frequency.  
- **Renewable‐energy stock prices.**  
  - Downloaded monthly closes for a basket of pure‐play names (e.g. BEP, CWEN, DNNGY, NEE).  
  - Applied an HP‐filter (λ=129 600) to extract each stock’s cyclical component, isolating short‐term price fluctuations.
- **Fats and Oils Stock (SIC Code 207) prices.**  
  - Downloaded monthly closes for ADM, BG, DAR.  
  - Applied an HP‐filter (λ=129 600) to extract each stock’s cyclical component, isolating short‐term price fluctuations.  

## 2. Modeling MEI Dynamics  
We tested three common time‐series approaches:  
- **Autoregression (AR).**  
  - Selected AR(6) by minimizing AIC; its coefficients exhibited very slow decay, confirming multi-month persistence.  
- **ARIMA.**  
  - Explored small (p,d,q) grids; ARIMA(2,0,2) was best by AIC but offered no diagnostic improvement over AR(6).  
- **Seasonal SARIMAX.**  
  - Fitted a handful of monthly‐seasonal specifications; SARIMAX(1,0,1)(1,0,1,12) passed serial-corr tests but added complexity for minimal gain.  

All three models produced residuals indistinguishable from white noise (Durbin–Watson ≈2, Breusch–Godfrey p≫0.05).

## 3. Generating Trading Signals  
- We used the **AR(6)** one-step-ahead forecast of MEI as our signal (optionally interchangeable with the SARIMAX forecast).  
- Converted forecasts into **z-scores** against the in-sample MEI mean and standard deviation.  
- Applied a **hysteresis rule**:  
  - **Short** (–1) when forecast z > +1.5σ.  
  - **Reverse to long** (+1) when forecast z falls below +1.25σ.  
  - **Flat** until the first trigger, avoiding over-trading in a narrow band.

## 4. Strategy Implementation  
- At each month-end we take the forecast-driven position and apply it to the **next month’s actual price return** of each stock.  
- Equity grows by compounding (1 + monthly strategy return).

## 5. Trade-by-Trade Analysis & Metrics  
For each name we:  
- **Identified** entry/exit dates, direction, holding months, and trade return.  
- **Computed** overall metrics:  
  - **Sharpe ratio** (annualized)  
  - **Max draw-down**  
  - **Turnover rate** (full round-trips per month)  
  - **Average holding period**

| Symbol | Trades | Sharpe | Max DD   | Turnover/mo | Avg Hold (mo) |
|:------:|:------:|:------:|:--------:|:-----------:|:-------------:|
| BEP    | 29     | 0.48   | –41.7%   | 0.121       | 7.93          |
| CWEN   | 12     | 0.36   | –52.3%   | 0.095       | 9.67          |
| DNNGY  | 11     | 0.25   | –66.0%   | 0.115       | 7.91          |
| NEE    | 30     | 0.40   | –52.0%   | 0.098       | 8.97          |
| ADM    | 12     | 0.47   | –55.4%   | 0.038       | 22.42         |
| BG     | 12     | 0.45   | –67.8%   | 0.041       | 22.42         |
| DAR    | 12     | 0.60   | –77.5%   | 0.038       | 22.42         |


## 6. Key Takeaways  
- MEI is a **strongly persistent** monthly series; a mid-lag AR captures most of its dynamics. SARIMAX also works
- A simple **hysteresis** on the SARIMAX forecast produces a manageable signal with limited churn.  
- **Performance varies** by ticker:  
  - Better Sharpe than Buy & Hold: BEP 0.48 (vs 0.32), CWEN 0.36 (vs 0.13), DNNGY 0.25 (vs 0.012)
  - Poor:  NEE 0.4 (vs 0.61)  
- **Next steps** could include regime-switching models, threshold optimization, and integration of fundamental/seasonal demand indicators.
