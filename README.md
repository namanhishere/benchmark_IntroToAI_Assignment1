# Introduction to AI - Assignment 1 - Benchmark Result


Benchmarks **DFS** and **A\*** across two logic puzzles — **Sudoku** and **Net (pipes)** — at multiple sizes and difficulty levels, iterating over every available heuristic for A\*.
All measurements are averaged over repeated runs with warm-up to reduce noise, and results are exported as CSV files and charts.

---
## Source Code

Algorithm and domain implementations are in the separate repository [namanhishere/IntroAiAssignment1](https://github.com/namanhishere/IntroAiAssignment1). That repository contains:

- `algorithms/`: DFS, BFS, A\* implementations
- `domain/`: Sudoku and Net problem definitions with heuristics
- `generator/`: Puzzle generation for Sudoku and Net
- `rendering/`: Visualization and video rendering

Refer to [IntroAiAssignment1's README](https://github.com/namanhishere/IntroAiAssignment1/blob/main/readme.md) for implementation details.

---

## Overview

| Puzzle | Variable | Algorithms tested |
|---|---|---|
| Sudoku | Difficulty: Easy (30), Medium (40), Hard (50), Max | DFS · A\* × {mrv, cells, srv, zero} |
| Net (pipes) | Grid size: 5×5 → 30×30 | DFS · A\* × {lookahead, cells, connectivity, zero} |

Each (algorithm × puzzle × trial) task is dispatched to a `ProcessPoolExecutor` that saturates all available CPU cores. Within each worker:

- **2 warm-up** runs — primes the interpreter & GC, excluded from averages
- **10 benchmark** runs — averaged and reported
- **120 s per-run timeout** enforced via `SIGALRM`

Metrics collected: average time (s), average steps (nodes explored), path length, peak memory (MB).

---

## Project Structure

```
newbenchmark/
├── run.ipynb                   # Main benchmarking notebook
├── benchmark_results.csv       # Root-level copy of the combined summary
├── benchmark_output/           # All generated outputs land here
│   ├── benchmark_results_raw.csv
│   ├── sudoku_summary.csv
│   ├── net_summary.csv
│   ├── sudoku_runs_detail.csv
│   ├── net_runs_detail.csv
│   ├── all_runs_detail.csv
│   ├── sudoku_time.png
│   ├── sudoku_steps.png
│   ├── sudoku_memory.png
│   ├── net_time.png
│   ├── net_steps.png
│   └── net_memory.png
└── IntroAiAssignment1/         # Cloned from github.com/namanhishere/IntroAiAssignment1
```

The benchmark notebook automatically clones [IntroAiAssignment1](https://github.com/namanhishere/IntroAiAssignment1) from GitHub if it's not present locally.

---

## Requirements

- Python 3.10+
- Jupyter (for running the notebook interactively)
- Git (for cloning the IntroAiAssignment1 repository)

The notebook will automatically:
1. Clone [namanhishere/IntroAiAssignment1](https://github.com/namanhishere/IntroAiAssignment1) if needed
2. Install dependencies from that repository

Manual setup:

```bash
# Clone IntroAiAssignment1 repo if not already present
git clone git@github.com:namanhishere/IntroAiAssignment1.git

# Install dependencies
pip install -r IntroAiAssignment1/requirements.txt
pip install psutil py-cpuinfo matplotlib pandas tabulate
```

---

## Running the Benchmark

Open and run `run.ipynb` top-to-bottom in VS Code or Jupyter Lab. Each section is independent:

| Section | Contents |
|---|---|
| **1 · Setup** | Installs packages, adds `IntroAiAssignment1` to `sys.path` |
| **2 · System Info** | Prints CPU model, core count, RAM |
| **3 · Benchmark Helper** | Defines the worker, timeout logic, and parallel runner |
| **4 · Sudoku Benchmark** | Generates puzzles → runs tasks → shows table & charts |
| **5 · Net Benchmark** | Generates puzzles → runs tasks → shows table & charts |
| **6 · Export** | Saves all CSVs listed in the table below |
| **7 · Script Export** | Converts the notebook to `benchmark_script.py` |

---

## Output Files

| File | Granularity | Description |
|---|---|---|
| `benchmark_results_raw.csv` | 1 row per task | Per-task averaged summary (all games) |
| `sudoku_summary.csv` | 1 row per (difficulty × algo) | Grouped averages for Sudoku |
| `net_summary.csv` | 1 row per (size × algo) | Grouped averages for Net |
| `sudoku_runs_detail.csv` | 1 row per individual run | Warmup + benchmark runs with seed & grid JSON |
| `net_runs_detail.csv` | 1 row per individual run | Warmup + benchmark runs with seed & grid JSON |
| `all_runs_detail.csv` | Combined | Both games, every run |

---

## Configuration

All tunable parameters are at the top of the *Benchmark Helper* cell:

```python
TIMEOUT        = 120   # seconds per single run before SIGALRM fires
WARMUP_RUNS    = 2     # warm-up iterations (saved but excluded from avg)
BENCHMARK_RUNS = 10    # benchmark iterations whose avg is reported
NUM_WORKERS    = os.cpu_count()  # worker processes
```

Puzzle configurations:

```python
SUDOKU_LEVELS   = {'Easy (30)': 30, 'Medium (40)': 40, 'Hard (50)': 50, 'Max': 'max'}
NUM_SUDOKU_TRIALS = 3

NET_SIZES       = [5, 10, 15, 20, 25, 30]
NUM_NET_TRIALS  = 10
```

---

## Algorithms

### DFS
Explores the deepest path first (LIFO stack). No optimality guarantee; memory-efficient. Each worker resets state between runs via `gc.collect()`.

### A\*
Priority queue ordered by $f(n) = g(n) + h(n)$. Optimal when the heuristic is admissible. The benchmark iterates over every registered heuristic automatically.

**Sudoku heuristics** (`SudokuProblem.HEURISTIC_FUNCTION`):

| Name | Description |
|---|---|
| `mrv` | Minimum Remaining Values — focuses on the most constrained cell |
| `cells` | Count of empty cells remaining |
| `srv` | Sum of remaining valid values across all empty cells |
| `zero` | Always 0 — A\* degenerates to uniform-cost search |

**Net heuristics** (`NetProblem.HEURISTIC_FUNCTIONS`):

| Name | Description |
|---|---|
| `lookahead` | Forward checking — prunes rotations that create dead ends |
| `cells` | Count of unlocked cells remaining |
| `connectivity` | Frontier connectivity — penalises isolated islands |
| `zero` | Always 0 — uniform-cost baseline |

---

## Reproducibility

Every puzzle is generated from a fixed seed:

- Sudoku trial `t`: seed = `1000 + t`
- Net size `s`, trial `t`: seed = `2000 + s×100 + t`

Both the seed and the full serialised grid (`grid_json`) are stored in the per-run CSVs so any individual run can be exactly reproduced.

---

