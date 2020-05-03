---
layout: post
title:  "Toward modular network verification"
authors: [todd]
categories: [research, network, verification]
image: assets/images/jigsaw.png
tags: []
---

Almost all of the techniques for network verification to date must analyze the entire network's state and/or configuration data monolithically.  In contrast, methodologies for verification in other domains support *modular* (or *compositional*) reasoning:  it is possible to verify a system by separately verifying its parts.  Modularity allows verification to scale to large systems, which is a key challenge for network verification today, and it provides many other system development and management benefits.  This post explains what modular verification is, why it is important, and how we might achieve forms of modular network verification.

## What is modular verification?

In a modular approach to verification, each component of a system is given a *specification*, and each component is separately verified to meet its specification.  Importantly, verifying each component only requires access to the specifications, not the implementations, of the components with which it interacts.  

As an example, consider a simple software system containing a main function $f$ that calls another function $g$.  To prove that $f$ meets some intended specification $\varphi$, a non-modular approach would analyze the behaviors of $f$ and $g$ jointly.  A modular approach, which is common in traditional software verification based on pre- and postconditions, is to instead define a specification $\varphi'$ for $g$ and thereby break the verification of $f$ into two independent parts:

* Prove that $ g $ meets the specification $\varphi'$.

* Prove that $f$ meets the specification $\varphi$, under the assumption that $g$ meets the specification $\varphi'$.

## Why modular verification?

Modularity will provide a number of important benefits for network verification that are absent today.

*Scalability.* Verification must explore all possible system behaviors.  Since the number of system behaviors can grow exponentially in the system size, monolithic verification is inherently unscalable.  And indeed, today the most complex forms of network verification, for example ensuring that a network’s control plane satisfies desired reachability properties in the face of any set of $k$ link failures, do not scale to large networks.  Modular verification can provide asymptotic gains in performance.  Roughly, if there are $n$ components each with $m$ possible behaviors, then the system as a whole can have up to $m^n$ behaviors, while a modular approach will only explore $m * n$ behaviors.

*Independent development.*  Since each component's correctness only depends on the specifications, but not the implementations, of the other components, modular verification allows each component to be developed and validated independently.  In the context of network verification, this means that different teams can be responsible for managing and validating different parts of a network, without requiring access to the configurations of other parts.

*Incremental re-verification.*  Similarly, as the system is updated, it can be incrementally re-verified.  For example, if a component $\mathcal{C}$ changes but its specification is unchanged, then only $\mathcal{C}$ must be re-verified.  Network configurations are changed daily, to enable new services, perform security updates, etc.  Fast incremental re-verification is critical to making network verification practical.

*Design verification.*  Finally, an under-appreciated benefit of modular verification is that it verifies not only the end-to-end properties of a system but also the system's internal design.  A good system has a modular structure, with different components responsible for different tasks, and networks are no exception (see the next section).  Modular verification directly validates this structure by verifying that each component properly performs its tasks, and verification failures are localized to the relevant component and task within the system.

## Modular network verification

Networks have a lot of structure that can be exploited for modular verification, at different granularities.

*Subnetwork modularity.* Many networks are composed of multiple subnetworks.  For example, a data center consists of multiple pods, and an enterprise network consists of multiple campuses.

*Protocol modularity.* Networks often run multiple protocols, each of which has a different purpose, for example OSPF for intradomain routing and BGP for interdomain routing.

*Role modularity.* Nodes in a network play different roles.  For example, a data center’s nodes are partitioned into leaf nodes, spine nodes, etc., and an enterprise network’s nodes are partitioned into border routers, core routers, etc.

*Function modularity.*  A router typically has multiple interfaces, each with its own policy.  Even for a single interface, the router configuration has separate components for different tasks, such as routing and access control.

I am only aware of two existing verification techniques that exploit this modular structure, and both target data-plane verification in data centers.  First, [Plotkin et al.](https://dl.acm.org/doi/pdf/10.1145/2837614.2837657?download=true) perform modular reachability checks on certain subnetworks in order to speed up verification, a technique that they call "surgery."  Second, [Jayaraman et al.](https://dlnext.acm.org/doi/pdf/10.1145/3341302.3342094?download=true) perform verification via separate, role-specific checks on each node in the network.


## Technical challenges

I believe that the field is ready to begin exploring all of the forms of modularity described above.  Some common technical challenges will arise, leading to important new insights and capabilities:

*What are the right specifications?* A good specification must be abstract, so that it hides much of the complexity of a component from the rest of the system, while also being precise enough to allow desired system properties to be validated.  Research is necessary to identify appropriate component specifications for each of the above forms of modularity.  I believe that this research direction will also lead to a new understanding of best practices for network design and configuration, for example to minimize dependencies across sub-networks in routing configuration.

*Where do specifications come from?*  Initially I expect component specifications to be written manually by tool developers or users.  Longer term, as in other forms of verification the community should investigate forms of automated *specification inference* for network components.  This direction can likely build on recent work on [network-wide specification inference](https://www.usenix.org/system/files/nsdi20-paper-birkner.pdf) and on role-level [configuration template inference](https://www.usenix.org/system/files/nsdi20-paper-kakarla.pdf).

*How to ensure end-to-end properties?*  Depending on the form of modularity exploited, modular verification may not directly ensure properties of the entire network.  For example, even if each router properly plays its network role, that does not directly imply end-to-end reachability properties.  Initially I expect the connection between modular verification and network-wide properties to be either unproven or proven manually.  Longer term, the community should investigate ways to automatically connect per-component specifications to end-to-end network properties.

## Summary

Now that the community has laid many of the foundations for network verification, exploiting modularity is a natural next step.  I believe that modularity will be necessary to make network verification a practical technology for real networks.  I’m excited to pursue this vision and hope that others will join me in making it a reality.

