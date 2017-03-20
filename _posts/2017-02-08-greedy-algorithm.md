---
layout: post
title: ICS260 - Greedy Algorithm on Graph
date: 2017-02-08 23:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



Last Modified: 20170319

__Greedy Algorithm: Make decision based on some local optima and hoping it can achieve global optima in the end.__  

---

#### Interval Scheduling Problem

Input: A set of jobs with start and end time.  

Output: Maximum number of jobs in a __sequence__ so that there is no overlap.  

Algorithm: Earliest Finish Time First O(nlogn). _You goal is to maximize the number of non-overlapping jobs within the time frame._  

---

#### Interval Coloring Problem

Input: A set of jobs with start and end time.  

Output: Minimum number of colors needed to label __all__ the jobs so that there is no overlap between same-color jobs.  

Algorithm: Earliest Start Time First O(nlogn). _All the jobs need to be labeled sooner or later._

NOTE:  
1. Sort by start time as well as __end__ time - O(nlogn) _So it only takes O(1) to find the next earliest finishing job, its actual finishing time and assigned color. Every time, you only need to compare with the next earliest finishing job. Otherwise, this algorithm takes O(n^2)._  
2. __Take/Add__ next available color from a LinkedList - O(1)  
3. Assign the next available color to a job when it overlaps with all previously scheduled jobs.  
4. Assign the color ready for reuse to a job when the current job starts later than the finished one.  

_Why not using earliest finishing time?_  
_Intuitively, earliest starting time first strategy allows you to start earlier, which saves time that would've wasted if you use earliest finishing time first strategy. Even the former doesn't guarantee you earlier ending time in any case, that's fine. Just assign the next available color to the next job. It is inevitable that you do them in parallel._

---

#### Minimize Lateness Problem

Input: A set of jobs with required processing time and deadline.  

Output: A sequence of jobs so that the lateness is minimized.  

For example: If the finish time of the last job exceeds its deadline by 10, lateness += 10.  

Algorithm: Earliest Deadline First O(nlogn).  

---

## Greedy Algorithm to Solve Graph Problems

---

#### Shortest Path Problem: Dijkstra's Algorithm

Input: G(V, E) with non-negative costs on edges.  

Output:  
1. Find the shortest path from s to t ($$ s, t \in G $$).  
2. Find the shortest path from s to every other node in G.  

__Idea behind this algorithm:__  
Let's say there are multiple paths that go from s to t. The shortest one is among them. If we keep growing the path tree from s till we reach t, we will be able to spot at least one path from s to t. If we denote the last one before t is t.parent, we can say the shortest path is either through t.parent or one that comes through some other node. If we keep growing the path tree in a way that __we keep increasing the reachability of the tree from s at the next lowest cost__, we will have output_1 when we reach t and we will have output_2 when we reach every other node in the graph.  

__Note: During this growing process, it doesn't hurt to go to some other branches that don't even remotely connect with t because:__  
1. __If t is some neighboring node of the current node x, x will still be the parent node of t in output_1.__
2. __If t is some neighboring node of some next node xx, we have to go through xx anyway to get to t so it doesn't hurt.__


``` python
def shortest_path_from(s, graph, costs):
  # @ s: start_node
  # @ graph: adjacency representation
  # @ costs: a function that can return the cost of an edge

  r = s.copy() # copy s as the root node of our tree
  d = HashMap<node, distance>() # O(1) to find the shortest distance from s to node
  q = PriorityQueue<distance, node>() # O(1) to find and delete the min-key element

  # store info in these two data structure where the key in PriorityQueue is referencing d[node]
  # so as d[node] gets updated, the keys in PriorityQueue can get updated as well.
  for node in graph:
    if node == s:
      d[node] = 0
    else:
      d[node] = MAX_INT # so that the nodes popped out from PriorityQueue must be some child of the start_node rather than some random node.
    q.add((d[node], node))

  # find the next lowest cost node to start increasing reachability
  while !q.isEmpty():
    _, node = q.popMin()
    for next_node in graph[node]:
      if d[next_node] <= d[node] + costs(node, next_node):
        continue # switch to next if the next_node already found on a smaller path from s or next_node is a backtracking node (this is a undirected graph)
      '''
        Considering how d is defined, it looks somewhat like dp.
      '''
      d[next_node] = d[node] + costs(node, next_node)
      next_node.parent = node # build the tree

  return r
```

<img src="{{ '/styles/images/greedy-algo-on-graph/DijkstraDemo.gif' }}" width="50%" />

NOTE:  
1. To find the shortest path from s to t, start from t and follow the parent pointer.  
2. The complexity is O((m + n)logn)
3. Use Fibonacci heap can further decrease the complexity.  

---

#### Minimum Spanning Tree - Prim's Algorithm and Kruskal's Algorithm

__Definition:__  
1. Output T is a tree.  
2. All the nodes in input G can be found in output T.  

