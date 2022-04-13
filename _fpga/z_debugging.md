---
layout: academic
permalink: /fpga/debugging
title: Debugging for the Frantic
description: Getting familiar with Alchitry Lab's debug feature
tags: [fpga]
---


* TOC
{:toc}

**50.002 Computation Structures**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Ian Goh (Spring 2022)**

# Debugging the FPGA

This document is written to guide you with debugging the FPGA using Alchitry Lab's debug feature. That said, there's nothing wrong using outputs such as the LEDS or 7-segment display to debug. Use which ever method you prefer.

## Capture Signals
### Debug Project
Start by selecting the `Debug Project` (bug looking icon) in the toolbar.

![](https://www.dropbox.com/s/ss9x34e5hoeu1xs/debug_icon.png?dl=1)

### Select Signals
Select the signals you would like to capture during debugging by ticking the respective checkboxes.

> Example to debug FSM current state (`0`-indexed in output)

![](https://www.dropbox.com/s/kn7srfoeff0xoe3/game_fsm.png?dl=1)

> Example to debug REGFILE read addresses and output

![](https://www.dropbox.com/s/bnnl2f9fzf0hkre/regfile.png?dl=1)

### Build Project
Build the project by clicking on `Debug`.

Wait patiently... very patiently...

### Load Project
Once the build has completed, load the project onto the FPGA by clicking the `Program (Flash)` button.

![](https://www.dropbox.com/s/csgnh6t2gvcojw5/flash_icon.png?dl=1)

### Wait
Wait for the FPGA to start up.

### Open Wave Capture
Open the `Wave Capture` tab by going to `Tools > Wave Capture`.

![](https://www.dropbox.com/s/earfi7g2mm9ohkq/wave_capture?dl=1)

### Connect and Capture
Click on `Connect` (left button) followed by `Capture` (right button).

![](https://www.dropbox.com/s/kosb8d3pbzz2lbe/conn_capture.png?dl=1)

### View Signals
You should now be able to view the signals you selected earlier. Hover over the signal lines to view the values in decimal or expand the view to see each individual bit.

![](https://www.dropbox.com/s/4t9nlagqii40br3/wave_capture_signals.png?dl=1)

### Capture At Will
You are now able to capture the signals at any point in time during execution by clicking on the `Capture` button in the `Wave Capture` tab.


***Good luck debugging!***






