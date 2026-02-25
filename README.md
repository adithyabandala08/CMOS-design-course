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


### **Drift current calculation for the saturation region**

**Saturation Current Equation (Ideal Model)**

<img width="1519" height="778" alt="image" src="https://github.com/user-attachments/assets/ec3253ba-4625-4ffd-959b-b962f90170cc" />

Replace VDS with (VGS – VT) in the linear equation

**ID = (kn / 2) × (VGS – VT)²**

Where kn = k_n' × (W/L) = μ_n × Cox × (W/L)

ID now appears independent of VDS → looks like perfect constant current source

**Channel Length Modulation (Real-World Effect)**

* Increasing VDS widens depletion region at drain → effective channel length (L_eff) shortens slightly
* This causes small increase in ID with VDS (not perfectly constant)

So, more accurate equation:

**ID = (kn / 2) × (VGS – VT)² × [1 + λ VDS]**

λ = channel length modulation parameter

### **NOTE:**

Reducing physical L should increase ID (per kn ∝ 1/L), but in short-channel devices → velocity saturation causes opposite behavior (ID decreases).

### **Equations involved upto now (recap):**

<img width="554" height="613" alt="image" src="https://github.com/user-attachments/assets/5e2dbf96-08ec-43a6-b19e-11b71f1d9322" />


# **6. SPICE SETUP**

<img width="770" height="828" alt="image" src="https://github.com/user-attachments/assets/347b148d-6bc4-4033-af1f-09132560e568" />


* SPICE is a simulation engine with built-in MOSFET models (threshold voltage, linear & saturation current equations derived earlier).
* Feed correct inputs + netlist → SPICE generates accurate waveforms → used for cell delay calculation.


**Model Parameters (Technology Constants)**
* Fixed values from equations: VTO, γ (body effect), Φ_f, μ_n, Cox, etc.
* Unique for each technology node (e.g., 180 nm, 20 nm).


Simplified Netlist Naming
* Vin → gate input voltage
* R1 → protection resistor
* M1 → MOSFET instance
* VDS → drain supply
* VSS → source/substrate ground

**Why SPICE??**
* Automates sweeps (VGS, VDS) → no manual calculation for every combination.
* Provides accurate ID, voltage waveforms using real models.

### **SPICE Netlist**

* First step: Define nodes (points with no obstruction between components → same potential).
* Components connect between nodes (e.g., resistor between two nodes, MOSFET between four nodes).

Example Circuit Values

<img width="752" height="556" alt="image" src="https://github.com/user-attachments/assets/4ab83cc5-eb9e-473d-a206-845d812424c1" />

* VDD = 2.5 V
* Vin (gate voltage) = variable (e.g., sweep)
* Protection resistor R1 = 55 Ω
* W = 1.8 μm, L = 1.2 μm (long channel to start)
* VSS = 0 V (ground)

Node Definitions (4 Nodes)

<img width="614" height="434" alt="image" src="https://github.com/user-attachments/assets/f51c92dd-7786-41c8-ad85-759b3a9813ff" />


* Node "in" → gate input side (Vin to R1)
* Node "n1" → between R1 and gate of MOSFET
* Node "vdd" → drain supply (VDD to drain)
* Node "0" → ground (source, substrate, VSS)

**SPICE Netlist Lines**

<img width="1385" height="474" alt="image" src="https://github.com/user-attachments/assets/669aa66a-e1ff-4866-84c9-70cbe534709e" />

**MOSFET (M1)**
  * M1 vdd n1 0 0 nmos W=1.8u L=1.2u
    * Order: drain gate source substrate (DGSS)
    * "nmos" = model name from technology file

**Resistor (R1)**
  * R1 in n1 55
    * Between nodes "in" and "n1", value 55 Ω

**Voltage Sources**
* Vdd vdd 0 2.5
  * Positive terminal vdd, negative 0, value 2.5 V

* Vin in 0 [value/sweep]
  * Positive in, negative 0


### **Technology File & Model Definition**

* Model file contains all constants (VTO, γ, μ_n, Cox, λ, etc.) for NMOS/PMOS.
* Defined using **.model** statement:
  * **.model** nmos nmos VTO=... GAMMA=... U0=... COX=...
  * **Note:** Name "nmos" must exactly match the name used in netlist (M1 ... nmos) and same for PMOS (e.g., .model pmos pmos ...).

