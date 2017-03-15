---
layout: post
title: ICS260 - Dynamic Programming
date: 2017-02-28 22:30:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



__The best way to learn DP is through practice.__

### Weight interval scheduling problem  



Input: a set of jobs with starting time, ending time and weights.  
Output: a set of non-overlapping jobs that have the maximum weights in total.  

Thought: Divide it into subproblems.  
1. sort total 10 jobs according to their ending time.
2. if the optimal schedule doesn't include 10th job, then the solution must be the same as the optimal schedule we figured for all previous job.  
3. if the optimal schedule includes 10th job, then the solution must be like this: the optimal schedule we figured for all previous job till the last one that doesn't overlap with 10th job + 10 job.  

Recursive algorithm:  

``` python
def opt_weights(jobs):
  '''
  jobs: [(starting_time, ending_time, weight)] - interval representation
  jobs: [class Job(starting_time, ending_time, weight)] - object representation
  '''
  jobs = sorted(jobs, key = lambda x: x.ending_time)

  # generate mapping relation between job and the last non-overlapping job
  # nonOverlaps[i] denoting the index of the last non-overlapping job
  nonOverlaps = [None for _ in range(len(jobs))]
  for i in range(len(jobs)):
    nonOverlaps[i] = find_last_nonOverlap(i, jobs)

  max_weights = recOpt(len(jobs) - 1, jobs, nonOverlaps)

  return max_weights


def find_last_nonOverlap(x, jobs):
  '''
  This is a very nice snippet of code

  input:
    x: index of the current job
    jobs: job list sorted by ending_time
  output:
    last_nonOverlap: index of the last non-overlapped job
  '''
  last = -1
  # when x == 0, jump to return;
  for i in range(x):
    if jobs.ending_time < x.starting_time:
      last = i
    else: break # use the fact that the given job list is sorted.
  return last


def recOpt(x, jobs, nonOverlaps):
  if x == -1:
    return 0

  left = recOpt(x - 1, jobs, nonOverlaps)
  right = recOpt(nonOverlaps[x], jobs, nonOverlaps) + x.weight

  return max(left, right)

```

_NOTE: two problems with the above solution_  
1. It runs in O(2^n) because every call on recOpt() creates another two recursive calls.  
2. You cannot find the schedule set that produces the max weights while recursively traversing the tree.  

Let's build the bottom up.  

``` python

def opt_schedule(jobs):
  jobs = sorted(jobs, key = lambda x: x.ending_time)

  nonOverlaps = [None for _ in range(len(jobs))]
  for i in range(len(jobs)):
    nonOverlaps[i] = find_last_nonOverlap(i, jobs)

  dp = [0 for _ in range(len(jobs + 1))]
  # dp[i] denotes the max weights that can be scheduled for jobs from 0 till i.
  for i in range(len(jobs)):
    # dp[-1] remains 0 for the use case when nonOverlaps[i] renders -1
    #        until it is lastly computed.
    dp[i] = max(dp[i - 1], dp[nonOverlaps[i]] + jobs[i].weight)

  # dp memory is built bottom up while output schedule is a top down process.
  # schedule has to be decided backward because
  # what is happening in the end is only determined by previous info
  # while what is happening in the beginning is affected both way.
  output_schedule(len(jobs) - 1, jobs, dp)

  return ;


def output_schedule(start_idx, jobs, dp):
  '''
  this is done in linear time.
  '''
  if start_idx == -1:
    return ;
  if dp[nonOverlaps[start_idx]] + jobs[start_idx].weight < dp[start_idx - 1]:
    output_schedule(start_idx - 1, jobs, dp)
  else:
    print jobs[start_idx]
    output_schedule(nonOverlaps[start_idx], jobs, dp)
  return ;
```

---

### Segmented Least Square Problem

__Problem Description:__
Image a linear regression problem where there is a bunch of points that you want to fit with a line. Now it is allowed to segment the line into a few pieces to form a better fit (namely reduce the sum of square residuals). However:
1. 1 segment may result in really bad fit.
2. n/2 (n is the number of points) segments tells us nothing about the data even though the error is Zero.

We want to find a good fit while keeping the number of segments small. So there is a fixed penalty for each segmentation in your solution. How to come up with the optimal solution?

__Problem Formulation:__

Let C be the fixed penalty.  

Sort all the points according some axis and label them 1 through n. Let e(i, j) (1 <= i <= j <= n) be the squared errors when a line is fit to points i through j to minimize the squared errors. A 2D matrix filled with e(i, j) can be pre-computed in:

