# ILP Solution Methods — From Zero to Exam Ready

---

## Table of Contents

1. [Why Do We Need Solution Methods?](#why-do-we-need-solution-methods)
2. [Branch-and-Bound](#branch-and-bound)
3. [LP Relaxation](#lp-relaxation)
4. [Branch-and-Bound with LP Relaxation](#branch-and-bound-with-lp-relaxation)
5. [Cutting Plane Method](#cutting-plane-method)
6. [Branch-and-Cut](#branch-and-cut)
7. [Exercise 9.1: Simple Branch-and-Bound](#exercise-91-simple-branch-and-bound)
8. [Exercise 9.2: TSP Separation](#exercise-92-tsp-separation)
9. [Exercise 9.3: Branch-and-Cut for Independent Set](#exercise-93-branch-and-cut-for-independent-set)

---

## Why Do We Need Solution Methods?

> **The problem:** ILPs are NP-hard. We can't just "solve" them directly like regular LPs.

**But NP-hard does NOT mean:**

- ✗ Problems are unsolvable
- ✗ Only heuristics can be applied
- ✗ Runtime must always be long

**What it DOES mean:**

- We need smart enumeration (not brute force)
- We can often solve instances in reasonable time

---

### The Three Main Methods

| Method               | Idea                                              |
| -------------------- | ------------------------------------------------- |
| **Branch-and-Bound** | Divide problem, prune subtrees that can't improve |
| **Cutting Planes**   | Add constraints to tighten LP relaxation          |
| **Branch-and-Cut**   | Combine both methods                              |

---

## Branch-and-Bound

### Core Idea

**Divide and conquer** + **early pruning**.

Instead of checking ALL $2^n$ possible 0/1 combinations, we:

1. Build a search tree (branching on variables)
2. At each node, compute bounds
3. **Prune** entire subtrees that can't contain a better solution

---

### The Two Bounds

For a **minimization** problem:

| Bound             | Meaning                                      | Also called  |
| ----------------- | -------------------------------------------- | ------------ |
| **Upper bound U** | Best solution we've FOUND so far             | Primal bound |
| **Lower bound L** | Best we could POSSIBLY get from this subtree | Dual bound   |

**Key insight:** If $L \geq U$ at some node → **PRUNE** that subtree!

Why? If the BEST we could possibly get ($L$) is already worse than what we already have ($U$), why explore further?

---

### Visual: MST on K₃ Example

**Problem:** Minimize $x_1 + 2x_2 + 3x_3$ subject to $x_1 + x_2 + x_3 = 2$

```
        (root)
        (.,.,.)
       L=3, U=5
       /      \
      /        \
    x₁=1      x₁=0
     /          \
  (1,.,.)     (0,.,.)
  L=3,U=4     L=5,U=5
   /   \        |
  /     \       |
x₂=1   x₂=0   DONE!
 |       |    (L=U)
(1,1,.)  (1,0,.)
L=3,U=3  L=4,U=4
  |        |
 DONE!    DONE!
(L=U)    (L=U)
```

**What happens:**

1. At root: Best found so far U=5, lower bound L=3
2. Branch on $x_1$
3. Left subtree ($x_1=1$): L=3, U=4 → keep exploring
4. At node (1,1,.): L=3, U=3 → L=U → OPTIMAL for this subtree!
5. Right subtree ($x_1=0$): L=5, U=5 → L=U → done, no need to go deeper

**Result:** We found optimal (1,1,0) with value 3, without exploring all 8 leaves!

---

### How to Calculate Bounds?

| Bound           | How to compute                               |
| --------------- | -------------------------------------------- |
| **Upper bound** | Any feasible solution (greedy, heuristic)    |
| **Lower bound** | LP relaxation (remove integrality, solve LP) |

The better your bounds, the more pruning you get!

---

## LP Relaxation

### Definition

> **LP Relaxation:** Remove the integrality constraints from an ILP.

**Original ILP:**

```
min  c^T x
s.t. Ax ≤ b
     x ∈ {0,1}^n    ← integrality constraint
```

**LP Relaxation:**

```
min  c^T x
s.t. Ax ≤ b
     0 ≤ x ≤ 1      ← replace with simple bounds
```

---

### Why Does This Help?

**Key observation:** LP relaxation is EASIER than ILP (polynomial time solvable).

**Key property:** The LP solution provides a **bound** on the ILP solution.

| Problem type | LP relaxation gives                                              |
| ------------ | ---------------------------------------------------------------- |
| Minimization | **Lower bound** (LP can only do better, since fewer constraints) |
| Maximization | **Upper bound**                                                  |

---

### Visual: LP Relaxation

```
         y
         │
       4 ┼         ┌─────────────┐
         │         │ P_R         │ ← LP relaxation polytope
       3 ┼         │   ●─────●   │
         │         │   │     │   │
       2 ┼         │   ●─────●   │ ← ILP feasible points
         │         │             │
       1 ┼         └─────────────┘
         │
       0 ┼───┬───┬───┬───┬───┬───► x
             1   2   3   4   5

P (ILP feasible) ⊆ P_R (LP relaxation)

Therefore: min over P_R ≤ min over P
```

The LP relaxation is a **larger** feasible region. So its optimum is at least as good as the ILP optimum.

---

### Three Cases After Solving LP Relaxation

| Case                          | What happened                       | What to do                      |
| ----------------------------- | ----------------------------------- | ------------------------------- |
| LP solution is **integer**    | Lucky! This is also ILP optimal     | **STOP** — we're done           |
| LP solution is **fractional** | Need to do more work                | Branch OR add cutting planes    |
| LP is **infeasible**          | No integer solution in this subtree | **PRUNE** — discard this branch |

---

## Branch-and-Bound with LP Relaxation

### The Algorithm

```
Branch-and-Bound with LP Relaxation:

1. Initialize:
   - Active nodes = {root}
   - Best solution = ∞ (for minimization)

2. While active nodes exist:
   a. Pick a node (subproblem)

   b. Solve LP relaxation → get solution x' and lower bound L

   c. If L ≥ best solution:
      PRUNE (this subtree can't improve)

   d. If LP infeasible:
      PRUNE (no integer solutions here)

   e. If x' is integer:
      Update best solution if x' is better
      PRUNE (this subtree is solved)

   f. If x' is fractional:
      Pick a fractional variable x_i
      Create two child nodes:
        - Add constraint x_i = 0
        - Add constraint x_i = 1
      Add both to active nodes

3. Return best solution
```

---

### Example Trace

```
Minimization problem

Node 0: Solve LP → L=0 (fractional x')
        Branch on x_i

Node 1: x_i=1, Solve LP → L=10.5 (fractional)
        Branch on x_j

Node 2: x_i=0, Solve LP → L=12.8 (fractional)
        Branch on x_k

Node 3: x_i=1, x_j=1, Solve LP → L=14 (INTEGER!)
        Update best = 14
        PRUNE (done with this node)

Node 4: x_i=1, x_j=0, Solve LP → INFEASIBLE
        PRUNE

Node 5: x_i=0, x_k=1, Solve LP → L=15 (fractional)
        But L=15 ≥ best=14
        PRUNE! (can't beat 14)
```

---

## Cutting Plane Method

### The Problem

LP relaxation gives a bound, but often the solution is fractional.

**Idea:** Add more constraints to "cut off" the fractional solution, without removing any integer points.

---

### What is a Cutting Plane?

> A **cutting plane** is a linear inequality that:
>
> 1. Cuts off the current fractional LP solution
> 2. Does NOT cut off any feasible INTEGER solution

```
         y
         │
       4 ┼   ●━━━━━━━━━● z' (fractional optimum)
         │    ╲
       3 ┼     ╲  cutting plane
         │      ╲
       2 ┼   ●───●───●   ← integer points preserved
         │   │   │   │
       1 ┼   ●───●───●
         │
       0 ┼───┬───┬───┬──► x
             1   2   3
```

The cutting plane "chops off" the fractional point $z'$ but keeps all integer points.

---

### The Algorithm

```
Cutting Plane Method:

1. Start with LP relaxation (subset of constraints)

2. Solve LP → get solution x'

3. If x' is integer: STOP (optimal!)

4. Find a violated cutting plane:
   - A constraint that x' violates
   - But all integer solutions satisfy

5. Add cutting plane to LP, goto (2)
```

---

### The Separation Problem

**Key question:** How do we FIND a cutting plane?

> **Separation Problem:**
> Given a point $x'$ and a class of constraints $C$,
> find a constraint from $C$ that $x'$ violates,
> or prove that no such constraint exists.

**Theorem (Grötschel, Lovász, Schrijver):**

> An LP over a class of constraints $C$ can be solved in polynomial time **if and only if** the separation problem for $C$ can be solved in polynomial time.

This is WHY cutting planes work in practice — we can efficiently separate for common constraint classes.

---

## Branch-and-Cut

### Combining Both Methods

**Branch-and-Cut = Branch-and-Bound + Cutting Planes**

At each node of the B&B tree:

1. Solve LP relaxation
2. Try to add cutting planes to improve the bound
3. If still fractional and no more cuts, BRANCH

---

### The Algorithm

```
Branch-and-Cut:

1. CUT: Solve current LP (maybe add cutting planes)

2. BRANCH: If LP solution is fractional and no violated
           cutting planes found → branch on a variable

3. EXPLOIT: Try to find a feasible integer solution
            from the fractional one (primal heuristic)

4. BOUND: Prune subtrees with L ≥ best solution

5. (Advanced) PRICE: For very large problems, use
                     column generation
```

---

### Why is This Better?

| Method              | Weakness                                          |
| ------------------- | ------------------------------------------------- |
| Pure B&B            | Many nodes, weak bounds from LP relaxation        |
| Pure Cutting Planes | May need many iterations, can be slow             |
| **Branch-and-Cut**  | Best of both: tight bounds + controlled branching |

---

## Exercise 9.1: Simple Branch-and-Bound — **Important**

### Problem

Consider the ILP for odd $n$:
$$\min x_{n+1}$$
subject to:
$$2 \cdot \sum_{i=1}^{n} x_i + x_{n+1} = n$$
$$x \in \{0,1\}^{n+1}$$

---

### Part (a): LP Relaxation Solution

**Question:** What is the optimal solution of the LP relaxation?

**Setup:** We have $n+1$ variables. The constraint says:
$$2x_1 + 2x_2 + \ldots + 2x_n + x_{n+1} = n$$

We want to minimize $x_{n+1}$.

**Observation:** To minimize $x_{n+1}$, we want to make the left sum as large as possible.

**Strategy:** Set $x_1 = x_2 = \ldots = x_n = \frac{1}{2}$ (use as much as allowed from each).

Then: $2 \cdot n \cdot \frac{1}{2} + x_{n+1} = n$
Thus: $n + x_{n+1} = n$
So: $x_{n+1} = 0$

**LP relaxation optimal solution:**
$$x_i = \frac{1}{2} \text{ for } i = 1, \ldots, n, \quad x_{n+1} = 0$$
$$\text{Optimal LP value} = 0$$

---

### Part (b): Branch-and-Bound Tree

**Key insight:** If we set any $x_i$ to an integer (0 or 1), the constraint forces others.

**Case $x_1 = 0$:**

- Remaining: $2(x_2 + \ldots + x_n) + x_{n+1} = n$
- Still need LP value $\geq 0$

**Case $x_1 = 1$:**

- Remaining: $2 + 2(x_2 + \ldots + x_n) + x_{n+1} = n$
- So: $2(x_2 + \ldots + x_n) + x_{n+1} = n - 2$

**The tree structure:**

Every branching decision on some $x_i$ leads to two subproblems. Since $n$ is odd and we need the sum to equal $n$, eventually we must set $x_{n+1}$ to 1.

**Number of subproblems:** The tree has depth at most $n+1$, with $2^{n+1}$ potential nodes. With LP bounds, many get pruned.

**Analysis:**

- When LP relaxation gives $x_{n+1} = 0$ but all other $x_i$ are fractional, we must branch.
- Each branch fixes one variable, creating an exponential tree in worst case.
- LP relaxation at each node provides a lower bound of 0 until we're forced to set $x_{n+1} = 1$.

---

## Exercise 9.2: TSP Separation — **Important**

### TSP ILP Formulation Recap

$$\min \sum_{e \in E} c_e x_e$$

subject to:

1. **Degree constraints:** $x(\delta(v)) = 2$ for all $v \in V$
2. **Subtour elimination:** $x(\delta(S)) \geq 2$ for all $\emptyset \neq S \subsetneq V$
3. **Bounds:** $x_e \in [0,1]$

**Notation:**

- $\delta(v)$ = edges incident to vertex $v$
- $\delta(S)$ = edges with exactly one endpoint in $S$ (the "cut")
- $x(F) = \sum_{e \in F} x_e$

---

### Part (a): Why Can't We Solve This LP Directly?

**Answer:** There are **exponentially many** subtour elimination constraints!

For a graph with $n$ vertices:

- Number of subsets $S$: $2^n - 2$ (excluding $\emptyset$ and $V$)
- Each gives a constraint → can't write them all down

---

### Part (b): The Cutting Plane Approach

**Strategy:**

1. Start with LP containing ONLY degree constraints (2n constraints)
2. Solve LP → get solution $x^*$
3. Check if any subtour constraint is violated
4. If yes: add violated constraint, goto (2)
5. If no: we have optimal TSP LP solution

**The Separation Problem:**

> Given fractional solution $x^*$, find a set $S$ such that:
> $$x^*(\delta(S)) < 2$$
> or prove that no such $S$ exists.

**How to Solve It (Polynomial Time):**

This is the **minimum s-t cut problem**!

1. Build graph $G$ with edge weights $x^*_e$
2. For each pair of vertices $s, t$:
   - Find minimum $s$-$t$ cut value
   - If cut value $< 2$: found a violated constraint!
3. Use min-cut algorithm (e.g., based on max-flow)

**Why?**

- An $s$-$t$ cut is a set $S$ with $s \in S$, $t \notin S$
- The cut value is $x^*(\delta(S))$
- If minimum cut $< 2$, the constraint $x(\delta(S)) \geq 2$ is violated

**Runtime:** $O(n^2)$ min-cut computations, each polynomial → total polynomial.

---

## Exercise 9.3: Branch-and-Cut for Independent Set — **Important**

### Maximum Independent Set (MIS) Problem

**Given:** Graph $G = (V, E)$

**Find:** Largest set $S \subseteq V$ such that no two vertices in $S$ are adjacent.

---

### Part (a): ILP Formulation

**Variables:** For each vertex $v$:
$$x_v = \begin{cases} 1 & \text{if } v \in S \\ 0 & \text{otherwise} \end{cases}$$

**Objective:** Maximize size of $S$:
$$\max \sum_{v \in V} x_v$$

**Constraints:** No two adjacent vertices both in $S$:
$$x_u + x_v \leq 1 \quad \forall \{u, v\} \in E$$

**Complete ILP:**

```
max  Σ x_v

s.t. x_u + x_v ≤ 1    for all {u,v} ∈ E
     x_v ∈ {0, 1}     for all v ∈ V
```

---

### Part (b): Odd Cycle Constraints

**Key observation:** In a cycle $C = (v_1, v_2, \ldots, v_k, v_1)$, an independent set can contain at most $\lfloor k/2 \rfloor$ vertices.

**Odd cycle constraint:** For any odd cycle $C$ of length $k$ (odd):
$$\sum_{v \in C} x_v \leq \frac{k-1}{2}$$

**Example:** For a triangle (3-cycle):
$$x_1 + x_2 + x_3 \leq 1$$

Only ONE vertex from a triangle can be in an independent set!

---

### Part (c): Why Only ODD Cycles Matter?

For **even** cycles (length $k$ even):
$$\sum_{v \in C} x_v \leq \frac{k}{2}$$

**But:** The edge constraints $x_u + x_v \leq 1$ already imply this!

**Why?** In an even cycle, we can take alternating vertices. The edge constraints ensure at most one vertex per edge, which gives exactly $k/2$ for an even cycle. So the cycle constraint is redundant.

For **odd** cycles: Edge constraints give bound $\lfloor k/2 \rfloor + \frac{1}{2}$ = $\frac{k}{2}$, but the true maximum is $\frac{k-1}{2}$. The odd cycle constraint is TIGHTER.

---

### Part (d): Separating Odd Cycle Constraints

**Separation problem:** Given LP solution $x^*$, find an odd cycle $C$ such that:
$$\sum_{v \in C} x^*_v > \frac{|C| - 1}{2}$$

**Reformulation:**
$$\sum_{v \in C} x^*_v > \frac{|C| - 1}{2}$$
$$\Leftrightarrow \sum_{v \in C} (1 - x^*_v) < \frac{|C| + 1}{2}$$

**Algorithm:**

1. Create graph with edge weights $w_e = 1 - x^*_u + 1 - x^*_v$ for edge $\{u,v\}$
2. Find minimum weight odd cycle (given as black-box)
3. If weight $< \frac{|C| + 1}{2}$: constraint violated, add it
4. Otherwise: no violated odd cycle constraint

---

### Part (e): Branching Strategy

**When we branch on a vertex $v$:**

**Case $x_v = 1$ (vertex IS in independent set):**

- All neighbors of $v$ must be OUT: $x_u = 0$ for all $u \in N(v)$
- Can propagate and potentially fix many variables

**Case $x_v = 0$ (vertex is NOT in independent set):**

- No immediate implications
- Less propagation

**Good branching choice:**

- Pick vertex with highest LP value (most fractional, closest to 1)
- Or pick vertex with highest degree (most propagation when fixed to 1)

**Drawback:**

- Fixing ONE vertex may not reduce problem much
- Consider branching on cliques or larger structures instead

---

## Exam Triage

**Exam-critical (go deep):**

- [Branch-and-Bound concept](#branch-and-bound): pruning when $L \geq U$
- [LP Relaxation](#lp-relaxation): why it gives bounds
- [Separation problem](#cutting-plane-method): finding violated constraints

**Secondary:**

- Specific algorithm implementation details
- Column generation / pricing

**Stop wasting time on:**

- Proving the Grötschel-Lovász-Schrijver theorem
- Implementing full B&C solver

---

## Quick Reference — Write From Memory

```
Branch-and-Bound:
    If L ≥ U: PRUNE (can't improve)
    If LP infeasible: PRUNE
    If LP integral: update best, PRUNE
    If LP fractional: BRANCH

LP Relaxation:
    Remove x ∈ {0,1}, add 0 ≤ x ≤ 1
    Gives LOWER bound (minimization)
    Gives UPPER bound (maximization)

Cutting Plane:
    1. Solve LP
    2. If fractional: find violated constraint
    3. Add constraint, repeat

Separation Problem:
    Given x', find violated constraint or prove none exists
    LP solvable in poly time ⟺ separation in poly time

MIS ILP:
    max Σ x_v
    s.t. x_u + x_v ≤ 1  for all edges

Odd cycle cut:
    Σ_{v∈C} x_v ≤ (|C|-1)/2  for odd cycle C
```

---
