---
layout: post
title: Dynamic Searchable Symmetric Encryption
subtitle: By Chenghong Wang and Norah Tan
tags: [searchable encryption]
---


## Introduction
<!-- - CKA security and formal definitions of SSE/DSSE -->

In this blog, we will talk about the paper titled ''Dynamic Searchable Symmetric Encryption'' (DSSE) by Seny Kamara, Charalampos Papamanthou, and Tom Roeder. This paper presents a formal security definition to DSSE which captures a stronger notion of security for searchable symmetric encryption (SSE): CKA2 security. They also construct the first SSE scheme that is dynamic, CKA2-secure and achieves optimal search time. 

To have a better understanding of the paper, we will start with re-visiting the formal definitions of SSE, DSSE, and CKA security. Then, we will focus on discussing the inverted-index based DSSE. 

## Searchable Symmetric Encryption

Searchable symmetric encryption (SSE) allows a client to encrypt data in such a way that this data can still be searched. We adopt a general client-server setting where the client securely outsources its data to an untrusted cloud provider. Then, when the client issues a search request specified by some keyword, the server can return the correct encrypted record that satisfies the search condition. In this process, only allowed leakages can be revealed to the untrusted server. This idea of security relates to adaptive security against chosen-keyword attacks (CKA2). We will talk more about it in the later section. 

## Dynamic SSE

The client can generate search tokens to send as queries to the untrusted server. Given a search token, the server can search over the encrypted data and return the appropriate encrypted files.

### Set-up and Notations
The data can be viewed as a sequence of $n$ files ${\bf f} = (f_1,\cdots,f_n)$, where file $f_i$ is a sequence of words $(w_1, \cdots, w_m)$ from a universe $W$. We assume that each file has a unique identifier $\text{id}(f_i)$. 

The data is dynamic, so at any time a file may be added or removed. We note that the files do not have to be text files but can be any type of data as long as there exists an efficient algorithm that maps each document to a file of keywords from $W$. 

Given a keyword $w$ we denote by $f_w$ the set of files in ${\bf f}$ that contain $w$. If ${\bf c} = (c_1,\cdots,c_n)$ is a set of encryptions of the files in ${\bf f}$, then $c_w$ refers to the ciphertexts that are encryptions of the files in $f_w$.

### Definition
A dynamic index-based SSE scheme is a tuple of nine polynomial-time algorithms $$SSE = (Gen, Enc, SrchToken, AddToken, DelToken, Search, Add, Del, Dec)$$ such that:

<center>

| Algorithm   | property |  Inputs | Outputs
|----------|-------|------|------|
| $K \leftarrow Gen(1^k)$ |  probabilistic | a security parameter $k$ | a secret key $K$
| $(\gamma,{\bf c}) \leftarrow Enc(K,{\bf f})$ |    probabilistic   |   a secret key $K$; <br> a sequence of files ${\bf f}$ | an encrypted index $\gamma$; <br>a sequence of ciphertexts ${\bf c}$
| $\tau_s ← SrchToken(K, w)$ | possibly probabilistic |   a secret key $K$; <br> a keyword $w$ |a search token $\tau_s$
|$(\tau_a, c_f ) ← AddToken(K, f)$| possibly probabilistic | a secret key $K$; <br> a file $f$ | an add token $\tau_a$; <br> a ciphertext $c_f$
| $\tau_d ← DelToken(K,f)$ | possibly probabilistic| a secret key $K$;<br> a file $f$| a delete token $\tau_d$
| ${\bf I}_w := Search(\gamma,{\bf c},\tau_s)$ | deterministic | an encrypted index $\gamma$;<br> a sequence of ciphertexts ${\bf c}$;<br> a search token $\tau_s$ | a sequence of identifiers ${\bf I}_w \subset {\bf c}$
| $(\gamma^′, {\bf c}^′) := Add(\gamma, {\bf c}, \tau_a, c)$ | deterministic | an encrypted index $\gamma$;<br> a sequence of ciphertexts ${\bf c}$;<br> an add token $\tau_a$; <br> a ciphertext $c$| a new encrypted index $\gamma^′$; <br> a new sequence of ciphertexts ${\bf c}^′$
| $(\gamma^′, {\bf c}^′) := Delete(\gamma, {\bf c}, \tau_d)$ | deterministic | an encrypted index $\gamma$;<br> a sequence of ciphertexts ${\bf c}$;<br> a delete token $\tau_d$| a new encrypted index $\gamma^′$; <br> a new sequence of ciphertexts ${\bf c}^′$ 
| $f := Dec(K, c)$| deterministic | a secret key $K$; <br> a ciphertext $c$ | a file $f$