<img width="634" height="473" alt="image" src="https://github.com/user-attachments/assets/4aa558df-7ded-4867-b860-44a6fa557cb7" />

* Parameters are technology-specific (different for 1.2 μm, 350 nm, 250 nm, etc.).

**Including Model File in Netlist**

* Use .lib to include external model file:
  * **.lib 'cmos_models.lib' CMOS_MODELS**
  * File name (cmos_models.lib) + section name (CMOS_MODELS).
**.include** or **.lib** brings in all model parameters → SPICE evaluates VT, ID equations using these values.

**Netlist Structure Sections**

* Netlist description → components (M1, R1, Vdd, Vin) and connections.
* Model inclusion → .lib statement to load technology file.
* Simulation commands → control what to sweep/measure.

**Example SPICE netlist:**

<img width="508" height="461" alt="image" src="https://github.com/user-attachments/assets/727f4f68-6327-4072-abde-b475effe8dff" />

**Running & Plotting**

<img width="1217" height="782" alt="image" src="https://github.com/user-attachments/assets/b475c6b6-1801-401c-92af-034b2917ba6c" />

**Behavior observed:**
* Low VDS → linear region (ID increases linearly with VDS).
* Higher VDS → quadratic/saturation region (curves bend, ID becomes nearly constant).

**Another example:** 

<img width="564" height="611" alt="image" src="https://github.com/user-attachments/assets/91c19a68-bbc8-41c0-b537-55a4156b285e" />


**Running & Plotting**

<img width="1171" height="740" alt="Screenshot 2026-02-24 183746" src="https://github.com/user-attachments/assets/f44bdb42-6bc8-4ed4-8a87-00ad5e8ac0d1" />

<img width="1124" height="677" alt="Screenshot 2026-02-24 183823" src="https://github.com/user-attachments/assets/5905e3b0-6682-412f-ab21-8ce0d2509d69" />



## **SPICE simulation for lower nodes**


**Objective**
* Simulate and compare long-channel (L = 1.2 µm) vs short-channel (L ≈ 0.15–0.25 µm) NMOS using Sky130 PDK to observe short-channel effects.

* Velocity Saturation → Id-Vgs becomes linear at high Vgs (instead of quadratic).
* Idsat ≈ W × Cox × (Vgs – Vt) × v_sat (velocity-limited, L-independent).
* Reduced Drive Current → Same W/L → short-channel Id is ~50% lower.

Example: W/L = 1.5
* Long: ~410 µA peak
* Short: ~210 µA peak

* Earlier Saturation → Vdsat smaller → flattens at lower Vds.
* Channel Length Modulation (λ) → Slight Id increase with Vds in saturation.
* Equation: Id = (kn/2) × (Vgs – Vt)² × (1 + λ Vds)
* Vt Roll-off → Slight reduction due to DIBL (short-channel effect).

Waveform Observations

<img width="2177" height="1339" alt="image" src="https://github.com/user-attachments/assets/121c5770-259f-4feb-82f4-0f663be77356" />

* Id-Vds Family Curves
* Linear region (low Vds): Id rises linearly
* Transition at Vds ≈ Vgs – Vt (pinch-off)
* Saturation: Id flattens (short-channel earlier & lower peak)
* Slight upward slope due to λ


## **Id–Vgs Behavior: Long-Channel vs Short-Channel MOSFETs**

Id–Vgs (drain current vs gate-to-source voltage) plots reveal how MOSFET drive current changes with gate bias. Long-channel and short-channel devices behave very differently due to velocity saturation and other short-channel effects.

<img width="2420" height="940" alt="image" src="https://github.com/user-attachments/assets/1fac94b3-1359-4a96-a6b6-e3cb31bcf493" />


**SPICE Setup**

* Model: .include sky130.lib.spice tt (typical corner)
* Sweep: .dc Vgs 0 1.8 0.05 Vds 1.8 (fixed Vds in saturation, sweep Vgs)
* MOSFET: M1 drain gate source bulk nfet01 W=__u L=__u
* Use pre-characterized W/L from cell library.

* **Long-Channel Behavior** (e.g., L = 1.2 µm, W/L = 1.5)

Follows classical square-law model.

**In saturation:**

**Id = (kn / 2) × (Vgs – Vt)² × (1 + λ Vds)**

