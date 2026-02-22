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

# **2. Why do we care about SPICE?**

* In physical design / STA we talk about delays, clock trees, crosstalk, OCV, etc. — but none of this makes sense without knowing where the cell delay actually comes from.
* **Answer:** Cell delays come from SPICE simulations of the actual transistor-level circuit.
* **Without SPICE →** no accurate delay numbers → no meaningful STA, no clock tree synthesis, no useful physical design flow.

# **3. Delay Tables – How are they created?**

We don’t run SPICE for every path in the chip. Instead, we characterize standard cells once → create delay tables (also called .lib timing arcs).

**Typical delay table format:**

* Rows: Input transition time (slew) – e.g., 20 ps, 40 ps, 80 ps, …
* Columns: Output load capacitance – e.g., 10 fF, 30 fF, 50 fF, 70 fF, …
* Table entries: Delay value (usually in ps) for that slew + load combination


