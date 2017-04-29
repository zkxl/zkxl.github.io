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

``` python
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


``` python
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

``` python
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

``` python
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

``` python
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

__Note:__ If you want it to work without half space wasted, a trick you can do is let every array cell stores a constant number of (k, v) pairs, which differs from hash chaining in that only constant number of pairs are allowed so it is still constant time access in a sense.  


#### Hash Function

1. Cryptographic hashing functions, such as SHA-256 and MD5, are a good candidate in theory. But they are too slow to be used in a data structure.
2. [Tabulation hasing.](https://en.wikipedia.org/wiki/Tabulation_hashing#Method)
3. Polynomial random linear hashing.

_Side note: MD5 stands for message digest 5._

__Tabulation Hashing:__

Quote from Wiki -  
_"Let p denote the number of bits in a key to be hashed, and q denote the number of bits desired in an output hash function. Choose another number r, less than or equal to p; this choice is arbitrary, and controls the tradeoff between time and memory usage of the hashing method: smaller values of r use less memory but cause the hash function to be slower. Compute t by rounding p/r up to the next larger integer; this gives the number of r-bit blocks needed to represent a key. For instance, if r = 8, then an r-bit number is a byte, and t is the number of bytes per key. The key idea of tabulation hashing is to view a key as a vector of t r-bit numbers, use a lookup table filled with random values to compute a hash value for each of the r-bit numbers representing a given key, and combine these values with the bitwise binary exclusive or operation. The choice of r should be made in such a way that this table is not too large; e.g., so that it fits into the computer's cache memory."_ [[1]](https://books.google.com/books?id=vMqSAwAAQBAJ&pg=SA11-PA3#v=onepage&q&f=false)

__Here is an example of how it works:__  
The hash function h(x) maps 32-bit keys to 16-bit hash values by breaking each key into four 8-bit bytes, using each byte as the index into four tables of 16-bit random numbers, and returning the bitwise exclusive or of the four numbers found in this table.

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


## Set

Last update: 20170419

__Standard Operations:__
1. test membership - always
2. add/remove - usually
3. union/intersect - sometimes

__Union/Intersect Operations can be easily implemented through 1 and 2.__  

``` python
def Union(S1, S2):
  C = deepcopy(S1)
  for element in S2:
    C.add(element)
  return C

def Intersect(S1, S2):
  C = emptySet
  for element in S1: # if S1 is shorter
    if element in S2:
      C.add(element)
  return C
```

__The chart shows how efficient test/add/remove ops will be under different Implementations.__

* n denotes the number of current keys in the SET.  
* U denotes the number of all possible keys that could go into the SET.

| Implementation     | Time      | Space         
|--------------------|-----------|---------------
| HashTable          | O(1)      | O(n) keys / O(nlogU) bits ?
| BitVector          | O(1) fast | O(U) bits
| BloomFilter        | O(1)      | O(n)

#### BitVector Implementation

If you want to represent a SET of integers {0, 1, 2 ... 31}, you can use a single 32-bit integer S to do it, with each bit 1 or 0 denoting corresponding element's membership. Such that:  

``` python
def isMember(x, S):
  return (S & (1 << x) != 0)

def add(x, S):
  S = S | (1 << x)

def remove(x, S):
  S = S & ~(1 << x)  
```

__Note__ two obvious constraints:  
1. the number of elements representable is limited by [word size](https://en.wikipedia.org/wiki/Word_(computer_architecture)).
2. the type of elements can only be integer.

#### Bloom Filter

[Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter) is a time/space efficient data structure when small chances of false positive is not a big issue.  

__Here is how it is designed:__

Create an bitArray of size (C * n) where C is a small constant and n is the number of keys you want to store.

Create k independent hash functions mapping each key to k bits.

What does the following mean?  
hash one time. key -> (klog(Cn)) bits - then split values to {h1, h2, ..., hk}  

``` python
# B = BloomFilter()
def isMember(x, B):
  for i in range(k):
    h = H[i] # think H as a mapping between i and the hash function.
    if B[h(x)] is False: return False
  return True

def add(x, B):
  for i in range(k):
    h = H[i] # think H as a mapping between i and the hash function.
    B[h(x)] = 1
