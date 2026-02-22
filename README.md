**CMOS Circuit Design and SPICE Simulation using SKY130 Technology**


This report summarizes what I learnt from a CMOS circuit design workshop that used the open‑source SKY130 technology and ngspice for simulations. The main focus was to understand how an NMOS transistor works, how its drain current varies with terminal voltages, and why SPICE simulations are essential in modern VLSI design flows. I have tried to write this document in my own words, consolidating concepts from multiple lab sessions and notes.


1. **NMOS Device Structure and Basic Operation**

   1.1 **Physical Structure**

   An NMOS transistor is a four‑terminal device built on a p‑type substrate, with two n+ diffusion regions forming the source and drain, and a gate made of polysilicon or metal separated from the substrate by         a thin gate oxide. The four terminals are Gate (G), Source (S), Drain (D) and Body or Bulk (B). Isolation oxides are used at the sides to separate this device electrically from neighbours on the same chip.

   1.2 **Initial Condition (VGS = 0)**

   When all terminals are at 0 V (G, S, D and B tied to ground), the source–body and drain–body junctions behave as reverse‑biased p–n diodes, and no inversion channel is present under the gate. The                   resistance between source and drain is very high and the NMOS effectively behaves like an open switch with almost zero drain current, ID≈0.

