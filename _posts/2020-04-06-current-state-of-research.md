---
layout: post
title:  "Capturing the current state of research on network verification and synthesis"
authors: [ratul, beckett]
categories: [ research, overview]
tags: []
---

Capturing the current state of research on network verification and synthesis

Verification and synthesis are old problems in computer science. Verification seeks to answer the question: “can any input to a program result in that program producing an incorrect output.” Likewise, synthesis seeks to answer the question: “can I generate a program that produces the correct output for every input.” 

The motivation for verification and synthesis comes from the practical realization that humans often make mistakes and commonly introduce bugs into their systems. Having a way automatically (and quickly) find/fix all bugs in a system is highly appealing, particularly when such systems become sufficiently complex.

While traditionally researchers have developed verification and synthesis tools to verify computer hardware and software, recent trends in networking have exacerbated the need for such tools for networks as well. For example, the rise of cloud networking powered by the creation of enormous data center networks has dramatically raised the operational complexity of those networks. At the same time, as more services move online, it has become more vital that networks be bug-free and reliable. 

In response, researchers have started exploring the use of formal methods for networking. While they have made substantial progress on some problems, others remain unsolved. 

This article presents our view of the state of research in this domain.  Our views are shaped by our own research on network verification and synthesis and by a [course](https://courses.cs.washington.edu/courses/cse599n1/19au/index.shtml) we taught to understand and coalesce the ideas in this space. We thank our collaborators and students for all we have learned from them. 

We realize that perspective is inherently biased by our experiences and inherently incomplete. We also realize that we risk irking research colleagues who do not agree with our characterization (or whose work we failed to categorize). We invite those researchers to share their own perspectives.

###The first wave: data plane verification

Some of the earliest work on network verification was adopted for network “data planes”. The data plane refers to the part of the network responsible for forwarding packets from point A to point B. In general, network forwarding is performed by a collection of “switches” that each maintain a forwarding table that matches a packet entering the switch and determines which port(s) the packet should go out of.

The matching mechanism performed by the table depends on the particular type of network. Traditional routers match packets based on the longest matching prefix, yet newer technologies such as [OpenFlow](https://www.opennetworking.org/wp-content/uploads/2014/10/openflow-switch-v1.5.1.pdf) and [P4](https://p4.org/) allow for more general kinds of packet matching.

Some of the first data plane verification work started with [Anteater](http://conferences.sigcomm.org/sigcomm/2011/papers/sigcomm/p290.pdf),
and later made more scalable with the [Header Space Analysis](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final8.pdf) approach. The HSA tool works by efficiently propagating the set of all possible packets through the switch tables to find where every input packet will end up. Using this approach HSA can find bugs such as packet forwarding loops and unreachability.

Later approaches to data plane verification improved upon HSA in one dimension or another, such as making the analysis incremental with [Veriflow](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final100.pdf), making the analysis faster with [AP](https://www.cs.utexas.edu/users/lam/Vita/Cpapers/Yang_Lam_AP_Verifier_2013.pdf), leveraging the network [topology structure](https://dl.acm.org/doi/10.1145/3341302.3342094), and so on.

###The second wave: control plane verification

While data plane verification involves analyzing how packets are forwarded according to the tables present at every switch, in real networks these tables are themselves populated by other protocols or software. This “control plane” typically comes in two flavors: (1) distributed routing protocols, or (2) a centralized orchestrator.

A good example of early work for (2) is [VeriCon](http://www.cs.technion.ac.il/~shachari/dl/pldi2014.pdf). Since the centralized orchestrator is typically written in a general purpose programming language, traditional software verification techniques such as those based on [Hoare Logic](https://en.wikipedia.org/wiki/Hoare_logic) apply. However, automatically verifying arbitrary software is undecidable in general, and highly challenging in practice.

As a result, arguably researchers have had more success with (1). For example, [Batfish](https://www.usenix.org/system/files/conference/nsdi15/nsdi15-paper-fogel.pdf) simulates the protocols that populate the forwarding tables and then leverages data plane verification tools to check for various kinds of bugs. Unfortunately such simulation can only explore one control plane configuration, and any changes such as a link failing would require re-executing the tool. [Minesweeper](https://ratul.org/papers/sigcomm2017-minesweeper.pdf) and [BagPipe](https://homes.cs.washington.edu/~ztatlock/pubs/bagpipe-weitz-oopsla16.pdf) go further than Batfish and encode these distributed protocols using [SMT solvers](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories) to search for any “network environments” such as a combination of failures that will result in poor forwarding behavior as a consequence of the control plane protocols. However, these tools are significantly less scalable than Batfish.

As with data plane verification, much of the later work in this area is focused on improving these techniques along one dimension or another, such as improving [performance](https://arxiv.org/pdf/1906.02043.pdf), or leveraging network [symmetry](https://ratul.org/papers/sigcomm2018-bonsai.pdf).

###The third wave: programmable networks

Programmable network devices such as [Barefoot’s Tofino switch](https://barefootnetworks.com/products/brief-tofino/) have recently come to the market with the promise of replacing fixed-function network ASICs with programmable switches. The capability to program new functionality into the switch itself enables exciting new opportunities and holds promise to allow networks to evolve more quickly.

However, programmable switches also make verification more challenging since the data plane can now perform arbitrary logic. An important paper in this space is [p4v](https://www.cs.cornell.edu/~jnfoster/papers/p4v.pdf), which leverages [verification condition generation](https://en.wikipedia.org/wiki/Verification_condition_generator) to reason about such programs with assumptions about the control plane. Fortunately, the absence of loops or recursion in data plane programs (since they must forward at line-rate) allows tools like p4v to be fully automatic.

###Network synthesis

In contrast to verification, network synthesis has received relatively less attention. In the data plane, there is some early work such as [NetGen](http://madhu.cs.illinois.edu/sosr15-netgen.pdf). For the control plane, a notable example of synthesis is [Propane](https://ratul.org/papers/sigcomm2016-propane.pdf), which generates configurations for the BGP routing protocol from a high-level specification. Researchers have also recently explored the related problem of [repairing](https://aaron.gember-jacobson.com/docs/gember-jacobson2017cpr.pdf) the control plane.

###So what problems have been solved?

In hindsight, perhaps the most clearly “solved” problem in network verification is that of stateless data plane verification, which also happens to be the earliest work in this space. Stateless data plane verification tools today can already scale to handle large networks with millions of forwarding table rules and thousands of routers, all at human time scales. Further work in this area has also revealed additional optimizations that make such tools even more scalable. These tools have been successful enough to find their way into practical use at various companies such as [Microsoft](https://dl.acm.org/doi/10.1145/3341302.3342094)

###So what problems remain open?

Although verifying stateless data planes is largely a solved problem. The closely related problem of verifying stateful data planes -- for instance those networks with stateful firewalls that retain state across packets -- remains hard today. There is some [early work](https://www.usenix.org/system/files/nsdi20spring_yuan_prepub_0.pdf) in this space, but many problems remain.

Control plane verification, while possible today, remains challenging to scale to large networks. Particularly so when reasoning about all possible failures or all possible route advertisements from other networks. For the latter, Minesweeper and BagPipe remain the only tools capable of doing such reasoning.

In hindsight however, it is not clear if reasoning about all possible route advertisements from other networks is worth the extra cost since, in practice, these routes are often known or highly constrained. As such, tools like Batfish may represent a sweet-spot in terms of completeness and performance.

Programmable networks remain another large open area in the field. While it is currently possible to verify the forwarding functionality of a single programmable switch, it is not possible to verify the correctness of an entire network of programmable switches. Given that many use cases for programmable switches require coordination, this is an important next step. Even further away is the possibility of jointly verifying both the control and data planes for networks leveraging programmable devices. At the same time, doing such a joint analysis is promising in that it could reduce the specification annotation burden on users (as required by p4v) since control plane invariants could be learned directly instead of inferred.

Finally, research into network synthesis in general remains in its infancy. However, the idea has taken off in practice under the terminology of [Intent-Based Networking](https://www.cisco.com/c/en/us/solutions/intent-based-networking.html). While there is currently a lot of marketing hype around Intent-Based Networking, there has been little work in this area to understand its theoretical underpinnings to date.

###Summary

Network verification and synthesis are timely and exciting technologies that hold the promise of increasing the reliability of our critical network infrastructure. We summarized the current state of the art in these two areas and detailed our observations after teaching a course on the topic at the University of Washington in 2019.

