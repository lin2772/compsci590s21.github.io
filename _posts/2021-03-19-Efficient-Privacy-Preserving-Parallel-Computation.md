---
layout: post
title: Efficient Privacy-Preserving Parallel Computation
subtitle: By Qiaoyi Fang and Grayson York
tags: [Parallel Computation]
---


In this blog post we will discuss two papers which deal with a recent problem in cryptography. It is well understood that computation can be performed un untrusted servers using trusted hardware and oblivious RAM, but if a computation would benefit from being executed on a parallel architecture, it is possible that that parallelism will leak information about the underlying dataset. In this post, we will analyze two solutions to the problem of implementing secure computation in parallel, secure map reduce and GraphSC. Each of these allows a user trying to perform a secure computation to efficiently utilize a parallel architecture, while exposing little to no information about his underlying data.

# Efficient MapReduce Under SGX

## Why Secure Map Reduce Is Necessary

Secure computation on individual machines is a well studied problem. Solutons exist involving performing secure computation on a secure enclave which that operating system cannot observe, and therefore cannot gain information to which it should not be privy. However, distributed computation is becoming more and more popular, and if a computation is running in parallel on two seperate machines, then the communication between those machines may leak information to an adversary observing network traffic. A common protocol for setting up such a parallel system is called map reduce, named after its two stages. In the first stage, jobs are "mapped" to M machines which will compute those jobs in parallel, and in the second, the results of these jobs are "reduced" to one concise result by R machines (possibly the same machines as before, or possibly new ones). One very popular implementation of mapreduce is the Hadoop infrastructure for distributed computation.

## A Motivating Example

Let us say that you are considering opening a rideshare service to compete with Uber and Lyft. In order to prepare for a VC pitch, you need to do some market research on taxis in New York City. You do not have enough computation power to process the entire dataset on your own servers, so you look to do the computation on the Google cloud. You know that Google is considering launching their Google car in NYC, and you would rather not reveal to a potential competitor that you are considering entering the market.

Fortunately, you have plenty of tools at your disposal to hide the specifics of the computation you are running from Google's prying eyes. Using encryption of the dataset and trusted hardware you can be confident that Google will not be able to see what kind of computations are begin performed on their machines. However, you would like to use Hadoop to run your computation in parallel, but you are worried that the way that Google's machines communicate with each other might reveal more information than you would like.

For example, imagine that you want to see how many rides were taken on each day of the last month. A map reduce might find the list of all taxi rides which happened on each day, and send the appropriate lists to the appropriate reducers in order to count them. An unscrupulous Google employee may see how many datapoints were sent to each reducer and realize that those numbers corresponded exactly to the number of taxi rides on a given day in New York. Then this employee could go to the google car division and warn them that a potential competitor was researching entering the market.

## What Would Secure Map Reduce Look Like?

The security of a map reduce system can be proved using a cryptographic game in which an adversary tries to gain access to information that he should not have by sending information to the secure model and observing the results. In the map reduce game, the adversary creates two datasets D0 and D1, and decides M and R. He then sends D0,D1,M, and R to someone using the map reduce implementation that we would like to test, and that user sends back \[D\], the network traffic data from the computation performed on either D0 or D1. This data will contain only the network traffic, and no information about the actual computation which was performed because the computation will be done on trusted hardware which will not reveal the computations to the adversary. It is then the job of the adversary to guess which dataset was used in the computation which resulted in \[D\]. If no adversary can distinguish between any pair of datasets with some reasonable amount of computation power, the n we know that it is very difficult for an adversary to gain any information about the data that was used in the computation

We will impose an additional restriction on the adversary, and propose a few conditions that the protocol must satisfy. We must force the adversary to use two datasets of the same size then hope that the adversary cannot win the map reduce game, and hope that
1. Each mapper produces the same number of outputs
2. Each reducer recieves the same number of inputs
3. The output size of the reduction is independent of the data

