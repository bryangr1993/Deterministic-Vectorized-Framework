# Vectorized Macro-Micro Framework for ORB Strategies

This repository contains the source code, datasets, and forensic tools associated with the research paper: **"A Deterministic Vectorized Macro-Micro Framework for Tick-Validated Backtesting of Opening Range Breakout Strategies"**.

This framework implements a high-performance vectorized backtest on EURUSD (16-month period) to evaluate over 338,000 strategy configurations, resolving intra-bar ambiguity via deterministic tick-level validation.

---

##  Language & Localization Note (Nota de Idioma)
**Please Note:** While the documentation in this README is in English, the source code artifacts and dataset column headers preserve their original **Spanish** naming conventions.
* **Code:** The notebook `Trader_Pro_2.1-LEGEND.ipynb` contains comments and logic descriptions in **Spanish**.
* **Data:** CSV column headers use Spanish terminology (e.g., `Margen_Utilidad` instead of *Efficiency Margin*, `Ticks_totales` instead of *Net Ticks*).
* **Functionality:** Despite the language difference, the code is fully functional, vectorized, and deterministic. A **Data Dictionary** is provided below to assist in interpreting the results.

---

##  Repository Structure

### 1. Code (`/code`)
* **`Trader_Pro_2.1-LEGEND.ipynb`** (Main Engine):
    * The core vectorized backtesting engine. It performs the exhaustive Grid Search, applies the Macro (OHLC) masks, and manages the initial data processing.
* **`Forensic_Audit_Tool.ipynb`**:
    * The diagnostic validation tool. It performs the stress tests ("Kamikaze"), the noise analysis ("Bottom-500"), and the final tick-by-tick certification of the selected strategies.

### 2. Datasets (`/data`)

#### A. Optimization Phase (In-Sample)
* **`todas_estrategias.csv`** (The Massive Dataset):
    * Contains the exhaustive evaluation of all **338,688 strategy configurations** over the **Training/In-Sample Period** (Dec 2023 – Nov 2024).
    * Used to map the fitness landscape and select candidates.
    * *Important:* This file represents the optimization phase; strictly no future data (Dec 2024+) is included here to prevent look-ahead bias.

#### B. Validation Phase (Out-of-Sample)
* **`Test_17_EURUSD.xlsx`**:
    * Performance results of the selected **Top 17 strategies** over the **Blind Holdout/Out-of-Sample Period** (Dec 2024 – Mar 2025).
    * Used to verify that the selected strategies maintained profitability in unseen market data.

#### C. Forensic Audit Subsets
* **`Top_50_Volume.csv`**:
    * The "Heavy Lifters". Contains the 50 strategies with the highest Net Ticks. Used to verify structural robustness (**0.00% Divergence**).
* **`top 500 efficiency margin.csv`**:
    * The "Naive Baseline". The top 500 strategies selected solely by efficiency. Used as a control group to demonstrate the risks of overfitting (referenced as the group with **non-zero divergence**).
* **`bottom 500.csv`**:
    * The "Negative Control". The 500 worst-performing strategies. Used to establish the baseline for market noise and ambiguity (**1.40% Divergence**).

#### D. Final Solution
* **`Top_17_Pareto.csv`**:
    * The parameters of the final proposed portfolio (Pareto Frontier + 30% Consistency Filter).

---

## Data Dictionary (Spanish to English Mapping)

To interpret the CSV files, please refer to this mapping table:

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
| `fecha_inicio` | **Start Date** | Start of the evaluation period. |
| `fecha_fin` | **End Date** | End of the evaluation period. |

---

## How to Reproduce the Findings

1.  **Environment:** Ensure you have Python 3.10+ installed with `pandas`, `numpy`, `tqdm`, and `joblib`.
2.  **Data Setup:**
    * If `todas_estrategias.csv` is compressed (.zip), extract it first.
    * Place all CSV files in a folder named `data` (or adjust the path in the notebook).
3.  **Running the Audit:**
    * Open `Forensic_Audit_Tool.ipynb`.
    * Load `Top_50_Volume.csv` or `Top_17_Pareto.csv`.
    * Execute the cells to verify the **0.00% Divergence Rate**.
4.  **Running the Stress Test:**
    * In the Audit tool, run the "Kamikaze" function (TP=3 ticks) to confirm that the detector correctly flags ambiguity when present.

---

##  Methodology Summary
* **Total Period:** Dec 1, 2023 – Mar 31, 2025 (16 Months).
* **Protocol:**
    * *In-Sample (Optimization):* Dec 2023 – Nov 2024 (Used for `todas_estrategias.csv`).
    * *Out-of-Sample (Validation):* Dec 2024 – Mar 2025 (Used for `Test_17_EURUSD.xlsx`).
* **Execution:** Parallelized grid search on 9 workers.
* **Validation:** 100% Tick-level replay for ambiguity resolution on gated windows.