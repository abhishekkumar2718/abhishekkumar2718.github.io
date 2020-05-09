---
layout: post
title: Reachability Queries
category: Programming
tags: ['GSoC', 'Git', 'Programming']
asset_img: 'reachablity_queries'
excerpt: Reachability forms an important intermediate step in many graph algorithms, avoiding large worst-case penalties.
---

{:.no_toc}

In graph theory, reachability refers to the ability to get from one vertex to another within a graph. A vertex _s_ can reach a vertex _t_ if there exists a path that starts with _s_ and ends with _t_.

Reachability is one of the most fundamental questions that one can ask about the graph. For example, in a network of papers with edges representing citations, one might ask if one paper is based on another paper, in some possibly indirect way. Even classical algorithms like _Dijkstra's Algorithm_ can be significantly sped up by avoiding expansion of vertices that cannot reach target vertex.

The advent of massive graph structures comprising of hundreds of millions of vertices and billions of edges can make simple graph operations slow. Due to many applications and expensive computation in classical databases, the reachability problem has obtained much attention in the database community.

Reachability can naively be answered by any graph exploration algorithm (like DFS, BFS) that is _zero preprocessing and linear query time_. On the other hand, it's possible to preprocess all possible pairs in quadratic time and answer that is _quadratic preprocessing and constant query time_.

Additionally, the index data structures must reside in the main memory, creating an upper bound of space consumption.

Trade-offs between preprocessing time, query time, and space requirements make reachability a fascinating problem to analyze. Understanding the nature of the graph involved is crucial in making the right decision.

