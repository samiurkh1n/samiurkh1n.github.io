---
layout: post
title: TIDK#3 - Spurious Wakeups in Conditional Variables
date: 2018-12-21 01:00:00.000000000 -05:00
type: post
published: false 
---

*TIDK (short for "Things I Didn't Know") is a series of blog posts about concepts I found difficult to understand. In each article, I take a concept and explain it in a way that allows you to reason about it intuitively. If you find it too complicated, shoot me an email specifying what part is complicated and I'll make edits.*



## Background: A primer on writing multithreaded applications.
### The process abstraction
### Threads, a.k.a Virtual CPUs.
### Mutex and CVs

## The Problem
CVs can wake up without a signal

## The (Lack of a) Solution
Would cause way too much overhead on architecture to ensure wakeups don't happen.

two types of interrrupts to coordinate things
- timer interrupts - needed to ensure smaller programs can get scheduled
- ipi interrupts - processes need to communicate duh

ensuring spurious wakeups don't occur means ensuring no interrupts can hit you?
