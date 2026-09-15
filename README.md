# Computational Intelligence Algorithms — Course Homework

> Implementations of three canonical meta-heuristics from scratch (no external optimization libraries): **Particle Swarm Optimization (PSO)**, **Genetic Algorithm (GA)**, and **Ant Colony Optimization (ACO)**. Built for the Computational Intelligence (智能计算) course.

***

## 📌 Overview

| Item                                       | Detail                                                          |
| ------------------------------------------ | --------------------------------------------------------------- |
| **Course**                                 | Computational Intelligence (智能计算)                               |
| **Algorithms**                             | PSO · GA · ACO                                                  |
| **Language**                               | Python (numpy / matplotlib / seaborn)                           |
| **External Dependencies for Optimization** | None — all operators hand-implemented                           |
| **Problems Solved**                        | (1) Model-fusion coefficient search · (2) Longest path in a DAG |

The repository contains three independent, self-contained modules — each is a single Python file that runs directly without configuration.

***

## 🗂️ Modules

| File               | Algorithm                   | Problem Solved                                                              |
| ------------------ | --------------------------- | --------------------------------------------------------------------------- |
| [`PSO.py`](PSO.py) | Particle Swarm Optimization | Find fusion coefficients that align two model predictions with ground truth |
| [`GA.py`](GA.py)   | Genetic Algorithm           | Find the longest (critical) path in a 10-node Directed Acyclic Graph        |
| [`ant.py`](ant.py) | Ant Colony Optimization     | Same longest-path problem as `GA.py` — solved with a swarm-walking approach |

***

## 1️⃣ Particle Swarm Optimization — `PSO.py`

### Problem

Two base models produce predictions `pre = [350, 328]`. The true label sum is `311`. Find fusion weights `X = [x1, x2] ∈ [0,1]²` such that the weighted sum `pre · X` matches `311` as closely as possible.

### Fitness

```
f(X) = | pre · X − sum |   →   minimize
```

### Algorithm Design

| Component                              | Implementation                                    |
| -------------------------------------- | ------------------------------------------------- |
| **Swarm size**                         | 10,000 particles                                  |
| **Inertia weight** `w`                 | 0.5                                               |
| **Acceleration coefficients** `c1, c2` | 2.0, 2.0                                          |
| **Velocity update**                    | `v ← w·v + c1·r1·(pbest − x) + c2·r2·(gbest − x)` |
| **Position update**                    | `x ← x + v`                                       |
| **Personal best**                      | Updated when a new fitness beats the stored one   |
| **Global best**                        | The best `pbest` across the swarm                 |
| **Iterations**                         | 100                                               |

### Output

- Convergence curve: best fitness vs. iteration count (matplotlib).

### Notes

- The legality clamp (forcing positions back into `[0,1]`) is included as commented code — the unconstrained version already converged on the assignment, but the clamp is documented for reproducibility.

***

## 2️⃣ Genetic Algorithm — `GA.py`

### Problem

Find the **longest path** (critical path) from `V0` to `V9` in a weighted Directed Acyclic Graph (DAG) with 10 nodes. Edge weights are stored in a 10×10 adjacency matrix `M` (with `NaN` for non-existent edges).

### Chromosome Encoding

A binary vector of length 10. The first gene (`V0`) and the last gene (`V9`) are fixed to `1`. Each interior gene indicates whether the corresponding vertex is included in the path.

### Algorithm Design

| Component              | Implementation                                                                                                               |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Population size**    | 100                                                                                                                          |
| **Fitness evaluation** | Sum of edge weights walked by the chromosome; invalid jumps (where `M[i][c]` is `NaN`) cause the offending gene to be zeroed |
| **Selection**          | Roulette wheel — selection probability ∝ fitness                                                                             |
| **Crossover**          | Single-point, probability `p = 0.88`, crossover point sampled in `[1, 9]` (terminal genes preserved)                         |
| **Mutation**           | Bit-flip, probability `p = 0.1` per gene, terminals excluded                                                                 |
| **Iterations**         | 1,000                                                                                                                        |

### Legality Handling

A chromosome may encode a vertex sequence that is not a valid walk in the DAG. During fitness evaluation, the algorithm walks the chromosome left-to-right; if the next `1`-gene has no edge from the current vertex, it is reset to `0` in place — repairing the chromosome into a legal path.

### Output

- Critical path length (max fitness) and the corresponding vertex sequence, e.g. `['V0', 'V1', 'V3', 'V4', 'V6', 'V9']`.

***

## 3️⃣ Ant Colony Optimization — `ant.py`

### Problem

