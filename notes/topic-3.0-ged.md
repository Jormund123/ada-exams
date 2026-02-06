# Graph Edit Distance (GED)

## Exam Relevance

Looking at `specs/questions.txt`, GED is part of **Topic B: Graph Similarity**. However, exam protocols focus heavily on **Weisfeiler-Leman**, not GED specifically.

**Verdict**: Secondary content. Know the concept, don't go deep on ILP formulations.

---

## Why GED Exists

### Frobenius Distance — The Problem It Solves

**Frobenius Distance** tries to compare graphs by permuting vertices to minimize adjacency matrix mismatches:

```
G₁:     G₂ (same graph, different numbering):

  1       4
 / \     / \
2   3   1   2
    |       |
    4       3

Matrix G₁:        Matrix G₂:
    1 2 3 4           1 2 3 4
1 [ 0 1 1 0 ]     1 [ 0 0 0 1 ]
2 [ 1 0 0 0 ]     2 [ 0 0 1 1 ]
3 [ 1 0 0 1 ]     3 [ 0 1 0 0 ]
4 [ 0 0 1 0 ]     4 [ 1 1 0 0 ]

→ Find permutation π minimizing ||A_π - B||
```

**Fatal flaw**: NP-hard **even for trees** or paths [Grohe et al. 2018].

---

### Graph Edit Distance — The Flexible Alternative

**Idea**: Compute minimum cost to transform G₁ into G₂ using:

- Vertex/edge **insertion** (add something)
- Vertex/edge **deletion** (remove something)
- Vertex/edge **substitution** (change a label)

Each operation has a **cost**. GED = minimum total cost.

```
G₁:  A ─── B ─── C        G₂:  A ─── B
         │                          │
         D                          E

Transformation G₁ → G₂:
1. Delete edge (B,C)    → cost c_del_edge
2. Delete vertex C      → cost c_del_vertex
3. Delete vertex D      → cost c_del_vertex
4. Insert vertex E      → cost c_ins_vertex
5. Insert edge (B,E)    → cost c_ins_edge

GED(G₁, G₂) = sum of all operation costs
```

**Why GED is useful**:

- Captures **structural differences** precisely
- **Flexible** — arbitrary edit costs
- **Error-tolerant** — can handle noisy data
- Works on **all graph types** (directed, undirected, labeled)

**The catch**: GED is **NP-hard**.

---

## Node Maps — The Key Simplification

Computing GED over ALL possible edit paths is intractable. Solution: restrict to **edit paths induced by node maps**.

### Definition: Node Map

A **node map** π: V^G ∪ {ε} → V^H ∪ {ε} where:

- Each vertex in G maps to exactly one vertex in H **or** to ε (deletion)
- Each vertex in H is the image of exactly one vertex in G **or** ε (insertion)

```
Node map example:

G:  a ─── b ─── c       H:  x ─── y
        │
        d

Possible node map π:
  a → x  (substitution: labels may differ)
  b → y  (substitution)
  c → ε  (deletion)
  d → ε  (deletion)
```

### Key Theorem

> **If edit costs satisfy the triangle inequality, then GED = GED-M**
> (GED restricted to node maps)

This means we only need to search over node maps, not arbitrary edit sequences.

---

## Algorithms for Computing GED

### A\* Algorithm

**Idea**: Build search tree of all partial mappings, use best-first search.

```
Search tree structure:

                      ∅ (empty mapping)
           ╱      |        |       ╲
        u₁→v₁   u₁→v₂   u₁→v₃    u₁→ε
        ╱  |  ╲
    u₂→v₂ u₂→v₃ u₂→ε
      ...
```

**Algorithm**:

1. Start with empty mapping
2. For each u_i ∈ V_G, extend partial mapping by trying all v_j ∈ V_H or ε
3. Always expand the path p minimizing **g(p) + h(p)**:
   - g(p) = actual cost so far
   - h(p) = **heuristic** estimated cost to complete

**Heuristics for h(p)**:

- **Label-based**: Compare remaining vertex/edge labels, count mismatches
- **Star-based**: Build stars around remaining vertices, use Hungarian algorithm to match

**Worst-case runtime**: **O(n!)** — exponential, only works for small graphs.

---

### Integer Linear Programming

GED can be formulated as an ILP. The slides present **four formulations** of increasing strength.

**Variables** (for IQP-GED):

- x\_{i,k} = 1 if vertex i ∈ G maps to vertex k ∈ H

**Quadratic objective** (edge matching creates quadratic terms):

```
min Σ c_V(i,k)·x_{i,k} + Σ c_E(ij,kl)·x_{i,k}·x_{j,l}
```

**Linearization**: Introduce y*{i,k,j,l} = x*{i,k} · x\_{j,l} to get linear program.

### Hierarchy of ILP Formulations

**F_ORI ≻ F₂ ≻ F₁**

Where ≻ means "strictly stronger LP relaxation".

| Formulation  | Variables         | Key Idea                                         |
| ------------ | ----------------- | ------------------------------------------------ |
| F₁           | n² + 2n + m² + 2m | Separate node/edge variables                     |
| F₂           | n² + m²           | Project out some variables, stronger constraints |
| F_ORI (2025) | n² + 2m²          | Use edge orientations for tighter bounds         |

**Practical result** [D'Ascenzo et al. 2025]:

- F₂ times out on 95%+ of hard instances
- **F_ORI solves ALL benchmark instances to optimality**

---

## What to Say If Asked

**"What approaches exist for graph similarity besides WL?"**

> "Frobenius distance and Graph Edit Distance. Frobenius minimizes adjacency matrix mismatches under vertex permutation — NP-hard even for trees. GED computes minimum cost to transform one graph into another using insertions, deletions, and substitutions. It's more flexible but also NP-hard. Can be solved with A\* search or ILP formulations."

**"How does A\* for GED work?"**

> "Build a search tree where each node is a partial vertex mapping. Use best-first search with g(p) + h(p) where g is cost so far and h is a heuristic estimate. Heuristics can use label matching or Hungarian algorithm on star structures. Worst case O(n!)."

---

## Exam Triage

**These parts are exam-critical**: NONE (GED not in exam protocols)

**These parts are secondary** (know conceptually):

- GED definition and purpose
- Why it's NP-hard
- A\* and ILP are solution approaches
- Triangle inequality → GED = GED-M

**Stop wasting time on**:

- ILP formulation details (variables, constraints)
- Proof that F_ORI is strongest
- Computational results and benchmarks

---

## Active Recall

**Q1**: What's the difference between Frobenius Distance and Graph Edit Distance?

**A1**: Frobenius permutes vertices to minimize adjacency matrix mismatches (sum of squared differences). GED allows explicit insertions, deletions, and substitutions with costs. Both are NP-hard, but GED is more flexible.

**Q2**: Why do we restrict GED to node maps?

**A2**: Searching over all possible edit paths is intractable. Node maps define a bijection-like correspondence between vertices that induces a unique set of edit operations. If costs satisfy triangle inequality, this restriction doesn't change the optimal GED value.

---
