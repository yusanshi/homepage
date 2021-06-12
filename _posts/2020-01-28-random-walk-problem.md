---
title: Random Walk Problem
tags:
  - Math
mathjax: true
---

# Problem

Consider the random walk $ X=\\{X_n\\}_{n \geq 0} $ on $ \mathbb{Z} $ that starts at $ X_0=0 $. The particle moves with probability $ p $ one unit to the right and with probability $ q=1-p $ one unit to the left at each transition. Please show that the state $ 0 $ is recurrent if and only if $ p=q=1/2 $, i.e.,

$$
\sum_{n=1}^{\infty} \mathbb{P}(X_n=0, X_k \neq 0, k = 1,\dots,n-1|X_0=0)=1\, \Leftrightarrow\, p=q=1/2.
$$

# Solution

Let $f_{ij}^{(n)}$ be the probability that a particle starting at state $i$ will visit state $j$ for the first time after $n$ steps, i.e.,

$$
	f_{ij}^{(n)} = \mathbb{P}(X_n=j, X_k \neq j, k = 1,\dots,n-1|X_0=i)
$$

Let $f_{ij}$ be the probability that the particle starting at state $i$ will ever visit state $j$, i.e.,

$$
	f_{ij} = \sum_{n=1}^{\infty} f_{ij}^{(n)}
$$

Now we need to prove

$$
	f_{00} = 1 \, \Leftrightarrow\, p=q=1/2
$$

Think about $ f_{00}^{(i)} $. If $ i $ is odd, obviously $ f_{00}^{(i)} = 0 $. If $ i $ is even, then we know the particle moves to the right $ \frac{i}{2} $ times and to the left $ \frac{i}{2} $ times.
What matters is that the particle must not return to state $0$ on the halfway.

Recall valid parenthesis expressions problem. The number of valid parenthesis expressions that consist of $n$ right parentheses and $n$ left parentheses is equal to the $n^{th}$ Catalan number ${\displaystyle C_{n}={\frac {1}{n+1}}\binom{2n}{n} }$.
If the particle moves to the right at first, then moving right is just like left parenthesis and moving left is like right parenthesis. The only difference is that in the valid parenthesis expressions problem,
initial segment of the string can have equal `(` and `)` while in the random walk problem it is not allowed, since the particle should not return to $ 0 $ on the halfway. We solve the problem in this way: If the particle moves to the right at first,
then we think it as valid parenthesis expressions problem with an extra requirement that there must be outermost parentheses which contain all others. With the requirement, the number of valid parenthesis expression changes from ${\displaystyle C_{n} }$ to ${\displaystyle C_{n-1} }$.
So, if particle moves to the right at first, there are ${\displaystyle C_{\frac{i}{2}-1} }$ ways to move. Moving to the left at first will contribute another ${\displaystyle C_{\frac{i}{2}-1} }$.
Each way has a probability of $(pq)^{\frac{i}{2}}$, so the final probability is ${\displaystyle 2 C_{\frac{i}{2}-1} (pq)^{\frac{i}{2}}}$.
Then we have

$$
f_{00}^{(i)} =
	\begin{cases}
		0                                     & \quad \text{if } i \text{ is odd}  \\
		2 C_{\frac{i}{2}-1}(pq)^{\frac{i}{2}} & \quad \text{if } i \text{ is even}
	\end{cases}
$$

Let $ m = pq $, then

$$
\begin{split}
		f_{00}  &= \sum_{i=1}^{\infty} f_{00}^{(i)}\\
		&= \sum_{i=1}^{\infty} 2 C_{i-1}(m)^i\\
		&= 2 m \sum_{i=0}^{\infty} C_i(m)^i
	\end{split}
$$

The generating function for the Catalan numbers is

$$
	c(x)=\sum_{n=0}^{\infty }C_{n}x^{n} ={\frac {1-{\sqrt {1-4x}}}{2x}}
$$

So

$$
\begin{split}
		f_{00} &= 2 m \,c(m) \\
		&= 2 m\,\frac{1 - \sqrt{1 - 4m} }{2m}\\
		&= 1- \sqrt{1 - 4m}
\end{split}
$$

Since $ m \in \left[0,\frac{1}{4}\right] $, we have

$$
f_{00} = 1 \, \Leftrightarrow\, m = \frac{1}{4} \, \Leftrightarrow\, p=q=1/2
$$
