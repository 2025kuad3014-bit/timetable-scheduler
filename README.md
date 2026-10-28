# AI-Based University Timetable Scheduler

An enterprise-grade, constraint-satisfaction and metaheuristic timetable scheduling engine developed for a B.Tech Artificial Intelligence course project.

The system formulates university course scheduling as a **Constraint Satisfaction Problem (CSP)** with multi-objective optimization, featuring five distinct AI algorithms, complete validation pre-checks, interactive Streamlit UI, multi-format exports (CSV, Excel, PDF), and a comprehensive benchmarking suite.

---

## 📌 Architecture & Design Principles

```
Ai project/
├── core/                       # Core AI Engine (Pure Python, Zero UI dependencies)
│   ├── models.py               # Type-annotated dataclasses (Dataset, Session, Timetable, etc.)
│   ├── loader.py               # JSON & multi-CSV dataset loaders
│   ├── validator.py            # Solvability pre-checks (capacity, teacher load, hours)
│   ├── constraints.py          # Single source of truth: H1-H9 hard & S1-S5 soft constraints
│   ├── teacher_assign.py       # Greedy load-balancing teacher allocation
│   ├── csp.py                  # Domain generator & constraint graph builder
│   ├── backtracking.py         # Informed Backtracking CSP (MRV, Degree, LCV, FC, AC-3)
│   ├── min_conflicts.py        # Min-Conflicts local search with random walk & restarts
│   ├── annealing.py            # Simulated Annealing with multi-operator neighbourhood
│   ├── hybrid.py               # Two-phase solver: Backtracking feasibility + SA optimization
│   ├── hill_climbing.py        # Steepest-ascent hill climbing baseline
│   ├── exporters.py            # Export to CSV, Multi-Sheet Excel, and Printable PDF
│   └── experiments.py          # Empirical comparison, ablation studies, and visualization
├── data/                       # Datasets & synthetic data generator
│   ├── small.json / large.json # Benchmark datasets
│   ├── csv_small/ / csv_large/ # CSV formatted datasets
│   └── generate_datasets.py    # Generator and verification script
├── ui/                         # Streamlit Presentation Helpers
│   └── styles.py               # Earthy modern typography & responsive HTML grid renderer
├── tests/                      # Automated test suite
│   └── test_core.py            # 17 Unit tests covering constraints, solvers, and exports
├── docs/                       # Technical & Viva Documentation
│   ├── FORMULATION.md          # Formal mathematical CSP definition
│   ├── HEURISTICS.md           # Deep dive into MRV, Degree, LCV, FC, and AC-3
│   ├── ALGORITHMS.md           # Comparative analysis of all 5 algorithms
│   └── VIVA_PREP.md            # 25+ Viva examination questions and answers
├── results/                    # Auto-generated benchmark charts & analysis reports
├── app.py                      # Interactive 7-screen Streamlit Web App
└── requirements.txt            # Dependency manifest
```

---

## 🚀 Quick Start

### 1. Installation
Clone the repository and install the dependencies:
```bash
pip install -r requirements.txt
```

### 2. Verify Datasets
Run the dataset generator and verifier:
```bash
python data/generate_datasets.py
```

### 3. Run Automated Tests
Execute the pytest suite:
```bash
pytest tests/test_core.py -v
```

### 4. Launch the Streamlit Web Application
Start the interactive UI:
```bash
streamlit run app.py
```
Open your browser at `http://localhost:8501`.

---

## 🧠 Solvers & AI Algorithms

The engine implements 5 AI solver paradigms:

| Algorithm | Paradigm | Strengths | Best Use Case |
| :--- | :--- | :--- | :--- |
| **Backtracking (Informed)** | Systematic Tree Search | Guaranteed complete; explores $< 100$ nodes with MRV + Degree + FC | Fast feasible schedules ($< 0.1$s) |
| **Hybrid (BT + SA)** | Two-Phase (Exact + Meta) | Guaranteed 0 hard violations + drives soft penalties to zero | **Production Recommended** |
| **Simulated Annealing** | Probabilistic Metaheuristic | Escapes local minima via temperature schedule | Soft constraint optimization |
| **Min-Conflicts** | Local Repair Search | $O(n)$ memory; fast on loosely-constrained schedules | Quick repair operations |
| **Hill Climbing** | Steepest Ascent | Rapid greedy convergence with random restarts | Baseline comparison |

