## 1. Introduction to QFN-48 Package, Chip, Pads, Core, Die, and IPs

The physical chip is housed within a Package. The specific package type is a QFN-48 (Quad Flat No-leads).
The physical dimensions of this square package are 7mm x 7mm.
The package's pinout consists of 48 connections, which include:
* **General Purpose I/O:** gpio0 through gpio15.
* **Power and Ground:** vdd3v3 (3.3V supply), vdd1v8 (1.8V supply), and vss (ground).
* **Flash Memory Interface:** flash_csb, flash_clk, sck, sdo, sdi, flash_io0, flash_io1, flash_io2, flash_io3.
* **Analog-to-Digital Converter (ADC) Pins:** adc0_in, adc1_in, vref_h (voltage reference high), vref_l (voltage reference low).
* **Clock Oscillator Pins:** XI and XO (crystal input/output).
* **Serial Interface:** ser_tx (serial transmit) and ser_rx (serial receive).
* **Other Pins:** analog_out, comp_inn, comp_inp, irq, and xclk.

The internal structure of the chip is defined by several constituent parts.
* The "**chip**" is the central component, with all internal logic connected to the external pads.
* The "**Die**" is the entire rectangular piece of silicon.
* The "**Core**" is the central area of the die where the primary logic is placed.
* The "**PADS**" are the connection points on the perimeter of the die, interfacing between the internal core logic and the external package pins.

The core itself is composed of multiple functional blocks, categorized as either Macros or Foundry IP's.
* **Foundry IP's** (Intellectual Property) are pre-designed, verified blocks from the fabrication facility (the foundry). These include:
    * SRAM (Static Random-Access Memory)
    * adc0 (Analog-to-Digital Converter 0)
    * adc1 (Analog-to-Digital Converter 1)
    * dac (Digital-to-Analog Converter)
* **Macros** are other large, pre-designed blocks. The specific macros are:
    * RISCV SoC (the main processor core block)
    * gpio bank (the block managing all GPIO functions)

Other integrated blocks on the die map include a PLL (Phase-Locked Loop) for clock generation and an SPI (Serial Peripheral Interface) block.

---

## 2. Introduction to RISC-V

RISC-V illustrates the complete flow from a high-level architecture down to a physical hardware implementation.
* The **RISC-V Architecture** is the abstract Instruction Set Architecture (ISA). A C code file (`swap.c`) compiled for this architecture results in a `swap.o` file with the "file format elf64-littleriscv".
* Disassembly of this object file reveals the human-readable RISC-V assembly instructions, such as `addi sp, sp, -16`, `sd a0, 8(sp)`, `ld a1, 0(sp)`, and `jalr ra`. The architecture is the functional contract between the software and the hardware.
* The **Implementation** is the concrete hardware design, written in a Hardware Description Language (HDL).
* The example is a "picorv32 cpu core" (`picorv32.v`). This Verilog file contains parameter definitions (e.g., `ENABLE_COUNTERS`, `TWO_STAGE_SHIFTERS`, `CATCH_MISALIGNED`) and the core's sequential logic, such as the `always @(posedge clk)` block.
* The **Layout** is the final physical realization of the design. The "qflow" view is the geometric representation of the transistors and metal interconnects that will be fabricated on silicon.

This demonstrates the complete path: abstract Architecture (instructions) $\rightarrow$ hardware Implementation (RTL code) $\rightarrow$ physical Layout (GDSII file).

---

## 3. From Software Applications to Hardware

This process details the complete journey from a user-facing software application to the physical hardware that executes it.
* The flow begins with **Application Software** or Apps (e.g., a "Stop Watch app"). These are written in high-level languages like C, C++, VB, or Java.
* The "**Compiler input**" is the high-level code, such as a "'c' function for the stopwatch" that includes headers like `<stdio.h>` and `<time.h>`.
* This code is fed into a **Compiler**, which translates it into an executable file (e.g., `*.exe`) composed of low-level instructions.
* **System Software**, or an OS (like macOS, Windows 7, Linux), is a supporting layer that manages hardware resources. It "Handle[s] IO operations," "Allocate[s] memory," and performs other "Low level system functions."
* The **Compiler and Assembler** work together to produce the "Compiler and assembler output," which is "A RISC-V assembly language program."
* This output corresponds to Part 1 - **RISC-V Instruction Set Architecture (ISA)**. The ISA defines the "Abstract Instructions" (like `add x6, x10, x6`) that the hardware must understand.
* The **Assembler** converts these instructions into binary machine code, the "architecture' of computer" (e.g., `00000000101000000000001100010011`).
* This binary code is interpreted by the hardware, which is first described by an "**RTL snippet of hardware**" (Verilog code) that "understands instructions like 'add x6, x10, x6'".
* This RTL description is fed into a synthesis tool to create a "**Synthesized netlist**," a gate-level representation of the logic (using MUXes, AND gates, flip-flops, etc.).
* Finally, this netlist undergoes "**Physical Design Implementation**" to create the final Hardware layout. This layout shows the physical placement of cells (DECAP cells, etc.) and the routing for outputs like Dout1, Dout2, Dout3, and Clk Out.

