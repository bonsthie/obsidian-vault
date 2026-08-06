---
title: Math
aliases:
  - CS Mathematical Notation
  - Proof Notation Dictionary
tags:
  - mathematics
  - computer-science
  - proofs
  - algorithms
  - latex
  - reference
---

# Math — Visual Index

> [!tip] Find something that looks like your formula, then open the link beside it.

## Basic symbols

| I see something like… | Open |
|---|---|
| $a=b$, $a\neq b$, $x\le 10$, $a:=b$ | [[math/01-basics#1. Equality and comparison\|Equality and comparison]] |
| $\mathbb N$, $\mathbb Z$, $\mathbb R$, $\mathbb C$ | [[math/01-basics#2. Number sets\|Number sets]] |
| $x\in S$, $A\cup B$, $A\subseteq B$, $\{x\in S:P(x)\}$ | [[math/01-basics#3. Set notation\|Sets]] |
| $\neg P$, $P\land Q$, $P\Rightarrow Q$, $P\iff Q$ | [[math/01-basics#4. Logical symbols\|Logic]] |
| $\forall x$, $\exists y$ | [[math/01-basics#5. Quantifiers\|Quantifiers]] |
| $x_i$, $a^n$, $i=1,\ldots,n$ | [[math/01-basics#6. Index notation\|Indices]] |

## Operators and values

| I see something like… | Open |
|---|---|
| $\sum_{i=1}^n a_i$, $\prod_{i=1}^n a_i$ | [[math/02-operators#7. Sums and products\|Sums and products]] |
| $\min S$, $\max_i x_i$ | [[math/02-operators#8. Minimum and maximum\|Minimum and maximum]] |
| $\min_x f(x)$ versus $\operatorname*{argmin}_x f(x)$ | [[math/02-operators#9. min versus argmin\|min versus argmin]] |
| $\lceil x\rceil$, $\lfloor x\rfloor$ | [[math/02-operators#10. Ceiling and floor\|Ceiling and floor]] |
| $\lvert x\rvert$, $\lvert S\rvert$, $\lVert v\rVert$ | [[math/02-operators#11. Absolute value, cardinality, and norm\|Bars and norms]] |
| $(a,b)$, $[a,b]$, $A[i]$, $f(x)$ | [[math/02-operators#12. Parentheses and brackets\|Brackets and intervals]] |

## Functions and optimization

| I see something like… | Open |
|---|---|
| $f:A\to B$, $x\mapsto f(x)$, $g\circ f$ | [[math/03-functions#13. Functions\|Functions]] |
| $f(x)=\begin{cases}\cdots\end{cases}$ | [[math/03-functions#14. Piecewise definitions\|Piecewise definitions]] |
| $\mathbf 1_{P(x)}$, $[P(x)]$ | [[math/03-functions#15. Indicator functions\|Indicator functions]] |
| $\min_x f(x)\quad\text{s.t. }g(x)\le0$ | [[math/03-functions#16. Common optimization form\|Optimization form]] |
| $\min\;\max\;\sum$ in one large formula | [[math/03-functions#17. Reading a Lem-in optimization formula\|Worked optimization formula]] |
| $T(\mathcal P)$ and path capacities | [[math/03-functions#18. Reading the capacity formula\|Capacity formula]] |

## Algebra and computer science

| I see something like… | Open |
|---|---|
| $\frac ab$, $a/b$, a ratio | [[math/04-algebra#19. Fractions\|Fractions]] |
| $a=b+c\Rightarrow a-c=b$ | [[math/04-algebra#20. Common algebra transformations\|Algebra transformations]] |
| $G=(V,E)$, $u\sim v$, $\deg(v)$, paths | [[math/04-algebra#21. Graph-theory notation\|Graph theory]] |
| $(a_n)_{n\ge0}$, $a_n=a_{n-1}+a_{n-2}$ | [[math/05-sequences#22. Sequence and recurrence notation\|Sequences and recurrences]] |
| $O(n)$, $O(n\log n)$, $\Theta(n^2)$ | [[math/05-sequences#23. Asymptotic complexity\|Complexity]] |
| $P(A)$, $P(A\mid B)$, $\mathbb E[X]$, $\operatorname{Var}(X)$ | [[math/05-sequences#24. Probability notation\|Probability]] |
| $x\leftarrow5$, `for`, `while`, `return` | [[math/06-cs#25. Common symbols used in algorithms\|Algorithms]] |
| $\Sigma^*$, $\varepsilon$, $L_1\cup L_2$ | [[math/06-cs#26. Formal-language notation\|Formal languages]] |
| $\Gamma\vdash e:\tau$, $e\Downarrow v$, $e[x:=v]$ | [[math/06-cs#27. Type-theory and programming-language notation\|Types and programming languages]] |

## Proofs and statements

| I see something like… | Open |
|---|---|
| “theorem”, “lemma”, “corollary”, “QED” | [[math/07-proofs#28. Proof vocabulary\|Proof vocabulary]] |
| contradiction, induction, counterexample | [[math/07-proofs#29. Common proof styles\|Proof styles]] |
| $P\Rightarrow Q$, “if”, “only if”, “iff” | [[math/07-proofs#30. Necessary versus sufficient\|Necessary and sufficient]] |
| $:=$, $\le$, $\min$, recurrence, cases | [[math/07-proofs#31. Common equation styles\|Equation styles]] |
| $:$, $;$, comma, $\mid$ inside a formula | [[math/07-proofs#32. Punctuation inside formulas\|Formula punctuation]] |

## Help me read or write it

| I need… | Open |
|---|---|
| $\alpha$, $\beta$, $\lambda$, $\mu$, $\sigma$, $\theta$ | [[math/08-reference#33. Greek-letter quick reference\|Greek letters]] |
| A step-by-step way to read a formula | [[math/08-reference#34. Reading checklist\|Reading checklist]] |
| To watch one formula being decoded | [[math/08-reference#35. Compact decoding example\|Decoding example]] |
| `\sum`, `\frac`, `\begin{aligned}` | [[math/08-reference#36. Quick LaTeX cheat sheet\|LaTeX cheat sheet]] |
| A template for analyzing a formula | [[math/08-reference#37. Personal formula-analysis template\|Analysis template]] |
| The shortest summary of everything | [[math/08-reference#38. One-page mental model\|One-page mental model]] |
