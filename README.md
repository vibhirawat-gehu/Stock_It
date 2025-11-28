# Stock_It

Stock_It is a lightweight Python project that fetches stock price data and financial news, performs ETL (extract-transform-load) into a PostgreSQL database, and supports continuous real-time monitoring and sentiment analysis.

This README explains how to set up the project, create a `.env` file, prepare the PostgreSQL database, install dependencies, and run the scripts on Windows (PowerShell) and other platforms.

---

## Features

- Fetch historical and intraday stock data (Yahoo Finance and Alpha Vantage)
- Fetch financial news (NewsAPI, Marketaux)
- ETL pipeline to clean and load stock and news data into PostgreSQL
- Sentiment analysis integration (TextBlob / VADER)
- Real-time monitoring / continuous tracking mode
- Utility scripts to run ETL once or continuously

---

## Prerequisites

- Python 3.10+ installed
- PostgreSQL installed and running (for the database-backed features)
- (Optional) API keys if you want to use external providers:
  - `ALPHA_VANTAGE_API_KEY` (Alpha Vantage) — optional (project defaults to Yahoo Finance)
  - `NEWS_API_KEY` (NewsAPI.org) — optional
  - `MARKETAUX_API_KEY` (Marketaux) — optional

---

## Quick setup (Windows PowerShell)

Open PowerShell and run these commands from the project root (where `requirements.txt` and Python files live):

```powershell
# Create and activate a virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Upgrade pip and install dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# Some modules used by the code may not be present in requirements.txt (e.g., yfinance).
# If you see import errors, run:
pip install yfinance
```

---

## Create a `.env` file

Create a file called `.env` in the project root (same folder as `main.py`). This file stores non-public configuration values used by the project.

Example `.env` (DO NOT commit real secrets to GitHub):

```env
# PostgreSQL settings
DB_HOST=localhost
DB_PORT=5432
DB_NAME=stock_tracker_db
DB_USER=postgres
DB_PASSWORD=your_postgres_password

# Optional API keys (leave blank if not using)
ALPHA_VANTAGE_API_KEY=your_alpha_vantage_api_key
NEWS_API_KEY=your_newsapi_key
MARKETAUX_API_KEY=your_marketaux_api_key
```

Notes:
- The project will use `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD` for database operations. Defaults are used in code when not provided, but providing them in `.env` is recommended.
- Keep `.env` out of source control. Add `.env` to `.gitignore` if not already ignored.

---

## Database setup

1. Ensure PostgreSQL is installed and running on your machine.
   - Download: https://www.postgresql.org/download/
   - Create/remember a `postgres` user or other DB user and its password.

2. With your `.env` configured, run the project's setup script to create the database and tables:

```powershell
python setup_database.py
```

The script uses `sql/create_tables.sql` to create tables. If the script reports connection or authentication errors, verify the values in your `.env`.

Alternative: You can also run the SQL files directly using `psql` or your preferred DB tool:

```powershell
# Example using psql (PowerShell)
psql -h localhost -p 5432 -U postgres -f sql/create_tables.sql
```

---

## Running the project

Common entry points (run from project root with virtualenv activated):

- Run the ETL pipeline once (initial news load):

```powershell
python run_etl_pipeline.py
```

- Run a one-off news ETL and exit:

```powershell
python run_news_once.py
```

- Run the continuous stock tracking (long-running):

```powershell
python main.py
```

The continuous tracker writes logs to `continuous_tracker.log`. The ETL runner writes logs to `etl_pipeline.log`.

---

## Project layout (important files)

- `main.py` — continuous tracking / monitoring loop
- `run_etl_pipeline.py` — initializes ETL, DB and runs continuous pipeline
- `run_news_once.py` — run news ETL one time and exit
- `setup_database.py` — create database and tables
- `etl_pipeline.py` — ETL orchestration for stock and news data
- `database_models.py` — SQLAlchemy/DB manager and models
- `alpha_vantage_fetcher.py`, `yahoo_finance_fetcher.py` — stock data sources
- `news_api_fetcher.py`, `marketaux_news_fetcher.py` — news fetchers
- `sentiment_analyzer.py` — sentiment scoring utilities
- `sql/` — SQL definitions: `create_tables.sql`, `create_database.sql`

---

## Environment variables summary

- `DB_HOST` — PostgreSQL host (default `localhost`)
- `DB_PORT` — PostgreSQL port (default `5432`)
- `DB_NAME` — Database name (default `stock_tracker_db`)
- `DB_USER` — DB username (default `postgres`)
- `DB_PASSWORD` — DB password (no default)
- `ALPHA_VANTAGE_API_KEY` — (optional) Alpha Vantage API key
- `NEWS_API_KEY` — (optional) NewsAPI.org API key
- `MARKETAUX_API_KEY` — (optional) Marketaux API key

---

## Troubleshooting

- Import errors for a missing module: install it with `pip install <module>` (e.g., `yfinance`).
- Database authentication or connection failures: ensure PostgreSQL is running and `.env` values are correct.
- API fetch errors / rate limits: API providers (Alpha Vantage, NewsAPI, Marketaux) often have rate limits — use keys responsibly, and consider switching to free data sources like Yahoo Finance when possible.

---

## Next steps / Suggested improvements

- Add `yfinance` to `requirements.txt` (used by `yahoo_finance_fetcher.py`).
- Add a `.gitignore` entry for `.env` and database or log artifacts if not present.
- Add unit tests and CI (GitHub Actions) for ETL and fetchers.
- Add Docker support for easier, portable DB + app setup.

---

If you'd like, I can also:

- Add `README.md` to the repository (done) and optionally update `requirements.txt` to include `yfinance`.
- Create a `.env.example` file with the template above.
- Add a short PowerShell script to automate virtualenv + install + DB setup.
