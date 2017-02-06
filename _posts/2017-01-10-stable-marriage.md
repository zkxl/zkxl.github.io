---
layout: post
title: ICS260 - Stable Marriage
date: 2017-01-10 16:20:00 -0800
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

---

#### Claims:  

1. The sequence of partners to whom a man proposes gets worse; the sequence of partners to whom a woman is engaged gets better.

2. The algorithm terminates after at most n^2 proposals.

3. If a man m is free, there is some woman to whom he has not yet proposed.  
_proof by contradiction:  
- suppose that m is free and he has proposed to all women.  
- then all women are engaged to some men.  
- since |M| == |W|, these means all men are engaged._  

4. They matching computed by the G-S algorithm is a perfect matching.  
_proof by contradiction:  
- suppose that the algorithm terminates without a perfect matching, meaning there is a free man m.  
- according to the code, m must have proposed to every woman.  
- according to the claim 3, it is impossible._  

5. The matching computed by the G-S algorithm is a stable matching.  
_proof by contradiction:  
- suppose that there is an instability, meaning that the matching contains two pairs (m, w) and (m', w') such that m prefers w' to w and w' prefers m to m'.  
- the last time m proposed, it was to w.  
- if m proposed to w' some earlier time, w' rejected m and she prefers m' to m, contradiction.  
- if m proposed to w' some later time, m doesn't prefer w' to w._  

---

#### Determinacy Analysis in G-S Algorithm

Q: The non-deterministic G-S algorithm however produces deterministic result, given any particular input. Why?

A: The G-S algorithm always pairs each man with his best valid partner.  

__claim: Stable matching problem has more than 1 possible outcomes; Different execution of G-S algorithm leads to different outcomes. However, if w is a valid partner of m, then (m, w) will appear at least once in certain valid outcome.__

_proof by contradiction:  
- suppose at least one proposal from m to a valid partner w is rejected (either immediately or as a result of a broken engagement.), consider that the_ __first rejection__ _ever occurred during this execution E of G-S algorithm: m proposes to a valid partner w and w immediately rejects m because she prefers her current match m'.  
- Because w is a valid partner of m, there exists some other execution E' of G-S algorithm where (m, w) are paired and m' is paired with some other woman w', which is stable.  
-_ __However__, _when w rejected m for m' during the execution of E, that was the first rejection ever occurred. That means, at that point, m' hasn't been rejected. Clearly, m' is proposing at decreasing order and w' is a valid partner of m'. So m' must prefer w to w'.  
- Since m' prefers w to w' and w prefers m' to m. The outcome from the execution of E' can't be stable, which is a contradiction.  
- Therefore, our original proposal is false._    

---

#### Complexity and Implementation

Input: two 2D matrices (man, preference_list) and (woman, preference_list)  

Output: a stable perfect matching S.  

The algorithm terminates at most n^2 proposals, looping over n^2 times. If every operation in this loop is O(1), the overall complexity will be O(n^2).  

Specifically:  

1. find a free man - keep only free man on a linked list so it takes O(1) to access the next free man;  

* It takes O(n) to convert an array into a LinkedList;  

2. for a given man m, find his next highest ranking woman w to whom m hasn't proposed - maintain a 1D array h, where h[m] == w; It takes O(n) to convert one man's preference_list into a LinkedList started with the highest ranking woman. Then it takes O(n^2) to convert all the men's preference_list to LinkedLists maintained within a 1D array.  

3. For a given woman w, find out if she's currently engaged or not - maintain a 1D array c, where c[w] == null if w is not engaged and c[w] == m' if w is engaged with m'.  

4. For a given woman w engaged with m', to whom m proposed, determine whether w prefers m to m' - this is tricky to do in O(1) because normally you'd need to reference to w's preference_list and find how m ranks to m'. __Solution: from woman's preference array, you need to create a "ranking" array r where r[w, m] = m's ranking in w's preference_list. This can be done in O(n^2).__  

---

#### Exercises

Same problem. Under the circumstances that there are k (k < n) good men and k (k < n) women, (n - k) bad men and (n - k) bad women. Every man ranks good women higher than bad women. Every woman ranks good men higher than bad men. Show that in every stable matching, every good man is married to a good woman.  

_Prove:  
- Suppose there is a good man m married to a bad woman w', then there must be a good woman w married to a bad man m'. Thus, (m, w') and (m', w).  
- Since good ones are always preferred, we have m prefers w to w', w prefers m to m'. This leads to an instability.  
- This makes a contradiction._  

Generalize the problem by explicitly stating that certain men-women pairs are forbidden (listed in set F). Under these circumstances, a stable matching can be established if:  
* there's no usual kind of instability (above).  
* there's no such pair (m', w) and a single m, who has higher rank than m' in w's preference_list and also (m, w) $$ \notin $$ F.  
* there's no such pair (m, w') and a single w, who has higher rank than w' in m's preference_list and also (m, w) $$ \notin $$ F.  
* there's no single m and single w whose pair (m, w) $$ \notin $$ F.  

_G-S algorithm can be adapted to work out this case:  
- the only change is an added restriction on "while condition"_  


```
while there is free man m who hasn't proposed to every woman w for which (m, w) not in F
  w = the highest ranking women in m's preference_list to which m hasn't proposed yet and not forbidden to propose.
  if w is free:
    (m, w) became engaged
  else if w prefers m to his current match m':
    (m, w) became engaged and m' became a free man
  else:
    (m', w) stays the same and m is still a free man
  endif
endwhile
return the set S of engaged pairs
```

_prove:  
- If there's a usual kind of instability, please see_ __Claim No.5__.  
_- Suppose there is a pair (m', w) and a single m who is ranked higher than m' in w's preference_list and (m, w) is not forbidden. m must've  proposed to w and w rejected m for m', which is contradictory.  
- Suppose there is a pair (m, w') and a single w who is ranked higher than w' in m's preference_list and (m, w) is not forbidden. Same as above.  
- Suppose there are a single m and a single w and (m, w) is not forbidden, then m must've proposed to w and w can't still be single._  
