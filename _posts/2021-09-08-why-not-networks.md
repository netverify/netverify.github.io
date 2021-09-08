---
layout: post
title:  "Why don't we verify networks like other complex systems?"
authors: [bruce]
categories: [ network, verification, synthesis ]
image: assets/images/thesis.jpg
tags: []
---



In recent weeks, I have been working with my colleagues on an update 
to our [SDN book](https://sdn.systemsapproach.org/), including a fresh 
look at the future of SDN. We believe network verification will be an 
important part of that future, partly because  recent advances in SDN 
provide a stronger foundation on which to base verification 
techniques. This line of thinking gave me a few flashbacks to my days 
as a PhD student working on verification in a different context. This 
post is on why I think verification is an essential and feasible next 
step for networking.


---


In 1984, Artificial Intelligence was having a moment. There was enough
optimism around it to inspire me to explore the role of AI in chip
design for my undergraduate thesis, but there were also early signs
that the optimism was unjustified. The term “[AI
winter](https://en.wikipedia.org/wiki/AI_winter)” was coined the same
year and came to pass a few years later. But it was my interest in AI
that led me to Edinburgh University for my PhD, where my thesis
advisor (who worked in the Computer Science department and took a dim
view of the completely separate department of Artificial Intelligence)
encouraged me to focus on the chip design side of my research rather
than AI. That turned out to be good advice at least to the extent that
I missed the bursting of the AI bubble of the 1980s.


The outcome of all this was that I studied formal methods for hardware
verification at a point in time where hardware description languages
(HDLs) were just getting off the ground. These days, HDLs are a
central part of chip design and [formal verification of chip
correctness](https://dl.acm.org/doi/10.1145/378239.378473) has been
used for about 20 years. I’m pretty sure my PhD had no impact on the
industry–these changes were coming anyway. My graduate
experience–particularly the sense that I wasn’t having a practical
impact with my work–persuaded me I should do something different once
I was finished. Which is how I came to work on networks. And now
network verification is starting to have _its _moment, so I find
myself looking back at how important formal models, languages, and
verification were in computer science 35 years ago. Can the lessons
from other fields be applied to networking?


Compared to hardware, and to most software development for that
matter, networking is a laggard in terms of verification. Networks are
generally not verified in anything like the way chip designs or large
software systems are. Often the best that can be done is that a test
network is used to try out some configuration changes, and if
everything works as expected, the changes can then be pushed out into
production. But it’s hard to capture the complexity of production
environments in a test lab, and there isn’t an easy way to measure
coverage–have all the possible failure scenarios been tested?
Consequently, there is no shortage of reports about network
configuration changes that led to a production outage (e.g.,
[this](https://www.bleepingcomputer.com/news/security/major-bgp-leak-disrupts-thousands-of-networks-globally/)
and
[this](https://blog.cloudflare.com/how-verizon-and-a-bgp-optimizer-knocked-large-parts-of-the-internet-offline-today/)).


I link these difficulties for networking back to Scott Shenker’s
influential talk “[The Future of Networking, And The Past of
Protocols](https://www.youtube.com/watch?v=YHeyuD89n1Y)”, which I
consider one of the foundational talks of SDN. (In fact I’d call it
one of the best networking talks of all time, still holding up ten
years later after many re-watchings.) His argument is that networking
lacks the strong abstractions of many other fields in computer
science, and I believe that this is one of the main reasons that
verification has been a tough nut to crack for networking. Programming
languages, for example, have abstractions like virtual memory that
save us from worrying about how bits are stored in physical memory,
while hardware design has abstractions that keep chip architects from
worrying about transistor layouts. Network architects, on the other
hand, have to figure out how every router should be configured to
implement their policies. They have to do both the high-level design
and map it down to low-level configuration of devices.


That said, I am cautiously optimistic that there is a path out of our
current situation. I see a few threads of networking research and
development coming together. First, the SDN future that was sketched
out by Shenker and others in 2011 is starting to become reality. While
we are far from a world where most networks are software-defined,
there is widespread adoption of SDN across cloud providers and
enterprise networks, especially when we include software-defined WAN
and network virtualization systems. Among the many benefits of SDN is
the fact that you can specify the desired state of the network through
a central API, and the control plane is responsible for bringing about
the changes to realize that desired state. Thus it becomes much more
feasible to verify that a network meets the intent of its operator.


A second thread is the rise of languages to specify network
behavior. Just as HDLs were essential to the specification and
verification of hardware, domain-specific languages for networking
such as P4 raise the possibility of verifying the correctness of
networks. This is a subject that we’ve touched on in the [latest
updates](https://sdn.systemsapproach.org/future.html) to our SDN book,
and is becoming enough of a mainstream topic to warrant its own
[tutorial at
SIGCOMM](https://conferences.sigcomm.org/sigcomm/2021/petr4-tutorial.html)
this year.


Finally, there is the increasing maturity of network verification and
validation tools, both commercial and open source. One of the
realities of networking is that brownfields are the common case, so
you have to deal with the diversity of device types and protocols that
exist in production networks, much as we might wish everything could
be cleanly designed again from the ground up. So tools that assist in
verifying that networks will behave as expected have to address this
complexity, with the ability to model the behavior of real-world
networks. Tools such as [Batfish](https://batfish.org)
and those of [Forward Networks](https://forwardnetworks.com/) are
getting increasingly good at this.  And when we do get the opportunity
to build greenfield networks, they benefit from the stronger
foundation made possible by verification tools.


Arguably the biggest hurdle to proper verification of networks is the
culture of the networking industry. Because networking has required us
to “master complexity” (in Scott Shenker’s words) we have not always
been quick to adopt techniques from other computer science disciplines
that would give us access to better abstractions. I saw this play out
with the early struggles to get the industry to accept SDN, and I
believe we need to do the same in pushing for more adoption of
verification approaches that are mainstream in other computing
fields.



