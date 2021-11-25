---
layout: academic
permalink: /lab/lab1
title: Lab 1 - CMOS
description: Lab 1 handout covering topics from Digital Abstraction and CMOS Technology
tags: [lab, cmos, voltage]
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

## Related Class Materials
The lecture notes on Digital Abstraction and CMOS Technology are closely related to this lab.

**Task A and Task B:** Studying the effect of MOSFET “ON” and MOSFET “OFF”. 
<br>Related Notes: **CMOS Technology**
  * Types of MOSFETs
  * Switching PFET and NFET ON and OFF
  * Realise that producing a logic ‘1’ is not always perfect, 
  * Highly depends on the MOSFET’s conductivity 
  * An “OFF” MOSFET isn’t always 100% off, there exist leaky current

**Task C and Task D:** finding optimal VTC 
<br>Related Notes: **Digital Abstraction**
  * VTC Section
  * Voltage Specifications and Noise Margin 
  * Realise that we can optimise the noise margin by optimising the MOSFET material 
  * Understand why static discipline is important, and how we can analyse VTC to choose the best MOSFET design

**Task E and Task F:** tcd and tpd 
<br>Related Notes: **CMOS Technolog**y
* Timing Specifications of Combinational Logic Devices
* Actually measure tpd and tcd from raw plots (input and output timing of a given gate or circuitry) to understand *by heart*  **why** we need these timing specifications to obey static discipline
* This is on the contrary to classroom exercises where we give you the tpd and tcd of each gate and simply do the math to compute the tpd and tcd of a given circuit

## Introduction to JSim 
*(you really should’ve read this intro section before coming to class)*

In this lab, we will be using a simulation program, JSim, to make measurements of an N-channel MOSFET (or NFET for short). JSim uses **mathematical** models of circuit elements to make predictions of how a circuit will behave both statically (DC analysis) and dynamically (transient analysis). The model for each circuit element is parameterised, e.g., the MOSFET model includes parameters for the length and width of the MOSFET, as well as many parameters that characterize the physical aspects of the manufacturing process. For the models we are using, the manufacturing parameters have been derived from measurements taken at the integrated circuit fabrication facility, and so the resulting predictions are quite accurate.

