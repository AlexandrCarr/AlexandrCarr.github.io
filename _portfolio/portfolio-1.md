---
title: "Transformer-based L/S Strategy on the Russell 1000: 1.31 OOS Sharpe (2020 - 2024)"
excerpt: "Short description of portfolio item number 1<br/><img src='/images/500x300.png'>"
collection: portfolio
---
Inspired by deep learning courses at Harvard and MIT, I decided to build a transformer-based L/S strategy for the Russell 1000 (R1000) index. My in-sample period was 07/2000 (post-2000 rebalancing) to 06/2020 and my out-of-sample period was 07/2020 (post-2020 rebalancing) to 12/2024. I detail the steps I took, my results, and some reflections.

**1) Data Collection and Cleaning.**

Using the Bloomberg Terminal and various WRDS datasets, I collected point-in-time data on all the components of the R1000 from June 2000 to June 2024. This included price data, 32 fundamental indicators (e.g. profitability, debt, and other theoretically accounting data), and more technical indicators (e.g. various types of momentum, like one-month or 12-month momentum). This involved laborious data cleaning, like learning how to deal with various types of outliers or missing data, and sharpened my knowledge of Pandas. 

If you want to see a sample of my data cleaning and formatting, please look at the final Russell 1000 dataset in the data section of the website or scroll through Project X (this was a very long and data-cleaning-intensive pset). 

**2) Model Design.**

After much experimentation, I settled on the following model. 

- I snapshot each company’s 35 indicators on each calendar date, exactly
as they were published—no peeking ahead or filling missing data.

- Inspired by Fama-Frecnch, every fundamental value is shifted forward by
three months so the model only sees information that would have been in
investors’ hands at the time.

- On each date, I standardize all variables across all stocks.

- For each ticker I stack the last four quarterly rows into a 4 × 35 “movie
strip.” Batching these gives a tensor X $\in$ ∈ R^{B×4×35} that shows one year
of dynamics per example.

-

**3) Portfolio Construction.**

Deriving from Professor Matthew Rothman's excellent class at MIT and independent reading, I imposed the following constraints on my model.

First, the model was factor neutral with respect to the 15 latent factors. 

Second, the model allocated at most 2% of the portfolio value to any given stock and limited gross leverage to 2x.

Third, the model's max volatility was 12% and its maximum drawdown was set to 5%.

Fourth, the model was L/S dollar neutral. 

Finally, I included 10 bps of TCosts.  

I found the process of portfolio optimization the most exciting and eye-opening part of the project! Firstly, it was fascinating how relatively small changes in gross leverage or the maximum weight per stock would result in very unexpected results. For instance, shifting my gross leverage from 2x to 2.5x reduced the Sharpe Ratio by approximately 0.15, increasing it from 2.5x to 2.75x brought my Sharpe to below 0.5, and I even had negative annualized returns for gross leverage exceeding 4.5. Secondly, almost certainly romanticizing the process, it felt good imposing responsibile behaviour on a model that is far smarter, faster, and more powerful than I can ever be. Although the model yielded a slightly lower Sharpe when I began constraining volatility (fell from 1.52 to 1.31), I never blew up and was a more reliable money-maker. I was just quite enthralled by the process of designing a system that reproducibly yielded an attractive return at an acceptable level of risk (i.e. 5% max drawdown and 12% vol with 2x gross leverage).

**4) Dealing with Poorly Behaved Edge Cases.**

Upon reading several papers from Ledoit, I realized that my optimization in step 3 would collapse when my variance-covariance matrix estimation would be poor.

For instance, let's say that we are facing a massive market drawdown. Through various mechanisms, be it Brunnermeier and Sannikov's (2014) fire sales or Geanakoplos' (2010) leverage cycles, the main factor driving individual stock returns becomes the exposure to the market. In this situation, if we undergo an eigen-decomposition of the variance-covariance matrix, we will find that the matrix of eigenvalues is zero or almost zero for all factors but market exposure. More mathematically, the spectral distribution of eigenvalues will be extremely concentrated at zero. This would plausibly mean that we cannot invert the eigenvalue matrix, and therefore cannot invert the entire variance-covariance matrix. This is very problematic because a lot of step 3 requires invertibility. 

There are many other cases like this. To address this, Ledoit develops linear and non-linear shrinkage methods. I imposed linear shrinkage methods to ensure that step 3 does not break down and I don't go out of business.
