---
layout: post
title: Is CCN more secure than IP?
---

Security is an annoyingly broad term, especially when used to describe a networking architecture. Many presentations often tout CCN and related information-centric networking designs as more secure than IP. Such bold claims are meaningless for several reasons. First, the notion of security is completely undefined. Is CCN more confidential than IP? Does it provide better privacy and anonymity? Can it withstand common networking attacks such as DDoS? The answer is unclear. Second, there is generally no adversarial model in which to assess these supposedly superior security properties. How can we say that "X is more secure than Y" without having defined an appropriate way to measure the security of X and Y. In this post I will attempt to answer these questions. 

To begin, we need a definition of security. For classical reasons, I will stick with the traditional CIA meaning of confidentiality, integrity, and availability. Moreover, I will only focus on the network layer itself. Most, if not all, security properties of CCN applications rely on festures provided by the network.