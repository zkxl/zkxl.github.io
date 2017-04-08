---
layout: post
title: ICS261 Data Structure
date: 2017-04-03 13:00:00 -0700
categories: Study-Notes
tag: Data-Structure
---
* content
{:toc}



Last Modified: 20170407

## Stack

#### Object-oriented Implementation

High-level description:
1. A stack is a collection of __objects__.
2. Each object has a value pointer and a next pointer.
3. __Value pointer__ points to the value of the object while the __next pointer__ points to the object that came previously.
4. The stack data structure has a __top pointer__ points to last came-in object.

``` python
class StackNode():
  def __init__(self, val):
    self.val = val
    self.next = None

class Stack():
  def __init__(self):
    self.top = NULL
    # the whole class is just a pointer to NULL upon initialization

  def push(self, val):
    node = StackNode(val)
    node.next = self.top
    self.top = node

  def pop(self):
    val = self.top.val
    self.top = self.top.next
    return val
```

__Analysis:__
1. No loop no subroutines. O(1) to push or pop.

__NOTE:__
1. __Locality__ is not good because every object is a node randomly placed in memory.
2. __Persistence:__ this implements stack as a [persistent data structure](https://en.wikipedia.org/wiki/Persistent_data_structure).  
Basically, the next state of the data structure is built on the current state. No in-place modification occurs. Each state can be preserved.  
For example, as long as you keep a pointer to some node in state A, even though some subroutines modified the stack to state B, you can still refer back to the node in state A as if nothing has been modified.


#### Array-based Implementation

High-level description:
1. A stack is an array, assuming the array size is sufficiently large within our scope.
2. Maintain a __top indexer(int)__ pointing at the last came-in value in the array.

``` python
class Stack():
  def __init__(self):
    self.A = [LARGE]
    self.top = 0

  def push(self, val):
    self.top += 1
    self.A[self.top] = val

  def pop(self):
    val = self.A[self.top]
    self.top -= 1
    return val
```

__Analysis:__
1. No loop no subroutines, O(1) to push or pop.
2. Theoretically 2 times more efficient in space usage because every value comes in with a pointer in object-oriented implementation.

__Note:__
1. Memory locality is good.
2. Potential memory usage waste is the only downside here. It depends on how good your estimate is.
3. Minor: array indexing access is a little bit more efficient than pointer access.

### Use case

#### Nearest neighbor chain algorithm:

Naive Hierarchical Clustering takes O(n^2) to decide the next pair of clusters to merge.  

Stack-based Hierarchical Clustering takes O(n). Here's the magic:  

```
S = new Stack()
while the number of clusters > 1:
  if S.isEmpty(): S.push(some arbitrary cluster centroid)
  c = the most top value on stack // O(1)
  n = the nearest neighbor of c // O(n)

  // checking if they are the same point can be done in O(1) with some optimization
  if S has some second to the most top value and n equals to this value:
    c = S.pop()
    n = S.pop()
    merge(c, n) // re-compute centroid
  else:
    S.push(n) // go to next loop, check n's nearest neighbor
```

__Note:__
1. This is a tricky algorithm to think about, you'd better do a hand stimulation by drawing a sequence of points on a line with decreasing distances between one and the next.

---

## Array

#### Fixed size array

Array has its primitive implementation in C language:   
1. An array of size S is represented as a block of __S * size(int) bytes__ memory.
2. It doesn't have a __size__ pointer and therefore out-of-bound access causes __segmentation fault__.
3. Indexing access A[i] translates into memory address access at __(start of A) + i * size(int)__.

__Pros and Cons:__
1. Look up operation takes O(1) while it takes O(n) in a LinkedList.
2. Insertion takes O(n) while it takes O(1) in a LinkedList.(Not including finding the element.)


#### Dynamic array

High-level description: Dynamic array wraps around primitive array.  

``` Python
class DynamicArray():
  def __init__(self):
    memory_block self.A = malloc[initial_size]
    int self.size = initial_size # size pointer
    int self.length = 0 # actual length pointer
    # data struc invariant : length <= size

  def get(int i): # indexing access at i
    if i >= self.length: raise Exception("Out of bound.") # check index within bound
    return self.A[i] # translates to memory address access

  def put(int i, int val):
    if i >= self.length: raise Exception("Out of bound.") # check index within bound
    self.A[i] = val

  def append(int val):
    if self.length == self.size:
      memory_block B = malloc[constant * self.size]
      for i in range(self.length): # copy over
        B[i] = self.A[i]
      self.A = B
      self.size = constant * self.size
      # canonical expansion: size = 2 * size (potentially 50% memory waste)
      # expansion in Java: size = 1.5 * size
      # constant in Python: size = size + size >>> 3

    self.A[self.length] = val # append
    self.length += 1

  def pop():
    if self.length <= 0: raise Exception("Nothing to pop from empty array.")
    return self.A[self.length]

  def remove():
    # A few things discussed below.
    # ------------------------------------
    # in case of constant being 2, the following
    # "3/2 implementation" can give a good amortized time.
    # ------------------------------------
    # if 3 * length == self.size:
    #   memory_block B = malloc[self.size/2]
    #   copy over
    # self.A = B
    # self.size = self.size/2
    return ;

```
* See amortized analysis for "3/2 implementation" in Mathematical Framework for Amortized Analysis.  

Removing an element from an array:  
1. takes O(n) if you create a new array and copy everything over except the target.  
2. takes O(n) if you move everything after the target upfront by 1 index.
3. takes O(1) if you don't care about its sequence, you can swap the target with the last element and do a pop.
4. takes O(1) if you'd rather mark the target as "unaccessible" than actually freeing up the memory.


__Amortized Analysis:__

If we can prove, any sequences of N operations on this data structure is in O(N), we can say that such an operation takes amortized O(1) time.  

The most expansive so far operation in dynamic array is "append" because it potentially takes O(N). Assuming the last append triggers array copy and thus takes O(N), in total:  

$$ \log_{2}^{N} $$ times of copy takes $$ O(N + n/2 + n/4 + ... 2 + 1) $$. N times of "direct" append takes O(N). Total amortized time is still O(N).


__Note:__
A trade off between potential space waste and ops efficiency. If the expansion factor shrinks from 2 (potential 50% waste) to 1.5, there will be higher constant factor placed on operation amortized time taken.

---

### __Mathematical Framework for Amortized Analysis:__
_(more example needed to make it clearer)_

__Define Potential Function__ $$ \Phi $$
1. map states of data structure to numbers.
2. must be 0 upon the initialization of the data structure.
3. no smaller than 0 after initialization.

__Define: Amortized Time__ = actual time + $$ C * (\Phi_{i} - \Phi_{j}) $$

1. _actual time: each operation's actual time cost._
2. _C: constant, arbitrarily large to make the amortized time ideal._
3. $$ \Phi $$ _: potential function._

With all the above constraints, we can say: __No matter how we choose potential function and C, we have: total time <= the sum of all operations' amortized time__ = time(ops1) + C($$ \Phi_1 - \Phi_0 $$) + time(ops2) + C($$ \Phi_2 - \Phi_1 $$) + ... + time(opsn) + C($$ \Phi_n - \Phi_{n-1} $$).

__In the case of dynamic array:__  
Potential function = max(0, 2 * length - size)

|                    | actual    | change        | total
|--------------------|-----------|---------------|------
| remove             | O(1)      | -2            | O(1)
| add_non-reallocate | O(1)      | +2            | O(1)
| add_reallocate     | O(length) | 2 - length    | O(1)
| "3/2" removal      | O(length) | 2 + 3/2 length| O(1)

* __add_reallocate:__ 2 - length = ((2 * length - new size before operation) - (2 * length - old size after operation) + 2) = (0 - length) + 2. This operation is only performed when length == size.
* __"3/2" removal:__ 2 - length = ((2 * length - new size before operation) - (2 * length - old size after operation) + 2) = (1/2 * length - (- length)) + 2

In the above case, as long as we choose C greater than the constant in O(length), we can have O(1) total time.  

If we define: $$ \Phi = 0 $$ means the data structure is in an ideal state. $$ \Phi $$ now has concrete meaning instead of an abstract mathematical measure - the distance from ideal state.

---

<!--
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
-->
