#data #security #privacy 

# Preliminaries
This is a section about what you need to know before digging in deeper about data privacy. Since the anonymized and [[Anonymized in protection of data privacy|k-anonymity]] (even with the l-diversity) cannot cover all 100% attack cases, we go to differential privacy.

First, a question was raised "What is privacy". To consider it and have a correct judgment, we should look up to "what has been looked up, queried" and from these data, what was exposed.
For example, for a database *D*, we queried some data about the medicine a user *A* used. But it really depended on what function was used to get these data.
- You don't directly ask "Hey, what is the medicine's status of *A*".
- But you used that record to define the condition of *B* and *C*
**And that's still counted as violation of data privacy** -> You used a sensitive data of someone to exposed the data of the others.

So we have another consideration: *The tradeoff between utility and privacy*:
- Less privacy comes with more accuracy (on analytics).
- More privacy comes with less accuracy (because we added noise, replaced data, surpressed data,...)

## Queries
We have a bunch of several queries to use for attacking
### 1. Counting queries
>[!Definition]
>Based on that, we can define a **counting query** as a query that can be expressed as a sum of contributions, one from each row in the dataset. Formally, it is a function:
>$$ q: [\texttt{row}] \to \mathbb{N} $$ such that there exists a predicate function $p : row \to \{0, 1\}$ with $$ q(D) = \sum_{r \in D} p(r). $$

You pick the value of the row (you don't really care about its value, you just count them) and count all the rows that satisfy condition (i.e: count all the row WHERE illness = flu).
### 2. Point functions
>[! Definition]
>A **point function** is one of the most basic kind of query. It checks whether a dataset row is equal to some fixed value.

Formally, if rows come from a domain $row$, then for any fixed point $p \in row$ we define:

$$
f_p :: row \to \{0, 1\}
$$

$$
f_p(r) = \begin{cases} 
1 & \text{if } r = p \\
0 & \text{otherwise.}
\end{cases}
$$

The corresponding point counting query over a dataset $D$ is:

$$
q_p(D) = \sum_{r \in D} f_p(r),
$$

which simply counts how many rows in the dataset are exactly equal to the point $p$.

**Example:** In a medical dataset where $row = \mathcal{Age} \times \mathcal{Disease}$, the point function for $p = (35, \text{flu})$ returns the number of 35-year-old patients with flu.
Suppose our dataset is:

$$
D = [(35, \text{flu}), (40, \text{covid}), (35, \text{flu}), (60, \text{flu}), (40, \text{flu}), (35, \text{covid}), (60, \text{covid}), (40, \text{covid})].
$$

The domain is $row = Age \times Disease$, with ages $\{35, 40, 60\}$ and diseases $\{\text{flu}, \text{covid}\}$.

For each point function $p_{(a,d)}$, we have:

$$
\begin{aligned}
q_{(35,\text{flu})}(D) &= 2, \\
q_{(35,\text{covid})}(D) &= 1, \\
q_{(40,\text{flu})}(D) &= 1, \\
q_{(40,\text{covid})}(D) &= 2, \\
q_{(60,\text{flu})}(D) &= 1, \\
q_{(60,\text{covid})}(D) &= 1.
\end{aligned}
$$

Collecting all these answers gives the **histogram**:

| Age | Disease | Count |
| :--- | :--- | :--- |
| 35 | flu | 2 |
| 35 | covid | 1 |
| 40 | flu | 1 |
| 40 | covid | 2 |
| 60 | flu | 1 |
| 60 | covid | 1 |
With this function, and assume that you try to count every "conditions" by generating so many point functions, you can construct the complete histogram of the database.
### 3. Threshold function
A function to check if the value is under (or higher) than a threshold.
>[! Definition]
>A **threshold function** is another common query. It checks whether a dataset row is below some fixed value.

Formally, if rows come from an **ordered** domain $row$, then for any threshold $t \in row$ we define: $$ f_t :: row \to \{0, 1\} $$ $$ f_t(r) = \begin{cases} 1 & \text{if } r \leq t \\ 0 & \text{otherwise.} \end{cases} $$ The corresponding threshold counting query over a dataset $D$ is: $$ q_t(D) = \sum_{r \in D} f_t(r), $$ which simply counts how many rows in the dataset are exactly below the threshold $t$. **Example:** suppose our dataset $D$ contains exam scores $$ D = \{45, 60, 72, 90, 30, 55\}. $$ Let's pick a threshold $t = 60$. The threshold function is $$ f_{60}(r) = \begin{cases} 1 & \text{if } r \leq 60, \\ 0 & \text{otherwise.} \end{cases} $$ Applying $f_{60}$ to each row in $D$: * $f_{60}(45) = 1$ * $f_{60}(60) = 1$ * $f_{60}(72) = 0$ * $f_{60}(90) = 0$ * $f_{60}(30) = 1$ * $f_{60}(55) = 1$ Now the threshold counting query is: $$ q_{60}(D) = \sum_{r \in D} f_{60}(r) = 1 + 1 + 0 + 0 + 1 + 1 = 4. $$
We evaluate $q_t(D)$ for all $t \in \{1, 2, \dots, 10\}$:

* $q_1(D) = \#\{r \leq 1\} = 0$
* $q_2(D) = 0$
* $q_3(D) = 1$ (since one value $= 3$)
* $q_4(D) = 1$
* $q_5(D) = 1$
* $q_6(D) = 1$
* $q_7(D) = 3$ (since three values are $\leq 7$)
* $q_8(D) = 3$
* $q_9(D) = 3$
* $q_{10}(D) = 4$ (all four values are $\leq 10$)

If we look at all threshold queries together, we get the **empirical cumulative distribution function (CDF)** of the dataset.

For this dataset, the empirical CDF is:

$$
F_D(t) = \frac{q_t(D)}{|D|}.
$$

So:

* At $t=3$, $F_D(3) = 1/4 = 0.25$.
* At $t=7$, $F_D(7) = 3/4 = 0.75$.
* At $t=10$, $F_D(10) = 1.0$.

>[! Reference]
>The empirical cumulative distribution function (ECDF) is ==a step function that estimates the cumulative distribution of a sample of data by calculating the proportion of observations less than or equal to a given value==. It provides a non-parametric way to visualize and analyze how data accumulates, showing the cumulative probability for each data point in the sample, sorted from lowest to highes


# Differential Privacy

>[! Definition]
>Differential Privacy is a mere definition. It characterizes how much the output (distribution) of an algorithm can change when a single individual’s data is modified, added, or removed.

Simply: we try to make the output result not depend on any individual record (or element). So that, every an attacker or an analysist try to capture any query, even with the intention to exclude some data, the returned answer must be the same.

So we move to the mathematical model of this concept: giving 2 dataset $D_1$ and $D_2$. We have 2 definition:
- Unbounded DP: $D_1$ and $D_2$ is unbounded different if there are differ with the presence or absence of *k* individual's record.
- Bounded DP: $D_1$ and $D_2$ are differ exactly *k* record (~ Hamming distant).
What is the different of these 2?
- Unbounded: **adding or removing one can result another*
- Bounded: **these two must have same size**

