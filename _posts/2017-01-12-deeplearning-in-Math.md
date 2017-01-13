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

$$ \underset{W} {\operatorname{\min}} - \log{P(D|W)} $$

Because:  

$$ P(D|W) = P({I(t), T(t) t = 1, …, M} | W) = \Pi_{t=1}^{M} P(I(t), T(t) | W) = \Pi_{t=1}^{M} P(I(t)) P(T(t)|O(t)) $$

* note: O stands for the output. Given data D, I(t) is a constant.    

_the above says: [given model W, the probability of (having input I(t) and target T(t))] equals to the probability of [(having input I(t) independently) and (having target value T(t) given output O(t))]._  

The problem became:  

$$ \underset{W} {\operatorname{\min}} - \log{(\Pi_{t=1}^{M} P(T(t)|O(t)))} $$

Equivalently:  

$$ \underset{W} {\operatorname{\min}} - \sum_{t=1}^{M} \log{(P(T(t)|O(t))} $$

__The above problem is obviously a regression/continuous probability problem. Now what? Here comes in Gaussian distribution, which is the most commonly used continuous probability distribution to model real-valued, random variables whose probability distribution is unknown. It is useful because of central limit theorem.__  

$$ f(x |\mu ,\sigma^{2}) = \frac {1}{\sqrt {2 \sigma^{2} \pi}} \e^{-{\frac {(x-\mu)^{2}} {2 \sigma^{2}}}} $$


Apply it to our problem:  

$$ P(T|O) = {\frac {1}{\sqrt {2 \sigma^{2} \pi}}} \e^{-1/2 {\frac {(T-O)^{2}} {\sigma^{2}}} $$  

Then the above problem became:  

$$ \underset{W} {\operatorname{\min}} - \sum_(t=1)^{M} \log{ ( {\frac {1}{\sqrt {2 \sigma^{2} \pi}}} \e^{-1/2 {\frac {(T-O)^{2}} {\sigma^{2}}} ) } $$  

Because $$ \frac {1}{\sqrt {2 \sigma^{2} \pi}} $$ is a constant, our above optimize problem became:  

$$ \underset{W} {\operatorname{\min}} - \sum_(t=1)^{M}\log{\e^{-1/2 {\frac {(T-O)^{2}} {\sigma^{2}}} $$  

Equivalently:  

$$ \underset{W} {\operatorname{\min}} \sum_{t=1}^{M} 1/2 {\frac {(T-O)^{2}} {\sigma^{2}}} $$  

Equivalently:  

$$ \underset{W} {\operatorname{\min}} \sum_{t=1}^{M} 1/2 {(T-O)^{2}} $$  

__This is where the "squared error" came from!__

* Note: We didn't omit the constant 1/2 here because it is we can drive a nicer derivative function with it.  
