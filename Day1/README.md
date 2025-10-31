## 1. Introduction to QFN-48 Package, Chip, Pads, Core, Die, and IPs

The physical chip is housed within a Package. The specific type shown is a QFN-48, which stands for Quad Flat No-leads.
The physical dimensions of this square package are specified as 7mm x 7mm.
The package's pinout diagram details all 48 connections. These connections include:
* **General Purpose I/O:** gpio0 through gpio15.
* **Power and Ground:** vdd3v3 (3.3V supply), vdd1v8 (1.8V supply), and vss (ground).
* **Flash Memory Interface:** flash_csb, flash_clk, sck, sdo, sdi, flash_io0, flash_io1, flash_io2, flash_io3.
* **Analog-to-Digital Converter (ADC) Pins:** adc0_in, adc1_in, vref_h (voltage reference high), vref_l (voltage reference low).
* **Clock Oscillator Pins:** XI and XO (crystal input/output).
* **Serial Interface:** ser_tx (serial transmit) and ser_rx (serial receive).
* **Other Pins:** analog_out, comp_inn, comp_inp, irq, and xclk.

The internal structure of the chip is defined by its constituent parts.
The "chip" is the central component inside the package, with all internal logic connected via lines to the external pads.
A distinction is made between the "Die", which is the entire rectangular piece of silicon, and the "Core", which is the central area of the die where the primary logic is placed.
The "PADS" are the connection points located on the perimeter of the die, which interface between the internal core logic and the external package pins.
The core itself is composed of multiple functional blocks, categorized as either Macros or Foundry IP's.
Foundry IP's (Intellectual Property) are pre-designed, verified blocks provided by the fabrication facility (the foundry). In the provided diagram, these are identified as:
* SRAM (Static Random-Access Memory)
* adc0 (Analog-to-Digital Converter 0)
* adc1 (Analog-to-Digital Converter 1)
* dac (Digital-to-Analog Converter)

Macros are other large, pre-designed blocks. The specific macros pointed out are:
* RISCV SoC (the main processor core block)
* gpio bank (the block managing all GPIO functions)

Other smaller, integrated blocks are also visible on the die map, including a PLL (Phase-Locked Loop) for clock generation and an SPI (Serial Peripheral Interface) block.

---

## 2. Introduction to RISC-V

The RISC-V concept illustrates the flow from high-level architecture down to physical implementation.
First, the RISC-V Architecture is defined as the abstract instruction set (ISA). This is demonstrated using a sample C code file, `swap.c`.
This C code, when compiled, is disassembled into a `swap.o` file with a "file format elf64-littleriscv".
This disassembly reveals the human-readable RISC-V assembly instructions, such as `addi sp, sp, -16`, `sd a0, 8(sp)`, `ld a1, 0(sp)`, and `jalr ra`. This architecture is the contract between the software and the hardware.
Second, the Implementation of this architecture is the concrete hardware design, written in a Hardware Description Language (HDL).
The example provided is Verilog code for a "picorv32 cpu core" (`picorv32.v`).
This Verilog file contains parameter definitions (like `ENABLE_COUNTERS`, `TWO_STAGE_SHIFTERS`, `CATCH_MISALIGNED`) and the core logic, such as the `always @(posedge clk)` block, which defines the processor's behavior on each clock cycle.
Third, the Layout is the final physical realization of the design. This "qflow" view is the geometric representation of the transistors and metal interconnects that will be fabricated on the silicon, representing the physical form of the picorv32 core.
A clear flow is shown from the abstract Architecture (instructions) to a hardware Implementation (RTL code), which is then physically mapped to a Layout (GDSII file).

---

## 3. From Software Applications to Hardware

