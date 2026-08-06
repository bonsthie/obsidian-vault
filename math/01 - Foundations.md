---
title: "Mathematical Notation — Foundations"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Foundations

> [!info] Dictionary part 1 of 8 · [[math/index|Index]] · Next: [[math/02 - Operators and Values|Operators and Values]]

## How to read an unfamiliar formula

Read a formula in this order:

1. Find the **main operator**: $=$, $\le$, $\min$, $\sum$, $\Rightarrow$, etc.
2. Identify every variable and its domain.
3. Read subscripts and superscripts.
4. Read constraints written below operators.
5. Translate the formula into one plain-English sentence.

---

## 1. Equality and comparison

| Symbol | Name | Meaning |
|---|---|---|
| $a=b$ | Equality | $a$ and $b$ have the same value |
| $a\neq b$ | Not equal | $a$ and $b$ differ |
| $a:=b$ | Definition | Define $a$ to be $b$ |
| $a\equiv b$ | Equivalence / identity | Equal by definition, always equal, or equivalent under a relation |
| $a\approx b$ | Approximately equal | Values are close but not necessarily identical |
| $a\sim b$ | Similar / asymptotically equivalent | Meaning depends on context |
| $a<b$ | Strictly smaller | $a$ is smaller than $b$ |
| $a\le b$ | Smaller or equal | $a$ is at most $b$ |
| $a>b$ | Strictly greater | $a$ is greater than $b$ |
| $a\ge b$ | Greater or equal | $a$ is at least $b$ |
| $a\propto b$ | Proportional | $a=cb$ for some constant $c$ |

### Common wording

$$
x\le 10
$$

Read:

> $x$ is at most 10.

$$
x\ge 10
$$

Read:

> $x$ is at least 10.

$$
a:=b+1
$$

Read:

> Define $a$ as $b+1$.

The distinction between $=$ and $:=$ is useful:

$$
x=5
$$

asserts that $x$ has value 5.

$$
x:=5
$$

introduces or defines $x$ to have value 5.

---

## 2. Number sets

| Symbol | Name | Typical content |
|---|---|---|
| $\mathbb N$ | Natural numbers | $0,1,2,\ldots$, or sometimes $1,2,\ldots$ |
| $\mathbb N_0$ | Nonnegative integers | $0,1,2,\ldots$ |
| $\mathbb Z$ | Integers | $\ldots,-2,-1,0,1,2,\ldots$ |
| $\mathbb Q$ | Rational numbers | Fractions $a/b$ |
| $\mathbb R$ | Real numbers | All ordinary continuous numbers |
| $\mathbb R_{\ge0}$ | Nonnegative reals | Real numbers $x\ge0$ |
| $\mathbb C$ | Complex numbers | Numbers $a+bi$ |
| $\{0,1\}$ | Boolean/binary set | False/true or off/on |

> [!warning]
> Some authors include zero in $\mathbb N$ and some do not. Check the author’s convention.

---

## 3. Set notation

| Symbol | Meaning |
|---|---|
| $x\in S$ | $x$ belongs to set $S$ |
| $x\notin S$ | $x$ does not belong to $S$ |
| $A\subseteq B$ | Every element of $A$ belongs to $B$ |
| $A\subsetneq B$ | $A$ is a strict subset of $B$ |
| $\varnothing$ | Empty set |
| $\{a,b,c\}$ | Set containing $a,b,c$ |
| $A\cup B$ | Union: elements in $A$ or $B$ |
| $A\cap B$ | Intersection: elements in both |
| $A\setminus B$ | Elements in $A$ but not in $B$ |
| $A\times B$ | Cartesian product |
| $\lvert A\rvert$ | Number of elements in $A$ |
| $\mathcal P(A)$ or $2^A$ | Set of all subsets of $A$ |

### Set-builder notation

$$
\{x\in S:P(x)\}
$$

Read:

> The set of all $x$ belonging to $S$ such that $P(x)$ is true.

