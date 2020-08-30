---
layout: post
title:  "The Verification-Synthesis Spectrum"
authors: [gemberjacobson,akella]
categories: [overview, research, network, verification, synthesis]
image: assets/images/verification-synthesis_spectrum.png
tags: []
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/5_bEoMj2Nps" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

(_Sung to the tune of ["The Big Bang Theory" theme
 song](https://www.youtube.com/watch?v=20i5eqwK178) by the Barenaked Ladies_)
Our whole network was in a simple, dense state  
Then nearly eighteen thousand days ago expansion started, wait  
The networks were complex, the engineers were quite perplexed,  
Then researchers developed tools  
That can verify (That can synthesize)  
Verification, synthesis, which of these are best for this?  
That's the focus of this article! Hey!

In recent years, researchers have developed a plethora of tools for verifying and synthesizing router configurations. Configuration verifiers analyze a snapshot of a network's current or proposed configurations to determine whether the network satisfies all forwarding policies (e.g., reachability, isolation, path preferences, etc.) under various conditions (e.g., link/node failures). Configuration verifiers output a list of (concrete) policy violations to alert network engineers to potential errors in the configurations. In contrast, configuration synthesis tools automatically produce brand-new configurations that satisfy a network's forwarding policies. The synthesized configurations are guaranteed to be error-free---or "correct-by-construction" in formal methods terms.

From a formal methods perspective, verification and synthesis are closely intertwined. First, they are both *search* problems: verifiers search for a proof of configuration correctness, and synthesizers search for a correct configuration. Second, they both require a *model* that encodes the semantics of configurations and routing algorithms. Finally, they both require a *specification* that encodes a network’s expected forwarding behavior, or *intents*.

But from a practical perspective, verification and synthesis are quite different. Verifiers operate on *existing configurations*, whereas synthesizers *create configurations*. Verifiers *find errors*, whereas synthesizers *avoid errors*. These differences make verification the best choice in some situations and synthesis the best choice in other situations; we discuss this in more detail below.

However, these extremes aren't the only option. Some tools, including our own research, strike a middle-ground between verification and synthesis. These configuration repair tools are designed to uncover violations of network policies and automatically produce configuration updates that eliminate the errors.

## The Verification-Synthesis Spectrum

As discussed above, configuration verification and configuration synthesis are closely intertwined but satisfy different goals. Moreover, different verifiers provide different amounts of information, and different synthesizers target different types of configurations. This prompts us to view configuration verification and synthesis as a spectrum.

<img src="/assets/images/verification-synthesis_spectrum.png" alt="Verification-Synthesis Spectrum" width="873" height="132"/>

On the far left are configuration verifiers that take a networks’ current configurations and expected forwarding behaviors as input, and produce a simple yes/no---i.e., satisfied/violated---answer for each forwarding requirement: e.g., [Tiramisu](https://www.usenix.org/system/files/nsdi20-paper-abhashkumar.pdf) and [ERA](https://www.usenix.org/system/files/conference/osdi16/osdi16-fayaz.pdf). On the far right are configuration synthesizers that require only a specification, and produce configurations entirely from scratch: e.g., [Propane](https://ratul.org/papers/sigcomm2016-propane.pdf) and [SyNET](https://vanbever.eu/pdfs/vanbever_synet_cav_2017.pdf).

However, some verifiers provide more than a simple yes/no answer for each forwarding requirement. For example, [Minesweeper](https://ratul.org/papers/sigcomm2017-minesweeper.pdf) and [ARC](http://aaron.gember-jacobson.com/docs/gember-jacobson2016arc.pdf) both produce a concrete counterexample---e.g., a failure scenario and resulting forwarding path---for each violated requirement. Going a step further, [Batfish](https://www.usenix.org/system/files/conference/nsdi15/nsdi15-paper-fogel.pdf) and [Plankton](https://www.usenix.org/system/files/nsdi20-paper-prabhu.pdf) produce complete forwarding information bases (FIBs), and Batfish tracks the provenance of each FIB entry, such that a network engineer can trace the sequence of events and set of configuration statements that led to the FIB entry’s existence. 

Similarly, not all synthesizers produce configurations entirely from scratch. 
For example, [Propane/AT](https://ratul.org/papers/pldi2017-propaneat.pdf)
synthesizes abstract templates based on device roles (e.g., _core_ and
_border_ routers) and generates/updates concrete configurations as the
topology grows. Similarly, [Zeppelin](http://pages.cs.wisc.edu/~sskausik08/papers/sigmetrics18zeppelin.pdf) allows a network engineer to specify policies regarding configuration structure---e.g., which routers should run BGP? how many OSPF areas should there be?---which inform the structure of the synthesized configurations. Going a step further, [NetComplete](https://vanbever.eu/pdfs/vanbever_netcomplete_nsdi_2018.pdf) requires network engineers to provide a configuration template with holes for key parameters---e.g., link costs, route filter match criteria, etc.---and synthesizes values for these holes, rather than the complete configuration.

(We’ll discuss the middle of the spectrum, which includes [CPR](https://aaron.gember-jacobson.com/docs/gember-jacobson2017cpr.pdf) and [Jinjing](https://dlnext.acm.org/doi/abs/10.1145/3341302.3342088), later in this article.)

It is important to note that this spectrum only captures some of the similarities and differences between various tools. Verifiers and synthesizers also differ in terms of the protocols/features they support, the environment variables they consider (e.g., link/node failures, external route advertisements), how well they perform/scale, etc. [Figure 1 in the Minesweeper paper](https://ratul.org/papers/sigcomm2017-minesweeper.pdf) provides a more nuanced overview of the network verifier landscape. 

Additionally, the spectrum does not include data plane verifiers---e.g., [HSA](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final8.pdf), [VeriFlow](https://www.usenix.org/system/files/nsdip13-paper16.pdf), and [Delta-net](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-horn-alex.pdf)---data plane synthesizers---e.g., [Genesis](http://pages.cs.wisc.edu/~sskausik08/papers/popl17genesis.pdf)---or configuration anomaly detectors---e.g., [rcc](https://www.usenix.org/legacy/event/nsdi05/tech/feamster/feamster.pdf), [Minerals](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.206.1938&rep=rep1&type=pdf), and [SelfStarter](https://www.usenix.org/system/files/nsdi20-paper-kakarla.pdf), because these tools are fundamentally different from the configuration verifiers and synthesizers included in the verification-synthesis spectrum. In particular, data plane verifiers and synthesizers ignore a network’s configurations, and configuration anomaly detectors ignore a network’s forwarding requirements.

## Choosing a point in the spectrum

The spectrum of configuration verifiers and synthesizers that exist today raises a fundamental question: _which point(s) in the spectrum should a network practitioner choose?_

One way to approach this is to ask a slightly different question: _are you working with a greenfield or a brownfield network?_ A greenfield (i.e., brand new) network is a perfect opportunity to employ configuration synthesis, because a greenfield network does not have any existing configurations. There may be configuration templates that were used in other (similar) networks, which could motivate choosing a partial configuration synthesizer (e.g., [NetComplete](https://vanbever.eu/pdfs/vanbever_netcomplete_nsdi_2018.pdf)) as opposed to a complete configuration synthesizer (e.g., [Propane](https://ratul.org/papers/sigcomm2016-propane.pdf) and [SyNET](https://vanbever.eu/pdfs/vanbever_synet_cav_2017.pdf)). In contrast, network verifiers are often better suited for a brownfield (i.e., partially existing) network, because configurations already exist, changes are often small, and network engineers already have established network management practices. (These observations are supported by studies of configuration changes in [large university campus networks](https://conferences.sigcomm.org/imc/2011/docs/p499.pdf) and [large data center networks](https://conferences2.sigcomm.org/imc/2015/papers/p395.pdf).)

A different way is to ask: _how much automated change are you willing to
tolerate?_ Clean-slate configuration synthesizers (e.g.,
[Propane](https://ratul.org/papers/sigcomm2016-propane.pdf) and
[SyNET](https://vanbever.eu/pdfs/vanbever_synet_cav_2017.pdf)) completely
ignore a network’s current configurations and synthesize brand-new
configurations. This requires wholesale replacement of a network’s
configurations each time forwarding policies change, which is a risky and
potentially disruptive operation---e.g., network engineers from a regional
research and education network reported that flash reliability problems 
cause some routers in their network to crash when the configuration is
updated. Moreover, any hand-tuning of configurations may be wiped out. Partial
configuration synthesizers (e.g.,
[NetComplete](https://vanbever.eu/pdfs/vanbever_netcomplete_nsdi_2018.pdf) and
[PropaneAT](https://ratul.org/papers/pldi2017-propaneat.pdf))
allow some aspects of a network’s configurations to remain fixed. However,
it’s up to network engineers to determine how much of a configuration to
manually specify (in a template) and how much to synthesize. More manual
specification means more work for network engineers and a relaxation of the
"correct-by-construction" guarantees provided by configuration synthesizers.
In contrast, configuration verifiers are "read-only" insofar as they flag
configuration errors but do not generate any part of the configurations. This
allows network engineers to have complete control over configuration changes,
which enables engineers to hand-tune changes according to existing network
management practices and preserve their networks’ configuration "style."

## A middle ground: configuration repair

It is hopefully clear from our discussion thus far that configuration verifiers and synthesizers each have their merits. However, an important question remains: _is there a middle ground?_ Is it possible to obtain the benefits of synthesis for a brownfield network? Is it possible to automate _some_ changes?

Fortunately, the answer is yes. At the middle of the verification-synthesis spectrum are tools designed for _configuration repair_: e.g., [CPR](https://aaron.gember-jacobson.com/docs/gember-jacobson2017cpr.pdf) and [Jinjing](https://dlnext.acm.org/doi/abs/10.1145/3341302.3342088). These tools can automatically find and correct errors in hand-written changes and/or automatically update configurations to satisfy new forwarding requirements. 

Unlike configuration synthesizers, configuration repair tools do not generate new configurations from scratch. Rather, configuration repair tools identify a minimal set of changes that must be made to the current/proposed configurations to ensure the network satisfies all forwarding policies. Moreover, unlike configuration verifiers, configuration repair tools do not leave the task of correcting errors as an exercise for network engineers. Rather, configuration repair tools automatically address forwarding policy violations.

Existing configuration repair tools have demonstrated the merits of this middle ground, but a lot of work remains to be done. In particular, CPR prefers changes that modify the fewest lines of configuration, but the smallest change is not always the best change. For example, enabling route redistribution is a single-line change, but route redistribution is [notoriously difficult to reason about](http://www.cs.cmu.edu/~4D/papers/rr-icnp07.pdf). Instead, a multi-line change that adds new neighbor relationships and applies appropriate filters may be easier for network engineers to reason about and better align with a network’s current configuration "style." Additionally, CPR and Jinjing are limited in the types of configurations they can handle: CPR only handles static routes, OSPF, and a limited subset of BGP, while Jinjing only handles ACLs. In contrast, existing configuration verifiers and synthesizers can handle a wide range of protocols and features.

In our own ongoing work, we are working to address these shortcomings (and hope to write a future article about this). We hope others will do the same.