```

__Note:__ When checking membership of some element x in a bloomFilter, there is a chance (less than $$ ({\frac{k}{C}})^{k} $$) that false positive occurs. That is returning True with a key that's not in BF.


Last update: 20170419

#### Cuckoo Filter

Similarity with Bloom Filter:  
1. constant rate of false positive.
2. linear number of bits space consumed.  

Difference from Bloom Filter:
1. increasing number of entries will not affect false positive rate  
2. allow deletion (because it uses a lookup table implementation)
3. better reference locality. WHY? BF only needs one array.

__Implementation:__
1. create two tables as they are in Cuckoo hashing.
2. hash the key to find its location in 1st table and put in a $$ \log_{2}^{\frac{1}{\epsilon}} $$ bits checksum (epsilon is the false positive rate). (there is a chance when different keys have the same checksum)
3. when kicking a checksum from 1st table to 2nd table, use the following to compute its location in 2nd table: original_location XOR hash(checksum). (Use XOR so that when the 2nd cell is found to be filled, using the above formula can throw it back to the same cell in 1st table.)


#### Inverted Bloom Filter

The use case of IBF is typically finding the small differences between two large sets. And this data structure is specifically tailored for that purpose.  

Similarity with Bloom Filter:
1. false positive rate? why would it work if false positive still possible.
2. linear bits space consumed ? not implemented by bits.

Difference from Bloom Filter:  
1. allow deletion.
2. designed to list remaining element.
3. For a fixed space, arbitrary number of entries are allowed, because the whole point of this data structure is not membership testing or whatsoever typically in a SET data structure.  

__Use case - "Set Reconciliation" - Distributed Version Control__

1. add every line in textA as an element into IBF
2. for every line in textB, remove it from IBF.
3. list the remaining element in IBF - the different content that A has but B doesn't.
4. repeat the above process to find out the different content that B has but A doesn't.

__Implementation:__

Each cell in IBF contains (x, y, z) where: WHY do we need these three info?  
1. x : the number of elements in this cell.
2. y : the sum of elements in this cell.
3. z : the sum of checksum of each element in this cell.

* "Pure cell" : the cell that has only one element within it.


Last Modified: 201704024

### Streaming Algorithm

__Characteristics:__ [Streaming algorithm](https://en.wikipedia.org/wiki/Streaming_algorithm) computes some non-trivial properties of the input sequence in a single path using constant space.

#### Warm up - Majority Vote Algorithm

__Problem:__ Find the majority out of a sequence of values.

__Tentative Solutions:__
1. HashTable, taking O(n) time and O(n) space.
2. Double loop with global optimal, taking O(n^2) time and O(1) space.

__Majority Vote Algorithm:__

``` python
def majorityVote(seq):
  majority = Pair(value == None, counter = 0)

  for x in seq:
    if majority.counter == 0:
      majority.value = x
    if majority.value == x:
      majority.counter += 1
    else:
      majority.counter -= 1

  return majority.value
```

__What the above algorithm says is:__ If there is no majority (counter == 0), whatever comes along next is the become the majority.

__Note:__ Given any input sequence without a clear majority, such as (1, 1, 2, 2, 3), the algorithm will not return anyone with the highest count.

__Claim:__
1. If there is a clear majority in input sequence of n elements, there are at most (n/2) times of counter decrements.

2. $$ O_{x} - {\frac{n}{2}} $$ <= estimate(x) <= $$ O_{x} $$ ($$ O_{x} $$: number of actual occurrences of elements x)

$$ estimate(x) = {\begin{cases}  majority.counter & {\text{ if }} x = majority.value \\ 0 & {\text{ otherwise }} \end{cases}} $$

__Generalization of Majority Vote Algorithm - estimate the number of occurrences of each element within some error range.__

For example, given a sequence of input, find the most majority one, the second to the most majority, the third ..., the kth.

``` python
def kthMajorityVote(seq, k):

  # some special data structure is needed for this algorithm to work
  # assuming we have such abstract data structure that:
  # 1. holds k pairs of (value, counter) subject to in-place modification
  # 2. O(1) lookup time
  # 3. return the next available pair whose counter == 0
  ads = AbstractDataStructure()

  for x in seq:
    if x not in ads and ads.next().counter == 0:
      ads.next().value = x
    if x in ads:
      increment counters of all pairs in ads
    else:
      decrement counters of all pairs in ads

  return ads
  # now ads holds counter info of each value in seq that we can use to estimate the actuall occurrences of each value.
