<div align="center">
  <img src="images/logo.png" alt="Project Logo" width="150" height="150" />
  <h1> Premier League Monte Carlo — Bivariate Poisson + Elo</h1>
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

A Jupyter notebook (`trial_project.ipynb`) that models football matches with a **Bivariate Poisson** (shared component) and uses **Elo** as a covariate, then runs **Monte Carlo** to produce full-season distributions (points, positions, title/top-4/relegation probabilities). Includes lightweight backtesting.

---

## Table of Contents

1. [Overview](#overview)  
2. [Project Structure](#project-structure)  
3. [Installation](#installation)  
4. [Data](#data)  
5. [Quickstart](#quickstart)  
6. [What the Notebook Does](#what-the-notebook-does)  
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
- **Team effects:** attack (α), defence (β), home advantage (η) with ridge regularization and sum-to-zero constraints.  
- **Elo with decay:** pre-match HomeElo/AwayElo; Elo ratio scales goal rates via exponent **γ**.  
- **Monte Carlo:** simulate full seasons to estimate distributions over points, positions, and key outcomes.  
- **Backtests:** compare simulated mean points vs. actual to get MAE.

---

## Project Structure

```plaintext
your-repo/
├── data/
│   ├── E0_19_20.csv
│   ├── E0_20_21.csv
│   ├── E0_21_22.csv
│   ├── E0_22_23.csv
│   ├── E0_23_24.csv
│   └── E0_24_25.csv
├── points/
│   ├── epl_2021_22_points.csv
│   └── epl_2022_23_points.csv
├── images/                 # optional: for README examples
│   ├── heatmap.png
│   ├── boxplot.png
│   └── outcomes.png
├── trial_project.ipynb     # ★ main notebook
├── README.md
└── LICENSE
```

​## Installation

Use a clean environment (venv or conda).

~~~bash
# clone
git clone https://github.com/your-org/your-repo.git
cd your-repo

# venv (Python)
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
~~~

**Example `requirements.txt`:**
~~~txt
numpy>=1.24
pandas>=2.0
scipy>=1.10
matplotlib>=3.7
seaborn>=0.13
jupyter
pyprojroot>=0.3   # optional, for here()-style paths
~~~

---

## Data

CSV columns expected (football-data style):

- `Date` (DD/MM/YYYY or parseable), `HomeTeam`, `AwayTeam`, `FTHG` (home goals), `FTAG` (away goals)

Example:
~~~csv
Date,HomeTeam,AwayTeam,FTHG,FTAG
10/08/2019,Liverpool,Norwich,4,1
10/08/2019,West Ham,Man City,0,5
~~~

Backtest “actual points” files:
~~~csv
Team,Points
Manchester City,91
Arsenal,89
...
~~~

Update the file lists at the top of the notebook/script (training, backtests, future fixtures).

---

## Quickstart

1. Open **`trial_project.ipynb`** in VS Code/Jupyter Lab/Notebook.  
2. Run cells top-to-bottom:
   - Load & clean data  
   - Compute Elo (with decay)  
   - Fit Bivariate Poisson  
   - Backtest (prints MAE)  
   - Simulate future seasons  
   - View plots inline

> Prefer a script? Run `python trial_project.py`. You can export the notebook to `.py` via *File → Save and Export As*.

---

## What the Notebook Does

- **Data engineering**: parse dates, cast goals to int, sort by date.  
- **Elo (decay)**: writes `HomeElo` / `AwayElo` per match; returns final team Elo.  
- **Model fit**:
  - team attack/defence + home advantage, shared component \( \lambda_3 \)  
  - ridge penalty on (α, β)  
  - Elo ratio \( (\text{Elo}_h/\text{Elo}_a)^\gamma \) enters \( \lambda_1, \lambda_2 \)  
- **Backtests**: simulate past seasons; compute **MAE** vs. actual points.  
- **Projections**: simulate future fixtures with **N** Monte Carlo seasons.

---

## Outputs & Visualizations

- **Predicted table** (median points)  
- **Points distribution** per team (horizontal **boxplots**)  
- **Finish-position probability heatmap**
  - Single hue (“Blues”): **dark = higher probability**  
  - Positions 1…N left→right; best teams at the **top**  
- **Outcome probabilities**
  - `P(Title)`, `P(Top-4)`, `P(Relegation)` horizontal bars

*Example (optional if you save figures):*
## Configuration

Minimal grid kept small for speed during development; expand for final tuning.

\```python
# example defaults inside the notebook/script
K = 40            # Elo K-factor
gamma = 0.06      # Elo exponent for λ1/λ2 scaling
lambda_ridge = 0.02
decay = 0.995     # effective Elo step ≈ (1 - decay) * K
N_SIMS = 1000     # Monte Carlo seasons (100–200 for quick runs)
\```

**Tips**
- Runtime scales roughly **linearly** with `N_SIMS`.
- Keep `N_SIMS` small while iterating; bump for final figures.

---

## Troubleshooting

- **Seaborn deprecation (`sns.set`)** → use:
  \```python
  import seaborn as sns
  sns.set_theme(style="whitegrid", rc={"figure.dpi": 120})
  \```
- **Pandas FutureWarning (`observed` in `pivot_table`)** → pass `observed=False` or use `.pivot()` instead of `.pivot_table()`.
- **Plots don’t show** → in some setups add `%matplotlib inline` at the top of the notebook.
- **Paths/working dir** → build paths from a project root (e.g., `pyprojroot.here()` or `pathlib.Path.cwd()`), or run `cd your-repo` before `python trial_project.py`.

---

## Roadmap / Future Work

- Dixon–Coles time decay in the likelihood (down-weight older games).
- More covariates: injuries, transfers, schedule congestion (rest days).
- Bayesian variants / hierarchical priors for team effects and shared component.
- Calibration checks (50/80/95% interval coverage) + reliability plots.
- Broader training window / cross-league support.

---

## Citing & Background

- Dixon & Coles (1997), *JRSS C* 46(2): 265–280 — football score modelling with correlated goals.

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


