<div align="center">

# 📈 NEPSE All Scraper

**A free data pipeline for the Nepal Stock Exchange.**  
Collects the full daily floorsheet and serves the existing price, dividend, right-share, and index history for **365 listed companies** — committed automatically via GitHub Actions.

🚀 **Live API + Swagger UI:** https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/  
🌱 **Original project:** [Samir Wagle / Nepse-All-Scraper](https://github.com/SamirWagle/Nepse-All-Scraper) · [original live site](https://samirwagle.github.io/Nepse-All-Scraper/)

[![Daily Scraper](https://github.com/ghimireprakaash/Nepse-All-Scraper/actions/workflows/daily_scraper.yml/badge.svg)](https://github.com/ghimireprakaash/Nepse-All-Scraper/actions/workflows/daily_scraper.yml)
[![Latest Scrape](https://img.shields.io/endpoint?url=https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/status.json&label=latest+scrape)](https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/status.json)
![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![Data Source](https://img.shields.io/badge/Source-ShareSansar%20%7C%20Merolagani-orange)
[![License](https://img.shields.io/badge/License-See%20CREDITS-lightgrey)](CREDITS.md)
![Companies](https://img.shields.io/badge/Companies-365-purple)

</div>

> **Origin & Credits:** This project was originally created by **Samir Wagle** ([@SamirWagle](https://github.com/SamirWagle)). This repository is a derivative maintained by **Prakash Ghimire** ([@ghimireprakaash](https://github.com/ghimireprakaash)). The original scraper, data pipeline, and API design are his work; see [CREDITS.md](CREDITS.md) for provenance and the changes made here.

---

## 📦 What's Inside

> This repo is **data-first**. The scheduled GitHub Actions workflow is **floorsheet-only**: it runs every day at 6:00 PM NPT and commits new floorsheet data directly to `data/`. Price, dividend, right-share, and index histories remain available in the repo and API, but they are no longer refreshed by the schedule. No database, no server — just flat CSV files you can plug into anything.

| Data | Where | Updated |
|---|---|---|
| OHLC price history | `data/company-wise/{SYMBOL}/prices.csv` | Historical (manual refresh) |
| Dividend history | `data/company-wise/{SYMBOL}/dividend.csv` | Historical (manual refresh) |
| Right share history | `data/company-wise/{SYMBOL}/right-share.csv` | Historical (manual refresh) |
| Full daily floorsheet | `data/floorsheet/floorsheet_YYYY-MM-DD.csv` | Every day ✅ |
| NEPSE index history | `data/indices/` | Historical (manual refresh) |

---

## 📁 Repository Layout

```
Nepse-All-Scraper/
│
├── .github/
│   └── workflows/
│       └── daily_scraper.yml      # GitHub Actions — floorsheet-only, runs daily at 6:00 PM NPT
│
├── data/
│   ├── company_list.json          # 365 priority company symbols
│   ├── company_id_mapping.json    # Symbol → ShareSansar internal ID
│   ├── floorsheet/
│   │   └── floorsheet_YYYY-MM-DD.csv  # Daily floorsheet (all trades)
│   ├── indices/                   # NEPSE index history (historical)
│   └── company-wise/
│       └── {SYMBOL}/
│           ├── prices.csv         # Full OHLC price history
│           ├── dividend.csv       # Dividend history
│           └── right-share.csv    # Right share history
│
└── scraper/
    ├── run_github_actions.py      # ← Floorsheet entry point (other modes retained)
    ├── run_daily.py               # ← Local price scraper CLI
    └── core/
        ├── daily.py               # Orchestrates price scraping
        ├── daily_prices.py        # Daily price summary updater
        ├── floorsheet.py          # Legacy standalone floorsheet scraper
        └── history.py             # OHLC price history scraper
```

---

## 🤖 Automation — GitHub Actions

The workflow [`.github/workflows/daily_scraper.yml`](.github/workflows/daily_scraper.yml) runs automatically **every day at 6:00 PM Nepal time** (12:15 UTC), right after NEPSE closes.

```
┌──────────────────────────────────────────────────────┐
│        GitHub Actions — Daily Floorsheet Run         │
├─────────────┬────────────────────────────────────────┤
│  Floorsheet │  Full day's trades from merolagani.com  │
│  API build  │  Rebuilds docs/api/floorsheet/index.json│
│  Commit     │  git push only when data/docs changed  │
└─────────────┴────────────────────────────────────────┘
```

> **Floorsheet-only CI:** the workflow saves data under Merolagani's actual market date, so weekend and holiday runs do not create fake files. It deduplicates contract numbers and refuses to replace an existing day with a smaller scrape. Price, dividend, right-share, and index update steps are intentionally disabled in the workflow; their code, historical CSVs, and API endpoints are retained for manual refreshes.

**Trigger manually**: `GitHub → Actions → Daily Scraper → Run workflow`

---

## ⚡ Quickstart

### Prerequisites
```bash
pip install requests beautifulsoup4
```

### Run the same scrape as GitHub Actions (floorsheet only)
```bash
python scraper/run_github_actions.py --floorsheet

# Preview only — does not write data
python scraper/run_github_actions.py --floorsheet --max-pages 3

# Optional historical scrapers (not run by the schedule)
python scraper/run_github_actions.py --dividends
python scraper/run_github_actions.py --right-shares
```

### Scrape full OHLC price history (local only, first-time)
```bash
# Full history for all 365 companies — takes ~2-4 hours on first run
python scraper/run_daily.py --full-scrape

# Incremental — only fetches newer records than what's already in prices.csv
python scraper/run_daily.py --incremental

# Only process newly listed companies (new IPOs)
python scraper/run_daily.py --new-only
```

> **Why are prices local-only?**  
> Price scraping needs the existing `prices.csv` files to know where to stop (incremental logic). Run it locally when you want to refresh prices; the scheduled GitHub Actions workflow only handles the daily floorsheet.

---

## 🌐 Public API (free, no auth)

Two equivalent ways to consume the data — pick whichever your client likes.

### A. Swagger / JSON via GitHub Pages

```
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/            # Swagger UI
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/openapi.json     # OpenAPI 3.1 spec
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/companies.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/latest.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/prices/ADBL.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/dividends/ADBL.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/right-shares/ADBL.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/floorsheet/index.json
https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/status.json  # health snapshot
```

### B. Raw CSV via jsDelivr CDN (faster, no rate limit)

[jsDelivr](https://www.jsdelivr.com/) auto-mirrors public GitHub repos on a global CDN. Same files, faster, no GitHub rate limits, automatic gzip:

```
https://cdn.jsdelivr.net/gh/ghimireprakaash/Nepse-All-Scraper@main/data/company_list.json
https://cdn.jsdelivr.net/gh/ghimireprakaash/Nepse-All-Scraper@main/data/company-wise/ADBL/prices.csv
https://cdn.jsdelivr.net/gh/ghimireprakaash/Nepse-All-Scraper@main/data/company-wise/ADBL/dividend.csv
https://cdn.jsdelivr.net/gh/ghimireprakaash/Nepse-All-Scraper@main/data/floorsheet/floorsheet_2026-11-06.csv
```

> Symbols with `/` in the name (e.g. `GBILD86/87`) live on disk as `GBILD86-87`. Use the dashed form in URLs.

### Quick examples

```python
# Python
import requests
prices = requests.get("https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/prices/ADBL.json").json()
print(prices["count"], prices["data"][-1])
```

```js
// JavaScript / TypeScript
const r = await fetch("https://ghimireprakaash.github.io/Nepse-All-Scraper/docs/api/latest.json");
const latest = await r.json();
console.log(latest["ADBL"]);
```

```bash
# CLI
curl https://cdn.jsdelivr.net/gh/ghimireprakaash/Nepse-All-Scraper@main/data/company-wise/ADBL/dividend.csv
```

---

## 📄 Data Formats

<details>
<summary><strong>prices.csv</strong></summary>

```
date, open, high, low, ltp, percent_change, qty, turnover
2024-01-15, 1200, 1250, 1190, 1240, +1.5%, 3400, 4216000
```

</details>

<details>
<summary><strong>dividend.csv</strong></summary>

```
fiscal_year, bonus_share, cash_dividend, total_dividend, book_closure_date
2079/80, 10%, 5%, 15%, 2023-12-01
```

</details>

<details>
<summary><strong>right-share.csv</strong></summary>

```
ratio, total_units, issue_price, opening_date, closing_date, status, issue_manager
1:1, 5000000, 100, 2023-11-01, 2023-11-15, Closed, XYZ Capital
```

</details>

<details>
<summary><strong>floorsheet/floorsheet_YYYY-MM-DD.csv</strong></summary>

```
date, sn, contract_no, stock_symbol, buyer, seller, quantity, rate, amount
2024-01-15, 1, 100012345, ADBL, 21, 42, 500, 1240, 620000
```

</details>

---

## ⚙️ How Incremental Price Scraping Works (manual)

```
prices.csv                    ShareSansar AJAX
─────────────                 ──────────────────
Latest date: 2024-01-10  →   Stop fetching when
                              record date ≤ 2024-01-10

Result: Only 1-2 pages fetched instead of 70+
```

`history.py` reads the most recent date from the existing `prices.csv`, passes it as a `stop_date` to the paginator, and halts the moment it hits older data. This makes daily updates take seconds instead of hours.

---

## 🚀 First-Time Setup

```bash
# 1. Clone the repo
git clone https://github.com/ghimireprakaash/Nepse-All-Scraper.git
cd Nepse-All-Scraper

# 2. Install dependencies
pip install requests beautifulsoup4

# 3. Run the full price history scrape (one-time, takes 2-4 hours)
python scraper/run_daily.py --full-scrape

# 4. Push everything to the repo
git add data/
git commit -m "chore: initial data load"
git push
```

From that point on, the daily GitHub Actions workflow keeps the floorsheet updated. Use the price, dividend, and right-share scripts manually when you want to refresh those historical datasets. ✅

---

## 🗺️ Roadmap

- [x] **Phase 1** — NEPSE Scraper (prices, dividends, right shares, floorsheet)
- [x] **Phase 2** — GitHub Actions daily floorsheet automation + manual incremental price updates
- [ ] **Phase 3** — Frontend / API layer

Want to help build Phase 3? **PRs are welcome.**

---

## ⚠️ Disclaimer

> This project is for **educational purposes only**.  
> Data is sourced from publicly available websites (ShareSansar, Merolagani).  
> Not financial advice. Do your own research before making investment decisions.

---

<div align="center">

Made with ❤️ for the Nepali investor community

</div>