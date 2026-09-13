# News Sentiment and Stock Price Volatility

A data science project exploring whether stock price volatility is actually correlated with news sentiment, or whether "the market reacted to the news" is more of a media narrative than a measurable pattern.

**Tickers analyzed:** GOOGL (Alphabet), NVDA (Nvidia), META (Meta Platforms), AAPL (Apple), MSFT (Microsoft)

---

## Table of Contents
- [Problem Definition](#problem-definition)
- [Data Collection](#data-collection)
- [Data Description](#data-description)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Findings](#findings)
- [Limitations and Ethics](#limitations-and-ethics)
- [References](#references)
- [AI Usage Disclosure](#ai-usage-disclosure)

---

## Problem Definition

**Research question:** Does news sentiment correlate with stock price volatility for GOOGL, NVDA, META, AAPL, and MSFT?

**Context:** Financial media and market commentary frequently attribute stock price swings to news coverage — a company gets a wave of headlines and its stock is said to have "reacted." This is often stated as fact, but rarely tested directly. Understanding whether sentiment actually correlates with volatility (rather than just being a convenient narrative after the fact) matters for how we interpret market behavior.

**Why it matters:** Investors, financial journalists, and anyone trying to make sense of day-to-day stock movements would benefit from knowing whether "the market reacted to the news" reflects a real, measurable pattern or is largely a post-hoc storytelling device layered onto price movements that would have happened anyway.

---

## Data Collection

**Price data — [`yfinance`](https://github.com/ranaroussi/yfinance):** Daily OHLCV (open, high, low, close, volume) data for 5 tickers, pulled via the `yfinance` Python library — an open-source tool providing programmatic access to Yahoo Finance market data for research and educational purposes.

**News data — [Finnhub API](https://finnhub.io/):** Company news headlines and summaries for the same 5 tickers, pulled via a free-tier API key using the `company-news` endpoint, filtered by ticker symbol and date range.

> Finnhub was chosen after an initial attempt using the Marketaux API, whose free tier returned articles clustered on only the most recent 1–2 days regardless of the requested date range. Finnhub's free tier advertises up to a year of historical news access; in practice, real article coverage was still found to concentrate heavily on the most recent several days of any requested window — documented further in [Limitations](#limitations-and-ethics).

**Sentiment scoring — [VADER](https://github.com/cjhutto/vaderSentiment):** Finnhub does not provide a built-in sentiment score, so sentiment was computed directly from each article's headline and summary text using VADER (Valence Aware Dictionary and sEntiment Reasoner), a lexicon-based sentiment tool from the `nltk` library, well-suited to short, informal text.

---

## Data Description

| | |
|---|---|
| **Stocks used** | GOOGL, NVDA, META, AAPL, MSFT — a coherent Big Tech/AI sector group |
| **Variables** | Daily return (volatility proxy), average daily sentiment score (VADER compound), article count per ticker/day |
| **Observations** | 13 (ticker, date) rows after merging: AAPL (3), GOOGL (3), META (3), MSFT (3), NVDA (1) |
| **Missing values** | Price and news data only overlapped Sept 8–11, 2026; days outside that window were excluded via inner join rather than filled with placeholder sentiment |

---

## Data Cleaning and Preparation

- Reshaped price data from wide format (one column set per ticker) to long format (one row per date + ticker) to enable merging.
- Combined each article's headline and summary into one text field, scored with VADER to produce a `sentiment_score` per article.
- Dropped articles with no usable text and duplicate articles (same ticker, date, headline).
- Converted Unix publish timestamps to a plain date (day-level) to align with price data granularity.
- Averaged sentiment scores per (ticker, date) into one daily value per ticker, with an accompanying article count.
- Merged price and sentiment data on (ticker, date) using an **inner join** — kept only days present in both datasets rather than filling missing news days with a neutral/zero placeholder.

---

## Exploratory Data Analysis

### Visualization 1 — Daily Return vs. Average News Sentiment, by Ticker
A scatter plot of all 13 observations, sentiment on the x-axis and same-day return on the y-axis, colored by ticker. Every observation had net-positive sentiment (no negative-sentiment day appeared in the sample), so this chart shows how the *strength* of positive sentiment related to returns, rather than a positive-vs-negative comparison. Points are broadly scattered with no single clean trend — high-sentiment points appear at both slightly positive and clearly negative returns.

### Visualization 2 — Correlation Between Sentiment and Return, by Ticker
A bar chart of the return–sentiment correlation coefficient calculated separately per ticker:

| Ticker | Correlation (r) |
|---|---|
| AAPL | +0.63 |
| GOOGL | −0.86 |
| META | −0.89 |
| MSFT | −0.74 |
| NVDA | excluded (single observation) |

---

## Findings

Across the full sample, daily return and news sentiment showed a **moderate negative overall correlation (r = −0.53)** — the opposite of what a simple "positive news drives positive returns" narrative would predict. This overall number masks sharp disagreement between companies: AAPL showed a clear positive relationship, while GOOGL, META, and MSFT all showed strong negative relationships.

**Important caveat:** every observation happened to have net-positive average sentiment — no clearly negative-sentiment day existed in this window. These findings speak to how the strength of positive coverage related to returns, not a true positive-vs-negative comparison.

Given the inconsistency across companies, it would be inaccurate to conclude that news sentiment reliably predicts stock returns for these companies. If anything, the sharp per-ticker disagreement supports the opposite reading: "the market reacted to the news" may describe individual cases well after the fact, but does not hold up as a consistent, generalizable pattern.

**What would be incorrect to conclude:** That news sentiment has no effect on stock price — the sample is small (13 observations, ~1 week, all positive-sentiment days) and doesn't control for other factors driving price on a given day. Correlation observed here also does not establish causation in either direction.

---

## Limitations and Ethics

- **API pivots:** This project went through three free-tier news APIs before landing on a usable dataset. Marketaux consistently returned only the most recent 1–2 days of coverage regardless of date range. Alpha Vantage's free tier capped requests at 25/day, making a wide date range impractical. Finnhub, despite advertising a year of historical access, still clustered 240+ raw articles per ticker on the final 3–4 days before the pull date — resulting in a final sample of only 13 observations over roughly one week rather than the multi-month window originally planned.
- **All-positive sentiment sample:** No net-negative sentiment day appeared in the data, limiting the analysis to examining the strength of positive sentiment rather than a true positive-vs-negative comparison.
- **Uneven ticker coverage:** NVDA had only 1 matching observation (vs. 3 for the others), since its available news concentrated on Sept 11–12, and Sept 12 fell outside the available price data range. No meaningful correlation could be calculated for NVDA.
- **General-purpose sentiment tool:** VADER is not finance-tuned and may misread finance-specific phrasing (e.g., "beat estimates," "missed guidance") that a model like FinBERT would score more accurately.
- **Uncontrolled confounders:** Macroeconomic conditions, sector-wide trends, and broader market movement are not controlled for; any correlation observed should not be read as causal.
- **Potential biases:** Article counts per matched observation ranged from 42–167, so daily sentiment averages rest on very different underlying sample sizes day to day. All five tickers are large-cap, heavily-covered tech companies — findings may not generalize to smaller or less-covered stocks.

**What I'd explore next:** A paid-tier news API with genuine multi-month historical access; a finance-tuned sentiment model (e.g., FinBERT) in place of VADER; a longer window with genuine sentiment variation in both directions; and controlling for broader market movement (e.g., comparing against a sector ETF like XLK) to isolate company-specific effects.

---

## References

Heston, S. L., & Sinha, N. R. (2017). News vs. sentiment: Predicting stock returns from news stories. *Financial Analysts Journal, 73*(3), 67–83. https://doi.org/10.2469/faj.v73.n3.3

Hutto, C. J., & Gilbert, E. (2014). VADER: A parsimonious rule-based model for sentiment analysis of social media text. *Proceedings of the International AAAI Conference on Web and Social Media, 8*(1), 216–225. https://doi.org/10.1609/icwsm.v8i1.14550

Wan, X., Yang, J., Marinov, S., Calliess, J. P., Zohren, S., & Dong, X. (2021). Sentiment correlation in financial news networks and associated market movements. *Scientific Reports, 11*(1). https://doi.org/10.1038/s41598-021-82338-6

---

## AI Usage Disclosure

Generative AI (Claude, Anthropic) was used throughout this project for: explaining API integration concepts (yfinance, Marketaux, Finnhub), debugging code errors (KeyError troubleshooting, merge issues, date range mismatches across three different APIs), and structuring the written report sections. All code was reviewed, run, and adjusted by the student; AI was not used to generate analytical conclusions or write the findings/storytelling section, which reflect the student's own interpretation of the data.