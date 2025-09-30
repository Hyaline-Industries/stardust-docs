---
layout: post
---

## The Journey Begins
---

I've spent a lot of my career working on databases. This isn't really a huge surprise: a lot of people have spent a lot of time working on databases, both academically and industrially. They are one of the major cornerstones of the web (Other cornerstones: TCP/IP and DNS, and HTML/Javascript). And even outside of the internet, databases are really useful tools, so there is quite an extensive group of people who know things about it. I'm only different in that I'm a member of that subset of database people who have worked on _distributed_ databases. And I've done a lot of that: I designed most of the core of SpliceMachine (which ended up failing in 2021), which was transactional SQL over HBase. I did a lot of work on the replicated storage system for Stardog Union, and I helped expand and build out a lot of the underlying database behind CloudKit (FoundationDB, specifically RecordLayer) at Apple. 

So I've got a bit of history dealing with multi-node database systems. But one thing that always bothered me about these systems is how _wasteful_ they are. We were always obsessed with performance: can we lower latency by X%, or increase throughput by Y%. But we never designed around costs. Not once in my career did my boss take me aside and say "Scott, your design uses too much electricity, you gotta make it better". Based on my reading of database literature, I would say that this is probably true for everyone.

But databases are huge energy sinks, especially distributed ones:

* You need higher end hardware to run them well (or, alternatively, more servers. Which can be even more expensive)
* The need for reliability means more servers: backups and hot spares
* The database is often a concurrency bottleneck for the entire system, so you need to scale up (or out) to avoid causing problems elsewhere on your system

And when you get to a multi-tenant environment like "the cloud"(Dum dum dum) these problems are multiplied by all the _other_ people doing the exact same thing you are. I have no idea how many MySQL instances are running in AWS right now, but I'd be willing to bet that it's in the billions. If MySQL were just 1% more energy-efficient, there are probably entire gas power plants that could be forced to shut down.

So I started to think: Can you design a distributed database that is reliable, but is _also_ aware of its resource footprint? Can we make a database that will operate _on a budget_? Stardust is my attempt to answer this question. It's a journey, to be sure, and one I'm not sure can be completed: it may not be possible at all, or it's possible that it can be done, but the improvements end up being meaningless. I don't know. But I want to find out.


