## 📝 SPICE Deck Creation for CMOS Inverter
The creation of a SPICE deck begins with understanding the fundamental circuit, which involves identifying key aspects of the schematic.
The primary requirements for building the deck are:
* Component connectivity: How each component is connected to others.
* Component values: The specific parameters for each component, like transistor dimensions (W/L) or voltage levels.
* Identifying 'nodes': Labeling all the unique connection points in the circuit.

For the CMOS inverter schematic, the main nodes were identified as `in` (the input), `out` (the output), `vdd` (the positive supply), and `0` (ground, or vss).

The SPICE deck is a text-based representation of this schematic. The core of the deck is the **NETLIST Description**.

The PMOS transistor (M1) was defined with the line:
`M1 out in vdd vdd pmos W=0.375u L=0.25u`
* **M1:** The name of the component.
* **out in vdd vdd:** The connection points for the Drain, Gate, Source, and Bulk terminals, respectively. The PMOS source and bulk are both tied to `vdd`.
* **pmos:** The name of the model to be used for this transistor.
* **W=0.375u L=0.25u:** The component values, specifying the Width and Length of the transistor.

The NMOS transistor (M2) was defined with the line:
`M2 out in 0 0 nmos W=0.375u L=0.25u`
* **M2:** The name of the component.
* **out in 0 0:** The connection points for the Drain, Gate, Source, and Bulk terminals. The NMOS source and bulk are both tied to ground (`0`).
* **nmos:** The name of the model to be used.
* **W=0.375u L=0.25u:** The component values for the NMOS.

---

## 💻 SPICE Simulation Lab for CMOS Inverter
For a complete simulation, the SPICE deck was expanded beyond just the netlist.

**MODEL Descriptions** are pointed to, which contain the complex parameters for the `pmos` and `nmos` models.

The **NETLIST Description** was completed by adding all circuit elements:
* The transistors `M1` (pmos) and `M2` (nmos) were defined as before.
* A load capacitor `cload out 0 10f` was added between the `out` node and ground (`0`) with a value of 10 femtoFarads.
* The supply voltage `Vdd vdd 0 2.5` was defined as a 2.5V source between the `vdd` node and ground (`0`).
* The input voltage `Vin in 0 2.5` was also defined as a 2.5V source, which will be varied by the simulation command.

**SIMULATION Commands** were included to control the simulation:
* `.op`: A command to find the DC operating point of the circuit.
* `.dc Vin 0 2.5 0.05`: This is the main DC sweep command. It instructs SPICE to vary the input voltage (`Vin`) from 0V to 2.5V in steps of 0.05V.

**Model/Library Inclusion** is critical:
* `.include tsmc_025um_model.mod`: This line includes the file containing the specific transistor model parameters.
* `.LIB "tsmc_025um_model.mod" CMOS_MODELS`: This line serves a similar purpose, linking the `CMOS_MODELS` library.

The file is concluded with the `.end` command.

This SPICE deck (e.g., `cmosVTC_PMOSWidth_NMOSWidth.sp`) was then run in a SPICE simulator.

The result of this DC sweep simulation is the **Voltage Transfer Characteristic (VTC) plot**. This graph plots the output voltage (`out` on the y-axis) as a function of the input voltage (`in` on the x-axis), showing the characteristic S-shaped curve of an inverter.

---

## 📈 Switching Threshold Vm
The **Switching Threshold (Vm)** is a critical static parameter for a CMOS inverter.

* It is defined as the point on the VTC where the input voltage equals the output voltage: **Vin = Vout**.
* Graphically, this point is found at the intersection of the inverter's VTC curve (the green S-curve) and a line drawn where `y = x` (the blue diagonal line).
* During the Vm transition, both the PMOS and NMOS transistors are simultaneously in the **saturation region**. The switching threshold is the point where the PMOS drain current (Idsp) is equal to the NMOS drain current (Idsn).

The value of Vm is highly dependent on the relative strengths (W/L ratios) of the PMOS and NMOS transistors.

Two different simulations were examined to understand this:
1.  **Symmetric Inverter:** In the first case, `Wn=Wp=0.375u` and `Ln=Lp=0.25u`. This gives a ratio of `(Wn/Ln) = (Wp/Lp) = 1.5`. The resulting VTC curve was nearly symmetrical, and the switching threshold was found to be **Vm ~ 0.98v**.
2.  **Skewed Inverter:** In the second case, the PMOS width was increased: `Wn=0.375u`, `Wp=0.9375u`. This gives `(Wn/Ln)=1.5` and `(Wp/Lp)=3.75`. Because the PMOS (pull-up) transistor is now stronger than the NMOS (pull-down) transistor, it "wins" the tug-of-war for a longer time. This shifts the transition point to the right, resulting in a higher switching threshold of **Vm ~ 1.2v**.

