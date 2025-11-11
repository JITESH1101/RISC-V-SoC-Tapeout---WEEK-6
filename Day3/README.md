## 📝 SPICE Deck Creation for CMOS Inverter
The creation of a SPICE deck begins with understanding the fundamental circuit, which involves identifying key aspects of the schematic.
The primary requirements for building the deck are:
* Component connectivity: How each component is connected to others.
* Component values: The specific parameters for each component, like transistor dimensions (W/L) or voltage levels.
* Identifying 'nodes': Labeling all the unique connection points in the circuit.

For the CMOS inverter schematic, the main nodes were identified as `in` (the input), `out` (the output), `vdd` (the positive supply), and `0` (ground, or vss).

The SPICE deck is a text-based representation of this schematic. The core of the deck is the **NETLIST Description**.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-35-04" src="https://github.com/user-attachments/assets/4623de38-904d-4901-94f7-6fd840b5666e" />

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
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-35-44" src="https://github.com/user-attachments/assets/a608ef30-9f93-481e-80b8-71b846f82962" />


The **NETLIST Description** was completed by adding all circuit elements:
* The transistors `M1` (pmos) and `M2` (nmos) were defined as before.
* A load capacitor `cload out 0 10f` was added between the `out` node and ground (`0`) with a value of 10 femtoFarads.
* The supply voltage `Vdd vdd 0 2.5` was defined as a 2.5V source between the `vdd` node and ground (`0`).
* The input voltage `Vin in 0 2.5` was also defined as a 2.5V source, which will be varied by the simulation command.

**SIMULATION Commands** were included to control the simulation:
* `.op`: A command to find the DC operating point of the circuit.
* `.dc Vin 0 2.5 0.05`: This is the main DC sweep command. It instructs SPICE to vary the input voltage (`Vin`) from 0V to 2.5V in steps of 0.05V.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-36-24" src="https://github.com/user-attachments/assets/0abf1571-25b4-4806-af0c-572b9c070ab9" />

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

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-37-37" src="https://github.com/user-attachments/assets/324b286a-fde5-4070-899b-479fdb27e45a" />

The operating regions of the transistors were also analyzed across the VTC sweep:
* **Vin (low, 0V):** PMOS is 'linear', NMOS is 'off'. Vout is high (2.5V).
* **Vin (increasing):** PMOS is 'linear', NMOS is 'sat'. Vout begins to drop.
* **Vin = Vm:** Both PMOS and NMOS are in 'sat'.
* **Vin (increasing further):** PMOS is 'sat', NMOS is 'linear'. Vout drops sharply.
* **Vin (high, 2.5V):** PMOS is 'off', NMOS is 'linear'. Vout is low (0V).

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-39-21" src="https://github.com/user-attachments/assets/48740c64-656b-49a6-90b6-62962fdfe81f" />


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
  
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-48-03" src="https://github.com/user-attachments/assets/3b469a23-898b-4f3d-be5d-7f550b3d0350" />

### Step 2: Creating Active Regions for Transistors (Mask1)
This step defines where the transistors will be built, separating them from the insulating "field oxide."
First, a stack of layers is deposited on the P-substrate:
1.  A thin **pad oxide** layer: ~40nm of SiO₂
2.  A **nitride** layer: ~80nm of Si₃N₄ (Silicon Nitride)
3.  A **photoresist** layer: ~1μm of photoresist

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-49-03" src="https://github.com/user-attachments/assets/c9920039-c71e-4008-81b7-5b59622a6ebc" />

**Photolithography** is then performed using **Mask1**. The mask patterns the photoresist.
* The wafer is exposed to UV light. The light passes through the clear parts of Mask1, chemically altering the photoresist below.
* The wafer is placed in a developing solution, which washes away the light-exposed photoresist.
* The patterned photoresist now acts as a mask. The exposed Si₃N₄ is etched away.
* The remaining photoresist is chemically removed, leaving a patterned Si₃N₄ layer.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-49-22" src="https://github.com/user-attachments/assets/e77e841a-f527-4bac-8d73-e2fed0d3b246" />

The wafer is then placed in an oxidation furnace.
* The Si₃N₄ layer acts as an "oxidation mask." Silicon not covered by Si₃N₄ is oxidized, growing a thick layer of SiO₂. This thick oxide is the **Field Oxide**.
* This entire process is known as **LOCOS (Local Oxidation of Silicon)**.
* A side effect called the "**Bird's beak**" was noted, where the field oxide tapers off and grows slightly under the edge of the nitride mask.
  
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-50-36" src="https://github.com/user-attachments/assets/e31adc3d-4cfe-4f97-957e-f517e31ce3df" />