```

__Claim:__ With the same estimate function, we have:  

$$ O_{x} - {\frac{n}{k}} $$ <= estimate(x) <= $$ O_{x} $$  

Meaning: we have an estimated occurrences within (n/k) error range for each element in sequence.

#### Counting BloomFilter

1. Instead of using bitArray, CBF uses IntArray to accumulate the frequencies of keys checking in and out. (size = const * number of keys)
2. With a set of K hash functions, every key corresponds to K different cells.
3. Insertion(item) increments corresponding cells.
4. Deletion(item) decrements corresponding cells.
5. Membership(item) tests if all corresponding cells are non-zero.

#### Count Min Sketch

[Count Min Sketch](https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch) __is built based on CBF with 2 differences:__
1. Essentially both can be implemented with 1D vector. But for the sake of clarity, CMS is often implemented with 2D matrix.
2. With fewer total cells, this data structure can gives us better approximation of frequencies of items, given that they constantly checks in and out in the input stream.

__Implementation Details:__
1. A 2D array to store a counter in each cell: table = int[w][k], where:  
* $$ w = {\frac{e}{\epsilon}} $$
* $$ \epsilon $$ defines the estimate accuracy.  
* e is natural logarithm base.  
* $$ k = \ln({\frac{1}{\delta}}) $$  
* $$ \delta $$ defines the probability of NOT getting that accuracy.  

2. Again, k hash functions, mapping each key to [one cell in each row for every row in the table].

``` python
# with k, table, hashFunctions() defined as above
def update(key, x):
  # checkin: x positive
  # checkout: x negative
  for i in range(k):
    table[i][hash_i(key)] += x
  return ;

def approxFreq(key):
  # get the estimated frequencies of the key
  return min([table[i][hash_i(key)] for i in range(k)])
```

__Analysis:__
1. The _approxFreq(key)_ defined above returns an estimated frequency of the key, which = [actual frequency] + [noise from other keys].
2. Let n be the length of the CMS, which is the number of remaining items. $$ E[noise] \leq {\frac{n}{w}} $$
3. [Markov's Inequality](https://en.wikipedia.org/wiki/Markov%27s_inequality) brings it the following: The probability of [noise in one row $$ \geq \epsilon n $$] $$ \leq {\frac{1}{e}} $$
4. So: The probability of [such big noise in all rows] <= $$ {\frac{1}{e}}^{k} = \delta $$ This is the probability of NOT getting that pre-defined accuracy.

__Note:__
The idea behind the design of the data structure is the space taken by all the cells (w * k) should be much smaller than the what we need to store exactly everything in the input stream.

__Use Case: compute document similarity__

1. In __"Bag of Words" model__, each document corresponds to a long vector, storing the frequencies of all the words in our vocabulary.
2. __Cosine Similarity__ between doc A and doc B is computed as: _dotProduct(vectorA, vectorB) normalized by product(norm2_lengthA, norm2_lengthB)_, which is also the angle [0, 90] of the two vectors in vector space. $$ norm2_length\;of\;\overrightarrow{\rm v} = \sqrt{ \overrightarrow{\rm v} \cdot \overrightarrow{\rm v}} $$
3. With __vector generalization of CMS__, for each document, let words stream through CMS, now each document corresponds to a CMS table.
4. Approximate similarity between docA and docB by computing:  
* dotProduct(row_i_of_cmsA, row_i_of_cmsB) for each row
* return the minimum value from them.  
5. The returned value will be an estimate of the dotProduct(vectorA, vectorB) in "Bag of Words" model. The probability of getting the estimate within [actual, actual + $$ \epsilon $$ cmsA.length cmsB.length], where cmsA.length is the sum of the absolute value of each cell in that row in cmsA.

__Note:__ When we compute the cosine similarity between two documents, we used their norm2 lengths. However, when we approximate the similarity between two documents __where a lot of words occur only once__, the accuracy deviation is proportional to the number of words, which is large.

__Review and Compare IBF, MajorityStream, CMS__

1. IBF allows stream in and out, returns very few remainings eventually.  
2. MajorityStream allows stream in only, uses constant space and always underestimates frequencies of any item.    
3. CMS allows stream in and out, gives overestimated frequencies of any item in a relatively efficient way of using limited space.

#### MinHash

The use of CMS in computing documents similarity help reduce the computation complexity and space taken because:  
1. Without CMS, each document corresponds to a words-frequencies table and you have to compare it with other words-frequencies tables.
2. With CMS, you reduce the space usage to constant and further simplify the computation.

Another way to measure documents similarity is __Jaccard Similarity__:    
$$ Jaccard(A, B) = \frac{\| A \cap B}{abs(A \cup B)} $$

where A is a set of words occurred in docA and B is a set of words occurred in docB.  
This way, we only need to store boolean value for each word instead of frequencies. __Our question__ is: any way to reduce the space usage of the SET data structure in this case?  

__Consider the following case:__  
1. Stream all the words from docA and docB through some hash function.
2. Represent the set of "words-frequencies" by the minimum hash value throughout the stream.
3. If the minimum hash value from streamA == the minimum hash value from streamB, the estimated similarity(A, B) = 1. Otherwise, 0.
4. This way, a random hash function makes a random estimator of their similarity.
5. __claim:__ Expected[estimated similarity] approximate J(A, B).

__Multi-hash version MinHash__ uses K hash functions and represents the set of "words-frequencies" by a collection of K minimum hash values.

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
