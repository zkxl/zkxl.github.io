---
layout: post
title: ICS261 Data Structure
date: 2017-04-03 13:00:00 -0700
categories: Study-Notes
tag: Data-Structure
---
* content
{:toc}


__RECOMMEND:__ [Fundamental Data Structure](https://en.wikipedia.org/wiki/Book:Fundamental_Data_Structures)

Last Modified: 20170403

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

### Classic Use case

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

Last Modified: 201704012

## Array

#### Fixed size array

Array has its primitive implementation in C language:   
1. An array of size S is represented as a block of __S * size(int) bytes__ memory.
2. It doesn't have a __size__ pointer and therefore out-of-bound access causes __segmentation fault__.
3. __Indexing__ access A[i] translates into memory address access at (start of A) + i * size(int).
4. __Removing__ an element by memory leads to copy over all other elements after the removed one, one step ahead, leaving the array size unchanged.


__Note:__
1. Practically, it is surprisingly fast because of __good memory locality__.  
2. Another fast strategy is __lazy removal__, it simply marks the removed element as unaccessible. Indexing it no longer makes sense.

__Pros and Cons:__
1. Look up operation takes O(1) while it takes O(n) in a LinkedList.
2. Insertion takes O(n) while it takes O(1) in a LinkedList.(Not including finding the element.)


#### Dynamic array

High-level description: [Dynamic array](https://en.wikipedia.org/wiki/Dynamic_array) wraps around primitive array.  


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

  def remove(int i):
    for j in range(i + 1, self.length+1):
      self.A[j-1] = copy(self.A[j])
    self.length -= 1
    # A few things discussed below.
    # ------------------------------------
    # in case of constant being 2, the following
    # "3/2 implementation" can give a good amortized time.
    # ------------------------------------
    if 3 * length == self.size:
      memory_block B = malloc[self.size/2]
      " copy over "
    self.A = B
    self.size = self.size/2
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

The most expansive so far operation in dynamic array is "append" because it potentially takes O(N). Assuming the last append triggers array copy and thus takes O(N), in total: $$ \log_{2}^{N} $$ times of copy takes $$ O(N + n/2 + n/4 + ... 2 + 1) $$. N times of "direct" append takes O(N). Total amortized time is still O(N).


__Note:__
A trade off between potential space waste and ops efficiency. If the expansion factor shrinks from 2 (potential 50% waste) to 1.5, there will be higher constant factor placed on operation amortized time taken.

---

### __Mathematical Framework for Amortized Analysis:__
_(more example needed to make it clearer)_

__Define:__ [Potential Function](https://en.wikipedia.org/wiki/Potential_method)  $$ \Phi $$
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

Last Modified: 201704012

## Dictionary / HashMap

Let's see how it came from the most primitive abstract to the most useful data structure in modern languages.  

[Dictionary](https://en.wikipedia.org/wiki/Associative_array) is the __abstract idea__ on a collection of (k, v) pairs with one key mapping to one value.  

Essential Operations:
1. Look-up (static/dynamic dictionary).
2. Store new value [add or replace] (dynamic dictionary).
3. Remove (k, v) pair.

### Classic Use case

#### Decorator Pattern

__Description:__ For some graph algorithms, where you want to associate some info with vertices without modifying these vertices.

__Solution:__
1. Each vertex is associated with a dictionary, where (field_name, info) are stored.
2. Each field is associated with a dictionary, where (vertex, info) are stored. _Things will not screw up if field_name changes._  

### Implementation - HashMap

__Maintain:__
1. An array of size N, greater than the number of (k, v) pairs. So __load factor__ < 1.
2. A hash function.

__Hash Function:__

Regardless of the choice of hash function, the following assumption is held as standard to conclude that __dict ops are O(1): Hash function's mapping of key to index is uniformly distributed and independent of other keys__.

__Performance Analysis:__  

With:
1. An array of size N
2. A uniformly distributed hash function
3. n entries in total  


Collision occurs while looking up one of them with $$ {\frac{n-1}{N}} $$ probability. Each collision takes one comparison. So the loop-up operation takes time of (load_factor - 1/N) = O(1).

the probability of a collision is (1/N). Each collision costs one comparison.

__Collision__ is unavoidable because [birthday paradox.](https://en.wikipedia.org/wiki/Birthday_problem)

#### Different Hashing to Counter Collision

__Hash chaining__  
GET: Find bucket and match key.  
SET: Find bucket and append new key value pair.  
REMOVE: Find bucket and do the same as remove from an array.  
Lazy Remove: Find bucket, match key and mark as "unaccessible".  

``` Python
# initialization
Array<List<Pair>> H = new Array<List<Pair>> # associative list

# SET(key, value) Method - find bucket and append
for pairs in H[hash(key)]:
  if pair.key == key:
    pair.value = value
    return ;
H[hash(key)].append(new Pair(key, value))
return ;

# GET(key) Method
for pair in H[hash(key)]:
  if pair.key == key: return pair.value;
raise Exception("can't find the key.")

# Note: see why hash chaining is object-heavy? These Pairs() obj can be replaced by Tuple() and no more reduction.

# REMOVE(key)
bucket = H[hash(key)]
for i in range(len(bucket)):
  pair = bucket[i]
  if pair.key == key: # find the removable
    for j in range(i + 1, len(bucket)): # toggle everything after
      bucket[j - 1] = bucket[j] # decrement length as needed
    return

# Lazy Removal(key) requires modifications on GET and SET.
for i in range(len(bucket)):
  pair = bucket[i]
  if pair.key == key:
    pair.key = "unaccessible" # mark as unaccessible
  else: # if pair.key is a mismatch or unaccessible
    continue # skip it without decrementing length
```

__Open addressing - linear probing__  
GET: Find bucket and probe the next one to match key.  
SET: Find bucket and probe the next empty one.  
REMOVE: Roll back one at a time.
Lazy Remove: Mark "unaccessible". (Create table pollution and drag down the data structure performance as the number of deletions goes high)

``` Python
# initialization
Array<Pair> H = new Array<Pair>
int N = H.size
int n = " the number of current entries in H "
int load_factor = 0.5

# SET(key, value) Method
bucket = hash(key)
while True:
  pair = H[bucket]
  if pair.key = key: # reset value for this key
    pair.value = value
    return ;
  elif pair.isEmpty():
    if (n / N) > 0.5:
      # check to ensure the efficiency of this data structure.
      " reallocate bigger memory to H and copy over existing entries. "
    pair.key = key
    pair.value = value
    n += 1
    return ;
  else: # the bucket has been taken
    bucket = (bucket + 1) % N

# GET(key) Method
bucket = hash(key)
while True:
  pair = H[bucket]
  if pair.key == key:
    return pair.value
  elif pair.isEmpty():
    raise Exception("KeyNotFound Error")
  else:
    bucket = (bucket + 1) % N

# REMOVE(key)
bucket = hash(key)
while True:
  if (bucket + 1) % N <= bucket: # turn around
    if H[(bucket + 1) % N].key == key:
      H[bucket] = H[(bucket + 1) % N] # replace
    bucket = (bucket + 1) % N
    continue
  if H[bucket + 1].isEmpty():
    " empty H[bucket] "
    return;
  else:
    if H[bucket + 1].key == key:
      H[bucket] = H[bucket + 1]
    bucket += 1
```

__Chaining Vs. Linear Probing__:  
1. In __theory__, chaining and probing give O(1) performance when load_factor is a lot smaller than 1. However when load_factor exceeds 1, chaining still works with a slightly higher constant. But the performance of probing will be destroyed.
2. In __practice__, probing is efficient because chaining involves going from hash cell to dynamic array which is an object wrapping around array. On the other hand, probing has better __memory locality__. Random memory access is expensive compared with cached-in memory operations.
3. Linear probing requires a higher-quality hash function. In case of low-quality hash function, a big __continuous chunk of array__ will be occupied, which drags down the performance significantly.

__Linear Probing Requires High-Quality Hash Function.__ Here's why:  
If the hash function can't produce uniformly distributed results. It's possible that we have a continuous chunk of array occupied. Let's say such block i has length $$ L_{i} $$ and every operation cost is $$ O(L_{i}) $$. The cost of n such operations is:  

$$ O({\frac{1}{n}} \sum_{block_i} L_{i}^{2}) $$

Now with load_factor = $$ \alpha $$, the expected number of keys mapped to any of such blocks is: $$ \alpha L $$. According to [Churnoff Bound](https://en.wikipedia.org/wiki/Chernoff_bound), the probability (P) of mapping a key to a continuous block:  

$$ P <= ({\frac{e^{\epsilon}}{(1+\epsilon)^{1+\epsilon}}})^{\alpha L} $$

s.t:  

$$ (1 + \epsilon) \alpha = 1 $$

In case $$ \alpha = 0.5 $$, $$ P < {\frac{e}{4}}^{\frac{L}{2}} $$

__Cuckoo Hashing__

Pros:
+ constant worse-case time for lookup and delete ops.  
Cons:
+ With a load_factor < 1/2, insertion fails at probability of (1 / N^2) where N is the size of the hash table. When fail occurs, table needs to be rebuilt.  
+ Bigger load_factor causes drastic increase of insertion failures.  
+ Thus, deterministic hash functions cannot be used. Otherwise, rebuild makes no sense.  

``` Python
# Initialize two hash table with size N/2 for each
Array<Pair> H0 = new Array<Pair> with hash function h0
Array<Pair> H1 = new Array<Pair> with hash function h1

# SET(key, value)
itern = 0
while True:
  if H0[h0(key)].isEmpty():
    H0[h0(key)] = new Pair(key, value)
    return
  if H1[h1(key)].isEmpty():
    H1[h1(key)] = new Pair(key, value)
    return
  if H0[h0(key)].key == key:
    H0[h0(key)].key = key
    H0[h0(key)].value = value
    return
  if H1[h1(key)].key == key:
    H1[h1(key)].key = key
    H1[h1(key)].value = value
    return
  # if both of them are not empty but none contains the key
  # start kicking from 0
  tmp0 = H0[h0(key)]
  H0[h0(key)] = new Pair(key, value)
  tmp1 = H1[h1(key)]
  H1[h1(key)] = tmp0
  key = tmp1.key
  value = tmp1.value
  if itern > threshold: start rebuilding
  itern += 1


# GET(key)
if H0[h0(key)].key == key:
  return H0[h0(key)].value
if H1[h1(key)].key == key:
  return H1[h1(key)].value
raise Exception("Not found.")

# REMOVE(key)
if H0[h0(key)].key == key:
  "clear the cell"
  return
if H1[h1(key)].key == key:
  "clear the cell"
  return
raise Exception("Not found.")
```

__Graph Representation of Cuckoo Hashing__

Graph: G = (V, E)  
Vertices: V = table cells of H0 and H1  
Edges: E = pairs of cells connected by hashing the same key through h0 and h1  

1. Direct edges to point to where each key is stored.
2. A valid assignment is: each vertex has <= 1 incoming edge.
3. Insertion could reverse edges along a single path to try out different possible assignment.

__If and only if all the components of G is a tree or a tree + one edge (potentially one circle allowed). Otherwise, infinite loop.__

When load_factor <= 1/2, there are $$ \Theta(N) $$ small trees and $$ \Theta(1) $$ tree + one edge. When load_factor gets even slightly larger than 1/2, small trees will gradually turn into a giant component, where infinite loops arise.


#### Hash Function

1. Cryptographic hashing functions, such as SHA-256 and MD5, are a good candidate in theory. But they are too slow to be used in a data structure.
2. [Tabulation hasing.](https://en.wikipedia.org/wiki/Tabulation_hashing#Method)
3. Polynomial random linear hashing.

_Side note: MD5 stands for message digest 5._

__Tabulation Hashing:__

Quote from Wiki -  
_"Let p denote the number of bits in a key to be hashed, and q denote the number of bits desired in an output hash function. Choose another number r, less than or equal to p; this choice is arbitrary, and controls the tradeoff between time and memory usage of the hashing method: smaller values of r use less memory but cause the hash function to be slower. Compute t by rounding p/r up to the next larger integer; this gives the number of r-bit blocks needed to represent a key. For instance, if r = 8, then an r-bit number is a byte, and t is the number of bytes per key. The key idea of tabulation hashing is to view a key as a vector of t r-bit numbers, use a lookup table filled with random values to compute a hash value for each of the r-bit numbers representing a given key, and combine these values with the bitwise binary exclusive or operation. The choice of r should be made in such a way that this table is not too large; e.g., so that it fits into the computer's cache memory."_ [[1]](https://books.google.com/books?id=vMqSAwAAQBAJ&pg=SA11-PA3#v=onepage&q&f=false)

__Polynomial random linear hashing:__

For a hash table with size N,  

Set a large prime number P (P >> N) so that hash(x) = (x % P) % N is close to being uniformly distributed.  

Set a small positive integer d as polynomial degree and generate a set of random coefficients $$\alpha_i = \alpha_i\;mod\;P $$ so that:  

$$ hash(x) = ((\sum_{i=0}^{d} x_i \alpha_i)\;mod\;P)\;mod\;N $$

where, if two keys: k1 and k2 have the same digest, $$ \sum_{i=0}^{d} x_i \alpha_i $$ is still uniformly distributed.  

which means:  

$$ Pr(collision) = 1/N $$

Note: P is either a random large number or a fixed number larger than the maximum key to ensure the above properties.  

__Introduce K-independent Hashing:__

The concept was derived from the above polynomial random linear hashing function. Such a function with degree d is viewed as d-independent hashing function.  

Formally defined as: K-independent hash function is a family of hash functions that can map any set of K keys to a set of K independent random variables.

Quote from Wiki -  
_"The original technique for constructing k-independent hash functions, given by Carter and Wegman, was to select a large prime number p, choose k random numbers modulo p, and use these numbers as the coefficients of a polynomial of degree k whose values modulo p are used as the value of the hash function. All polynomials of the given degree modulo p are equally likely, and any polynomial is uniquely determined by any k-tuple of argument-value pairs with distinct arguments, from which it follows that any k-tuple of distinct arguments is equally likely to be mapped to any k-tuple of hash values."_ [[2]](http://www.fi.muni.cz/~xbouda1/teaching/2009/IV111/Wegman_Carter_1981_New_hash_functions.pdf)



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