Finally, the Si₃N₄ mask layer is stripped away, typically using hot phosphoric acid. The process now has defined "active regions" (bare silicon) surrounded by thick field oxide.

---

## 🌊 Formation of N-well and P-well

### Step 3: N-Well and P-Well Formation
This step creates the doped regions (wells) in the P-substrate where the NMOS and PMOS transistors will be built.
The process starts with the wafer from Step 2, which has active regions defined by field oxide.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-51-35" src="https://github.com/user-attachments/assets/13c6342a-b165-404f-b167-50ea21e01967" />

**P-Well Formation:**
1.  A layer of photoresist is applied.
2.  **Mask2** is used, which has an opening over the area destined to become the P-well (the area for NMOS).
3.  The wafer is exposed to UV light and developed, leaving a photoresist mask.
4.  **Ion implantation** is performed using **Boron** (a P-type dopant) at an energy of approximately ~200keV. This dopes the exposed active region, creating the P-well.
5.  The photoresist (Mask2) is then stripped.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-51-55" src="https://github.com/user-attachments/assets/679c16d4-048e-4e3f-bba8-890e3d4ca2ac" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-52-10" src="https://github.com/user-attachments/assets/b401b2c5-03a6-4485-a323-2eabbc43ee40" />

**N-Well Formation:**
1.  A new layer of photoresist is applied.
2.  **Mask3** is used, which has an opening over the area for the N-well (the area for PMOS).
3.  The wafer is exposed and developed.
4.  **Ion implantation** is performed using **Phosphorous** (an N-type dopant) at a higher energy, approximately ~400keV. This creates the N-well.
5.  The photoresist (Mask3) is stripped.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-52-31" src="https://github.com/user-attachments/assets/84734a70-f344-417a-b88e-6c267d20b883" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-52-43" src="https://github.com/user-attachments/assets/a658f074-8f9a-4e6d-994d-1326b1f5018d" />


**Drive-in:**
* After both implants are complete, the wafer is placed in a high-temperature furnace. This step, often called "drive-in," activates the implanted dopants and causes them to diffuse deeper to form the final well profiles.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-52-57" src="https://github.com/user-attachments/assets/7c7e3444-74e5-4860-a488-c611215941a5" />

---

## GATE Formation of Gate Terminal

### Step 4: Formation of 'gate'
This step builds the central component of the transistor: the gate stack.
First, the concept of **Threshold Voltage (Vt)** was explained. This is the voltage needed at the gate (Vgs) to create an inversion channel and turn the transistor on. An equation for Vt was provided, showing its dependence on the zero-bias threshold voltage (Vto), the body effect coefficient (gamma), and the Fermi potential (phi\_f).

**Vt Adjustment Implants:**
* To fine-tune the transistor's threshold voltages, special implants are used before the gate is built.
* **Mask4** and photoresist are used to open a window over the P-well. **Boron** is implanted at ~60keV to adjust the Vt of the future NMOS device.
* The resist is stripped. **Mask5** and new photoresist are used to open a window over the N-well. **Arsenic** is implanted to adjust the Vt of the future PMOS device.
  
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-53-51" src="https://github.com/user-attachments/assets/ed011056-7730-456b-a71f-a941c682cd0a" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-54-05" src="https://github.com/user-attachments/assets/6896485a-8735-4c8b-82cc-82ee253760f9" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-54-12" src="https://github.com/user-attachments/assets/76fc93e1-0fbc-44e6-99cb-60017a889a75" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-54-15" src="https://github.com/user-attachments/assets/e2d81cbd-6531-45a7-a1ff-55af0ed5cdf1" />

**Gate Stack Formation:**
1.  After the Vt-adjust implants, the existing thin oxide (pad oxide) is etched/stripped using a dilute hydrofluoric (HF) solution.
2.  A new, extremely clean, high-quality **gate oxide** is re-grown on the active silicon. This oxide is very thin, around ~10nm.
3.  A layer of **polysilicon** (the red layer) is deposited over the entire wafer. This will become the gate electrode.
4.  The polysilicon layer itself is then doped using N-type ion implants (phosphorous or arsenic). This is done to reduce the resistance of the poly gate.
5.  **Mask6** and photoresist are used to create a pattern for the gates.
6.  The wafer undergoes an etch process, which removes all polysilicon except where it was protected by the resist. This leaves behind the final polysilicon gate structures.
7.  The remaining photoresist is stripped.
   
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-54-45" src="https://github.com/user-attachments/assets/216009eb-a91c-4fa9-b88a-a22e72ccf816" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-54-54" src="https://github.com/user-attachments/assets/3e3e4ad4-361f-42bc-97ee-83933e8f6f0f" />

