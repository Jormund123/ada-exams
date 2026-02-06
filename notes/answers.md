# Exam Questions — Quick Answers Reference

All questions from `specs/questions.txt` with concise answers and links to detailed explanations.

---

## Topic A: Shortest Paths &amp; Speedup Techniques

### Q: Which speedup techniques for shortest path search do you know?

**Answer:**

- **A\*** — Uses heuristic to direct search toward target
- **Bidirectional Search** — Search from both start and target, meet in middle
- **ALT** — A\* with Landmarks and Triangle inequality
- **Contraction Hierarchies** — Contract unimportant vertices, add shortcuts
- **Arc Flags** — Precompute which edges lead to which regions
- **Hub Labeling** — Precompute distance labels for each vertex

📚 **Revision:** [topic-1.md → Speedup Techniques](topic-1.md#speedup-techniques)

---

### Q: What properties should a heuristic for A\* have?

**Answer:** The heuristic must be **feasible** (also called admissible):

- It must **never overestimate** the true cost to the goal
- May underestimate (that's okay)
- Ensures A\* finds optimal path

📚 **Revision:** [topic-1.md → A\*](topic-1.md#a-star)

---

### Q: How does bidirectional search work?

**Answer:**

- Run Dijkstra from **start** (forward) and from **target** (backward)
- Backward graph = same graph with edges reversed, same weights
- Two priority queues, always extract minimum from both
- Stop when: minimum distance found ≤ sum of top keys in both queues

📚 **Revision:** [topic-1.md → Bidirectional Search](topic-1.md#bidirectional-search)

---

### Q: How does ALT work and how are landmarks chosen?

**Answer:**

- Choose **landmarks** around the graph (often on periphery)
- Precompute distances from every vertex to all landmarks
- Heuristic: $h(u) = \max_l \{d(l,t) - d(l,u), d(u,l) - d(t,l)\}$
- Best landmark is "behind" the target relative to current position

📚 **Revision:** [topic-1.md → ALT](topic-1.md#alt)

---

### Q: How do contraction hierarchies work? What is witness search?

**Answer:**

1. Order vertices by "importance"
2. Contract least important vertex $v$: remove $v$, add shortcuts if needed
3. **Witness search:** Before adding shortcut $(u,w)$, check if path $u \to w$ exists without $v$
4. Query: bidirectional search, only relax edges going "upward" in hierarchy

📚 **Revision:** [topic-1.md → Contraction Hierarchies](topic-1.md#contraction-hierarchies)

---

## Topic A: Centrality

### Q: Which centrality measures did we discuss? Intuitions?

**Answer:**

| Measure         | Formula                                                              | Intuition                   |
| --------------- | -------------------------------------------------------------------- | --------------------------- |
| **Closeness**   | $c_C(v) = \frac{1}{\sum_u d(v,u)}$                                   | How close to everyone       |
| **Betweenness** | $c_B(v) = \sum_{s \neq v \neq t} \frac{\sigma_{st}(v)}{\sigma_{st}}$ | How often on shortest paths |

📚 **Revision:** [topic-1.md → Centrality](topic-1.md#centrality-measures)

---

### Q: How does Brandes' Algorithm work? (Write pseudocode)

**Answer:**

```
for each s ∈ V:
    // Phase 1: Forward BFS/Dijkstra
    σ[s] = 1; dist[s] = 0
    while Q not empty:
        v = extractMin(Q)
        S.push(v)
        for w ∈ neighbors(v):
            if dist[w] > dist[v] + c(v,w):
                update dist[w], pred[w]
            if dist[w] == dist[v] + c(v,w):
                σ[w] += σ[v]; pred[w].append(v)

    // Phase 2: Backward accumulation
    while S not empty:
        w = S.pop()
        for v ∈ pred[w]:
            δ[v] += (σ[v]/σ[w]) · (1 + δ[w])
        c_B[w] += δ[w]
```

**Runtime:** $O(VE)$ unweighted, $O(VE + V^2 \log V)$ weighted

📚 **Revision:** [topic-1.md → Brandes Algorithm](topic-1.md#brandes-algorithm) | [memorize.md → Item 2](memorize.md)

---

### Q: Prove Brandes produces correct results / Dependency recursion

**Answer:** The key formula:
$$\delta_{s\bullet}(v) = \sum_{w: v \in pred_s(w)} \frac{\sigma_{sv}}{\sigma_{sw}} \cdot (1 + \delta_{s\bullet}(w))$$

**Why?** Fraction $\frac{\sigma_{sv}}{\sigma_{sw}}$ = proportion of shortest $s \to w$ paths through $v$. The $(1 + \delta[w])$ accounts for paths ending at $w$ plus paths continuing beyond.

📚 **Revision:** [topic-1.md → Dependency Recursion](topic-1.md#dependency-recursion)

---

## Topic B: Weisfeiler-Lehman

### Q: What does WL algorithm do?

**Answer:** Finds the **coarsest stable partition** of color classes on a graph. Used for **isomorphism detection** — if color histograms differ, graphs are NOT isomorphic.

📚 **Revision:** [topic-2-wl.md → WL Algorithm](topic-2-wl.md#weisfeiler-lehman-algorithm)

---

### Q: What do you mean it can detect isomorphisms between some graphs but not others?

**Answer:** WL is **one-sided**:

- Different histograms → NOT isomorphic (definite)
- Same histograms → UNKNOWN (might be isomorphic or not)
- **Fails on regular graphs** (all vertices have same degree → no refinement)

📚 **Revision:** [topic-2-wl.md → WL Limitations](topic-2-wl.md#limitations)

---

### Q: What runtime does WL have?

**Answer:**
| Version | Runtime |
|---------|---------|
| Naive WL | $O(n^3)$ |
| Efficient WL | $O(n^2 \log n)$ |
| Even better | $O((n+m) \log n)$ |

📚 **Revision:** [topic-2-wl.md → Runtime](topic-2-wl.md#runtime) | [memorize.md → Item 16](memorize.md)

---

### Q: What is k-dimensional WL? What is it used for?

**Answer:**

- Colors **k-tuples** of vertices instead of single vertices
- Can distinguish graphs that 1-WL cannot (e.g., regular graphs for k≥3)
- Runtime: $O(k^2 \cdot n^{k+1} \cdot \log n)$

📚 **Revision:** [topic-2-wl.md → k-WL](topic-2-wl.md#k-wl)

---

### Q: What is local k-WL? How is it different?

**Answer:**

- Neighbors must be **connected by edges** (not any element swap)
- **Faster** (exploits sparsity)
- **At least as strong** as k-WL, sometimes strictly stronger

📚 **Revision:** [topic-2-wl.md → k-LWL](topic-2-wl.md#k-lwl)

---

## Topic C: Combinatorial Problems &amp; ILP

### Q: What are combinatorial problems?

**Answer:** We search for a **combination (subset)** of elements that maximizes or minimizes a goal function of the form:
$$c(I) = \sum_{e \in I} c_e$$
(Sum of partial costs)

📚 **Revision:** [topic-3.2-ilp.md → COP Definition](topic-3.2-ilp.md#combinatorial-optimization-problems)

---

### Q: This goal function has a certain form, how does it look?

**Answer:** It is the **sum of partial costs**:
$$c(I) = \sum_{e \in I} c_e$$
Not quadratic, not product, just a simple weighted sum.

📚 **Revision:** [topic-3.2-ilp.md → Cost Function](topic-3.2-ilp.md#formal-definition)

---

### Q: What does the Theorem of Minkowski-Weyl say?

**Answer:** Every polyhedron $P$ can be represented as:
$$P = \text{conv}(V) + \text{cone}(E)$$
where $V$ and $E$ are finite sets. Equivalently:
$$P = \{x \mid Ax \leq b\}$$

A polyhedron is the intersection of finitely many half-spaces.

📚 **Revision:** [topic-3.2-ilp.md → Minkowski-Weyl](topic-3.2-ilp.md#minkowski-weyl-theorem) | [memorize.md → Item 20](memorize.md)

---

### Q: Write down a binary integer programme

**Answer:** The **Linear Ordering Problem (LOP)**:

```
max  Σ c_uv · x_uv

s.t. x_uv + x_vu = 1          (tournament)
     x_uv + x_vw + x_wu ≤ 2   (no 3-cycles)
     x ∈ {0,1}
```

📚 **Revision:** [topic-3.4-lop.md → LOP ILP](topic-3.4-lop.md#ilp-formulation-for-lop) | [memorize.md → Item 34](memorize.md)

---

### Q: Write down the problem for a graph with three vertices (LOP)

**Answer:**

- Variables: $x_{12}, x_{13}, x_{23}$ (for pairs with $u < v$)
- Tournament: $x_{uv} + x_{vu} = 1$ (use $x_{vu} = 1 - x_{uv}$)
- No 3-cycle: $x_{12} + x_{23} + x_{31} \leq 2$ → $x_{12} + x_{23} - x_{13} \leq 1$
- Objective: $\max \sum c_{uv} x_{uv}$

📚 **Revision:** [topic-3.4-lop.md → LOP 3 vertices](topic-3.4-lop.md#example-lop-on-3-vertices)

---

### Q: What is the base set E for LOP (3 vertices)?

**Answer:** All **6 directed arcs**: $\{(1,2), (2,1), (1,3), (3,1), (2,3), (3,2)\}$

---

### Q: How does the feasible set I look like for LOP?

**Answer:** Edge combinations that:

1. Contain exactly **3 edges** (one per pair)
2. Form a **tournament** (exactly one direction per pair)
3. Do **not form a cycle**

📚 **Revision:** [topic-3.4-lop.md → LOP as COP](topic-3.4-lop.md#lop-as-a-combinatorial-problem)

---

## Topic C: Graph Streaming

### Q: What is the graph streaming model and what motivates it?

**Answer:**

- **Model:** Edges arrive one at a time, limited memory (can't store whole graph)
- **Motivation:** Massive graphs that don't fit in RAM
- **Goal:** Compute properties in $O(\log n)$ passes with $O(n \cdot \text{polylog } n)$ space

📚 **Revision:** [topic-3.3-ilp-methods.md] (not covered in current notes — streaming content may be separate)

---

### Q: How can we compute spanning trees in streaming model?

**Answer:** Union-Find with semi-streaming:

- Process edges one by one
- Keep edge if it connects two different components
- Space: $O(n)$ for Union-Find structure

**Runtime factor:** Extra $O(\log n)$ factor because we may need multiple passes.

---

## Quick Navigation

| Topic                    | Main File                                            | Memorize Section |
| ------------------------ | ---------------------------------------------------- | ---------------- |
| Shortest Paths           | [topic-1.md](topic-1.md)                             | Items 1-9        |
| Centrality               | [topic-1.md](topic-1.md)                             | Items 2-6        |
| Weisfeiler-Lehman        | [topic-2-wl.md](topic-2-wl.md)                       | Items 10-18      |
| Linear Programming       | [topic-3.1-lp.md](topic-3.1-lp.md)                   | Items 19-28      |
| ILP / BLP                | [topic-3.2-ilp.md](topic-3.2-ilp.md)                 | Items 19-28      |
| B&amp;B / Cutting Planes | [topic-3.3-ilp-methods.md](topic-3.3-ilp-methods.md) | Items 29-33      |
| Linear Ordering          | [topic-3.4-lop.md](topic-3.4-lop.md)                 | Items 34-36      |

---
