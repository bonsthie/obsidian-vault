---
title: "Mathematical Notation — Algebra and Graphs"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Algebra and Graphs

> [!info] Dictionary part 4 of 8 · Previous: [[math/03 - Functions and Optimization|Functions and Optimization]] · [[math/index|Index]] · Next: [[math/05 - Sequences Complexity and Probability|Sequences, Complexity, and Probability]]

## 19. Fractions

$$
\frac{a}{b}
$$

means $a$ divided by $b$.

- top: **numerator**
- bottom: **denominator**

Example:

$$
A_k=
\frac{N+D_k-k}{k}
$$

Read the entire numerator before dividing:

> Add $N$ and $D_k$, subtract $k$, then divide the complete result by $k$.

In code:

```text
A_k = (N + D_k - k) / k
```

---

## 20. Common algebra transformations

### Add the same value to both sides

From:

$$
a-b=c
$$

derive:

$$
a=c+b
$$

### Multiply both sides

From:

$$
\frac{x}{k}\ge y
$$

and $k>0$:

$$
x\ge ky
$$

When multiplying by a negative number, reverse the inequality:

$$
-2x<6
$$

becomes:

$$
x>-3
$$

### Substitution

If:

$$
D_{k+1}=D_k+\Delta_{k+1}
$$

then in:

$$
A_{k+1}
=
\frac{N+D_{k+1}-(k+1)}{k+1}
$$

replace $D_{k+1}$:

$$
A_{k+1}
=
\frac{N+D_k+\Delta_{k+1}-(k+1)}{k+1}
$$

Authors often write:

> Substituting $D_{k+1}=D_k+\Delta_{k+1}$, we obtain…

---

## 21. Graph-theory notation

| Notation | Meaning |
|---|---|
| $G=(V,E)$ | Graph with vertices $V$ and edges $E$ |
| $V(G)$ | Vertex set of graph $G$ |
| $E(G)$ | Edge set of graph $G$ |
| $u\sim v$ | $u$ and $v$ are adjacent |
| $(u,v)\in E$ | Directed edge from $u$ to $v$ |
| $\{u,v\}\in E$ | Undirected edge between $u,v$ |
| $\deg(v)$ | Degree of vertex $v$ |
| $N(v)$ | Neighbourhood of $v$ |
| $P:u\leadsto v$ | Path from $u$ to $v$ |
| $\ell(P)$ | Length of path $P$ |
| $d(u,v)$ | Shortest-path distance |
| $G[S]$ | Subgraph induced by vertices $S$ |
| $G-v$ | Graph after removing vertex $v$ |
| $G-e$ | Graph after removing edge $e$ |

### Disjoint paths

$$
V(P_i)\cap V(P_j)=\{s,t\}
$$

means:

> Paths $P_i$ and $P_j$ share only `start` and `end`.

Equivalently:

$$
\operatorname{Int}(P_i)
\cap
\operatorname{Int}(P_j)
=
\varnothing
$$

---