The operating regions of the transistors were also analyzed across the VTC sweep:
* **Vin (low, 0V):** PMOS is 'linear', NMOS is 'off'. Vout is high (2.5V).
* **Vin (increasing):** PMOS is 'linear', NMOS is 'sat'. Vout begins to drop.
* **Vin = Vm:** Both PMOS and NMOS are in 'sat'.
* **Vin (increasing further):** PMOS is 'sat', NMOS is 'linear'. Vout drops sharply.
* **Vin (high, 2.5V):** PMOS is 'off', NMOS is 'linear'. Vout is low (0V).

---

## 🔬 Static and Dynamic Simulation of CMOS Inverter
This topic covers both the static evaluation of the inverter's robustness and its dynamic (transient) performance.

### Static Behavior Evaluation: Robustness and Vm
The robustness of an inverter is evaluated by its **Switching Threshold (Vm)**.
A mathematical formula for Vm was introduced, which is derived from equating the saturation currents of the NMOS and PMOS transistors.

The switching threshold Vm can be calculated with the formula:
> $Vm = \frac{R \cdot Vdd}{1+R}$

The factor `R` represents the ratio of the transistor strengths and is defined as:
> $R = \frac{Kp \cdot Vdsatp}{Kn \cdot Vdsatn}$

This can be expanded to:
> $R = \frac{(\frac{Wp}{Lp}) \cdot Kp' \cdot Vdsatp}{(\frac{Wn}{Ln}) \cdot Kn' \cdot Vdsatn}$

This shows Vm's direct dependency on the W/L ratios. A more complex formula was also presented, which accounts for the threshold voltages (Vt).

A table was shown to organize different simulation experiments, where the `Wp/Lp` ratio was held constant while the NMOS ratio (`Wn/Ln`) was scaled by a factor `x` (from 1 to 5).

For one such simulation (`Wn/Ln=1.5`, `Wp/Lp=1.5`), the results from the static VTC plot were:
* Vm = 0.99v
* Rise delay = 148ps
* Fall delay = 71ps
(Note: The delays are dynamic properties but were listed alongside the static Vm result).

### Dynamic Behavior Evaluation: Transient Simulation
A dynamic (or transient) simulation was performed to observe the inverter's behavior over time.
This requires a time-varying input signal. A **PULSE** input (`in`, red waveform) was defined for the `Vin` source.
The pulse parameters were:
* Goes from 0V to 2.5V.
* Rise time: 10ps
* Fall time: 10ps
* Period: 2ns

The simulation plots the input (`in`) and output (`out`, green waveform) versus time (in nanoseconds).
The output waveform was observed to be the inverse of the input, as expected.
Crucially, the output does not switch instantaneously. A **propagation delay** is clearly visible, where the output's rising or falling edge occurs slightly after the corresponding input edge. This delay is what the **Rise delay (148ps)** and **Fall delay (71ps)** values quantify.

---

## 🛠️ Create Active Regions
This is the beginning of the 16-mask CMOS fabrication process.

### Step 1: Selecting a Substrate
The entire process starts with a bare silicon wafer.
* A **P-type substrate** was selected.
* This substrate has high resistivity (around 5-50 ohms).
* The doping level is low, approximately $10^{15} \text{ cm}^{-3}$. This low doping is important because the 'well' doping in later steps will be higher.
* The silicon crystal orientation is (100).

### Step 2: Creating Active Regions for Transistors (Mask1)
This step defines where the transistors will be built, separating them from the insulating "field oxide."
First, a stack of layers is deposited on the P-substrate:
1.  A thin **pad oxide** layer: ~40nm of SiO₂
2.  A **nitride** layer: ~80nm of Si₃N₄ (Silicon Nitride)
3.  A **photoresist** layer: ~1μm of photoresist

**Photolithography** is then performed using **Mask1**. The mask patterns the photoresist.
* The wafer is exposed to UV light. The light passes through the clear parts of Mask1, chemically altering the photoresist below.
* The wafer is placed in a developing solution, which washes away the light-exposed photoresist.
* The patterned photoresist now acts as a mask. The exposed Si₃N₄ is etched away.
* The remaining photoresist is chemically removed, leaving a patterned Si₃N₄ layer.

The wafer is then placed in an oxidation furnace.
* The Si₃N₄ layer acts as an "oxidation mask." Silicon not covered by Si₃N₄ is oxidized, growing a thick layer of SiO₂. This thick oxide is the **Field Oxide**.
* This entire process is known as **LOCOS (Local Oxidation of Silicon)**.
* A side effect called the "**Bird's beak**" was noted, where the field oxide tapers off and grows slightly under the edge of the nitride mask.

Finally, the Si₃N₄ mask layer is stripped away, typically using hot phosphoric acid. The process now has defined "active regions" (bare silicon) surrounded by thick field oxide.