$$ O(n^2) $$

Now the problem became:  

Find one or more k (1 <= k <= n) so that:

$$ {\underset{1 \leq k_{1} < k_{2} < ... k_{m} \leq n}{\operatorname{min}}} \; (C * |K| +  e(1, k_1) + e(k_1, k_2) + ... + e(k_m, n)) $$

__Observation and Recurrence:__  
If the min-err for the first i points is known to us, for some point j > i:
$$ minErr(j) = e(i, j) + C + minErr(i - 1) $$
And yet, we don't know such a fixed i, so for some arbitrary point 1 <= j <= n:  

$$ minErr(j) = {\underset{1 \leq i < j}{\operatorname{min}}}\;\{e(i, j) + C + minErr(i - 1)\} $$


__Solution:__
```
def leastSquaredErr(points):
  // pre-computed errors in O(n^2)
  e = 2D Matrix
  for i in points:
    for j in points:
      e[i][j] = SSR(i, j)
  // dp
  minErr = 1D Vector whose length is 1 + len(points)
  minErr[0] = 0
  for j in range(1, len(points) + 1):
    minErr[j] = e(1, j) + C
    for i in range(1, j):
      minErr[j] = min(minErr[j], e(i, j) + C + minErr(i-1))

  return minErr(len(points))
```

---

## More DP Exercises

---

#### Max weighted set of vertices out of a simple path

__Problem Description:__ Given a simple path G, find an independent set of weighted vertices with the maximized sum of weights.

_Note: an independent set means any two vertices in this set are not connected._  

__Key Observation:__ For the first i vertices, the maximized set either contains the ith vertex or not. If it doesn't, it is the same as the maximized set of the first (i-1) vertices. If it does, it equals to the max weight from the first (i-2) vertices + the weight of ith vertex.

__Formulation:__

1. Subproblems: indexed by vertices {0, 1, 2, ..., n}  

2. Functions: Let X(i) be the max weight of the first i vertices (i >= 2):  

3. Goal: X(n)

4. Initial value: X(0) = 0, X(1) = weight(1)  

5. Recurrence:
$$ X(i) = max(X(i-1), \; X(i-2) + weight(i)) $$

__Note: initialization of X(0) = 0 is critical to compute X(2) even though the numbering of vertices start from 1.__

---

#### High-Low stress reward trade-off

__Problem Description:__ Given a list of high-stress job scheduled every day with different reward values for each; And a list of low-stress job scheduled every day with different but lower reward values for each. Restrict that one can do nothing during the day before taking a high-stress job. It is OK for one to take high-stress on the first day. Find a optimal strategy to schedule which job to take every day.

__Key Observation:__ For the first i days, the optimal schedule either contains the ith high-stress job or ith low-stress job or nothing.  
+ If it contains ith high-stress job, the max reward is R(i-2) + highReward(i).  
+ If it contains ith low-stress job, the max reward is R(i-1) + lowReward(i).
+ If it contains nothing, the max reward is R(i-1), which is less than the above.

__Formulation:__

1. Subproblems: indexed by days {0, 1, 2, ..., n}  

2. Functions: Let R(i) be the max reward for the first i days (i >= 2):  

3. Goal: R(n)

4. Initial value: R(0) = 0, R(1) = highReward(i)

5. Recurrence:
$$ R(i) = max(R(i-2) + highReward(i), \; R(i-1) + lowReward(i)) $$

__Note: this is simply a minor extension of the first problem. The common thing both of them need to trace back 2 steps, each with different operations.__

---

#### Longest Path Problem

__Problem Description:__ Given an __ordered__ graph G, where only edges from lower numbered vertex to higher numbered vertex are allowed. Find the path from the first vertex to the last vertex with the maximum number of edges.  

_Note: It is also implied that the frist vertex has only outgoing edges and the last vertex has only incoming edges._

__Key Observation:__ For the first i vertices (with the ith vertex being the last vertex in this case), there is no more discussion about whether ith vertex is in the solution set or not because according to the problem requirement, it must be the last vertex in the solution path. The maximum length of the path L(i) equals to 1 + L(j) where j is any vertex with an edge to ith vertex.

__Formulation:__

1. Subproblems: indexed by days {1, 2, ..., n}  

2. Functions: Let L(i) be the max length for the first i vertices (i >= 1):  

3. Goal: L(n)

4. Initial value: L(1) = 0

5. Recurrence:
$$ L(i) = \underset{j}{\operatorname{\max}} \; \{ \; L(j) + r(j, i) \; | \; 1 \leq j < i \; \} $$

