![Project Logo](images/logo.png)

# Premier League Monte Carlo — Bivariate Poisson + Elo

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
