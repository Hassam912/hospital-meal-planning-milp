# Hospital Meal Planning — Mixed-Integer Linear Program

> Plans a 3-day hospital menu that is least-cost, clinically compliant **and** varied
> enough that patients keep eating — then puts a dollar figure on what that variety costs.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat&logo=scipy&logoColor=white)
![HiGHS](https://img.shields.io/badge/solver-HiGHS-0b6b58?style=flat)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?style=flat&logo=plotly&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat&logo=numpy&logoColor=white)

📄 **[Full write-up — business framing, constraint design, and the cost-of-variety result](https://hassam912.github.io/projects/hospital-meal-planning-milp)**

---

## At a glance

| | |
|---|---|
| **Context** | MMA 861 — Analytical Decision Making, Queen's Smith School of Business |
| **My role** | Model formulation, constraint design, sensitivity analysis |
| **Problem type** | Constrained optimization (MILP) |
| **Scale** | 41 ingredients × 9 meals × 5 clinical profiles, 11 constraint families |
| **Headline result** | Enforcing menu rotation costs **+37% on food spend** — quantified, not guessed |

---

## The problem

A hospital food service manager is asked to do three incompatible things at once:

1. **Minimise ingredient cost** — food is a large line item on a fixed budget.
2. **Meet strict clinical nutrition bounds** — daily *and* per-meal. A diabetic patient
   needs 45–60g of carbs **per meal**, not just within a daily total.
3. **Keep menus varied** — repetition causes food fatigue, and a patient who stops
   eating is a clinical problem, not a catering one.

Manual planning fails on at least one of the three, and — more importantly — gives the
manager **no number** for what trading one against another actually costs.

## Approach

**Decision variables**

| Variable | Type | Meaning |
|---|---|---|
| `x[i,j]` | Binary | Is ingredient *i* served in meal *j*? |
| `q[i,j]` | Continuous | Portion size in grams |

Linked by `qMin·x ≤ q ≤ qMax·x`, so an unselected ingredient is forced to exactly zero
grams and a selected one lands in a realistic serving range.

**Objective:** minimise total ingredient cost.

**11 constraint families** — each one added because the solver did something absurd
without it:

| Constraint | Closes the loophole |
|---|---|
| Daily nutrient bounds | "Cheapest" would otherwise mean nutritionally void |
| Per-meal calorie split (35/40/25) | Solver dumps all calories into one giant meal |
| Diabetic per-meal carb window | Daily bounds alone allow one carb spike |
| Meal composition rules | Nutritionally valid ≠ recognisable as breakfast |
| Culinary incompatibilities | A solver has no palate — it will pair granola with salmon |
| Rotation limits | Otherwise the same cheap ingredients repeat every day |

**Sensitivity analysis.** Integer programs don't yield clean shadow prices, so the
optimal binary selections are fixed from the MILP, then an **LP relaxation** runs on the
continuous portion variables — recovering usable dual values on the nutrient constraints
while respecting the discrete menu the MILP chose.

## Results

Each of five clinical profiles was solved three ways — cost-only, and two rotation
strictness levels — to isolate the price of variety:

| Profile | Cost-only | Max 3-day rotation | Max 2-day rotation |
|---|---|---|---|
| Normal Male | $7.05 | $9.93 | $9.81 |
| Normal Female | $7.31 | $10.67 | $10.67 |
| Diabetic | $7.88 | $9.57 | $9.30 |
| High Cholesterol | $5.75 | $8.28 | $8.28 |
| DASH | $6.06 | $8.63 | $8.63 |

*(3-day food cost per patient, CAD)*

**Guaranteeing no ingredient repeats more than once every two days costs ~37% more**
and roughly doubles distinct-ingredient count. That single number reframes the
conversation: a manager arguing for variety is no longer making a soft appeal.

## Repo contents

| File | What's in it |
|---|---|
| `Meal_Planner.ipynb` | Full model — data loading, constraint construction, all solver runs, LP-relaxation sensitivity analysis, result charts |
| `Meal_Planner_Data.xlsx` | Ingredient nutrient-density matrix, clinical profile bounds, culinary-incompatibility rules |
| `MILP_Model_Results.xlsx` | Solver output — cost, ingredient diversity and per-nutrient compliance for every profile × solver run |

## Running it

```bash
pip install numpy scipy openpyxl plotly pandas
jupyter notebook Meal_Planner.ipynb
```

Both `.xlsx` files load from the repo directory — no path edits needed. Solve time is
capped at 180s per run; all five profiles × three solvers complete in a few minutes on a
laptop.

## Data sources

| Input | Source |
|---|---|
| Nutrient profiles per gram | USDA food databases (public) |
| Clinical min/max intake | Health Canada guidelines, scaled to a 400-patient baseline |
| Ingredient unit costs | Wholesale market averages — **synthetic**, so the model can be shared without a procurement NDA |

---

*Course project for MMA 861, Queen's Smith School of Business. I owned the model —
formulation, constraint design and sensitivity analysis.*
