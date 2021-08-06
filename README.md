# srl-auto-router-id
Agent to auto-assign router IDs using a distributed spanning tree algorithm over LLDP

[Back in 1983](https://wayback.archive-it.org/all/20161012010517/http://courses.csail.mit.edu/6.852/05/papers/p66-gallager.pdf) Gallager et al. from MIT wrote a paper about a distributed algorithm for determining a Minimum Spanning Tree in a network. This agent represents a modern Python based implementation, using LLDP (changing hostnames) to communicate messages between nodes.

Each node starts in SLEEPING, transitioning to FOUND when the agent starts. It then proceeds to FIND by initiating MST discovery.

There are 7 types of messages:
1. Connect(Level, starts at 0)
2. Initiate(Level,Fragment Identity,State(FIND or FOUND)) - can sometimes be multicast to multiple neighbors
3. Test(Level,Fragment Identity)
4. Report(best edge weight)
5. Accept
6. Reject
7. Change-root

Given the nature of LLDP, there may be a need for a confirmation protocol to avoid changing the hostname too quickly

## Pending messages at end-of-queue
The algorithm has 3 situations in which messages "are placed at the end of the queue":
1. Receive Connect with L>=LN, on a port in BASIC state
2. Receive Test with L>LN
3. Receive Report on in-branch port while in FIND state

These conditions can get cleared upon receipt of Initiate or Report, and basically form substates of FIND (FIND with pending Connect on port X, etc.)

## Edge weights
In this implementation, e1-1 has the lowest weight, then e1-2, etc. System MAC address serves as a tie breaker; the lower MAC ID wins.

## Allocating router IDs
Once the MST is determined, there is a root node (node connected to root edge with lowest port and/or system MAC). This root node gets ID 0 ( out of a configured range, say 1.1.0.0/22 then root=1.1.0.0/32 ). Subsequent levels get IDs according to the root port they are connected to: e1-1 = 1, e1-2 = 2, etc.
Note that in a spine-leaf CLOS this would lead to a situation where 2 "Spines" have IDs from different levels (spine1=level 0=root, spine2=level 2) - there could be a configuration parameter to collapse disjoint layers for the purpose of ID assignment.
