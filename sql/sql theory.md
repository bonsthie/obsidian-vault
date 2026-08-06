# Big O Notation: $\mathcal{O}(f)$

**The Goal:** Evaluate the algorithm's growth in the worst-case scenario. As the input gets massive, smaller terms stop mattering, so we evaluate the "asymptotic growth" and discard them.

- **Correct Notation:** $\mathcal{O}(N^2 + 2N + 100) = \mathcal{O}(N^2)$
## The Mathematical Proof

To officially prove we are allowed to drop those small terms and categorize the algorithm, it must pass this test:
$$|c(n,p,\dots)| < \alpha \times f(n,p,\dots)$$
### The Variables

- **$n, p, \dots$** $\rightarrow$ Input size (ex: number of rows/columns).
- **$c(n,p,\dots)$** $\rightarrow$ The messy, **precise actual cost** of the function.
- **$f(n,p,\dots)$** $\rightarrow$ The clean **reference function** representing the shape of the growth.
- **$\alpha$** $\rightarrow$ A positive constant number used to scale up $f$.

### The Meaning

The actual cost of your algorithm ($c$) is **strictly less than** a scaled-up "ceiling" of your reference function ($\alpha \times f$) _for all sufficiently large values of $n$_.

---

## Concrete Example

Let's prove an algorithm belongs in the $\mathcal{O}(n^2)$ class:

- **$c$ (Actual cost):** $3n^2 + 50n + 100$
- **$f$ (Reference):** $n^2$
- **$\alpha$ (Constant):** $4$

**The Equation:**

$$3n^2 + 50n + 100 < 4 \times n^2$$

**Conclusion:** This equation isn't true for small numbers (like $n = 1$), but it **is** true for any $n > 51$. Because the actual cost eventually gets trapped strictly under $4n^2$ as the data gets huge, we have successfully proven the algorithm is $\mathcal{O}(n^2)$.

---
# DB slang

- **Attribute:** A **column** in the table (e.g., Name, Age, Price).
- **Arity (or Degree):** The **total number of columns** in the table.
- **Relation of arity _k_:** A table that has exactly **_k_ columns**.
- **Record (or Tuple):** A **row** in the table

- vecteur des attribute de $\langle\alpha_0,\alpha_1,\dots,\alpha_{k-1}\rangle$ d'une table T is `attributs(T)` 
- table T is $\![[ \alpha_0,\alpha_1,\dots,\alpha_{k-1} \!]]$  



