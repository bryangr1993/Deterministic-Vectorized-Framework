# Vectorized Macro-Micro Framework for ORB Strategies

This repository contains the source code, datasets, and forensic tools associated with the research paper: **"A Deterministic Vectorized Macro-Micro Framework for Tick-Validated Backtesting of Opening Range Breakout Strategies"**.

This framework implements a high-performance vectorized backtest on EURUSD (24-month period) to evaluate over 338,000 strategy configurations. It features a **Validation Gating** mechanism that resolves intra-bar ambiguity via deterministic tick-level validation, achieving a **~60x speedup** versus standard iterative methods.

---

## Language & Localization Note (Nota de Idioma)
**Please Note:** While this documentation is in English to support Open Science standards, the source code artifacts and dataset column headers preserve their original **Spanish** naming conventions.
* **Code:** The notebook `Trader_Pro_2.1-LEGEND.ipynb` contains comments and logic descriptions in **Spanish**.
* **Data:** CSV column headers use Spanish terminology (e.g., `Margen_Utilidad` instead of *Efficiency Margin*, `Ticks_totales` instead of *Net Ticks*).
* **Functionality:** Despite the language difference, the code is fully functional and the logic is mathematically universal. A **Data Dictionary** is provided below.

---

## Experimental Protocol (The "Clean Split")

To ensure rigorous validation and prevent data leakage, this study enforces a strict three-phase chronological protocol spanning **December 2023 to November 2025 (~62.5M ticks)**.

| Phase | Period | Duration | Role |
| :--- | :--- | :--- | :--- |
| **I. In-Sample** | Dec 2023 – Nov 2024 | 12 Months | **Optimization:** Exhaustive Grid Search (338,688 configs). |
| **II. Validation** | Dec 2024 – Mar 2025 | 4 Months | **Calibration:** Selection of Top 17 candidates & freezing of Structural Orientation ($\theta \in \{1, -1\}$). |
| **III. Pristine Test** | Apr 2025 – Nov 2025 | 8 Months | **Blind Evaluation:** Pure out-of-sample test with all parameters frozen. |

---

## Repository Structure

### 1. Code (`/code`)
* **`Trader_Pro_2.1-LEGEND.ipynb`** (Main Engine):
    * The core vectorized backtesting engine. It performs the exhaustive Grid Search (Phase I), applies the Macro (OHLC) masks, and manages the initial data processing.
* **`Forensic_Audit_Tool.ipynb`**:
    * The diagnostic validation tool. It performs:
        * **Speed Benchmark:** Comparing Vectorized vs. Iterative performance.
        * **Ambiguity Audit:** Detecting divergence in "Bottom-500" vs. Top strategies.
        * **Pristine Test:** Executing the Bidirectional Portfolio on future data.

### 2. Datasets (`/data`)
* **`todas_estrategias.csv`** (Phase I Output):
    * The massive dataset containing metrics for all 338,688 configurations during the In-Sample period.
* **`Top_17_Pareto.csv`** (Phase II Selection):
    * The parameters of the final 17 selected strategies, including their calibrated orientation ($\theta$).
* **`Validation_Holdout_Results.xlsx`**:
    * Performance metrics during the Calibration Phase (Dec 24 - Mar 25).
* **`Top_50_Volume.csv`** & **`bottom 500.csv`**:
    * Subsets used for the Forensic Audit (Positive/Negative controls).

*Note: Due to data licensing restrictions, raw tick data (`.parquet` / `.csv` ticks) are not redistributed. The repository provides the schema and loaders to reproduce experiments using an equivalent EURUSD tick source.*

---

## Key Results (Pristine Test Phase)

### 1. Computational Efficiency
A controlled benchmark on the 8-month pristine test set (20M+ ticks) demonstrated that the Vectorized Framework is significantly faster than standard iterative approaches:
* **Iterative Loop:** ~25.0 seconds.
* **Vectorized Framework:** ~0.40 seconds.
* **Speedup Factor:** **57x – 65x**.

### 2. Structural Persistence (Bidirectional Portfolio)
On the blind test set (Apr–Nov 2025), the selected portfolio demonstrated robust structural behavior:
* **Directional Persistence:** **76.5%** (13 of 17 strategies maintained their In-Sample bias).
* **Adjusted Return:** By applying the frozen orientation ($\theta$), the portfolio generated **+7,405 Net Ticks** (vs. -13,715 if orientation were ignored).
* **Divergence:** **0.00%** ambiguity rate observed throughout the test period.

---

## Data Dictionary (Spanish to English Mapping)

| Spanish Column Header | English Equivalent | Description |
| :--- | :--- | :--- |
| `Ticks_totales_estrategia` | **Net Ticks** | Cumulative profit/loss in points. |
| `Margen_Utilidad` | **Efficiency Margin** | Normalized efficiency metric derived from Profit Factor. |
| `TPs_alcanzados` | **TP Hits** | Number of trades that hit Take Profit. |
| `SLs_alcanzados` | **SL Hits** | Number of trades that hit Stop Loss. |
| `TP_Ticks` | **Take Profit (pts)** | Distance to TP in points/ticks. |
| `SL_ticks` | **Stop Loss (pts)** | Distance to SL in points/ticks. |
| `hora_apertura` | **Opening Time** | The anchor hour for the session (UTC-5). |
| `v_rango` | **Range Duration** | Duration of the opening range (in 5-min bars). |
| `v_espera` | **Wait Window** | Duration of the valid breakout window (in 5-min bars). |

---

## How to Reproduce

1.  **Environment:** Ensure Python 3.10+ is installed with `pandas`, `numpy`, `tqdm`, and `joblib`.
2.  **Data Setup:**
    * Place your tick data (CSV or Parquet) in the expected folder structure (see `Forensic_Audit_Tool.ipynb` for schema details).
    * If `todas_estrategias.csv` is compressed, unzip it first.
3.  **Run the Benchmark:**
    * Open `Forensic_Audit_Tool.ipynb`.
    * Run the "Speed Benchmark" cell to verify the ~60x efficiency gain.
4.  **Run the Blind Test:**
    * Execute the "Portfolio Validation" function to reproduce the Adjusted PnL and Persistence metrics on the 2025 data.
