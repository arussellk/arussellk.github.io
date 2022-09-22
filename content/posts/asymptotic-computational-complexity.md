---
layout: post
title: Asymptotic Computational Complexity
has_math: true
---

Since asymtotic complexity describes membership in a set,
I wish the preferred notation was \\(f(n) \in O(T(n))\\)
but when talking about asymptotic complexity
it is typically written \\(f(n)=O(T(n))\\).
(Interestingly, the
[German](https://de.wikipedia.org/wiki/Landau-Symbole#Definition)
and
[French](https://fr.wikipedia.org/wiki/Comparaison_asymptotique#La_famille_de_notations_de_Landau_O.2C_o.2C_.CE.A9.2C_.CF.89.2C_.CE.98.2C_.C3.95)
Wikipedia articles on Big O notation use \\(\in\\).)

Let's say we determine that our algorithm takes \\(T(n)\\) time to run.
We might write something like:

\\[
T(n) = 5n^2 + 2n \log n + 3n + C
\\]

Then we might want to compare it to some \\(f(n)\\) using Big O notation like
\\(O(f(n))\\). We'd write something like this:

\\[
\begin{aligned}
T(n) &= 5n^2 + 2n \log n + 3n + C\\\\
T(n) &= O(n^2)
\end{aligned}
\\]

### Big O and Friends Quick Reference

\\(T(n) = o(f(n))\\) means \\(T(n) < f(n)\\)

\\(T(n) = O(f(n))\\) means \\(T(n) \leq f(n)\\)

\\(T(n) = \Theta(f(n))\\) means \\(T(n) = f(n)\\)

\\(T(n) = \Omega(f(n))\\) means \\(T(n) \geq f(n)\\)

\\(T(n) = \omega(f(n))\\) means \\(T(n) > f(n)\\)
