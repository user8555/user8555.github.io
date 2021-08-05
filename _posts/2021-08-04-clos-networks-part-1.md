---
layout: post
title: Clos networks - part 1
---

Recently I was reading the book Cloud Native Data Center Networking. I became interested in this topic because even though I’ve worked on Cloud scale products for several years, the internals of data center design and network have remained opaque and inaccessible to me all this time. I really enjoyed reading the first-half of this book because I felt it did a good job of explaining how old network architectures from the client-server era used to work and then did a good job of explaining what changed with the advent of large scale online services to lead to creation of the modern data center network architectures that we see today.

<!-- more -->

One of the key topics that you will quickly be exposed to as you delve into the world of data center networking is Clos Networks. A Clos network, while new to modern data center designs, is not a new idea at all. In fact, it was invented by Charles Clos in the 1950s while he was working at Bell laboratories in an attempt to provide non-blocking switching networks for telecommunications.

## Resurgence of Clos Networks for data center networking

Before the boom of online internet services (Google, Youtube, Netflix etc.), around the 1980s and 1990s, client-server architectures were popular. In those architectures, the client and server were located at a distance. The amount of data transferred between client and server in those days was small enough that it didn’t make much economic sense to have dedicated paths between the client and server machines. Instead, it made sense to aggregate the network traffic at different levels leading to a “fat tree” model.

However, with the rise of online internet services, a couple of things changed. Large scale applications like Google search, YouTube caused dramatic increases in the amount of data transferred between computers. This change was much more dramatic within a data center where now servers needed to exchange large amounts of data for applications like MapReduce. These applications were run on commodity servers and could be located anywhere in the data center. The rapid growth of internet also meant that the new architecture needed to scale easily as more servers were added to the data center.

The fat-tree network was ill-suited to address these challenges for several reasons:

1. It requires use of specialized equipment at the aggregation layer to support high data transfer rates. These are expensive devices and hence we cannot have many of them spread out through the data center to allow the commodity servers to be located anywhere in the data center. The core-agg-access design pattern used Spanning tree protocol which doesn’t work well when you want many redundancies for equal cost paths between servers to provide an even distribution of traffic and avoid head-of-line blocking. 
2. What we needed was a scalable network architecture that provided non-blocking path between any two servers on the network. It turns out, during the 1950s, Charles Clos designed a multi-stage switching network that provided non-blocking network switching for the telecommunications industry. This network, called a CLOS network, is exactly what we needed to solve our data center problems.

Here are the salient features of the CLOS network that make it well-suited for our problems:

1. It aligns with the commodity computing philosophy of modern data center design because it enables the use of the same kind of network switches at various layers – ToR and Spine.
2. It is based on routing instead of switching, so scales well to large number of servers.
3. It enables multiple valid equal cost routes to communicate between servers and uses ECMP routing instead of STP switching to provide efficient and reliable routing of data between the servers
4. The commoditization of network switches means lower TCO
5. More capacity can be easily added by adding more network switches on the spine and connecting the new spine to the existing ToRs. The ECMP protocol would then take over and automatically start sending traffic through this new switch instead of requiring any manual network configuration like the core-agg-access model
6. It is possible to have an N-layer architecture as well for very large networks


In the next post we will talk about the design considerations when building a Clos network

