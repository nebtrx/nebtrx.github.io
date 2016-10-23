---
title: Quick start with Redis, Windows & .Net
categories:
- getting_started_with
tags:
- .Net
- open source project
- Redis
- Redis Server
- Windows
---

[Redis](http://redis.io/) is data structure server (a sort of advanced key-value store) largely recognized for being the most efficient NoSQL solution to manage rich data structures. For those who haven’t tried it and are familiar with [Memcached](http://memcached.org/), Redis is like a Memcached decorator with extra commands, extra data types(strings, lists, sets, sorted sets or hashes), extra speed (even more than 100 000 rps in some scenarios), nonvolatile persistence and, last but not less important, a huge ease of use. Redis flexibility and speed makes it a powerful tool that can be used extensively for building and scaling IT solutions as NoSQL storage, distributed cache system, and even as message queue solution. For a deep dive into Redis you should consider consulting its [documentation](http://code.google.com/p/redis/wiki/README)

**Versions**  
Redis first development began in 2009 as a key value store by Salvatore Sanfilippo in order to improve the performance of an analytics product implemented by him. It grew in popularity after getting support from people and companies in the development community, as for example, VMware which is its actual sponsor. Redis is written in ANSI C and works in most POSIX systems like Linux, *BSD, OS X without external dependencies. Unfortunately there is no native support for Redis in Windows, so we depend on third party versions. So, the candidates are a couple of GitHub hosted open source project:  
• [MSOpenTech](https://github.com/MSOpenTech/redis)  
• [Windows 32 and x64 port of Redis server](https://github.com/dmajkic/redis)

MSOpenTech recently become the Official Version Redis for Windows. It’s a good looking prototype still without a stable release but with a promising future. MSOpenTech took a lot from the other mentioned candidate, which is a quiet and older stable open source project run by Dušan Majkic (https://github.com/dmajkic, https://github.com/dmajkic/redis/) which has been for some time the most reliable port of Redis for Windows(I’m 90% sure also the unique until MSOpenTech). Also there is this [fork](https://github.com/rgl/redis) of Majkic’s project which lets you run Redis Server as a Windows Service and comes with Windows Installer. It was written by some guy which name I can infer from its [website](http://ruilopes.com/) but I prefer not to take the risk of put it here. So, as you may guess by now this is my favorite version (I didn’t include it as candidate for a being a fork of one of them). Plus!, the aka redis-server-for-lazy-ones-edition makes a perfect fit. The Windows Installer for 32/64 bits is available for download [here](http://ruilopes.com/redis-setup/).

**Server Installation**  
As easy as double click the windows installer and get a server instance running on localhost under port 6379 as a service. The installer also makes room for a redis-cli client.

**.Net Developing + Redis**  
In order to begin developing with Redis under .Net framework there are two main choices in the spotlight when the discussion goes about C# Redis Client Libraries.

• Booksleve: It offers a fully pipelined, asynchronous, multiplexed and thread-safe access to Redis in busiest environments. For further info about it click [here](http://marcgravell.blogspot.com/2011/04/async-redis-await-booksleeve.html). BookSleeve is available via Nuget and you may install it from the Package Manager Console by typing:

```
PM> Install-Package BookSleve
```

• ServiceStack.Redis: Is an Open Source C# Redis client library based on [Miguel de Icaza](http://twitter.com/migueldeicaza) previous efforts with [redis-sharp](http://github.com/migueldeicaza/redis-sharp). ServiceStack.Redis propose different client connection managers so you can choose the proper one for multi-threaded applications scenarios as well as a variety of clients optimized for maximum efficiency and armed with layered functionality for a maximum developer productivity. The library is an independent project which lies inside the ServiceStack Web Services Framework, used in the stackoverflow’s web architecture, but can be used with or without it. Its huge and flexible API makes it the ideal option when dealing with large scale projects using Redis. ServiceStack.Redis is available at Nuget repositories and you may install it from the Package Manager Console by typing:

```
PM> Install-Package ServiceStack.Redis
```

**Summary**  
Redis is an incredible powerful and easy to learn technology which trades complex generic storage solutions for simplicity and stunning performance suitable for more specific solutions such as cache and NoSQL storage, and message queue as well. Unfortunately Windows doesn’t have the latest word on Redis because it isn’t officially supported by its developing team. This post tries to minimize the underlying distance of that fact for those who want make the jump. So, good luck and good jump, you won’t get killed trying it, that you can have it for sure.

See you folks.
