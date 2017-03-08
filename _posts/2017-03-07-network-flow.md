---
layout: post
title: ICS260 - Network Flow Problem
date: 2017-03-07 21:30:00 -0800
categories: Study-Notes
tag: algorithms
---

* content
{:toc}




## Define Maximum Network Flow Problem

__Network Flow:__ A directed graph G = (V, E) with the following properties:  
1. each edge has a non-negative integer value denoting its capacity.
2. there is a source node with outgoing edges __only__ and a terminal(sink) node with incoming edges __only__.
3. every node is an incident on at least one edge.  

__s-t flow:__ A function that assigns an integer number to each edge so that:  
1. __capacity condition:__ 0 <= number <= edge capacity
2. __conservation condition:__ every node except s and t having: sum of numbers on incoming edges equal to the sum of numbers on outgoing edges

__the value of a s-t flow__ is the sum of numbers on the outgoing edges of source node, also equivalent to the sum of numbers on the incoming edges of sink node.

__max network flow problem:__ Given a network flow, find the maximum flow value of the network.  

## Ford-Fulkerson Algorithm

__high level description:__  
Set flow(e) = 0 for all the edges in given graph.  
While there is a __s-t augment path__ in the current __residual graph__:   
Augment flow(e) for all edges along the path by:
+ increase the flow on forward edges
+ decrease the flow on backward edges

__residual graph:__ A graph generated based on the original network flow graph and the flow function with the following properties:
1. Denoted by Gf, containing all the vertices in the original network flow graph G.
2. For each edge e = (u, v) in G, Gf contains (u, v) with capacity = (capacity(e) in G - flow(e)).
3. For each edge e = (u, v) in G, Gf contains (v, u) with capacity = flow(e).

__s-t augment path:__ A simple path from s to t. It is used to increment flow by some value X where X is the minimum capacity in Gf of all the edges in the augment path.

__Note:__ this algorithm must terminate after at most C iterations, where C is the sum of each edge's capacity. It is because after each iteration, if an augment path can be found, the flow must increment by at least 1 while there are at most C such iterations. 



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
