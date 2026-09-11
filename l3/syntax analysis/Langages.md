# AS — Languages

## Empty word

- $\varepsilon$ = word of length $0$
- $\{\varepsilon\}$ = language containing only the empty word
- $\varnothing$ = empty language

Important: $\varnothing \neq \{\varepsilon\}$

---

## Number of words

For alphabet $\Sigma=\{a,b,c\}$:

$|\Sigma|=3$

Number of words of length $3$: $3^3=27$

In general, if $|\Sigma|=n$:

$\text{words of length }k=n^k$

---

## Set operators

Let $A=\{a,b\}$ and $B=\{b,c\}$.

- Union: $A\cup B=\{a,b,c\}$
- Intersection: $A\cap B=\{b\}$
- Difference: $A-B=\{a\}$
- Complement: $\overline{A}=\Sigma-A$
- Right quotient: $L_1/L_2=\{u\mid \exists v\in L_2,\ uv\in L_1\}$
  - Example: $\{abc\}/\{c\}=\{ab\}$
- Left quotient: $L_2\backslash L_1=\{v\mid \exists u\in L_2,\ uv\in L_1\}$
  - Example: $\{a\}\backslash\{abc\}=\{bc\}$

---

## Concatenation

$u\cdot v=uv$

Example:

$\{a,b\}\cdot\{c,d\}=\{ac,ad,bc,bd\}$

With the empty language:

$\varnothing\cdot L=L\cdot\varnothing=\varnothing$

---

## Power of a language

$L^0=\{\varepsilon\}$

$L^{i+1}=L^i\cdot L$

Example: $L^2=L\cdot L$

---

## Kleene star

$L^*=\{\varepsilon\}\cup L\cup L^2\cup L^3\cup\cdots$

= words obtained with $0,1,2,\ldots$ concatenations of words from $L$.

Example:

$\varnothing^*=\{\varepsilon\}$

because $\varnothing^0=\{\varepsilon\}$ and $\varnothing^i=\varnothing$ for $i>0$.

---

## Arden's Lemma

Equation: $X=AX+B$

Smallest solution: $X=A^*B$

If $\varepsilon\notin A$, then $A^*B$ is the unique solution.




They are different operations in formal languages.

For languages $L_1,L_2$:

### Right quotient — $L_1 / L_2$

$L_1/L_2$ = remove a word of $L_2$ from the **right** of a word in $L_1$.

$L_1/L_2={u\mid \exists v\in L_2,\ uv\in L_1}$

Example:

$L_1={abc}$, $L_2={c}$

Then:

$L_1/L_2={ab}$

because $ab\cdot c=abc$.

---

### Left quotient — $L_2\backslash L_1$

$L_2\backslash L_1$ = remove a word of $L_2$ from the **left** of a word in $L_1$.

$L_2\backslash L_1={v\mid \exists u\in L_2,\ uv\in L_1}$

Example:

$L_1={abc}$, $L_2={a}$

Then:

$L_2\backslash L_1={bc}$

because $a\cdot bc=abc$.

So the ultra-short memory trick is:

$L_1/L_2$ → remove $L_2$ from the **right**

$L_2\backslash L_1$ → remove $L_2$ from the **left**

Be careful: $L_1\setminus L_2$ can also mean **set difference** ("words in $L_1$ but not in $L_2$"). The symbol is usually written `\setminus`, not just `\backslash`.