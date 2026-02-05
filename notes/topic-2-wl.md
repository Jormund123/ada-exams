# Graph Similarity

## Table of Contents

1. [Weisfeiler-Leman Algorithm](#weisfeiler-leman-algorithm)
2. [The Naive WL Algorithm — KNOW THIS COLD](#the-naive-wl-algorithm--know-this-cold)
3. [WL as Isomorphism Test — One-Sided](#wl-as-isomorphism-test--one-sided)
4. [Equitable Partition — What WL Computes](#equitable-partition--what-wl-computes)
5. [When WL Succeeds and Fails](#when-wl-succeeds-and-fails)
6. [Practice Exercises (5.2, 5.3, 5.4)](#exercise-52-wl-example--important)
7. [Efficient WL Algorithm — O(n² log n)](#efficient-wl-algorithm--on-log-n)
8. [k-Dimensional WL (k-WL)](#k-dimensional-wl-k-wl)
9. [k-WL Can Distinguish Regular Graphs](#k-wl-can-distinguish-regular-graphs)
10. [Local k-WL (k-LWL) — EXAM RELEVANT](#local-k-wl-k-lwl--exam-relevant)
11. [Runtime Summary](#runtime-summary)
12. [Exam Triage](#exam-triage)
13. [Oral Defense Summary](#oral-defense-summary--if-interrupted)
14. [Quick Reference — Write From Memory](#quick-reference--write-from-memory)

---

## Weisfeiler-Leman Algorithm

> ⚠️ **Exam question**: "What is the Weisfeiler-Leman algorithm used for?"

WL is a **color refinement algorithm** that tries to distinguish non-isomorphic graphs. If two graphs get different color histograms, they are NOT isomorphic.

> ⚠️ **Exam question**: "What is graph isomorphism?"
>
> **Answer**: Two graphs G and H are isomorphic (G ≃ H) if there exists a bijection π: V(G) → V(H) that preserves edges: (u,v) ∈ E(G) ⟺ (π(u), π(v)) ∈ E(H).

---

## The Naive WL Algorithm — KNOW THIS COLD

> ⚠️ **Exam question**: "Describe the Weisfeiler-Leman algorithm."

```
WL-Algorithm:
    Initial: All vertices get the SAME color (e.g., color 1)

    Repeat until stable:
        For each vertex v:
            new_color[v] = hash(old_color[v], sorted list of neighbor colors)
        old_color = new_color
```

**In words**: Each vertex looks at its own color and the colors of its neighbors, then gets a new color based on this combination.

> ⚠️ **This matters in the exam**: You MUST be able to write this pseudocode from memory. If you can't trace WL on a 4-vertex graph, you will fail this topic.

> ⚠️ **Exam question**: "What is the runtime of the naive WL and where does it come from?"
>
> **Answer**: O(n³). Up to n iterations (because we create at most n color classes), and each iteration takes O(n²) to compute all vertex hashes (n vertices × n neighbors to check).

---

### Trace Example — Draw This on Whiteboard

```
Graph G:
    a ─── b
    │     │
    c ─── d
```

**Initial (all same color = 1):**

```
a:1  b:1
c:1  d:1
```

**Iteration 1**: Each vertex has 2 neighbors, all color 1

- a: hash(1, [1,1]) = 2
- b: hash(1, [1,1]) = 2
- c: hash(1, [1,1]) = 2
- d: hash(1, [1,1]) = 2

All vertices get same color → **STABLE** (no refinement possible)

This is a 2-regular graph. WL cannot distinguish regular graphs!

> 🎤 **If examiner asks**: "Why did WL fail here?"
>
> **Your answer**: "All vertices have the same degree. In the initial round, everyone has the same color. When we hash (color, neighbor_colors), everyone computes the same hash because their neighbor multisets are identical. The coloring never refines."

---

### Trace Example 2 — Path Graph

```
Graph: 1 ─── 2 ─── 3 ─── 4
```

**Initial:**

```
1:A  2:A  3:A  4:A
```

**Iteration 1**: Count neighbor colors

- Vertex 1: 1 neighbor → hash(A, [A]) = B
- Vertex 2: 2 neighbors → hash(A, [A,A]) = C
- Vertex 3: 2 neighbors → hash(A, [A,A]) = C
- Vertex 4: 1 neighbor → hash(A, [A]) = B

```
1:B  2:C  3:C  4:B
```

**Iteration 2**:

- Vertex 1: neighbor has color C → hash(B, [C]) = D
- Vertex 2: neighbors B, C → hash(C, [B,C]) = E
- Vertex 3: neighbors C, B → hash(C, [C,B]) = E
- Vertex 4: neighbor has color C → hash(B, [C]) = D

```
1:D  2:E  3:E  4:D
```

**Iteration 3**:

- Vertex 1: hash(D, [E]) = F
- Vertex 2: hash(E, [D,E]) = G
- Vertex 3: hash(E, [E,D]) = G
- Vertex 4: hash(D, [E]) = F

```
1:F  2:G  3:G  4:F
```

**STABLE** — 2 color classes: endpoints {1,4} and middle {2,3}

---

**STOP. Explain back to me:**

Why did vertices 2 and 3 get the same final color, but 1 and 4 also got the same color (different from 2,3)?

**Answer**: 1 and 4 are symmetric (both endpoints). 2 and 3 are symmetric (both middle). WL captures this structural equivalence.

---

## WL as Isomorphism Test — One-Sided

> ⚠️ **Exam question**: "What do you mean WL can detect isomorphisms between some graphs but not others?"

**How to test if G ≃ H:**

1. Run WL on G and H simultaneously
2. Compare final color histograms

**If histograms differ** → G ≄ H (definitely NOT isomorphic)

**If histograms match** → **We don't know!** They might be isomorphic, or WL might be too weak.

```
G ≄ H  IF  WL colors differ
G ≃ H  ???  IF  WL colors match (inconclusive)
```

> 🎤 **If examiner interrupts**: "So WL can prove graphs are isomorphic?"
>
> **Your answer**: "No! WL is one-sided. It can prove non-isomorphism (different histograms = definitely different graphs). But matching histograms does NOT prove isomorphism — WL might just be too weak to distinguish them."

**What does "too weak" mean?**

"Too weak" means WL cannot see structural differences that actually exist. For example:

- Two non-isomorphic 3-regular graphs might both have all vertices with degree 3
- WL gives them the same color histogram (everyone gets the same color)
- WL concludes "maybe isomorphic" — but they're actually different graphs!

WL only looks at **local neighborhood structure** (degree, neighbor colors). It misses **global structure** (cycles, connectivity patterns). That's why k-WL is stronger — it looks at tuples, capturing more structure.

**What is a histogram?**

The histogram is NOT the vertex→color mapping. It's a **count of how many vertices have each color**.

Example from the path graph above:

```
Final coloring:  1:F  2:G  3:G  4:F

Histogram:
  Color F: 2 vertices
  Color G: 2 vertices

Written as: Φ(G) = (2, 2)  or  {F:2, G:2}
```

Two graphs have the **same histogram** if they have the same count for each color (even if vertex indices differ).

---

## Equitable Partition — What WL Computes

**Definition**: A partition C = {C₁, ..., Cₗ} is **equitable** if:

$$\forall i,j \in \{1,...,\ell\}, \forall u,v \in C_i : |N(u) \cap C_j| = |N(v) \cap C_j|$$

**where:**

- $C = \{C_1, ..., C_\ell\}$ = the cells (groups) of the partition
- $N(u)$ = neighbors of vertex $u$
- $N(u) \cap C_j$ = neighbors of $u$ that are in cell $C_j$
- $|N(u) \cap C_j|$ = count of how many neighbors $u$ has in cell $C_j$

**In words**: Any two vertices in the same cell have the same number of neighbors in every cell.

**Theorem**: WL computes the **coarsest equitable partition**.

> 🎤 **If examiner asks**: "What does WL actually compute?"
>
> **Your answer**: "WL computes the coarsest equitable partition. An equitable partition is one where any two vertices in the same cell have the same number of neighbors in every cell. 'Coarsest' means we use the fewest cells possible while still being equitable."

---

### Exercise 5.1: Equitable ↔ Regular + Biregular

**Claim**: Partition C is equitable ⟺ both conditions hold:

- **(a)** Each induced subgraph G[Cᵢ] is **regular**
- **(b)** Each induced bipartite graph between Cᵢ and Cⱼ is **biregular**

---

**Direction (⇒): Equitable → Regular & Biregular**

**(a) Within a cell (i = j):**

Set target $R = C_i$. For any $u, v \in C_i$:

$$|N(u) \cap C_i| = |N(v) \cap C_i| \quad \text{(by equitable definition)}$$

This is the degree within $G[C_i]$ → same for all vertices → **Regular** ✓

**(b) Between cells (i ≠ j):**

For all $u \in C_i$: $|N(u) \cap C_j| = d_{i \to j}$ (constant)

For all $x \in C_j$: $|N(x) \cap C_i| = d_{j \to i}$ (constant)

→ All $C_i$ vertices have same degree into $C_j$, and vice versa → **Biregular** ✓

---

**Direction (⇐): Regular & Biregular → Equitable**

Need to show: ∀u,v ∈ Cᵢ : |N(u) ∩ Cⱼ| = |N(v) ∩ Cⱼ|

**Case i = j:**

By (a), $G[C_i]$ is regular → all vertices have same internal degree

$$\Rightarrow |N(u) \cap C_i| = |N(v) \cap C_i| \quad ✓$$

**Case i ≠ j:**

By (b), bipartite graph is biregular → all $C_i$ vertices have same degree into $C_j$

$$\Rightarrow |N(u) \cap C_j| = |N(v) \cap C_j| \quad ✓$$

∴ Partition is equitable. ∎

---

## When WL Succeeds and Fails

| Graph Type                    | WL Result                                   |
| ----------------------------- | ------------------------------------------- |
| **Trees/Forests**             | ✓ Always distinguishes non-isomorphic trees |
| **Random graphs**             | ✓ Works with high probability               |
| **Regular graphs**            | ✗ FAILS — all vertices get same color       |
| **Cai-Fürer-Immerman graphs** | ✗ Constructed specifically to fool WL       |

**Key failure**: If all vertices have the same degree, initial colors match, and if neighbor structure is symmetric, they stay matched.

---

### Exercise 6.2: Individualization Refinement — **Important**

**Technique**: When WL fails (all same color), "individualize" one vertex by giving it a unique color, then rerun WL.

**How it works**:

1. Run WL until stable → get color classes C₁, ..., Cₖ
2. Pick a vertex v ∈ G in some color class Cᵢ
3. Give v a NEW unique color (separate it from Cᵢ)
4. Simultaneously, pick a vertex w ∈ H from the "same" color class
5. Give w the same new unique color
6. Rerun WL from this new initial coloring

**Why it helps**: By "breaking symmetry" at one vertex, WL can now distinguish previously indistinguishable vertices based on their relationship to the individualized vertex.

---

**(a) Distinguish 6-cycle from two triangles**

```
G: 6-cycle              H: two triangles
  1───2                   a───b
  │   │                    ╲ ╱
  6   3                     c
  │   │
  5───4                   d───e
                           ╲ ╱
                            f
```

**Without individualization**: WL gives all vertices same color (all 2-regular).

**With individualization**:

1. Individualize vertex 1 in G and vertex a in H → give them unique color X
2. Rerun WL:

In G (6-cycle):

- Vertex 2 is adjacent to X → new color Y₁
- Vertex 6 is adjacent to X → new color Y₁
- Vertices 3,5 are distance 2 from X → new color Y₂
- Vertex 4 is distance 3 from X (opposite) → new color Y₃

In H (two triangles):

- Vertices b,c are adjacent to X → new color Y₁
- Vertices d,e,f are DISCONNECTED from X → color stays Z (unreachable)

**Different histograms after 1 iteration** → G ≄ H detected!

---

**(b) Example where one iteration doesn't suffice**

Consider two non-isomorphic 3-regular graphs that are "locally identical" even after one individualization:

- After individualizing, vertices might still be symmetric
- Need multiple rounds of individualization to break all symmetry

(This is the basis of practical graph isomorphism algorithms like nauty)

---

### Exercise 5.2: WL Example — **Important**

> Apply WL on this graph and give color histogram and queue at each step.

```
        1
       ╱ ╲
      3───2
       ╲ ╱
        4
       ╱ ╲
      5───6
       ╲ ╱
        7
       ╱ ╲
      9───8
       ╲ ╱
       10
       ╱ ╲
     11───12
```

**Initial**: All vertices get color A. Q = {A}

**Iteration 1**: Count neighbors in A (everyone). Degree matters:

- Vertices 1, 4, 7, 10: degree 2 → color B
- Vertices 2,3,5,6,8,9,11,12: degree 3 → color C

Split: C₁ = {1,4,7,10}, C₂ = {2,3,5,6,8,9,11,12}
Q = {B} (smaller class)

**Iteration 2**: Refine by neighbors in B = {1,4,7,10}:

- Vertices 2,3: 1 neighbor in B (just vertex 1)
- Vertices 5,6: 2 neighbors in B (vertices 4 and 4... wait, only 4)

Actually by symmetry, vertices 2,3 are adjacent to 1 only, 5,6 to 4 only, etc.
Continue refining...

**Final stable partition**: WL distinguishes vertices by their position in the "ladder" structure.

---

### Exercise 5.3: WL Iterations on Paths — **Important**

> How many iterations does WL need on a path with n vertices?

**Answer**: ⌈n/2⌉ iterations (ceiling of n/2)

**Proof**:

```
Path: v₁ ─── v₂ ─── v₃ ─── ... ─── vₙ
```

- After iteration 1: endpoints (degree 1) get one color, middle (degree 2) get another
- After iteration 2: v₂ and vₙ₋₁ are distinguished (they're adjacent to endpoints)
- Each iteration, the "distinguished zone" expands by 1 from each end
- Meeting in the middle takes ⌈n/2⌉ iterations

**Example (n=6)**:

```
Initial:    A─A─A─A─A─A
Iter 1:     B─C─C─C─C─B    (endpoints vs middle)
Iter 2:     B─D─C─C─D─B    (next to endpoint gets new color)
Iter 3:     B─D─E─E─D─B    (stable by symmetry)
```

3 = ⌈6/2⌉ ✓

---

### Exercise 5.4: WL on Isomorphic Graphs — **Important**

> Prove: If G ≃ H (isomorphic), then WL gives the same color histogram for both.

**Proof**:

Let π: V(G) → V(H) be an isomorphism.

**Claim**: At every iteration, if v has color c in G, then π(v) has color c in H.

**Base case** (iteration 0): All vertices get color 1 in both graphs. ✓

**Inductive step**: Assume true at iteration i. At iteration i+1:

Color of v = hash(colorᵢ(v), multiset of neighbor colors)

Since π preserves edges:

- u is a neighbor of v in G ⟺ π(u) is a neighbor of π(v) in H
- By induction, colorᵢ(u) = colorᵢ(π(u))

So the neighbor color multisets are identical → v and π(v) get the same new color. ✓

**Conclusion**: Same coloring → same histogram. ∎

---

## Efficient WL Algorithm — O(n² log n)

> ⚠️ **Exam question**: "What is the runtime of the efficient WL?"
>
> **Answer**: O(n² log n) using Hopcroft's trick. There's also an O((n+m) log n) implementation.

> ⚠️ **Exam question**: "Describe how efficient WL is different from naive. Draw a small graph and apply it."

The naive WL is O(n³) — up to n iterations, each O(n²).

**Hopcroft's "process the smaller half" trick** gives O(n² log n):

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

**Why O(n² log n)?**

- Each vertex appears at most O(log n) times in Q
- When a class is split, smaller halves go to Q → size halves each time
- Per vertex: O(n) work per appearance × O(log n) appearances = O(n log n)
- Total: n vertices × O(n log n) = O(n² log n)

> ⚠️ **This step matters in the exam**: The log n factor comes from "each vertex appears at most log n times in Q" — because each time a vertex goes into Q, its color class has at most half the size. If examiner asks "why log n?", this is what you say.

**There's also an O((V+E) log V) implementation.**

---

### Traced Example: Efficient WL on Path Graph

```
Graph: a ─── b ─── c ─── d

Initial: All vertices have color 1
         Color class: C₁ = {a, b, c, d}
         Q = {1}  (start with the only color class)
```

**Step 1**: Pop color 1 from Q, refine all classes by counting neighbors in C₁

```
Count neighbors in C₁ for each vertex:
  a: 1 neighbor in C₁ (just b)
  b: 2 neighbors in C₁ (a and c)
  c: 2 neighbors in C₁ (b and d)
  d: 1 neighbor in C₁ (just c)

Split C₁ into:
  C₂ = {a, d}  (degree 1) — size 2
  C₃ = {b, c}  (degree 2) — size 2

Both same size, so add C₂ to Q (could pick either)
Q = {2}
```

**Step 2**: Pop color 2 from Q, refine by counting neighbors in C₂ = {a, d}

```
Count neighbors in C₂:
  a: 0 neighbors in C₂
  b: 1 neighbor in C₂ (just a)
  c: 1 neighbor in C₂ (just d)
  d: 0 neighbors in C₂

C₂ = {a, d}: both have 0 neighbors in C₂ → no split
C₃ = {b, c}: both have 1 neighbor in C₂ → no split

No splits → Q is empty → DONE
```

**Final coloring:**

```
C₂ = {a, d}  (endpoints)
C₃ = {b, c}  (middle vertices)
```

**Key difference from naive**: We only process color classes in Q, and we add only the _smaller_ splits to Q, ensuring each vertex appears O(log n) times.

---

### Exercise 6.3: Grouping Color Strings in O(m) — **Secondary**

> This exercise is about implementing one WL iteration in O(m) time.

**Problem**: Given color histograms (strings) for each vertex from the previous iteration, group vertices with identical strings so we can assign them the same new color.

**Key insight**: String lengths sum to O(m) because each neighbor contributes one character to a vertex's string.

**Algorithm**: BucketSort-based grouping

```
Algorithm: DistinguishColorStrings
Input: Set of strings S = {S₁, ..., Sₙ} representing color histograms

1. Initialize queue ToRefine = {(S, 0)}
2. Initialize ColorClasses = ∅

3. while ToRefine not empty:
     Pop (S', k) from ToRefine

     if |S'| = 1:
         Add S' to ColorClasses
         continue

     // Bucket by k-th character
     for each string Sᵢ in S':
         if length(Sᵢ) ≤ k:
             Add Sᵢ to bucket[n]  // "ended" strings
         else:
             cₖ = k-th character of Sᵢ
             Add Sᵢ to bucket[cₖ]

     // Finalize "ended" strings
     if bucket[n] non-empty:
         Add bucket[n] to ColorClasses

     // Queue remaining buckets for further refinement
     for each non-empty bucket[i]:
         Add (bucket[i], k+1) to ToRefine
```

**Runtime Analysis**:

- Total string length: L = Σᵢ lᵢ = O(m) (each edge contributes to both endpoints)
- Each character is processed once → O(L) = O(m) total
- Space: O(n + m)

**Why this matters**: This is how you actually implement efficient WL — not by comparing full strings, but by bucketing character-by-character.

---

## k-Dimensional WL (k-WL)

> ⚠️ **Exam question**: "What is k-dimensional Weisfeiler-Leman?"

Standard WL works on **vertices**. k-WL works on **k-tuples (or k-sets)** of vertices.

> ⚠️ **Exam question**: "Describe the k-WL. Draw a small graph and apply it."

**k-WL Algorithm:**

```
Initial: k-sets U, W get same color if G[U] ≃ G[W] (induced subgraphs isomorphic)

Iteration: U, W (same color) get different colors if they have
           different numbers of neighbors of some color c

Neighbors: Two k-sets are neighbors if they differ in exactly one element
```

### Traced Example: 2-WL Distinguishing Two Graphs

```
G: triangle          H: path of 3
   a─b                  a─b─c
   ╲╱
    c
```

**2-sets in G (triangle):**

- {a,b}: edge → "E"
- {a,c}: edge → "E"
- {b,c}: edge → "E"

**2-sets in H (path):**

- {a,b}: edge → "E"
- {b,c}: edge → "E"
- {a,c}: NO edge → "N" (non-edge)

**Histogram comparison:**

- G: (E:3, N:0)
- H: (E:2, N:1)

**Different histograms → G ≄ H** ✓

---

### Exercise 6.1: k-dimensional WL — **Important**

> Apply (a) 2-WL and (b) sparse 2-WL on this graph:

```
    A ─── B
    │╲   ╱│
    │ ╲ ╱ │
    │  E  │
    │ ╱ ╲ │
    │╱   ╲│
    D ─── C
```

**(a) 2-WL (Standard):**

**Step 0: List all 2-sets**

There are C(5,2) = 10 pairs. Initial color = induced subgraph structure:

| Pair  | Edge? | Initial Color |
| ----- | ----- | ------------- |
| {A,B} | Yes   | E (edge)      |
| {A,D} | Yes   | E             |
| {A,E} | Yes   | E             |
| {B,C} | Yes   | E             |
| {B,E} | Yes   | E             |
| {C,D} | Yes   | E             |
| {C,E} | Yes   | E             |
| {D,E} | Yes   | E             |
| {A,C} | No    | N (non-edge)  |
| {B,D} | No    | N             |

**Count**: 8 edge-pairs (color E), 2 non-edge pairs (color N)

**Iteration 1**: For each 2-set, count neighbors of each color.

Neighbors of {A,B} (differ in one element):

- {A,C}, {A,D}, {A,E} — swap B for C,D,E
- {B,C}, {B,D}, {B,E} — swap A for C,D,E

{A,B} has: {A,C}=N, {A,D}=E, {A,E}=E, {B,C}=E, {B,D}=N, {B,E}=E

Neighbor histogram: (E:4, N:2)

By symmetry (the graph is highly symmetric), all edge-pairs have the same neighbor histogram → **no refinement**.

**(b) Sparse 2-WL (Local 2-WL):**

In local k-WL, neighbors must be connected by edges. This means:

- {A,B} and {A,C} are neighbors only if B-C have an edge (yes)
- {A,B} and {B,D} are neighbors only if A-D have an edge (yes)

Sparse 2-WL typically refines more aggressively because it uses edge structure.

(Full trace requires checking edge connectivity for each neighbor relationship)

---

> ⚠️ **Exam question**: "How does k-WL compare to normal WL?"

**Properties:**

- k-WL (k > 2) is **stronger** than 1-WL
- For large enough k: k-WL distinguishes ALL graphs
- Babai's algorithm uses k = O(log n)
- Some graph classes need k = Θ(n)
- **Runtime**: O(k² · n^(k+1) · log n) — exponential in k!

> ⚠️ **Exam question**: "Can k-dimensional WL distinguish any two graphs?"
>
> **Answer**: Yes, for large enough k. If k = n (number of vertices), k-WL can distinguish any two non-isomorphic graphs. But the runtime becomes exponential.

> ⚠️ **Exam question**: "What happens if we use k = n/2 (big number)?"
>
> **Answer**: k-WL becomes very powerful (can distinguish almost all graphs), but runtime is O(n^(n/2)), which is impractical. This is why we prefer small k (2 or 3) in practice.

> 🎤 **If examiner asks**: "Why don't we always use k-WL instead of 1-WL?"
>
> **Your answer**: "Runtime. k-WL is O(n^(k+1)), which is exponential in k. For practical graphs, 1-WL or 2-WL is usually enough, and much faster."

---

## k-WL Can Distinguish Regular Graphs

```
2-WL on two 2-regular graphs:

G: 6-cycle                    H: two triangles (K₃ ∪ K₃)

    1───2                         a───b
    │   │                          ╲ ╱
    6   3                           c
    │   │
    5───4                         d───e
                                   ╲ ╱
                                    f
```

**Why 1-WL fails:**

All vertices have degree 2 → same hash → same color. No refinement. 1-WL says "maybe isomorphic."

**How 2-WL succeeds:**

2-WL works on **pairs of vertices**. Initial color = structure of the pair (edge or not).

```
In G (6-cycle):
  Pair {1,2}: adjacent → color "ADJ"
  Pair {1,3}: not adjacent, dist=2 → color "D2"
  Pair {1,4}: not adjacent, dist=3 (opposite) → color "D3"

In H (two triangles):
  Pair {a,b}: adjacent → color "ADJ"
  Pair {a,c}: adjacent → color "ADJ"
  Pair {a,d}: not adjacent (different component!) → color "DISC"
```

**Key difference:**

In G: No pair has distance > 3 (connected graph)
In H: Some pairs are disconnected (different triangles)

2-WL detects this: color "DISC" exists in H but not in G → different histograms!

**Result:**

- 1-WL: Same histogram (cannot distinguish)
- 2-WL: Different histograms → G ≄ H ✓

---

## Local k-WL (k-LWL) — EXAM RELEVANT

> ⚠️ **Exam question**: "Describe local k-WL. How is it different from k-WL? What are its advantages?"

**Idea**: k-LWL restricts which k-sets are considered "neighbors" by using the graph structure.

---

### Difference: k-WL vs k-LWL Neighbors

**What does "differ in one element" mean?**

Two k-sets U and W "differ in one element" if:

- They share exactly k-1 vertices
- U has one vertex that W doesn't have (call it $s$)
- W has one vertex that U doesn't have (call it $t$)

Example with 2-sets:

```
U = {a, b}
W = {a, c}

They differ in one element: U has b, W has c, both have a.
```

| Algorithm | Neighbor Definition                                                                                                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **k-WL**  | Two k-sets are neighbors if they differ in exactly **one element**                                                                       |
| **k-LWL** | Two k-sets are neighbors if they differ in one element, **AND** there's an edge connecting the differing vertices to the shared vertices |

---

**k-LWL neighbor condition (formal)**:

For k-sets U and W with $s \in U \setminus W$ and $t \in W \setminus U$:

- There exists $u \in W$ such that $(s, u) \in E$
- There exists $v \in U$ such that $(t, v) \in E$

**where:**

- $U, W$ = two k-sets (subsets of vertices of size k)
- $s$ = the vertex in U but not in W (U's unique vertex)
- $t$ = the vertex in W but not in U (W's unique vertex)
- $u$ = some vertex in W that s is connected to
- $v$ = some vertex in U that t is connected to
- $E$ = the edge set of the graph

**In words**: U and W are k-LWL neighbors only if s has an edge to some vertex in W, and t has an edge to some vertex in U.

---

### Why k-LWL is Better (Advantages)

**1. Much Faster:**

- k-WL: Every k-set is neighbor to O(n) other k-sets (can swap any vertex)
- k-LWL: Each k-set only considers neighbors along actual edges → sparse graphs = fewer neighbors
- **Exploits sparsity** of the original graph

**2. At Least as Strong:**

- k-LWL refines **at least as much** as k-WL
- If k-WL can distinguish two graphs → k-LWL can too
- Theorem: Local k-LWL refines at least as much as k-WL

**3. Sometimes Strictly Stronger:**

- There exist graphs that k-LWL can distinguish but k-WL cannot
- Example: Cai-Fürer-Immerman graphs — 2-WL gives 4 color classes, 2-LWL gives 5

---

### Summary Table

| Property    | k-WL                              | k-LWL                                  |
| ----------- | --------------------------------- | -------------------------------------- |
| Neighbors   | Any k-sets differing in 1 element | Must be connected by edges             |
| Speed       | O(k² · n^(k+1) · log n)           | Faster (exploits sparsity)             |
| Strength    | Strong                            | At least as strong, sometimes stronger |
| When to use | Dense graphs                      | Sparse graphs                          |

> 🎤 **If examiner asks**: "What is the advantage of k-LWL over k-WL?"
>
> **Your answer**: "Two advantages: (1) It's faster because it only considers neighbors connected by edges, which is much smaller for sparse graphs. (2) It's at least as strong as k-WL — if k-WL can distinguish two graphs, so can k-LWL. In fact, k-LWL is sometimes strictly stronger."

---

## Runtime Summary

| Algorithm      | Time                       |
| -------------- | -------------------------- |
| Naive WL       | O(n³)                      |
| Efficient WL   | O(n² log n)                |
| Even better WL | O((n+m) log n)             |
| k-WL           | O(k² · n^(k+1) · log n)    |
| k-LWL          | Faster (exploits sparsity) |

---

## WL for Classification — Feature Vectors

**Idea**: Use WL color histograms as feature vectors for machine learning.

After each WL iteration, create vector Φ(G) = (count of color 1, count of color 2, ...)

```
Example:
G after round 1: Φ₁(G) = (3, 1, 1)  — 3 vertices color A, 1 color B, 1 color C
H after round 1: Φ₁(H) = (3, 1, 1)  — same histogram

G after round 2: Φ₂(G) = (2, 0, 0, 1, 1, 1, 0)
H after round 2: Φ₂(H) = (2, 1, 1, 0, 0, 0, 1)  — different!
```

**WL Subtree Kernel**:
$$k_t(G_i, G_j) = \langle \Phi_t(G_i), \Phi_t(G_j) \rangle$$

Use with SVM for graph classification.

---

## Exam Triage

**Exam-critical (write from memory):**

- WL algorithm pseudocode
- Trace WL on a small graph (4-5 vertices)
- What WL computes (coarsest equitable partition)
- When WL fails (regular graphs)
- One-sided isomorphism test interpretation

**Know conceptually:**

- Efficient WL O(n² log n) — why? (Hopcroft trick, log n appearances)
- k-WL is stronger, but exponential runtime
- k-LWL is faster and at least as strong

**Stop wasting time on:**

- Classification with WL (ML applications)
- Specific graph kernels
- Detailed k-LWL proofs
- Connections to deep learning/GNNs

---

## STOP. Draw and Trace

Take this graph:

```
    a ─── b
    │
    c ─── d ─── e
```

Run WL until stable. What are the final color classes?

**Hint**: Start with all same color. Count neighbors. Who has degree 1? Who has degree 2? Who has degree 3?

---

## Oral Defense Summary — If Interrupted

| Examiner Question                     | Your Answer                                                      |
| ------------------------------------- | ---------------------------------------------------------------- |
| "Write the WL algorithm"              | Draw pseudocode: Initial all same, repeat hash(color, neighbors) |
| "Trace WL on this graph"              | Draw, label colors, iterate until stable, state histogram        |
| "Why does WL fail on regular graphs?" | Same degree → same hash → no refinement                          |
| "Can WL prove isomorphism?"           | No! One-sided: can disprove but not prove                        |
| "What does WL compute?"               | Coarsest equitable partition                                     |
| "Why is efficient WL O(n² log n)?"    | Each vertex appears log n times in Q (halving)                   |
| "Why not always use k-WL?"            | Runtime is O(n^(k+1)) — exponential in k                         |

---

## Quick Reference — Write From Memory

```
WL-Algorithm:
    Initial: All vertices get same color
    Repeat until stable:
        For each v: new_color[v] = hash(color[v], sorted neighbor colors)
        color = new_color
```

**Histogram** = count of vertices per color, e.g., (2, 3, 1)

**One-sided test**: Different histograms → NOT isomorphic. Same → inconclusive.

**Fails on**: Regular graphs (same degree everywhere)

**Runtime**: Naive O(n³), Efficient O(n² log n), Best O((n+m) log n)

---
