---
layout: post
title: fully-connected neural network
date: 2016-12-23 22:24:00 -0800
categories: deeplearning-basics
tag: fully-connected-nn
---

* content
{:toc}



__This deeplearning series is inspired by Stanford open course — cs231n. My special thanks.__


_Deeplearning is involved in a wide range of techniques that allow a neural network to make reliable predications. This series will introduce several key components of a neural network, from linear classifier loss functions to fully connected neural nets, from convNets to RNN._


### Affine transformation

$$\sum_iw_ix_i + b$$

``` python
def affine_forward(x, w, b):

  """
  Computes the forward pass for an affine (fully-connected) layer.

  The input x has shape (N, d_1, ..., d_k) and contains a minibatch of N
  examples, where each example x[i] has shape (d_1, ..., d_k). We will
  reshape each input into a vector of dimension D = d_1 * ... * d_k, and
  then transform it to an output vector of dimension M.

  Inputs:
  - x: A numpy array containing input data, of shape (N, d_1, ..., d_k)
  - w: A numpy array of weights, of shape (D, M)
  - b: A numpy array of biases, of shape (M,)

  Returns a tuple of:
  - out: output, of shape (N, M)
  - cache: (x, w, b)
  """

  out = None
  x_shape = x.shape
  N = x_shape[0]
  x = x.reshape(N, -1)

  out = np.dot(x, w) + b # (N, M) + (M, ) broadcast == (N, M) + (1, M) broadcast

  x = x.reshape(x_shape) # keep input/output dimensionality

  cache = (x, w, b)

  return out, cache


def affine_backward(dout, cache):

  """
  Computes the backward pass for an affine layer.

  Inputs:
  - dout: Upstream derivative, of shape (N, M)
  - cache: Tuple of:
    - x: Input data, of shape (N, d_1, ... d_k)
    - w: Weights, of shape (D, M)

  Returns a tuple of:
  - dx: Gradient with respect to x, of shape (N, d1, ..., d_k)
  - dw: Gradient with respect to w, of shape (D, M)
  - db: Gradient with respect to b, of shape (M,)
  """

  x, w, b = cache
  dx, dw, db = None, None, None

  x_shape = x.shape
  N = x_shape[0]
  x = x.reshape(N, -1) # for computing dw

  dx = np.dot(dout, w.T)
  dx = dx.reshape(x_shape) # keep input/output dimensionality

  dw = np.dot(x.T, dout)

  db = np.sum(dout, axis = 0)

  return dx, dw, db
```

### Activation

Activation function evolved from sigmoid() to tanh(), making it ZERO-centered; from tanh() to ReLU(), making it easier to compute. Historically, gradients vanishing is the major problem that negatively impact the efficacy of neural networks. Using ReLU also faces dead ReLU problem that we will use batch normalization to address.

* Note: sigmoid() and tanh() both have very nice forms of derivative functions.

$$\sigma(\sum_iw_ix_i + b) $$


``` python
def relu_forward(x):

  """
  Computes the forward pass for a layer of rectified linear units (ReLUs).

  Input:
  - x: Inputs, of any shape

  Returns a tuple of:
  - out: Output, of the same shape as x
  - cache: x
  """

  out = None

  # initially I didn't use copy() which lead to the changes occurred in x, wasted 2 hours on this.
  out = np.copy(x)

  out[out <= 0] = 0

  cache = x

  return out, cache


def relu_backward(dout, cache):

  """
  Computes the backward pass for a layer of rectified linear units (ReLUs).

  Input:
  - dout: Upstream derivatives, of any shape
  - cache: Input x, of same shape as dout

  Returns:
  - dx: Gradient with respect to x
  """

  dx, x = None, cache

  dx = np.copy(dout)

  dx[x <= 0] = 0

  return dx

```

### Batch normalization

$$\frac {X - \mu}{\sigma}$$


Batch normalization usually follows affine transformation and make sure that the input of the following activation function is in a unit guassian distribution so that there will be enough "alive" ReLUs in each layer to deliver effective communications.


