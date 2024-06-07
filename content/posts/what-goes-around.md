---
title: What Goes Around Comes Around
description: the history of data modeling
summary: the history of data modeling
tags: ["DBMS"]
date: 2024-05-19
author: Ben Cuan
---

Stonebraker & Hellerstein, 2005 ([pdf](https://people.cs.umass.edu/~yanlei/courses/CS691LL-f06/papers/SH05.pdf))

## Background

In modern computing, we often take databases for granted as plug-and-play solutions for reliable, durable, and consistent data storage. 

The modern relational database model and subsequent innovations were build upon decades of iteration. This paper provides an overview of the history of data modeling proposals and standards from the 1960's to 2000's, split into 9 distinct eras.


## IMS

The IBM Information Management System (IMS) was one of the earliest database design proposals when it released in 1968. 

It was tree-based and hierarchical, which is simple but restrictive. (Trees tend to create dependencies on parents, are slow to traverse, and easy to break when modified.)

The biggest lesson from IMS was the importance of **physical and logical data independence.** (IMS's restrictions were mostly tied to poor data independence.)
 - Physical data independence allows us to change how data is stored on disk without affecting the underlying schemas.
 - Logical data independence allows us to change how data is represented (i.e. by modifying records, splitting tables) without needing to rewrite any applications that rely on the data.

## CODASYL

CODASYL (Committee on Data Systems Languages) came together in an attempt to form a industry-wide standard for data modeling. They released a report just a year after IMS, in 1969.

The CODASYL protocol was mostly based on IMS; its main difference was that it supported **general directed graphs.**

Although the graph solution was much more flexible than trees, the complexity made CODASYL more difficult to manage in practice (i.e. recovering from crashes or reloading).

It also didn't address several limitations lingering from IMS:
 - CODASYL was still **record-at-a-time**, which meant that every record in the database had to be fetched individually.
 - Query optimization must be done manually, since the naive fetching algorithm doesn't have any knowledge of the graph structure.
 - CODASYL also doesn't have any physical or logical data independence guarantees.

## The Relational Era
Introduced in 1970 by Ted Codd, the relational database model is the first to resemble modern DBMS architecture.

Codd proposed three major principles:
1. Use a simple table data structure.
2. Use a **set-at-a-time** language: in contrast to record-at-a-time, it can fetch batches of similar records all at once, improving efficiency.
3. Focus on physical data independence by excluding any hardware design in the standard.

One massive benefit of the relational model was the ability to **automate query optimization.** Although early query optimization was very primitive, it had the potential to *eventually* surpass manual human optimizations (clearly, this prediction came true).

Since relational databases were such a departure from CODASYL, computer researchers created a lot of internal drama about which one should prevail. It wasn't until 1984 when IBM released DB/2, making relational databases mainstream and settling the debate.


## Some small relational DB improvements

The next few eras didn't really stick around. The moral of the story here is that making small changes that don't impact performance or core functionality don't tend to stick around.

 - The **entity-relationship model** introduced a new way to think about databases as a collection of entities, each with specifically defined relationships between them. While used widely for schema design, no query language was ever proposed for it.
 - The **R++ Era** is a collection of suggested minor improvements to the relational model (such as graphics support and aggregation). These were also never widely adopted into a standard language.
 - The **semantic data model** attempted to add inheritance to relational databases, like a primitive object-oriented system. Since this idea was incompatible with SQL (and even opposed it in some cases) it never took off.

## Object Oriented Databases

In the 1980s, object-oriented programming started taking off (notably C++), and researchers tried to adapt its principles to fit into databases. 

The most promising idea was that of a **persistent programming language** which allowed variables to represent persistent data stored on the disk. Examples included Pascal-R and Rigel.

Although very useful for certain niches in software engineering, OODB's negated some advantages for databases when used in desktop applications:
 - It operated in the same address space as the application itself.
 - It required rewriting the application code to support the new languages.
 - Once it was rewritten in a persistent programming language, it caused lock-in to that language. (OODB vendors never converged on a standardized, compatible implementation.)

The lasting legacy of OODB's ended up being its inspiration as a precursor to the next era of object-relational databases.


## Object-Relational Databases

Object-relational databases allow users to define their own types, operators, functions, and access methods on top of SQL itself. In other words, the language could now directly manipulate stored data.

User-defined types (UDTs) and functions (UDFs) could be made much more efficient than any comparable SQL-only solution for certain abstractions, especially multidimensional ones like geography (i.e. searching for locations near a given coordinate).

The most prominent example of an object-relational databases is Postgres, which is still widely used today.

The biggest drawback of object-relational models is their lock-in: defining UDTs/UDFs meant it would be difficult to switch languages later on.

## Semi-Structured Data

At the time of publication, the rising newcomer to the data model scene was XML, a semi-structured, self-describing (aka schema-last) data language. In practice, it allows users to arbitrarily structure their data however they want; records can also link to, inherit from, or exist in sets with other records.

The authors of this paper criticize XML and its derivatives for being far too complex to be practical, and expect it to fall out of fashion. We can now see that they were correct.

## What came next?

This paper was written in 2005, so we'd expect some aspects of it to be outdated. Most notably, it doesn't cover two future major industry innovations:
 - The rise of NoSQL solutions like MongoDB and Firebase directly succeeded early semi-structured data models like XML. They continue to be widely popular today, though only for a specific subset of use cases that are unfavorable towards relational models.
 - The data science and big data pipeline is much more formalized now, and serves increasing needs for extremely large amounts of data. This creates outsized requirements for modern DBMS's to be scalable, parallelizable, and distributed. 
 
The current cutting edge of DBMS includes distributed solutions like Snowflake, Databricks, Apache Spark, and DynamoDB. They notably strike a middle ground by using complex, optimized internal data models that may be exposed via a SQL-compatible language.

The paper's main thesis of coming "full-circle" from simplicity to complexity back to simplicity continues to hold true: even with modern engineering requirements demanding complex and feature-rich data modeling solutions, the simple relational model remains by far the most popular implementation to this day.

