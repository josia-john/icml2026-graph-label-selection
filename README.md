# Experiments: An Approximation Algorithm for Graph Label Selection

Code for the experiments in:

> **An Approximation Algorithm for Graph Label Selection**  
> Josia John, Maximilian Probst Gutenberg, Simon Meierhans  
> *ICML 2026*

Some of the codebase is originally from [https://github.com/mesimon/graph_label_selection](https://github.com/mesimon/graph_label_selection) (Cesa-Bianchi et al., 2010).

## Problem

Given a weighted graph G = (V, E, w) and a budget k, the **Graph Label Selection** problem asks for a set L ⊆ V with |L| ≤ k that maximizes

$$\Psi(L) := \min_{C \subseteq V \setminus L} \frac{w(C, V \setminus C)}{|C|}$$

## Algorithms

| Flag | Algorithm |
|------|-----------|
| `ours` | This paper: hierarchical decomposition sparsifier + tree DP |
| `cb` | Cesa-Bianchi et al. (2010) |
| `gb` | Guillory & Bilmes (2009) |
| `ca` | Cohen-Addad et al. (2025) |

For `ours`, the sparsifier is built using one of several bisection heuristics (`--bisectalgos`):

| Flag | Description |
|------|-------------|
| `spectral` | Fiedler vector sweep |
| `spectralbalanced_10` | Fiedler, balanced (β = 0.1) |
| `spectralbalanced_1` | Fiedler, balanced (β = 0.01) |
| `metismulti_1` | METIS, √n target sizes |
| `metismulti_10` | METIS, 10√n target sizes |
| `metismulti_100` | METIS, 100√n target sizes |

## Installation

Install the Python dependencies:

```
pip install -r requirements.txt
```

The `metis` Python package wraps the native METIS library, which must be installed separately (see [metis.readthedocs.io](https://metis.readthedocs.io)). Build it from source with shared library support:

```
git clone https://github.com/KarypisLab/METIS
cd METIS
make config shared=1
make
```

If the library is not found automatically, point the wrapper to it explicitly:

```
export METIS_DLL=/path/to/libmetis.so
```

On NixOS/Nix, `shell.nix` handles everything:

```
nix-shell
```

## Input format

Graphs are edge lists: one edge per line, two space-separated integer node IDs. Self-loops are removed automatically and the largest connected component is used.

The SNAP datasets used in the paper are available at https://snap.stanford.edu/data.

## Usage

**Step 1 — run experiments:**

```
python main.py <graph> [options]
```

Results are written as text files under `results/<run_id>/<graph>/<algo>/k-<k>/itr-<i>`.

```
options:
  --algos       subset of {ours, cb, gb, ca}       (default: all)
  --bisectalgos subset of bisection heuristics      (default: all)
  --ks          list of budget values k             (default: 10 50 100)
  --num_itr     iterations for randomised algos     (default: 10)
```

**Step 2 — collect results:**

```
python results_to_json.py [results/] [-o results.json]
```

**Step 3 — plot:**

```
python visualize.py [results.json] [--out-dir plots/]
```

One PDF per (run, dataset) is written to `plots/`.

## File overview

| File | Description |
|------|-------------|
| `main.py` | Orchestrator: spawns all experiment subprocesses |
| `sparsifier.py` | Builds the hierarchical decomposition (tree cut sparsifier) |
| `bisectalgos.py` | Bisection heuristics: Fiedler sweep and METIS multi-try |
| `treeDP.py` | Tree DP solver for the Leaf Label Selection problem |
| `cesa_bianchi_et_al.py`* | Cesa-Bianchi et al. baseline |
| `guillory_bilmes.py`* | Guillory–Bilmes baseline |
| `greedy_via_flow.py`* | Cohen-Addad et al. baseline (greedy via max-flow) |
| `helper_functions.py`* | Shared utilities: graph I/O, cut computation, spanning trees |
| `results_to_json.py` | Converts the `results/` directory tree to JSON |
| `visualize.py` | Plots results from JSON as PDFs |
| `requirements.txt` | Python dependencies |
| `shell.nix` | Nix shell with METIS and Python environment |

\* Originally from [https://github.com/mesimon/graph_label_selection](https://github.com/mesimon/graph_label_selection).
