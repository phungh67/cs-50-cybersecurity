## Part 1
A company published an average salary of all employees. Person p1 joins, company re-publishes again
a. Is it possible to figure out the actual salary of p1? Proof?
b. Given 2 means: $u_1$ and $u_1'$

***Answer***
$$
u_1 = sum(s_i) | {i, A}; u_1' = sum(s_i + p_1) | {i, A}
$$
Final result:
$$
(n+1)u_1' - nu_1 = sp_1
$$
With:
$s$ is the total salary of all employees in this company
$n$ is the size of the company (total employees)

Replace this formular with given values -> calculate the estimation of the $p_1$

## Part 2: (DP proof)

Assume the following mechanism $M : \mathcal{D} \to \mathcal{D}$:

$$
\begin{align}
M(D) &= \emptyset, && \text{with prob. } 1 - \delta \\
M(D) &= D, && \text{with prob. } \delta
\end{align}
$$

For this exercise, assume that the dataset are uniformly chosen from $\mathcal{D}$.

**a)** Prove that given $M(D)$ is $(0, \delta)$-DP.

$$
\forall O \subseteq \mathcal{D}, D \sim_1 D' \cdot \Pr[M(D) \in O] \le \Pr[M(D') \in O] + \delta
$$

***Answer***
Start with:
$$
O = {D \in \mathcal{D} }
$$
Which means O is a subset of the $\mathcal{D}$ so we will have:
$Pr(M(D) \in O) = 1$ and $Pr(M(D') \in O)=1$ (because O is a subset of $\mathcal{D}$) so we of course have $1 \le 1 + \delta$
Case empty:
$$
O = \emptyset
$$
We will have $0\le0+\delta$ because the dataset we observe is not null (not empty), the probability to get a transformed set equals to zero is zero, so we have the above result
Case normal, with the $\delta$
$O\subset \mathcal{D}$ we have $Pr[M(D) \in O] = \delta \cdot \frac{|O|}{|D|}$, of course, the result will be $\delta \cdot \frac{|O|}{|D|} \le \delta \cdot \frac{|O|}{|D|} + \delta$

Repeat the same with the case that $1-\delta$. we also have $Pr[M(D) \in O] = (1 - \delta) \cdot \frac{|O|}{|D|}$

Case that:
$Pr[M(D) \in O] = (1-\delta) \cdot \frac{|O|}{|D|}$
$Pr[M(D')\in O] = \delta \cdot \frac{|O|}{|D|}$

### Proof Derivation (Step-by-Step)

**1. Define the Events**
Let $E$ be the event that the mechanism outputs the empty set $\emptyset$.
Let $E^c$ be the event that the mechanism outputs the dataset $D$.

* $\Pr(E) = 1 - \delta$
* $\Pr(E^c) = \delta$

**2. Law of Total Probability**
The probability of the output falling into set $O$ is:
$$\Pr[M(D) \in O] = \Pr[M(D) \in O \mid E]\Pr(E) + \Pr[M(D) \in O \mid E^c]\Pr(E^c)$$

**3. Substitution (From Blackboard)**
Based on the assumption that datasets are uniformly chosen from $\mathcal{D}$:
* If $M(D) = D$, then $\Pr[D \in O] = \frac{|O|}{|\mathcal{D}|}$.
* The professor simplifies the derivation by assuming the probability is consistent across branches:
$$
\begin{align}
\Pr[M(D) \in O] &= \frac{|O|}{|\mathcal{D}|}(1 - \delta) + \frac{|O|}{|\mathcal{D}|}(\delta) \\
&= \frac{|O|}{|\mathcal{D}|} (1 - \delta + \delta) \\
&= \frac{|O|}{|\mathcal{D}|}
\end{align}
$$

**4. Final DP Proof**
We check the standard Differential Privacy inequality:
$$\Pr[M(D) \in O] \le \Pr[M(D') \in O] + \delta$$

Substitute our result from step 3:
$$\frac{|O|}{|\mathcal{D}|} \le \frac{|O|}{|\mathcal{D}|} + \delta$$

This simplifies to $0 \le \delta$, which is always true.

### Sensitivity and Bounding

To bound sensitivity, you must assume, or *enforce*, lower and upper bounds on the attribute domain, i.e.,

$$
v_i \in [L, U]
$$

In this case, the sensitivity is reduced to:

$$
\Delta q = \max\{|U|, |L|\}
$$

To see why, let us consider the neighbor datasets $D$ and $D' = D \cup v$. Then

$$
\|q(D) - q(D')\|_1 = |q(D) - q(D')| = |\sum_{i \in D} v_i - \sum_{i \in D \cup \{x\}} v_i| = |x| \quad \text{where } x \in [L, U]
$$

The maximum possible change is then $\max |L|, |U|$ – the absolute value is there for when $L < 0$.

> [!warning] Warning
> If the data could violate the established bounds, you need to **clip** or **truncate** values before summation
> 
> $$
> v'_i = \min(U, \max(L, v_i))
> $$

The clipping ensures your sensitivity bound holds, but introduces bias (you are distorting the data). There is a trade-off:
* Narrower bounds $\to$ smaller sensitivity $\to$ less noise, but more distortion (bias).
* Wider bounds $\to$ more noise $\to$ lower utility, but less distortion (bias).

Choosing the bounds is part of design.

> [!note] Note
> If we are in a bounded setting, that is, $D \sim_1^n D'$, what is the sensitivity of the sum?

---

### Problem 3: (Sensitivity)

We have seen in class the notion of $\ell_1$ sensitivity for a function $f : \mathcal{D} \to \mathcal{R}$ as follows:

$$
\Delta_1 f = \max_{D,D':D \sim_1 D'} |f(D) - f(D')|
$$

Assume that datasets have only a numeric column in the range $[a, b]$, where $a \ge 0$.

**a)** What is the sensitivity of the function **maximum**? Justify your answer.

**b)** What is the sensitivity of the function **maximum** if we drop the assumption $a \ge 0$? Justify your answer.

---

### Problem 5: (Queries)

Given a dataset `Sales(date, amount)` where:
* `date` is a date of sale
* `amount` $\in [0, 1000]$ SEK

Assume an *event-level adjacency notion*: two datasets are adjacent by the presence or absence of *one sale*. So, the privacy unit is the event.

You are being called to advice a company using a in-house implementation of DP for SQL. The CTO invites you in and show you some queries they are doing.

Here, there is a query runs the following DP query to count sales per month:

```SQL
SELECT month(date) AS m, LAPLACE(COUNT(*), epsilon=0.1) AS
FROM Sales
GROUP BY month(date)
```


### Your Questions on Problem 5 
You are asking if the budget is **1.2** and if safety depends on the **left quota**. Here is the breakdown: **a) What is the total budget?** * **Your Answer:** "The budget = total right. For each count, consume 0.1 epsilon? so it should be 1.2" * **Verdict:** **Correct.** * **Reasoning:** Since the query groups by `month`, there are 12 distinct groups (January–December). The hint explicitly says to use **Sequential Composition**. $$Cost = \sum \epsilon_i = 12 \times 0.1 = 1.2$$ **b) Is this query safe?** * **Your Answer:** "For the safe, it depends on the left quota?" * **Verdict:** **Partially Correct, but there's a deeper reason.** * **Reasoning:** 1. **Budget Violation:** Yes, if your remaining budget is less than 1.2, the query fails. But "unsafe" often implies a flaw in the mechanism itself. 2. **The "Safety" Flaw:** A standard `GROUP BY` in SQL suppresses empty groups. If "February" has zero sales, the row is missing. The **absence of a row** leaks information (that there were 0 sales) without any privacy noise added to mask it. This is a common privacy vulnerability. To be safe, you must force the output to include all 12 months (even if count is 0) and add noise to the 0s. **c) Can you write a better query?** * **The Trick:** Use **Parallel Composition**. * Since `month(date)` partitions the data into **disjoint** sets (one sale cannot be in two months), the Global Sensitivity for the entire histogram is **1**. * **The Gain:** We can apply the **full budget** to *every* month simultaneously. * **Old Query:** Spent 1.2 budget. Noise was based on $\epsilon=0.1$ (High noise). * **New Query:** Spend 1.2 budget. Apply $\epsilon=1.2$ to *each* month. * **Result:** You get the **same privacy cost** (1.2), but the noise is drastically lower (12x more precise).