---
layout: post
title: Secure Two-party Computation in Sublinear Time
subtitle: By Yanping Zhang and Rahul Ramesh
tags: [Two-party computation, ORAM, Yao Garbled Circuits]
---

# Introduction

Current two-party computation models are based around building boolean or arithmetic circuits to access information. This can be seen with GMW, Yao's etc. However, any operation computed in this fashion must have at least a linear time. For example, if a user only accesses certain elements while executing a binary search, an adversary could gain information based on the user's access patterns.

At the same time, modern day technologies require both the ability to work with large databases and the ability to prevent data leakage. A linear runtime is simply too slow in several use cases. The paper thus identifies two criterion of a new protocol.

1. Using RAM as a starting point (instead of circuits) to compute sublinear functions
2. Using preprocessing to improved the amortized runtime of running the same function multiple times

The goal of this paper is to apply oblivious RAM principles to accomplish these criterion. 

The paper focuses on a server with a large database, and a client who wants to compute a function repeatedly with different inputs. The client may want to update the contents of the database. The paper focuses on the semi-honest model for two parties, though it can be extended to work with malicious adversaries in a multiple-party model. Security is proven if no data is leaked from the computation of the functions.


# Generic Protocol

## Overview

The paper's protcol is based on Oblivious RAM, or ORAM. ORAM allows a client to securely compute functions with the memory of a possibly adversarial server.

The ORAM has two innate functions that are applied. First, the ORAM has an initialization process, which generates keys and creates an empty array for the ORAM data structure. Second, ORAM execution, which securely takes the current ORAM state and the virtual instruction to be run, outputting the actual instruction to run and the updated ORAM state.

## Functions

The paper's protocol has three main functions: secure initialization, secure evaluation, and the doInstruction subroutine. 

1. For secure initialization, the client and server both run ORAM initialization (key generation) to get a secret share of the initial ORAM state. Then, for each element in the database, the server uses the ORAM's execution to insert each of the elements of the database into memory.

2. For secure evaluation, both server and client hold secret shares of an ORAM state. The two compute the value of the function using secure computation and the next instruction function. After computation, the server sends the client whatever share of the output it received, so the client can combine the shares to recover the output of the function.

3. For the doInstruction subroutine, the client and server repeatedly use the ORAM's execution to obtain the real instructions from the desired virtual instruction. Anytime a read instruction is computed, the server shares its secret of the value with the client.

## Runtime

Importantly, each secure computation is run over a small input, which intuits the amortized runtime over numerous calls of the function.

Formally, the paper argues that if the function can be computed in time t and space s, then the amortized runtime is O(t)*polylog(s), the client's space is O(log(s)), and the server's space is O(s)*polylog(s).



# Optimized Protocol

## Overview

For the generic protocol, we compute the entire next ORAM instruction inside a secure computation protocol. As for a more efficient and practical scheme, rather than applying the expensive secure computation primitive on the entire ORAM instruction, we can perform parts of the secure computation locally. 

Specifically, the client and server only have secret shares of instructions (accessed address) and ORAM states (leaf identifier of the last tree of ORAM). Thus, whenever they compute any functions using these values, they need to perform secure two party computation over the secret shares. In the end, they will output secret shares of the accessed data. 

## High Level Protocol

This paper illustrates how to construct an optimized protocol based on Yao’s garbled circuit and tree based ORAM. Past blogs have explained the tree based ORAM, so we will explain where we need to perform secure computation when executing ORAM. 

We mentioned that the client and server need to execute secure computation protocol when they need to know the accessed address and the leaf identifiers of the smallest tree of ORAM. So when do they need to access these values?  When the client looks up position map to find the leaf identifiers of the accessed address, the client needs to know the accessed address and leaf identifiers of the smallest tree. Besides, when the client needs to remap the accessed block to a new path and add the access block back to the root, the client needs to know the address of the accessed block. But the client only have secret shares of these values, so the client and server execute two-party computation to achieve these functions over the secret shares they have. 

For a recursive based tree ORAM, the protocol is almost the same, except that the client and server need to perform secure computation to compute the accessed addresses for each recursive trees. 

# Conclusion

We introduces a generic solution to sublinear 2PC applying to any oblivious RAM and any 2PC protocol, and an optimized efficiency construction based on tree-based ORAM and Yao’s garbled circuit. It has significantly asymptotic improvement over traditional generic secure computation techniques.

