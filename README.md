# **CMOS Circuit Design and SPICE Simulations: Lecture Notes and Analysis**

# **Author**

Pavan Adithya B (Student, VLSI Design Course)

# **Introduction**
This report summarizes what I learnt from a CMOS circuit design workshop that used the open‑source SKY130 technology and ngspice for simulations. The main focus was to understand how an NMOS transistor works, how its drain current varies with terminal voltages, and why SPICE simulations are essential in modern VLSI design flows. I have tried to write this document in my own words, consolidating concepts from multiple lab sessions and notes.


# **1. Introduction to CMOS Circuit Design**

The lecture started with basics: how all standard cells (inverter, NAND, NOR, AND, OR, buffer, etc.) are built using only PMOS and NMOS transistors.

PMOS network (pull-up): connects output to Vdd when condition is met
NMOS network (pull-down): connects output to Vss when condition is met

The way we arrange series/parallel connections gives the logic function.

<img width="485" height="567" alt="Screenshot 2026-02-22 192912" src="https://github.com/user-attachments/assets/8529d152-4335-4d06-8446-f0d00500070c" />

This is the classic CMOS inverter diagram shown early in class:

* PMOS at top: source to Vdd, gate to Vin, drain to Vout
* NMOS at bottom: source to Vss, gate to Vin, drain to Vout
* Load capacitor CL attached at Vout (represents fanout + wire cap)

# **2. SPICE Simulations**

We simulate these circuits in SPICE to see real behavior.

Input a waveform at Vin → get output waveform at Vout.

For inverter, output is inverted version of input.

From the waveforms we measure propagation delay → time from 50% of input change to 50% of output change.

* Delay depends on:
  * How much current the transistors can provide (depends on W/L ratio)
  * Output load capacitance (CL)

Higher W/L → more current → faster switching → lower delay

We use SPICE to try different W/L and see how delay changes.

### **Why SPICE matters in real projects?**

In physical design (placement, routing, clock tree, crosstalk fix), we don’t run SPICE every time, but all timing numbers (delays, slacks) come from SPICE-characterized libraries.

If SPICE models are wrong → entire timing analysis is wrong → chip may fail.

<img width="1014" height="872" alt="Screenshot 2026-02-22 192956" src="https://github.com/user-attachments/assets/caa072cc-d07f-4554-b89c-26016bf42747" />

**Transistor Behavior in SPICE**

To understand delay at lowest level, we look at transistor curves.

* Upper plot
  * Ids vs Vout (or Vds) curves for PMOS and NMOS at different Vin values (0V, 0.5V, 1V, 1.5V, 2V)
  * Shows regions: cutoff (off), linear, saturation
  * Labels like “PMOS linear”, “NMOS sat”, etc.

* Bottom plot:
  * Vout vs Vin (DC transfer characteristic)
  * Inverted curve from ~Vdd to ~0V
  * Regions marked where PMOS/NMOS are in linear/sat/off during transition


These curves show why delay happens - during switching both transistors conduct for a short time, current fights, output moves slowly.

# **3. Delay Tables (Look-up Tables)**
In practice, we don’t run SPICE for every path. Instead, cell libraries have delay tables.

Delay table = 2D table

* Rows: input transition time (slew rate) in ps
* Columns: output load capacitance in fF
* Table entry = propagation delay in ps for that slew + load

If exact load not present (e.g. 60 fF between 50 and 70), we interpolate.

<img width="1264" height="449" alt="Screenshot 2026-02-22 193200" src="https://github.com/user-attachments/assets/02c7b129-0634-4cc0-bcb3-e5a860c11cf8" />

**Buffer tree example**

* Level 1 buffer driving two Level 2 buffers
* Each Level 2 driving two loads (C1, C2, C3, C4 = 25 fF each assumed)
* Buffer input caps assumed 30 fF each
* Total cap at node A ≈ 60 fF, at B and C ≈ 50 fF each
* Notes about using identical buffers at same level for balanced drive

This is why we size buffers and use trees - to control total capacitance and delay.


