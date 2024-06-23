---
params:
  math: true
title: "SIEVE is Simpler than LRU: an Efficient Turn-Key Eviction Algorithm for Web Caches"
description: a simpler-than-LRU cache that performs better than it on web cache workloads
summary: a simpler-than-LRU cache that performs better than it on web cache workloads
tags: ["Cache"]
date: 2024-06-02
author: Bryan Li
---

Zhang, Yang, Yue, Vigfusson, Rashmi 2024 ([pdf](https://junchengyang.com/publication/nsdi24-SIEVE.pdf))

## Overview

Web caches are a critical part of today's Internet infrastructure, reducing data access latency and bandwidth costs. One of the key parts of this system is the eviction algorithm, which helps manage the limited cache space.

Previously, many algorithms aimed to maximize efficiency - a lower miss ratio. In the process, these algorithms often sacrificed simplicity, which typically correlates with scalability, efficiency, and maintainability.

This paper introduces and analyzes the SIEVE algorithm, a simple but high-efficiency variation on the FIFO-reinsertion algorithm optimized for web cache workloads.

## Background

**Metrics:** There are two primary metrics used to measure cache performance: efficiency, throughput performance, and scalability. Generally, as complexity increases, throughput performance and scalability decreases as longer critical sections and more metadata may be necessary.   <!--Cache efficiency metrics include object miss ratio (fraction of requests that are cache misses) and byte miss ratio (fraction of bytes that are cache misses). Throughput performance -->

**Access Patterns:** Web cache workloads typically follow a power-law (Zipfian) distribution, where a small portion of objects make up the majority of the requests. An important distinction from other cache use cases is that scan and loop/sequential access patterns are rare, and new objects are extremely common, being created every second.

To expand on the power-law distribution, it follows a pattern where the $i^{th}$ most popular object has a relative frequency of $\frac 1{i^\alpha}$. $\alpha$ here is a parameter determining skewness and has been found to range from 0.55 to 1.5 depending on type of workload, cache level, etc.

**Efficient Cache Eviction Algorithms:** Many of the recent algorithms have greatly increased in complexity, using multiple LRU queues, machine learning, and so on. However, though they might perform better in certain workloads, the improvement is often not significant. In this way the increase in complexity outweighs the benefits.

What makes a cache eviction algorithm more efficient can be boiled down into two parts: **lazy promotion** and **quick demotion**.

- Lazy promotion promotes cached objects only when they are about to be evicted, retaining popular objects with minimal effort.
  - For comparison, FIFO has no promotion, and LRU promotes on cache hit (eager promotion).
- Quick demotion removes most objects shortly after they are cached, which takes advantage of the characteristics of the power-law distribution - most objects are unpopular.

## Algorithm

### Overview

SIEVE uses a single FIFO queue and "hand" pointer.

The queue maintains insertion order, and new objects are added to the head of the queue.

Every object in the queue uses 1 bit to track visited/non-visited status, with the hand pointing to the next eviction candidate. The hand pointer moves from the tail to the head.

- **Cache hits** set the visited bit of the cache line to 1.
  - This also happens to be lock-free, which improves throughput.
- **Cache misses** look at the hand pointer and set the visited bit to 0 if it is 1, moving the hand pointer.
  - Otherwise (`visited=0`) it will remove the eviction candidate from the cache, and move the hand pointer.

This fulfills the idea of 'lazy promotion' by only promoting at eviction time, by keeping the object at the original location. Because "survived" objects are generally more popular than evicted ones, they will likely be accessed in the future as well. (similar to CLOCK algorithm).

Then, the hand can quickly move towards the head where new objects are inserted, examining new objects shortly after admission. This achieves 'quick demotion'.

### Key Differences

Unlike other similar algorithms (CLOCK, LRU, FIFO/FIFO-Reinsertion), the eviction candidate is not necessarily the tail (or head) of the queue. However, inserted objects are always at the head of the queue. This means that new objects and older objects are not mixed together!

- CLOCK inserts *new* objects where the previous object was evicted, rather than always at the head.
- FIFO-reinsertion and LRU insert *retained* objects at the head, rather than keeping them in place.

The moving hand is unique in that it essentially acts as an "adaptive filter" that removes unpopular objects from the cache quickly.

### Performance and Methodology Summary

SIEVE was implemented via [libCacheSim](https://github.com/1a1a11a/libCacheSim), with the full code on [GitHub](https://github.com/cacheMon/NSDI24-SIEVE/tree/main). Datasets from public sources such as Twitter, Meta, Wikimedia, and Tencent were used to test the cache performance.

Test metrics used miss ratio reduction relative to FIFO ($\frac{m_{FIFO} - m_{algo}}{m_{FIFO}}$ where $m$ represents miss ratio) to measure efficiency (1 is best, -1 is worst), and millions of operations per second (Mops) to measure throughput.

Overall, miss ratio reduction was generally roughly on par with other "state of the art" eviction algorithms. However, SIEVE has a particularly high throughput (e.g., with [2x faster cache hits](https://github.com/mfleming/sieve) than LRU) and scales very well with the number of cores.

## Why not SIEVE?

SIEVE seems to be promising in terms of web cache workloads, with high and scalable throughput and efficiency seemingly on-par with other eviction algorithms. In that case - [why haven't we started using SIEVE](https://brooker.co.za/blog/2023/12/15/sieve.html)? (That is, other than the fact that it's relatively new...)

Perhaps part of the issue here is that it isn't scan-resistant. Access patterns for block and page accesses, which prior cache research focused on, has a combination of random access patterns as well as large sequential access (such as a backup). These sequential accesses might then push out the "popular" items that are accessed.

Currently, SIEVE is not known to be in use by any large web caches. Future research and development might address its limitations in scan resistance. Either way, SIEVE stands out as a simpler eviction algorithm for web cache workloads and shows the potential for simple, scalable, and efficient caches in the near future.


<!-- A [blog post from 2023](https://brooker.co.za/blog/2023/12/15/sieve.html) proposes SIEVE-k as a solution, using a counter rather than a visited bit. However, it ends up being worse for web cache workloads, with varying results on block workloads. -->

