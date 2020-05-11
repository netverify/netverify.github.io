---
layout: post
title:  "You can't verify what you can't specify"
authors: [lvanbever]
categories: [overview, research, network, verification]
image: assets/images/smt.png
tags: []
---


Back when I was studying for my master degree, [Prof. Axel van Lamsweerde](https://www.info.ucl.ac.be/~avl/) was
teaching us formal logic. Axel is world-famous for his works on requirements
engineering, that is, the process of defining and maintaining the prerequisites
that must be met by software systems.

During that time, I remember reading one of Axel's seminal paper entitled
["Formal Specification: a Roadmap"](https://dl.acm.org/doi/10.1145/336512.336546) which describes the
strengths and weaknesses of formal specification technologies. I re-discovered Axel's paper as we started to work on making network verification technologies more usable (more on this later) and found its lessons to be invaluable.

In this post, I'd like to quickly summarize Axel's views on why writing good
specifications is challenging; why I believe these challenges apply almost
verbatim to network verification; and what we can do to address them. I'll also mention some of our [recent work](https://nsg.ee.ethz.ch/fileadmin/user_upload/publications/config2spec-final.pdf) on automatically mining network specifications.


## The problem with formal specifications

As Axel puts it, a formal specification is "the expression, in some formal
language and at some level of abstraction, of a collection of properties some
system should satisfy". Of course, not all formal specifications are useful.
Axel goes therefore a bit further in defining "good" specifications as those
satisfying a set of key properties: (i) they adequately capture the problem at
hand; (ii) they are consistent and unambiguous; (iii) they are complete; and
yet, at the same time, (iv) they are minimal.

Writing good specifications is difficult, perhaps as difficult as writing a correct program in the first place. Axel lists many reasons behind this complexity. Let me describe the ones that resonated with me the most.

To start with, one needs to figure out what the properties to specify actually
are. Specifications are indeed never formal in the first place: one must figure
them out. Doing so requires people with different background, mental models, and
languages to come together (e.g., customers, domain experts, architects, programmers). Finding a common ground is hard and often leads to inconsistencies.

Once the properties of interest are known, they need to be expressed in some
kind of formal language. Doing so is again hard as most users lack the relevant
expertise in formal languages (e.g., mathematical logic). Specification
languages also provide little guidance to the user on how to elaborate a good
specification (e.g., constructive methods). Besides, they tend to focus on capturing functional properties (what the system is expected
to do), leaving out important non-functional properties.

Despite being two decades old, I believe the problems Axel pointed at are still largely relevant. This recent [tweet](https://twitter.com/heidykhlaaf/status/1206289442484473856) from Heidy Khlaaf ([@HeidyKhlaaf](https://twitter.com/heidykhlaaf/)) illustrates this particularly well:

<div class="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In the past three years of working on large safety critical systems, I&#39;ve learned that verification isn&#39;t the real problem, but it&#39;s writing specifications. Don&#39;t @ me. <a href="https://t.co/k4zi1j5QkX">https://t.co/k4zi1j5QkX</a></p>&mdash; Dr Heidy Khlaaf (هايدي خلاف) (@HeidyKhlaaf) <a href="https://twitter.com/HeidyKhlaaf/status/1206289442484473856?ref_src=twsrc%5Etfw">December 15, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>


## Back to the network

At this point, you're probably thinking _"what does all this have to do with the network anyway?"_ Well, when it comes to network verification, pretty much
everything. Both network verification and synthesis require a formal
specification of the _intended_ network behavior, and writing this
specification manually is difficult, if not downright impossible in some cases.

As an exercise, put yourself in the shoes of a network operator working for,
say, a large Internet Service Provider (ISP). Tired of human-induced downtimes,
your boss asks you, from now on, to verify the network configurations before
pushing changes into production. Before using a verification tool though, you
need to specify what it is that you want the network to do...

A few generic requirements come to mind: surely you want the network to "ensure
reachability", even if there are failures. Who wants forwarding loops or blackholes anyway? Those obvious requirements out of the way, you start to realize
that your network does _way_ more than just ensuring reachability... Among
others, it load-balances traffic, routes it so as to minimize transit
costs, isolates important customer traffic on disjoint links, and reroute parts
of it via predefined waypoints. Thinking even further, non-functional
requirements start to come to mind such as the need to converge rapidly upon
failures or to maintain the number of routes below a reasonable threshold.
Figuring out the entire specification quickly becomes daunting, especially as
most of it has been homegrown over years by an entire team of network engineers
(some of whom most likely do not even work there anymore).

Writing the specification is daunting too. Since most (existing) network
specification languages focus on capturing low-level forwarding and routing
properties, the specification ends up being gigantic. As an illustration, the
formal specification for Internet2 (the US research network, a network composed
of around 10 routers) contains up to _thousands_ of predicates. Imagine writing
this by hand, without making any mistake. Besides, important non-functional
properties (like fast convergence) cannot be specified and, therefore, cannot
be verified.


## Looking forward

So what do we do? Again, Axel's
[paper](https://dl.acm.org/doi/10.1145/336512.336546) describes many
interesting avenues for future research... in networking. Like Axel, one of the
main avenue I see pertains to designing higher-level specification languages, a
little bit like [Frenetic](http://www.frenetic-lang.org/) did for OpenFlow. Ideally, these
languages should be multiparadigm, to cope with the diversity of the users, and modular, to build large-scale specification from smaller
components. Finally, these languages should also support non-functional properties such as convergence time, memory requirements, or maintainability. 

Another promising avenue, and one which we started to explore in a recent
[paper](https://nsg.ee.ethz.ch/fileadmin/user_upload/publications/config2spec-fi
nal.pdf), is to partially automate the task of writing the specifications. In
[Config2Spec](https://nsg.ee.ethz.ch/fileadmin/user_upload/publications/config2spec-final.pdf), we propose to do this by extracting the
specification from a network configuration. Once the full specification is
extracted, the operator can then focus her attention on formalizing the deltas with respect to an existing network behavior.

Further techniques are required besides languages and mining techniques though.
In particular, like Axel, I believe network analysis tools should enable
reasoning _in spite of errors_, based on partial and possibly bogus
specifications (e.g., conflicting ones). Here, I could imagine systems in which
the specification is incrementally refined and debunked with the help of a
human. All this is probably for another post though... Stay tuned!


#### Our group is recruiting!

If you are interested in network verification and synthesis, our
[group](https://nsg.ee.ethz.ch/) has open PhD and post-doc positions in the area! Please [reach out](https://nsg.ee.ethz.ch/people/laurent-vanbever/)!