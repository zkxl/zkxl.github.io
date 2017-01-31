---
layout: post
title: Algorithm Design - ICS260
date: 2017-01-30 16:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}


## Stable Marriage

#### Problem: How to compute a stable perfect matching?

Key Definitions:
* matching?  
_Given M = {m1, m2, ... mn} and W = {w1, w2, ..., wn}, a matching is a subset of M x W (cartesian product), S, such that each element of M and each element of W appears in_ __AT MOST__ _one pair in S._

* perfect match?  
_In a perfect matching S, each element of M and each element of W appear in_ __EXACTLY ONE__ _pair in S._

* instability? stable perfect match?  
_In a matching S, instability means there exists a pair (m, w) that's not in S but each of m and w prefers the other to their current partners in S_

---

#### G-S Algorithm

```
While there’s a free man // indicating he has not proposed to every woman
  m = such man // non-deterministic
  w = the highest ranking women in m’s preference list to whom m has not yet proposed
  if w is free
    (m, w) becomes engaged
  else  // w is currently engaged to some other man m'
    if w prefers m to m'
      (m, w) becomes engaged
      m' becomes free
    else
      (m', w) remain unchanged
      m remain free
    endif
  endif
endwhile
```

#### Claims:  

1. The sequence of partners to whom a man proposes gets worse; the sequence of partners to whom a woman is engaged gets better.

2. The algorithm terminates after at most n^2 proposals.

3. If a man m is free, there is some woman to whom he has not yet proposed.  
\- _proof by contradiction:_  
\- _suppose that m is free and he has proposed to all women._  
\- _then all women are engaged to some men._  
\- _since |M| == |W|, these means all men are engaged._  

4. They matching computed by the G-S algorithm is a perfect matching.  
\- _proof by contradiction:_  
\- _suppose that the algorithm terminates without a perfect matching, meaning there is a free man m._  
\- _according to the code, m must have proposed to every woman._  
\- _according to the claim 3, it is impossible._  

5. The matching computed by the G-S algorithm is a stable matching.  
\- _proof by contradiction:_  
\- _suppose that there is an instability, meaning that the matching contains two pairs (m, w) and (m', w') such that m prefers w' to w and w' prefers m to m'._  
\- _the last time m proposed, it was to w._  
\- _if m proposed to w' some earlier time, w' rejected m and she prefers m' to m, contradiction._  
\- _if m proposed to w' some later time, m doesn't prefer w' to w._  

#### Determinacy Analysis in G-S Algorithm

Q: The non-deterministic G-S algorithm however produces deterministic result, given any particular input. Why?