The colon means **such that**. A vertical bar is also common:

$$
\{x\in S\mid P(x)\}
$$

Example:

$$
\{x\in\mathbb N:x<5\}
=
\{0,1,2,3,4\}
$$

assuming zero belongs to $\mathbb N$.

Example from optimization:

$$
T=
\min\left\{
t\in\mathbb N:
C(t)\ge N
\right\}
$$

Read:

> $T$ is the smallest natural number $t$ such that the capacity at time $t$ is at least $N$.

---

## 4. Logical symbols

| Symbol | Name | Meaning |
|---|---|---|
| $\neg P$ | Negation | $P$ is false |
| $P\land Q$ | Conjunction | $P$ and $Q$ |
| $P\lor Q$ | Disjunction | $P$ or $Q$, possibly both |
| $P\oplus Q$ | Exclusive or | Exactly one of $P,Q$ |
| $P\Rightarrow Q$ | Implication | If $P$, then $Q$ |
| $P\Leftarrow Q$ | Reverse implication | $Q$ implies $P$ |
| $P\Leftrightarrow Q$ | Equivalence | $P$ if and only if $Q$ |
| $\forall x$ | Universal quantifier | For every $x$ |
| $\exists x$ | Existential quantifier | There exists an $x$ |
| $\exists!x$ | Unique existence | There exists exactly one $x$ |
| $\therefore$ | Therefore | The conclusion follows |
| $\because$ | Because | Introduces a reason |

### Implication

$$
P\Rightarrow Q
$$

means:

> Whenever $P$ is true, $Q$ must also be true.

It does **not** automatically mean:

$$
Q\Rightarrow P
$$

Example:

$$
x>5\Rightarrow x>0
$$

is true, but:

$$
x>0\Rightarrow x>5
$$

is false.

### Equivalence

$$
P\Leftrightarrow Q
$$

means both:

$$
P\Rightarrow Q
$$

and:

$$
Q\Rightarrow P
$$

Read:

> $P$ if and only if $Q$.

Often abbreviated **iff**.

---

## 5. Quantifiers

### Universal statement

$$
\forall x\in S,\quad P(x)
$$

Read:

> For every $x$ in $S$, property $P(x)$ holds.

Example:

$$
\forall i\in\{1,\ldots,k\},\quad d_i\ge1
$$

Read:

> Every path length $d_i$ is at least 1.

### Existential statement

$$
\exists x\in S,\quad P(x)
$$

Read:

> There is at least one $x$ in $S$ for which $P(x)$ holds.

Example:

$$
\exists P\text{ from }s\text{ to }t
$$

Read:

> At least one path exists from $s$ to $t$.

### Quantifier order matters

These statements are different:

$$
\forall x,\exists y,\quad P(x,y)
$$

> Every $x$ has some possibly different $y$.

$$
\exists y,\forall x,\quad P(x,y)
$$

> There is one single $y$ that works for every $x$.

Example:

$$
\forall n\in\mathbb N,\exists m\in\mathbb N,\quad m>n
$$

is true.

But:

$$
\exists m\in\mathbb N,\forall n\in\mathbb N,\quad m>n
$$

is false.

---

## 6. Index notation

### Subscript

$$
x_i
$$

usually means:

> Element $i$ of a sequence, array, vector, or family.

Examples:

$$
d_i=\text{length of path }i
$$

$$
x_i=\text{number of ants assigned to path }i
$$

A double subscript:

$$
a_{ij}
$$

often means:

> Entry at row $i$, column $j$ of a matrix.

### Superscript

$$
x^2
$$

normally means exponentiation.

But:

$$
x^{(k)}
$$

often means the value of $x$ at iteration $k$, not $x$ raised to $k$.

Examples:

$$
x^3=x\cdot x\cdot x
$$

but:

$$
x^{(3)}
$$

may mean “the third version of $x$.”

### Range notation

$$
i=1,\ldots,k
$$

means:

$$
i\in\{1,2,\ldots,k\}
$$

---