Set r(j, i) = 1 when there is an edge from j to i, else negative infinite.

__Note: this problem is different in that: What makes the previous problems dynamic programming problems is the trade-off between taking or not taking or which to take. But in this problem, it is always good to add an edge to the final solution. However, what lies in the heart of this problem is the dynamics how we jump from some previous vertex to the current vertex.__

---

#### SF-NY switch problem

__Problem Description:__ Every month, you choose to run your business in SF or NY. Each month, different locations give you different operation costs. The switch cost is fixed F. What is the optimal traveling schedule for your business? You can choose wherever to spend the first month, which doesn't cost you anything.

__Key Observation:__ The problem is to minimize the total cost. Two arrays is needed for the dynamic solution of this problem. Why? Because every time you make your decision, you have to refer the accumulative cost from the last month of both cities. If they are close and F is bigger, you don't want to travel. Otherwise, you'd like to travel. In other words, you need to maintain two arrays, S and N where S[i] is the accumulative cost for you to stay in SF on ith month, and N[i] is the accumulative cost for you to stay in NY on ith month.

__Formulation:__

1. Subproblems: indexed by months {1, 2, ..., n}  

2. Functions:  
Let S(i) be the accumulative cost of staying SF on month i.  
And N(i) be the accumulative cost of staying NY on month i.

3. Goal: min(S(n), N(n))

4. Initial value:  
S(1) = operationCostSF(1)  
N(1) = operationCostNY(1)

5. Recurrence:
$$ S(i) = min(S(i-1), N(i-1)+F) + operationCostSF(i) $$
$$ N(i) = min(N(i-1), S(i-1)+F) + operationCostNY(i) $$

---

#### Word Segmentation

__Problem Description:__ Given a string consisting of a bunch of words but they are not separated by space. How to optimally segment the string so that it makes the maximum sense? Provided a function call that works as following: quality('hello') > quality('hel').  

__Key Observation:__ For the first ith characters in the string S, the addition of the (i+1)th character may render a better segmentation strategy for all the previous characters. For example, a break point j might be found to render the max optimal quality = optimal(j-1) + quality(S[j, i]), which means we have to check back each j for 1 <= j < i.

__Formulation:__

1. Subproblems: indexed by months {1, 2, ..., n}  

2. Functions: Let Q(i) be the optimal quality found for the first ith characters in S.

3. Goal: Q(n)

4. Initial value: Q(1) = quality([0, 1])

5. Recurrence:
$$ Q(i) = max(Q(j) + quality(\; S[j, i] | 1<= j < i \;))$$

---

#### Trunk Loading Problem

__Problem Description:__ You've got a trunk with maximum loading weight W and n boxes each weights:  

$$ \{\; w_{i} \;|\; 1 \leq i < n \;and\; w_{i} \leq W \;\} $$  

Design a strategy to load as heavy goods as possible.

__Key Observation:__ If we try all the combinations, it is exponential time. If we:  
1. __reduce__ the problem by the first few boxes, with function maxLoad(i) denoting the max load if we only have boxes from 1 to i.  

$$ maxLoad(i) = max(maxLoad(i-1),\;\; {\underset{1\leq j <i}{\operatorname{max}}} \{\; maxLoad(j) + w_{i}\;|\;maxLoad(j) + w_{i} \leq W \;\}) $$  

2. __reduce__ the problem by both the number of boxes and the maximum capacity of the trunk, with maxLoad(i, j) denoting the max load we can carry if we only have boxes from 1 to i and the maximum capacity of the trunk is j.  

$$ maxLoad(i, j) = max(w_{i} + maxLoad(i-1, j-w_{i}),\; maxLoad(i-1, j)) $$

Meaning: The max load is you either include box i at capacity j, or exclude it.  

_Note: both cases are initialized as 0._  

---

#### Knapsack Problem