---

## 💧 Lightly Doped Drain (LDD) Formation

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-55-57" src="https://github.com/user-attachments/assets/0724186a-054a-4a33-904c-ab34a514f345" />

### Step 5: Lightly Doped Drain (LDD) Formation
This step is a crucial technique used in modern transistors to improve reliability.
The two main reasons for LDDs were explained:
1.  **Hot Electron Effect:** In short-channel devices, the electric field near the drain (E=V/d) becomes very high. This accelerates electrons ("hot electrons"), which can gain enough energy (overcome the 3.2eV barrier) to get injected into the gate oxide, damaging it and altering the transistor's Vt over time.
2.  **Short Channel Effect:** The high drain field can also "penetrate" the channel, reducing the gate's control over the channel.

LDDs work by creating a more graded, lightly doped region (N- or P-) between the channel and the heavily doped (N+ or P+) drain. This spreads the voltage drop, reducing the peak electric field.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-56-38" src="https://github.com/user-attachments/assets/bfab2d9f-f7d8-41ac-a2fa-061492f3716c" />

**LDD Implantation Process:**
* **Mask7** and photoresist are used to cover the PMOS side (P-well) of the wafer.
* A light **Phosphorous** (N-type) ion implant is performed. The poly gate acts as a "self-aligning" mask. This creates the **N- implant regions** (the LDDs) for the NMOS transistor.
* The resist is stripped.
* A new mask (implied **Mask8**) and photoresist are used to cover the NMOS side (N-well).
* A light **Boron** (P-type) ion implant is performed, creating the **P- implant regions** for the PMOS transistor.
* The resist is stripped.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-56-50" src="https://github.com/user-attachments/assets/76b6de9c-e436-490a-a407-f837af6cd9de" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-56-58" src="https://github.com/user-attachments/assets/e36e7422-0e7a-441e-92cc-7a7b122a65b0" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-57-22" src="https://github.com/user-attachments/assets/0537c47e-6d48-49fb-a613-5e3855935c81" />

**Sidewall Spacer Formation:**
1.  A layer of insulating material (e.g., ~0.1μm Si₃N₄ or SiO₂) is deposited over the entire wafer (the green layer).
2.  A **plasma anisotropic etching** process is performed. This is a highly directional vertical etch. It removes the material from all flat, horizontal surfaces but leaves it "stuck" to the vertical sidewalls of the polysilicon gates.
3.  These remaining vertical features are the **sidewall spacers**. They are critical for the next step.

---

## ➕ Source – Drain Formation

### Step 6: Source and Drain Formation
This step creates the heavily doped source and drain contact regions. The LDD regions from the previous step will be "under" the spacers.
* A thin **screen oxide** is grown over the wafer. This is to prevent the high-energy implants from "channeling," or traveling too deep into the crystal.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-57-55" src="https://github.com/user-attachments/assets/de7456db-8fb4-476a-80ce-77938399ab12" />

**NMOS Source/Drain (N+) Implant:**
1.  **Mask9** and photoresist are used to completely cover the PMOS device (P-well).
2.  A heavy **Arsenic** (N-type) ion implant is performed.
3.  The implant is blocked by the poly gate and the sidewall spacers. This is a self-aligned process. The **N+ region** forms in the exposed S/D area, while the N- LDD region is protected by the spacer, creating a perfect junction.
4.  The resist is stripped.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-58-10" src="https://github.com/user-attachments/assets/8bb2433f-12ad-4f71-b145-de4b24c57bae" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-58-26" src="https://github.com/user-attachments/assets/b280b569-8e75-411c-a335-f6625a5deb06" />

**PMOS Source/Drain (P+) Implant:**
1.  **Mask10** and photoresist are used to cover the NMOS device (N-well).
2.  A heavy **Boron** (P-type) ion implant is performed.
3.  This creates the **P+ source and drain** regions for the PMOS.
4.  The resist is stripped.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-58-32" src="https://github.com/user-attachments/assets/c96550b9-5b7f-4cce-90ab-86fc22c0449c" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 19-58-49" src="https://github.com/user-attachments/assets/cad5439f-3ff4-48eb-8927-a73671c3f7ff" />


