# Memorization Reference — Topic 1

Everything you must be able to write from memory on the whiteboard.

---

## 1. Dijkstra's Algorithm

```
Dijkstra(s):
    dist[s] = 0; InsertQ(s, 0)
    for all v ≠ s: dist[v] = ∞; InsertQ(v, ∞)

    while Q ≠ ∅:
        v = ExtractMinQ()
        for each edge (v, w):
            if dist[w] > dist[v] + c(v,w):
                dist[w] = dist[v] + c(v,w)
                DecreasePrioQ(w, dist[w])
    return dist
```

**Runtime:**

- Binary Heap: O((n+m) log n)
- Fibonacci Heap: O(m + n log n)

---

## 2. Brandes' Algorithm — CRITICAL

```
Brandes():
    for each v ∈ V: c_B(v) = 0

    for each s ∈ V:
        // PHASE 1: Forward BFS/Dijkstra
        S = empty stack
        pred[w] = empty list for all w
        σ[s] = 1; σ[w] = 0 for w ≠ s
        dist[s] = 0; dist[w] = ∞ for w ≠ s
        Q = priority queue with s

        while Q ≠ ∅:
            v = ExtractMin(Q)
            S.push(v)

            for each neighbor w of v:
                if dist[w] > dist[v] + c(v,w):
                    dist[w] = dist[v] + c(v,w)
                    UpdateQ(w, dist[w])

                if dist[w] == dist[v] + c(v,w):
                    σ[w] += σ[v]
                    pred[w].append(v)

        // PHASE 2: Backward accumulation
        δ[v] = 0 for all v

        while S ≠ ∅:
            w = S.pop()
            for each v ∈ pred[w]:
                δ[v] += (σ[v]/σ[w]) · (1 + δ[w])
            if w ≠ s:
                c_B(w) += δ[w]

    return c_B
```

**Runtime:** O(|V||E|) unweighted, O(|V||E| + |V|² log |V|) weighted

---

## 3. The Dependency Recursion Formula

$$\delta_{s\bullet}(v) = \sum_{w: v \in pred_s(w)} \frac{\sigma_{sv}}{\sigma_{sw}} \cdot (1 + \delta_{s\bullet}(w))$$

**In code:**

```
δ[v] += (σ[v]/σ[w]) × (1 + δ[w])
```

---

## 4. Brandes One Table Method (by hand)

| #   | Vertex | Dist | σ   | Parents | δ   | c_B |
| --- | ------ | ---- | --- | ------- | --- | --- |

**Steps:**

1. Fill Dist, σ, Parents using Dijkstra
2. Add # column (farthest = 1)
3. Process rows 1, 2, 3...
   - For each parent p: δ[p] += (σ[p]/σ[w]) × (1 + δ[w])
   - c_B[w] = δ[w] (unless source)
4. Repeat for all sources, sum c_B

---

## 5. Centrality Formulas

**Closeness:**
$$c_C(v) = \frac{1}{\sum_u d(v,u)}$$

**Betweenness:**
$$c_B(v) = \sum_{s \neq v \neq t} \frac{\sigma_{st}(v)}{\sigma_{st}}$$

---

## 6. Pairing Heap Operations

| Operation   | Time     |
| ----------- | -------- |
| GetMin      | O(1)     |
| Insert      | O(1)     |
| Merge/Meld  | O(1)     |
| DeleteMin   | O(log n) |
| DecreaseKey | O(log n) |

**Merge:** Smaller root becomes child of larger root (leftmost)

**DeleteMin:** Remove root, two-pass merge on children

---

## 7. Tree Betweenness — O(n)

**Post-order DFS for subtree sizes:**

```
PostOrderDFS(v):
    subtree_size[v] = 1
    for each child c of v:
        PostOrderDFS(c)
        subtree_size[v] += subtree_size[c]
```

**Edge betweenness:** C_B(e) = left_size × right_size

**Vertex betweenness:** C_B(v) = sum of (size_i × size_j) for all pairs of subtrees

---

## 8. Topological Sort (for DAGs)

```
TopologicalSort():
    Run DFS
    Add vertices to list in post-order (when finished)
    Reverse the list
```

**Use:** Process vertices in order, all predecessors done → O(n+m) shortest paths on DAGs

---

## 9. Key Runtime Summary

| Algorithm            | Time             | Space    |
| -------------------- | ---------------- | -------- |
| Dijkstra (Fib heap)  | O(m + n log n)   | O(n)     |
| Brandes (unweighted) | O(VE)            | O(V + E) |
| Brandes (weighted)   | O(VE + V² log V) | O(V + E) |
| Basic betweenness    | O(V³)            | O(V²)    |
| Tree betweenness     | O(n)             | O(n)     |

---

## 10. Bellman Criterion — When v is on shortest s-t path