I have been selected as Google Summer Of Code Student for Git, working on the project [Implement Generation Number v2](http://summerofcode.withgoogle.com/projects/6140278689234944).

{% include toc.html %}

### Undirected Graphs

In an undirected graph, reachability is symmetric (_s_ reaches _t_ iff _t_ reaches _s_). Therefore, reachability between any pair of vertices can be determined by identifying the _connected components_ of the graph.

The connected components of an undirected graph can be identified in _linear time_ [Tarjan's Algorithm](https://en.wikipedia.org/l/Tarjan's_strongly_connected_components_algorithm) or more efficiently by a straightforward application of [Disjoint Set Unions](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).

Both solutions can answer queries in _constant time_.

### Directed Graphs

For directed graphs, reachability is no longer symmetric. However, we can still form [Strongly Connected Components](https://en.wikipedia.org/wiki/Strongly_connected_component) i.e., subgraphs where each vertex is reachable from every other vertex of the subgraph.

We can then condense SCCs into "super-vertices". By definition, this condensed graph would be a [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph).  It's important to note that the queries Q(G, u, v) and Q(G', [u], [v]) are equivalent, where G' is the condensed graph and [u], [v] are the representative elements of the strongly connected components.

Therefore, one of the preliminary steps before using the algorithms below is to find strongly connected components, which can be done in _linear time_ using [Kosaraju's Algorithm](https://en.wikipedia.org/wiki/Kosaraju%27s_algorithm) or [Tarjan's Strongly Connected Components Algorithm](https://en.wikipedia.org/wiki/Tarjan%27s_strongly_connected_components_algorithm).

#### Topological Level

Topological levels for a vertex are defined as follows:

1. If the vertex has no incoming edges, it has topological level 0.
2. Otherwise, the vertex's topological level is one more than the maximum of all vertices with incoming edges (usually called a "parent" vertex).

Sometimes, it's easier to work backward - that is vertices with no outgoing edges are assigned topological level of infinity, and parents have one fewer level.

As it's easy to see from the image, vertex _s_ can reach vertex _t_ if the topological level of _s_ is greater or equal to that of _t_. Topological levels can quickly find unreachable vertices, and are examples of "negative-cut filters".

{% include image.html filename="topological_levels.png" caption="Topological Levels. Taken from 'Semantic Zooming for Ontology Graph Visualizations' by Vitalis Wiens" %}

#### Post Order Interval

The key idea for post-order intervals is to assign each vertex a numeric identifier and represent reachable sets of vertices in a compressed form with an interval.

To assign node identifiers, the tree is traversed in a depth-first manner, visiting in post order. In the figure (b) below, vertex _e_ is visited first and is assigned post order number 1.

Post-order labeling results in _identifier locality_ i.e., for every complete subtree of T form a contiguous sequence of integers. Subtrees of vertex _a_ form the contiguous sequence [1, 4] in figure (c).

However, a node can have multiple parents (like vertex _c_), we need to propagate the intervals as well - forming figure (d).

To answer a query, we need to check if the post order number of the target is contained in one of the intervals associated with the source. Queries can be answered in _logarithmic time_.

{% include image.html filename="post_order_interval.png" alt="Post Order Interval Assignment" caption="Post Order Interval Assignment. Taken from 'Flexible and Efficient Reachablity Range Assignment for gRaph Indexing'" %}

#### Contraction Heirachies

Generally, [contraction hierachies](https://en.wikipedia.org/wiki/Contraction_hierarchies) are a speed-up technique for finding the shortest path in a graph. In preprocessing phase, shortcuts are created to skip over "unimportant" vertices. Intuitively, the process is similar to calculating distances for a road network. Distance between two important highway junctions can be pre-computed as a "shortcut" so that the algorithm does not have to consider the full path between the two.

Along the same lines, a Reachability Contraction Hierarchy can be defined based on any number order. We successively "contract" vertices with increasing order, adding shortcuts between pair of vertices that are now disconnected.

The query algorithm uses the ordering and shortcuts as follows: _t_ is reachable from _s_ in the RCH obtained if there is an "up-down" path i.e., a path in which order of vertices has only one maximum. Proof can be found in Section 3.1 of PReaCH.

{% include image.html filename="contraction_heirachies.png" alt="Contraction Hierarchies" caption="To find a path from s to t, the algorithm can skip over the gray vertices and use the dashed shortcut instead. The algorithm has to look at fewer elements. Taken from Contraction Heirachies - Wikipedia, by WhereAreMyPointersAt." %}

### Summary

Reachability refers to the ability to get from one vertex to another within a graph.

Reachability queries are an interesting problem, improving performance for many graph operations. Better and more sophisticated solutions are being created as the size of working graphs keeps increasing.

Reachability for the undirected graph can be found in linear preprocessing and constant query time with disjoint set unions. The answer isn't as evident for a directed graph because of differing performance on positive and negative queries, nature and size of the graph, and other factors. Topological levels, Post Order DFS interval, and Contraction Hierarchies are some of the building blocks for such algorithms.

In a later article, I will talk about the specifics of generation number for Git. We will discuss how Git uses reachability queries, the need for Generation Number v2 i.e., _Corrected Commit Date With Strictly Monotonic Offset_, and other interesting tidbits I came across. 

### Further Reading

1. [Reachablity](https://en.wikipedia.org/wiki/Reachability) - Wikipedia Page on Reachability. It describes Floyd-Warshal Algorithm, Thorup's Algorithm, and Kameda's Algorithm, which can answer queries in _constant time_ with greater preprocessing time or memory use.
2. [GRAIL](https://www.researchgate.net/publication/225756202_GRAIL_A_scalable_index_for_reachability_queries_in_very_large_graphs) - Graph Reachability Interval Labeling uses multiple passes of randomized post order DFS traversals.
3. [FERRARI](https://arxiv.org/abs/1211.3375) - Paper introducing Flexible and Efficient Reachability Range Assignment for gRaph Indexing, which uses _approximate intervals_ with exact intervals to trade off space required by post order DFS for some additional queries.
4. [PReaCH](https://arxiv.org/abs/1404.4465) - Pruned Reachability Contraction Hierarchies. Uses pruning based on topological levels, DFS numbering, and RCH together.
