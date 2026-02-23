# **CMOS Circuit Design and SPICE Simulations: Lecture Notes and Analysis**

# **Author**

Pavan Adithya B (Student, VLSI Design Course)

# **Introduction**
This report summarizes what I learnt from a CMOS circuit design workshop that used the open‑source SKY130 technology and ngspice for simulations. The main focus was to understand how an NMOS transistor works, how its drain current varies with terminal voltages, and why SPICE simulations are essential in modern VLSI design flows.

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

### **Some questions discussed in the lecture:**

**Where does the delay of a cell actually come from?**

* Delay tables come from SPICE simulations (characterization), not random numbers.

**We learned delay models/tables, but are they accurate?**

* Different cells (e.g. BUF1 vs BUF2) have different delays because of different transistor sizing (circuit design).
* To check accuracy → run SPICE with same input slew + same output load → measure delay from waveform → compare with table value.

**How do we verify if static timing analysis (STA) results are correct?**

* STA verification → compare STA delay/slack on critical paths with SPICE simulation of same path.



# **4. Introduction to basic element in Circuit design – NMOS**

<img width="568" height="534" alt="Screenshot 2026-02-22 213430" src="https://github.com/user-attachments/assets/c60cc264-6ba8-447f-893a-ef42d8ca2dff" />

**NMOS structure**

Built on P-type substrate

N-channel MOSFET (N-channel forms when turned on)

Main layers (from cross-section diagram):

* P-substrate (base, lightly doped P-type)
* Body terminal (B) → connected to substrate, usually grounded
* Isolation regions (left & right) → separate neighboring transistors
* n⁺ regions (heavily doped N-type):
  * Left → Source (S)
  * Right → Drain (D)
* Channel area → P-substrate between S and D
* Gate oxide → thin SiO₂ insulator over channel
* Gate → Poly-Si or metal on top of oxide → Gate terminal (G)

**Why this structure?**

* n⁺ S/D for low resistance
* Gate controls channel formation
* Isolation prevents transistor interference
* Thin gate oxide allows gate voltage to strongly influence channel

### **NOTE**
**PMOS Structure** : Opposite to NMOS, i.e.  N-substrate + p⁺ source/drain.


# **5. Threshold Voltage in NMOS**

**Threshold Voltage (VT):** Minimum gate-to-source voltage (VGS) needed to form a conducting channel between source (S) and drain (D).

Consider, 

**(i) Initial State (VGS = 0):**

* All terminals (G, S, D, B) grounded.
* P-substrate + n⁺ S/D form back-to-back PN junctions (like diodes).
* Junctions off → high resistance between S and D → no channel, no conduction.

**(ii) Apply Small Positive VGS (>0):**

<img width="1321" height="496" alt="image" src="https://github.com/user-attachments/assets/e646bc6d-6dfa-4939-84ad-09c6a75e8c71" />

**Accumulation**

* Gate positively charged (+ve on G).
* Gate-oxide-substrate acts as capacitor: +ve gate repels majority carriers (holes, +ve) in P-substrate downward.
* Leaves behind minority carriers (electrons, -ve) → accumulation of -ve charge near oxide interface.
* Starts forming depletion region.

(iii) Reaching Strong Inversion (Threshold Voltage - VT):

<img width="866" height="504" alt="image" src="https://github.com/user-attachments/assets/26c3e4f7-fd09-402e-9947-abc11db1ddb6" />

As VGS increases beyond small positive values:

* Depletion region widens (more majority carriers repelled).
* More negative charge (electrons) accumulates at the surface.
* At a specific VGS value → surface of P-substrate inverts from P-type to N-type (strong inversion).
* This forms a continuous N-type channel connecting source to drain.
* The VGS at which strong inversion occurs = Threshold Voltage (VT).

### **NOTE**

After inversion, further increase in VGS widens the channel (more electrons attracted), but depletion width stays almost constant (region already depleted).

