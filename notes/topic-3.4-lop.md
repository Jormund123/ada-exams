# Linear Ordering Problem (LOP) — From Zero to Exam Ready

---

## Table of Contents

1. [What is the Linear Ordering Problem?](#what-is-the-linear-ordering-problem)
2. [ILP Formulation for LOP](#ilp-formulation-for-lop)
3. [Why 3-Cycle Constraints Suffice](#why-3-cycle-constraints-suffice)
4. [Solving LOP with Branch-and-Cut](#solving-lop-with-branch-and-cut)
5. [Exercise 10.1: LOP ILP Proof](#exercise-101-lop-ilp-proof)

---

## What is the Linear Ordering Problem?

> ⚠️ **Exam question**: "Write down a binary integer programme."
> **Student's answer**: "One binary programme is the linear ordering problem."
>
> ⚠️ **Exam question**: "What does it do?"
> **Answer**: "In LOP, we have a fully connected graph. We want to search for a set of edges that contains exactly half the edges such that every vertex is totally ordered compared to every other."

---

### Definition (Graph Version)

**Given:**

- Complete directed graph $G = (V, A)$ with $n$ vertices
- Arc weights $c_{uv}$ for all arcs $(u,v) \in A$

**Find:**

- A **linear ordering** (permutation) of the vertices
- Such that the sum of weights of arcs **respecting** this ordering is **maximized**

---

### What is a "Linear Ordering"?

A linear ordering is simply putting all vertices in a line:

$$\pi = (v_1, v_2, v_3, \ldots, v_n)$$

meaning: $v_1$ comes first, then $v_2$, then $v_3$, etc.

An arc $(u, v)$ **respects** the ordering if $u$ comes before $v$ in the permutation.

---

### Example with 3 Vertices

```
Vertices: 1, 2, 3

Complete directed graph (both directions for each pair):
  → arc (1,2) with cost c₁₂
  ← arc (2,1) with cost c₂₁
  → arc (1,3) with cost c₁₃
  ← arc (3,1) with cost c₃₁
  → arc (2,3) with cost c₂₃
  ← arc (3,2) with cost c₃₂

If we choose ordering π = (2, 1, 3):
  "Respecting" arcs: (2,1), (2,3), (1,3)
  Total weight: c₂₁ + c₂₃ + c₁₃
```

---

### Alternative Definition (Matrix Version)

**Given:** $n \times n$ matrix $C$ with entries $c_{ij}$

**Find:** Permutation $\pi$ of rows and columns maximizing:

$$\max \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} c_{\pi(i)\pi(j)}$$

This is the sum of all entries in the **upper triangle** after permutation.

---

### Applications

- **Tournament ranking** (from pairwise comparisons)
- **Crossing minimization** in graph drawing
- **Triangulation** of input-output matrices in economics
- **Recommender systems**, web search ranking

---

## ILP Formulation for LOP

> ⚠️ **Exam question**: "Write down the problem for a graph with three vertices."

---

### The Key Insight

A linear ordering is equivalent to a **spanning acyclic tournament**.

**What is a tournament?** For each pair of vertices $u, v$, we pick EXACTLY ONE direction:

- Either $(u, v)$ is selected
- Or $(v, u)$ is selected
- But NOT BOTH

**What does "acyclic" mean?** No directed cycles (no $u \to v \to w \to u$).

---

### Variables

For each arc $(u, v)$ in the complete directed graph, define:

$$x_{uv} = \begin{cases} 1 & \text{if arc } (u,v) \text{ is selected (i.e., } u \text{ comes before } v \text{)} \\ 0 & \text{otherwise} \end{cases}$$

---

### Objective Function

Maximize total weight of selected arcs:

$$\max \sum_{(u,v) \in A} c_{uv} \cdot x_{uv}$$

> ⚠️ **Exam question**: "How do you calculate the total cost?"
> **Answer**: "You sum over the cost of all chosen edges."

---

### Constraint 1: Tournament (Exactly One Direction)

For each pair of vertices $u, v$:

$$x_{uv} + x_{vu} = 1$$

> ⚠️ **Exam question**: "For all vertices, either the edge from $s$ to $t$ or the other edge needs to be included. I write down $x_e + x_{e'} = 1$."

**Meaning:** Either $u$ comes before $v$, or $v$ comes before $u$. Never both, never neither.

---

### Constraint 2: No 3-Cycles (Acyclicity)

For each triple of vertices $u, v, w$:

$$x_{uv} + x_{vw} + x_{wu} \leq 2$$

> ⚠️ **Exam question**: "We also need to make sure that no cycles are included, so of the three edges in this cycle (1,2,3) we can only include 2. I write down $x_1 + x_2 + x_3 \leq 2$."

**Meaning:** We can't have all three arcs of a directed 3-cycle.

---

### Why Just 3-Cycles? (Not Longer Cycles)

**Theorem:** If all 3-cycles are excluded, then ALL cycles are excluded.

**Proof idea:** Any longer cycle can be decomposed into 3-cycles. If we forbid all 3-cycles, longer cycles automatically become impossible.

(This is Exercise 10.1a — we'll prove it below.)

---

### Complete ILP for LOP

```
max  Σ c_uv · x_uv

s.t. x_uv + x_vu = 1              for all pairs u,v     (tournament)
     x_uv + x_vw + x_wu ≤ 2       for all triples u,v,w (no 3-cycles)
     x_uv ∈ {0, 1}                for all arcs
```

---

### Simplified Form (Only Variables for $u < v$)

To avoid redundancy, we can use only variables $x_{uv}$ where $u < v$ (comparing vertex IDs).

Then set $x_{vu} = 1 - x_{uv}$.

**Constraints become:**

For $u < v < w$:
$$x_{uv} + x_{vw} - x_{uw} \leq 1$$
$$x_{uv} + x_{vw} - x_{uw} \geq 0$$

(These are the "3-cycle inequalities" in simplified form.)

---

### Example: LOP on 3 Vertices

**Variables:** $x_{12}, x_{13}, x_{23}$ (using $u < v$ convention)

**Objective:** $\max \ c_{12}x_{12} + c_{21}(1-x_{12}) + c_{13}x_{13} + c_{31}(1-x_{13}) + c_{23}x_{23} + c_{32}(1-x_{23})$

**Constraints:**
$$x_{12} + x_{23} - x_{13} \leq 1$$
$$x_{12} + x_{23} - x_{13} \geq 0$$

**The 6 possible orderings and their characteristic vectors:**

| Permutation | $x_{12}$ | $x_{13}$ | $x_{23}$ |
| ----------- | -------- | -------- | -------- |
| (1,2,3)     | 1        | 1        | 1        |
| (1,3,2)     | 1        | 1        | 0        |
| (2,1,3)     | 0        | 1        | 1        |
| (2,3,1)     | 0        | 0        | 1        |
| (3,1,2)     | 1        | 0        | 0        |
| (3,2,1)     | 0        | 0        | 0        |

---

## Why 3-Cycle Constraints Suffice

### Exercise 10.1(a): Prove 3-Cycles Exclude All Cycles

**Claim:** If you forbid all 3-cycles, then no cycle of ANY length can exist.

**Proof by Contradiction:**

Assume we have a valid solution (no 3-cycles) but there exists a cycle of length $k > 3$:

$$v_1 \to v_2 \to v_3 \to \ldots \to v_k \to v_1$$

Consider vertices $v_1, v_2, v_3$ in this cycle:

- We have $x_{v_1 v_2} = 1$ (arc in cycle)
- We have $x_{v_2 v_3} = 1$ (arc in cycle)

From the tournament constraint: either $x_{v_1 v_3} = 1$ or $x_{v_3 v_1} = 1$.

**Case 1:** $x_{v_1 v_3} = 1$

- Then we can "shortcut" the cycle: $v_1 \to v_3 \to \ldots \to v_k \to v_1$
- This is a cycle of length $k-1$

**Case 2:** $x_{v_3 v_1} = 1$

- Then $v_1 \to v_2 \to v_3 \to v_1$ forms a 3-cycle
- **Contradiction!** (We assumed no 3-cycles)

By induction, keep shortening until we get a 3-cycle → contradiction.

Therefore, forbidding 3-cycles forbids ALL cycles. ∎

---

### Exercise 10.1(b): LP Relaxation and Larger Cycles

**Question:** For the LP relaxation, would adding cycle inequalities of larger size change the polytope?

**Answer for small $n$:**

- For $n < 6$: The LP relaxation of LOP is already **integer**!
- The 3-cycle inequalities completely describe the convex hull.
- No need for additional constraints.

**Answer for $n \geq 6$:**

- Additional constraints (like Möbius ladder inequalities) ARE needed.
- For $n = 6$: The LP relaxation can have fractional optima.
- Adding longer cycle inequalities or Möbius ladder constraints tightens the polytope.

---

## Solving LOP with Branch-and-Cut

### Why Can't We Just Solve the ILP?

The number of constraints grows polynomially:

- Tournament constraints: $O(n^2)$
- 3-cycle constraints: $O(n^3)$

For $n = 79$: This is tractable! Branch-and-Cut solves it in ~3 seconds.

---

### Separation for Cycle Inequalities

**Separation Problem:** Given LP solution $x^*$, find a violated cycle inequality.

**For LOP (3-cycles only):** Just enumerate all $O(n^3)$ triples and check!

**For Acyclic Subgraph Problem:** Use shortest path algorithms:

1. For each arc $f = (u,v)$
2. Find shortest $(v,u)$-path with edge weights $1 - x_e$
3. If path length $+ (1 - x_{uv}) < 1$: violated inequality found!

---

### The Algorithm

```
LOP Branch-and-Cut:

1. Initialize LP with tournament constraints only (no cycle constraints)

2. Solve LP → get solution x*

3. If x* is integral: DONE

4. Separate: Find violated 3-cycle constraints
   - Check all O(n³) triples
   - Add violated constraints to LP
   - Goto 2

5. If no violated constraints but x* fractional:
   BRANCH on a fractional variable
   Goto 2 for each child
```

---

## LOP as a Combinatorial Problem

> ⚠️ **Exam question**: "How does the combinatorial problem for this look?"
>
> ⚠️ **Exam question**: "What is the base set E?"
> **Answer**: "All six edges, we want to choose from those edges."
>
> ⚠️ **Exam question**: "How does the feasible set I look like?"
> **Answer**: "The edge combinations that fulfil the conditions, i.e., exactly three edges that do not form a circle."

**COP formulation of LOP (on 3 vertices):**

| Component                       | Description                                                         |
| ------------------------------- | ------------------------------------------------------------------- |
| **Ground set $E$**              | All 6 directed arcs: $\{(1,2), (2,1), (1,3), (3,1), (2,3), (3,2)\}$ |
| **Feasible sets $\mathcal{I}$** | All arc sets of size 3 that form a tournament with no 3-cycles      |
| **Cost function**               | $c(I) = \sum_{(u,v) \in I} c_{uv}$                                  |

---

## Quick Reference — Write From Memory

```
LOP Definition:
    Complete directed graph, arc weights c_uv
    Find: linear ordering maximizing sum of "respecting" arcs

ILP:
    max Σ c_uv · x_uv

    s.t. x_uv + x_vu = 1          (tournament: exactly one direction)
         x_uv + x_vw + x_wu ≤ 2   (no 3-cycles)
         x ∈ {0,1}

Key facts:
    - 3-cycle constraints suffice (longer cycles automatically excluded)
    - For n < 6: LP relaxation is already integer
    - For n ≥ 6: need Möbius ladder inequalities

Separation:
    - Enumerate all O(n³) triples, check for violated 3-cycle
    - Polynomial time!
```

---

## Exam Triage

**Exam-critical (go deep):**

- [What is LOP](#what-is-the-linear-ordering-problem): definition, what it finds
- [ILP formulation](#ilp-formulation-for-lop): tournament + 3-cycle constraints
- [Writing LOP for 3 vertices](#example-lop-on-3-vertices)
- [COP formulation](#lop-as-a-combinatorial-problem): ground set, feasible set

**Secondary:**

- Möbius ladder inequalities (mentioned, not asked in detail)
- PACE challenge details

**Stop wasting time on:**

- Graph drawing applications (just context)
- Exact polyhedral combinatorics proofs

---
