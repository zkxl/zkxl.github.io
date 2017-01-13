---
layout: post
title: Deep Learning in Math
date: 2016-12-15 15:19:00 -0800
categories: deeplearning-basics
tag: undecided
---

* content
{:toc}


## universal approximator

From "Multilayer feedforward networks are universal approximators", it is shown that a neural net with a single hidden layer already has __universal approximation property__, meaning: it can approximate any continuous function from a compact domain to the reals, to arbitrary precision.  


Mathematically, for any $$ \epsilon > 0 $$ there exists an $$ \alpha > 0 $$, so that:  

$$ |x_1 - x_2| < \alpha => |f(x_1) - f(x_2)| < \epsilon $$  

Here, with $$ \epsilon $$ being the arbitrary precision, for every small interval $$ [\alpha_{n-1}, \alpha_{n}) $$ in input space, you can allocate a neural with a threshold activation function that produce a value of $$ f(\alpha_{n}) - f(\alpha_{n-1}) $$ when activated.  

The above neural net gives you an "approximator" $$ F(x) $$ !  

Now for any x in input space, you have:  

$$ |f(x) - F(x)| < \epsilon $$  

---

## why squared error?

For data D = {I(t), T(t) where t = 1, …, M} where I(t) is a vector of dim N and T(t) is a 1-dim target output.  

P(W) is the prior distribution of the model;  

P(D) is the evidence distribution of the model;  

$$ P(W|D) $$ is given the data the probability distribution of the mode;  

Having:  

$$ P(W|D) = \frac {P(D|W) P(W)} {P(D)} $$  

__The ultimate goal of any machine learning problem is to find the "most correct" model given data.__  

Problem:  

$$ \underset{W} {\operatorname{\min}} P(W|D) $$
