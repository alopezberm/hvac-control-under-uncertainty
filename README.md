# HVAC Control under Uncertainty — Sequential Decision Policies

Group project for DTU course **02435 Decision-Making under Uncertainty** (spring 2026). This repository
is a fork of the group repository [ccanoyrac/Decision-Making-under-Uncertainty](https://github.com/ccanoyrac/Decision-Making-under-Uncertainty);
the work of each member is stated in [Authors and contributions](#authors-and-contributions).

![Policy comparison on 100 identical days](dmuu_policy_group14_s250421_s243368_s253272_s260267/output/all_policies_comparison.png)

## Problem

A two-room restaurant buys electricity at an hourly price that is not known in advance and has an
occupancy that is not known in advance either. Every hour the controller decides the heating power of
each room and whether to run the ventilation. The goal is to minimise the electricity cost of the day
(10 hourly decisions) while respecting the comfort rules: temperature and humidity limits, overrule
controllers that force heating or ventilation when a limit is hit, and a minimum ventilation run time.

The decision has to be taken hour by hour with only the current price and occupancy observed, so it is
a sequential decision problem under uncertainty rather than a single optimisation.

## Approach

Each policy maps the current state (room temperatures, humidity, ventilation counter, overrule flags,
current price and occupancy) to a decision. All policies are MILPs written in Pyomo and solved with
Gurobi; the binary variables model the ventilation and the big-M logic of the overrule controllers.

| Policy | Idea | File |
|---|---|---|
| Hindsight optimum | Perfect-information MILP over the whole day; a lower bound no causal policy can beat | `hindsight_optimization_14.py` |
| Dummy | Reacts only to the overrule controllers; the baseline every policy should beat | `main_evaluation.ipynb` |
| Deterministic lookahead (EV-MPC) | Solves the remaining horizon with future prices and occupancy replaced by their means; executes the first decision | `Deterministic_policy_14.py` |
| Two-stage stochastic | Scenario tree with one stage of uncertainty (10 branches) | `Two_stage_policy_14.py` |
| Multi-stage stochastic (SP) | Scenario tree over a 2-hour horizon (10 branches), scenarios reduced by clustering | `SP_policy_14.py` |
| Approximate dynamic programming (ADP) | Single-step MILP plus a time-indexed linear value function approximation, trained offline by approximate policy iteration with ridge regression | `ADP_policy_14.py`, `ADP_Training_14.py` |
| Hybrid (SP + CFA) | Stochastic lookahead combined with a cost function approximation | `Hybrid_policy_14.py` |

`Task6_Environment.py` runs every policy through the same simulator, day by day. The policies only
see the information available at each hour.

## Data

The price and occupancy processes, the system parameters, the policy template and the feasibility
checker in `Data/` and `v2_Checks.py` are provided by the course (author: course staff) and are used
unchanged. The assignment text and the official solution of Part A are course material and are not
included in this repository.

## Results

Mean daily cost over the same 100 days (from `main_evaluation.ipynb`):

| Policy | Mean daily cost (EUR) | Gap to hindsight optimum |
|---|---:|---:|
| Hindsight optimum (lower bound) | 117.7 | — |
| Hybrid (SP + CFA) | 142.9 | 21.4% |
| Multi-stage SP | 145.4 | 23.5% |
| ADP | 146.9 | 24.8% |
| Two-stage | 148.8 | 26.4% |
| Deterministic lookahead (EV-MPC) | 172.9 | 46.9% |
| Dummy | 183.0 | 55.5% |

The stochastic policies close more than half of the gap between the reactive baseline and the
perfect-information bound. The ADP reaches almost the same cost as the multi-stage SP while solving one
small MILP per hour (about 30 ms per decision against about 270 ms for the SP).

## Tech stack

Python · Pyomo · Gurobi · NumPy · pandas · scikit-learn · Matplotlib · Jupyter

## How to run

```bash
cd dmuu_policy_group14_s250421_s243368_s253272_s260267
pip install -r requirements.txt          # needs a Gurobi licence (academic or size-limited)
jupyter lab main_evaluation.ipynb        # evaluates every policy and regenerates output/
```

To retrain the ADP value function approximation: `python ADP_Training_14.py`.

## Authors and contributions

Group 14 (four students), DTU 02435 Decision-Making under Uncertainty, spring 2026. Code
contributions as recorded in the git history:

- **Carlos Canoyra Calero**: stochastic programming, two-stage, deterministic lookahead, hindsight and
  hybrid policies; evaluation environment and notebook; final integration of the code, including the
  final version of the ADP policy and its training script.
- **Philipp**: distributed coordination of the HVAC systems of 15 stores under a shared
  power limit by dual decomposition (Task 7).
- **Alejandro López Bermejo** (this fork), see below.

### My contribution

- Designed and implemented the first versions of the **ADP policy** (Task 4 of Part B): the
  time-indexed linear value function approximation, the hand-crafted normalised state features, the
  single-step MILP under certainty equivalence and the offline approximate policy iteration loop with
  ridge regression. My implementation is preserved in the history at commit
  [`1cbec38`](https://github.com/alopezberm/hvac-control-under-uncertainty/tree/1cbec38/Part_B)
  (`Part_B/ADP_policy_14.py`, `Part_B/train_adp.py`, `Part_B/ADP_policy_14.ipynb`). The ADP in the
  final submission is the version rewritten by Carlos during integration, which is the one evaluated
  above.
- Part A: notebook structure, scenario plots and result figures.
- Repository documentation (earlier README) and, in this fork, the cleanup and this README.