---

## 4. Introduction to All Components of Open-Source Digital ASIC Design

Digital ASIC Design requires three key inputs to produce an ASIC: RTL IP's, EDA Tools, and PDK DATA.
Open Source Digital ASIC Design centers on creating an ASIC using components that are all open. The three main inputs are:
* **Open RTL Designs:** Sourced from online repositories like librescores.org, opencores.org, github.com, and others.
* **Open EDA Tools:** Open-source Electronic Design Automation tools, such as QFlow, OpenROAD, and OpenLANE.
* **Open PDK Data:** This is made possible by the "FOSS 130nm Production PDK" from a partnership between Google and Skywater, which is accessible at [github.com/google/skywater-pdk](https://github.com/google/skywater-pdk).

A PDK (Process Design Kit) is "the interface between the FAB and the designers."
* Historically, IC design was "tightly integrated with the manufacturing processes available within each company."
* Lynn Conway and Carver Mead "pioneered the 'structured' design methodology based on $\lambda$-based rules." This separated design from technology (manufacturing).
* This separation enabled the modern model of "Pure Play Fabs" (manufacturing only) and "Fabless" design companies (design only).

A PDK is a "Collection of files used to model a fabrication process for the EDA tools used to design an IC." Its contents include:
* Process Design Rules (DRC, LVS, PEX)
* Device Models
* Digital Standard Cell Libraries
* I/O Libraries

EDA Tools cover the full spectrum of the design flow: HDL Design Entry, Logic Synthesis, Floor Planning, Global/Detailed Placement, Clock Tree Synthesis (CTS), Global/Detailed Routing, RC Extraction, Static Timing Analysis (STA), Logic Equivalence Checking (LEC), DRC, LVS, and more.
Regarding the 130nm node, it is still considered fast. The Intel P4EE @ 3.46 GHz (Q4'04) was a 130nm chip. An OSU team reported a 327 MHz post-layout clock frequency for a single-cycle RV32i CPU, and "a pipelined version can achieve > 1 GHz clock."
A performance table for the sky130_osu_18T_hs standard cell library details Frequency, Area, and PDP (Power-Delay Product) at both the Synthesis and Place-and-route stages.

---

## 5. Simplified RTL2GDS Flow

The "Simplified RTL to GDSII Flow" is the core process of converting hardware description code (RTL) into a final physical layout file (GDSII) for fabrication.
The flow begins with RTL as the primary design input and uses the PDK as the technology input for all stages.
The main stages of this flow are: Synth $\rightarrow$ FP+PP $\rightarrow$ Place $\rightarrow$ CTS $\rightarrow$ Route $\rightarrow$ Sign Off.

1.  **Synthesis (Synth):**
    * This stage "Converts RTL to a circuit out of components from the standard cell library (SCL)."
    * An `always` block in Verilog is converted by the Synth tool, using the SCL, into a gate-level netlist (a schematic of gates and flip-flops).
    * "Standard Cells" from the SCL have a "regular layout" and come with different "views/models" such as "Electrical (HDL, SPICE)" and "Layout (Abstract and Detailed)."

2.  **Floor and Power Planning (FP+PP):**
    * **Chip Floor-Planning** involves "Partition[ing] the chip die between different system building blocks" (e.g., CPU, SRAM, I/O) and "plac[ing] the I/O Pads" around the periphery.
    * **Macro Floor-Planning** defines the "Dimensions, pin locations, [and] rows definition" for large IP blocks.
    * **Power Planning** creates the power grid using "Power pads (VDD, VSS)," "Power rings (VDD, VSS)," and "Power straps."

3.  **Placement (Place):**
    * This stage "place[s] the cells on the floorplan rows, aligned with the sites." The gate-level netlist cells (INV, NOR, NAND, DFF) are physically placed.
    * This is "usually done in 2 steps: Global and Detailed." Global placement is a rough, overlapping arrangement, while Detailed placement organizes the cells neatly.

4.  **Clock Tree Synthesis (CTS):**
    * The purpose is to "Create a clock distribution network."
    * This network must "deliver the clock to all sequential elements (e.g., FF)" with "minimum skew" (as zero is hard to achieve) and in a "good shape."
    * It is "usually a Tree (H, X, ...)" structure to ensure simultaneous clock arrival.

5.  **Routing (Route):**
    * This stage "Implement[s] the interconnect using the available metal layers." "Tracks" on different metal layers (e.g., M1, M2) are used to wire the cells.
    * These "Metal tracks form a routing grid," which is noted as being "huge."
    * A "Divide and Conquer" strategy is used:
        * **Global Routing:** "Generates routing guides" (a high-level plan for wiring).
        * **Detailed Routing:** "Uses the routing guides to implement the actual wiring."

6.  **Sign Off:**
    * This is the final verification stage.
    * **Physical Verifications:**
        * **Design Rules Checking (DRC):** Ensures the layout meets all foundry manufacturing rules.
        * **Layout vs. Schematic (LVS):** Confirms the physical layout is electrically identical to the original netlist.
    * **Timing Verification:**
        * **Static Timing Analysis (STA):** Checks that the design meets all timing requirements (setup and hold).

---

## 6. Introduction to OpenLANE and Strive Chipsets

OpenLANE is an automated ASIC flow that "Started as an Open-Source Flow for a True Open Source Tape-out Experiment."
The "striVe SoC Family" is "a family of open everything SoCs," meaning they utilize "Open PDK, Open EDA, [and] Open RTL." The family is built around the StriVe RISC-V 32b/Y core.
The features for the various SoCs in the family (striVe, striVe 2, striVe 2a, striVe 3, striVe 5, striVe 6) include:
* "Sky130 SCL"
* "Synthesized 1 Kbytes SRAM"
* "1 Kbytes OpenRAM block"
* DFT (in striVe 6)

The open-source ecosystem partners include Skywater, Google, OpenROAD, and Efabless.
The "OpenLANE ASIC Flow" has a "Main Goal": to "Produce a clean GDSII with no human intervention (no-human-in-the-loop)."
A "Clean" GDSII is one with:
* "No LVS Violations"
* "No DRC Violations"

"Timing Violations?" are noted as a "WIP!" (Work In Progress).
The flow is "Tuned for SkyWater 130nm Open PDK."
* It is "Containerized," making it "Functional out of the box," though "Instructions to build and run natively will follow."
* The flow "can be used to harden Macros and Chips."
* It has "Two modes of operation: Autonomous or Interactive."
* A key feature is "**Design Space Exploration**" to "Find the best set of flow configurations."
* This is supported by a "Large number of design examples," including "43 designs with their best configurations," with "More will be added soon."

---

## 7. Introduction to OpenLANE Detailed ASIC Design Flow

The "OpenLANE ASIC Flow" is a detailed, automated process using a specific chain of open-source tools.

* **Inputs:** The flow begins with Design RTL and the SKY130 PDK.
* **Synthesis Stage:**
    * **RTL Synthesis** is performed by **Yosys + abc**. This can be guided by **Synth Exploration**.
    * Post-synthesis **STA (OpenSTA)** is run.
    * **DFT (Fault)** insertion is an optional step at this stage.
* **Physical Implementation (PnR) Stage:**
    * The **OpenROAD App** integrates **Floorplanning**, **Placement**, **CTS (Clock Tree Synthesis)**, **Optimization**, and **Global Routing**.
    * This stage is also informed by **Design Exploration** to find optimal parameters.
* **Routing and Post-Routing Stage:**
    * **Detailed Routing** is performed by **TritonRoute**.
    * A "**Fake ant. diodes Insertion Script**" is run to address potential violations.
    * **RC Extraction** is then performed.
* **Verification Stage:**
    * **LEC (Yosys)** is used for Logic Equivalence Checking.
    * **STA (OpenSTA)** is run again with accurate parasitic data from RC extraction.
    * **Physical Verification (Magic & netgen)** is run to check for DRC and LVS errors.
    * A "**Fake ant. diodes Swapping Script**" may be run based on verification feedback.
* **Output:** **gds2 Streaming (Magic)** is used to generate the final **GDSII** file.

**Key Flow Components Explained:**
* **Design Exploration & Regression Testing:** The "design exploration utility" is used for tuning, as seen in the tables for `aes` and `cordic` designs (listing Runtime, TR Vios, FP_CORE_UTIL, etc.). This utility is also used for "regression testing (CI)," where OpenLane is "run on ~70 designs" to "compare the results to the best known ones" and ensure TR Vios remain at zero.
* **Design for Test (DFT):** This includes "Scan Insertion," "Automatic Test Pattern Generation (ATPG)," "Test Patterns Compaction," "Fault Coverage," and "Fault Simulation." This is implemented using a scan chain (sin, sout) through the design's flip-flops.
* **Physical Implementation:** This is also called "automated PnR (Place and Route)" and is managed by **OpenROAD**. The steps include "Floor/Power Planning," "End Decoupling Capacitors and Tap cells insertion," "Placement: Global and Detailed," "Post placement optimization," "CTS," and "Routing: Global and Detailed."
* **Logic Equivalence Check (LEC):** This check is performed using **Yosys**. It is essential because "every time the netlist is modified, verification must be performed." Both "CTS modifies the netlist" and "Post Placement optimizations modif[y] the netlist." "LEC is used to formally confirm that the function did not change after modifying the netlist."
* **Dealing with Antenna Rules Violations:** This is a manufacturing issue where a "metal wire segment... act[s] as an antenna" during "Reactive ion etching." This causes "charge to accumulate on the wire," and "Transistor gates can be damaged."
    * **Solutions:** 1) "Bridging," which "attaches a higher layer intermediary," and 2) "Add[ing] antenna diode cell to leak away charges" (diodes are "provided by the SCL").
    * **OpenLANE Approach:** A "preventive approach" is taken:
        1.  "Add a Fake Antenna Diode next to every cell input after placement."
        2.  "Run the Antenna Checker (Magic) on the routed layout."
        3.  "If the checker reports a violation... replace the Fake Diode cell by a real one."