<img width="528" height="459" alt="Screenshot 2026-02-22 221006" src="https://github.com/user-attachments/assets/727eee41-3347-4dfa-9bcf-c5435ccc5f66" />

Channel electrons come from heavily doped n⁺ source region (attracted like a magnet).


### **Body Effect on Threshold Voltage (VT):**

Normally VSB = 0 (source and body both grounded).

When VSB > 0 (source positive relative to body/substrate):
  * Additional reverse bias on source-body PN junction.
  * Increases depletion width around source (extra reverse bias widens depletion).
  * Makes inversion harder → requires higher VGS to achieve same inversion level → VT increases (body effect).

Same VGS increase shows delayed/less inversion when VSB > 0 compared to VSB = 0.

<img width="1296" height="603" alt="image" src="https://github.com/user-attachments/assets/d1002ce7-191d-4b3b-bdf5-d472ae6bb83d" />

Left: VSB = 0 → normal depletion width.

Right: VSB = +ve (below threshold voltage) → wider depletion under source.

Additionally,  

When increasing VGS in both scenarios (VSB=0 vs VSB>0):

Depletion width increases and negative charges (electrons) accumulate at the surface in both, expecting inversion.

**Key difference with VSB>0:** Positive potential on source attracts accumulated electrons back toward source.

This delays surface inversion → channel forms with a slant.

**At same VGS:** Inversion occurs at lower VGS for VSB=0 (full N-type surface at VTO).

For VSB>0: Needs extra VGS (VTO + V1) to overcome pull and achieve strong inversion.

**Conclusion:** Positive VSB requires additional gate potential for strong inversion.

<img width="1390" height="748" alt="image" src="https://github.com/user-attachments/assets/5554603c-7130-414b-897f-7c3e5e224a28" />


Threshold Voltage Equation:

<img width="607" height="232" alt="image" src="https://github.com/user-attachments/assets/6c66c4dd-7f56-4a52-89b3-b8b63ce3186a" />


Body Effect Coefficient (γ):

<img width="371" height="205" alt="image" src="https://github.com/user-attachments/assets/ba639b24-7be7-43bc-8158-d988d3134726" />


Fermi Potential (Φ_f):

<img width="461" height="122" alt="image" src="https://github.com/user-attachments/assets/f2e96acc-7c51-4c69-bfbd-4141b4a51b4e" />


### **What happens when we apply voltage to drain (VDS) ?**

Consider,

When VGS > VT (with small VDS):

Increasing VGS increases the number of induced electrons in the channel → channel becomes wider/more conductive.

For instance: At VGS = VT → minimal channel.

At VGS = 1 V → wider channel.

At VGS = 1.5 V or 2 V → even wider/more charge carriers.

**Example Conditions:**

VGS = 1 V

VDS = 0.05 V (small positive voltage)

VT = 0.45 V

Since, VGS > VT → channel is on.

Small VDS applied to drain → current can flow from S to D through the channel.

<img width="975" height="687" alt="image" src="https://github.com/user-attachments/assets/26fdeaa5-383a-4ea5-9b71-5a9cd639d4d1" />

In the figure, NMOS with VGS = 1 V on gate, VDS = 0.05 V on drain, VT = 0.45 V.

Voltage V(x) along channel from x=0 (source end ≈ 0 V) to x=L (drain end ≈ VDS).

**Observation** - Voltage Along Channel:

* Without VDS → voltage constant across channel (≈ VGS – VT effective).
* With small VDS → voltage varies along channel length (x from 0 to L).
* In between: V(x) increases gradually from 0 to VDS → voltage gradient exists.
* Effective gate-to-channel voltage at any point = VGS – V(x).

### **Note:**

Actual drawn channel length > effective length L (due to fabrication effects like lateral diffusion).

### **Charge & Drift Current Derivation**

Induced Charge in Channel (Q(x)):

In presence of VDS, induced electron charge density varies along channel.

Charge at any point x proportional to effective gate-to-channel voltage minus threshold:

<img width="386" height="193" alt="image" src="https://github.com/user-attachments/assets/85e112ee-fc76-4522-9d8b-6ad2d2cac756" />