* Id–Vgs plot: Quadratic rise above threshold (parabolic shape).
* Current increases rapidly with Vgs (strong gate control).
* Peak Id higher (e.g., ~410 µA at Vgs = 1.8 V, Vds = 1.8 V).

<img width="1376" height="1104" alt="image" src="https://github.com/user-attachments/assets/3dbf4333-b2d6-402c-9fb8-c7d0a32c3545" />


* **Short-Channel Behavior** (e.g., Sky130, L ≈ 0.15–0.25 µm, W/L = 1.5)

  * Velocity saturation dominates at high lateral fields.
  * Carrier velocity caps at v_sat → Id no longer follows square-law.

<img width="1377" height="1098" alt="image" src="https://github.com/user-attachments/assets/f09c96da-bbb7-4939-953f-634d18bd9d62" />

    
 **In saturation:**
 
  * Idsat ≈ W × Cox × (Vgs – Vt) × v_sat
  * Id–Vgs plot: Linear rise at high Vgs (straight line instead of parabola).
  * Peak Id significantly lower (e.g., ~210 µA at same Vgs/Vds → ~50% reduction).
  * Earlier onset of saturation-like behavior even in moderate Vgs.


<img width="2352" height="927" alt="image" src="https://github.com/user-attachments/assets/0b09a4b4-57f7-4813-a970-faa887761718" />



## **Velocity Saturation at Lower vs Higher Electric Fields (Short-Channel MOSFETs)**

* Velocity saturation is a key short-channel effect that limits carrier velocity in high electric fields, causing deviation from long-channel square-law behavior.

<img width="2400" height="954" alt="image" src="https://github.com/user-attachments/assets/b3ade936-5c01-48c3-99c6-0c13b7f2c5b9" />


<img width="1126" height="668" alt="image" src="https://github.com/user-attachments/assets/72242db4-9851-4cf3-8997-df9cfb3c3e68" />


At Lower Electric Fields (Long-Channel / Low Vds or Long L)

* Electric field E = Vds / L is low → carriers accelerate linearly.
* Drift velocity v = μ_n × E → increases with field.

Id follows classical model:

* Linear region: Id ∝ Vds
* Saturation: Id ∝ (Vgs – Vt)²

* Id–Vgs plot: Quadratic rise above Vt.
* No velocity limit → current scales well with gate overdrive.

At Higher Electric Fields (Short-Channel / High Vds or Short L)

* E = Vds / L becomes very high in scaled devices (short L).
* Carrier velocity saturates at v_sat (~10⁷ cm/s for electrons in silicon) due to phonon scattering.
* v = v_sat (constant) instead of μ_n × E.
* Id becomes velocity-limited:
* Idsat ≈ W × Cox × (Vgs – Vt) × v_sat
* Id–Vgs plot: Linear rise at high Vgs (loses quadratic dependence).
* Peak Id significantly lower than long-channel prediction for same W/L.

<img width="1000" height="235" alt="image" src="https://github.com/user-attachments/assets/3bd1d60d-cf69-411a-aa47-ded9ff01ce4a" />


**Modelling the velocity**

Boundary conditions:

* For electric field E ≤ Ec (critical field): Electron velocity vn increases linearly with electric field
* For electric field E ≥ Ec: Electron velocity reaches saturation velocity (vsat) and becomes approximately constant

<img width="804" height="152" alt="image" src="https://github.com/user-attachments/assets/370693bd-0bdb-418a-91fc-a5e50c0b6e42" />


**SPICE Observations**

* Long-channel (L = 1.2 µm): Quadratic Id–Vgs curve.
* Short-channel (L ≈ 0.15–0.25 µm): Transitions to linear Id–Vgs at high Vgs.
* Result: Reduced drive current (~50% lower peak Id), earlier saturation onset.
* Equation shift: From square-law → linear in high-field regime.


## **Velocity saturation drain current model**

* Vgt = Vgs – Vt, This is the gate overdrive voltage. We use Vgt when analyzing the device at high gate voltages (strong inversion region).
* Drain current equations are expressed using Vgt rather than Vgs.
* For small Vds, we usually neglect the channel length modulation term (λVds) to simplify analysis.

* Vdsat — a process/technology parameter
  * It is the drain-to-source voltage where velocity saturation starts.
  * When Vds exceeds Vdsat, carrier velocity reaches its maximum and stops increasing with electric field.
  * This causes the drain current to deviate from the classical quadratic (long-channel) behavior.

