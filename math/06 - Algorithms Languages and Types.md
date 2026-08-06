---
title: "Mathematical Notation — Algorithms, Languages, and Types"
tags:
  - mathematics
  - computer-science
  - reference
---

# Mathematical Notation — Algorithms, Languages, and Types

> [!info] Dictionary part 6 of 8 · Previous: [[math/05 - Sequences Complexity and Probability|Sequences, Complexity, and Probability]] · [[math/index|Index]] · Next: [[math/07 - Proofs and Mathematical Statements|Proofs and Mathematical Statements]]

## 25. Common symbols used in algorithms

| Symbol | Common meaning |
|---|---|
| $n$ | Input size |
| $k$ | Count, parameter, or iteration number |
| $i,j$ | Indices |
| $T(n)$ | Running time |
| $S(n)$ | Space consumption |
| $G=(V,E)$ | Graph |
| $w(e)$ | Weight of edge $e$ |
| $d(v)$ | Distance to vertex $v$ |
| $\pi(v)$ | Parent/predecessor of $v$ |
| $\infty$ | Infinity or unreachable initial distance |
| $\bot$ | Undefined, false, failure, or bottom value |
| $\top$ | True or top value |
| $\varepsilon$ | Empty string or a small positive value |
| $\lambda$ | Parameter, eigenvalue, or anonymous function |
| $\delta$ | Difference, transition function, or small change |
| $\Delta$ | Larger change, discriminant, or maximum degree |
| $\mu$ | Mean or parameter |
| $\sigma$ | Standard deviation or alphabet |
| $\Sigma$ | Alphabet or named set |

> [!note]
> Symbols are overloaded. Their meaning comes from their definition in the paper.

---

## 26. Formal-language notation

| Notation | Meaning |
|---|---|
| $\Sigma$ | Alphabet |
| $\varepsilon$ | Empty string |
| $\Sigma^*$ | All finite strings over $\Sigma$ |
| $\Sigma^+$ | All nonempty finite strings |
| $|w|$ | Length of string $w$ |
| $uv$ | Concatenation of strings |
| $L\subseteq\Sigma^*$ | Language over alphabet $\Sigma$ |
| $w\in L$ | String $w$ belongs to language $L$ |
| $L^*$ | Kleene closure |
| $L_1L_2$ | Concatenation of two languages |
| $L_1\cup L_2$ | Union of languages |

Example:

$$
\Sigma=\{a,b\}
$$

$$
\Sigma^*=
\{\varepsilon,a,b,aa,ab,ba,bb,\ldots\}
$$

---

## 27. Type-theory and programming-language notation

| Notation | Meaning |
|---|---|
| $e:\tau$ | Expression $e$ has type $\tau$ |
| $\Gamma\vdash e:\tau$ | Under environment $\Gamma$, $e$ has type $\tau$ |
| $\Gamma,x:\tau$ | Environment extended with variable $x$ of type $\tau$ |
| $e\to e'$ | $e$ reduces/evaluates one step to $e'$ |
| $e\to^*e'$ | Zero or more evaluation steps |
| $\lambda x.e$ | Anonymous function taking $x$ and returning $e$ |
| $\tau_1\to\tau_2$ | Function type |
| $\tau_1\times\tau_2$ | Product/pair type |
| $\forall\alpha.\tau$ | Polymorphic type |

### Inference rule

$$
\frac{
\Gamma\vdash e_1:\tau_1
\qquad
\Gamma\vdash e_2:\tau_2
}{
\Gamma\vdash (e_1,e_2):\tau_1\times\tau_2
}
$$

Read:

> If the statements above the line are true, then the statement below the line follows.

- top: **premises**
- bottom: **conclusion**

---

