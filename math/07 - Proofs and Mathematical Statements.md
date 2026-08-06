---
title: "Mathematical Notation — Proofs and Mathematical Statements"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Proofs and Mathematical Statements

> [!info] Dictionary part 7 of 8 · Previous: [[math/06 - Algorithms Languages and Types|Algorithms, Languages, and Types]] · [[math/index|Index]] · Next: [[math/08 - Formula Reading Reference|Formula Reading Reference]]

## 28. Proof vocabulary

| Term | Meaning |
|---|---|
| Definition | Introduces the exact meaning of an object |
| Axiom | Assumed truth |
| Proposition | Mathematical claim |
| Theorem | Important claim proved from definitions/results |
| Lemma | Supporting theorem used in a larger proof |
| Corollary | Result following quickly from another theorem |
| Conjecture | Claim believed true but not proven |
| Counterexample | Example proving a universal claim false |
| Invariant | Property remaining true throughout an algorithm |
| Necessary condition | Must be true, but may not be sufficient |
| Sufficient condition | Guarantees the result, but may not be necessary |
| Necessary and sufficient | Exact condition; an iff statement |
| Upper bound | Value the result cannot exceed |
| Lower bound | Value the result cannot go below |
| Tight bound | Matching upper and lower bounds |
| Contradiction | Logical impossibility used to reject an assumption |
| WLOG | Without loss of generality |
| QED / $\square$ | End of proof |

---

## 29. Common proof styles

### Direct proof

To prove:

$$
P\Rightarrow Q
$$

assume $P$, then derive $Q$.

### Proof by contradiction

To prove $P$:

1. assume $\neg P$;
2. derive an impossibility;
3. conclude that $P$ must hold.

Common wording:

> Suppose, for contradiction, that…

### Proof by induction

Used for statements indexed by integers:

1. **Base case:** prove $P(0)$ or $P(1)$.
2. **Induction hypothesis:** assume $P(k)$.
3. **Inductive step:** prove $P(k+1)$.
4. Conclude $P(n)$ for all relevant $n$.

### Proof of equivalence

To prove:

$$
P\Leftrightarrow Q
$$

prove both:

$$
P\Rightarrow Q
$$

and:

$$
Q\Rightarrow P
$$

### Counterexample

To disprove:

$$
\forall x,\quad P(x)
$$

find one $x$ such that:

$$
\neg P(x)
$$

---

## 30. Necessary versus sufficient

Suppose:

$$
P\Rightarrow Q
$$

Then:

- $P$ is **sufficient** for $Q$;
- $Q$ is **necessary** for $P$.

Example:

$$
x>10\Rightarrow x>0
$$

- $x>10$ is sufficient to know $x>0$;
- $x>0$ is necessary for $x>10$;
- $x>0$ is not sufficient for $x>10$.

An exact characterization uses:

$$
P\Leftrightarrow Q
$$

Then both conditions are necessary and sufficient.

---

## 31. Common equation styles

### Definition

$$
C(t):=
\sum_i\max(0,t-d_i+1)
$$

> Define $C(t)$ as the sum.

### Constraint

$$
\sum_i x_i=N
$$

> Every valid solution must satisfy this equality.

### Inequality bound

$$
T(n)\le cn^2
$$

> Running time is bounded above by $cn^2$.

### Optimization

$$
\min_{x\in S}f(x)
$$

> Find the smallest objective value over valid choices.

### Recurrence

$$
T(n)=2T(n/2)+n
$$

> Define the current value using smaller instances.

### Set characterization

$$
S=\{x\in\mathbb N:x^2<10\}
$$

> Define a set through a condition.

### Case distinction

$$
f(x)=
\begin{cases}
-x,&x<0,\\
x,&x\ge0.
\end{cases}
$$

> Use a different expression depending on the condition.

### Chain of relations

$$
a=b\le c<d
$$

means:

$$
a=b,\qquad b\le c,\qquad c<d
$$

---

## 32. Punctuation inside formulas

| Symbol | Meaning in context |
|---|---|
| $,$ | Separates expressions or means “and then” |
| $:$ | “Such that” or introduces a type |
| $\mid$ | “Such that,” divides, or conditional probability |
| $;$ | Separates conditions |
| $\quad$ | Visual spacing only |
| $\ldots$ | Pattern continues |
| $\underbrace{\cdots}_{\text{text}}$ | Labels part of an expression |
| $\overbrace{\cdots}^{\text{text}}$ | Labels part from above |

The vertical bar is highly overloaded:

$$
x\mid y
$$

means $x$ divides $y$.

$$
P(A\mid B)
$$

means probability of $A$ given $B$.

$$
\{x\mid P(x)\}
$$

means all $x$ such that $P(x)$.

---

