
# Unnormalized Spectral Clustering of the ITU Campus Road Network

Applied unnormalized spectral clustering to the ITU Ayazağa campus walk-road
graph to detect community structure on a sparse urban road network. Three
edge-based Laplacian constructions (unweighted, inverse-length, Gaussian) are
compared against a coordinate-based `k`-means baseline. Eigenpairs are computed
with a custom shift–invert inverse power method featuring normalization,
orthogonalization, and deflation.

**Author:** Furkan Yalçın · **Student ID:** 150220116
**Course:** Istanbul Technical University

## Contents

| File | Description |
| --- | --- |
| `7799641.ipynb` | End-to-end notebook: graph loading (OSMnx), Laplacians, eigen-solver, clustering, modularity scoring, and cluster maps |
| `7800458.pdf`  | Written report (IEEE format) with methodology, results table, and discussion |

## Graph

The ITU Ayazağa campus walk-road graph is harvested with OSMnx and converted to
an undirected graph:

- **594 nodes**, **864 edges**
- Edge density ≈ **0.49%**, average degree **k̄ ≈ 2.9**
- Sparsity makes distance-based affinities challenging — down-weighting long
  edges can disconnect the graph too early.

## Method

### Similarity models and Laplacians

Let `ℓ_ij` be edge length and `ℓ̄ = mean(ℓ_ij)`. Four affinity choices:

1. **Unweighted** — `W_ij = 1`. Keeps every edge equally.
2. **Inverse length** — `W_ij = ℓ̄ / ℓ_ij`.
3. **Gaussian** — `W_ij = exp(−(ℓ_ij / ℓ̄)²)` with a single global `σ = ℓ̄`.
4. **Coordinate-based k-means** — cluster node `(x, y)` positions directly (no
   Laplacian).

Each weighted `W` yields an unnormalized Laplacian `L = D − W`, where `D` is the
diagonal degree matrix of `W`.

### Inverse power–shift eigen-solver

The `k` smallest nonzero eigenpairs of `L` are extracted iteratively:

- **Shift** (`μ > 0`): solve `(L + μI)⁻¹ x = λ⁻¹ x` to target small eigenvalues.
- **Normalization**: `x ← x / ‖x‖₂` each step to prevent over/underflow.
- **Orthogonalization**: Gram–Schmidt against previously found eigenvectors.
- **Deflation**: `L ← L − λᵢ xᵢ xᵢᵀ` after each pair is found.

### Clustering

For each `k ∈ {4, 5, 7, 10, 15}` we form two embeddings per Laplacian:

- **Standard**: `U = [x₁, …, x_k] ∈ ℝⁿˣᵏ`.
- **Discarded-first**: `U = [x₂, …, x_{k+1}]` — drop the (near-constant)
  trivial eigenvector.

KMeans is run on the rows of `U`. The coordinate baseline runs KMeans directly
on node `(x, y)` positions.

## Results — Newman–Girvan Modularity `Q`

Higher is better.

| Method                          |   k=4  |   k=5  |   k=7  |  k=10  |  k=15  |
| ------------------------------- | :----: | :----: | :----: | :----: | :----: |
| Unweighted                      | 0.663  | 0.683  | 0.765  | 0.814  | 0.848  |
| Unweighted (discarded)          | 0.675  | 0.703  | 0.768  | 0.812  | 0.838  |
| Inverse-length                  | 0.489  | 0.493  | 0.753  | 0.789  | 0.837  |
| Inverse-length (discarded)      | 0.493  | 0.499  | 0.729  | 0.747  | 0.837  |
| Gaussian                        | 0.026  | 0.289  | 0.499  | 0.743  | 0.782  |
| Gaussian (discarded)            | 0.026  | 0.275  | 0.494  | 0.752  | 0.768  |
| Coord. k-means                  | 0.649  | 0.659  | 0.771  | 0.810  | 0.807  |

## Key take-aways

- **Discarding the first eigenvector** helps smaller `k` but the benefit shrinks
  as `k` grows.
- **Unweighted Laplacian** is the safest choice on this sparse road graph —
  best `Q` overall and intuitive cluster shapes.
- **Inverse-length Laplacian** becomes competitive once `k ≥ 7`, surfacing
  meaningful medium-sized pockets.
- **Gaussian kernel** is too harsh at low `k` (one giant cluster + tiny
  islands), but catches up as `k` grows.
- **Coordinate k-means** is a strong baseline at low `k` but loses to edge-based
  methods once the graph's connectivity matters (`k ≥ 7`).

## Conclusion

On sparse urban road networks like the ITU campus, unweighted and inverse-length
Laplacians outperform pure spatial clustering once `k` is sufficiently large.
The unweighted Laplacian offers the best combination of high modularity,
interpretability, and computational efficiency. Gaussian weighting with the
chosen `σ` tends to shear off small isolated clusters and is too harsh for this
sparse graph.

## Reproducing

Open `7799641.ipynb` and run top-to-bottom. Main dependencies:

```
osmnx networkx numpy scipy scikit-learn matplotlib
```

Globals at the top of the notebook control the run:

```python
K_CLUSTERS  = 15     # number of clusters
SHIFT       = 1e-3   # μ for inverse power method
IP_MAXITER  = 500    # iteration cap
IP_TOL      = 1e-6   # convergence tolerance
```

## References

[1] U. von Luxburg, "A tutorial on spectral clustering," *Statistics and
    Computing*, vol. 17, no. 4, pp. 395–416, 2007.
