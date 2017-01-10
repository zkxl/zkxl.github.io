---
layout: post
title: linear-classifier svm-sfmx-loss
date: 2016-12-15 15:19:00 -0800
categories: deeplearning-basics 
tag: svm-softmax
---

* content
{:toc}



__This deeplearning series is inspired by Stanford open course — cs231n. My special thanks.__


_Deeplearning is involved in a wide range of techniques that allow a neural network to make reliable predications. This series will introduce several key components of a neural network, from linear classifier loss functions to fully connected neural nets, from convNets to RNN._


### SVM loss - 

$$ L_i = \sum_{j\neq y_i} \max(0, s_j - s_{y_i} + \Delta) $$

``` python

def svm_loss_vectorized(W, X, y, reg):
    """
    Structured SVM loss function, vectorized implementation.

    Inputs have dimension D, there are C classes, and we operate on minibatches
    of N examples.

    Inputs:
    - W: A numpy array of shape (D, C) containing weights.
    - X: A numpy array of shape (N, D) containing a minibatch of data.
    - y: A numpy array of shape (N,) containing training labels; y[i] = c means
      that X[i] has label c, where 0 <= c < C.
    - reg: (float) regularization strength

    Returns a tuple of:
    - loss as single float
    - gradient with respect to weights W; an array of same shape as W
    """
    loss = 0.0
  
    dW = np.zeros_like(W) # initialize the gradient as zero

    N, _, _, C = X.shape, W.shape

    scores = np.dot(X, W) # (N, C)

    correct_scores = scores[np.arange(N), y]
    correct_scores = np.dot(correct_scores.reshape(N, 1), np.ones((1, C))) # (N, C)

    # NOTE: mask scores matrice with [np.arange(num_train) in axis=0, y in axis = 1]
    #       its shape got reduced to 1D (num_train, ) array and then reshape to 2D (num_train, 1) array

    delta = np.ones((N, C))
 
    L = np.fmax(0, scores - correct_scores + delta)
    # everything between +/- is in (N, C)
    # np.fmax gives element-wise comparison

    L[np.arange(N), y] = 0 # correct scores don't count into loss

    loss = np.sum(L) / N + 0.5 * reg * np.sum(W * W)

    # gradients dW is calculated as follows:

    L[L > 0] = 1 # these contribute to the loss

    # for each instance i in range(N):
    #     for each class in range(C):
    #         if L[i, j] == 1 contributing to the loss:
    #             L[i, y_i] -= 1, the right class score got boosted.

    L[np.arange(N), y] = - np.sum(L, axis = 1)


    dW = np.dot(X.T, L) # NOTE: quite neat!
   
    dW = dW / N + reg * W

    return loss, dW

```

### softmax loss - 

$$ L_i = -\log\left(\frac{e^{f_{y_i}}}{ \sum_j e^{f_j} }\right) \hspace{0.5in} \text{or equivalently} \hspace{0.5in} L_i = -f_{y_i} + \log\sum_j e^{f_j} $$

```python
def softmax_loss_vectorized(W, X, y, reg):

    """
    Softmax loss function, vectorized implementation.

    Inputs have dimension D, there are C classes, and we operate on minibatches
    of N examples.

    Inputs:
    - W: A numpy array of shape (D, C) containing weights.
    - X: A numpy array of shape (N, D) containing a minibatch of data.
    - y: A numpy array of shape (N,) containing training labels; y[i] = c means
      that X[i] has label c, where 0 <= c < C.
    - reg: (float) regularization strength

    Returns a tuple of:
    - loss as single float
    - gradient with respect to weights W; an array of same shape as W
    """

    loss = 0.0
    dW = np.zeros_like(W)

    N, _ = X.shape
    _, C = W.shape

    scores = np.dot(X, W)

    # avoid numerical unstability due to large number exponential
    scores = scores - np.max(scores, axis = 1).reshape(N, 1)

    sum_exp_scores = np.sum(np.exp(scores), axis = 1).reshape(N, 1)

    possibility_scores = np.exp(scores) / sum_exp_scores # broadcast

    # softmax loss:
 
    loss = np.sum(- np.log(possibility_scores[np.arange(N), y]))

    loss /= N

    loss += 0.5 * reg * np.sum(W * W)
    
    # dW:
    # affine transform -> scores = np.dot(X, W) - for each instance, sum up weights from all features
    # backprop -> dW = np.dot(X.T, dscores) - for each feature, sum up gradients from all instances
    dW = np.dot(X.T, possibility_scores)
    
    y_mask = np.zeros((N, C))

    y_mask[np.arange(N), y] = 1

    dW -= np.dot(X.T, y_mask)

    dW /= N

    dW += reg * W

    return loss, dW

```