$$d(s,t) = d(s,v) + d(v,t) \implies v \text{ lies on some shortest s-t path}$$

**Path count through v:**
$$\sigma_{st}(v) = \sigma_{sv} \cdot \sigma_{vt}$$

---

## 11. Edge Betweenness Formula

$$C_B(e) = \sum_{s,t} \frac{\sigma_{st}(e)}{\sigma_{st}}$$

**In Brandes**: The term (σ[v]/σ[w]) · (1 + δ[w]) **already** counts paths through edge (v,w).

Just add: `c_B_edge[(v,w)] += (σ[v]/σ[w]) · (1 + δ[w])`

---

## 12. Fan Graph F(n₁, n₂) Betweenness

```
  v₁..vₙ₁ ─── a ─── b ─── c ─── w₁..wₙ₂
```

**Betweenness of b:** $c_B(b) = n_1 \cdot n_2 + n_1 + n_2$

**Betweenness of a:** $c_B(a) = \frac{n_1(n_1-1)}{2} + n_1(2 + n_2)$

---

## 13. Common Exam Traps

| Trap                                                                | Answer                                                                 |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| "Why process vertices in non-increasing distance order in Phase 2?" | Recursion needs δ[w] computed BEFORE updating δ[v]                     |
| "Why does DecreasePrioQ being O(1) matter?"                         | Called m times; saves log n factor on dense graphs                     |
| "What does δ[v] represent?"                                         | Total betweenness contribution from source s to all destinations via v |
| "Why is space O(V+E) not O(V²)?"                                    | Dependency recursion avoids storing full pairwise matrix               |

---

## 14. Fibonacci Heap vs Pairing Heap

| Operation   | Fibonacci  | Pairing  |
| ----------- | ---------- | -------- |
| DecreaseKey | O(1) amort | O(log n) |

**Key**: Fib heap makes Dijkstra O(m + n log n) instead of O(m log n).

---

# Memorization Reference — Topic 2 (Graph Similarity)

---

## 10. Naive WL Algorithm — CRITICAL

```
WL-Algorithm:
    Initial: All vertices get the SAME color (e.g., color 1)

    Repeat until stable:
        For each vertex v:
            new_color[v] = hash(old_color[v], sorted list of neighbor colors)
        old_color = new_color
```

**Runtime:** O(n³) — up to n iterations, each O(n²)

---

## 11. Efficient WL Algorithm

```
EfficientWL:
    Q = queue of color classes to refine

    while Q not empty:
        R = pop from Q (refining set)
        for each color class S with neighbors in R:
            refine S by counting neighbors in R
            split S into new classes S₁, S₂, ..., Sₖ
            add all but the LARGEST new class to Q
```

**Runtime:** O(n² log n) — each vertex in Q at most O(log n) times

---

## 12. Graph Isomorphism Definition

Two graphs G and H are **isomorphic** (G ≃ H) if:

$$\exists \pi: V(G) \to V(H) \text{ bijection such that } (u,v) \in E(G) \Leftrightarrow (\pi(u), \pi(v)) \in E(H)$$

---

## 13. Equitable Partition Definition

Partition C = {C₁, ..., Cₗ} is **equitable** if:

$$\forall i,j, \forall u,v \in C_i : |N(u) \cap C_j| = |N(v) \cap C_j|$$

**In words:** Vertices in the same cell have the same number of neighbors in every cell.

**WL computes the coarsest equitable partition.**

---

## 14. k-WL Algorithm

```
Initial: k-sets U, W get same color if G[U] ≃ G[W]

Iteration: Refine by counting neighbors of each color
           Neighbors = k-sets differing in exactly one element
```

**Runtime:** O(k² · n^(k+1) · log n)

---

## 15. k-LWL (Local k-WL)

**Difference:** Neighbors must be connected by edges (not any element swap)

**Advantages:**

1. Faster (exploits sparsity)
2. At least as strong as k-WL
3. Sometimes strictly stronger

---

## 16. WL Runtime Summary

| Algorithm      | Time                       |
| -------------- | -------------------------- |
| Naive WL       | O(n³)                      |
| Efficient WL   | O(n² log n)                |
| Even better WL | O((n+m) log n)             |
| k-WL           | O(k² · n^(k+1) · log n)    |
| k-LWL          | Faster (exploits sparsity) |

---

## 17. WL Iterations on Path

On a path with n vertices: **⌈n/2⌉ iterations**

---

## 18. Key Facts to State

- WL is **one-sided**: Different histograms → NOT isomorphic. Same histograms → UNKNOWN.
- WL fails on **regular graphs** (all same degree → no refinement)
- k-WL (k ≥ 2) can distinguish regular graphs
- For large k, k-WL distinguishes ALL graphs (but exponential runtime)

---