<img width="1725" height="627" alt="image" src="https://github.com/user-attachments/assets/487f711a-e77b-4b1a-8f09-2b3b132cfb2b" />

<img width="1622" height="626" alt="image" src="https://github.com/user-attachments/assets/46de5e9d-a6d9-4008-a6df-2943d8ebed71" />


**Why Saturation Current is Smaller in Lower Technology Nodes??**

* In advanced (short-channel) nodes, saturation current is smaller than expected — not larger as simple scaling might suggest.
* Main reason: Velocity saturation effect becomes much stronger in short-channel transistors.
* Short channel length means even a modest Vds creates a very high electric field along the channel (E = Vds / L is large).
* Carriers quickly reach their maximum velocity (v_sat ≈ 10⁷ cm/s for electrons) due to phonon scattering.
* Once velocity saturates, it cannot increase further — drain current no longer grows proportionally with Vgs or Vds.
* Result: Peak saturation current is significantly lower compared to long-channel devices with the same W/L ratio.

<img width="1725" height="735" alt="image" src="https://github.com/user-attachments/assets/781908ea-5c2e-46b3-885c-982ad98d5de2" />


### **Drain Current Models**

* **Long-Channel (No Velocity Saturation)**
  * Low electric field → carrier velocity proportional to field (v = μ_n × E).
  * Saturation current: Id = (kn / 2) × Vgt² × (1 + λ Vds)
  * Quadratic dependence on Vgt.

* **Velocity Saturation (Short-Channel / High Field)**
  * High electric field → velocity capped at v_sat.
  * Saturation current becomes: Idsat ≈ W × Cox × Vgt × v_sat
  * Linear dependence on Vgt (no longer quadratic).
  * Independent of channel length L.


## **Labs Sky130 Id-Vgs**

In this SPICE simulation, a short-channel NMOS device with a channel length of around 0.12–0.15 µm is analysed at a supply voltage of 1.8 V. The drain voltage VDS is swept from 0 to 1.8 V with a step of 0.1 V, while the gate voltage VGS is swept in steps of 0.2 V to generate the ID–VDS characteristics.

<img width="2148" height="1512" alt="bcfha" src="https://github.com/user-attachments/assets/73bee55a-27c4-4150-9916-41000e2c1183" />


* For lower values of Vgs (just above Vt), the drain current shows a quadratic behavior with respect to (Vgs − Vt).
* For higher values of Vgs, the behavior becomes more linear, mainly due to velocity saturation effects in short-channel devices.
* Peak current is about 197uA


## **Labs Sky130 Vt**

* Open day2_nfet_idvgs_L015_W039.spice
* Run ngspice
* Plot Id current (vdd-branch)

<img width="989" height="795" alt="image" src="https://github.com/user-attachments/assets/99191f88-3692-4572-94d7-f4823899541b" />


* Threshold voltage (Vt) is obtained by drawing a tangent at the maximum slope of the Id–Vgs curve and extending it to intersect the Vgs axis
* Threshold voltage, Vt=0.737V



# **7.CMOS Voltage Transfer Characteristics (VTC)**

**MOSFET as a Switch**

* NMOS or PMOS transistor acts like a voltage-controlled switch.
* Condition to turn ON: |VGS| ≥ |Vt|
  * NMOS: VGS ≥ Vt (positive) → channel forms → low resistance (RON) between source-drain.
  * PMOS: VGS ≤ Vt (negative) → channel forms → low resistance.

* When |VGS| < |Vt| → transistor OFF → very high resistance (open circuit) → no current from source to drain.

<img width="1088" height="631" alt="image" src="https://github.com/user-attachments/assets/8969689c-4664-4e77-8251-4a5281e431cc" />

**CMOS Inverter Structure**

* Complementary MOS: PMOS (pull-up) at top + NMOS (pull-down) at bottom.

<img width="525" height="460" alt="image" src="https://github.com/user-attachments/assets/782e7f7c-5ef8-41b3-826c-3c3a8c623436" />

* Connections:
  * PMOS source → VDD
  * NMOS source → VSS (ground)
  * Gates of both tied together → input (Vin)
  * Drains of both tied together → output (Vout)
  * Load capacitance (CL) at Vout (includes next gate + wire cap).


