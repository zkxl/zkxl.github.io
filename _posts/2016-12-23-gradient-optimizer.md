---
layout: post
title: commonly used first order optimizer
date: 2016-12-23 18:06:00 -0800
categories: deeplearning-basics 
tag: optimizer
---

* content
{:toc}



__These deeplearning series are inspired by Stanford open course — cs231n. My special thanks.__


_Deeplearning is involved in a wide range of techniques that allow a neural network to make reliable predications. This series
 will introduce several key components of a neural network, from linear classifier loss functions to fully connected neural ne
ts, from convNets to RNN._


The deeplearning library that I intended to introduce here consists of a design of architecture that depends on your chosen model and a Solver() class that facilitate the training/testing process. I will dedicate one post specifically about the Solver() class. The following is how it calculates loss and update params:

``` python
  def _step(self):
    
    """
    Make a single gradient update. This is called by train() and should not
    be called manually.
    """
    
    # Make a minibatch of training data
    ...

    # Compute loss and gradient
    ...

    # Perform a parameter update
    for p, w in self.model.params.iteritems():
      
      # grads is a dictionary-type output of the above loss calculation; it has the same key work mapping toself.model.params.
      dw = grads[p]

      config = self.optim_configs[p]

      ################################################################################################################
      # self.update_rule = kwargs.pop('update_rule', 'sgd') -> update_rule links to one of the following optimizers. #
      ################################################################################################################
      next_w, next_config = self.update_rule(w, dw, config) # generally next_w <- w - lr * dw, which is also how bnGamma and bnBeta got updated.

      self.model.params[p] = next_w
      self.optim_configs[p] = next_config
```

---

### general function implementation notes:

``` python
"""
This file implements various first-order update rules that are commonly used for
training neural networks. Each update rule accepts current weights and the
gradient of the loss with respect to those weights and produces the next set of
weights. Each update rule has the same interface:

def update(w, dw, config=None):

Inputs:
  - w: A numpy array giving the current weights.
  - dw: A numpy array of the same shape as w giving the gradient of the
    loss with respect to w.
  - config: A dictionary containing hyperparameter values such as learning rate,
    momentum, etc. If the update rule requires caching values over many
    iterations, then config will also hold these cached values.

Returns:
  - next_w: The next point after the update.
  - config: The config dictionary to be passed to the next iteration of the
    update rule.

NOTE: For most update rules, the default learning rate will probably not perform
well; however the default values of the other hyperparameters should work well
for a variety of different problems.

For efficiency, update rules may perform in-place updates, mutating w and
setting next_w equal to w.
"""
```

---

### vanilla sgd - 

$$ w = w - \eta \sum_{i = 1}^{n} \nabla Q_{i}(w) $$

``` python

def sgd(w, dw, config=None):
  """
  Performs vanilla stochastic gradient descent.

  config format:
  - learning_rate: Scalar learning rate.
  """
  if config is None: config = {}
  config.setdefault('learning_rate', 1e-2) #

  w -= config['learning_rate'] * dw
  return w, config

```

---

### sgd + momentum 

$$ w = w - \eta \sum_{i = 1}^{n} \nabla Q_{i}(w) + \alpha \Delta w $$

``` python
def sgd_momentum(w, dw, config=None):
  """
  Performs stochastic gradient descent with momentum.

  config format:
  - learning_rate: Scalar learning rate.
  - momentum: Scalar between 0 and 1 giving the momentum value.
    Setting momentum = 0 reduces to sgd.
  - velocity: A numpy array of the same shape as w and dw used to store a moving
    average of the gradients.
  """
  if config is None: config = {}
  
  # dict.setdefault(k, v): return dict[k] if k in dict else return v and set dict[k] = v
  config.setdefault('learning_rate', 1e-2)
  config.setdefault('momentum', 0.9)
  
  # dict.get(k, v): return dict[k] if k in dict else return v but k will still not be in dict.
  v = config.get('velocity', np.zeros_like(w))

  m = config['momentum']

  lr = config['learning_rate']

  v = - lr * dw + m * v # initially it makes no changes as compared with w -= learning_rate * dw

  w += v

  config['velocity'] = v           # but it accumulates "dw" into velocity and thus builds up the speed to descent 
                                   # if dw stays positive/negative for quite a few updates
  return w, config
```

