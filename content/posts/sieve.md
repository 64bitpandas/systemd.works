---
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

**Access Patterns:** Web cache workloads typically follow a power-law (Zipfian) distribution, where a small portion of objects make up the majority of the requests. An important distinction from other caches is that scan and loop/sequential access patterns are rare, and new objects are extremely common, being created every second.

To expand on the power-law distribution, it follows a pattern where the $i^{th}$ most popular object has a relative frequency of $1/i^\alpha$. $\alpha$ here is a parameter determining skewness and has been found to range from 0.55 to 1.5 depending on type of workload, cache level, etc.

**Efficient Cache Eviction Algorithms:** Many of the recent algorithms have greatly increased in complexity, using multiple LRU queues, machine learning, and so on. However, though they might perform better in certain workloads, the improvement is often not significant. In this way the increase in complexity outweighs the benefits.

What makes a cache eviction algorithm more efficient can be boiled down into two parts: **lazy promotion** and **quick demotion**.

- Lazy promotion promotes cached objects only when they are about to be evicted, retaining popular objects with minimal effort.
  - For comparison, FIFO has no promotion, and LRU promotes on cache hit (eager promotion).
- Quick demotion removes most objects shortly after they are cached, which takes advantage of the characteristics of the power-law distribution - most objects are unpopular.

## Algorithm

SIEVE 