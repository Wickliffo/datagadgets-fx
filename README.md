# DataGadgets FX — Multi-Currency Sales Normalizer

## The Problem

DataGadgets started as a US-only gadget webshop. Life was simple: every transaction arrived in USD, reports balanced perfectly, and the finance team slept soundly. Then came international expansion. Overnight, orders started arriving in British Pounds and Euros. The in-house reporting pipeline — built for a single-currency world — broke completely. Totals were nonsensical, and the team couldn't answer one basic question: *"How much did we actually sell today, in USD?"* With hundreds of orders per day across three currencies, manual conversion was out of the question.

## The Solution

This project builds a clean Python pipeline that solves the problem end-to-end. It loads all raw transactions from a CSV file into a pandas DataFrame, queries the VATcomply API for ECB-published historical exchange rates on the exact transaction date, maps the correct exchange rate to every order based on its currency, computes the USD equivalent for each transaction, and produces a single headline figure — total USD sales for the day. The result is a reproducible, date-aware script that can be run for any trading day, not just a one-off calculation.

## How It Works

The pipeline starts by importing pandas and requests, then loading the orders CSV into a DataFrame with two columns: `amount` and `currency`. Next, it queries the VATcomply rates API — a free, no-auth-required service that sources its data from the European Central Bank — passing `base='USD'` and the target date as parameters. With `base='USD'`, each rate in the response represents how much of that foreign currency equals one US Dollar (for example, `EUR: 0.918527` means 0.9185 EUR = 1 USD). The exchange rate for each order's currency is then mapped onto the DataFrame using pandas' `.map()` method, which performs a vectorized dictionary lookup across the entire column in one shot — no loops needed. Finally, `amount_usd` is calculated by multiplying each order's amount by its exchange rate, and `.sum()` is called on the resulting column to produce `total_usd_sales`.

## The Key Insight

The most important thing to understand is the direction of the conversion. Because `base='USD'`, the rates are expressed as *foreign units per one USD*. This means multiplying — not dividing — gives you the correct USD equivalent. For example, 43.75 EUR multiplied by 0.918527 gives 40.19 USD. For USD orders, the rate is 1.0, so the amount is unchanged. This distinction is subtle but critical — getting it backwards produces a plausible-looking but completely wrong result.

## Result

Running the pipeline against 723 real orders from January 21st, 2024 — split across 252 GBP transactions, 237 EUR transactions, and 234 USD transactions — produces a `total_usd_sales` of **$326,864.49**. The final DataFrame contains four columns: `amount`, `currency`, `exchange_rate`, and `amount_usd`, giving full traceability from every raw order to its USD equivalent.

## Installation

Clone the repository, install the two dependencies with `pip install -r requirements.txt` (pandas and requests), and run `python solution.py`. No API key or account registration is needed — VATcomply is completely free and open for public use.

## Project Structure

```
datagadgets-fx/
├── data/
│   └── orders-2024-01-21.csv
├── solution.py
├── requirements.txt
└── README.md
```

## Requirements

```
pandas
requests
```

---

*Data sourced from the European Central Bank via the VATcomply API.*