``` python
def batchnorm_forward(x, gamma, beta, bn_param):

  """
  Forward pass for batch normalization.

  During training the sample mean and (uncorrected) sample variance are
  computed from minibatch statistics and used to normalize the incoming data.
  During training we also keep an exponentially decaying running mean of the mean
  and variance of each feature, and these averages are used to normalize data
  at test-time.

  At each timestep we update the running averages for mean and variance using
  an exponential decay based on the momentum parameter:

  running_mean = momentum * running_mean + (1 - momentum) * sample_mean
  running_var = momentum * running_var + (1 - momentum) * sample_var

  Note that the batch normalization paper suggests a different test-time
  behavior: they compute sample mean and variance for each feature using a
  large number of training images rather than using a running average. For
  this implementation we have chosen to use running averages instead since
  they do not require an additional estimation step; the torch7 implementation
  of batch normalization also uses running averages.

  Input:
  - x: Data of shape (N, D)
  - gamma: Scale parameter of shape (D,)
  - beta: Shift paremeter of shape (D,)
  - bn_param: Dictionary with the following keys:
    - mode: 'train' or 'test'; required
    - eps: Constant for numeric stability
    - momentum: Constant for running mean / variance.
    - running_mean: Array of shape (D,) giving running mean of features
    - running_var: Array of shape (D,) giving running variance of features

  Returns a tuple of:
  - out: of shape (N, D)
  - cache: A tuple of values needed in the backward pass
  """

  mode = bn_param['mode']
  eps = bn_param.get('eps', 1e-5)
  momentum = bn_param.get('momentum', 0.9)

  N, D = x.shape
  running_mean = bn_param.get('running_mean', np.zeros(D, dtype=x.dtype))
  running_var = bn_param.get('running_var', np.zeros(D, dtype=x.dtype))

  out, cache = None, None

  if mode == 'train':

    batch_mean = np.mean(x, axis = 0)
    batch_var = np.std(x, axis = 0)

    running_mean = momentum * running_mean + (1 - momentum) * batch_mean
    running_var = momentum * running_var + (1 - momentum) * batch_var

    x_mu = x - batch_mean

    var = np.mean(np.square(x_mu), axis = 0)

    sqrtvar = np.sqrt(var + eps)

    i_sqrtvar = 1.0 / sqrtvar

    x_hat = x_mu * i_sqrtvar

    out = np.multiply(x_hat, gamma) + beta

    cache = (x_hat, i_sqrtvar, sqrtvar, var, x_mu, gamma, eps)

  elif mode == 'test':

    x = (x - running_mean) / running_var
    out = gamma * x + beta

  else:
    raise ValueError('Invalid forward batchnorm mode "%s"' % mode)

  # Store the updated running means back into bn_param
  bn_param['running_mean'] = running_mean
  bn_param['running_var'] = running_var

  return out, cache


def batchnorm_backward(dout, cache):

  """
  Backward pass for batch normalization.

  For this implementation, you should write out a computation graph for
  batch normalization on paper and propagate gradients backward through
  intermediate nodes.

  Inputs:
  - dout: Upstream derivatives, of shape (N, D)
  - cache: Variable of intermediates from batchnorm_forward.

  Returns a tuple of:
  - dx: Gradient with respect to inputs x, of shape (N, D)
  - dgamma: Gradient with respect to scale parameter gamma, of shape (D,)
  - dbeta: Gradient with respect to shift parameter beta, of shape (D,)
  """

  dx, dgamma, dbeta = None, None, None
  x_hat, i_sqrtvar, sqrtvar, var, x_mu, gamma, eps = cache

  # gate: f = x + beta - broadcast
  dbeta = np.sum(1 * dout, axis = 0) # with 1 being the local gradient df/dbeta
  backprop_x = 1 * dout # with 1 being the local gradient df/dx

  # gate: f = x * gamma
  dgamma = np.sum(np.multiply(x_hat, backprop_x), axis = 0) # with x_hat being the local gradient
  backprop_x = np.multiply(backprop_x, gamma) # with gamma being the local gradient

  # gate: f = x_mu * i_sqrtvar ((N, D) * (D,))
  dx_mu1 = np.multiply(backprop_x, i_sqrtvar) # with i_sqrtvar being the local grad
  di_sqrtvar = np.sum(np.multiply(backprop_x, x_mu), axis = 0) # with x_mu being the local grad and itself broadcasted

  # gate: i_sqrtvar = 1 / sqrtvar
  dsqrtvar = np.multiply((-1.0 / sqrtvar ** 2), di_sqrtvar) # with (-1.0 / sqrtvar ** 2) being the local grad

  # gate: f = (var + eps) ^ (1/2)
  dvar = 0.5 * (var + eps) ** (-0.5) * dsqrtvar # with (-0.5 * (var + eps) ** (-1.5)) being the local grad

  # gate: var = np.mean(np.square(x_mu), axis = 0)
  dsq = np.multiply(np.ones(x_mu.shape), dvar) / float(x_mu.shape[0])

  # gate: sq = (x_mu) ^ 2
  dx_mu2 = np.multiply(2 * x_mu, dsq) # with 2 * x_mu being the local grad

  # gate: x_mu = x - mu
  dx1 = 1 * (dx_mu1 + dx_mu2) # with 1 being the local grad
  dmu = -1 * (np.sum(dx_mu1, axis = 0) + np.sum(dx_mu2, axis = 0))

  # gate: mu = np.mean(x, axis = 0)
  dx2 = np.multiply(np.ones(x_mu.shape), dmu) / float(x_mu.shape[0])

  # When multiple gradients come to the same node, add them together.
  dx = dx1 + dx2

  return dx, dgamma, dbeta
```