## The Solution

First, we will find a way to conceal which mappers are sending data to which reducers. Then, we will find a way to conceal how large of a job each mapper sends out and how large of a job each reciever recieves.

The trick to concealing which mappers are sending data to which reducers is to use another round of MapReduce to obliviously shuffle the data. Each mapper will send its outputs along with a permutation as a new mapreduce job. This job will be run on some mappers and reducers and compute the shuffle obliviously. After this shuffle is computed, we will send the output to the reducers to perform their reduce operation on. In this way, all of the mapper-reducer links are concealed.

Unfortunately, even with this modification to the protocol, the attacker can still learn information about the dataset. In particular, if one mapper outputs a lot more data than another, or if one reducer recieves an abnormally large amount of data, then an attacker will be able to tell, and use that information to make judgments about the underlying dataset.

The easiest way to fix this vulnerability is to simply determine a theoretical maximum output of any mapper, and pad every mapper's output with dummy data in order to reach that size. Unfortunately, this method requires some knowledge of the codebase which uses the map reduce in order to run in some reasonable amount of time. The more accurately we can predict the maximum output of a mapper, the fewer unnecessary padding blocks we need to add. In addition, prior to any computation being performed, we will shuffle the data randomly, then attempt to assign jobs to mappers which balance out the amount of data output by each. For this reason, the protocol is called shuffle and balance.

Some tricks can be employed to facilitate this balancing. First, we can see that randomly assigning the data to mappers will help prevent output sizes from being correlated. This will help when we attempt to bound the maximum possible output size (with high probability). In addition, we can sample some subset of the data prior to running the map reduce on the full dataset and observe the sizes of the mapper outputs. Then, we can collect statistics on this smaller dataset, and based on these statistics estimate the maximum output of a mapper. In theory, problems could occur if the sample is not representative, but fortunately if the sample is selected randomly, the odds of a nonrepresentative sample are extremely low.


## Performance

The authors tested their algorithm on several datasets, most notably a sample of taxi rides in New York over some period of time. This dataset spans over several months, and included 2.5 gb of data per month. The authors attempted to aggregate the data based on various factors in parallel on these datasets. They found that the shuffle and balance protocol ran in almost the same amount of time as a standard map reduce with no security. On the 4 month, 10gb dataset, the worst slowdown was 20%, and on the 2.5 gb dataset, the worst slowdown was 12%. This means that shuffle and balance can be implemented on fairly large datasets for only a small increase in total runtime of the algorithm.

# GraphSC: Parallel Secure Computation Made Easy

## The Need for Parallel Secure Computation
GraphSC was created at a time when the massive scale of data has motivated the invention of parallelism in computational architecture while secure parallelism remained an open question. For example, despite with great amount of data on social relations available, data engineers were unable to understand the joint influences of these social graphs which transcend multiple platforms (such as Facebook and LinkedIn), because platforms were unable to share them securely. This necessitates oblivious graph-based parallel algorithms for secure multiparty computation on graphs. We need algorithms to fulfill the privacy requirements for no graph information leakage revealing, the performance requirements for scalability and the efficiency demand of parallelizability. In addition, as major parallel architectures such as MapReduce and GraphLab were widely adopted, we need the algorithms to be accessible to programmers who were not expert in cryptography. Therefore, the researchers ask "_Can we build an efficient secure computation framework that uses familiar parallel programming paradigms?_"

In the following sections, we will first explain what graph-parallel algorithms are using the example of the PageRank algorithm. Then we will discuss what we mean by "obliviousness" on graph computation. Then, we will present two naive solutions to this problem. Finally, we will introduce GraphSC, an efficient solution to the oblivious problem. 