**Q(x) = –Cox × (VGS – V(x) – VT)** (Negative sign because electrons are negative carriers)

* Cox = oxide capacitance per unit area = ε_ox / t_ox
* ε_ox = 3.97 × ε₀ (relative permittivity of oxide film)
* t_ox = gate oxide thickness (technology constant from foundry)

We know that there are two current types in MOSFET: **drift and diffusion**

* Drift current: Due to electric field (potential difference) → dominant here.

* Diffusion current: Due to carrier concentration gradient → small here.

**For small VDS → drift dominates → current flows from source to drain.**

### **Drift current ID = charge density × velocity × area**

<img width="855" height="780" alt="image" src="https://github.com/user-attachments/assets/bdbaf970-7bea-4a07-9298-dd528202ac81" />

Top view: Gate overlaps source/drain, channel is rectangle of length L and width W.


More precisely:

ID (Drift currrent) = (induced charge per unit length) × (carrier velocity) × W

Velocity of carriers proportional to electric field → function of dV/dx along channel.

Induced charge per unit length = W × Q(x) = W × Cox × (VGS – V(x) – VT)

To find total ID, integrate along channel (from x=0 to x=L).


![WhatsApp Image 2026-02-23 at 10 51 45 AM](https://github.com/user-attachments/assets/b0120b9a-fb1e-40b5-9191-58400a9602a4)

This is quadratic in VDS

**ID = μ_n × Cox × (W/L) × [(VGS - VT) × VDS - (VDS² / 2)]**

Simplified Linear Region Approximation (Small VDS):

![WhatsApp Image 2026-02-23 at 10 58 25 AM](https://github.com/user-attachments/assets/98374948-753e-454a-8b69-134f7806b409)


When VDS is very small i.e.  VDS << (VGS - VT), neglect the VDS²/2 term.

Approximate ID ≈ μ_n × Cox × (W/L) × (VGS - VT) × VDS

**Linear relationship between ID and VDS → acts like a resistor (hence "linear/resistive" region).**


**Example scenarios:**

Consider, **VT = 0.45 V**

<img width="486" height="262" alt="image" src="https://github.com/user-attachments/assets/80f00dc4-c768-438c-8eef-e4d07e59116c" />


**Sweep VDS from 0 to max for linear operation**

**Question:** What happens when VDS exceeds VGS - VT? 
* Enters saturation.

### **Saturation Region (Pinch-Off Phenomenon)**

<img width="666" height="661" alt="image" src="https://github.com/user-attachments/assets/8c6c9302-5004-4d08-95c2-a9ed4ddc2844" />

* As VDS increases while keeping VGS fixed, the effective gate-to-channel voltage (VGS – VDS) decreases near the drain.
* When VGS – VDS ≤ VT → surface inversion condition fails near drain → channel begins to disappear (pinch-off starts), as shown in the figures.

<img width="633" height="573" alt="Screenshot 2026-02-23 221520" src="https://github.com/user-attachments/assets/88c53997-ad59-41c3-9d85-b41846e20894" />

<img width="870" height="683" alt="image" src="https://github.com/user-attachments/assets/6646cf86-d0a7-49dc-9e73-e43cbc9800ec" />


* Channel remains present near source (where VGS – V(x) > VT) but vanishes near drain.



Full saturation: VDS ≥ VGS – VT → channel pinched off near drain, no inversion there.

<img width="1464" height="735" alt="image" src="https://github.com/user-attachments/assets/ab448ee1-bcad-45cf-8162-452c798fc74c" />


Condition for Pinch-Off / Saturation Entry:
* Pinch-off begins when VGS – VDS ≤ VT



**Comparision between linear and saturation regions**

<img width="441" height="224" alt="image" src="https://github.com/user-attachments/assets/fce6e7e5-e272-490c-9401-eb6cb22974eb" />


### **NOTE:**

Even after pinch-off, current still flows because, electrons drift through pinched region due to high electric field near drain.







































