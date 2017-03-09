---
layout: post
title: ICS260 - Priority Queue
date: 2017-02-05 16:20:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}



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