## Graph-parallel Algorithms
Before diving into the model and implementation of GraphSC, we would like to briefly introduce the paradigm of graph-parallel algorithms and the requirements for obliviousness. In the graph-parallel paradigm, a graph is an ordered pair \\(G=(V,E)\\) consisting of: \\(V\\), a set of vertices, and \\(E \subseteq \{\{x,y\}|x,y \in V \mathrm{and~} x \neq y \} \\). Every vertex is data-augmented and \\(v \in V\\) performs computations based on the data associated with neighboring edges and vertices and itself in a parallel fashion. More specifically, there are three main operations: 

- _Scatter_: A vertex propagates data to its neighboring edges and updates the edges' data 
- _Gather_: A vertex aggregates data from neighboring edges and updates its own data
- _Apply_: Vertices perform local ccomputation on their own data

### Example: PageRank
The above framework of _scatter_, _gather_ and _apply_ supports the implementation of a wide variety of graph-based algorithms, including computing gradient descent, creating histograms, and matrix factorization. In this section we discuss one important example, PageRank (PR), the algorithm used by search engines to rank web pages in their search results. PageRank works by counting the quantity and quality of links to a web page to measure the importance of a website. Therefore, the more links a website has from other websites, the higher PR it has. To be specific, for each vertex \\(u\\) of a graph \\(G\\), PageRank computes a ranking score \\(PR\\) through a repeated iterating process of updating based on the following assignment: 
$$
\begin{aligned}
PR(u) = \frac{0.15}{|V|}+0.85*\sum_{e(u,v)\in E}\frac{PR(v)}{L(v)}, \forall u \in V,
\end{aligned}
$$
where  \\(L(v)\\) is the number of outgoing edges, and all vertices are initialized to have a PR of \\(\frac{1}{|V|}\\). 

PageRank algorithm can also be expressed in the paradigm of the three primitive operations: _scatter_, _gather_ and _apply_:
```{python}
function computePageRank(G(V,E,D)):
    fs(e.data, u.data): e.data:= u.data.PR/u.data.L
    ⊕(e1.data, e2.data): e1.data + e2.data
    fa(v.data): v.data.PR := 0.15/|V| + 0.85 * v.data.agg
    for i = 1 to K:
        Scatter(G, fs, "out")
        Gather(G, ⊕, "in")
        Apply(G, fa)
```
in which the data of each vertex \\(v\\) stores PageRank of \\(v\\) and the number of outgoing edges \\(L(v)\\) while the data of each edge \\(e(u,v)\\) stores the weight contribution of PageRank to the outgoing vertex \\(u\\). 

## Oblivious Graph-parallel Algorithms
Now that we have explained what a graph-parallel algorithm is and potential applications, we can introduce the property of obliviousness. As discussed in the initial section, the privacy needs of computing sensitive graph data have motivated us to ask the question: how we can compute graph-based algorithms without revealing the data stored in the graph. In order to allow this obliviousness in graph-parallel algorithms, we want to achieve the following:
+ only reveal the final output 
+ not reveal data on \\((V,E)\\)
+ not reveal the graph structure and ensure that the information of degrees and neighbors remain hidden to other vertices. 

### Naive Solutions
An naive solution that achieves obliviousness on the algorithm level would be to carry out operations to every vertex from every vertex to hide any information about graph structure and data. This approach would take \\(O(|V|^2)\\) time, which is far from ideal.
Another naive approach would be on the circuit level. One could write programs using a language designed for sequential secure computation and then use a program-to-circuits compiler to take advantage the security and parallelism that occur at the circuit level. However, the solution is also far from ideal because the converted circuits would be sequential in nature and this process sacrifices many opportunities for pallelism.  


## GraphSC Solution
Compared to the listed two naive approaches, GraphSC is an efficient framework solution for graph-based parallel computation that supports invocations of basic operations. Inspired by GraphLab and Pregel, it is generalizable by nature and allows the migration of parallel data mining and machine learning algorithms. 

### Algorithm
In this section, we describe the algorithm specifics of its unique graph representation and the three primitives _Scatter_, _Gather_, and _Apply_ in the single-processor setting. 

