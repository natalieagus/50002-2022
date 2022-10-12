---
layout: academic
permalink: /notes/virtualmachine
title: Week 11 Part 2 - Virtual Machine
description: To give each process the illusion that they can operate in the whole address space and use the entire machine to itself, while in fact, we are sharing the machine among multiple processes.
tags: [virtual, kernel, asynchronous, io, dualmode]
---

- TOC
  {:toc}

**50.002 Computation Structures**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design

# Virtual Machine

[You can find the lecture video here.](https://youtu.be/4pizOgCT11k) You can also **click** on each header to bring you to the section of the video covering the subtopic.

## [Overview](https://www.youtube.com/watch?v=4pizOgCT11k&t=0s)

Suppose we have 10 processes running in our computer right now: Web Browser, Spotify, Telegram, etc (in fact, the number of running processes at any given time in a typical computer is more than 10). There has to be some kind of **manager** program that oversees the execution of these processes because we only have limited amount of resources: CPU cores, RAM size, cache size, etc. This **manager** program is called the Operating System. Specifically, the part of the OS that is responsible for process management is the **operating system kernel**.

### [The Operating System Kernel](https://www.youtube.com/watch?v=4pizOgCT11k&t=95s)

The Operating System (OS) Kernel is a _special_ program that is written to **_manage_** and **oversees** the execution of all other processes in system. It has the **highest privilege** in computer system, i.e: it can terminate any program, has access to all kinds of hardware resources (Physical Memory, I/O devices).

A few of its important role include _memory management_, _I/O handling_, and _process scheduling_.

> There are many other roles of the OS Kernel that is not discussed here. We will learn more about them next term.

### [A Complete Process Context](https://www.youtube.com/watch?v=4pizOgCT11k&t=336s)

In the previous chapter, we learned that each process has its own _`VA` to `PA`mapping_ we call as part of a process **context**, hence allowing it to run on its own _virtual memory_.

Assigning a separate context for each process has two crucial benefits:

1.  Allows **timesharing** among processes, hence user can multitask, i.e: switch the execution of multiple programs using a single core.

1.  Allows each process to run in **isolation** -- therefore every program can be written as if it has **access to all memory**, without considering where other programs reside.

The Kernel need to store more information about a process (_and not just its `VA` to `PA` mapping_), so that it can _pause the execution_ and _resume_ any of them later on without any conflict.

A more complete list of components that make up a process **context** are:

- Values of `R0, R1, ... R30`
- `VA` to `PA`mapping
- PC value
- Stack state
- Program (and shared code)
- Virtual I/O devices (console, etc)

In the Figure below, we illustrate `N` processes that are present in the system: `P1, P2, ..., P3` -- each having its own information:

> These processes are **isolated** from one another, meaning that `Pi` cannot access (or corrupt) the memory space of other process `Pj` because each of them run on a separate virtual memory.

<img src="https://dropbox.com/s/fvo6fllqrwwg2qr/context.png?raw=1"  style="width: 70%;"  >

Writing an Operating System Kernel is not a trivial task as one has to take into consideration a plethora of _issues_ (security, performance, memory management, scheduling, etc). However with its presence, it makes _easier to write all other programs_. **It provides a layer of abstraction, allowing each program to run on a _virtual machine_**, devoid of any knowledge about any other processes.

### [Building a Virtual Machine ](https://www.youtube.com/watch?v=4pizOgCT11k&t=710s)

#### [Kernel Mode and User Mode](https://www.youtube.com/watch?v=4pizOgCT11k&t=798s)

To support a safe _virtual machine_ for each process, we need to establish the notion of **dual mode system**, that is a system that has a **Kernel Mode** (privileged mode) and a **User Mode** (non-privileged mode):

- The OS Kernel runs in _full privilege_ mode called the **Kernel Mode**, and it oversees the execution of all processes in the computer system, handles real I/O devices, and emulate virtual I/O device for each process.

- All other programs do **not** have such _privileged_ features like the kernel. We call these programs as running in _non-privileged_ **mode** called the **User Mode** with limited access to any hardware resources:
  - No direct access to actual hardware
  - No direct access other process' address space
  - No knowledge about other processes' context and processor state

The Kernel will **handle** the need of these programs running in user mode for access to various hardware resources: access to I/O devices, interprocess communication, allocation/deallocation of shared memory space, etc.

This is a **major benefit**: programs can be easily written as if they have _absolute_ access to _all_ hardware resources (not just the physical memory), without having to worry about sharing them with other running processes.

## [OS Multiplexing and Context Switching](https://www.youtube.com/watch?v=4pizOgCT11k&t=1080s)

**Multiplexing** is a method of sharing the resources in a computer **system** for multiple running programs at the same time. The OS kernel handles the multiplexed execution of various running programs in a single CPU -- **switching between _contexts_ so rapidly** -- so that for the users, the computer is seemingly able to run multiple processes in "_parallel_".

The main idea of OS multiplexing is illustrated below using two processes `P1` and `P2`, sharing a single system:

<img src="https://dropbox.com/s/p5r7q2uit6vbdkz/process.png?raw=1"  style="width: 90%;"  >
  
The `ARROW` illustrates the timeline of execution:
* At first, the CPU runs some task from `P1`.

- After some time `t`, imagine that a _timed interrupt_ (caused by other asynchronous hardware, e.g: a _timer_) occurs. This causes the CPU to execute part of the kernel program that handles such **asynchronous interrupt**, hence _pausing_ the execution of `P1`.
- This _interrupt handler_ takes control of the CPU when hardware interrupt occurs, and **saves** the current **states** (PC, Registers, etc) of P1 to a dedicated space **(Kernel Stack)** in the Memory Unit (_so that P1's progress is not lost and can be resumed later on_) before performing a "_context switch_": 1. Load the states of `P2` to the CPU (and also the required resources, mapping, etc), and 2. Resume the execution of `P2`.

  > In practice, the _interrupt handler_ will examine the _cause_ of the asynchronous interrupt. In the event of periodic interrupt caused by a timer, the handler will delegate the task to the **kernel scheduler** whose job is to decide _which_ process to run next, and prepare the necessary information / context to load this process back into the CPU so that the selected process may resume smoothly. When the scheduler returns to the handler, the handler resumes execution of the CPU by simply setting `PC` $$\leftarrow$$ `Reg[XP] - 4`.

- `P2` runs and progresses for some time `t` before another _hardware interrupt_ occurs. The entire context switch process is repeated to pause `P2`, resume `P1`, and so forth.

The key technology that allows for OS Multiplexing is the **asynchronous hardware interrupt.**

> We will simply call asynchronous interrupt as just "interrupt" for simplicity. A synchronous interrupt is called as "_trap_" instead (see the later chapters).

## [Hardware Support for OS Multiplexing ](https://www.youtube.com/watch?v=4pizOgCT11k&t=1285s)

To allow for proper multiplexing, four things must be supported \***\*in the hardware level\*\***:

1.  There has to be a way to **asynchronously interrupt** a currently running program _periodically_ via hardware, since that program is currently using the CPU and will not stop voluntarily.

2.  The hardware has to know how to **direct** the PC CPU to the right handler program when **interrupt** occurs.

3.  **Two execution modes** in the system:

    - **Kernel mode**: that allows the CPU to have ultimate access to all hardware and data, so that it can perform crucial process management tasks such as "_saving_" the states (Register contents, stack, PC, etc) of the interrupted process (to be resumed safely later on).
    - **User mode**: a non-privileged mode that disallow programs to corrupt illegal memory space of other programs or hijack resources.

4.  Other interrupts must be **disabled** when this process of "saving state" occurs (otherwise data will be lost).

### $$\beta$$ Asynchronous Interrupt Hardware

Recall the **asynchronous interrupt** datapath as shown in the figure below:

<img src="/50002-2022/assets/contentimage/beta/irq.png"  class="center_full"/>

One of the inputs that is received by the Control Unit is `IRQ` (1-bit). In the event of _interrupt_, the `IRQ` value will be `1`.

At each CLK cycle, the Control Unit always checks whether `IRQ` is `1` or `0`.

> Note that `IRQ` may turn to be `1` asynchronously, e.g: in the "_middle_" of a particular CPU CLK cycle. However the Control Unit is synchronised with CPU CLK. Therefore, this will only _trigger_ an interrupt in the next CPU CLK tick.

- If `IRQ==0`, the Control Unit produces all control signals as dictated by `OPCODE` received.
- Else if `IRQ==1`, the Control Unit _traps_ the PC onto the interrupt handler located at `XAddr`, by setting `PCSEL` value into `100`; _so that the PC points to `XAddr` in the next clock cycle._
  - At the same time, it stores the address of the _next_ instruction (`PC+4`) at Register `XP` (`R30`).
  - `R30` is a **special register** is always used to hold the _return address_ in the event of interrupt (or illegal operation) so that the system knows how to resume the interrupted program later on.

A kernel scheduler will typically configure some system timer to _fire_ at some interval. This timer runs **asynchronously** with the CPU, and sets the `IRQ` signal to `1` each time it _fires_.

> The interrupt hardware configuration **forces** the PC CPU to execute the interrupt handler at `XAddr` in the next cycle each time the timer _fires_.

The register transfer language that describes what happens in the datapath when `IRQ==1` is:

```cpp
If (IRQ==1 && PC31 == 0):
	Reg[XP] <- PC + 4`
	PC <- Xaddr
```

### [Asynchronous Interrupt Handler ](https://www.youtube.com/watch?v=4pizOgCT11k&t=1664s)

The asynchronous interrupt handler is located at `XAddr`, which is usually pre-determined memory address. In $$\beta$$ CPU, `XAddr` is set at `0x8000 0008`.

The first few instructions of the interrupt handler saves current process states (`R0` to `R30` contents, PC state, stack, and others) in **process table**.

> **Process table:** a Kernel data structure that stores all the states of running processes in the machine. It lives in the Kernel memory space. The kernel keeps track on which process is currently scheduled to run in the CPU.
> <img src="https://dropbox.com/s/ypgac0w1uotc471/proctable.png?raw=1"   style="width: 70%;" >

Then, the handler will figure out which specific **service routine** needs to be called to _service_ the interrupt, e.g: scheduler, or I/O routines.

Afterwards, the service routine returns back to this interrupt handler. The handler finally sets `PC` $$\leftarrow$$ `Reg[XP]-4`.

> What is the value of `Reg[XP]-4`?

_It depends._ The **service routine** may or may not change the value of `Reg[XP]` before returning to the interrupt handler:

- If the value of `Reg[XP]` is unchanged, then the interrupted program resumes.
- Else, it means that the CPU executes _another_ program.

> In any case, `Reg[XP]-4` contains the address of instruction that the CPU should execute when the interrupt handler returns.

### [Dual Mode Hardware Support](https://www.youtube.com/watch?v=4pizOgCT11k&t=1971s)

Since the OS Kernel is a program that manages the execution of all other processes in the system, it is **crucial** to _restrict_ access to the Kernel for **safety reasons**.

> That is, to prevent a normal program from _jumping_ to the address in memory that contains Kernel code and "_hack_" the system.

This **prevention** is done via hardware.

Firstly, we need to establish some notion:

- We call the MSB (most significant bit) of the PC register as the **Supervisor Bit**.
- Whenever the PC executes any code in an address where its MSB is `1`, it means that the CPU is running in the **Kernel Mode**.
- Otherwise, if the MSB of the content in PC Register MSB is 0, the CPU is said to be running in the **User Mode**.

That means we can divide the physical memory address space into two sections:

- **User space**: Addresses which MSB is `0`: from `0x0000 0000` to `0x7FFF FFFF`
- **Kernel space**: Addresses which MSB is `1`: from `0x8000 0000` to `0xFFFF FFFF`.

Kernel program and kernel data (privileged information, data structures, etc) are stored in the Kernel space. The rest of the program in the system live in the user space.

With this notion, it is easy to enforce restricted access to the kernel space:

- Programs running in user mode (`PC31 == 0`) can never _branch_ or _jump_ to instructions in the kernel space.
  > Computations of next instruction address in`BEQ`, `BNE`, and `JMP` cannot change `PC31` value from `0` to `1`.
- Programs runing in user mode (`PC31 == 0`) can never _load_/_store_ to data from/to the kernel space.
  > Computations of addresses in `LD`, `LDR` and `ST` ignores the MSB.
- Entry to the kernel mode can only be done via restricted entry points. In $$\beta$$, there are only three entry points:
  - Interrupts (setting PC to `Xaddr: 0x8000 0008`),
  - Illegal operations (setting PC to `ILLOP: 0x8000 0004`), or
  - Reset (setting PC to `RESET: 0x8000 0000`)

### [Reentrancy](https://www.youtube.com/watch?v=4pizOgCT11k&t=2045s)

When the CPU is in the kernel mode (`PC31 == 1`), i.e: handling an interrupt -- it is important to consider whether or not we should allow interrupts to occur. Handlers which are interruptible are called **re-entrant**.

In $$\beta$$ CPU, handlers are **not re-entrant.** Interrupts are **disabled** when it is in kernel mode:

- `IRQ` signal is ignored in the hardware when `PC31 == 1`

> This means that while user programs are interruptible, kernel programs are not.

The reason behind disabling interrupt while being in the Kernel mode is to prevent the Kernel from corrupting itself.

> Consider the scenario where the interrupt handler is in the middle of saving program states. Allowing another interrupt to occur in the middle of a save might cause data corruption.

The drawback to an uninterruptible kernel is that there's no way to get the system to work again if the kernel is buggy and runs into an infinite loop, except via hard reset. The kernel program has to be written very carefully so as not to contain such bugs.

### Timer Example

In this section, we illustrate an example of how a basic Kernel Scheduler works with the support of hardware.

Consider a $$\beta$$ computer having a 16-bit counter (_hardware_) with frequency of 50 Hz that runs **asynchronously** with the CPU. This counter will be used as a timer for process scheduling.

Upon start-up, the Kernel can set the IRQ signal to point to the an arbitrary bit of the counter (assume that the pointer's output is passed through a rising-edge detector so IRQ is `1` for 1 CPU clock cycle during a rising edge only).

For example, if it points to the `4`$$^{th}$$ bit:

- The value of the `4`$$^{th}$$ bit of the counter changes every $$0.02 \times 2^{3}$$ = 0.16 seconds because it takes 0.02 seconds for the counter to increase by 1.
- There's 0.32 seconds between rising edges.

This means that the `IRQ` value will be `1` **once** every 0.32 seconds.

If at first the CPU is executing instructions of Program `P1`:

1.  After 0.32 seconds, `IRQ` turns to `1`. This triggers an interrupt, and the control signals will cause the PC will execute the interrupt handler instruction at `XAddr` in the next cycle (and saving the _supposed_ _next_ instruction at `Reg[XP]`).

2.  The handler at `XAddr` must _save register states_, branch to the _scheduler_, and resume the program after the scheduler returns. Note that `Reg[XP]` may or may not be the same as when _before_ `BR(scheduler_handler, LP)` is executed.

```cpp
X_addr : ST(R0, save_location) || save register states at an allocated address
ST(R1, save_location+4)
ST(R2, save_location+8)
ST(R3, save_location+12)
....
ST(R30, save_location+30*4)

CMOVE(kstack, SP) || use kernel stack
BR(scheduler_handler, LP) || branch to the scheduler

|| return instruction from scheduler
LD(save_location,R0) ||  restore register states
LD(save_location+4,R1)
...
LD(save_location+30*4, R30)

SUBC(XP, 4, XP) || Reduce XP by 4 to re-execute the instruction that was interrupted by the timer
JMP(XP)  || Resume execution
```

> Although not written, `save_location` is a label, representing an address to store P1's states.

Observation: in this simple example, the handler is written such that it _always branches to the scheduler_. In practice, there are many kinds of hardware interrupts (not just from a timer) that needs to be handled differently depending on its _type_. We will have a hands-on experience about this in Lab 8, and also in the next term.

## [Trap](https://www.youtube.com/watch?v=4pizOgCT11k&t=2120s)

A trap, is type of **synchronous interrupt** caused by an _exceptional_ condition when the CPU executes an instruction, such as illegal operations, division by zero, invalid memory access, system calls etc.

This results in a switch to kernel mode via trap handler (e.g: `PC` $$\leftarrow$$ `ILLOP`). The handler will examine the cause of the trap, and perform the appropriate action before (_if possible_) returning control to the originating process.

> If it is not possible to return control to the originating processes, then the Kernel may choose to terminate it.

User processes do not have _privileged_ access, meaning that they _do not directly control the use of_ any hardware (I/O) devices, such as getting keyboard input, mouse click, perform disk saves, etc, _without the help of the OS Kernel program_.

> This is because hardware devices **are actually shared** among all processes in the system, but their programs are written with complete disregard for other processes in the memory.

Therefore, user processes may utilise **traps** to synchronously interrupt themselves, and _legally_ switch to the Kernel mode whenever they need access to the I/O devices (or other kernel services).

<span style="background-color:yellow; color: black"> The event of transferring control of the CPU to OS Kernel synchronously / voluntarily when a process needs Kernel's services is known as the **system call** (a.k.a: **SVC**, or **supervisor call**). This can be done by leaving the index of the requested service at `Reg[R0]` and executing a specific _illegal operation_. </span>

> There are many types of Kernel services, one of them includes read/write access from/to the I/O devices. They are typically _indexed_, and the process needs to leave the index of the needed system call in `Reg[R0]` before trapping itself to the Kernel Program. We will learn about Kernel services next term.

The datapath in the event of _illegal operation_ is:

<img src="/50002-2022/assets/contentimage/beta/illop.png"  class="center_full"/>

During this event,

- Control unit sets `PCSEL = 011`, and saves `PC+4` into `Reg[XP]`
- The PC will execute the instruction at location `ILLOP` in the _next cycle_ where the illegal operation handler resides.
- The _illop_ handler will look at `Reg[R0]` and invoke the right _service routine_ to provide the requested service.
  - Upon returning, the service routine will put its return the result in `Reg[R0]`.
- The _illop_ handler resumes the execution of the originating process:
  - `Reg[XP] = Reg[XP] -4`
  - `JMP[XP]`

One common scenario where a process running in user mode needs the Kernel service is when it asks for keyboard / mouse input, for example:

```cpp
int c;
c = getchar();
```

The function `getchar` contains several instructions that perform a **supervisor call** in order to fetch any character input from the keyboard. When translated into assembly, the supervisor call is made by _trapping_ the process into the _illop_ handler, thus **transferring** CPU control to the Kernel so that it can fetch any character input from the keyboard, and **resuming** the process execution after the task is done.

> The process stores the character input left at `Reg[R0]` by the Kernel into memory location `c`.

## [Summary](https://www.youtube.com/watch?v=4pizOgCT11k&t=2495s)

[You may want to watch the post lecture videos here. ](https://youtu.be/uG1HEKdJpxY)

In summary, we have learned how the presence of OS Kernel and hardware support provide an abstraction for each running process, thus allowing them to run in an isolated manner; on their own virtual machine.

The Kernel **manages** the execution of all processes, as well as all I/O devices, and provides **services** to all these processes. There are two ways to transfer CPU control between user programs to kernel programs:

- Firstly, is through **asynchronous interrupt**: `IRQ` is set to `1`
- Secondly, is through **s ynchronous interrupt**: when the process generates an **exception** hence **trapping** itself to the handler and enters Kernel mode.

During either case of interrupt, `PC+4` is stored at `Reg[XP]` so that the system knows how to resume the process later on.

In $$\beta$$ ISA, the Kernel is **non-preemptive** (the CPU cannot be interrupted while in Kernel Mode). It is designed as such to prevent security breach, data loss if it traps into itself while still being in the Kernel Mode, etc. However, careful writing and construction of the Kernel program is required.
