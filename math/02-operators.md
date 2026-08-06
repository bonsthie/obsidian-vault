---
title: Operators
tags:
  - mathematics
  - computer-science
  - reference
---

# Operators

> [!info] Part 2 of 8 · Previous: [[math/01-basics|Basics]] · [[math/index|Index]] · Next: [[math/03-functions|Functions]]

## 7. Sums and products

### Summation

$$
\sum_{i=1}^{k}x_i
$$

means:

$$
x_1+x_2+\cdots+x_k
$$

The bottom gives the initial index:

$$
i=1
$$

The top gives the final index:

$$
k
$$

Example:

$$
\sum_{i=1}^{3}x_i=x_1+x_2+x_3
$$

Constraint example:

$$
\sum_{i=1}^{k}x_i=N
$$

Read:

> The ants assigned to all $k$ paths add up to $N$.

### Product

$$
\prod_{i=1}^{k}x_i
$$

means:

$$
x_1x_2\cdots x_k
$$

Example:

$$
\prod_{i=1}^{4}i=1\cdot2\cdot3\cdot4=24
$$

---

## 8. Minimum and maximum

### Minimum

$$
\min(a,b,c)
$$

returns the smallest value.

$$
\min(4,9,2)=2
$$

### Maximum

$$
\max(a,b,c)
$$

returns the largest value.

$$
\max(4,9,2)=9
$$

### Minimum over a domain

$$
\min_{x\in S}f(x)
$$

Read:

> The smallest value of $f(x)$ among all valid $x\in S$.

Example:

$$
\min_{x\in\mathbb N,\;x\ge5}x=5
$$

### Constraints under an operator

$$
\min_{\substack{x_i\in\mathbb N_0\\
\sum_i x_i=N}}
F(x_1,\ldots,x_k)
$$

Read:

> Minimize $F$, considering only nonnegative integer values $x_i$ whose sum is $N$.

> [!important]
> The material below $\min$ is not being minimized. It defines the allowed choices.

---

## 9. `min` versus `argmin`

These are different.

### Minimum value

$$
\min_{x\in S}f(x)
$$

returns the smallest **value** of $f$.

### Argument attaining the minimum

$$
\operatorname{argmin}_{x\in S}f(x)
$$

returns the value of $x$ that produces the minimum.

Example:

$$
f(x)=(x-3)^2
$$

Then:

$$
\min_{x\in\mathbb R}f(x)=0
$$

while:

$$
\operatorname{argmin}_{x\in\mathbb R}f(x)=3
$$

So:

- `min` answers: **What is the best score?**
- `argmin` answers: **Which solution gives the best score?**

Example:

$$
T^*=\min_{\mathcal P}T(\mathcal P)
$$

is the optimal number of turns.

$$
\mathcal P^*
=
\operatorname{argmin}_{\mathcal P}T(\mathcal P)
$$

is the best path family.

The star often means **optimal**.

---

## 10. Ceiling and floor

| Symbol | Name | Meaning |
|---|---|---|
| $\lfloor x\rfloor$ | Floor | Greatest integer not larger than $x$ |
| $\lceil x\rceil$ | Ceiling | Smallest integer not smaller than $x$ |

Examples:

$$
\lfloor4.8\rfloor=4
$$

$$
\lceil4.2\rceil=5
$$

In scheduling, ceiling appears when a fractional result must become a whole number of turns:

$$
T=
\left\lceil
\frac{N+D_k-k}{k}
\right\rceil
$$

If the fraction is $5.2$, the solution needs 6 turns:

$$
\lceil5.2\rceil=6
$$

---

## 11. Absolute value, cardinality, and norm

The same vertical bars can mean different things.

### Absolute value

For a number:

$$
|x|
$$

means its distance from zero.

$$
|-5|=5
$$

### Set cardinality

For a set:

$$
|S|
$$

means the number of elements in $S$.

If:

$$
S=\{a,b,c\}
$$

then:

$$
|S|=3
$$

### Vector norm

For a vector:

$$
\|v\|
$$

means its length or magnitude.

For a two-dimensional Euclidean vector:

$$
v=(x,y)
$$

$$
\|v\|_2=\sqrt{x^2+y^2}
$$

---

## 12. Parentheses and brackets

| Notation | Common purpose |
|---|---|
| $(a+b)$ | Grouping |
| $f(x)$ | Function application |
| $[a,b]$ | Closed interval |
| $(a,b)$ | Open interval |
| $[a,b)$ | Includes $a$, excludes $b$ |
| $\{a,b\}$ | Set |
| $\langle u,v\rangle$ | Tuple or inner product |

### Intervals

$$
x\in[2,5]
$$

means:

$$
2\le x\le5
$$

$$
x\in(2,5)
$$

means:

$$
2<x<5
$$

$$
x\in[2,5)
$$

means:

$$
2\le x<5
$$

---