__Prim's Algorithm: Start with a seed node s. Repeatedly grow T by adding the cheapest edge connecting (some node in T) and (some node in G but not yet in T). Prim's Algorithm differs from Dijkstra's Algorithm in that: a node's distance is defined as how far it is from the start node in Dijkstra's Algorithm while distance is define as how far it is from any node already in T to any node not yet in T in Prim's Algorithm because they have different focuses on the output. Dijkstra's Algorithm was used to find the shortest path from s to t while Prim's Algorithm was used to find the MST, even though they are related.__


``` python
def prims_MST(s, graph, costs):
  # @ s: start_node
  # @ graph: adjacency representation
  # @ costs: a function that can return the cost of an edge

  r = s.copy() # copy s as the root node of our tree
  d = HashMap<node, distance>() # O(1) to find the shortest distance from any node in the growing tree to the this node
  q = PriorityQueue<distance, node>() # O(1) to find and delete the min-key element

  # store info in these two data structure where the key in PriorityQueue is referencing d[node]
  # so as d[node] gets updated, the keys in PriorityQueue can get updated as well.
  for node in graph:
    if node == s:
      d[node] = 0
    else:
      d[node] = MAX_INT # so that the nodes popped out from PriorityQueue must be some child of the start_node rather than some random node.
    q.add((d[node], node))

  # find the next lowest cost node to start increasing reachability
  while !q.isEmpty():
    _, node = q.popMin()
    for next_node in graph[node]:
    # ---------------- difference below -------------------
      if d[next_node] <= costs(node, next_node):
        continue
        # continue if the next_node already found on a smaller path from s or next_node is a backtracking node (this is a undirected graph)
      d[next_node] = costs(node, next_node)
    # ---------------- difference above -------------------
      next_node.parent = node # build the tree

  return r
```

There are 3 data structures to be maintained in a Dijkstra-prim algorithm:  
1. HashMap<node, distance> two algorithms varies in terms of the definition of distance here  
2. BinaryHeap<distance, node> to prepare for the next node asked for.  
3. Tree<node>, connected by parent pointer.  

---


__Kruskal’s Algorithm does two things:  
Initialize a graph T with every node from G but no edge at all.  
Connect nodes in T with edges in G sorted in order of increasing weight.  
(skip the edge that creates a circle, in other word if the edge points to a node that’s already in T. This is equivalent to connecting a node in T with a node in G yet not in T, in Prim's.)__

To efficiently detect cycle, a Union-Find data structure comes in handy.  
The most important take away here is how this Union-Find data structure is constructed and utilized.  

``` python
"""
specify needed attributes of Node()
"""
class Node(object):
  self.parent = NULL
  self.size = NULL # indicate the size of the tree rooted at this node


"""
specify how Union-Find() data structure is constructed and how find() union() functions are optimized.
"""
class Union-Find(object):

  self.set = set()

  def add(self, node):
    self.set.add(node)
    return ;

  # path compression so that self.find(x) will take amortize O(1)
  def find(self, node):
    # this functions take O(n) to recursively traverse a tree bottom up
    # from the input node to find the root node and then redirect the parent
    # pointer of each node except root node along the way to root node.
    if node.parent = NULL:
      return node
    r = find(node.parent)
    node.parent = r
    return r

  # size balancing so that self.find(x) will take amortize O(1)
  def union(self, node1, node2):
    if self.find(node1) == self.find(node2):
      return ;
    # make sure r1, r2 are in the same tree
    r1, r2 = self.find(node1), self.find(node2)
    if r1.size < r2.size:
      r1.parent = r2
      r2.size += r1.size
    else:
      r2.parent = r1
      r1.size += r2.size
    return ;
  # analysis:
  # 1. x.size doesn't change once x became a non-root node
  # 2. x.parent.size >= 2 * x.size
  # 3. All path <= log(n)


"""
kruskal's algorithm using the Union-Find() data structure defined as above.
"""
def kruskals_MST(graph, s, costs):
  # @ s: start_node
  # @ graph: adjacency representation
  # @ costs: a function that can return the cost of an edge

  T = EmptyTree()
  uf = Union-Find()
  q = PriorityQueue<cost, edge>()

  for node in graph:
    T.add(node)
    uf.add(node)
    for neighbor in graph[node]:
      q.add((costs(node, neighbor), (node, neighbor)))

  # Build the tree with T and detect cycle using uf data structure.
  while !q.isEmpty():
    _, edge = q.popMin()
    from, to = edge[0], edge[1]
    if uf.find(from) == uf.find(to):
      continue # detect cycle
    T.connect(edge)
    uf.union(from, to)
    # union a tree that contains "from" and another tree that contains "to"

  return T

```

__NOTE:__ For a second, I was wondering why we can't use a "visited" array to detect cycle like what we did in Prim's. There are ultimate differences between these two algorithms, the union-find data structure is necessary for this algorithm to work.  

__A very natural use case of Kruskal's Algorithm: K-clustering:__  

How to find the K-clustering for a bunch of points. Between every two clusters is the distance maximized.  
1. Now we see each point as a vertice.  
2. Run Kruskal's Algorithm k steps before it is naturally terminated.  
3. After it is terminated, a MST is found. But k steps before it, a k-clustering is found.  










<!-- ###########################################################################
######### Cushion ##############################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
################################################################################
-->
