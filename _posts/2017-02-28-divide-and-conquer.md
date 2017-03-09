---
layout: post
title: ICS260 - Divide and Conquer
date: 2017-02-28 13:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



## Divide and Conquer Theorem

In the example of __Merge Sort__:  
During divide-phase, the algorithm splits the problem into half and recursively call itself on the two subproblems.   
During conquer-phase, the algorithm does some operations that take O(n) to merge the sorted sub results into real higher-level results.  

Mathematically:  

$$ T(n) = 2 T(\frac{n}{2}) + O(n) $$

$$ T(n) = n \log{n} $$


Now let constants:  
1. a >= 1 denoting the number of recursive calls we need at each recursion
2. b >= 1 denoting the number of subproblems
3. f(n) denoting the asymptotical cost we need to pay at each recursion.  

The above became:  

$$ T(n) = a T(\frac{n}{b}) + f(n) $$

#### The master theorem says:  

1. Operation cost is __upper bounded__:  

$$ if\;f(n) = O(n^{\log_{b}^{a - \epsilon}})\;\;for\;some\;constant\;\epsilon > 0 $$

$$ T(n) = \Theta(n^{\log_{b}^{a}}) $$

2. Operation cost is __sandwiched__:  

$$ if\;f(n) = \Theta(n^{\log_{b}^{a}}) $$

$$ T(n) = \Theta(n^{\log_{b}^{a}} \log{n})$$

3. Operation cost is __lower bounded__:  

$$ if\;f(n) = \Omega(n^{\log_{b}^{a + \epsilon}})\;\;for\;some\;constant\;\epsilon > 0 $$

$$ T(n) = \Theta(f(n)) $$

Taking __Merge Sort__ as an example, __intuitively__:

1. If operation cost is __linearly__ decreasing as we divide the problem (case2), T(n) is the operation cost at each level times the number of levels.  

2. If operation cost is __superlinearly__ decreasing as we divide the problem (case3), T(n) is dominated by the operation cost at the first level where the number of instance is n.  

3. If operation cost is __sublinearly__ decreasing as we divide the problem (case1), T(n) is dominated by the operation cost of each recursive call while the number of levels doesn't matter.  

#### The simplified method says:  

Having a problem with cost:  

$$ T(n) = a T(\frac{n}{b}) + \Theta(n^k) \; where \; k >= 1 $$

1. If a > b^k, recursive work dominates:
$$ T(n) = \Theta(n^{\log_{b}^{a}}) $$

2. If a = b^k, both matters:
$$ T(n) = \Theta(n^{\log_{b}^{a}} \log{n})$$

3. If a < b^k, the only non-recursive call dominates:
$$ T(n) = \Theta(n^k) $$

---

## Algorithm Problems

#### construct a binary heap recursively  

_NOTE: we've talked about running "heapify()" iteratively in the post "Priority Queue" in O(n)._

Here is the recursive one, which runs in:  

$$ T(n) = 2 T(\frac{n}{2}) + O(\log{n}) $$

$$ T(n) = O(n) $$

``` python
class MinHeap(object):

  def __init__(self, nums):
		self.size = len(nums)
		self.array = [num for num in nums]
		self.heapify()

  def heapify(self):
    self._recHeapify(self.array, 0)
    # 0 index the first entry in self.array
    # aka the root if you think the array as a binary tree.
    return ;

  def _recHeapify(self, a, k):
    if 2 * k + 1 < len(a):
      self._recHeapify(a, 2 * k + 1) # take care of the left child
    if 2 * k + 2 < len(a):
      self._recHeapify(a, 2 * k + 2) # take care of the right child
    # the above is to divide till the last leaf is reached.
    self._siftDown(k)
    # the above is to conquer
    return ;

  def _siftDown(self, k):
    """
    refer to post "Priority Queue" for implementation details.
    """
    return ;
```

---

#### Planar Closest Pair Problem

Input: a list of points: [{Point(x, y)}]  
Output: the distance between the closest pair of points

Algorithm:  
1. recursively find the closest distance of the left half
2. recursively find the closest distance of the right half
3. find the closest pair with one point in left and one in right and compare their distance with what we found above.


``` python
def closestPair(points):
  # base case
  if len(points) <= 3:
    dist1 = dist(points[0], points[1])
    dist2 = dist(points[1], points[2])
    dist3 = dist(points[0], points[2])
    return min(dist1, dist2, dist3)

  # O(nlogn)
  Y = [sorted points in increasing order along y-axis]

  mid_x = sum[points.x] / len(points) # float

  # divide into left half and right half by mid_x
  left = [p if p.x <= mid_x for p in points]
  right = [p if p.x > mid_x for p in points]

  # recursive calls to further divide them,
  # which returns the closest distance found
  left_delta = closestPair(left)
  right_delta = closestPair(right)

  # restrict searching area to points
  # whose x is within [mid_x - delta, mid_x + delta]
  delta = min(left_delta, right_delta)
  left = [p if p.x > mid_x - delta for p in left]
  right = [p if p.x < mid_x + delta for p in right]

  # the following is the whole essence of the problem, which takes O(n)
  # so the problem can be solved in O(nlogn)
  for i in range(len(Y)):
    # starting from the point whose y value is the smallest
    # find one that is in left part of the searching area defined above
    if Y[i] not in left:
      continue
    # compare p with the next 6 points in Y found in right part of the searching area
    p = Y[i]
    j = 1 # pointer to the next Point in Y
    count = 0
    while i + j < len(Y) and count < 6:
      # why 6? TODO: add graph
      # but at least you know why the base case consists of 3 points.
      if Y[i + j] not in right:
        j += 1
        continue
      count += 1 # found one
      if dist(p, Y[i + j]) < delta:
        delta = dist(p, Y[i + j])
      j += 1

  return delta

```

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
