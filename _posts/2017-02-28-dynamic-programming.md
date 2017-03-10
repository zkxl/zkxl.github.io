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

Find one or more k (1 <= k <= n), sorted in an array K = [k1, k2, k3, ..., km] so that:

$$ {\underset{k}{\operatorname{min}}} \; (C * |K| +  e(1, k_1) + e(k_1, k_2) + ... + e(k_m, n)) $$

---

### DP Exercises

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
