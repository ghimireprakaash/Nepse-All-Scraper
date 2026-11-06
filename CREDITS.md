# Credits & Provenance

## Original project

This repository is a derivative of **[SamirWagle/Nepse-All-Scraper](https://github.com/SamirWagle/Nepse-All-Scraper)**, originally created by **Samir Wagle** ([@SamirWagle](https://github.com/SamirWagle)).

- Original repository: <https://github.com/SamirWagle/Nepse-All-Scraper>
- Original live site: <https://samirwagle.github.io/Nepse-All-Scraper/>
- Original author's website: <https://samirwagle.com.np>

The original scraper, data pipeline, GitHub Actions automation, and static JSON API design are his work.

## This repository

Maintained by **Prakash Ghimire** ([@ghimireprakaash](https://github.com/ghimireprakaash)).

Changes made in this derivative include:

- Consolidated the duplicate ShareSansar ID mappings into a single `data/company_id_mapping.json` (union of the original mapping files).
- Price update scripts use incremental-only mode (`--skip-new`) so a manual run never starts a multi-hour full-history scrape.
- Capped first-run index bootstrap to 30 days and added a 120-minute job timeout.
- Unified floorsheet output under `data/floorsheet/` as CSV only.
- Fixed `build_api.py` when `data/company-wise/` does not exist yet.
- Sorted per-symbol price JSON oldest-to-newest so `latest.json`, snapshots, and `data[-1]` return the actual latest row.
- Seeded the upstream price, dividend, right-share, and index history (the full floorsheet archive was excluded).
- Enabled GitHub Pages publishing for the generated `docs/` site.
- Switched the scheduled workflow to a floorsheet-only daily run. Price, dividend, right-share, and index steps remain in the workflow as disabled/commented steps; their historical data, API outputs, and scraper code are preserved.

## Current scheduled behavior

The GitHub Actions workflow is **floorsheet-only** and runs every day at 6:00 PM Nepal time. It saves under Merolagani's actual market date, deduplicates contract numbers, and refuses to replace an existing day with a smaller scrape. Prices, dividends, right shares, and indices are no longer refreshed by the schedule; their historical CSVs, API endpoints, and scraper code are retained for manual runs.

## Data provenance

On 2026-11-06, the historical `data/company-wise/` and `data/indices/` files were seeded from the original repository to preserve the upstream data contribution. The full upstream floorsheet history (about 690 MB) was **not** imported because of repository size limits; floorsheet data in this repository starts from the first run of this fork's workflow.

All new floorsheet data after the seed date is scraped by this repository's own GitHub Actions workflow. The seeded price, dividend, right-share, and index data remains as a historical snapshot unless refreshed manually.

## License status

No `LICENSE` file exists in the original repository. The original README carries an "Educational" badge, but that is not a formal software license, so this derivative does not claim to relicense the original work.

If you plan to reuse this project beyond educational/personal use, please contact the original author to clarify licensing. A `LICENSE` file can be added here once the original author grants permission.

## Syncing with upstream

```bash
git remote add upstream https://github.com/SamirWagle/Nepse-All-Scraper.git
git fetch upstream
git merge upstream/main
```

Upstream is actively maintained, so review changes before merging.