---

## 🌊 Formation of N-well and P-well

### Step 3: N-Well and P-Well Formation
This step creates the doped regions (wells) in the P-substrate where the NMOS and PMOS transistors will be built.
The process starts with the wafer from Step 2, which has active regions defined by field oxide.

**P-Well Formation:**
1.  A layer of photoresist is applied.
2.  **Mask2** is used, which has an opening over the area destined to become the P-well (the area for NMOS).
3.  The wafer is exposed to UV light and developed, leaving a photoresist mask.
4.  **Ion implantation** is performed using **Boron** (a P-type dopant) at an energy of approximately ~200keV. This dopes the exposed active region, creating the P-well.
5.  The photoresist (Mask2) is then stripped.

**N-Well Formation:**
1.  A new layer of photoresist is applied.
2.  **Mask3** is used, which has an opening over the area for the N-well (the area for PMOS).
3.  The wafer is exposed and developed.
4.  **Ion implantation** is performed using **Phosphorous** (an N-type dopant) at a higher energy, approximately ~400keV. This creates the N-well.
5.  The photoresist (Mask3) is stripped.

**Drive-in:**
* After both implants are complete, the wafer is placed in a high-temperature furnace. This step, often called "drive-in," activates the implanted dopants and causes them to diffuse deeper to form the final well profiles.

---

## GATE Formation of Gate Terminal

### Step 4: Formation of 'gate'
This step builds the central component of the transistor: the gate stack.
First, the concept of **Threshold Voltage (Vt)** was explained. This is the voltage needed at the gate (Vgs) to create an inversion channel and turn the transistor on. An equation for Vt was provided, showing its dependence on the zero-bias threshold voltage (Vto), the body effect coefficient (gamma), and the Fermi potential (phi\_f).

**Vt Adjustment Implants:**
* To fine-tune the transistor's threshold voltages, special implants are used before the gate is built.
* **Mask4** and photoresist are used to open a window over the P-well. **Boron** is implanted at ~60keV to adjust the Vt of the future NMOS device.
* The resist is stripped. **Mask5** and new photoresist are used to open a window over the N-well. **Arsenic** is implanted to adjust the Vt of the future PMOS device.

**Gate Stack Formation:**
1.  After the Vt-adjust implants, the existing thin oxide (pad oxide) is etched/stripped using a dilute hydrofluoric (HF) solution.
2.  A new, extremely clean, high-quality **gate oxide** is re-grown on the active silicon. This oxide is very thin, around ~10nm.
3.  A layer of **polysilicon** (the red layer) is deposited over the entire wafer. This will become the gate electrode.
4.  The polysilicon layer itself is then doped using N-type ion implants (phosphorous or arsenic). This is done to reduce the resistance of the poly gate.
5.  **Mask6** and photoresist are used to create a pattern for the gates.
6.  The wafer undergoes an etch process, which removes all polysilicon except where it was protected by the resist. This leaves behind the final polysilicon gate structures.
7.  The remaining photoresist is stripped.

---

## 💧 Lightly Doped Drain (LDD) Formation

### Step 5: Lightly Doped Drain (LDD) Formation
This step is a crucial technique used in modern transistors to improve reliability.
The two main reasons for LDDs were explained:
1.  **Hot Electron Effect:** In short-channel devices, the electric field near the drain (E=V/d) becomes very high. This accelerates electrons ("hot electrons"), which can gain enough energy (overcome the 3.2eV barrier) to get injected into the gate oxide, damaging it and altering the transistor's Vt over time.
2.  **Short Channel Effect:** The high drain field can also "penetrate" the channel, reducing the gate's control over the channel.

LDDs work by creating a more graded, lightly doped region (N- or P-) between the channel and the heavily doped (N+ or P+) drain. This spreads the voltage drop, reducing the peak electric field.

**LDD Implantation Process:**
* **Mask7** and photoresist are used to cover the PMOS side (P-well) of the wafer.
* A light **Phosphorous** (N-type) ion implant is performed. The poly gate acts as a "self-aligning" mask. This creates the **N- implant regions** (the LDDs) for the NMOS transistor.
* The resist is stripped.
* A new mask (implied **Mask8**) and photoresist are used to cover the NMOS side (N-well).
* A light **Boron** (P-type) ion implant is performed, creating the **P- implant regions** for the PMOS transistor.
* The resist is stripped.

**Sidewall Spacer Formation:**
1.  A layer of insulating material (e.g., ~0.1μm Si₃N₄ or SiO₂) is deposited over the entire wafer (the green layer).
2.  A **plasma anisotropic etching** process is performed. This is a highly directional vertical etch. It removes the material from all flat, horizontal surfaces but leaves it "stuck" to the vertical sidewalls of the polysilicon gates.
3.  These remaining vertical features are the **sidewall spacers**. They are critical for the next step.

