---
layout: post
title:  "Concurrency Models"
date:   2023-10-30
desc: "Comparing CSP vs Actor vs Multithreading"
keywords: "Concurrency,Multithreading"
categories: [prog]
permalink: /concurprog/
tags: [Concurrency,Multithreading]
---

## Introduction

Before advent of threads as lightweight processing units that can share memory, processes had to share memory by communicating (IPC). To design concurrent processes that can scale, fault tolerant and efficiently distribute compute load, two approaches were proposed - CSP and Actor model.

## Communicating Sequential Processes (CSP)

<img src="/site/static/assets/img/blog/csp-model.png" alt="CSP Model" width="700"/>

In the 1978 paper [Communicating Sequential Processes](https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf), Tony Hoare put forth a concurrency model that was designed around the fact that concurrent processes communicate among themselves for sharing messages and synchronization using `channels`.
* Channels are named, can be independently created, written to, read from, and passed between tasks.
* Writer to channel is blocked till a reader consumes the message, i.e, synchronous write.
* Processes in CSP are anonymous and can write to/read from any channel they create or have a reference to.

**Go** implements **CSP** using go routines and channels for example.

## Actor model

<img src="/site/static/assets/img/blog/actor-model.png" alt="Actor Model" width="700"/>

In the 1973 paper [Synchronization in Actor Systems](https://dl.acm.org/doi/pdf/10.1145/512950.512975), Carl Hewitt came up with the Actor model to design communication among concurrent processes.
* Processes will send messages to a `mailbox` which is associated with the other process it wants to communicate with to share memory.
* Sending messages to `mailbox` is asynchronous, i.e, non blocking and sender and can continue with it's own execution after message is queued in the `mailbox`.
* When `mailbox` recieves a message, process is scheduled by a scheduler if not already running. After processing all messages in the `mailbox`, it can be removed from the schedule again.
* `Mailbox` or the communicating channel in the Actor model is anonymous and the processes have identities, each identified as an `Actor` in the system.

The **Actor model** can be found in one of it's more popular implementation of [Akka](https://doc.akka.io/docs/akka/current/typed/guide/actors-intro.html) in **Java**.

## Multithreading

Most modern languages have implemented this model of concurrency, where threads are compute units of a process that can share memory during execution without communication. To synchronize threads and prevent race conditions, mechanisms like locks, mutexes, signals and conditions are widely used.

Multithreading involves mutating shared memory among threads, which is very difficult to synchronize and detect design flaws. So unless the design mandates sharing of mutable memory between threads, it's always preferable to go with CSP/Actor model. As mentioned in [Effective Go](https://go.dev/doc/effective_go):
> Do not communicate by sharing memory; instead, share memory by communicating.

## CSP vs Actor Model

* In the first glance, one major difference that can be identified here is that CSP, in it's original thesis, was in fact blocking as writing to `channels` was a synchronous operation. So compared to the Actor model where messages to `mailboxes` are sent asynchronously, **CSP doesn't scale per request basis**.
* Because CSP mandates synchronous communication, it's easier to detect race conditions and deadlocks. Although it's a blocking way of communication that might reduce response times, given right design, spinning up right amount consumers vs producers should increase the overall throughput of the system. So **for the same throughput, CSP can significantly reduce code complexity and easier detection of flaws in design.**
* On closer look, another difference is that in CSP, `channels` or the mode of communication is the primary object, processes are anonymous. In Actor model, `mailboxes` are not created independently (associated with process) and have no existence outside scope of the `Actor` process, which is the primary object here. This provides **CSP flexibility to write to/read from multiple channels that can help design more intuitive systems** unlike Actor model where processes are tied to mailboxes.
* Although Hoare had consciously advocated against having buffers in his paper, modern implementations of CSP (like Go) are also providing the ability to make channels bufferred and non-blocking. This has reduced the gap between both the models.