A: The G-S algorithm always pairs each man with his best valid partner.  
\- _proof by contradiction:_  
\- _suppose at least one proposal to a valid partner is rejected(either immediately or as a result of a broken engagement.), consider that the_ __first rejection__ _: m proposes to a valid partner w and w immediately rejects m for m'._  
\- _there exists some woman w' that leaves (m, w) and (m', w') in a stable matching X._  
\- __However__, _X cannot be stable because of the following claims:_  
\- _w prefers m' to m. [w rejected m for m']_  
\- _m' prefers w to w'. [If he preferred w' to w, he would have proposed to w' first in G-S algorithm and end up being rejected before he proposes to w. (Not until then, can w be engaged with m' and w later rejects m, which contradicts our_ __"first rejection"__ _)]_  

#### Complexity and Implementation

Input: two 2D matrices (man, preference_list) and (woman, preference_list)  

Output: a stable perfect matching S.  

The algorithm terminates at most n^2 proposals, looping over n^2 times. If every operation in this loop is O(1), the overall complexity will be O(n^2).  

Specifically:  

1. find a free man - keep only free man on a linked list so it takes O(1) to access the next free man;  

* It takes O(n) to convert an array into a linkedlist;  

2. for a given man m, find his next highest ranking woman w to whom m hasn't proposed - maintain a 1D array h, where h[m] == w;  

3. For a given woman w, find out if she's currently engaged or not - maintain a 1D array c, where c[w] == null if w is not engaged and c[w] == m' if w is engaged with m'.  

4. For a given woman w engaged with m', to whom m proposed, determine whether w prefers m to m' - this is tricky to do in O(1) because normally you'd need to reference to w's preference_list and find how m ranks to m'. __Solution: from woman's preference array, you need to create a "ranking" array r where r[w, m] = m's ranking in w's preference_list. This can be done in O(n^2).__  

---

## Binary Heap Implementation of Priority Queue

#### Notes

1. __O(logn) for all add/delete/min operations and O(n) for heapify()__  
2. __Binary Heap is an implicit data structure, meaning:__  
  * No extra space required beyond what is needed to store an array.  
  * Extra information is how items are order in the array.  

---
#### Implementation Generals


1. Given a list of items to initiate the heap structure, calling __\_siftDown()__ from the last item to the first item gives O(n) complexity.  
\- _proof:_  
\- _given an array of length N. N is also the total number of nodes on the implicit tree, whose root has a height of floor(log(N))._  
\- _For any node at height of j, at most j siftDown() operations needed to find its correct place._  
\- _For any node at height of j, aka at depth of [(floor(logN) - j]. There are:_ $$ 2^{floor(logN) - j} $$ _such nodes at the same level._  

$$ 2^{floor(logN) - j} < O(\frac {n} {2^j}) $$  

$$ Total Time \le O(\sum_{j=0}^{logN} j \frac {n} {2^j}) \le O(N \sum_{j=0}^{\infty} \frac {j} {2^j}) = O(n) $$  


2. _use siftUp() to insert elements._  
3. _to delete an item, substitute it with the last item in the array and call siftDown() on the item._  
4. _pop out min() value is simply call delete() on the first item._  

---
#### Implementation Details

``` python
class MinHeap(object):
	# minHeap - k < 2k + 1 and k < 2k + 2 - support heapify(), push(), pop(), top(), isEmpty(), len() methods etc.
  def __init__(self, nums):
		self.size = len(nums)
		self.array = [num for num in nums]
		self.heapify()


	def heapify(self):
		if self.size == 0:
			return ;
		# _siftUp N elements takes O(NlogN)
		# _siftDown N elements takes O(N)
		for i in xrange(self.size - 1, -1, -1):
			self._siftDown(i)
		return ;

	def _siftUp(self, k):
		# when k == 3 || 4, (k - 1) / 2 == 1 holds for both values
		while k > 0:
			# if k > 0 but it can no longer siftup: break
			if (k - 1) / 2 < 0 or self.array[(k - 1) / 2] <= self.array[k]:
				break

			self.array[(k - 1) / 2], self.array[k] = self.array[k], self.array[(k - 1) / 2]
			k = (k - 1) / 2

			# reversely:
			# if (k - 1) / 2 >= 0 and self.array[(k - 1) / 2] > self.array[k]:
			# 	self.array[(k - 1) / 2], self.array[k] = self.array[k], self.array[(k - 1) / 2]
			# 	k = (k - 1) / 2
			# else:
			# 	break

		return ;

	def _siftDown(self, k):
		while 0 <= k < self.size:
			idx_of_min = k # assume k is the smallest among itself and its children
			if 2 * k + 1 < self.size and self.array[2 * k + 1] < self.array[idx_of_min]:
				idx_of_min = 2 * k + 1
			if 2 * k + 2 < self.size and self.array[2 * k + 2] < self.array[idx_of_min]:
				idx_of_min = 2 * k + 2
			if idx_of_min == k: # no need to change
				return ;
			else:
				self.array[k], self.array[idx_of_min] = self.array[idx_of_min], self.array[k]
				k = idx_of_min
		return ;


	def add(self, num):
		# add cost - O(logn)
		self.size += 1
		self.array.append(num)
		self._siftUp(self.size - 1)
		return ;

	def popMin(self):
		# a minHeap pops out the min value
		return self.delete(0)


	def delete(self, idx):
		# delete operation takes in the idx of the element you want to delete
		# different from adding element taking in its value
		# if finding the element takes more than O(1), delete operation would take more than O(nlogn)
		# after find the element, it's either going to be siftDown or Up.
		# delete cost - O(logn)
		ret = self.array[idx]
		self.array[idx] = self.array[self.size - 1]
		del self.array[self.size - 1]
		self.size -= 1
		if idx == 0 or self.array[idx] > self.array[(idx - 1) / 2]:
			self._siftDown(idx)
		else:
			self._siftUp(idx)
		return ret
```

---
## Graphs

#### Key Definitions

1. G = (V, E): a __graph__ is a pair of vertices and edges. Vertices are nodes and an edge is a pair of nodes e = (u, v).  
2. __Directed__ graph imposes an order on each edge while __undirected__ graph doesn't.  
3. An __undirected__ graph is connected if, for every pair of nodes (u, v), there exists a path from u to v.  
4. A __directed__ graph is __strongly connected__ if, for every pair of nodes (u, v), there exist a path from u to v and a path from v to u.  
* __Other relevant: Path, Simple Path, Cycle, Degree of a node...__  

5. The __distance__ from u to v is the minimum number of edges in a u-v path.  
6. A __free tree__ is an undirected connected graph that doesn't contain a cycle.
7. A __rooted tree__ is a tree with a root node r and all the edges are directed away from the root.  
* __Other relevant: parent, child, ancestor/descendant, proper ancestor/descendant, leaf...__  
---

#### Existence and Reachability Problem

__Existence problem: given a graph and a pair of nodes in the graph, (v, u), if there exists such a path from u to v?__  

__Reachability problem: given a graph and a node u, find all nodes which there exists a path from u to.__  
---

#### Graph Representation

Given G = (V, E), n = # of nodes, m = # of edges.

1. __Adjacency Matrix:__ Array A[n, n] where A[u, v] = 1 if (u, v) is an edge of G, else 0.  
* O(1) lookup time for an edge;
* O(n^2) space needed;
* O(n) time to find all edges on a given node.

2. __Adjacency List:__ Array A[n] where A[v] contains a list of all the neighbor nodes to v.  
* O(n+m) space needed;
* O(1) time to find all edges on a given node;

---

#### BFS

```
Given a undirected connected graph G under Adjacency list representation:  

def BFS(graph, start_node, target_value):

  if !start_node: return ;

  # init
  visited = HashMap<node, False>()
  q = Queue<node>()

  q.enqueue(start_node)
  while !q.isEmpty():
    curt_node = q.dequeue()
    if curt_node.val == target_value: return curt_node;

    visited[curt_node] = True
    neighbors = graph[curt_node]
    for node in neighbors:
      if visited[node]:
        continue
      else:
        q.enqueue(node)

  return Null;

```
---

#### DFS

```
Given a undirected connected graph G under Adjacency list representation:  

def DFS(graph, start_node, target_value):

  if !start_node: return ;

  # init
  visited = HashMap<node, False>()
  s = Stack<node>()

  s.push(start_node)
  while !s.isEmpty():
    curt_node = s.pop()
    if curt_node.val == target_value: return curt_node

    visited[curt_node] = True
    neighbors = graph[curt_node]
    for node in neighbors:
      if visited[node]:
        continue
      else:
        s.push(node)

    return Null;
```
---

#### Common Algorithm Problems using BFS/DFS

1. Connected-components Labeling Problem:  
\- _Run BFS/DFS on an unvisited node u and label them as connected till there is no unvisited node left._  

---

2. Bipartite Problem:  
\- _Run BFS on a start_node and label nodes on odd layers "blue" and even layers "red". Return True if there is no red-red edge or blue-blue edge, else false._  

__NOTE- Bipartite definition:__ $$ G = X \bigcup Y \; X \bigcap Y = \emptyset $$  

---

3. Test Strong Connectivity on Directed Graph:  
\- _Pick a node in Graph and test if it can reach to every other node in the same graph._  
\- _Starting from the same node, test on_ $$ G^REV $$ _(reverse all the edges) if it can still reach to every other node._  
\- _If both tests pass, G is strongly connected, else return False._  

---

4. A directed acyclic graph has topological ordering  
\- _Find a node v with no incoming edges. Assign the next order number to v.__  
\- _Delete v and all its outgoing edges._  
\- _Repeat the above steps till no such node v exists._  
\- _If all nodes have been deleted, a topological ordering has been found, else, the graph contains cycles._  

__NOTE- topological ordering is the numbering of the vertices so that each edge is directed from a vertex with lower number to one with higher number.__  

```
Implementation Details: graph represented in Adjacency list.

def topological_ordering(graph):

  # init
  inDegrees = HashMap<node, int>()
  labels = HashMap<node, int>()
  zeroDegrees = Queue<node>()
  counter = 0

  # O(n + m)
  for start_node in graph:
    inDegrees[start_node] = 0
    for end_node in graph[start_node]:
      if end_node in inDegrees:
        inDegrees[end_node] += 1
      else:
        inDegrees[end_node] = 1

  # find the node without incoming edge to start
  for node in inDegrees:
    if inDegrees[node] == 0:
      zeroDegrees.enqueue(node)

  # labeling
  while !zeroDegrees.isEmpty():
    curt_node = zeroDegrees.dequeue()
    labels[curt_node] = counter
    counter += 1
    neighbors = graph[curt_node]
    for node in neighbors:
      inDegrees[node] -= 1
      if inDegrees[node] == 0:
        zeroDegrees.enqueue(node)

  # check if all the node got labeled
  for node in graph:
    if ! node in labels: return Null;

  return labels
```

---

5. Finding strong components problem

To do