</center>

<!-- 1. $K \leftarrow Gen(1^k)$ is a probabilistic algorithm that takes a security parameter $k$ as input  and outputs a secret key $K$.
2. $(\gamma,\boldsymbol{c}) \leftarrow Enc(K,\boldsymbol{f})$ is a probabilistic algorithm that takes a secret key $K$ and a sequence of files $\boldsymbol{f}$ as inputs. It outputs an encrypted index $\gamma$, and a sequence of ciphertexts $\boldsymbol{c}$.
3. $\tau_s ← SrchToken(K, w)$ is a (possibly probabilistic) algorithm that takes a secret key $K$ and a keyword $w$ as inputs. It outputs a search token $\tau_s$.
4. $(\tau_a, c_f ) ← AddToken(K, f)$ is a (possibly probabilistic) algorithm that takes a secret key $K$ and a file $f$ as inputs. It outputs an add token $\tau_a$ and a ciphertext $c_f$.
5. $\tau_d ← DelToken(K,f)$ is a (possibly probabilistic) algorithm that takes a secret key $K$ and a file $f$ as inputs. It outputs a delete token $\tau_d$. -->

A dynamic SSE scheme is correct if for all positive integer $k$, for all keys $K$ generated by $Gen(1^k)$, for all ${\bf f}$, for all $(\gamma,{\bf c})$ output by $Enc(K,{\bf f})$, and for all sequences of add, delete or search operations on $\gamma$, search always returns the correct set of indices.

We can simplify SSE as $(Setup, Search)$ and DSSE as $(Setup, Update, Search)$.



## CKA2 Security
CKA2 security is a stronger notion of the adaptive security against chosen-keyword attacks. The paper also extends the normal notion of CKA2 security to the dynamic setting. Let $\mathcal{A}$ be a probabilistic polynomial-time (PPT) adversary: 
1. In a real experiment denoted as $Real_{\mathcal{A}}$, let $\mathcal{C}$ be the challenger and let us consider a randomly generated dataset $DS$: 
- $\mathcal{C}$ runs $Setup(DS)$ and reveals the outputs to $\mathcal{A}$.
- $\mathcal{A}$ adaptively chooses a set of search queries. For each query $q_i$, $\mathcal{C}$ runs $Search(q_i)$ and reveals the outputs to $\mathcal{A}$.
- $\mathcal{A}$ outputs a bit at the end as the return value of this experiment.

2. In an ideal experiment denoted as $Ideal_{\mathcal{A}, \mathcal{S}}$, let $\mathcal{S}$ be the simulator and let us consider profiles $(L\_setup, L\_query)$:
- $\mathcal{S}$ takes $L\_setup$ as inputs and outputs the results to $\mathcal{A}$.
- $\mathcal{A}$ adaptively chooses a set of search queries. For each query $q_i$, $\mathcal{A}$ sends $L\_query(q_i)$ to $\mathcal{S}$.
- $\mathcal{S}$ simulates outputs according to $L\_query(q_i)$ and return the outputs to $\mathcal{A}$.
- $\mathcal{A}$ outputs a bit at the end as the return value of this experiment.

We say that SSE is CKA2-secure if for all PPT adversaries $\mathcal{A}$, there exists a PPT simulator $\mathcal{S}$ such that
$$|\mathbb{P}[Real_{\mathcal{A}}(k)=1]-\mathbb{P}[Ideal_{\mathcal{A}, \mathcal{S}}(k)=1]| < negl(k).$$

Notice that by using the leakage $L\_setup$ and $L\_query$, it is sufficient enough to reconstruct all messages sent when evaluating the SSE scheme. In addition, after evaluating SSE, any PPT adversary cannot obtain information other than $L\_setup$ and $L\_query$. 


## Dynamic Searchable Encryption based on Inverted Index
- Introduce the inverted-index based DSSE

In this section, we first present the implementation of static SSE based on inverted index structure, and then discuss how to extend static SSE to DSSE. for ease of illustration, we refer to static and dynamic SSE as SSE-1 and SSE-2, respectively. 

