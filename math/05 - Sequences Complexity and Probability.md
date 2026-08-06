---
title: "Mathematical Notation — Sequences, Complexity, and Probability"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Sequences, Complexity, and Probability

> [!info] Dictionary part 5 of 8 · Previous: [[math/04 - Algebra and Graphs|Algebra and Graphs]] · [[math/index|Index]] · Next: [[math/06 - Algorithms Languages and Types|Algorithms, Languages, and Types]]

## 22. Sequence and recurrence notation

### Sequence

$$
(a_n)_{n\ge0}
$$

means:

$$
a_0,a_1,a_2,\ldots
$$

### Recurrence

$$
T(n)=T(n-1)+n
$$

defines $T(n)$ using a smaller input.

A recurrence normally needs a base case:

$$
T(0)=0
$$

Then:

$$
T(1)=T(0)+1=1
$$

$$
T(2)=T(1)+2=3
$$

Common algorithm recurrence:

$$
T(n)=2T(n/2)+O(n)
$$

This describes an algorithm that:

- creates two subproblems of size $n/2$;
- performs $O(n)$ additional work.

---

## 23. Asymptotic complexity

| Notation | Informal meaning |
|---|---|
| $O(f(n))$ | Grows no faster than $f(n)$, up to a constant |
| $\Omega(f(n))$ | Grows at least as fast as $f(n)$ |
| $\Theta(f(n))$ | Grows at the same asymptotic rate |
| $o(f(n))$ | Grows strictly slower |
| $\omega(f(n))$ | Grows strictly faster |
| $f(n)\sim g(n)$ | Their ratio approaches 1 |

### Big-O

$$
T(n)=O(n^2)
$$

formally means that there exist constants $c>0$ and $n_0$ such that:

$$
T(n)\le cn^2
$$

for every:

$$
n\ge n_0
$$

Big-O is an upper bound, not automatically an exact bound.

### Theta

$$
T(n)=\Theta(n^2)
$$

means both:

$$
T(n)=O(n^2)
$$

and:

$$
T(n)=\Omega(n^2)
$$

---

## 24. Probability notation

| Symbol | Meaning |
|---|---|
| $\Pr(A)$ or $P(A)$ | Probability of event $A$ |
| $P(A\mid B)$ | Probability of $A$, given $B$ |
| $A\perp B$ | $A$ and $B$ are independent |
| $\mathbb E[X]$ | Expected value of $X$ |
| $\operatorname{Var}(X)$ | Variance of $X$ |
| $\sigma(X)$ | Standard deviation |
| $X\sim D$ | $X$ follows distribution $D$ |
| $X_1,\ldots,X_n\overset{\text{iid}}{\sim}D$ | Independent, identically distributed samples |

### Conditional probability

$$
P(A\mid B)
=
\frac{P(A\cap B)}{P(B)}
$$

Read:

> Probability of $A$, knowing that $B$ occurred.

### Expected value

For a discrete random variable:

$$
\mathbb E[X]
=
\sum_x xP(X=x)
$$

This is a probability-weighted average, not necessarily a value that actually occurs.

---

