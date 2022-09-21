---
layout: post
title: Arithmetic, Geometric, and Harmonic means
has_math: true
---

An arithmetic mean is the value \\(a\\) such that:

$$
x_1+x_2+\cdots+x_n =
\underbrace{a+a+\cdots+a}_{n}
$$

A geometric mean is the value \\(g\\) such that:

$$
x_1 \times x_2 \times \cdots \times x_n =
\underbrace{g \times g \times \cdots \times g}_{n}
$$

A harmonic mean is the value \\(h\\) such that:

$$
\frac{1}{x_1}+\frac{1}{x_2}+\cdots+\frac{1}{x_n}=
\underbrace{\frac{1}{h}+\frac{1}{h}+\cdots+\frac{1}{h}}_{n}
$$

### Deriving the standard formulas

Arithmetic mean:

$$
\begin{aligned}
\underbrace{a+a+\cdots+a}_{n} &=
x_1+x_2+\cdots+x_n \\\\
na &= x_1+x_2+\cdots+x_n \\\\
a &= \frac{x_1+x_2+\cdots+x_n}{n}
\end{aligned}
$$

Geometric mean:

$$
\begin{aligned}
\underbrace{g \times g \times \cdots \times g}_{n} &=
x_1 \times x_2 \times \cdots \times x_n \\\\
g^n &= x_1 \times x_2 \times \cdots \times x_n \\\\
g &= \sqrt[n]{x_1 \times x_2 \times \cdots \times x_n}
\end{aligned}
$$

Harmonic mean:
$$
\begin{aligned}
\underbrace{\frac{1}{h}+\frac{1}{h}+\cdots+\frac{1}{h}}_{n} &=
\frac{1}{x_1}+\frac{1}{x_2}+\cdots+\frac{1}{x_n} \\\\
\frac{n}{h} &= \frac{1}{x_1}+\frac{1}{x_2}+\cdots+\frac{1}{x_n} \\\\
h &= \frac{n}{\frac{1}{x_1}+\frac{1}{x_2}+\cdots+\frac{1}{x_n}} \\\\
h &= \frac{n}{ \displaystyle \sum\_{i=1}^n \frac{1}{x_i}}
\end{aligned}
$$
