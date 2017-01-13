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

Given the data, the probability distribution of the model is, the probability distribution of the data given the model times the prior distribution of the model divided by the evidence distribution of the model.  

Mathematically:  

$$ P(W|D) = \frac {P(D|W) P(W)} {P(D)} $$  

__The ultimate goal of any machine learning problem is to find the "most correct" model given data.__  

Problem:  

$$ \underset{W} {\operatorname{\max}} P(W|D) $$

Equivalently:  

$$ \underset{W} {\operatorname{\max}} \log{P(W|D)} $$  

Equivalently:  

$$ \underset{W} {\operatorname{\max}} (\log{P(D|W)} + \log{P(W)} - \log{P(D)}) $$

Under uniform prior weight distribution, P(W) is a constant;  
Because of given data, P(D) is a constant;  

So problem became:  

$$ \underset{W} {\operatorname{\max}} \log{P(D|W)} $$

Equivalently (negative loglikelihood):  

$$ - \underset{W} {\operatorname{\min}} \log{P(D|W)} $$

Because:  

$$ P(D|W) = P({I(t), T(t) where t = 1, …, M} | W) = \pi_{t=1}^{M} P(I(t), T(t) | W) = \pi_{t=1}^{M} P(I(t)) P(T(t)|O(t)) $$

* note: O stands for the output; the above says: [given model W, the probability of (having input I(t) and target T(t))] equals to [the probability of (having input I(t) independently) and (the probability of having target value T(t) given output O(t))].  
