# Paxos Algorithm

I found Paxos Algorithm quite hard to understand by reading the introduction by wikipedia and Leslie Lamport's original paper. Therefore, I develop a single page here to depict my understanding of the algorithm. 

Lamport starts decribing the algorithm from its origin, which is confusing for me due to the fact that I don't have knowledge regarding the history background he derives algorithm from. 

The goal of paxos algorithm is to reach consensus in a distributed system. 

 _"Consensus is the process of agreeing on one result among a group of participants. This problem becomes difficult when the participants or their communication medium may experience failures."_ 

from:  [https://en.wikipedia.org/wiki/Paxos\_\(computer\_science\)](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)

Paxos Algorithm numerates each and every state on all Servers in the Cluster. Obviously,  a concensus would be reached if a one and only sequence of natural numbers are distributed to states. How Paxos decides which number should be distributed to certain state on certain server is by voting. Paxos lets all servers participate in voting, and the number distributed is decided by majority voting. The number decided by voting here is called a _value_. 

What is crucial in Paxos Algorithm is that **"only a single value is chosen"**. 

As a supplement to the original introduction listed in reference, here I distinguish the actions Acceptors could do to Proposers:

1. **promise**: Acceptor promises to Proposer that Acceptor would accept the value the Proposer proposes, if there is no proposal that proposes a larger value. 
2. **accept**: Acceptor accepts the proposal, now that the Acceptor doesn't receive and proposal that proposes a larger value than the former one. 
3. **chosen**: a proposal is finally chosen, when the majority of Acceptors accept the proposal. 

**P1: An acceptor must accept the first proposal that it receives.** 

P1 seems to be unfair to the second proposal: given the circumstance where all acceptors are given the same first proposal, the first proposal would be chosen no matter what. 

Therefore, from my perspective, the word "accept" should be corrected as the action "promise" as is distinguished above. 

It is clear that an algorithm based on P1 only could not ensure a majority vote, and what an acceptor should do to the second proposal is not ruled as well.  

**P2: If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.** 

Astonishingly, P2 does not follow the path of P1. P2 does not solve the problem of ensuring a majority vote, and what should an acceptor do to the second proposal either. While P1 focuses on how to deal with the first proposal, P2 abruptly jump to describing how a decision is maintained once a choice is made. There seems to be a huge gap between them. 

Disregarding P1, P2 is a "safety property". P2 has 3 requirements: 

**P2a: If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v.**

P2a is P2's requirement for acceptor. 

**P2b: If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v.** 

P2b is P2's requirement for proposer. 

**P2c: For any v and n, if a proposal with value v and number n is issued, then there is a set S consisting of a majority of acceptors such that either**

1. **no acceptor in S has accepted any proposal numbered less than n, or**
2. **v is the value of the highest-numbered proposal among all proposals numbered less than n accepted by the acceptors in S.**

P2C is P2b's requirement for both acceptor and proposer. Because it is hard to meet P2b's conditions, P2c provides a path which makes anything meeting P2c's conditions satisfies P2b. 

The proof is given in the original paper. 



#### Reference: 

\[1\]  Leslie Lamport, _"Fast Paxos"_, 14 July 2005, MSR-TR-2005-112, to appear in _Distributed Computing_. 

\[2\]  [https://en.wikipedia.org/wiki/Paxos\_\(computer\_science\)](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29) 



