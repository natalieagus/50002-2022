---
layout: academic
permalink: /lab/lab3
title: Lab 3 - Arithmetic Logic Unit
description: Lab 3 handout covering topics from Logic Synthesis, and Designing an Instruction Set
tags: [lab, beta, booleanalgebra, binary]
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

# Lab 3: Arithmetic Logic Unit

## Starter Code
Download the materials required for this lab [here](https://drive.google.com/open?id=102xUAuturtlEo0a8Cms6wI8YGAgy4EMo&authuser=nn88blue%40gmail.com&usp=drive_fs). 

## Related Class Materials
The lecture notes on [Logic Synthesis](https://natalieagus.github.io/50002/notes/logicsynthesis) and [Designing an Instruction Set](https://natalieagus.github.io/50002/notes/instructionset) are closely related to this lab. 

**Task A:** Design the ALU (Part 1-4) and **Task B:** Studying a Multiplier Design
<br>Related sections in the notes: **[Logic Synthesis](https://natalieagus.github.io/50002/notes/logicsynthesis)**	
* [N-input gates](https://natalieagus.github.io/50002/notes/logicsynthesis#n-input-gates) (all kinds of gates to produce the logic of each component in the ALU)
* [Special combinational logic devices](https://natalieagus.github.io/50002/notes/logicsynthesis#special-combinational-logic-devices) (multiplexer with 1 or 2 selectors, and combining multiplexers together to form an even bigger one)

**Task C:** Combine each combinational element in Task A and B to form an ALU
<br>Related sections in the notes: **Designing an Instruction Set**
* [Basics of programmable control systems](https://natalieagus.github.io/50002/notes/instructionset#an-example-of-a-basic-programmable-control-system) (using control signals like ALUFN to **perform different operations (`ADD`, `SHIFT`, `MUL`, etc)** between two **32-bit** inputs A and B in the same ALU circuit -- no hardware modification needed). 

The lab will reinforce your understanding on how you can build the circuit to conform to the logic that you want, e.g: adder circuit will perform binary addition of input A and B, etc, and make it **PROGRAMMABLE** using the control signal: `ALUFN`. 

## Introduction: More JSim
*(you really should’ve read this intro section before coming to class)*

In this lab, we will build the **arithmetic and logic unit (ALU)** for the Beta processor. 
> The ALU is a **combinational** logic device that has two **32-bit inputs** (which we will call “A” and “B”) and produces one **32-bit output**. 
> We will start by designing each piece of the ALU as a separate circuit (Task A and B), each producing its own 32-bit output. 
> We will then combine these outputs into a single ALU result (Task C).

Before we begin, there are a few more JSim tricks that you have to know to make it easier to build such a huge programmable device. 

### Standard Cell Library
The building blocks for our design will be a family of **logic gates **that are part of a **standard cell library**, declared for you in the file called `stdcell.jsim` given in your courseware. [You can find its documentation here](https://drive.google.com/open?id=1ArkRewfiBqJGmVqzkiGzFxbS0fZ-2eWw&authuser=nn88blue%40gmail.com&usp=drive_fs). The available combinational gates are listed in the table below along with information about their **timing**, **loading** and **size**. You can access the library by starting your netlist with the following include statements:

```cpp
.include " nominal.jsim"
.include " stdcell.jsim" 
```

<div class="redbox"><div class="custom_box">We will no longer need to create custom gates from scratch using MOSFET, unlike in our Lab 1 and 2. From **now onwards**, please build your combinational logic devices using the gates provided in `stdcel.jsim`.</div></div><br>

### Gate-level Simulation

Since we are designing at the **gate level** we can use a **faster** simulator that only knows about gates and logic values (<span style="background-color:yellow; color: black">instead of transistors and voltages</span>).

> You can run JSim’s gate-level simulator by clicking the GATE button in the JSim toolbar. DO NOT click the transient analysis button anymore. 

<img src="/50002/assets/contentimage/lab3/1.png"  class="center_seventyfive"/>

*Note that your design **cannot** contain any mosfets, resistors, capacitors, etc.; this simulator only supports the gate primitives in the standard cell library.*

Inputs are still **specified in terms of voltages** (to maintain netlist compatibility with the other simulators) but the gate-level simulator converts voltages into one of three possible logic values using the Vil and Vih thresholds specified in nominal.jsim:
 
* `0`	logic low (voltages less than or equal to VIL threshold)
* `1`	logic high (voltages greater than or equal to VIH threshold)
* `X`	unknown or undefined (voltages between the thresholds, or unknown voltages)
* `Z`   the value of nodes that aren’t being driven by any gate output (e.g., the outputs of tristate drivers that aren’t enabled).  

The following diagram shows how these values appear on the waveform display:
<img src="/50002/assets/contentimage/lab3/2.png"  class="center_seventyfive"/>

### .connect in JSim
JSim has a control statement that lets you connect two or more nodes together so that they behave as a **single** electrical node:

```cpp
.connect node1 node2 node3...
```

The `.connect` statement is useful for connecting two terminals of a subcircuit or for connecting nodes directly to ground. For example, the following statement ties nodes `cmp1, cmp2, ..., cmp31` directly to the ground node (node `0`):

```cpp
.connect 0 cmp[31:1]
```

### .connect Warning
Note that the `.connect` control statement in JSim works differently than many people expect: it **does NOT connect element-wise** For example,

```cpp
.connect A[5:0] B[5:0]
```

will connect all twelve nodes (`A5, A4, ..., A0, B5, B4, ..., B0`) **TOGETHER**. It is essentially like soldering all 12 pins together, instead of connecting them element-wise. To connect the two buses element-wise, you should use:

```cpp
.connect A5 B5
.connect A4 B4
.connect A3 B3
.connect A2 B2
.connect A1 B1
.connect A0 B0
```

The above can be **tedious** to type. To fix this, can define a two-terminal device that uses    internally, and then use the usual **iteration** rules (see next section) to make many instances of the device with one `X` (device) statement:

```cpp
* declare knex subcircuit
.subckt knex a b
.connect a b
.ends

* use knex subcircuit to connect A and B buses element-wise
X1 A[5:0] B[5:0] knex
```

### JSim Iterators

JSim makes it **easy** to specify **multiple gates** with a single `"X"` statement. 
> You can create multiple instances of a device by supplying some multiple of the number of nodes it expects, e.g., if a device has 3 terminals, supplying 9 nodes will create 3 instances of the device. 

For example, a device called `xor` with 3 terminals: two inputs and one outputs (**POSITIONAL ARGUMENT**) can be instantiated 3 times in a single instruction as such:

```cpp
Xtest a[2:0] b[2:0] z[2:0] xor2
```

is equivalent to:
```cpp
Xtest#0 a2 b2 z2 xor2
Xtest#1 a1 b1 z1 xor2
Xtest#2 a0 b0 z0 xor2
```

### Duplicating a signal
There is also a handy way of duplicating a signal. For example, xor-ing a 4-bit bus with a control signal `ctl` could be written as

```cpp
Xbusctl in[3:0] ctl#4 out[3:0] xor2
```

which is equivalent to:

```cpp
Xbusctl#0 in3 ctl out3 xor2
Xbusctl#1 in2 ctl out2 xor2
Xbusctl#2 in1 ctl out1 xor2
Xbusctl#3 in0 ctl out0 xor2
```

### Connecting Multiple Nodes to ground 
Using iterators and the “constant0” device from the standard cell library, here’s a better way of connecting a sample 32-bit bus `a[31:0]` to ground:

```cpp
Xgnd a[31:0] constant0
```

`constant0` is a device defined in standard cell library and it has only one terminal (just one input and no output).  This is equivalent to: 

```cpp
Xgnd#0 a31 constant0
Xgnd#1 a30 constant0
Xgnd#2 a29 constant0
...
Xgnd#31 a0 constant0
```

> The effect of doing this is that `a[31:0]` will **always** give an output of `0` since it is **connected to ground**. It is a convenient way to *zero* unused output ports. 

### Optimising Circuitry
There are many gates that are available in the standard cell library. We can use any gates to synthesize any logic device. However, when designing circuits there are three separate factors that can be optimised:
1. Design for **maximum performance** (minimum latency)
2. Design for **minimum cost** (minimum area)
3. Design for the best cost / performance **ratio** (minimise area * latency)

It is often possible to do all three at once but in some portions of the circuit some sort of *design tradeoff *will need to be made.  When designing your circuitry you should **choose** which of these three factors is most **important** to you and optimize your design (use the correct gates) accordingly. You will have to make such design choices in your 2D project. 


