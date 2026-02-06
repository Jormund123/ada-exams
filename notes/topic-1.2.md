# Shortest Path Speedup Techniques

## Exam Relevance — CRITICAL

⚠️ Exam questions from `specs/questions.txt`:

- "Which speedup techniques for shortest path search do you know?"
- "What properties should a heuristic for A\* have?"
- "How does bidirectional search work?"
- "How does ALT work and how are landmarks chosen?"
- "How do contraction hierarchies work and draw small example?"
- "What is the witness search step?"

These are **primary exam topics**. You must be able to explain ALL of these under pressure.

---

## Table of Contents

1. [Why We Need Speedup Techniques](#why-we-need-speedup-techniques)
2. [⚠️ A\* Algorithm — EXAM CRITICAL](#️-a-algorithm--exam-critical)
3. [⚠️ Bidirectional Search — EXAM CRITICAL](#️-bidirectional-search--exam-critical)
4. [⚠️ ALT (Landmarks + Triangle Inequality)](#️-alt-landmarks--triangle-inequality)
5. [⚠️ Contraction Hierarchies — EXAM CRITICAL](#️-contraction-hierarchies--exam-critical)
6. [Arc Flags (Secondary)](#arc-flags-secondary)
7. [Hub Labeling (Secondary)](#hub-labeling-secondary)

---

## Why We Need Speedup Techniques

**Dijkstra's Problem**: It's fast (O(m + n log n) with Fibonacci heaps), but it explores **all directions equally**.

```
                        ╭─────────────╮
                        │   Target    │
                        ╰──────▲──────╯
                               │
    Dijkstra explores          │
    this entire circle         │
    before reaching target     │
         ╭─────────────────────┼─────────────────────╮
         │                     │                     │
         │        ●────────────●────────────●        │
         │       ╱│╲                       ╱│╲       │
         │      ● ● ●                     ● ● ●      │
         │     ╱│╲                           ╱│╲     │
         │    ● ● ●       START ●           ● ● ●   │
         │                                           │
         ╰───────────────────────────────────────────╯
```

**Goal of speedup techniques**: Search **toward the target**, not everywhere.

---

## ⚠️ A\* Algorithm — EXAM CRITICAL

⚠️ Exam question: "What properties should a heuristic for A\* have?"

### Why A\* Exists

Dijkstra picks the next vertex with smallest **distance from source**: d(s,v).

A\* picks the vertex with smallest **estimated total path length**: d(s,v) + h(v).

Here h(v) is a **heuristic** estimating distance from v to target t.

### The Heuristic — What It Must Satisfy

**Feasibility (Admissibility)**: h(v) must **never overestimate** the true distance to t.

$$h(v) \leq d(v, t) \quad \forall v$$

**Why?** If h overestimates, A\* might skip the optimal path thinking it's too expensive.

```
If h(v) = 0 for all v:
  → A* becomes Dijkstra (explores everything)

If h(v) = d(v,t) exactly:
  → A* only explores vertices on the shortest path (optimal)

If h(v) > d(v,t) (overestimates):
  → A* may find WRONG answer!
```

**The examiner might ask**: "Can h underestimate?"

**Answer**: Yes! Underestimating is safe — we just explore more than necessary. Overestimating is dangerous — we might miss the optimal path.

### How A\* Works

```
A*(s, t):
    dist[s] = 0, dist[v] = ∞ for v ≠ s
    Q = priority queue with priorities f(v) = dist[v] + h(v)
    Insert s with priority h(s)

    while Q ≠ ∅:
        v = ExtractMin(Q)
        if v == t:
            return dist[t]    // Found target!

        for each neighbor w of v:
            if dist[w] > dist[v] + c(v,w):
                dist[w] = dist[v] + c(v,w)
                UpdateQ(w, dist[w] + h(w))

    return ∞  // No path
```

### Visual: A\* vs Dijkstra

```
Dijkstra (explores circular):    A* (explores toward target):

        ●●●●●                            ●
       ●●●●●●●                          ●●●
      ●●●●●●●●●                        ●●●●●
     ●●●●S●●●●●                       ●●S●●●●
      ●●●●●●●●●                        ●●●●●●●
       ●●●●●●●         T                ●●●●●●●●●  T
        ●●●●●                            ●●●●●●●●●●

S = Start, T = Target
● = Explored vertices
```

### Common Heuristics

**Euclidean distance** (for geometric graphs):
$$h(v) = \sqrt{(x_v - x_t)^2 + (y_v - y_t)^2}$$

**Manhattan distance** (for grid graphs):
$$h(v) = |x_v - x_t| + |y_v - y_t|$$

---

**STOP. Active Recall**:

If h(v) = 1000 for all v, is this a valid A\* heuristic?

**Answer**: It's only valid if the true distance to target is ≥ 1000 for all vertices. If any vertex is closer than 1000 to the target, this overestimates and could give wrong answers. In practice, this is almost certainly invalid.

---

## ⚠️ Bidirectional Search — EXAM CRITICAL

⚠️ Exam question: "How does bidirectional search work?"

### The Idea

Instead of searching from s to all vertices until we hit t, search from **both ends**:

- **Forward search**: from s in original graph
- **Backward search**: from t in **reversed graph** (same weights, flipped edges)

```
Forward from s:              Backward from t:

    s ───→ ● ───→ ●          ● ←─── ● ←─── t
    │      │      │          │      │      │
    ▼      ▼      ▼          ▼      ▼      ▼
    ● ───→ ● ───→ ●    +     ● ←─── ● ←─── ●
    │      │                        │      │
    ▼      ▼                        ▼      ▼
    ● ───→ ●                 ● ←─── ●

              Meet in the middle!
```

### How It Works

```
BidirectionalDijkstra(s, t):
    dist_f[s] = 0, dist_b[t] = 0
    Q_f = forward queue, Q_b = backward queue
    μ = ∞  // best path length found so far

    while Q_f ≠ ∅ and Q_b ≠ ∅:
        // Expand from whichever queue has smaller minimum
        if key(Q_f) ≤ key(Q_b):
            v = ExtractMin(Q_f)
            for each neighbor w of v (forward):
                relax(v, w)
                if w visited by backward search:
                    μ = min(μ, dist_f[v] + c(v,w) + dist_b[w])
        else:
            // Symmetric for backward direction

        // TERMINATION CONDITION
        if μ ≤ key(Q_f) + key(Q_b):
            return μ

    return μ
```

### The Termination Condition — Exam Trap!

**Wrong stopping condition**: "Stop when both searches visit the same vertex."

**Correct stopping condition**:
$$\mu \leq \text{key}(Q_f) + \text{key}(Q_b)$$

**Why?** Meeting at a vertex doesn't mean you found the shortest path through it!

```
Counterexample:

    s ───1─── a ───1─── t
         ╲           ╱
          5         5
           ╲       ╱
            ╲─ b ─╱

Forward visits: s, a  (distances 0, 1)
Backward visits: t, a (distances 0, 1)

They meet at 'a' with path length 1+1=2 ✓

But wait — could there be a shorter path through b?
No, because key(Q_f) + key(Q_b) = 1 + 1 = 2 ≤ μ = 2

We can stop because any undiscovered path would cost ≥ 2.
```

---

**STOP. The examiner asks**: "Why can't we stop as soon as both searches visit the same vertex?"

**Answer**: Because the path through that vertex might not be optimal. There could be a shorter path through a vertex not yet visited by both searches. We can only stop when the sum of the queue minimums exceeds our best found path — at that point, no undiscovered path can be shorter.

---

## ⚠️ ALT (Landmarks + Triangle Inequality)

⚠️ Exam question: "How does ALT work and how are landmarks chosen?"

### The Problem with A\*

We need a heuristic h(v) that estimates d(v, t). For road networks, Euclidean distance works, but for general graphs we have nothing!

**ALT solution**: Precompute distances to/from a small set of **landmarks**.

### Preprocessing

1. Choose k **landmarks** L = {l₁, l₂, ..., l_k} (usually ~16)
2. For each landmark l and each vertex v, precompute:
   - d(v, l) = distance from v to l
   - d(l, v) = distance from l to v

**Storage**: O(k·n) — very reasonable.

### The Heuristic

Using the **triangle inequality**:

$$d(v, t) \geq d(l, t) - d(l, v)$$
$$d(v, t) \geq d(v, l) - d(t, l)$$

So the heuristic is:

$$h(v) = \max_{l \in L} \{ |d(l, t) - d(l, v)|, |d(v, l) - d(t, l)| \}$$

This is **always admissible** (never overestimates) because it's derived from triangle inequality.

### Visual: Why "Behind" Landmarks Work Best

```
        l (landmark behind target)
        │
        │  d(l,t)
        │
        ▼
    ────●──────●──────●────  ← Target t
        ▲      ▲
        │      │
        │      │ d(l,v)
        │      │
    ────●──────v──────●────  ← Current vertex v
                      │
                      │
                      Start s

Heuristic: h(v) = d(l,t) - d(l,v) ≈ d(v,t)

The more "behind" the landmark, the tighter the bound!
```

### How to Choose Landmarks

**Strategies**:

- **Farthest selection**: Pick vertices far from each other (covers periphery)
- **Planar selection**: For road networks, place landmarks around the boundary
- **Random**: Surprisingly okay for some applications

---

**STOP. Active Recall**:

You have landmarks l₁ with d(l₁, t) = 100, d(l₁, v) = 30, and l₂ with d(l₂, t) = 50, d(l₂, v) = 60.

What is h(v)?

**Answer**:

- From l₁: |100 - 30| = 70
- From l₂: |50 - 60| = 10
- h(v) = max(70, 10) = **70**

---

## ⚠️ Contraction Hierarchies — EXAM CRITICAL

⚠️ Exam question: "How do contraction hierarchies work? What is witness search?"

This is the most powerful speedup technique for road networks. **Know it cold.**

### The Idea

1. **Preprocessing**: Remove vertices one by one, add **shortcuts** to preserve distances
2. **Query**: Bidirectional search that only goes "upward" in the hierarchy

### Preprocessing: Contraction

**Contract vertex v**:

1. Remove v from the graph
2. For each pair of neighbors u, w of v:
   - Check if the shortest u→w path used v
   - If yes, add **shortcut edge** (u, w) with weight d(u,v) + d(v,w)

```
Before contracting v:           After contracting v:

    u ───2─── v ───3─── w       u ─────────5────────── w
                                        (shortcut)
```

### Witness Search — CRITICAL!

⚠️ Exam question: "What is the witness search step?"

Before adding shortcut (u, w), we check: **Is there already a path u→w that doesn't use v?**

```
WitnessSearch(u, w, v):
    // Run limited Dijkstra from u, ignoring v
    // If we find w with distance ≤ d(u,v) + d(v,w):
    //   → Don't add shortcut (a "witness" path exists)
    // Otherwise:
    //   → Add shortcut (u,w)
```

**Why limit the search?** For efficiency. Often we limit to k hops or a distance threshold.

**Exam follow-up**: "When is it sensible to cut witness search short?"

**Answer**: When we're confident no witness exists. If after exploring k vertices we haven't found w, the probability of a witness is low. Cutting short may add unnecessary shortcuts, but it speeds up preprocessing.

### Example: Contraction Step-by-Step

```
Original graph:

    a ───2─── b ───1─── c
    │         │         │
    3         2         4
    │         │         │
    d ───1─── e ───2─── f

Contract vertex 'e' (least important):

Neighbors of e: b, d, f

Check pairs:
  b-d: d(b,e) + d(e,d) = 2+1 = 3
       Witness? Is there b→d without e? b→a→d = 2+3 = 5 > 3. No witness.
       → Add shortcut (b,d) = 3

  b-f: d(b,e) + d(e,f) = 2+2 = 4
       Witness? b→c→f = 1+4 = 5 > 4. No witness.
       → Add shortcut (b,f) = 4

  d-f: d(d,e) + d(e,f) = 1+2 = 3
       Witness? None. → Add shortcut (d,f) = 3

After contracting e:

    a ───2─── b ───1─── c
    │        ╱│╲        │
    3      3╱ │ ╲4      4
    │      ╱  │  ╲      │
    d ────╱   │   ╲──── f
         (shortcuts)
```

### Query: Upward-Only Bidirectional Search

After preprocessing, each vertex has a "level" (order of contraction).

**Query algorithm**:

- Forward search from s, only relax edges going to **higher-level** vertices
- Backward search from t, only relax edges going to **higher-level** vertices
- They meet at the "apex" — highest vertex on the shortest path

```
Level:  5       ●───────────────────●       5
               ╱│╲                 ╱│╲
        4    ●  │  ●             ●  │  ●    4
             │  │  │             │  │  │
        3    ●  │  ●             ●  │  ●    3
             │  │  │             │  │  │
        2    ●──●──●             ●──●──●    2

        1    s                         t    1

Forward search: s → level 2 → level 3 → level 4 → level 5
Backward search: t → level 2 → level 3 → level 4 → level 5
Meet at level 5!
```

### Why It's Fast

- Most queries only visit ~1000 vertices (even in graphs with millions)
- The hierarchy compresses long-distance information into shortcuts
- Query time: typically 0.5ms on continental road networks

---

**STOP. Draw this on your whiteboard**:

Contract vertex 'b' from this graph and show which shortcuts are added:

```
    a ───3─── b ───2─── c
              │
              1
              │
              d
```

**Answer**:

Neighbors of b: a, c, d

Check pairs:

- a-c: d(a,b) + d(b,c) = 3+2 = 5. Witness? No direct a-c edge. Add shortcut (a,c) = 5.
- a-d: d(a,b) + d(b,d) = 3+1 = 4. Witness? No. Add shortcut (a,d) = 4.
- c-d: d(c,b) + d(b,d) = 2+1 = 3. Witness? No. Add shortcut (c,d) = 3.

After:

```
    a ─────5───── c
    │╲           ╱│
    │ ╲    3    ╱ │
    4  ╲       ╱  │
    │   ╲     ╱   │
    d ────────    │
```

---

## Arc Flags (Secondary)

Not in main exam questions, but mentioned. **Brief coverage only**.

### The Idea

1. Partition graph into **k regions** (e.g., by geography)
2. For each edge, precompute **k bits** (arc flags):
   - Bit i = 1 if edge is on a shortest path to **some** vertex in region i
3. During query to target t in region r, only use edges with flag r = 1

```
Preprocessing:

Region 1 │ Region 2 │ Region 3
         │          │
   ●─────●──────────●─────●
   │     │          │     │
   ●─────●          ●─────●
         │                │

For edge e:
  flags[e] = [1, 1, 0]  // On path to region 1 and 2, not 3
```

**Query**: Standard Dijkstra, but skip edges where flag[target_region] = 0.

---

## Hub Labeling (Secondary)

Mentioned in exam protocols. **Brief coverage**.

### The Idea

For each vertex v, precompute two label sets:

- **Forward label** L_f(v): Set of "hub" vertices reachable from v with their distances
- **Backward label** L_b(v): Set of "hub" vertices that can reach v with their distances

**Query**: d(s, t) = min over all h ∈ L_f(s) ∩ L_b(t) of d(s,h) + d(h,t)

```
Labels for vertex a:
  L_f(a) = {(hub1, 10), (hub2, 25), (hub3, 40)}
  L_b(a) = {(hub1, 8), (hub4, 15)}

Query d(a, b):
  Find common hubs in L_f(a) and L_b(b)
  Return minimum sum of distances through any common hub
```

**Key insight**: If labels are small and share common hubs, queries are O(|L|) — extremely fast.

---

## Exam Triage

**These parts are exam-critical**:

- A\* algorithm and **heuristic properties** (admissibility)
- Bidirectional search and **termination condition**
- ALT heuristic formula and landmark selection
- Contraction hierarchies with **witness search**

**These parts are secondary**:

- Arc Flags (know the idea)
- Hub Labeling (know the idea)

**Stop wasting time on**:

- Implementation details of preprocessing
- Complexity proofs for CH preprocessing

---

## Quick Reference: One-Sentence Summaries

| Technique                   | One-Sentence Summary                                         |
| --------------------------- | ------------------------------------------------------------ |
| **A\***                     | Dijkstra with heuristic h(v) directing search toward target  |
| **Bidirectional**           | Search from both ends, stop when queue sums exceed best path |
| **ALT**                     | A\* with precomputed landmark distances as heuristic         |
| **Contraction Hierarchies** | Remove vertices adding shortcuts, query goes upward only     |
| **Arc Flags**               | Precompute which edges lead to which regions                 |
| **Hub Labeling**            | Precompute distance labels through shared "hubs"             |

---