This section details the complete journey from a user-facing software application to the physical hardware that executes it.
The process begins at the top with Application Software or Apps, such as the "Stop Watch app" example. These applications are written in high-level languages like C, C++, VB, or Java.
The "Compiler input," in this case, is a 'c' function for the stopwatch. This C code includes headers like `<stdio.h>` and `<time.h>` and contains the logic for timekeeping.
This high-level code is fed into a Compiler. The compiler translates the code into an executable file (like a `*.exe` file) composed of low-level instructions (Instr1, Instr2, ...).
The System Software, or OS (like macOS, Windows 7, or Linux), is a supporting layer that manages hardware resources, "Handle[s] IO operations," "Allocate[s] memory," and performs other "Low level system functions."
The compiler and Assembler work in tandem to produce the "Compiler and assembler output," which is identified as "A RISC-V assembly language program." A side-by-side view clearly shows the C code on the left and its corresponding RISC-V assembly output on the right.
This leads to Part 1 - RISC-V Instruction Set Architecture. This ISA is the level of "Abstract Instructions" (like `add x6, x10, x6`) that the hardware must understand.
These instructions are converted by the Assembler into binary machine code, the "architecture' of computer" (e.g., `00000000101000000000001100010011`).
This binary code is interpreted by the hardware, which is first described by an "RTL snippet of hardware" (Verilog code) that "understands instructions like 'add x6, x10, x6'".
This RTL description is then fed into a synthesis tool to create a "Synthesized netlist," which is a gate-level representation of the logic (showing MUXes, AND gates, flip-flops, etc.).
Finally, this gate-level netlist undergoes "Physical Design Implementation" to create the final Hardware layout. This layout shows the physical placement of cells, including DECAP (decoupling capacitors), and the routing for outputs like Dout1, Dout2, Dout3, and Clk Out.

---

## 4. Introduction to All Components of Open-Source Digital ASIC Design

The process of Digital ASIC Design is first introduced using an analogy of an AND gate. To produce an "ASIC" (the output), three key inputs are required: "RTL IP's", "EDA Tools", and "PDK DATA".
This is then expanded to Open Source Digital ASIC Design, which centers around creating an ASIC using components that are all open.
The three main inputs for an open-source ASIC are:
* **Open RTL Designs:** These are sourced from online repositories like librescores.org, opencores.org, github.com, and others.
* **Open EDA Tools:** These are the open-source Electronic Design Automation tools used to process the design, such as QFlow, OpenROAD, and OpenLANE.
* **Open PDK Data:** The availability of open Process Design Kits was initially raised as a question ("Open?"). This is answered affirmatively, highlighting the "FOSS 130nm Production PDK" made available through a partnership between Google and Skywater. This PDK is accessible on github.com/google/skywater-pdk.

The fundamental question, "What is a PDK?", is then addressed.
Historically, in the "age of Gods," an IC's design was "tightly integrated with the manufacturing processes available within each company."
A paradigm shift was envisioned by Lynn Conway and Carver Mead, who "pioneered the 'structured' design methodology based on $\lambda$-based rules." This crucial step separated the design process from the technology (manufacturing) process.
This separation enabled the modern semiconductor industry model of "Pure Play Fabs" (which only manufacture chips) and "Fabless" design companies (which only design chips).
Therefore, the PDK is defined as "the interface between the FAB and the designers."
A PDK, or Process Design Kit, is a "Collection of files used to model a fabrication process for the EDA tools used to design an IC."
The contents of a PDK include:
* Process Design Rules (DRC, LVS, PEX): The geometric and electrical rules for manufacturing.
* Device Models: Simulation models for transistors.
* Digital Standard Cell Libraries: Pre-designed logic gates (AND, OR, FF, etc.).
* I/O Libraries: Libraries for the input/output pads.

A slide titled "EDA Tools" lists the wide variety of processes involved in a full ASIC flow, including: HDL Design Entry, Logic Synthesis, Floor Planning, Global/Detailed Placement, Clock Tree Synthesis (CTS), Global/Detailed Routing, RC Extraction, Static Timing Analysis (STA), Logic Equivalence Checking (LEC), Design Rule Checking (DRC), Layout vs. Schematic (LVS), and many others.
Finally, the question "Is 130nm Fast?" is answered with "Yes!".
A comparison is made to the Intel P4EE @ 3.46 GHz from 2004, which was a 130nm chip.
It is reported that an OSU team achieved a 327 MHz post-layout clock frequency for a single-cycle RV32i CPU.
It is also stated that "a pipelined version can achieve > 1 GHz clock."
A table shows performance metrics for the sky130_osu_18T_hs standard cell library, detailing Frequency, Area, and PDP (Power-Delay Product) at both the Synthesis and Place-and-route stages.

---

## 5. Simplified RTL2GDS Flow

A high-level overview of the "Simplified RTL to GDSII Flow" is presented. This is the core process of turning hardware code into a physical file for the foundry.
The flow starts with RTL (Verilog/VHDL code) as the primary input and produces a GDSII file as the final output. The PDK is a critical input used by the tools throughout the flow.
The main stages of this flow are identified in order: Synth $\rightarrow$ FP+PP $\rightarrow$ Place $\rightarrow$ CTS $\rightarrow$ Route $\rightarrow$ Sign Off.
Each of these stages is explained in greater detail:

1.  **Synthesis (Synth):**
    * This stage "Converts RTL to a circuit out of components from the standard cell library (SCL)."
    * An example shows a Verilog `always` block being processed by the Synth tool, which uses the SCL to generate a gate-level netlist (a schematic of gates and flip-flops).
    * The "Standard Cells" from the SCL have a "regular layout" and come with different "views/models" such as "Electrical (HDL, SPICE)" and "Layout (Abstract and Detailed)." Example layouts for an Inverter (INV), NAND2, and XOR2 are shown.

2.  **Floor and Power Planning (FP+PP):**
    * **Chip Floor-Planning** is performed, which involves "Partition[ing] the chip die between different system building blocks" (like CPU, SRAM, Flash CTRL, I/O) and "plac[ing] the I/O Pads" around the periphery.
    * **Macro Floor-Planning** is done for large blocks, defining their "Dimensions, pin locations, [and] rows definition."
    * **Power Planning** is a critical step where the power grid is created using "Power pads (VDD, VSS)," "Power rings (VDD, VSS)," and "Power straps" that crisscross the core area.

3.  **Placement (Place):**
    * This stage involves "plac[ing] the cells on the floorplan rows, aligned with the sites." The gate-level netlist is taken, and its individual cells (like INV, NOR, NAND, DFF) are physically placed onto the rows defined during floorplanning.
    * This process is "usually done in 2 steps: Global and Detailed." An image depicts Global Placement as a rough, overlapping arrangement, while Detailed Placement shows the cells neatly organized and aligned.

4.  **Clock Tree Synthesis (CTS):**
    * The purpose of CTS is to "Create a clock distribution network."
    * The goal of this network is to "deliver the clock to all sequential elements (e.g., FF)" with "minimum skew" (as zero skew is hard to achieve).
    * This network is "usually a Tree (H, X, ...)" structure to ensure the clock signal arrives at all flip-flops at as close to the same time as possible.

5.  **Routing (Route):**
    * This stage "Implement[s] the interconnect using the available metal layers." A diagram shows how horizontal and vertical "Tracks" on different metal layers (e.g., M1, M2) are used to connect the placed cells.
    * These "Metal tracks form a routing grid," which is noted as being "huge."
    * A "Divide and Conquer" strategy is employed, which consists of:
        * **Global Routing:** This step "Generates routing guides" (a high-level plan for where wires should go).
        * **Detailed Routing:** This step "Uses the routing guides to implement the actual wiring."

6.  **Sign Off:**
    * This is the final verification stage before the design is complete.
    * It includes **Physical Verifications:**
        * **Design Rules Checking (DRC):** Ensures the layout meets the foundry's manufacturing rules.
        * **Layout vs. Schematic (LVS):** Confirms that the physical layout is electrically identical to the original synthesized netlist.
    * It also includes **Timing Verification:**
        * **Static Timing Analysis (STA):** Checks if the design meets all timing requirements (e.g., setup and hold times).

---

## 6. Introduction to OpenLANE and Strive Chipsets

OpenLANE is an automated ASIC flow. It "Started as an Open-Source Flow for a True Open Source Tape-out Experiment."
The "striVe SoC Family" is presented as a case study. This family is described as "a family of open everything SoCs," meaning they utilize "Open PDK, Open EDA, [and] Open RTL." A block diagram and layout for the StriVe RISC-V 32b/Y core are shown.
A table details the features of various SoCs in the family (striVe, striVe 2, striVe 2a, striVe 3, striVe 5, striVe 6).
Features include using the "Sky130 SCL," "Synthesized 1 Kbytes SRAM," "1 Kbytes OpenRAM block," and the inclusion of DFT (Design for Test) in striVe 6.
The partners involved in this open-source ecosystem are listed as Skywater, Google, OpenROAD, and Efabless.
The "OpenLANE ASIC Flow" is then described in more detail.
The "Main Goal" of the flow is to "Produce a clean GDSII with no human intervention (no-human-in-the-loop)."
A "Clean" GDSII is defined as one with:
* "No LVS Violations"
* "No DRC Violations"

It is noted that "Timing Violations?" are still a "WIP!" (Work In Progress).
The flow is specifically "Tuned for SkyWater 130nm Open PDK."
It is "Containerized," which means it is "Functional out of the box." Instructions are also available to "build and run natively."
The OpenLANE flow "can be used to harden Macros and Chips."
It supports "Two modes of operation: Autonomous or Interactive."
A key feature highlighted is "Design Space Exploration," which is used to "Find the best set of flow configurations."
This feature is supported by a "Large number of design examples," including "43 designs with their best configurations," with a note that "More will be added soon."