---

## ➕ Source – Drain Formation

### Step 6: Source and Drain Formation
This step creates the heavily doped source and drain contact regions. The LDD regions from the previous step will be "under" the spacers.
* A thin **screen oxide** is grown over the wafer. This is to prevent the high-energy implants from "channeling," or traveling too deep into the crystal.

**NMOS Source/Drain (N+) Implant:**
1.  **Mask9** and photoresist are used to completely cover the PMOS device (P-well).
2.  A heavy **Arsenic** (N-type) ion implant is performed.
3.  The implant is blocked by the poly gate and the sidewall spacers. This is a self-aligned process. The **N+ region** forms in the exposed S/D area, while the N- LDD region is protected by the spacer, creating a perfect junction.
4.  The resist is stripped.

**PMOS Source/Drain (P+) Implant:**
1.  **Mask10** and photoresist are used to cover the NMOS device (N-well).
2.  A heavy **Boron** (P-type) ion implant is performed.
3.  This creates the **P+ source and drain** regions for the PMOS.
4.  The resist is stripped.

At this point, the fundamental transistor structures (Source, Drain, Gate, and Wells) are fully formed.

---

## 🔗 Local Interconnect Formation

### Step 7: Steps to form contacts and interconnects (local)
This step, also known as "**salicidation**," is designed to reduce the resistance of the contacts.

**HF Etch:**
* First, the wafer is cleaned, and the thin screen oxide is etched away from the tops of the source, drain, and gate regions using a dilute HF solution. This exposes the bare silicon and polysilicon.

**Sputtering:**
* **Titanium (Ti)** is deposited over the entire wafer surface.
* The method used is **sputtering**, a physical vapor deposition process. In this process, high-energy Argon (Ar+) gas ions are shot at a Titanium target, physically knocking Ti atoms off, which then coat the wafer.

**Annealing (Heating):**
1.  The wafer is heated (annealed) in a Nitrogen (N₂) ambient at about 650-700°C for 60 seconds.
2.  This heat causes a chemical reaction only where the Ti is touching silicon (the S/D regions) or polysilicon (the gate).
3.  The result is the formation of low-resistant **Titanium Silicide (TiSi₂) ** on these contacts.
4.  The unreacted Titanium over the oxide areas (like the spacers and field oxide) simultaneously reacts with the nitrogen ambient to form **Titanium Nitride (TiN)**.

This TiN layer can also be used for local communication paths.
The unreacted Ti and TiN can be selectively etched away, leaving the stable TiSi₂ "silicide" contacts. This is a "self-aligned" process, hence "**salicide**."

---

## ⛓️ Higher Level Metal Formation

### Step 8: Higher Level Metal Formation
This is the final, multi-step process to build the "wiring" (interconnects) that connects all the transistors together.

**Inter-Layer Dielectric (ILD) Deposition:**
* A thick (~1μm) layer of SiO₂ is deposited over the entire wafer. This is often **BPSG** (borophosphosilicate glass), which flows at high temperatures to help planarize the surface.
* The surface is made perfectly flat using **Chemical Mechanical Polishing (CMP)**. This is a crucial step for building multiple metal layers.

**Contact (Plug) Formation:**
1.  **Mask12** and photoresist are used to define the "contact holes" (vias).
2.  The BPSG layer is etched, opening holes down to the TiSi₂/TiN contact points.
3.  A thin (~10nm) TiN layer is deposited. This acts as a barrier and "glue" layer.
4.  A blanket **tungsten (W)** layer is deposited, filling all the contact holes.
5.  A second CMP step is performed. This polishes the wafer flat again, removing all excess tungsten and leaving only the W as contacts (plugs) in the holes.

**Metal 1 (M1) Formation:**
1.  A layer of **Aluminum (Al)** is deposited over the wafer.
2.  **Mask13** and photoresist are used to pattern the desired wiring paths for Metal 1.
3.  The Aluminum is plasma etched, removing all Al except for the patterned "wires."
4.  The resist is stripped.

**Higher Metal Layers (M2, M3, ...):**
1.  Another thick layer of SiO₂ (passivation) is deposited and planarized using CMP.
2.  **Mask14** is used to define the via holes that will connect Metal 1 to the next layer, Metal 2.
3.  The TiN/W/CMP plug-filling process is repeated.
4.  A new Aluminum layer (Metal 2) is deposited.
5.  **Mask15** is used to pattern the Metal 2 layer.
6.  This process (deposit oxide, CMP, pattern vias, deposit metal, pattern metal) is repeated for all required metal layers.

**Final Pad Formation:**
* A final passivation layer is deposited.
* The **Final Mask16** is used to open contact holes on this top layer, exposing the pads for the S (Source), G (Gate), and D (Drain) terminals.

This completes the fabrication, and the chip is ready for probing and packaging.