The same 10-node DAG longest-path problem as `GA.py`, solved by a fundamentally different mechanism: a colony of ants walks the graph probabilistically, leaving pheromone trails that bias future ants toward high-weight edges.

### Algorithm Design

| Component                    | Implementation                                                              |
| ---------------------------- | --------------------------------------------------------------------------- |
| **Ant count**                | 20                                                                          |
| **Pheromone matrix** **`T`** | Initialized to the ant count on every legal edge                            |
| **Transition probability**   | `p(i→j) ∝ T[i][j] · weight(i→j)` — combines pheromone with edge weight      |
| **Selection**                | Roulette wheel over normalized transition probabilities                     |
| **Pheromone evaporation**    | `T ← T × 0.5` (50% decay per iteration)                                     |
| **Pheromone deposit**        | Each ant deposits `20%` of its total path weight on every edge it traversed |
| **Termination**              | All ants must reach `V9` before pheromone update (synchronous update)       |

### Key Design Choices

- **Edge-weight in transition rule** — unlike pure ACO (pheromone-only), this implementation multiplies pheromone by the static edge weight, biasing exploration toward long edges from the very first iteration. This is a common pragmatic tweak for the longest-path problem.
- **Synchronous pheromone update** — all ants complete their walk before any pheromone is deposited, avoiding order bias.

### Output

- Longest path length (max ant path weight) and the corresponding vertex sequence.

***

## 🏗️ Shared DAG Definition (for GA & ACO)

Both `GA.py` and `ant.py` operate on the same 10-vertex DAG. Vertices are `V0 … V9`; edges and weights are encoded as a 10×10 numpy matrix where `NaN` denotes "no edge":

```
V0 → V1 (3)   V0 → V2 (4)
V1 → V3 (5)   V1 → V4 (6)
V2 → V3 (8)   V2 → V5 (7)
V3 → V4 (3)
V4 → V6 (9)   V4 → V7 (4)
V5 → V7 (6)
V6 → V9 (2)
V7 → V8 (5)
V8 → V9 (3)
```

`V0` is the source and `V9` is the sink — the goal is to find a path `V0 → … → V9` maximizing total edge weight.

***

## ⚙️ Engineering Highlights

- **From-scratch implementations** — no `DEAP`, `pyswarm`, or `sklearn` optimizers; every operator (selection, crossover, mutation, pheromone update, velocity update) is implemented explicitly.
- **Self-contained modules** — each `.py` file is a single class with a `run()` method; no shared state, no config files.
- **Legality repair in GA** — chromosomes that encode invalid walks are repaired in place during fitness evaluation, keeping the search inside the feasible region without discarding individuals.
- **Convergence visualization** — PSO plots the best-fitness-vs-iteration curve, useful for tuning `w`, `c1`, `c2`.
- **Cross-algorithm comparison** — GA and ACO solve the same problem, making it easy to compare convergence speed, solution quality, and implementation complexity.

***

## 📁 Project Structure

```
github/
├── PSO.py        # Particle Swarm Optimization — model-fusion coefficient search
├── GA.py         # Genetic Algorithm — longest path in a DAG
├── ant.py        # Ant Colony Optimization — longest path in a DAG
└── README.md     # This file
```

***

## 🚀 How to Run

```bash
# No external setup beyond numpy / matplotlib / seaborn
pip install numpy matplotlib seaborn

# Run each algorithm independently
python PSO.py    # PSO — prints gbest history, shows convergence plot
python GA.py    # GA — prints critical path length and vertex sequence
python ant.py   # ACO — prints longest path length and vertex sequence
```

Each script prints its result to stdout and (for PSO) displays a matplotlib convergence curve.

***

## 📝 Key Takeaways

- **Three classical meta-heuristics, three different search philosophies** — PSO leverages social sharing of best positions; GA recombines partial solutions via crossover; ACO uses stigmergic communication through pheromone fields.
- **Encoding matters more than the algorithm** — for the DAG longest-path problem, GA's binary chromosome with legality repair and ACO's graph-walk encoding produce quite different search dynamics on the same problem.
- **From-scratch implementation forces understanding** — writing roulette-wheel selection, pheromone evaporation, and velocity updates by hand is a different (and deeper) exercise than calling `sklearn`'s `GA` or `pyswarm`.
- **Parameter sensitivity is real** — PSO's `w=0.5` and `c1=c2=2.0`, GA's `p_c=0.88` and `p_m=0.1`, and ACO's `ρ=0.5` and `Q=0.2` were tuned empirically; the convergence plot in PSO is a useful debugging tool for this.

***

## 📜 License

Course homework for educational purposes.
