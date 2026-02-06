# ILP for Combinatorial Optimization — From Zero to Exam Ready

---

## Table of Contents

1. [Combinatorial Optimization Problems](#combinatorial-optimization-problems)
2. [Polyhedra: Two Representations](#polyhedra-two-representations)
3. [Minkowski-Weyl Theorem](#minkowski-weyl-theorem)
4. [Binary Linear Programs (BLP)](#binary-linear-programs-blp)
5. [Characteristic Vectors](#characteristic-vectors)
6. [The Connection: COP ↔ BLP](#the-connection-cop--blp)
7. [Example: MST as ILP](#example-mst-as-ilp)
8. [Exercise 8.1: More ILP Formulations](#exercise-81-more-ilp-formulations--important)
9. [Exercise 8.4: Logic as Linear Constraints](#exercise-84-logic-as-linear-constraints--important)
10. [Exercise 7.3: Graph Coloring ILP](#exercise-73-graph-coloring-ilp)

---

## Combinatorial Optimization Problems

> ⚠️ **Exam question**: "What are combinatorial problems?"
> **Answer**: We search for a combination (subset) of elements that maximizes or minimizes a goal function. The goal function has the form $c(I) = \sum_{e \in I} c_e$ — a sum of partial costs.

---

### Formal Definition

A **Combinatorial Optimization Problem (COP)** consists of three parts:

**Part 1: Ground Set $E$** (finite)

- This is the set of ALL possible "items" you could choose from.
- Example: In a graph problem, $E$ might be all edges.

**Part 2: Feasible Sets $\mathcal{I}$** (subset of the power set $2^E$)

- These are the "allowed" combinations.
- Not every subset of $E$ is feasible — only those in $\mathcal{I}$.
- Example: For spanning trees, $\mathcal{I}$ = all edge sets that form spanning trees.

**Part 3: Cost Function $c: E \to \mathbb{R}$**

- Each element $e \in E$ has a cost $c(e)$.
- For a set $I \subseteq E$: total cost is $c(I) = \sum_{e \in I} c(e)$.

> ⚠️ **Exam question**: "This goal function has a certain form, how does it look?"
> **Answer**: It is the **sum of partial costs**: $c(I) = \sum_{e \in I} c_e$. Not some weird formula, just a simple sum.

---

**Goal:** Find $I^* \in \mathcal{I}$ such that $c(I^*)$ is maximum (or minimum).

---

### Examples — What IS and ISN'T a COP

| Problem                                                     | COP? | Why?                                                     |
| ----------------------------------------------------------- | ---- | -------------------------------------------------------- |
| Travelling Salesman (TSP)                                   | ✓    | $E$ = edges, $\mathcal{I}$ = Hamiltonian cycles          |
| Minimum Spanning Tree (MST)                                 | ✓    | $E$ = edges, $\mathcal{I}$ = spanning trees              |
| 0/1-Knapsack                                                | ✓    | $E$ = items, $\mathcal{I}$ = subsets fitting in knapsack |
| Knapsack (unlimited copies)                                 | ✗    | Can take unlimited copies → not a finite subset choice   |
| $\min 3x^2 + 2, x \in \mathbb{R}$                           | ✗    | Continuous variable, not a finite set choice             |
| LP: $\max 2x_1 + 3x_2$ s.t. $Ax \leq b$, $x \in \mathbb{N}$ | ✗    | NP, but infinite choices (not finite $E$)                |

---

### Recognizing COPs: The Three Questions

When asked to identify a COP, answer these:

1. **What is the ground set $E$?** (the elements you choose from)
2. **What is the feasible set $\mathcal{I}$?** (which combinations are allowed)
3. **What is the cost function?** (always a sum: $\sum_{e \in I} c_e$)

---

## Polyhedra: Two Representations

Before connecting COPs to LPs, you need to understand polyhedra.

---

### Definition: Polyhedron

A **polyhedron** $P \subseteq \mathbb{R}^n$ is the set of all points satisfying a system of linear inequalities:

$$P = \{x \in \mathbb{R}^n \mid Ax \leq b\}$$

**where:**

- $A$ is an $m \times n$ matrix (each row is one constraint)
- $b$ is an $m$-dimensional vector (right-hand sides)
- Each constraint $a_i^T x \leq b_i$ defines a **half-space**

**Observation:** $P$ is the intersection of $m$ half-spaces.

---

### Definition: Polytope

A **polytope** is a **bounded** polyhedron:

$$P \subseteq \{x \in \mathbb{R}^n \mid \|x\|_\infty \leq B\}$$

for some constant $B > 0$.

**Intuition:** A polytope is "finite" — it doesn't extend to infinity in any direction.

---

## Minkowski-Weyl Theorem

> ⚠️ **Exam question**: "What does the Theorem of Minkowski-Weyl say?"
> **Answer**: Every polyhedron $P$ can be represented in the form $P = \text{conv}(V) + \text{cone}(E)$, where $V$ and $E$ are finite subsets of $\mathbb{R}^n$. A polyhedron is the intersection of finitely many half-spaces.

---

### Intuition for conv and cone (don't memorize formulas)

**Convex hull (conv):** The "smallest bubble" containing all points. Any point inside is a weighted average of the vertices, where weights are non-negative and sum to 1.

**Conic hull (cone):** All points you can reach by scaling and adding the given vectors with non-negative coefficients.

> **For the exam:** You don't need to write $\sum \lambda_i x_i$ with conditions. Just say:
>
> - "conv = weighted average of vertices"
> - "cone = non-negative combinations"

---

### The Theorem

**Theorem (Minkowski 1896, Weyl 1935):**

Every polyhedron $P \subseteq \mathbb{R}^n$ can be written as:

$$P = \text{conv}(V) + \text{cone}(E)$$

where $V$ and $E$ are **finite** subsets of $\mathbb{R}^n$.

**Conversely:** Every set of the form $\text{conv}(V) + \text{cone}(E)$ is a polyhedron.

---

### What does this mean practically?

**Two equivalent representations of any polyhedron:**

| Representation       | Description                                            |
| -------------------- | ------------------------------------------------------ |
| **H-representation** | Inequalities: $P = \{x \mid Ax \leq b\}$               |
| **V-representation** | Vertices + Rays: $P = \text{conv}(V) + \text{cone}(E)$ |

**Special case (Polytopes):** If $P$ is bounded, then $E = \emptyset$, so:
$$P = \text{conv}(V)$$

The polytope is just the convex hull of its vertices!

---

### Visual Example

```
           V₃ = (0,3)
              ●
             ╱ ╲
            ╱   ╲
           ╱     ╲
V₁ = (0,1)●───────● V₂ = (1,1)

Polytope P (a triangle)

V = {(0,1), (1,1), (0,3)}
P = conv(V)

Inequality form:
P = {(x,y) | x ≥ 0, y ≥ 1, 2x + y ≤ 3}
```

Both representations describe the SAME triangle!

---

## Binary Linear Programs (BLP)

> ⚠️ **Exam question**: "Write down a binary integer programme."

---

### Definition

A **Binary Linear Program (BLP)** is:

$$\max c^T x \quad \text{s.t.} \quad Ax \leq b, \quad x \in \{0,1\}^n$$

**where:**

- $x \in \{0,1\}^n$ means every variable is either 0 or 1
- $c \in \mathbb{R}^n$ are the costs
- $A$ is an $m \times n$ constraint matrix
- $b \in \mathbb{R}^m$ are the right-hand sides

---

### Why Binary?

Each $x_i \in \{0,1\}$ represents a **yes/no decision**:

- $x_i = 1$ → "include element $i$"
- $x_i = 0$ → "exclude element $i$"

This is PERFECT for combinatorial problems where we're choosing subsets!

---

## Characteristic Vectors

This is the key to connecting COPs and BLPs.

---

### Definition

For a finite set $E$ and a subset $F \subseteq E$, the **characteristic vector** $\chi^F \in \{0,1\}^E$ is defined as:

$$\chi^F_e = \begin{cases} 1 & \text{if } e \in F \\ 0 & \text{if } e \notin F \end{cases}$$

---

### Visual Example

Let $E = \{a, b, c, d, e\}$ be our ground set (e.g., 5 edges).

If $F = \{a, c, e\}$ is our chosen subset:

```
Element:    a   b   c   d   e
            ↓   ↓   ↓   ↓   ↓
χ^F:      ( 1   0   1   0   1 )
```

The 1s mark which elements are "in" the subset.

---

### Key Properties

1. **Every subset $F \subseteq E$** has exactly one characteristic vector $\chi^F$.

2. **Every 0/1-vector $x \in \{0,1\}^E$** corresponds to exactly one subset:
   $$F_x = \{e \in E \mid x_e = 1\}$$

3. **The cost of a subset** equals the dot product:
   $$c(F) = \sum_{e \in F} c_e = c^T \chi^F$$

This is WHY the cost function is "a sum of partial costs" — it's just a dot product!

---

## The Connection: COP ↔ BLP

Now the beautiful theorem:

---

### Theorem: COP ↔ BLP Equivalence

**Any combinatorial optimization problem can be formulated as a Binary Linear Program (BLP), and vice versa.**

---

### Why does COP ↔ BLP work? (Intuition only)

**BLP → COP:** Each 0/1 variable decides "include or exclude" an element. The constraints define which subsets are feasible.

**COP → BLP:** Use characteristic vectors. Each feasible set becomes a 0/1 vector. The polytope of all such vectors can (theoretically) be described by linear inequalities.

**The catch:** This LP description may have exponentially many constraints and cannot always be computed in polynomial time.

> **For the exam:** You won't be asked to prove this. Just know: COP ↔ BLP equivalence EXISTS, but may not be practical.

---

## Example: MST as ILP

> ⚠️ **This example pattern appears in exams: formulating a graph problem as ILP.**

---

### Problem: Minimum Spanning Tree

**Given:** Complete graph $K_3$ with 3 vertices and 3 edges.

```
        1
       / \
     e₁   e₂
     /     \
    2───────3
        e₃
```

**Find:** Spanning tree with minimum total edge weight.

---

### Step 1: Define Variables

For each edge $e$, define:
$$x_e = \begin{cases} 1 & \text{if edge } e \text{ is in the tree} \\ 0 & \text{if not} \end{cases}$$

---

### Step 2: Objective Function

Minimize total cost:
$$\min \sum_{e \in E} c_e x_e = c_1 x_1 + c_2 x_2 + c_3 x_3$$

---

### Step 3: Constraints

**Constraint 1: Exactly $|V| - 1 = 2$ edges** (spanning tree definition)
$$x_1 + x_2 + x_3 = 2$$

**Constraint 2: Non-negativity**
$$x_e \geq 0 \quad \text{for } e = 1, 2, 3$$

**Constraint 3: At most 1**
$$x_e \leq 1 \quad \text{for } e = 1, 2, 3$$

---

### Complete ILP for MST on $K_3$:

```
min  c₁x₁ + c₂x₂ + c₃x₃

s.t. x₁ + x₂ + x₃ = 2
     xₑ ≥ 0       for e = 1, 2, 3
     xₑ ≤ 1       for e = 1, 2, 3
     xₑ ∈ {0, 1}  for e = 1, 2, 3
```

---

### Feasible Characteristic Vectors

The spanning trees of $K_3$ are:

- Edges $\{e_1, e_2\}$ → $\chi = (1, 1, 0)$
- Edges $\{e_1, e_3\}$ → $\chi = (1, 0, 1)$
- Edges $\{e_2, e_3\}$ → $\chi = (0, 1, 1)$

These are the ONLY integer solutions to our ILP!

---

### General MST ILP (larger graphs)

For a graph $G = (V, E)$:

**Formulation 1 (Cut Constraints):**

$$\min \sum_{e \in E} c_e x_e$$

subject to:

- $\sum_{e \in E} x_e = |V| - 1$ (exactly $n-1$ edges)
- $\sum_{e \in \delta(W)} x_e \geq 1$ for all $\emptyset \neq W \subset V$ (connectivity: every cut has at least one edge)
- $x_e \in \{0, 1\}$

**where** $\delta(W)$ = edges with exactly one endpoint in $W$ (the "cut").

---

**Formulation 2 (Subtour Elimination):**

$$\min \sum_{e \in E} c_e x_e$$

subject to:

- $\sum_{e \in E} x_e = |V| - 1$
- $\sum_{e \in E(W)} x_e \leq |W| - 1$ for all $W \subseteq V$ (no subtours: can't use all edges of any subset)
- $x_e \in \{0, 1\}$

**where** $E(W)$ = edges with BOTH endpoints in $W$.

---

### Exercise 8.2: MST with Positive Weights — **Important**

> **Question:** Can we simplify the MST ILP if all weights are strictly positive ($w(e) > 0$)?

**Answer:** YES! If all weights are positive, the minimization objective naturally avoids extra edges.

We can **remove** the constraint $\sum_{e \in E} x_e = |V| - 1$ (exactly $n-1$ edges).

**Why?** With positive costs, adding extra edges only increases cost. The solver will naturally choose exactly $n-1$ edges to minimize cost while maintaining connectivity.

**Do we lose generality?** NO — any MST instance with zero or negative weights can be transformed by adding a constant to all weights.

---

## Exercise 8.1: More ILP Formulations — **Important**

These pattern formulations appear frequently in exams.

---

### (a) Travelling Salesperson (TSP)

**Given:** Complete undirected graph $G = (V, E)$, distance function $d: E \to \mathbb{R}^+_0$.

**Goal:** Find a Hamiltonian cycle (visits each vertex exactly once) with minimum total distance.

---

**Variables:**

For each edge $e \in E$:
$$x_e = \begin{cases} 1 & \text{if edge } e \text{ is in the tour} \\ 0 & \text{otherwise} \end{cases}$$

---

**Objective:**
$$\min \sum_{e \in E} d_e x_e$$

---

**Constraints:**

1. **Each vertex has exactly degree 2** (enter once, leave once):
   $$\sum_{e \in \delta(v)} x_e = 2 \quad \forall v \in V$$

   where $\delta(v)$ = edges incident to $v$.

2. **No subtours** (must visit ALL vertices, not split into smaller cycles):
   $$\sum_{e \in E(W)} x_e \leq |W| - 1 \quad \forall W \subset V, 2 \leq |W| \leq |V| - 1$$

3. **Binary:**
   $$x_e \in \{0, 1\}$$

---

**Complete TSP ILP:**

```
min  Σ d_e x_e

s.t. Σ_{e ∈ δ(v)} x_e = 2       for all v ∈ V
     Σ_{e ∈ E(W)} x_e ≤ |W| - 1  for all W ⊂ V, 2 ≤ |W| ≤ n-1
     x_e ∈ {0, 1}
```

> **Exam trap:** The subtour elimination constraints are exponentially many. In practice, add them lazily (cutting plane method).

---

### (b) Vertex Cover

**Given:** Undirected graph $G = (V, E)$.

**Goal:** Find minimum subset $V' \subseteq V$ such that every edge has at least one endpoint in $V'$.

---

**Variables:**

For each vertex $v \in V$:
$$x_v = \begin{cases} 1 & \text{if } v \in V' \\ 0 & \text{otherwise} \end{cases}$$

---

**Objective:**
$$\min \sum_{v \in V} x_v$$

---

**Constraints:**

For each edge $\{u, v\} \in E$, at least one endpoint must be covered:
$$x_u + x_v \geq 1 \quad \forall \{u, v\} \in E$$

**Binary:**
$$x_v \in \{0, 1\}$$

---

**Complete Vertex Cover ILP:**

```
min  Σ x_v

s.t. x_u + x_v ≥ 1    for all {u,v} ∈ E
     x_v ∈ {0, 1}     for all v ∈ V
```

---

### (c) Bin Packing

**Given:** Items $1, \ldots, n$ with sizes $w_i \leq K$, bins of capacity $K$.

**Goal:** Minimize number of bins to fit all items.

---

**Variables:**

For each item $i$ and potential bin $j$:
$$x_{ij} = \begin{cases} 1 & \text{if item } i \text{ is in bin } j \\ 0 & \text{otherwise} \end{cases}$$

For each potential bin $j$:
$$y_j = \begin{cases} 1 & \text{if bin } j \text{ is used} \\ 0 & \text{otherwise} \end{cases}$$

---

**Objective:**
$$\min \sum_{j=1}^{n} y_j$$

(Minimize number of bins used)

---

**Constraints:**

1. **Each item in exactly one bin:**
   $$\sum_{j=1}^{n} x_{ij} = 1 \quad \forall i$$

2. **Bin capacity not exceeded:**
   $$\sum_{i=1}^{n} w_i x_{ij} \leq K \quad \forall j$$

3. **Bin is used if any item in it:**
   $$x_{ij} \leq y_j \quad \forall i, j$$

4. **Binary:**
   $$x_{ij}, y_j \in \{0, 1\}$$

---

## Exercise 8.4: Logic as Linear Constraints — **Important**

This shows how to encode logical operations in ILPs. Useful for complex constraints.

---

### (a) AND: $z = x \land y$

$z = 1$ if and only if BOTH $x = 1$ AND $y = 1$.

**Constraints:**
$$z \leq x$$
$$z \leq y$$
$$z \geq x + y - 1$$

**Why?**

- First two: $z$ can only be 1 if both $x$ and $y$ are 1.
- Third: $z$ must be 1 when both $x$ and $y$ are 1.

---

### (b) OR: $z = x \lor y$

$z = 1$ if EITHER $x = 1$ OR $y = 1$ (or both).

**Constraints:**
$$z \geq x$$
$$z \geq y$$
$$z \leq x + y$$

---

### (c) XOR: $z = x \oplus y$

$z = 1$ if EXACTLY ONE of $x, y$ is 1.

**Constraints:**
$$z \leq x + y$$
$$z \geq x - y$$
$$z \geq y - x$$
$$z \leq 2 - x - y$$

Or equivalently:
$$z = x + y - 2xy$$

But $xy$ is nonlinear! Linearize by introducing auxiliary variable $w = xy$:
$$w \leq x, \quad w \leq y, \quad w \geq x + y - 1$$
$$z = x + y - 2w$$

---

## Exercise 7.3: Graph Coloring ILP — **Important**

> ⚠️ This exercise tests ILP formulation skills — exactly what the exam asks.

---

### Problem Statement

**Given:** Undirected graph $G = (V, E)$.

**Goal:** Find the minimum $k$ such that $G$ has a valid $k$-coloring.

A **valid $k$-coloring** is a function $f: V \to \{1, \ldots, k\}$ such that:
$$f(v) \neq f(w) \quad \text{for all } \{v, w\} \in E$$

Adjacent vertices must have different colors.

---

### Part (a): First ILP Formulation (Assignment Model)

**Variables:**

For each vertex $v$ and each color $c \in \{1, \ldots, k\}$:
$$x_{v,c} = \begin{cases} 1 & \text{if vertex } v \text{ gets color } c \\ 0 & \text{otherwise} \end{cases}$$

---

**Objective:** Minimize the number of colors used.

Define auxiliary variable $y_c$:
$$y_c = \begin{cases} 1 & \text{if color } c \text{ is used by any vertex} \\ 0 & \text{otherwise} \end{cases}$$

$$\min \sum_{c=1}^{k} y_c$$

---

**Constraints:**

1. **Each vertex gets exactly one color:**
   $$\sum_{c=1}^{k} x_{v,c} = 1 \quad \forall v \in V$$

2. **Adjacent vertices have different colors:**
   $$x_{v,c} + x_{w,c} \leq 1 \quad \forall \{v, w\} \in E, \forall c \in \{1, \ldots, k\}$$

3. **Color is used if assigned:**
   $$x_{v,c} \leq y_c \quad \forall v \in V, \forall c$$

4. **Binary:**
   $$x_{v,c}, y_c \in \{0, 1\}$$

---

**Complete ILP (Formulation 1):**

```
min  Σ yc

s.t. Σc x_{v,c} = 1                for all v ∈ V
     x_{v,c} + x_{w,c} ≤ 1         for all {v,w} ∈ E, all c
     x_{v,c} ≤ yc                  for all v ∈ V, all c
     x_{v,c}, yc ∈ {0, 1}
```

---

### Part (b): Second ILP Formulation (Direct Comparison)

**Variables:**

For each ordered pair $(v, w) \in E$:
$$x_{vw} = \begin{cases} 1 & \text{if } f(v) < f(w) \\ 0 & \text{if } f(v) > f(w) \end{cases}$$

And for counting colors, let $z_v = f(v)$ be the color assigned to vertex $v$.

---

**Constraints:**

1. **Different colors for adjacent vertices:**
   Either $f(v) < f(w)$ or $f(w) < f(v)$ — they can't be equal:
   $$z_v - z_w + n \cdot x_{vw} \geq 1 \quad \forall \{v, w\} \in E$$
   $$z_w - z_v + n \cdot (1 - x_{vw}) \geq 1 \quad \forall \{v, w\} \in E$$

2. **Minimize maximum color:**
   $$\min k$$
   where $z_v \leq k$ for all $v$.

---

### Part (c): Complexity Comparison

| Formulation        | # Variables                     | # Constraints    |
| ------------------ | ------------------------------- | ---------------- |
| **1 (Assignment)** | $O(nk)$ for $x$, $O(k)$ for $y$ | $O(n + mk + nk)$ |
| **2 (Direct)**     | $O(m + n)$                      | $O(m)$           |

**Formulation 1:** Number of variables grows with $k$ (which is unknown).

**Formulation 2:** Fewer variables, but uses big-M constraints (numerically unstable).

---

### Part (d): Practical Considerations

**Problems with Formulation 1:**

- Need to guess $k$ upfront or use a large upper bound
- Many variables if $k$ is large
- Symmetry: permuting colors gives equivalent solutions (slows solvers)

**Problems with Formulation 2:**

- Big-M constraints are numerically unstable
- Weak LP relaxation (fractional solutions far from integer)

**In practice:** Formulation 1 with symmetry-breaking constraints is usually preferred.

---

## Complete LP Characterizations

Sometimes we know the EXACT polytope of a problem:

| Problem                     | Complete LP known? |
| --------------------------- | ------------------ |
| MST                         | ✓ Yes              |
| Largest Forest              | ✓ Yes              |
| Min-Weight Perfect Matching | ✓ Yes              |
| TSP                         | ✗ No (NP-hard)     |
| Graph Coloring              | ✗ No (NP-hard)     |

When complete LP is known → can solve in **polynomial time** using LP.

When NOT known → use incomplete LP + Branch-and-Bound/Cut.

---

## Exam Triage

**Exam-critical (go deep):**

- [Definition of Combinatorial Optimization Problem](#combinatorial-optimization-problems) (ground set, feasible sets, cost function)
- [Minkowski-Weyl theorem statement](#minkowski-weyl-theorem)
- [Writing a BLP for a graph problem](#example-mst-as-ilp) (MST, TSP, Vertex Cover, coloring)
- [Characteristic vectors](#characteristic-vectors) and why cost is $\sum c_e$

**Secondary:**

- Detailed convex combination definitions
- LP relaxation algorithms

**Stop wasting time on:**

- Proving Minkowski-Weyl
- Implementing ILP solvers

---

## Quick Reference — Write From Memory

```
Combinatorial Optimization Problem:
    E = ground set (finite)
    I = feasible sets ⊆ 2^E
    c: E → R (cost function)
    Goal: find I* ∈ I maximizing Σ_{e∈I*} c(e)

Minkowski-Weyl:
    P = conv(V) + cone(E)  ↔  P = {x | Ax ≤ b}

Characteristic Vector χ^F:
    χ^F_e = 1 if e ∈ F, else 0
    c(F) = c^T χ^F

BLP form:
    max c^T x
    s.t. Ax ≤ b
         x ∈ {0,1}^n
```

---
