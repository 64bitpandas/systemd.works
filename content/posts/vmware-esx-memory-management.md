---
params:
  math: true
title: "Memory Resource Management in VMware ESX Server"
description: various techniques to improve hypervisor resource usage
summary: various techniques to improve hypervisor resource usage
tags: ["Virtualization"]
date: 2024-07-06
author: James Weng
---

[Waldspurger](https://www.waldspurger.org/carl/) 2002 ([pdf](https://www.usenix.org/legacy/event/osdi02/tech/waldspurger/waldspurger.pdf))

## Overview

Before hypervisors were ubiquitous and operating systems had built-in mechanisms to be aware of running as a guest on a hypervisor, mechanisms from the two often conflicted. This paper addresses several examples of this, including memory management, resource sharing, and deduplicating resources in virtual machines managed by ESX server.

## Background

**Memory management**: Because ESX is a hypervisor/operating system in itself, it must manage memory that its guest virtual machines can use. However, this conflicts with the fact the guest operating systems themselves have sophisticated page eviction algorithms, which can lead to a kind of "split brain" scenario - one example is where the host evicts memory but the guest still believes it is resident. The guest later evicts the page, which requires the host to page it back in before the guest evicts it in the worst case.

**Data sharing**: Many guest VM workloads on a hypervisor will run similar workloads by nature of virtual machines. At the very least, many guest VMs share the same operating system, same binaries, and similar data. Deduplication of data could provide substantial savings in terms of memory.

**Resource sharing**: It is uneconomical for each virtual machine to have all resources available from the hypervisor when it does not need most of them - however, the hypervisor must determine a "fair" way to distribute resources

## Methods

### Memory management

To allow guest OS page eviction policies to function, special drivers called "memory balloon(s)" are installed into them. When the hypervisor needs to reclaim memory, it communicates to the driver to "inflate" the balloon and tricks the guest into thinking more memory is used and then evicts some memory that the hypervisor can then reclaim and allocate to another virtual machine. The balloon stays inflated to ensure the guest retains the specified memory ceiling. When the hypervisor has available resources for the guest to use, the balloon is deflated, allowing memory to be used again. This method generally works, with few exceptions - on Windows, the ballooning driver may not load in quickly enough and memory usage may spike past a set limit, forcing the hypervisor to make non-optimal evictions. Additionally, this relies on guest co-operation, which may not always be available; the hypervisor must have a fallback in case guests are unable to respond to the balloon.

### Data sharing

When multiple guests share the same data (code pages, read-only data), ESX deduplicates the data by adding another layer of indirection from a "physical" (to the operating system) page to a machine (managed by ESX) page. This allows the hypervisor to deduplicate the pages when they are mapped in by hashing the contents of a page and if the hashes match, verifying the content actually matches before remapping the "physical" page. ESX also performs copy-on-write (CoW) so when a guest writes to a shared page, CoW occurs and preserves correctness. However, this becomes untenable at higher-scale due to $O(n^2)$ page search, so ESX picks random times to deduplicate pages. In testing, up to 60% of a guest's memory can be deduplicated, though this is the optimistic case - diverse workloads fared worse due to the time required to check for potential deduplication.

### Resource sharing

ESX uses a ticket/lease mechanism to distribute resources to guests - the more tickets a guest has, the more resources available to them. To prevent a guest from hogging/acquiring all resources and refusing to release them, ESX implements the concept of the "idle memory tax"; the tax comes at reclamation time - guests with more unused resources will be picked first to have resources taken away from them.

## Benchmarks

Many benchmarks were done - refer to the paper for methodology, equations, and results. The readers/author of this summary do not have the pre-requisite background to accurately interpret those results.

## Future Work

In the time since the paper was released, there does not appear to have been much advancement in this area - Linux/KVM [still uses](https://pmhahn.github.io/virtio-balloon/) memory ballooning. At a higher-level, containerization has taken over many of the old use cases of virtual machines and in that case, a regular operating system kernel can rely on traditional OS techniques to do resource management. Additionally, we hypothesize many of these techniques are "good enough" and no one has felt motivated to significantly improve on these techniques.