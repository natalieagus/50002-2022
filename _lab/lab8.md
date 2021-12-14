---
layout: academic
permalink: /lab/lab8
title: Lab 8 - Tiny OS
description: Lab 8 handout covering topics from Virtual Machine and Asynchronous IO Handling
tags: [lab, kernel, asynchronous, io, dualmode, interrupt, exception]
---

* TOC
{:toc}

**50.002 Computation Structures**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
Modified by: Kenny Choo, Natalie Agus, Oka Kurniawan (2021)
# Lab 1: CMOS

## Starter Code
The following files inside your `/50002/` folder are what you're going to open and modify or study for this lab, then submit (unless otherwise stated):
- `lab8.uasm` 


## Related Class Materials
The lecture notes on [Virtual Machine](https://natalieagus.github.io/50002/notes/virtualmachine), and [Asynchronous handling of I/O Devices](https://natalieagus.github.io/50002/notes/asyncio) are closely related to this lab. 

<br>Related sections in [Virtual Machine](https://natalieagus.github.io/50002/notes/virtualmachine):
* [The OS Kernel](https://natalieagus.github.io/50002/notes/virtualmachine#the-operating-system-kernel): The "TinyOS" in this lab represents a very simple kernel that timeshare the execution of three processes: P0, P1, and P2. It keeps track of the execution of these processes via a process table 
* [OS Multiplexing and Context Switching](https://natalieagus.github.io/50002/notes/virtualmachine#os-multiplexing-and-context-switching): Round robin execution of P0, P1, and P2 in the lab 
* [Asynchronous Interrupt Hardware](https://natalieagus.github.io/50002/notes/virtualmachine#beta-asynchronous-interrupt-hardware): Input from Keyboard and Mouse will interrupt the current process while safely saving the current process' context for later execution
* [Asynchronous Interrupt Handler](https://natalieagus.github.io/50002/notes/virtualmachine#asynchronous-interrupt-handler): the TinyOS is in charge of handling asynchronous Keyboard and Mouse interrupt from the user
* [Trap or Synchronous Interrupt](https://natalieagus.github.io/50002/notes/virtualmachine#trap): A process can ask for user input or OS Services via an ILLOP, thereby forcing a synchronous interrupt or *trap* for the TinyOS to handle

<br>Related sections in [Asynchronous handling of I/O Devices](https://natalieagus.github.io/50002/notes/asyncio): 
* [The Supervisor Call](https://natalieagus.github.io/50002/notes/asyncio#the-supervisor-call): More details on *trap*
* [Asynchronous Input Handling](https://natalieagus.github.io/50002/notes/asyncio#asynchronous-input-handling) and [Real time I/O Handling](https://natalieagus.github.io/50002/notes/asyncio#real-time-io-handling): More details on *async IO* due to Keyboard and Mouse interrupt, as well as timer interrupt