**Case 1: Vin = VDD (Logic High)**

* NMOS: VGS = VDD – VSS = VDD (> Vt) → NMOS ON → acts as low resistance → Vout pulled to VSS (≈ 0 V) → Logic Low output.
* PMOS: VGS = VDD – VDD = 0 V (less than |Vt|) → PMOS OFF → open circuit from VDD.
* **Result:** Vout = 0 V (strong pull-down by NMOS).

 <img width="1553" height="861" alt="image" src="https://github.com/user-attachments/assets/716d64a4-706d-4302-9e95-19014eb5bf84" />


**Case 2: Vin = 0 V (Logic Low)**

* NMOS: VGS = 0 – VSS = 0 V (< Vt) → NMOS OFF → open circuit.
* PMOS: VGS = 0 – VDD = –VDD (< Vt) → PMOS ON → acts as low resistance → Vout pulled to VDD → Logic High output.
* Result: Vout = VDD (strong pull-up by PMOS).

<img width="1529" height="864" alt="image" src="https://github.com/user-attachments/assets/95b33a20-539c-4458-869f-c07334c54093" />

### **Note: Why Complementary??**
* When one transistor is ON, the other is OFF → no static DC path from VDD to ground → zero static power (ideally).

<img width="1495" height="788" alt="image" src="https://github.com/user-attachments/assets/cf6ea1e6-4b3e-4e57-9377-0b0233ea8105" />

* When Vin = Vdd:
* Direct discharging path: Vout → Rn → Vss
* CL discharges → Vout falls to 0 V

* When Vin = 0 V:
* Direct charging path: Vdd → Rp → Vout
* CL charges → Vout rises to Vdd


<img width="1511" height="780" alt="image" src="https://github.com/user-attachments/assets/2e421430-546d-4fd7-ac26-e2c8891723fb" />

* G = Gate (both PMOS & NMOS)
* S = Source (PMOS → Vdd, NMOS → Vss)
* D = Drain (both connected to Vout)


* **Vgs = Gate-to-Source voltage**
  * NMOS: VgsN = Vin – Vss
  * PMOS: VgsP = Vin – Vdd

* **Vds = Drain-to-Source voltage**
  * NMOS: VdsN = Vout – Vss
  * PMOS: VdsP = Vout – Vdd

* **Ids = Drain-to-Source current**
  * IdsN = current through NMOS (pull-down)
  * IdsP = current through PMOS (pull-up)

### **Load Curves & Current Directions**

* **NMOS Load Curve (Idsn vs Vdsn)**

  * Vgsn = Vin – Vss = Vin (since Vss = 0)
  * Vdsn = Vout – Vss = Vout
  * Plot: Idsn (positive) vs Vdsn (positive)
  * At Vgsn = 0 (< Vt) → Idsn ≈ 0 (NMOS off)
  * As Vgsn increases above Vt → Idsn rises
    * Low Vdsn → linear region
    * Higher Vdsn → saturation region (flattens)
  * Multiple curves for different Vgsn values (higher Vgsn → higher Idsn).

* **PMOS Load Curve (Idsp vs Vdsp)**

  * Vdsp = Vout – Vdd (negative when Vout < Vdd)
  * Vgsp = Vin – Vdd (negative when Vin < Vdd)
  * Idsp is negative of Idsn (opposite current direction).
  * Plot: Idsp (negative) vs Vdsp (negative) → mirrored & flipped quadrant.
  * When |Vgsp| > |Vt| (Vgsp more negative) → PMOS on → |Idsp| increases.
  * Shape similar to NMOS but inverted:
    * Low |Vdsp| → linear
    * Higher |Vdsp| → saturation

<img width="1563" height="871" alt="image" src="https://github.com/user-attachments/assets/3fa16733-d427-460f-b0eb-0a881cf7eb30" />

**Current Directions in CMOS Inverter**

**Vin = Vdd (High):**
* NMOS on → Idsn flows from Vout → Vss (downward, discharging CL) → Vout falls to 0 V.
* PMOS off → no current from Vdd.

**Vin = 0 V (Low):**
* PMOS on → Idsp flows from Vdd → Vout (upward, charging CL) → Vout rises to Vdd.
* NMOS off → no current to Vss.

* Idsn and Idsp are opposite: Idsp = –Idsn (directions cancel in steady state).

Current direction in CMOS Invertor: 