---

### RMSprop - root mean squared propgation

Quote from Wiki -
"RMSProp (for Root Mean Square Propagation) is also a method in which the learning rate is adapted for each of the parameters. The idea is to divide the learning rate for a weight by a running average of the magnitudes of recent gradients for that weight. So, first the running average is calculated in terms of means square."

$$ v(w,t) = \gamma v(w,t-1) + (1- \gamma )( \nabla Q_{i}(w))^{2} $$

$$ w = w - {\frac {\eta }{\sqrt {v(w,t)}}} \nabla Q_{i}(w) $$

``` python
def rmsprop(w, dw, config=None):
  """
  Uses the RMSProp update rule, which uses a moving average of squared gradient
  values to set adaptive per-parameter learning rates.

  config format:
  - learning_rate: Scalar learning rate.
  - decay_rate: Scalar between 0 and 1 giving the decay rate for the squared
    gradient cache.
  - epsilon: Small scalar used for smoothing to avoid dividing by zero.
  - cache: Moving average of second moments of gradients.
  """
  if config is None: config = {}
  config.setdefault('learning_rate', 1e-2)
  config.setdefault('decay_rate', 0.99)
  config.setdefault('epsilon', 1e-8)
  config.setdefault('cache', np.zeros_like(w))

  lr = config['learning_rate']
  dr = config['decay_rate']
  eps = config['epsilon']
  cache = config['cache']

  # rmsprop is built on adagrad which follows
  # ----------------------------------------
  # cache += dw ** 2 -> tracks of per-parameter sum of squared historical gradients
  # w -= dw * learning_rate / (sqrt(cache) + eps) -> benefit: equalizing learning rate for big/small gradients
  #                                                -> disadvantage: monotonically decreasing dw, too aggresive
  # ----------------------------------------
  
  # rmsprop, instead of using sum of histroical effects, uses a moving average (leaky) strategy to adaptively determine the learning_rate

  cache = dr * cache + (1 - dr) * dw ** 2

  w -= dw * lr / (np.sqrt(cache) + eps)

  config['cache'] = cache
  
  return w, config

```

---

### Adam 

Adam combines RMSprop and momentum, running an average on both gradients momentum and their magnitude.

$$ m(w,t) = \beta_{1} m(w,t-1) + (1- \beta_{1}) \nabla Q_{i} (w) $$

$$ v(w,t) = \beta_{2} v(w,t-1) + (1- \beta_{2}) (\nabla Q_{i}(w))^{2} $$

$$ w(t+1) = w(t) - {\frac {\eta} {\sqrt {v(w,t)} + \epsilon}} m(w,t) $$

``` python
def adam(w, dw, config=None):
  """
  Uses the Adam update rule, which incorporates moving averages of both the
  gradient and its square and a bias correction term.

  config format:
  - learning_rate: Scalar learning rate.
  - beta1: Decay rate for moving average of first moment of gradient.
  - beta2: Decay rate for moving average of second moment of gradient.
  - epsilon: Small scalar used for smoothing to avoid dividing by zero.
  - velocity: Moving average of gradient.
  - cache: Moving average of squared gradient.
  - t: Iteration number.
  """
  if config is None: config = {}
  lr = config.setdefault('learning_rate', 1e-3)
  beta1 = config.setdefault('beta1', 0.9)
  beta2 = config.setdefault('beta2', 0.999)
  eps = config.setdefault('epsilon', 1e-8)
  momentum = config.setdefault('momentum', np.zeros_like(w))
  cache = config.setdefault('cache', np.zeros_like(w))
  t = config.setdefault('t', 0)
  
  # adam is the combination of rmsprop (cache part) and sgd_momentum (only difference: raw dw -> (1 - beta1) * dw for a smoothier update)
  momentum = beta1 * momentum + (1 - beta1) * dw

  cache = beta2 * cache + (1 - beta2) * dw ** 2

  w -= lr * momentum / (np.sqrt(cache) + eps)
  
  t += 1

  config['momentum'] = momentum
  config['cache'] = cache
  config['t'] = t

  return w, config
```
