---
layout: post
title: File-Injection Attacks
subtitle: By Weilie Lin and Chenghong Wang
tags: [file injection attack, searchable encryption]
---

## Introduction
Previously, we have learned powerful security techniques like secure two-party computation, oblivious RAM that eliminate information leakage during the communication with servers. However, due to the expensive cost, it’s hard to deploy those techniques. Researchers came up with another efficient method called searchable encryption (SE) that allows the client to search keywords over encrypted files stored on an untrusted server while preserving some degree of privacy. The design is efficient at the expense of allowing query patterns and file-access patterns to be revealed to the server. The leakage was first thought to be acceptable but turns out to cause sensitive data leakages. The file-injection attacks are designed to prove the serious leakage by showing that the server can determine the keywords corresponding to the tokens. Since the token for each keyword is deterministic, the server can learn the query pattern on repeated queries, additionally, the returned file identifiers reveal the file access pattern. 
## Binary-Search Attack
The binary-search attack does not require the server to have any knowledge about the files and can recover all keywords been searched with 100% accuracy. The algorithm is designed as follows. Suppose for simplicity *|K|* is a power of 2 and we choose *K = 8* as an example and identify K with the set *\{0,1,...,|K|-1\}* Written in binary. 
The server first generates a set F of *log|K|* files to be injected, where the ith file contains the keywords whose ith most significant bit equal to 1.Then for each token returned, the server learns the keywords corresponding to the location. 
![](https://i.imgur.com/9EMqhyN.png)
For example: if file 3 is returned in response to some token, but file 1 and file 2 are not, the keyword corresponding to that token is *k_1*. Same applies to *k_2* and *k_4*. If file 2 and file 3 are returned in response to some token, but file 1 is not, then the keyword corresponding to that token is *k_3*, same applies to *k_5* and *k_6*. If all files are returned in response to some token, then the keyword that corresponds to that token is *k_7*. If no files are returned in response to some token, then the keyword that corresponds to that token is *k_0*. In this way, the server learns all the relationships between tokens and keywords. 
A possible countermeasure is to limit the number of keywords per indexed file to some threshold much less than *|K|/2*. In this way, the server needs to inject more files to accomplish the goal. However, as shown in the next attack, the threshold counter measurement can be defeated using fewer injected files.

## Hierarchical-Search Attack 
The hierarchical-search attacks make use of the fact that the threshold countermeasure does not affect the binary-search attack with a small keyword universe *K’* in *K* if *|K’|<=2T*. The attacks work in the following steps:
> 1. Partition the keyword universe into *|K/T|* subset and each subset contains *T* keywords. 
> 2. The server generates *|K/T|* files, each contains keywords from a subset ad injects files to learn which subset the client’s keyword lies in.
> 3. Then it uses the small-universe, binary-search attack on these subsets to determine the keyword.

In step 2, the server injects *|K/T|* files. And in step 3, it injects *|K/2T|\*|log2T|* where *T* is the threshold of the maximum number of keywords in a file. So the total number to inject is *|K/2T|\*(|log2T|+2)*. We can further eliminate the number of files injected since for each i the first file in the set *F_i* is the same as *F_{i-1}* and the server does not need to inject it again. Additionally, the server does not need to generate the last file in step 2 as if the keyword is not in other files, it will be in the last file. And the total number of the injected file becomes: *|K/2T|\*(|log2T|+1)-1*
## Attacks Using Partial Knowledge
The number of injected files can be further decreased by leveraging prior information about the files (called the leaked file). The attack is based on the frequency of occurrence of the tokens and keywords in the client’s file. And there is the observation that if the leaked files are representative of all files, then the exact frequency of token t *f(t)* and estimated frequency of token k *f\*(k)* are close when t is the token corresponding to the keyword k.
To recover one keyword to token *t*. The server first observes the frequency of token *f(t)* and constructs a candidate universe *K’* for t which consists of *2T* keywords whose estimated frequencies are close to *f(t)*. Then by using a small-universe, binary-search attack, the keyword will be recovered. This attack only works for SE schemes without satisfying forward privacy and may fail due to the uncertainty of the estimated frequencies.
To recover multiple keywords, the paper proposed a complex attack consisting of two steps.
In the first step, it will recover a small set of n (n<<m) keywords as “ground truth”. The server makes a candidate universe for n tokens such that the union does not exceed 2T keywords. And then we can use the small-universe, binary-search approach to recover the keywords.
The second step is based on the observation that if *k’* is the keyword corresponding to *t’*, then the observed joint frequency *f(t, t’)* should be “close” to the estimated joint frequency *f*(k, k’)* for all pairs *(t, k)* in ground-truth. The attacker can construct a small set of *k’* of candidate keywords for *t’* by discarding candidates that do not satisfy the observation. Then we can recover keywords by using the small-universe, binary-search method.



## Experiments 
### Experiment Setup
> **Dataset**: To evaluate the attack, we use the Enron email dataset, consisting of 30,109 emails from the “sent mail” folder of 150 employees of the Enron corporation that were sent between 2000–2002. We extracted keywords from this dataset as in CGPR15: words were first stemmed using the standard Porter stemming algorithm, and we then removed 200 stop words such as “to,” “a,” etc. Then we pick the top 5000 most frequent keywords for the experiment.
> **Threshold countermeasure**: *T\gets200*, the reason for picking this value is that only 3% of the files may contain more than this many keywords.
> **Client query**: The client uniform randomly selects a keyword from the entire keyword space and generates a search query.


### Recovery of Single Token
The following figure shows the performance of our attack for recovering the keyword associated with a single token. According to the analysis, the server only needs to inject $\log(2T)=9$ files to process the attack. It can be observed that the inference attack performs quite well even with only a small fraction of leaked files, e.g., recovering the keyword about 70% of the time once only 20% of the files are leaked, and achieving a 30% recovery rate even when given only 1% of the files.
![](https://i.imgur.com/QNc8NOu.png)


### Recovery of Multiple Tokens
We have also provided experiments that target the recovery of the keywords associated with 100 tokens (*m=100*); The following figure shows the results for recovering tokens with our proposed file injection attack as well as the CGPR15 method. Both attacks do well when the fraction of leaked files is large, however, the recovery rate of the CFPR15 attack drops dramatically as the fraction of leaked files decreases. In contrast, our attack continues to perform well, recovering 65% of the keywords given access to 50% of the client’s files, and still recovering 20% of the keywords when only 10% of the client’s files have been leaked.
![](https://i.imgur.com/iW8xaZQ.png)


## Countermeasure
### Keyword Padding
The basic idea of **Keyword Padding** is to distort the real frequency of each keyword *k* by randomly associating files that do not contain that keyword with *k*; The aforementioned padding is done at setup time when the client uploads its encrypted files to the server. To test the feasibility of keyword padding, we have provided similar evaluation experiments, and the results are shown in the following figure.
![](https://i.imgur.com/axLD4ze.png)

 As shown in the previous figure, the recovery rate of our attacks degrades only slightly when keyword padding is used. This indicates that the keyword padding method is ineffective in defending against file injection attacks.


### Semantic Filtering
Another countermeasure is semantic filtering, as one may be tempted to think that the files injected by our attacks will not “look like” normal English text, and can therefore be filtered easily by the client. Thus the client will need to identify the injected files with semantic filtering methods and then request the removal of them from the cloud space. However, we argue that such an approach is unlikely to prevent our attacks. First, although as described our attacks inject files containing arbitrary sets of keywords, the server actually has some flexibility in the choice of keywords; e.g., the binary-search attack could be modified to group sets of keywords that appear naturally together. Second, within each injected file, the server can decide the order and number of occurrences of the keywords, can choose variants of the keywords (adding “-ed” or “-s,” for example), and can freely include non-keywords (“a,” “the,” etc.) 


### Batching Updates
The idea of batching updates is that rather than uploading each new file as it arrives, the client should wait until there are several (say, B) new files and then upload this “batch” of B files at once. Assuming only one of those files was injected by the server, this means the server only learns that the injected file corresponds to one of B possibilities. However, this countermeasure can be trivially circumvented if the server can inject B files before any other new files arrive. (If the server additionally has the ability to mount chosen-query attacks—something we have not otherwise considered in this paper—then the total number of injected files remains the same.) 


## Conclusion

Our paper shows that file-injection attacks are devastating for query privacy in searchable encryption schemes that leak file-access patterns. This calls into question the utility of searchable encryption and raises doubts as to whether existing SE schemes represent a satisfactory tradeoff between their efficiency and the leakage they allow. Nevertheless, we briefly argue that searchable encryption may still be useful in scenarios where file-injection attacks are not a concern, and then suggest directions for future research.


## Reference
[Enron](https://www.cs.cmu.edu/~enron/)
[All Your Queries Are Belong to Us: The Power of File-Injection Attacks on Searchable Encryption](https://eprint.iacr.org/2016/172.pdf)