At this point, the fundamental transistor structures (Source, Drain, Gate, and Wells) are fully formed.

---

## 🔗 Local Interconnect Formation

### Step 7: Steps to form contacts and interconnects (local)
This step, also known as "**salicidation**," is designed to reduce the resistance of the contacts.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 21-54-41" src="https://github.com/user-attachments/assets/d1848f50-bc5a-43bb-8d87-40650a7b05bd" />

**HF Etch:**
* First, the wafer is cleaned, and the thin screen oxide is etched away from the tops of the source, drain, and gate regions using a dilute HF solution. This exposes the bare silicon and polysilicon.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 21-55-33" src="https://github.com/user-attachments/assets/e5b7bc96-aff1-494e-8649-f84699dfd59b" />

**Sputtering:**
* **Titanium (Ti)** is deposited over the entire wafer surface.
* The method used is **sputtering**, a physical vapor deposition process. In this process, high-energy Argon (Ar+) gas ions are shot at a Titanium target, physically knocking Ti atoms off, which then coat the wafer.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 21-56-00" src="https://github.com/user-attachments/assets/f7606721-0f71-4c93-b8ff-25031b022e9b" />



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

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-00-51" src="https://github.com/user-attachments/assets/ce2a0a63-c65a-497b-a9af-ed2d7ed93274" />

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
   
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-01-35" src="https://github.com/user-attachments/assets/3ec01206-0a1a-4ff7-93ce-bc018603e628" />


**Higher Metal Layers (M2, M3, ...):**
1.  Another thick layer of SiO₂ (passivation) is deposited and planarized using CMP.
2.  **Mask14** is used to define the via holes that will connect Metal 1 to the next layer, Metal 2.
3.  The TiN/W/CMP plug-filling process is repeated.
4.  A new Aluminum layer (Metal 2) is deposited.
5.  **Mask15** is used to pattern the Metal 2 layer.
6.  This process (deposit oxide, CMP, pattern vias, deposit metal, pattern metal) is repeated for all required metal layers.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-02-00" src="https://github.com/user-attachments/assets/5e0342e9-3f76-4a75-b368-dcffe640df50" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-02-24" src="https://github.com/user-attachments/assets/34c89845-abbc-4c2b-b0bc-8d871e514175" />

**Final Pad Formation:**
* A final passivation layer is deposited.
* The **Final Mask16** is used to open contact holes on this top layer, exposing the pads for the S (Source), G (Gate), and D (Drain) terminals.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-02-37" src="https://github.com/user-attachments/assets/5f3a300b-5524-4c38-b0ad-415beee57d36" />


This completes the fabrication, and the chip is ready for probing and packaging.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-02-42" src="https://github.com/user-attachments/assets/e779d833-78a8-4496-875f-afb4bacc0cac" />

## Lab Task

-  Clone the github repository and enter the following commands to open the  custom inverter standard cell design 

```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign

# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

<img width="1198" height="538" alt="vsdstdcelldesign_techfile_magic" src="https://github.com/user-attachments/assets/62c4538e-0571-4832-b462-3c79db4ee205" />

The loaded design in magic tool is as follows

<img width="1280" height="768" alt="magic_sky130A_inv" src="https://github.com/user-attachments/assets/1630322e-fea3-499c-8b43-548cb2206097" />

- Spice extraction of inverter in magic tool

Commands for spice extraction of the custom inverter layout to be used in tkcon window of magic

```
# Check current directory
pwd

# Extraction command to extract to .ext format
extract all

# Before converting ext to spice this command enable the parasitic extraction also
ext2spice cthresh 0 rthresh 0

