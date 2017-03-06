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

#### Weight interval scheduling problem  



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
  if dp[start_idx - 1] > dp[nonOverlaps[start_idx]] + jobs[start_idx].weight:
    output_schedule(start_idx - 1, jobs, dp)
  else:
    print jobs[start_idx]
    output_schedule(nonOverlaps[start_idx], jobs, dp)
  return ;
```






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