* **Static Timing Analysis (STA):** This step involves "**RC Extraction**" (using `DEF2SPEF`) to extract parasitic resistance and capacitance. Then, "**STA**" is run using "**OpenSTA (OpenROAD)**." A timing report is generated that calculates the slack (MET) based on data arrival time and data required time.
* **Physical Verification (DRC & LVS):** **Magic** (the "Magic VLSI Layout Tool") is used for "Design Rules Checking and SPICE Extraction from Layout." **Magic** and **Netgen** are used together for LVS by comparing the "Extracted SPICE by Magic" against the "Verilog netlist."

## LABS ON Open Lane (directory Structure, Design Prep, synthesis and synthesis results)

The following is the pdk structure

```
pdks/
└── sky130A/
    ├── libs.ref/  ← Design Libraries
    │   └── sky130_fd_sc_hd/
    │       ├──  lef/      ← Layout format
    │       ├──  lib/      ← Timing data
    │       ├── gds/      ← Physical layout
    │       └──  verilog/  ← Cell models
    │
    └──  libs.tech/  ← Technology Files
        ├──  magic/,  klayout/,  ngspice/
        ├──  openroad/,  drc/,  lvs/,  pex/
```

The following is the working directory all required files involved in the process

<img width="1280" height="768" alt="working_directory" src="https://github.com/user-attachments/assets/8475e016-ff0e-4ff9-9bd5-7604749b224a" />