<img width="544" height="227" alt="image" src="https://github.com/user-attachments/assets/20e24ff3-9f5e-46ac-99c5-878a78223f33" />


### **CMOS Inverter – Load Curves Conversion for VTC**

**Objective**

* Convert NMOS & PMOS Id-Vds load curves into a form that depends only on Vin and Vout (no internal Vgsp, Vdsp, etc.) → used to derive full Voltage Transfer Characteristic (VTC) curve of CMOS inverter.

Key Observations from Previous Lectures

* NMOS Idsn-Vdsn curves: positive Idsn, Vgsn = Vin, Vdsn = Vout
* PMOS Idsp-Vdsp curves: negative Idsp, Vgsp = Vin – Vdd, Vdsp = Vout – Vdd
* Idsp = –Idsn (opposite direction due to current flow)
* Internal voltages (Vgsp, Vdsp) are not visible in digital design → only Vin and Vout matter.

Step-by-Step: Load Curves → VTC Ready

1. Vgsp = Vin – Vdd → Vin = Vgsp + Vdd
2. Idsp = –Idsn (flip direction)
3. Shift PMOS curve: Plot |Idsp| = Idsn at corresponding Vin


<img width="1553" height="854" alt="image" src="https://github.com/user-attachments/assets/192f6a30-97cc-4a92-a6ad-70fac9e0a357" />

4. Replace Vdsp with Vout → Vout = Vdsp + Vdd **(Load curve for PMOS)**

<img width="1549" height="513" alt="image" src="https://github.com/user-attachments/assets/094af2e6-3af0-4b3e-9094-d1df7cc43c7a" />

5. Do the same to obtain load curve for NMOS

<img width="558" height="502" alt="image" src="https://github.com/user-attachments/assets/6e9bf260-3f20-4ac2-b39a-1182fdf4e6c5" />

6. Create VTC graph:
* X-axis: Vin (0 to Vdd, e.g., 0 to 2 V)
* Y-axis: Vout (0 to Vdd)
* For each Vin value:
  * Find intersection point between NMOS and PMOS curves at that Vin.
  * Plot (Vin, Vout) on VTC graph.
 
<img width="1548" height="444" alt="image" src="https://github.com/user-attachments/assets/5fd0fc88-bf82-4255-87af-f0e0b06f36f6" />

### **VTC Regions (Vdd = 2 V)**

| Vin     | Vout       | NMOS State    | PMOS State    | Region                  |
|---------|------------|---------------|---------------|-------------------------|
| 0 V     | ≈ 2 V      | Cutoff        | Linear        | Logic High              |
| 0.5 V   | 1.5–2 V    | Saturation    | Linear        | High                    |
| 1.0 V   | 0.5–1.5 V  | Saturation    | Saturation    | High Gain / Transition  |
| 1.5 V   | 0–0.5 V    | Linear        | Saturation    | Low                     |
| 2.0 V   | ≈ 0 V      | Linear        | Cutoff        | Logic Low               |

7. Connect the points → get S-shaped VTC curve.


<img width="543" height="424" alt="image" src="https://github.com/user-attachments/assets/99a1d0ff-e8ea-497c-a17c-74b5605ad830" />


## **SPICE deck creation for CMOS inverter**

**SPICE Deck Components**

**Transistors:**
* PMOS: W = 0.375 µm, L = 0.25 µm (or scaled)
* NMOS: W = 0.375 µm, L = 0.25 µm (same size for simplicity; ideally PMOS 2–3× wider)
* Supply voltages: Vdd = 2.5 V (typical for 250 nm node)
* Input sweep: Vin from 0 to 2.5 V
* Model include: .include sky130.lib.spice tt (or equivalent technology file)

<img width="1531" height="844" alt="image" src="https://github.com/user-attachments/assets/65b00c44-4f93-48cb-8693-36856aa2c284" />

Node Naming (Important for SPICE)

Node 0: Vss / ground (common)
Node vdd: Vdd supply
Node in: Vin (input)
Node out: Vout (output)

**Typical SPICE Netlist Structure**


<img width="738" height="249" alt="image" src="https://github.com/user-attachments/assets/626c38bf-d46e-4059-a57c-3e5bab267299" />


<img width="531" height="361" alt="image" src="https://github.com/user-attachments/assets/473928cc-9438-4556-9f9f-e186659c08b5" />



