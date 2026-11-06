# Nepsy Scraper

Scrapes NEPSE market data from ShareSansar and Merolagani for 365 priority companies.
The scheduled GitHub Actions workflow updates the daily floorsheet. Price, dividend, and right-share histories are retained in `data/` and can be refreshed manually.

> Originally based on [SamirWagle/Nepse-All-Scraper](https://github.com/SamirWagle/Nepse-All-Scraper).
> See [../CREDITS.md](../CREDITS.md) for provenance and attribution.

---

## 📂 File Structure

```
scraper/
├── run_github_actions.py   # Floorsheet entry point (dividend/right-share modes retained)
├── run_daily.py            # Local price update CLI
└── core/
    ├── daily.py            # Orchestrates price scraping
    ├── daily_prices.py     # Daily price summary updater
    ├── floorsheet.py       # Legacy standalone floorsheet scraper
    └── history.py          # OHLC price history scraper (incremental)
```

---

## 🤖 GitHub Actions (Automated Daily)

Runs **every day at 6:00 PM Nepal time** via `.github/workflows/daily_scraper.yml`.

```
floorsheet → data/floorsheet/floorsheet_YYYY-MM-DD.csv
```

The workflow uses Merolagani's actual market date, so weekend and holiday runs do not create fake files. It skips an existing day when the new scrape has fewer rows and deduplicates contract numbers. Dividend, right-share, price, and index steps are intentionally disabled; their code and historical files remain available for manual runs.

All output is committed and pushed back to the repo automatically when data changes.

---

## 🛠️ Usage

### Install

```bash
pip install requests beautifulsoup4
```

### Floorsheet (used by GitHub Actions)

```bash
python scraper/run_github_actions.py --floorsheet

# Preview only — does not write data
python scraper/run_github_actions.py --floorsheet --max-pages 3
```

### Optional historical scrapers (not scheduled)

```bash
python scraper/run_github_actions.py --dividends
python scraper/run_github_actions.py --right-shares
```

### Price history (run locally)

```bash
# Incremental update — only fetches records newer than what's in prices.csv
python scraper/run_daily.py --incremental

# Full scrape — downloads complete history for all companies (~2-4 hours first time)
python scraper/run_daily.py --full-scrape

# Only process new companies (new IPOs/listings)
python scraper/run_daily.py --new-only
```

---

## 📊 Output Format

### `data/company-wise/{SYMBOL}/prices.csv`
```
date, open, high, low, ltp, percent_change, qty, turnover
```

### `data/company-wise/{SYMBOL}/dividend.csv`
```
fiscal_year, bonus_share, cash_dividend, total_dividend, book_closure_date
```

### `data/company-wise/{SYMBOL}/right-share.csv`
```
ratio, total_units, issue_price, opening_date, closing_date, status, issue_manager
```

### `data/floorsheet/floorsheet_YYYY-MM-DD.csv`
```
date, sn, contract_no, stock_symbol, buyer, seller, quantity, rate, amount
```

---

## ⚙️ How Incremental Price Scraping Works (manual)

`history.py` reads the latest date from `prices.csv`, then passes it as a `stop_date` to the AJAX paginator. The moment it encounters a record older than `stop_date`, it stops fetching — so a daily update fetches only 1-2 pages instead of 70+ pages.

---

## 📝 First-Time Setup

1. Run full price scrape locally (once):
   ```bash
   python scraper/run_daily.py --full-scrape
   ```
2. Commit all data to the repo:
   ```bash
   git add data/
   git commit -m "chore: initial data load"
   git push
   ```
3. GitHub Actions will keep the daily floorsheet updated from then on. Run the price, dividend, or right-share commands manually to refresh those historical datasets.