### SSE-1 Implementation
According to the definition of static SSE, we abstract $SSE-1=(\Sigma, \texttt{Setup}, \texttt{Query})$ as a suite of a symetric encryption scheme $\Sigma = (Enc, Dec, Gen)$ with two secure protocols, $\texttt{Setup}$, $\texttt{Query}$. In the following sections, we will describe the details of how to implement these two protocols. 

#### $\texttt{Setup} (\lambda, {\bf f}, {\bf w})$
> $\lambda$, The security parameter
> ${\bf f} \gets \{ f_1, f_2, f_3 ..., f_m\}$, is the collection of all files to be encrypted.
> ${\bf w} \gets \{w_1, w_2, w_3,...,w_n\}$, is the collection of all keywords.
1. First, for each file $f_i$ the client creates an unique file identifier $idx_i$.
2. For each keyword $w_i \in {\bf w}$, the client find out all files that contain the keyword, and generate a linked list as follows: (i)Initiate a pointer as the header of the linked list; (ii) For each selected file $f_j$, computes $val \gets idx_j\oplus G_K(w_i)$, where $G_K(w_i)$ is a PRF. (iii) Create a node with the value $val$ and append this node to the end of the linked list. And repeat the above steps until all the $val$ corresponding to the selected files are appended.
3. The client selects a PRF function $F_K()$ such that for any given keyword $w_i$, $F_K(w_i)$ returns the header address of the corresponding link list generated in step 2.
4. The client encrypts each all files, and for each ciphertext $c_i$ of file $f_i$, label it with the file's unique identifier $c_i \gets <idx_i, c_i>$.
5. The client uploads all the linked lists generated from step 2 as well as all encrypted and labeled files to the server.


#### $\texttt{Query} (F_K, G_K, w_i)$
> $F_K$ the PRF that maps the keywords to its's corresponding linked list header address.
> $G_K$ the PRF function that is used to mask the file's unique identifier.
> $w_i$ the keyword for the search query.
1. The client sends the tokens $(F_K, G_K, w_i)$ to the server.
2. Upon receiving the token, the server applies $F_K(w_i)$ and locate the linked list for keyword $w_i$. Starting from the header, the server traverses the entire linked list, and for each visited, retrieves the node value $val$, and then recovers the corresponding file ID by computing $idx_j \gets val \oplus G_K(w_i)$
3. After all file IDs associated with that link were recovered, the server returns the corresponding ciphertexts according to the retrieved IDs.


### SSE-2 Implementation
In this section, we discuss the extension of SSE-1 to support runtime updates (i.e. inserting and deleting encrypted data). We mainly focus on the implementation of $\texttt{Add}$ and $\texttt{Delete}$ protocol.

#### $\texttt{Add}(idx_{f'}, c'_i)$
> $F_K$ the PRF that maps the keywords to its's corresponding linked list header address.
> $G_K$ the PRF function that used to mask the file's unique identifier.
> idx_{f'} the unique identifier of file to be added
> c'_i the encryption of file $f'$
1. The client finds all keywords that the given file $f'$ contains, denoted as ${\bf w'} = \{w'_1, w'_2,...\}$.
2. The client sends $F_K, G_K, {\bf w'}$, and $f'$ to the server.
3. Then for each $w'_i \in {\bf w'}$, the server computes $F_K(w'_i)$ to locate the header address for $w'_i$'s linked list, and append a new node with value $val \gets idx_{f'} \oplus G_K(w'_i)$ at the end of the linked list.
4. After all nodes has been added, the server append $<idx_{f'}, c'>$ to the encrypted data.

#### $\texttt{Delete}(f^*)$
> $f^*$, The file to be deleted
1. The client needs to outsource a new set of linked list structures, where each linked list belongs to a specific file in ${/bf f}$. The client finds all the keywords contained in the file and appends their corresponding keyword hash (computed by $F_K$) to that linked list. To clarify, we refer the aforementioned linked list structure as secure index, and the newly created linked list as deletion array.
2. When a deletion request is posted, the server receives file $f^*$ and lookup the deletion array to retrieve the corresponding linked list for $f^*$. 
3. The server dump out all keywords that file $f^*$ contains, and search through the secure index to locate nodes corresponding to $f^*$. The server then remove all nodes with respect to $f^*$ from the secure index.
4. After all nodes of $f^*$ has been removed, the server deletes the pair $<idx_{f^*}, c*>$ from the encrypted data.

