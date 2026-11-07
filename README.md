# Deep Learning for Commodities Forecasting

This repository currently implements the OHLCV data pipeline for forecasting 3–4 week price direction for WTI crude oil, gold, wheat, and copper. It downloads daily futures data from Yahoo Finance, cleans it, flags missing observations, creates forward targets, and makes chronological train/validation/test splits.

News sentiment, contextual imputation, forecasting models, evaluation metrics, and a user interface are planned, but are not implemented yet.

**Team:** P10_T2
**Tutor:** Charaf LOUHMADI  
**Constraints:** Python only · no Streamlit

## Quick start

The project requires Python 3.12 or newer. The dependency list in `pyproject.toml` is not yet complete: the data pipeline also needs PyYAML, PyArrow, and yfinance, and the tests need pytest. Add those dependencies before syncing the environment:

```powershell
uv add pyyaml pyarrow yfinance
uv add --dev pytest
uv sync
$env:PYTHONPATH = "src"

uv run pytest

uv run python -m commodities.data.download
uv run python -m commodities.data.clean
uv run python -m commodities.data.split
```

All settings (assets, dates, horizon, split) live in `configs/assets.yaml`.

## Layout

```text
configs/assets.yaml          Pipeline settings: assets, dates, horizon, and split
docs/data_contract.md        OHLCV dataset format and rules
src/commodities/
  config.py                  Loads project configuration
  data/
    download.py              Downloads raw daily OHLCV data
    clean.py                 Cleans data and flags missing business days
    features.py              Creates forward-return targets
    split.py                 Creates chronological splits and provides a train-only scaler
  imputation/                 Planned; not implemented
  models/                     Planned; not implemented
  nlp/                        Planned; not implemented
  evaluation/                 Planned; not implemented
  ui/                         Planned; not implemented
tests/                        Offline tests for the implemented data pipeline
notebooks/                    Reserved for exploration
data/                         Generated data; not committed to git
```

## Data and leakage safeguards

The implemented pipeline and tests cover these safeguards:

1. Splits are chronological; rows are not shuffled.
2. Splits purge rows whose forward target reaches the next split.
3. `TrainOnlyScaler` fits statistics on training data and reuses them for validation and test.
4. Missing OHLCV values are flagged in the cleaned dataset and are not forward-filled.

Imputation models and train-only imputer fitting are not implemented yet. 
Run `uv run pytest` to test the current safeguards.


## Git workflow

- `main` is protected: changes go through a pull request with one reviewer.
- One branch per issue: `data/clean-calendar`, `nlp/finbert-baseline`, `ui/forecast-chart`, ...
- Write `Closes #<issue>` in the PR description so the project board updates automatically.
- Commit code and configs, never data or model weights.

## Libraries

| Library | Current use |
|---|---|
| pandas / numpy | Data manipulation and target calculation |
| PyYAML | Reads `configs/assets.yaml` |
| PyArrow | Reads and writes Parquet files |
| yfinance | Downloads daily futures data from Yahoo Finance |
| pytest | Runs the test suite |
