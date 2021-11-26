---
layout: academic
permalink: /lab/lab5
title: Lab 5 - Beta Processor
description: Lab 5 handout covering topics from Beta Datapath
tags: [lab, instructionset, combinational, sequential, beta, interrupt, exception, datapath]
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

# Lab 5: Beta Processor

## Materials
Download the materials required for this lab here. 

## Related Class Materials
The lecture notes on [Building the Beta CPU](https://natalieagus.github.io/50002/notes/betacpu), and [Designing an Instruction Set](https://natalieagus.github.io/50002/notes/instructionset) are closely related to this lab. 

There are two major Tasks: **Task 1** and **Task 2** in this Lab. They will reinforce your understanding on how the Beta CPU works, and all data paths for OP, OPC, Control Transfer, and Memory Access operations. 

**Task 1:** Support OP/C and Memory Access Instructions (Part 1 to 4)
<br> Related sections in [Beta CPU](https://natalieagus.github.io/50002/notes/betacpu):	
* [OP datapath](https://natalieagus.github.io/50002/notes/betacpu#op-datapath)
* [OPC datapath](https://natalieagus.github.io/50002/notes/betacpu#opc-datapath)
* [Memory Access datapath](https://natalieagus.github.io/50002/notes/betacpu#memory-access-datapath)
* This lab will help you to familiarise yourselves with all Beta instructions
* By the end of this lab, you should know how to build the schematic of the entire Beta CPU based on its [ISA](https://natalieagus.github.io/50002/notes/instructionset#the-beta-instruction-set-architecture) (blueprint) 

<br>Related sections in [Designing an Instruction Set](https://natalieagus.github.io/50002/notes/instructionset):	
* [The Von Neumann model](https://natalieagus.github.io/50002/notes/instructionset#the-von-neumann-model): CPU, Memory, IO
* [Programmability of a Von Neumann Machine](https://natalieagus.github.io/50002/notes/instructionset#programmability-of-a-von-neumann-machine): basics of programmable control systems (using control signals like `OPCODE` to activate different data paths in the Beta CPU). 
* [Beta ISA Format](https://natalieagus.github.io/50002/notes/instructionset#beta-isa-format)
* [Beta Instruction Encoding](https://natalieagus.github.io/50002/notes/instructionset#beta-instruction-encoding)

**Task 2:** Adding Support for Transfers of Control to Complete the Beta CPU (Part 1-6)
<br> Related sections in [Beta CPU](https://natalieagus.github.io/50002/notes/betacpu):	
* [Control transfer datapath](https://natalieagus.github.io/50002/notes/betacpu#control-transfer-datapath)
* [Exception handling](https://natalieagus.github.io/50002/notes/betacpu#exception-handling)