__This is too classic. I suggest the following:__ [催天翼 背包九讲](https://github.com/tianyicui/pack).

---

#### Edit Distance Problem

__Problem Description:__ You are given two strings A and B, return the minimum edit distance of them. Every operation, include insertion, deletion (gap) and modification (match) takes a unit distance.  

__Formulation:__  
1. Subproblems: indexed by months i {1, 2, ..., n} and j {1, 2, ..., m}  

2. Functions: ED(i, j) be the edit distance between A[:i+1] and B[:j+1]

3. Goal: ED(n, m)

4. Initial value:  
ED(0, 0) = 0  
ED(0, j) = j for j from 1 to m  
ED(i, 0) = i for i from 1 to n  

5. Recurrence:

$$ ED(i, j) = min(ED(i-1, j-1) + 1, ED(i-1, j) + 1, ED(i, j-1) + 1) $$

#### Bellman-Ford Shortest Path Algorithm

__Problem Description:__ Given a weighted(with negative weights), directed graph G, find the min-cost path from s to t. Previously given a weighted (non-negative weights) directed graph, Dijkstra's Algorithm can find the min-cost path. Now, let's take a look at Bellman-Ford Algorithm.  

_Note: if the graph contains negative cycle, then as long as s can find a path to that cycle, there's no solution to this problem. Here we are looking for a_ __simple__ _path solution._  

__Bellman-Ford Algorithm:__ Different from Dijkstra's Algorithm, which greedily extends the reachability from s at the next lowest cost, Bellman-Ford Algorithm simply extends the reachability from s and updates the min-cost from s to all other vertex v at the same time __for (n-1) times__ where n is the number of vertices and (n-1) is the maxlength of the solution path because it is simple.  

__Solution:__
```
def bellmanFord(graph):
  cost = 2D Matrix // where cost[i][v] is the min-cost of from s to v at ith iteration.
  cost[0][v] = + infinity // init
  cost[0][s] = 0
  predecessor = 1D Vector // note the predecessor of each vertex
  for i from 1 to n: // where n is the # of vertices
    for each edge (u, v) in E(graph):
      if cost[i][u] + weight(u, v) < cost[i][v]:
        cost[i][v] = cost[i][u] + weight(u, v)
        predecessor[v] = u

  for each edge (u, v) in E(graph): // the edge direction is implied
    if cost[-1][u] + weight(u, v) < cost[-1][v]:
      raise Error("negative cycle detected.")

  return cost[-1][t], predecessor
```

NOTE:  
1. the above algorithm takes O(nm) where n is the # of vertices and m is the # of edges. The space complexity can be reduced from O(n^2) to O(n) because we will never use the other "historical" records except the "last" ones.
2. To understand how the detection of negative cycle works, you'd better draw a graph with such a cycle. But basically, if during the last iteration through all the edges and for some edge (u, v), `cost[-1][v]` got reduced to below `cost[-1][u]`, there is a cycle.

---

#### All Pairs Shortest Path Floyd's Algorithm

__Problem Description:__ In a directed weighted graph G, find the min-cost path from every vertex to every other vertex.  

__Analysis:__ If you read through the last problem using Bellman-Ford Algorithm, you will see that it can actually return the min-cost path from s to every other vertex. So this problem is solvable in O(mn^2). However, in extreme cases, m could be theoretically as large as n^2, which makes the above solution less efficient. Floyd's Algorithm runs in O(n^3).

__Key Observation:__ Bellman-Ford Algorithm maintains a 2D matrix to record the min-cost from s to every other vertex v at all iterations. Here Floyd's Algorithm maintains a 3D matrix to record the min-cost from every other vertex v to every other vertex u at all iterations. The number of iterations needed is the same, n. Also, the space complexity can be reduced to O(n^2) like how we reduce it from O(n^2) to O(n) in Bellman-Ford.

__Function:__ cost(k, i, j) is the min-cost of path from i to j with maximum k intermediate vertices allowed.  

__Recurrence:__  

$$ cost(k, i, j) =  min(cost(k-1, i, j), \; cost(k-1, i, k) + cost(k-1, k, j)) $$

__Solution:__

```
// the following code assumes that the graph is under adjacency matrix representation

def floyd(G):
  cost = 3D Matrix // consider space complexity reduction
  for i from 1 to n: // init
    for j from 1 to n: // weight(i, i) = 0
      cost[0][i][j] = weight(i, j) // weight(i, unreachable) = + infinity
  for k from 1 to n:
    for i from 1 to n:
      for j from 1 to n:
        cost[k][i][j] = min(cost[k-1][i][j], cost[k-1][i][k] + cost[k-1][k][j])
  return cost[-1]
```

__Additional Note:__
1. Think about why the second term is cost(k-1, i, k) + cost(k-1, k, j), instead of cost(k-1, i, k) + weight(k, j).
2. Think about the situation where only a 2D matrix was maintained. If you can figure out the update code as follows: `cost[i][j] = min(cost[i][j], cost[i][k] + cost[k][j])`, you truly understand what the third dimension k means and how this algorithm works.


<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
<!-- #################################### -->