# Converting to ext to spice
ext2spice
```

<img width="845" height="214" alt="parasite_extract_files" src="https://github.com/user-attachments/assets/88401e90-fa86-4125-acd7-50f949d4f122" />


<img width="980" height="339" alt="extract_parasites" src="https://github.com/user-attachments/assets/ff0b7cc1-57fd-4de9-a8d9-faf35d56f374" />

the following is the extracted spice file

<img width="1001" height="371" alt="extracted_spice_File" src="https://github.com/user-attachments/assets/8eefcf9a-29f5-4274-b602-b490b9156bac" />

- Ngspice Simulation

  Enter the following command to run ngpsice simulation and also to get the plot of y vs time a

```
ngspice sky130_inv.spice
plot y vs time a
```
<img width="1280" height="768" alt="nspice_sky130AINV SPICE" src="https://github.com/user-attachments/assets/7ee07130-5e5c-433f-8488-640903a45d1d" />

<img width="1216" height="743" alt="plot_y vs time a" src="https://github.com/user-attachments/assets/d904ff25-565b-42fb-8a00-fe41106c72be" />

- RISE TRANSITION TIME AND FALL TRANSITION TIME CALCULATIONS

  For Rise calculations,

  At 20%,

  <img width="1398" height="771" alt="rise_20%_waveform" src="https://github.com/user-attachments/assets/9fe24853-e083-4cbd-8e3f-eef6d74ba25e" />

  At 80%,

  <img width="1417" height="780" alt="rise_80%_Waveform" src="https://github.com/user-attachments/assets/9171922f-40ca-4897-8852-8edf183fc6a3" />

  <img width="1060" height="591" alt="rise_transition_time" src="https://github.com/user-attachments/assets/1aacb12f-ccf6-4ad3-aaa6-fffe13dd0f77" />

   Rise   transition   time = Time   taken   for   output   to   rise   to   80% − Time   taken   for   output   to   rise   to   20%
                  20 %   of   output = 660 m V
                  80 %   of   output = 2.64 V 

   Fall Transition Time calculations

   At 20%,

   <img width="1387" height="763" alt="fall_20%_waveform" src="https://github.com/user-attachments/assets/b2b3bf81-f98f-412b-b5c4-a2c5d425585b" />

  At 80%,

   <img width="1394" height="761" alt="fall_80%_waveform" src="https://github.com/user-attachments/assets/503908ef-a4df-4723-b62d-601cf19ea812" />

   <img width="962" height="710" alt="fall_transition_time" src="https://github.com/user-attachments/assets/612d331d-49d1-4b03-ae65-34a3f4c394cd" />

 R i s e   t r a n s i t i o n   t i m e = 2.24638 − 2.18242 = 0.06396   n s = 63.96   p s

Fall transition time calculation
F a l l   t r a n s i t i o n   t i m e = T i m e   t a k e n   f o r   o u t p u t   t o   f a l l   t o   20 % − T i m e   t a k e n   f o r   o u t p u t   t o   f a l l   t o   80 %
20 %   o f   o u t p u t = 660   m V
80 %   o f   o u t p u t = 2.64   V 

   Now ,at 50%,

   For Rise ,

   <img width="1365" height="759" alt="rise_50%_waveform" src="https://github.com/user-attachments/assets/2ce13aff-4761-43d7-ac9a-65d8710ce40b" />

   <img width="1389" height="726" alt="rise_Calc_50%" src="https://github.com/user-attachments/assets/209c9ca4-9b41-4c84-82d4-6b095c817234" />

 R i s e   C e l l   D e l a y = 2.21144 − 2.15008 = 0.06136   n s = 61.36   p s

Fall Cell Delay Calculation
F a l l   C e l l   D e l a y = T i m e   t a k e n   f o r   o u t p u t   t o   f a l l   t o   50 % − T i m e   t a k e n   f o r   i n p u t   t o   r i s e   t o   50 %
50 %   o f   3.3   V = 1.65   V 

   For fall,

   <img width="1366" height="757" alt="fall_50%_waveform" src="https://github.com/user-attachments/assets/ffcdba5d-8e18-4852-a027-10e2d56483b3" />

   <img width="960" height="715" alt="fall_calc_50%" src="https://github.com/user-attachments/assets/c595c1d1-60b7-4834-b6ba-8dbd2af3360d" />

  F a l l   C e l l   D e l a y = 4.07 − 4.05 = 0.02   n s = 20   p s 


 ## DRC rules and check

 Go to home directory and enter the following commands to download and verify the drc checks

 ```
 # Change to home directory
cd

# Command to download the lab files
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz

# Since lab file is compressed command to extract it
tar xfz drc_tests.tgz

# Change directory into the lab folder
cd drc_tests

# List all files and directories present in the current directory
ls -al

# Command to view .magicrc file
gvim .magicrc

