---
layout: post
title: Asymptotic Computational Complexity (Big O and friends)
has_math: true
---

Since asymtotic complexity describes membership in a set,
I wish the preferred notation was $$f(n) \in O(T(n))$$
but when talking about asymptotic complexity
it is typically written $$f(n)=O(T(n))$$.

Let's say we determine that our algorithm takes $$T(n)$$ time to run. We might write something like:

$$
T(n) = 5n^2 + 2n \log n + 3n + C
$$

Then we might want to compare it to some $$f(n)$$ using Big O notation like $$O(f(n))$$. We'd write something like this:

$$
\begin{align}
T(n) &= 5n^2 + 2n \log n + 3n + C\\
T(n) &= O(n^2)
\end{align}
$$

### Big O and Friends

$$T(n) = o(f(n))$$ means $$T(n) < f(n)$$

$$T(n) = O(f(n))$$ means $$T(n) \leq f(n)$$

$$T(n) = \Theta(f(n))$$ means $$T(n) = f(n)$$

$$T(n) = \Omega(f(n))$$ means $$T(n) \geq f(n)$$

$$T(n) = \omega(f(n))$$ means $$T(n) > f(n)$$