---

## 7. Introduction to OpenLANE Detailed ASIC Design Flow

A detailed flowchart of the "OpenLANE ASIC Flow" outlines the specific tools used at each stage.
**Inputs:** The flow begins with Design RTL and the SKY130 PDK.

**Synthesis Stage:**
* The RTL is first processed by **RTL Synthesis**, which uses **Yosys + abc**. This can be guided by **Synth Exploration**.
* After synthesis, initial **STA (OpenSTA)** is run.
* **DFT (Fault)** insertion can also be performed at this stage.

**Physical Implementation (PnR) Stage:**
* The synthesized netlist enters the **OpenROAD App**, which integrates multiple steps: **Floorplanning**, **Placement**, **CTS (Clock Tree Synthesis)**, **Optimization**, and **Global Routing**.
* This stage is also informed by **Design Exploration** to find optimal parameters.

**Routing and Post-Routing Stage:**
* After Global Routing, **Detailed Routing** is performed by **TritonRoute**.
* After routing, a "**Fake ant. diodes Insertion Script**" is run to address potential manufacturing violations.
* **RC Extraction** is then performed.

**Verification Stage:**
* **LEC (Yosys)** is used for Logic Equivalence Checking.
* **STA (OpenSTA)** is run again, this time with accurate parasitic data from RC extraction.
* **Physical Verification (Magic & netgen)** is run to check for DRC and LVS errors.
* A "**Fake ant. diodes Swapping Script**" may be run based on verification feedback.

**Output:** If all checks pass, **gds2 Streaming (Magic)** is used to generate the final **GDSII** file.

Several of these specific steps are explained in greater detail:
* **Design Exploration & Regression Testing:**
    * The "design exploration utility" is used for tuning, as shown in a table for `aes` and `cordic` designs which lists metrics like Runtime, TR Vios (TritonRoute Violations), and FP_CORE_UTIL (Core Utilization).
    * This utility is also used for "regression testing (CI)," where OpenLane is "run on ~70 designs" to "compare the results to the best known ones," ensuring that all TR Vios remain at zero.

* **Design for Test (DFT):**
    * This involves techniques like "Scan Insertion," "Automatic Test Pattern Generation (ATPG)," "Test Patterns Compaction," "Fault Coverage," and "Fault Simulation." A diagram illustrates a scan chain (sin to sout) passing through flip-flops and combinational logic.

* **Physical Implementation:**
    * This is also referred to as "automated PnR (Place and Route)" and is managed by **OpenROAD**.
    * The steps within this stage are listed as: "Floor/Power Planning," "End Decoupling Capacitors and Tap cells insertion," "Placement: Global and Detailed," "Post placement optimization," "CTS," and "Routing: Global and Detailed."

* **Logic Equivalence Check (LEC):**
    * This check is performed using **Yosys**.
    * It is required because "every time the netlist is modified, verification must be performed." Both "CTS modifies the netlist" and "Post Placement optimizations modif[y] the netlist."
    * "LEC is used to formally confirm that the function did not change after modifying the netlist."

* **Dealing with Antenna Rules Violations:**
    * This manufacturing issue is explained: "When a metal wire segment is fabricated, it can act as an antenna." "Reactive ion etching" can cause charge to "accumulate on the wire," which "can be damaged" transistor gates.
    * Two solutions are described: 1) "Bridging," which "attaches a higher layer intermediary," and 2) "Add[ing] antenna diode cell to leak away charges" (these diodes are "provided by the SCL").
    * A "preventive approach" is taken in OpenLANE:
        * "Add a Fake Antenna Diode next to every cell input after placement."
        * "Run the Antenna Checker (Magic) on the routed layout."
        * "If the checker reports a violation... replace the Fake Diode cell by a real one."

* **Static Timing Analysis (STA):**
    * This step first involves "**RC Extraction**" using `DEF2SPEF` to extract parasitic resistance and capacitance from the layout.
    * Then, "**STA**" is run using "**OpenSTA (OpenROAD)**." A sample timing report is shown, detailing the data arrival time and data required time to calculate the final slack (MET).

* **Physical Verification (DRC & LVS):**
    * **Magic** (the "Magic VLSI Layout Tool") is used for "Design Rules Checking and SPICE Extraction from Layout."
    * **Magic** and **Netgen** are used together to perform LVS (Layout vs. Schematic). This is done by comparing the "Extracted SPICE by Magic" against the "Verilog netlist."