### Dropout

Dropout can prevent overfitting. It is in a sense a way to regularize the model. Intuitively it forces the network to learning useful features redundently and it makes training a sampling process where variants of the model grow to form an ensemble effects during testing.

``` python
def dropout_forward(x, dropout_param):

  """
  Performs the forward pass for (inverted) dropout.

  Inputs:
  - x: Input data, of any shape
  - dropout_param: A dictionary with the following keys:
    - p: Dropout parameter. We drop each neuron output with probability p.
    - mode: 'test' or 'train'. If the mode is train, then perform dropout;
      if the mode is test, then just return the input.
    - seed: Seed for the random number generator. Passing seed makes this
      function deterministic, which is needed for gradient checking but not in
      real networks.

  Outputs:
  - out: Array of the same shape as x.
  - cache: A tuple (dropout_param, mask).
    dropout mask in training mode, None mask in testing mode.
  """

  out, mask = None, None
  p, mode = dropout_param['p'], dropout_param['mode']

  # In the case of p == 0, this layer acts as if it doesn't exist.
  if p == 0:
    return x, (dropout_param, mask)

  if not dropout_param['seed'] is None:
    np.random.seed(dropout_param['seed'])

  if mode == 'train':

    mask = np.random.rand(*x.shape) < p
    # Here's sth new: call fnc(*(1, 2, 3)) "*" is a splate operator unpacking list or tuple to feed into the fnc call
    # np.random.rand(m, n) is valid while np.random.rand((m, n)) is not.
    # Now mask is a bool array of the same size as x

    out = x * mask / p # if p == 0, this fnc shouldn't invoked at all.
    # The vanilla dropout is simply out = x * mask. Its expected out became p times of that of the original out without dropout.
    # Accordingly, in testing mode, forward pass should output p times testing out so that it matches the training magnitude.
    # The above calculation however, slows down the forward pass during testing time.
    # Therefore the inverted dropout is implemented here to compensate the expected magnitude of out.

  elif mode == 'test':

    out = x

  cache = (dropout_param, mask)
  out = out.astype(x.dtype, copy=False)

  return out, cache


def dropout_backward(dout, cache):

  """
  Perform the backward pass for (inverted) dropout.

  Inputs:
  - dout: Upstream derivatives, of any shape
  - cache: (dropout_param, mask) from dropout_forward.
  """

  dropout_param, mask = cache
  p, mode = dropout_param['p'], dropout_param['mode']

  # In the case of p == 0, this layer acts as if it doesn't exist.
  if p == 0:
    return dout

  dx = None

  if mode == 'train':

    # gate: x * mask / p with (mask / p) being the local gradient
    dx = dout * mask / p

  elif mode == 'test':

    dx = dout

  return dx

```


### Convenient stacking

The following function stitches affine - ReLU - batchnorm together so that you can easily stack these convenient layers when you build you neural network.

* the 1st layer that accept input features will need to be tailored according to feature dimensions;
* no need to deploy batchnorm for every layer in between;
* where and how dropout layers are deployed depends on your architecture;
* finally, don't forget the scores-and-loss layer where you got to choose which loss function to use;


``` python

def affine_bn_relu_forward(x, w, b, gamma, beta, bn_param):

  """
  Convenience layer that performs an affine transform followed by a batch normalization followed by a ReLU activation
  Inputs:
  - x: Input to the affine layer
  - w, b: weights and bias for the affine layer
  - gamma, beta, bn_params: parameters for translation, rescaling and training/testing running

  Returns a tuple of:
  - out: Output from the ReLU
  - cache: Object to give to the backward pass
  """

  a, af_cache = affine_forward(x, w, b) # def affine_forward(x, w, b): return out, cache;

  na, bn_cache = batchnorm_forward(a, gamma, beta, bn_param) # def batchnorm_forward(x, gamma, beta, bn_param): return out, cache;

  out, relu_cache = relu_forward(na) # def relu_forward(x): return out, cache;

  cache = (af_cache, bn_cache, relu_cache)

  return out, cache


def affine_bn_relu_backward(dout, cache):

  """
  the backward process of what is defined above
  """

  af_cache, bn_cache, relu_cache = cache

  dna = relu_backward(dout, relu_cache) # def relu_backward(dout, cache): return dx;

  da, dgamma, dbeta = batchnorm_backward(dna, bn_cache) # def batchnorm_backward(dout, cache): return dx, dgamma, dbeta;

  dx, dw, db = affine_backward(da, af_cache) # def affine_backward(dout, cache): return dx, dw, db;

  return dx, dw, db, dgamma, dbeta
```
