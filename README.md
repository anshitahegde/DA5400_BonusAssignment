# DA5400_BonusAssignment

This repo consists of a modified implementation of the algorithm discussed in Optimised robust classification trees. The paper employs a complete MIO optimisation algorithm for the master problem; however, I use the built-in library CART to generate the tree. It is not a multi-integer optimal, but a globally good tree. The core logic of the paper: the two-stage decision-making process, the uncertainty budget is maintained. 

## Algorithm Overview

The framework structures the training process as an iterative game between a tree learner and an adversary using a Benders-style decomposition loop.

### 1. The Uncertainty Set ($\Xi$)
The adversary is allowed to flip or modify feature values to force misclassifications, but they operate under a strict total budget ($\varepsilon$). The budget allocation is bounded by feature reliability costs ($\gamma_f$):

$$\Xi := \{ \xi \in \mathbb{Z}^{|\mathcal{I}| \times |\mathcal{F}|} \quad : \sum_{i \in \mathcal{I}} \sum_{f \in \mathcal{F}} \gamma_{f} |\xi_{f}^{i}| \le \varepsilon \}$$

* **$\gamma_f$ (Feature Cost):** Features that are highly reliable or difficult to tamper with are assigned higher penalty costs.
* **$\varepsilon$ (Total Budget):** Derived from the regularization parameter $\lambda$ scaled by the training set size.

### 2. The Benders Decomposition Loop

The algorithm alternates between solving a **Master Problem** and a **Subproblem** until the tree structure completely stabilises:

1. **Master Problem:** Trains a standard decision tree (using CART rules via `DecisionTreeClassifier`) using the current state of the dataset.
2. **Subproblem (Adversarial Attack):** For every sample correctly predicted by the tree, a Depth-First Search (`dfs`) identifies the absolute cheapest feature flips required to navigate that sample into a misclassified leaf node.
3. **Greedy Budget Allocation:** The framework sorts these misclassification paths from cheapest to most expensive and greedily applies the features flips until the total adversary budget $\varepsilon$ is entirely exhausted.
4. **Rebuild:** The dataset is modified with these worst-case perturbations, and the loop restarts to build a new tree capable of resisting them.

### 3. The Iterative Subproblem Loop

The loop that updates worst case modification, flows as such 

```markdown
```text
┌────────────────────────────────────────────────────────┐
│                    Master Problem:                     │
│  Train standard CART DecisionTree on perturbed data    │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                 Adversary Subproblem:                  │
│  Run depth-first search to find the smallest feature   │
│  flips to trigger misclassification for each correct   │
|  sample                                                |
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                Greedy Budget Allocation:               │
│  Sort paths and apply flips to cheapest samples        │
│  until total global budget ε is exhausted              │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
                    [ Rebuild Tree ] -- loops back

```

For every sample currently predicted correctly by the tree, a recursive DFS (`_misclassify_cost`) evaluates the tree paths to find the absolute cheapest alternative leaf node containing a mismatching class ($k \neq y_i$). 
* Pushing a sample across an encountered splitting node against its natural path incurs a feature manipulation cost of:
  $$\text{Cost} = \gamma_f \times \Delta$$
* Where $\Delta$ is the absolute difference needed to step over the branch threshold.

The master function aggregates the calculated attack costs for all active training points and sorts them from cheapest to most expensive. It steps through the sorted samples, applying adversarial flips to the data matrix, and terminates the iteration once the accumulated costs hit the total adversary budget threshold $\varepsilon$.