The (increasingly) complete JSim documentation can be found [here](https://drive.google.com/file/d/1Lc04nVEe6ch9-3wOMoKEa_sBVPuEDtkZ/view?usp=sharing). But we will try to include pertinent information for JSim in each lab writeup.

> Extract 50002.zip, open it and simply double-click jsim.jar.

## User Interface
![Overview of the JSim user interface](/50002/assets/contentimage/lab1/1.png)

Each icon means as follows:
![Icons in JSim](/50002/assets/contentimage/lab1/2.png)

## Waveform Window
The waveform window shows various **waveforms** in one or more “channels.”  Initially one channel is displayed for each `.plot` control statement in your netlist.  If more than one waveform is assigned to a channel, the plots are overlaid on top of each using a different drawing color for each waveform.  If you want to add a waveform to a channel simply add the appropriate signal name to the list appearing to the left of the waveform display (the name of each signal should be on a separate line).  You can also add the name of the signal you would like displayed to the appropriate `.plot` statement in your netlist and rerun the simulation.  If you simply name a node in your circuit, its voltage is plotted. You can also ask for the current through a voltage source by entering `I(Vid)`.

The waveform window has several other buttons on its toolbar:
![waveform button](/50002/assets/contentimage/lab1/3.png)

You can zoom and pan over the traces in the waveform window using the controls found along the bottom edge of the waveform display. The scrollbar at the bottom of the waveform window can be used to scroll through the waveforms. You can recentre the waveform display about a particular point by placing the cursor at that position and pressing `c`.

## JSim netlist format
The JSim netlist format is quite similar to that used by [SPICE](https://en.wikipedia.org/wiki/SPICE), a well-known circuit simulator. Each line of the netlist is one of the following:

* **Comment line**
  * Indicated by an “*” (asterisk) as the first character. 
  * Comment and blank lines are ignored when JSim processes your netlist
  * C++/Java style comments can also be used
  * “//”, all characters starting with this and to the end of the line are ignored.
  * “/*” and “*/”, any lines or parts of lines enclosed by these are ignored.
* **Continuation line**
  * Indicated by a “+” as the first character.
  * Treated as if they had been typed at the end of the previous line (without the “+” of course)
  * No limit to length of an input line, but breaking long lines using “+” makes it easier to edit and understand
  * “+” also continues comment lines
* **Control statement**
  * Indicated by “.” (period) as the first character.
  * Provides information about how the circuit is to be simulated
* **Circuit element**
  * Indicated by a letter as the first character, that represents the type of circuit element. e.g. “r” for resistor, “c” for capacitor, “m” for MOSFET, “v” for voltage source.
  * Remainder of line specifies which circuit nodes connect to which device terminals and any parameters needed by that type of circuit element. For example, the following line describes a 1000Ω resistor called “R1” that connects to nodes A and B: `R1 A B 1k`

> Note that the numbers can be entered using engineering suffixes for readability. Common suffixes are kilo:
> * “k”=1000 
> * micro:“u”=1E-6 
> * nano:“n”=1E-9  
> * pico:“p”=1E-12


## Part 1: Characterising MOSFETs (45 mins)
Make some measurements of an  NFET by hooking it up to a couple of voltage sources to generate different values for VGS and VDS. Our end goal is to obtain the VTC plot of the NFET. Recall that we learn this in the Digital Abstraction chapter in our lecture. 

The setup to characterise our NFET is as follows:
![setupnfet](/50002/assets/contentimage/lab1/4.png)

We have included an ammeter (built from a 0V voltage source) so we can measure IDS, the current flowing through the MOSFET from its drain terminal to its source terminal. Here’s the translation of the above schematic into our netlist format:

```cpp
* plot Ids vs. Vds for 5 different Vgs values
.include "/50002/nominal.jsim"
Vmeter vds drain 0v
Vds vds 0 0v
Vgs gate 0 0v
* N-channel MOSFET used for our test
M1 drain gate 0 0 NENH W=1.2u L=600n
.dc Vds 0 5 .1 Vgs 0 5 1
.plot I(Vmeter)
```


Line | Description
---------|----------
1, 6 | A **comment**. Any line that begins with a * signifies a comment.
2 | **A control statement** that directs JSim to include a netlist file containing the MOSFET model parameters for the manufacturing process we will be targeting this semester. The pathname shown **MUST be MODIFIED** to point at where your `nominal.jsim` file is located
3-5 | These specify **three** voltage sources; each voltage source specifies the two terminal nodes and the voltage we want between them. **Note that the reference node for the circuit (marked with a GROUND symbol in the schematic) is always called 0**. The `v` following the voltage specification is not a legal scale factor and will be ignored by JSim--it is included **just to remind ourselves** that the last number is the voltage of the voltage source. All three sources are initially set to 0 volts but the voltage for the Vds and Vgs sources will be changed later when JSim processes the `.dc` control statement. We can ask JSim to plot the current through voltage sources which is how we’ll see what Ids is for different values of Vgs and Vds.  We could just ask for the current of the Vds voltage source, but the sign would be wrong since JSim uses the convention that positive current flows from the positive to negative terminal of a voltage source.  So we introduce a 0V source with its terminals oriented to produce the current sign we’re looking for.
7 | **This is the MOSFET**, where we have specified in order the names of the drain, gate, source and substrate nodes. The next item names the set of model parameters JSim should use when simulating this device; “NENH” for an NFET, and “PENH” for a p-channel MOFET (PFET). The final two entries specify the width and length of the MOSFET.  Note that the dimensions are in microns (1E-6 meters) since we’ve specified the “u” scale factor as a suffix.  Do not forget the "u" or your MOSFETS **will be meters long**!  You can always use scientific notation (e.g., 1.2E-6) if suffixes are confusing.
8 | **A control statement** requesting a DC analysis of the circuit made with different settings for the Vds and Vgs voltage sources: the voltage of Vds is swept from 0V to 5V in .1V steps, and the voltage of Vgs is swept from 0V to 5V in 1V steps.  Altogether $$51 \times 6$$ separate measurements will be made.
9 | **A control statement** that gets JSim to plot the current through the voltage source named “Vmeter”.  JSim knows how to plot the results from the dual voltage sweep requested on the previous line: it will plot I(Vmeter) vs. the voltage of source Vds for each value of voltage of the source Vgs—there will be 6 plots in all, each consisting of 51 connected data points.

<br><br>
<div class="yellowbox">
<div class="custom_box">
After you enter the netlist above, you might want to **save** your efforts for later use by using the “**save file**” button.  To run the simulation, **click** the “**device-level simulation**” button on the toolbar.
</div>
</div>  
<br>
After a pause, a **waveform** window will pop up where we can take some measurements.  
  * As you move the mouse over the waveform window, a moving cursor will be displayed on the first waveform above the mouse’s position and a readout giving the cursor coordinates will appear in the upper left hand corner of the window.  
  * To measure the delta (difference) between two points, position the mouse so the cursor is on top of the first point. Now click left and drag the mouse (i.e., move the mouse while holding its left button down) to bring up a second cursor that you can then position over the second point.  
  * The readout in the upper left corner will show the coordinates for both cursors and the delta between the two coordinates.  
  * You can return to one cursor by releasing the left button.



