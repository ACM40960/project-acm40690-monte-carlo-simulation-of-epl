<div align="center">
  <img src="images/logo.png" alt="Project Logo" width="150" height="150" />
  <h1>Premier League Monte Carlo — Bivariate Poisson + Elo</h1>
</div>

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![NumPy](https://img.shields.io/badge/NumPy-Latest-blue)
![Pandas](https://img.shields.io/badge/Pandas-Latest-blue)
![SciPy](https://img.shields.io/badge/SciPy-Latest-blue)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Latest-blue)
![Seaborn](https://img.shields.io/badge/Seaborn-Latest-blue)
![License](https://img.shields.io/badge/License-MIT-brightgreen)
![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey)
![Stars](https://img.shields.io/github/stars/your-org/your-repo?style=social)

A notebook/script that models football matches with a **Bivariate Poisson** (shared component) and uses **Elo** as a covariate, then runs **Monte Carlo** to produce full-season distributions (points, positions, title/top-4/relegation probabilities). Includes lightweight backtesting.

---

## Table of Contents

1. [Overview](#overview)  
2. [Project Structure](#project-structure)  
3. [Installation](#installation)  
4. [Data](#data)  
5. [Quickstart](#quickstart)  
6. [What It Does](#what-it-does)  
7. [Outputs & Visualizations](#outputs--visualizations)  
8. [Configuration](#configuration)  
9. [Troubleshooting](#troubleshooting)  
10. [Roadmap / Future Work](#roadmap--future-work)  
11. [Citing & Background](#citing--background)  
12. [Contributing](#contributing)  
13. [License](#license)  
14. [Contact](#contact)

---

## Overview

- **Scoring model:** Bivariate Poisson with a shared latent component \( \lambda_3 \) to capture goal dependence.  
- **Team effects:** attack (α), defence (β), home advantage (η) with **ridge** regularization and sum-to-zero constraints on α and β.  
- **Elo with decay:** pre-match `HomeElo` / `AwayElo`; Elo ratio scales goal rates via exponent **γ**.  
- **Monte Carlo:** simulate full seasons to estimate distributions over points, positions, and key outcomes.  
- **Backtests:** compare simulated mean points vs actual to get **MAE**, using past fixtures and points tables.

---

## Project Structure

```plaintext
your-repo/
├── data/
│   ├── E0_19_20.csv
│   ├── E0_20_21.csv
│   ├── E0_21_22.csv
│   ├── epl_22_23_fixtures.csv
│   ├── epl_23_24_fixtures.csv
│   ├── epl_24_25_fixtures.csv
│   └── epl_25_26_fixtures.csv
├── points/
│   ├── epl_2022_23_points.csv
│   └── epl_2023_24_points.csv
├── images/                 # optional: saved figures / logo
│   ├── heatmap.png
│   ├── boxplot.png
│   └── outcomes.png
├── trial_project.ipynb     # notebook (optional; mirrors the script)
├── trial_project.py        # ★ main script with CLI entrypoint
├── README.md
└── LICENSE
```

> Adjust file names if yours differ; the script reads the lists declared at the top (see [Configuration](#configuration)).

---

## Installation

Use a clean environment (venv or conda).

```bash
# clone
git clone https://github.com/your-org/your-repo.git
cd your-repo

# create & activate venv
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

**Example `requirements.txt`:**
```txt
numpy>=1.24
pandas>=2.0
scipy>=1.10
matplotlib>=3.7
seaborn>=0.13
jupyter
```

---

## Data

We use football-data style match CSVs with:

- `Date` (DD/MM/YYYY or parseable), `HomeTeam`, `AwayTeam`, `FTHG` (home goals), `FTAG` (away goals)

Example:
```csv
Date,HomeTeam,AwayTeam,FTHG,FTAG
10/08/2019,Liverpool,Norwich,4,1
10/08/2019,West Ham,Man City,0,5
```

Backtest “actual points” files:

- Columns: `Team,Points` (the script renames `Points` → `ActualPts` internally).
```csv
Team,Points
Manchester City,91
Arsenal,89
...
```

Backtest/forecast fixture files:

- Columns: `HomeTeam,AwayTeam` (no dates required for simulation).

> **Name consistency:** Ensure team names match across *all* files (training, fixtures, points). Simple mismatches (“Man City” vs “Manchester City”) will degrade results.

---

## Quickstart

**Option A — Script (recommended for repeatable runs)**

```bash
python trial_project.py
```

What it does:

1. Loads training seasons (2019/20–2021/22).
2. Builds Elo with decay, fits the bivariate Poisson model.
3. Tunes hyper-parameters via backtests (22/23 and 23/24).
4. Simulates future seasons (24/25 and 25/26), prints predicted tables, and shows plots.

**Option B — Notebook**

Open **`trial_project.ipynb`** and run cells top-to-bottom:
- Load & clean data → Compute Elo → Fit model → Backtest → Simulate → Plot.

---

## What It Does

- **Data engineering**
  - Parse dates, cast goals to `int`, sort chronologically.
  - Collect all unique teams from training + fixture files to ensure complete parameter vectors.

- **Elo (with decay)**
  - Baseline 1500 per team; update per match with
    - K-factor grid (e.g., 20/30/40), decay factor (e.g., 0.995).
  - Expected home result \( E_h=\frac{1}{1+10^{(R_a-R_h)/400}} \); update both teams with a decayed step.

- **Model fit (Bivariate Poisson)**
  - For match \( (h,a) \):
    - \( \lambda_1 = \exp(\alpha_h - \beta_a + \eta_h)\cdot(\text{Elo}_h/\text{Elo}_a)^\gamma \)
    - \( \lambda_2 = \exp(\alpha_a - \beta_h)\cdot(\text{Elo}_h/\text{Elo}_a)^{-\gamma} \)
    - \( \lambda_3 = \exp(\theta) \) shared component
  - Penalized log-likelihood with ridge on \( \alpha,\beta \), fitted via **BFGS** with zero-mean constraints on \( \alpha \) and \( \beta \).

- **Hyper-parameter search**
  - Small grid for speed: `K ∈ {20,30,40}`, `γ ∈ {0.04,0.06}`, `λ_ridge = 0.02`.
  - Backtests on two seasons; objective = average **MAE** between simulated mean points and actual points.

- **Monte Carlo**
  - Sample k ~ Poisson(λ3), x ~ Poisson(λ1), y ~ Poisson(λ2); score = (x+k, y+k).
  - Simulate each fixture list **N** times; aggregate to points tables and rank distributions.

- **Reproducibility**
  - Simulation RNGs seeded: `_rng = np.random.default_rng(1)` for match sims; a separate `default_rng(0)` for tie-break jitter in ranking.

---

## Outputs & Visualizations

- **Predicted table** (printed): median simulated points for the current league cohort.
- **Points distribution** per team: horizontal **boxplots**.
- **Finish-position probability heatmap**
  - Positions 1…N left→right; **darker** = higher probability; best teams at the **top**.
- **Outcome probabilities** bars
  - `P(Title)`, `P(Top-4)`, `P(Relegation)`.

If you save figures, a typical set might live in `images/`:
- `boxplot.png`, `heatmap.png`, `outcomes.png`.

---

## Configuration

All file paths and knobs live at the **top of `trial_project.py`** (and mirrored in the notebook):

```python
from pathlib import Path

ROOT = Path(__file__).resolve().parent if "__file__" in globals() else Path.cwd().resolve()
DATA_DIR   = ROOT / "data"
POINTS_DIR = ROOT / "points"

training_csvs = [DATA_DIR / "E0_19_20.csv", DATA_DIR / "E0_20_21.csv", DATA_DIR / "E0_21_22.csv"]
backtest_fixtures_csvs   = [DATA_DIR / "epl_22_23_fixtures.csv", DATA_DIR / "epl_23_24_fixtures.csv"]
backtest_actual_pts_csvs = [POINTS_DIR / "epl_2022_23_points.csv", POINTS_DIR / "epl_2023_24_points.csv"]
future_fixtures_csvs     = [DATA_DIR / "epl_24_25_fixtures.csv", DATA_DIR / "epl_25_26_fixtures.csv"]

# Monte Carlo simulations per season
N_SIMS = 500
```

Model / search defaults (inside the code):

- Elo decay: `0.995` (effective step ≈ `(1 - decay) * K`)
- Search grids (for speed; expand for final tuning):
  - `K ∈ {20, 30, 40}`
  - `γ ∈ {0.04, 0.06}`
  - `λ_ridge = 0.02`

**Tips**
- Runtime scales roughly **linearly** with `N_SIMS`.
- Keep `N_SIMS` small while iterating; bump for final figures.

---

## Troubleshooting

- **Seaborn theming**  
  Use:
  ```python
  import seaborn as sns
  sns.set_theme(style="whitegrid", rc={"figure.dpi": 120})
  ```

- **Pandas `observed` warning**  
  For `pivot_table`, pass `observed=False` (already set), or switch to `.pivot()` if categories are fixed.

- **Plots not showing**  
  In some environments, add `%matplotlib inline` at the top of a notebook.

- **Paths / working directory**  
  The script uses project-relative paths via `pathlib.Path`. Run from the repo root (`cd your-repo`) or adjust `ROOT`.

- **Team name mismatches**  
  Make sure training, fixtures, and points files use identical team strings.

---

## Roadmap / Future Work

- Dixon–Coles **time decay** in the likelihood (down-weight older matches).
- More covariates: injuries, transfers, schedule congestion (rest days).
- Bayesian / hierarchical variants for team effects and \( \lambda_3 \).
- Calibration checks (50/80/95% interval coverage) & reliability plots.
- Expanded hyper-parameter search and cross-league support.

---

## Citing & Background

- Dixon & Coles (1997). *Modelling Association Football Scores and Inefficiencies in the Football Betting Market*. **JRSS C** 46(2): 265–280. DOI: **10.2307/2986290**.  
- Related reading: https://royalsocietypublishing.org/doi/10.1098/rsos.210617

If this repo helps you, please ⭐ the project.

---

## Contributing

PRs welcome — loaders, metrics (rank corr, Brier), visual polish, or tuning refactors.

**Steps**
1. Fork  
2. Create a branch  
3. Commit changes with examples  
4. Open a PR

---

## License

Released under the **MIT License**. See [LICENSE](LICENSE).

---

## Contact

Questions or ideas? Open an issue or email **your.email@domain.com**.