**Case 1: Balanced W/L (Same for NMOS & PMOS)**
* Transistor sizes:
  * NMOS: W = 0.9 µm, L = 0.6 µm (W/L = 1.5)
  * PMOS: W = 0.9 µm, L = 0.6 µm (W/L = 1.5)

* Input sweep: Vin from 0 V to 2.5 V
* Step size: 0.05 V
* Purpose: Observe baseline VTC with equal W/L ratio for NMOS and PMOS.

<img width="821" height="788" alt="vtc_!" src="https://github.com/user-attachments/assets/dc211833-d1ea-421e-a70f-8e8acf85ab25" />


**Case 2: Realistic W/L (PMOS Wider for Balanced Drive)**
* Transistor sizes:
  * NMOS: W = 0.9 µm, L = 0.6 µm (W/L = 1.5)
  * PMOS: W = 3.0 µm, L = 0.6 µm (W/L = 5.0)

* Input sweep: Vin from 0 V to 2.5 V
* Step size: 0.05 V
* Purpose: Simulate more realistic inverter with PMOS wider than NMOS (due to lower hole mobility) → better rise/fall time symmetry.

<img width="806" height="790" alt="vtc_2" src="https://github.com/user-attachments/assets/e752b392-28ce-49ee-a036-5bd1793b87a5" />


### **Labs – Sky130 SPICE Simulation for CMOS Inverter**

**Objective**
* Perform DC and transient simulations in Sky130 PDK to obtain:

  * Voltage Transfer Characteristic (VTC) curve
  * Switching threshold voltage
  * Rise and fall delays

**DC Simulation – VTC & Switching Threshold**

* Used Sky130 library (typical corner)
* Inverter sizing: NMOS & PMOS with appropriate W/L ratios
* DC sweep: Vin from 0 to 1.8 V (or as configured)
* Result: VTC waveform (Vout vs Vin)
* Switching threshold (Vin = Vout point):
  * From previous lecture calculations: ≈ 1.07479 V
  * Observed around the midpoint of sharp transition region on VTC plot
 
<img width="830" height="783" alt="Vtc_3" src="https://github.com/user-attachments/assets/9a0747e2-5c6d-4ed7-8f69-fa0e69c3f146" />

**Transient Analysis – Rise & Fall Delay**

* Input: Pulse from 0 to 1.8 V
* Measured at 50% voltage point (Vdd/2 = 0.9 V)
* Rise delay:
  * Time from Vin reaching 0.9 V to Vout reaching 0.9 V (rising edge)
  * Calculated: 2.4833 ns – 2.1511 ns = 0.3321 ns
* Fall delay:
  * Time from Vin reaching 0.9 V to Vout reaching 0.9 V (falling edge)
  * Calculated: 4.3360 ns – 4.0495 ns = 0.2865 ns
* Propagation delay ≈ average of rise + fall delays

### **CMOS Inverter – VTC Analysis & Switching Threshold (Vm)**

1. VTC Waveform Comparison

* Case 1: Equal W/L for NMOS & PMOS (W = 0.375 µm, L = 0.25 µm, W/L = 1.5 for both)
* Case 2: PMOS wider (Wp = 0.9375 µm = 2.5 × Wn, Wn = 0.375 µm, L = 0.25 µm, W/L PMOS = 3.75)
* Result: VTC shape remains very similar in both cases — S-shaped transition from Vdd to 0 V.

2. Robustness of CMOS Inverter

* Regardless of PMOS/NMOS sizing ratio, the fundamental behavior stays the same:
* Vin = 0 → Vout = Vdd
* Vin = Vdd → Vout = 0

* CMOS inverter is extremely robust - core logic function is preserved across different sizes.
* That's why CMOS is the most widely used logic family.

<img width="1492" height="793" alt="image" src="https://github.com/user-attachments/assets/0ce9cd99-9d3d-4019-b35d-3f98b8cb31f4" />

3. Switching Threshold (Vm or Vth)

* Definition: Point where Vin = Vout (intersection of VTC with 45° line).
* Method: Draw Vin = Vout line on VTC plot → find intersection.
* Results:
  * Case 1 (equal W/L): Vm ≈ 0.9 V
  * Case 2 (PMOS 2.5× wider): Vm ≈ 1.2 V
* Vm shifts right when PMOS is wider (stronger pull-up → higher input needed to balance).