---

## 📋 Constraints Modeled

### Hard Constraints ($H_1 - H_9$) — Zero Violations Allowed
- **$H_1$ (Teacher Clash):** A teacher cannot teach two sessions simultaneously.
- **$H_2$ (Class Clash):** A student batch cannot have two classes simultaneously.
- **$H_3$ (Room Clash):** A room cannot host two sessions simultaneously.
- **$H_4$ (Room Capacity):** Room capacity must accommodate class strength.
- **$H_5$ (Room Type):** Labs in lab rooms; lectures in lecture classrooms.
- **$H_6$ (Faculty Availability):** Respects teacher unavailable slots.
- **$H_7$ (Contact Hours):** Every subject receives exactly its weekly required hours.
- **$H_8$ (Teacher Workload):** Teacher weekly hours cannot exceed `max_weekly_load`.
- **$H_9$ (Lunch Break):** No sessions during the class lunch period.

### Soft Constraints ($S_1 - S_5$) — Multi-Objective Minimization
- **$S_1$ (Idle Gaps):** Minimizes free periods between classes for students.
- **$S_2$ (Subject Spread):** Avoids duplicate theory lectures for the same subject in one day.
- **$S_3$ (Lab Continuity):** Keeps 2-hour laboratory sessions contiguous.
- **$S_4$ (Teacher Preferences):** Respects teacher preferences to avoid specific periods.
- **$S_5$ (Load Smoothing):** Balances student daily hours evenly across the week.

---

## 📊 Running Experiments & Benchmarks via CLI

Run the full comparative benchmark suite directly from the command line:
```bash
# Compare all algorithms on small dataset
python -m core.experiments --dataset small --runs 5 --out results/

# Run complete benchmarks on both small and large datasets
python -m core.experiments --dataset both --runs 5 --out results/
```
Generated artifacts will be placed in `results/`:
- `comparison_small.csv` & `comparison_small.png`: Runtime, node counts, and violation comparisons.
- `ablation_small.csv`: Heuristic ablation isolating MRV, Degree, FC, and AC-3.
- `cost_iterations_small.png`: Cost reduction trajectories over search iterations.
- `analysis_small.md`: Automatically drafted performance analysis report.

---

## 🖥️ Streamlit Web Interface Features

The application (`app.py`) provides 7 views:
1. **📂 Dataset Management:** Load sample benchmarks, upload custom JSON/CSV files, or view class/teacher/room tables.
2. **⚙️ Generate Timetable:** Configure solver algorithm, heuristic toggles (MRV, Degree, LCV, FC, AC-3), soft constraint weights, and launch solver with live progress updates.
3. **📅 Timetable Views:** View schedules by **Class**, **Teacher**, or **Master Campus View** in high-contrast color-coded grids.
4. **✏️ Interactive Editor:** Modify timeslots or rooms with instant conflict detection against all 9 hard constraints.
5. **⚖️ Multi-Solver Comparison:** Run multiple algorithms head-to-head on the active dataset with side-by-side performance metrics.
6. **📈 Empirical Experiments:** Trigger ablation studies and cost-over-time graphs directly within the browser.
7. **💾 Export Center:** Download finalized schedules as CSV, multi-sheet formatted Excel (`.xlsx`), or printable PDF (`reportlab`).

---

## 🎓 Academic Defense & Viva Preparation

For detailed viva preparation and theoretical explanations, refer to the documentation inside `docs/`:
- [docs/FORMULATION.md](docs/FORMULATION.md): Formal mathematical CSP equations.
- [docs/HEURISTICS.md](docs/HEURISTICS.md): CSP heuristic explanations and complexity analysis.
- [docs/ALGORITHMS.md](docs/ALGORITHMS.md): Detailed algorithm comparisons and trade-offs.
- [docs/VIVA_PREP.md](docs/VIVA_PREP.md): 25+ detailed questions & model answers for oral defense.
