---
title: "Mathematical Notation — Formula Reading Reference"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Formula Reading Reference

> [!info] Dictionary part 8 of 8 · Previous: [[math/07 - Proofs and Mathematical Statements|Proofs and Mathematical Statements]] · [[math/index|Index]]

## 33. Greek-letter quick reference

| Letter | Common CS uses |
|---|---|
| $\alpha,\beta$ | Parameters, type variables |
| $\gamma,\Gamma$ | Paths, environments, distributions |
| $\delta,\Delta$ | Change, transition function, difference |
| $\varepsilon$ | Empty string, error tolerance |
| $\theta,\Theta$ | Angle, parameter, tight asymptotic bound |
| $\lambda$ | Anonymous function, eigenvalue, rate |
| $\mu$ | Mean |
| $\pi$ | Probability distribution, predecessor |
| $\rho$ | Ratio, correlation |
| $\sigma,\Sigma$ | Standard deviation, alphabet |
| $\tau$ | Type, time/deadline |
| $\phi,\Phi$ | Formula, potential function |
| $\omega,\Omega$ | Frequency, lower asymptotic bound |

There is no universal rule saying what each letter means. Definitions take priority.

---

## 34. Reading checklist

When you encounter an unfamiliar formula, ask:

### 1. What is being defined or claimed?

Look to the left of:

$$
=,\quad :=,\quad \le,\quad \Rightarrow
$$

### 2. What are the variables?

Find statements like:

$$
x\in\mathbb N,\qquad P\in\mathcal P,\qquad G=(V,E)
$$

### 3. What are the constraints?

Look below:

$$
\min,\quad\max,\quad\sum,\quad\prod
$$

### 4. What does each index range over?

Example:

$$
\sum_{i=1}^{k}
$$

### 5. What is the outermost operation?

For:

$$
\min_x\max_i f(x,i)
$$

first compute the maximum for each $x$, then minimize those results.

Order matters:

$$
\min_x\max_i f(x,i)
$$

is generally not equal to:

$$
\max_i\min_x f(x,i)
$$

### 6. Is this an exact equality or a bound?

Compare:

$$
T(n)=n^2
$$

with:

$$
T(n)\le n^2
$$

and:

$$
T(n)=O(n^2)
$$

### 7. Is anything rounded?

Look for:

$$
\lfloor\cdot\rfloor,\qquad
\lceil\cdot\rceil
$$

### 8. Is a symbol overloaded?

Determine whether $|x|$ means:

- absolute value;
- set size;
- string length;
- determinant.

---

## 35. Compact decoding example

Take:

$$
T_k=
\min
\left\{
\tau\in\mathbb N:
\sum_{i=1}^{k}
\max(0,\tau-d_i+1)
\ge N
\right\}
$$

Decode from the inside outward:

1. $\tau-d_i+1$: capacity of path $i$ before deadline $\tau$.
2. $\max(0,\tau-d_i+1)$: prevent negative capacity.
3. $\sum_{i=1}^{k}$: add capacities of all $k$ paths.
4. $\ge N$: require enough capacity for all ants.
5. $\{\tau\in\mathbb N:\cdots\}$: collect every valid integer deadline.
6. $\min$: select the earliest valid deadline.
7. $T_k=$: call that result $T_k$.

Plain English:

> $T_k$ is the earliest integer turn at which the $k$ paths can collectively deliver all $N$ ants.

---

## 36. Quick LaTeX cheat sheet

| Desired symbol | LaTeX |
|---|---|
| $\forall$ | `\forall` |
| $\exists$ | `\exists` |
| $\in$ | `\in` |
| $\notin$ | `\notin` |
| $\subseteq$ | `\subseteq` |
| $\cup$ | `\cup` |
| $\cap$ | `\cap` |
| $\varnothing$ | `\varnothing` |
| $\Rightarrow$ | `\Rightarrow` |
| $\Leftrightarrow$ | `\Leftrightarrow` |
| $\le$ | `\le` |
| $\ge$ | `\ge` |
| $\neq$ | `\neq` |
| $\sum$ | `\sum` |
| $\prod$ | `\prod` |
| $\min$ | `\min` |
| $\max$ | `\max` |
| $\arg\min$ | `\arg\min` |
| $\lfloor x\rfloor$ | `\lfloor x\rfloor` |
| $\lceil x\rceil$ | `\lceil x\rceil` |
| $\mathbb N$ | `\mathbb N` |
| $\mathbb Z$ | `\mathbb Z` |
| $\mathbb R$ | `\mathbb R` |
| $\Theta$ | `\Theta` |
| $\Omega$ | `\Omega` |
| $\lambda$ | `\lambda` |
| $\varepsilon$ | `\varepsilon` |
| $\infty$ | `\infty` |
| $\square$ | `\square` |

### Inline formula

```markdown
The running time is $O(n \log n)$.
```

### Display formula

```markdown
$$
T(n)=2T(n/2)+O(n)
$$
```

### Multi-line derivation

```markdown
$$
\begin{aligned}
A_{k+1}
&=
\frac{N+D_{k+1}-(k+1)}{k+1}\\
&=
\frac{N+D_k+\Delta_{k+1}-(k+1)}{k+1}
\end{aligned}
$$
```

---

## 37. Personal formula-analysis template

Copy this under any equation in your Obsidian notes:

```markdown
### Formula analysis

$$
% Paste the formula here
$$

#### Variables

- $x$:
- $n$:
- $i$:

#### Domains

- $x\in$:
- $n\in$:
- $i\in$:

#### Main operator

- Operator:
- Meaning:

#### Constraints

- Constraint 1:
- Constraint 2:

#### Read from the inside outward

1.
2.
3.
4.

#### Plain-English translation

> 

#### Why it is useful

-

#### Small numerical example

-
```

---

## 38. One-page mental model

> [!tip]
> When reading a formula, use this sequence:
>
> **Objects → domains → constraints → inner operations → outer operation → conclusion**

Example:

$$
\min_{\substack{x_i\in\mathbb N_0\\\sum_i x_i=N}}
\max_i(d_i+x_i-1)
$$

- **Objects:** $x_i,d_i,N$
- **Domains:** $x_i\in\mathbb N_0$
- **Constraint:** $\sum_i x_i=N$
- **Inner operation:** calculate $d_i+x_i-1$
- **Next operation:** take the largest value with $\max_i$
- **Outer operation:** choose the allocation minimizing that maximum
- **Conclusion:** best possible finishing time