Now lets invoke the tool and start the design in interactive mode

```
# Enter OpenLANE directory
cd OpenLane

docker

# Start interactive mode
./flow.tcl -interactive
```

<img width="1056" height="274" alt="openlane_flow tcl_invoke" src="https://github.com/user-attachments/assets/68cbd117-2fef-4652-aaf6-ec496f4e95d8" />

Now , Import openlane package

```
package require openlane 0.9
```

Now , prepare the design picorv32a

```
prep -design picorv32a
```

<img width="1214" height="635" alt="prep_Design" src="https://github.com/user-attachments/assets/9e2c77f9-68d4-4cd1-8adc-1662dadd7545" />

By doing so , we get to see that it Creates organized directory structure,Merges Technology LEF (.tlef) with Cell LEF (.lef),Sets up configuration files,Prepares design for synthesis.

Now, start the synthesis

```
run_synthesis
```

<img width="1216" height="741" alt="synthesis_picorv32a" src="https://github.com/user-attachments/assets/dde761d4-91c2-463f-94cd-86980a58cc3b" />


The tool Reads RTL files,Maps to standard cells,Optimizes logic,Generates statistics,Creates gate-level netlist.

Now we see the synthesis results

<img width="259" height="655" alt="synthesis_results" src="https://github.com/user-attachments/assets/af523a36-3db4-4469-813c-44b7479ff993" />

From this we get the dflop to total cells ratio as follows

<img width="2334" height="1403" alt="flopratio" src="https://github.com/user-attachments/assets/cae252ac-6f51-42ed-ae83-1a0d0ce192f2" />