# Command to open magic tool in better graphics
magic -d XR &
```

<img width="843" height="214" alt="wget_Tests_drc" src="https://github.com/user-attachments/assets/ea244044-9c61-487f-bd58-3eda75b18a52" />

the following is the picture of .magicrc file

<img width="1280" height="768" alt="magicrc" src="https://github.com/user-attachments/assets/23d1478a-45c6-492b-96db-9836c038f672" />

Now open the magic tool by command ``magic -d XR &`` open met3 file

<img width="1280" height="768" alt="met3" src="https://github.com/user-attachments/assets/89b88878-68e6-4ea7-baf6-3fc5f23fd16b" />

Now to create via 2 enter the following commands in the console below

<img width="760" height="322" alt="MET3_POLY_CONSOLE" src="https://github.com/user-attachments/assets/4a960cba-fb8e-4b27-a29a-2d3a7d4f8066" />

then we get to see the via formation

<img width="956" height="594" alt="MET3_POLY" src="https://github.com/user-attachments/assets/4af24c98-4d50-41db-b0b5-0799f80eee84" />

- Poly rules

  <img width="1048" height="1144" alt="poly_rules" src="https://github.com/user-attachments/assets/b5c45fc8-421f-4b9a-9471-e9fda7650f73" />

Incorrectly implemented poly.9 , no drc violation

<img width="1216" height="743" alt="load_poly" src="https://github.com/user-attachments/assets/497eb2bd-8893-4dca-adab-34f0f6185daf" />

while this is running , make the following changes to the ``sky130A.tech`` file 

<img width="918" height="440" alt="sky130A_tech_change1" src="https://github.com/user-attachments/assets/dc2a1557-2458-4c11-bf94-7658339735e1" />

<img width="750" height="218" alt="sky130A_tech_change2" src="https://github.com/user-attachments/assets/0f0be5d7-58b8-4265-9a76-03aa5bfe69f3" />

Now to load these changes , enter the following command

```
# Loading updated tech file
tech load sky130A.tech

# Must re-run drc check to see updated drc errors
drc check

# Selecting region displaying the new errors and getting the error messages 
drc why
```

then we get to see that it is correctly implemented

<img width="1405" height="806" alt="magic_window with rule implemented" src="https://github.com/user-attachments/assets/9f749eef-4126-483b-bcb3-b1cce3acbba5" />

<img width="931" height="346" alt="Screenshot 2025-11-11 191918" src="https://github.com/user-attachments/assets/b0d1200c-6f04-42db-9051-ed5562d8e4de" />


- Incorrectly implemented difftap.2 simple rule correction

  DIFFTAP rules

  <img width="1396" height="705" alt="difftap_rules" src="https://github.com/user-attachments/assets/e9f8d229-e392-4b4c-aeed-4b36a136627b" />

  the following is the incorrect implementation

  <img width="1407" height="782" alt="difftap 2_rule" src="https://github.com/user-attachments/assets/8c057bd3-66ba-4010-9e29-66e2dae049f6" />

  the following is the changes made to the tech file in drc

  <img width="1370" height="610" alt="drc_update_sky130A_Tech" src="https://github.com/user-attachments/assets/a3ad9d01-d702-4751-8321-c5915e756066" />

  Now it is correctly implemented by loading the techfile again as done above

  <img width="1403" height="724" alt="difftap_2_rule_implemented" src="https://github.com/user-attachments/assets/83c16ef7-f032-4677-b949-62a20c43522a" />

  - Incorrectly implemented nwell.4 complex rule correction

  NWELL RULES

  <img width="1410" height="740" alt="nwell2_rules" src="https://github.com/user-attachments/assets/3b20dc3c-528a-4bf0-8541-16038d7da83e" />

  The incorrectly implemented one is as follows

  <img width="1403" height="761" alt="nwell2_error" src="https://github.com/user-attachments/assets/f62f5851-ebf2-4f9d-9076-283588bbe252" />

    the change in tech file for drc is as follows

  <img width="1053" height="626" alt="drc_update_nwell2" src="https://github.com/user-attachments/assets/23b3e5b3-c06e-4400-8e88-af2c1859b4e4" />

  <img width="1251" height="656" alt="drc_update_nwell2 1" src="https://github.com/user-attachments/assets/a7a70588-9222-4d23-9565-6b211c24d4fa" />

  Now by loading the tech file again as done above we get the correct implementation for the observed drc

  <img width="1408" height="747" alt="nwell_Rule_implemented" src="https://github.com/user-attachments/assets/69b6b4d9-1ad6-4383-a364-7be23648de5b" />
























    