4. Critical Region – Both Transistors in Saturation

* Around Vm (transition region):
  * Both NMOS and PMOS are in saturation (fully turned ON).
  * Direct current path from Vdd → PMOS → NMOS → ground → short-circuit current flows.
  * High current leakage in this region (static power dissipation).
  * High gain (small ΔVin → large ΔVout) — good for analog, but power-hungry for digital.


<img width="1253" height="571" alt="image" src="https://github.com/user-attachments/assets/5f0fc143-1786-413d-95d2-16a4ea697fde" />


### **CMOS Inverter – Analytical Derivation of Switching Threshold (Vm)**

**Assumptions for Derivation**

* Long-channel model (velocity saturation region)
* Ignore channel length modulation (λ ≈ 0 → 1 + λVds ≈ 1)
* Vgt = Vgs – Vt (gate overdrive)
* At Vm: VgsN = Vm (NMOS), VgsP = Vm – Vdd (PMOS)
* VdsN = Vm (NMOS), VdsP = Vm – Vdd (PMOS)

**Drain Current Equations at Vm**

* NMOS (Idsn):
  * Idsn = kn × (VgsN – Vtn) × Vdsatn – (Vdsatn² / 2)
  * Substitute: VgsN = Vm, Vtn = Vt (NMOS threshold)
  * Idsn = kn × (Vm – Vtn) × Vdsatn – (Vdsatn² / 2)
 
<img width="998" height="284" alt="image" src="https://github.com/user-attachments/assets/309e579b-4088-4a23-ab24-dde3de44bb7b" />

* PMOS (Idsp):
  * Idsp = kp × (VgsP – |Vtp|) × Vdsatp – (Vdsatp² / 2)
  * Substitute: VgsP = Vm – Vdd, Vtp = |Vt| (PMOS threshold)
  * Idsp = kp × (Vm – Vdd – |Vtp|) × Vdsatp – (Vdsatp² / 2)
 
<img width="682" height="330" alt="image" src="https://github.com/user-attachments/assets/6b983b19-934a-4a16-ae8c-3d268e2713bf" />
 
* Condition: Idsn + Idsp = 0

**Solving for Vm**

* Set Idsn = –Idsp
* Substitute kn = μn Cox (W/L)n, kp = μp Cox (W/L)p
* After simplification (ignoring small terms):
* Vm = [constant × Vdd] / [1 + constant]
* where constant depends on kn/kp ratio (i.e., (W/L)n / (W/L)p, μn/μp, Vdsatn/Vdsatp, etc.)


<img width="1495" height="808" alt="image" src="https://github.com/user-attachments/assets/e0b15af1-a495-4f39-be9e-534d0b3478a2" />


### **Vm Definition**
* Vm = point where Vin = Vout on VTC (switching threshold).

At Vm, 

* Both NMOS & PMOS in saturation
* Idsn + Idsp = 0 (equal magnitude, opposite direction)

Drain Current at Vm, 

* Idsn = kn × (Vm – Vtn) × Vdsatn – (Vdsatn² / 2)
* Idsp = kp × (Vm – Vdd – |Vtp|) × Vdsatp – (Vdsatp² / 2)

Simplifications,

* Ignore λVds (≈ 0)
* VgsN = Vm, VdsN = Vm
* VgsP = Vm – Vdd, VdsP = Vm – Vdd

<img width="410" height="318" alt="image" src="https://github.com/user-attachments/assets/e9b418b3-c351-4ea8-a1bd-7d01ff380fa3" />

Solving for Vm, 

* Idsn + Idsp = 0 → Vm = [constant × Vdd] / [1 + constant]
* Constant ∝ kn/kp ratio = (μn/μp) × (W/L)n / (W/L)p
* Results (Examples)
* Wp/Wn = 1 → Vm ≈ 0.9 V
* Wp/Wn = 2.5 → Vm ≈ 1.2 V
* Wider PMOS → higher Vm

Reverse Design, 
* Set desired Vm (e.g., Vdd/2 = 1.25 V) → solve for required Wp/Wn ratio
* Applications: Clock inverters need Vm ≈ Vdd/2 → balanced rise/fall


<img width="732" height="425" alt="image" src="https://github.com/user-attachments/assets/63e0ebdc-a833-4cdf-883e-63a0a33179e5" />
































































