#### Graph Representation
One key challenge of graph-based oblivious algorithms is hiding graph structure during computation. GraphSC uses an alternative graph representation, expressing tuples in the form <u, v, isVertex, data>, in which each vertex u is represented by the tuple <u, u, 1, data> and each edge (u, v) by <u, v, 0, data>.

#### Three Primitives
With the graph representation defined, we now describe the implementation of GraphSC primitives:  _Scatter_, _Gather_, and _Apply_:

#### _Scatter_
In the _Scatter_ primitive, we first perform an oblivious sort on G in order to group tuples with the same source vertex together and to ensure that each vertex appears before all the edges originating from it. Next, we conduct a single linear scan to update the value of each edge cell with the nearest preceding vertex cell based on the user defined function \\(f_s\\). The _scatter_ operation is oblivious because its two components, oblivious sort and propagation, both do not reveal any information of graph data and structure, as the algorithm treats both vertex and edge data indistinguishably. The operation can be performed in \\(O(M\log|M|)\\) for \\(M=|V|+|E|)\\) and using an oblivious sort with \\(O(M\log|M|)\\).

```{python}
# G: list of tuples <u, v, isVertex, data>, 
# M: |V|+|E|
# fs: user-defined function
# b: "in" or "out"
function Scatter(G, fs, b):
    sort G by (u, -isVertex)
    for i := 1 to M: # Propagate
        if G[i].isVertex:
            val := G[i].data
        else:
            G[i].data := fs(G[i].data, val)
```
#### _Gather_ 
Similar to _Scatter_, we first perform an oblivious sort and then conduct a linear scan to update each vertex cell with the ⊕-sum of the longest preceding sequence of edge cells so that values on all edges ending at a vertex are aggregated into it. The oblivious argument of _Gather_ is similar to _Scatter_, and the time complexity of the operation is also equal to that of the _Scatter_ operation.
```{python}
# G: list of tuples <u, v, isVertex, data>, 
# M: |V|+|E|
# fs: user-defined function
# b: "in" or "out"
function Gather(G, b):
    sort G by (u, -isVertex)
    agg := 1⊕
    for i := 1 to M: # Aggregate
        if G[i].isVertex:
            G[i].data := G[i].data || agg
            agg := 1⊕
        else:
            agg := agg ⊕ G[i].data
```
#### _Apply_
We perform a linear scan over the list and apply the user-defined function \\(f_a\\) to each vertex tuple in the list and a dummy operation to each edge tuple to keep the operation oblivious. The operation can be performed in time \\(O(\log|M|)\\) for \\(M=|V|+|E|)\\).
```{python}
# G: list of tuples <u, v, isVertex, data>, 
# M: |V|+|E|
# fa: user-defined function
# b: "in" or "out"
function Apply(G, fa):
    for i:= 1 to M:
        G[i].data := fa(G[i].data)
```

### Parallelize GraphSC 
Having described the GraphSC algorithm, we now briefly argue that _Scatter_, _Gather_ and _Apply_ in GraphSC can be parallelized given that sufficient resources of processors are available. As _Apply_ is easily parallelizable and both _Scatter_ and _Gather_ need oblivious sort and _aggregate/propagate_ operations, it would suffice to show that GraphSC is parallelizable if we could show oblivious sort and _aggregate_ and _propagate_ operations are parallelizable. As the oblivious sort is a \\(\log (|V|+|E|)\\)-depth circuit, it is easy to parallelize directly at the circuit level. Both _aggregate_ and _propagate_ operations are also shown in the paper to be parallelizable by taking advantage of the unique graph representation of GraphSC. Therefore, GraphSC is also parallelizable.

## References
[Efficient MapReduce Under SGX](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/MSR-TR-2015-70.pdf)
[GraphSC: Parallel Secure Computation Made Easy](https://users.cs.duke.edu/~kartik/papers/3_graphsc.pdf)
