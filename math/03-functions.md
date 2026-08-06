---
title: "Functions & Optimization"
tags:
  - mathematics
  - computer-science
  - reference
---

# Functions & Optimization

> [!info] Part 3 of 8 · Previous: [[math/02-operators|Operators]] · [[math/index|Index]] · Next: [[math/04-algebra|Algebra]]

## 13. Functions

### Function type

$$
f:A\to B
$$

Read:

> $f$ is a function from set $A$ to set $B$.

- $A$: domain
- $B$: codomain

Example:

$$
\ell:\mathcal P\to\mathbb N
$$

may mean:

> $\ell$ maps every path to its integer length.

### Mapping notation

$$
x\mapsto x^2
$$

Read:

> Map $x$ to $x^2$.

Together:

$$
f:\mathbb R\to\mathbb R,\qquad x\mapsto x^2
$$

### Function composition

$$
(f\circ g)(x)=f(g(x))
$$

Read from right to left:

1. apply $g$;
2. then apply $f$.

---

## 14. Piecewise definitions

A function can have different definitions in different cases:

$$
f(x)=
\begin{cases}
0, & x<0,\\
x, & x\ge0.
\end{cases}
$$

Read:

- $f(x)=0$ when $x<0$;
- $f(x)=x$ when $x\ge0$.

The function:

$$
\max(0,x)
$$

can also be written:

$$
\max(0,x)=
\begin{cases}
0, & x<0,\\
x, & x\ge0.
\end{cases}
$$

---

## 15. Indicator functions

$$
\mathbf 1[P]
$$

or:

$$
\mathbb 1_P
$$

means:

$$
\mathbf 1[P]
=
\begin{cases}
1,&P\text{ is true},\\
0,&P\text{ is false}.
\end{cases}
$$

Example:

$$
\sum_a
\mathbf 1[\text{ant }a\text{ occupies room }v]
\le1
$$

Read:

> Count all ants occupying room $v$; that count must be at most one.

Indicator functions are common in:

- algorithms;
- probability;
- combinatorics;
- integer programming;
- graph optimization.

---

## 16. Common optimization form

A mathematical optimization problem often looks like:

$$
\begin{aligned}
\text{minimize}\quad & f(x)\\
\text{subject to}\quad & g(x)\le0,\\
& x\in S.
\end{aligned}
$$

Terminology:

- $x$: decision variable;
- $f(x)$: objective function;
- $g(x)\le0$: constraint;
- $S$: feasible domain;
- **feasible solution**: a value satisfying every constraint;
- **optimal solution**: a feasible value producing the best objective.

Compact form:

$$
\min_{\substack{x\in S\\g(x)\le0}}f(x)
$$

---

## 17. Reading a Lem-in optimization formula

Consider:

$$
T(\mathcal P)
=
\min_{\substack{x_i\in\mathbb N_0\\
\sum_i x_i=N}}
\max_{i:x_i>0}
(d_i+x_i-1)
$$

### Left-hand side

$$
T(\mathcal P)
$$

means:

> Completion time obtained with path family $\mathcal P$.

### Outer minimum

$$
\min
$$

means:

> Choose the best allocation.

### Constraints below the minimum

$$
x_i\in\mathbb N_0
$$

means every $x_i$ is a nonnegative integer.

$$
\sum_i x_i=N
$$

means all $N$ ants are assigned.

### Inner maximum

$$
\max_{i:x_i>0}
$$

means:

> Among paths receiving at least one ant, find the one that finishes last.

### Path completion expression

$$
d_i+x_i-1
$$

means:

- $d_i$: time for the first ant;
- $x_i-1$: additional turns for the remaining ants.

### Full translation

> The completion time is the smallest possible finishing time of the slowest used path, considering every valid way to distribute all $N$ ants among the paths.

---

## 18. Reading the capacity formula

$$
T(\mathcal P)
=
\min
\left\{
\tau\in\mathbb N:
\sum_{P_i\in\mathcal P}
\max(0,\tau-d_i+1)
\ge N
\right\}
$$

### Translation by component

$$
\tau\in\mathbb N
$$

Candidate completion turn $\tau$ must be a natural number.

$$
\max(0,\tau-d_i+1)
$$

Number of ants path $i$ can deliver by turn $\tau$.

$$
\sum_{P_i\in\mathcal P}
$$

Add this capacity for every compatible path.

$$
\ge N
$$

There must be enough total capacity for all $N$ ants.

$$
\min\{\cdots\}
$$

Choose the earliest deadline satisfying the condition.

### Full translation

> $T(\mathcal P)$ is the earliest integer turn at which the combined capacity of the compatible paths is sufficient to deliver all $N$ ants.

---
