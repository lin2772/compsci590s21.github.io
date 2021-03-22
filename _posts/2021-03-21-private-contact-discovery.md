---
layout: post
title: Private Contact Discovery
subtitle: By Jian Yao and Yuqing Zhang
tags: [trusted hardware]
---
# SGX Contact Discovery
Software Guard Extension(SGX) is a feature supported by modern Intel chips which allows applications to run within a "secure enclave" that is isolated from the host operating system(host OS) and kernel. The enclave protects application from being eavesdropped by the host OS. In addition, SGX supports a feature called *remote attestation* which provides a cryptographic guarantee of the code running in a remote enclave over a network.

For a contact discovery service running within a SGX enclave, nothing about the contents of the client request should be observed by the remote server and host OS. Remote attestation encures that the service is running as expected. Hence, ideally the client executing queries in the enclave should be the same as running locally in terms of security. 

**Simple SGX contact discovery at high level:**
1. Start a contact discovery service in a secure SGX enclave.
2. Volunteering clients establish a secure connection through remote OS to the enclave.
3. Clients attest code running in the clave is the same as published open source code.
4. Clients send encrypted identifier from local to the clave.
5. The enclave loops through all registered users and returns the encrypted results containing contacts back to the client.

# Concerns about SGX security
Even though an SGX enclave prevents the host OS from being to see an enclave's memory contents, the host OS is still able to learn the *memory access patterns* of the service running within. For example, the function below finds registered users among the contacts of a client.

![](https://i.imgur.com/AsNsLC3.png)


The function above loops through all the contacts provided by a client and then checks for users already existing in the hash table containing registered users. There are two main security concerns about this function.
* The OS is capable of generating a layout of the hash table by tracking new user sign-up requests or by submitting its own requests to the enclave and observe the behavior of enclave. Given the knowledge of the layout, the host OS can monitor the memory access pattern when processing a new request and thus learn the content of the request. 
* In addition, a SGX enclave only supports an encrypted RAM of 128MB. The limitation forces the SGX service to store a large set of registered users outside the enclave. Then, the OS don't even need to learn the layout of the hash table from inside the enclave.

# Oblivious RAM Solutions
Elegant generalized ORAM techniques have limitations when dealing with the security issue with SGX enclaves. An ideal solution should be able to handle an extreme large number of keys and almost zero-sized values and also promising scalability. Most ORAM techniques such as Path ORAM are more favorable to situations where a small number of keys maps to large values. Complicated schemes like Recursive Path ORAM don't scale well and are difficult to build for concurrent access. Hence, an oblivious RAM solution for this issue needs to be constructed from scratch. 

A simple but inefficient solution is described below:

![](https://i.imgur.com/nuS63i9.png)


The solution abandoned the hash table structure. Instead, the function loops through all the registered users and contacts reported by client. A write is performed when a match is found. Since all the memories are being accessed, it made impossible for the host OS to learn about any memory access patterns. However, the scalability is compromised because of the linearity.

Another approach is to switch the structure of registered users and client contacts.

![](https://i.imgur.com/mjNEwdc.png)


The function above saves the dataset containing contacts of a client into a hash table rather than the set of registered users. Then, the function loops through all the registered users to find matches with elements in the hash table. It is still relatively inefficient comparing to the initial approach as the size of registered users is significantly larger than client contacts. Yet, it is much faster than looping through all the data. By looping through the set of registered users, the host OS is unable to discover which registered user is being accessed. Since the size of client contacts can be small, it is possible to fit the hash table inside the encrypted RAM. However, there is still chances that the host OS learns the layout of the clientContacts and thus making this approach not oblivious.

# Oblivious Hash Table
Why OS might be able to know the content or the arrangement of a normal hash table of client contacts? Let’s take a look at a common hash table construction which iterates and stores all the encrypted client contacts:

![](https://i.imgur.com/SmMqWYQ.jpg)


The above way to construct a hash table might reveal the layout of the hash buckets. For example, OS can learn whether a bucket is empty from the information of which hash table memory addresses received writes, and the OS might then know whether the enclave is checking an empty hash bucket during the findRegisteredUsers function mentioned before. In addition, attackers might even attach physical hardware (e.g., FPGA) to the memory bus to monitor memory access patterns outside the enclave.

Therefore, we need to make the construction of the clientContacts hash table oblivious. A possible but inefficient way (introduced by [Moxie Marlinspike (2017)](https://signal.org/blog/private-contact-discovery/) )to construct a hash table would be:

![](https://i.imgur.com/hdeEgOj.jpg)

As shown above, there are N +1 logical “buckets” (from Bucket 1 to N). Each of them contains two cache lines (from B1 to B64), the “request” cache lines and the “result” cache lines. The “request” one is used to store all the client’s submitted contacts from client queries, and the other one is for the storage of look-up results, where the enclave writes look-up result to the corresponding hash index of the “result” cache line for each contact. For the oblivious property, the enclave should write the look-up result for each contact to the same hash index in the “result” cache line no matter whether the look-up is successful or not. 

Based on the above hash table structure, the enclave can iterate over each cache line of the “request” and determine which client contacts should be stored there:

![](https://i.imgur.com/90tyzS7.jpg)


This client contacts hash table is now constructed obliviously, and it can prevent observers to reveal the content or the layout of the table, even though the OS might still be able to know some trivial knowledge. For example, the OS knows the capacity (N contacts) of each bucket, so if there were N+1 users which correspond to the same hash bucket, the OS could learn the fact that it’s impossible for all the N+1 users to appear in the client request, even though the enclave tried to hide this information by indexing into the cache lines more than N times. However, the OS does not know any other useful information about that, because it cannot figure out which of the visited registered users were in the client request. Therefore, as desired, the OS cannot reveal anything important from the oblivious hash table structure.


# Summary of secure contact discovery procedures in SGX

In conclusion, contact discovery service can have the following procedures to ensure security: 
1. Run the contact discovery service in a secure SGX enclave. Transmit a batch of encrypted contact discovery queries from the requesting client to the enclave.
2. The enclave decrypts the contact discovery requests and constructs the oblivious hash table to store all of the client contacts.
3. Iterate over all registered users through the enclave, and each of the users is compared with every contact in the oblivious hash table.
4. The enclave writes the look-up result to the corresponding hash index in the “results” cache line, irrespective of the comparison result.
5. The enclaves generates an oblivious response list for every client query after all the registered users are checked.
6. The enclaves encrypts and returns the batch result to the contact discovery service.
7. The service sends out the encrypted response to the client.


# Reference
[Technology preview: Private contact discovery for Signal](https://signal.org/blog/private-contact-discovery/)

