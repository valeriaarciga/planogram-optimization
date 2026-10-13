# 🛒 Planogram Optimization

**Deciding which shelf and position each product gets in a convenience-store cooler**, balancing visibility of high-value products against physical space and disruption to the current layout. Includes a greedy heuristic, an exact MILP in Gurobi, and an interactive Streamlit dashboard.

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![Gurobi](https://img.shields.io/badge/solver-Gurobi-EE3524)
![Streamlit](https://img.shields.io/badge/dashboard-Streamlit-FF4B4B?logo=streamlit&logoColor=white)

> Course project for **MA2008 (Tecnológico de Monterrey)** with store data.

---

## The problem

A planogram assigns every product of a store segment (e.g. cooler furniture *BCO / CF*) to a **shelf** and a **position**. A good one:

- puts high-value products on **premium (eye-level) shelves**,
- **fits horizontally** on each shelf, including a separation between products,
- **stays close to the current layout**, because every move costs store labor.

Business rule confirmed with the data: tall products (e.g. family-size sodas) always go on the bottom shelf, which has plenty of vertical clearance, so height and depth constraints are intentionally omitted.

## Model

Decision variables: `x[p,c] = 1` if product *p* goes on shelf *c*, plus shelf and position deviations from the historical layout and a slack for over-use of shelf width.

```
minimize   α · Σ |shelf change|  +  β · Σ |position change|
         − γ · Σ value_p · premium_c · x[p,c]
         + λ · Σ width over-use_c
```

subject to each product on exactly one shelf, horizontal shelf capacity (with separation δ), and linearized absolute deviations. Product value is estimated by Monte Carlo from historical data.

**Two solution paths:**

1. **Greedy heuristic** — rank products by visibility score, assign each to the feasible shelf closest to its historical one, pack with **First-Fit Decreasing** bin packing, and order positions with **2-opt**.
2. **MILP (Gurobi)** — the exact model above, warm-startable from the heuristic, with configurable time limit and MIP gap.

A **sensitivity analysis** mode sweeps the weights (α, β, γ, λ) to show how much layout disruption buys how much visibility.

## Dashboard

`dashboard_planograma.py` is a Streamlit app with four tabs:

| Tab | Content |
|---|---|
| General results | KPIs and the optimized shelf layout rendered with product images |
| Data explorer | Filter and download the resulting planogram |
| Scenario simulator | Change the weights and compare layouts |
| Product glossary | Product catalog with images |

## Repository structure

```
planograma_gurobi (2).py                 # Heuristic + Gurobi MILP + sensitivity analysis (CLI)
planograma_streamlit_backend vs2.ipynb   # Backend notebook: cleaning, FFD, bin packing, 2-opt, CSV export per segment
dashboard_planograma.py                  # Streamlit dashboard
image_manager.py                         # Product image lookup for the dashboard
static/images/products/                  # Product images
requirements.txt
```

## Running it

```bash
pip install -r requirements.txt
pip install gurobipy                     # for the MILP (needs a Gurobi license)

# Heuristic only
python "planograma_gurobi (2).py" --heuristica-solo

# MILP for one segment, with weights and time limit
python "planograma_gurobi (2).py" --segmento BCO --gamma 5 --tiempo 300

# Sensitivity analysis
python "planograma_gurobi (2).py" --sensibilidad --heuristica-solo

# Dashboard (expects output_planograma_todos.csv next to the script)
streamlit run dashboard_planograma.py
```

> The input data (`df_filtered.csv`) and the dashboard's output CSV are not included because they come from the business partner.

## Team

Team project for MA2008, Tecnológico de Monterrey: **Valeria Arciga Valencia**, Brisma Alvarez, Ximena Montes, Emiliano Ruiz.
