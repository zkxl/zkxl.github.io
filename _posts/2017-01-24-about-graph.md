---
layout: post
title: ICS260 - About Graph
date: 2017-01-17 16:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



## Graphs

---

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

Connected-components Labeling Problem:
\- _Run BFS/DFS on an unvisited node u and label them as connected till there is no unvisited node left._

---

Bipartite Problem:
\- _Run BFS on a start_node and label nodes on odd layers "blue" and even layers "red". Return True if there is no red-red edge or blue-blue edge, else false._

__NOTE- Bipartite definition:__ $$ G = X \bigcup Y \;\;and\;\; X \bigcap Y = \emptyset $$ where each edge in G has one end in X and the other in Y.  

---

Test Strong Connectivity on Directed Graph:
\- _Pick a node in Graph and test if it can reach to every other node in the same graph._
\- _Starting from the same node, test on_ $$ G^{REV} $$ _(reverse all the edges) if it can still reach to every other node._
\- _If both tests pass, G is strongly connected, else return False._

---

A directed acyclic graph has topological ordering
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

Finding strong components problem

To do
