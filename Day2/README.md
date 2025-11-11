## DAY2
## 1. Utilization factor and aspect ratio

### Core vs. Die Concepts:
A silicon wafer contains multiple individual circuits, and each individual circuit is known as a 'die'.
A 'die' is the specific, small specimen of semiconductor material on which the fundamental circuit is fabricated.
Each 'die' is composed of two main parts: the 'core' and the surrounding I/O (Input/Output) area.
The core is the inner, central area where all the logical cells and circuitry are placed. Its dimensions are defined by 'H' (Height) and 'W' (Width).
The I/O area is the region between the core and the die edge, which is reserved for I/O pads.

### Netlist as a Logical Starting Point:
The entire design process begins with a 'netlist'. A netlist is a description of all the connectivity within an electronic design.
An example netlist was used, consisting of two flip-flops (FFs) (which can also be latches or registers) and two standard cells: an AND gate (A1) and an OR gate (O1).
The connectivity in this example is as follows: A single clock (Clk) drives both FFs. The output (Q) of the first FF connects to inputs on both the AND gate (A1) and the OR gate (O1). The outputs from A1 and O1 are then combined and fed into the 'D' input of the second FF.

### From Logical to Physical Dimensions:
The next step in the process is to convert these logical netlist components into their physical dimensions.
The symbols for the FFs and gates are mapped to physical footprints, which represent the actual silicon area each cell will occupy.

### Area Calculation:
A simplified area calculation was performed for the example netlist.
A standard cell (like the AND/OR gates) was assumed to have dimensions of 1 unit by 1 unit, giving it an area of 1 square unit.
Similarly, a flip-flop (FF) was also assumed to be 1x1 unit, for an area of 1 square unit.
Based on this, the total area occupied by the example netlist (2 FFs and 2 gates) is 4 square units.

### Defining Utilization Factor:
These logical cells are then placed inside the 'core'. The four example cells were arranged in a 2x2 grid to illustrate this.
Utilization Factor is a key metric defined by the equation:
> Utilization Factor = (Area Occupied by Netlist) / (Total Area of the Core)

A 100% Utilization scenario occurs when the logical cells (the netlist) occupy the entire area of the core. In this case, the Utilization Factor is 1.

### Defining Aspect Ratio:
Alongside the utilization factor, the Aspect Ratio of the core is also defined.
The formula for this is:
> Aspect Ratio = Height / Width

In the 2x2 grid example, the total core width is 2 units and the total core height is 2 units.
Therefore, the Aspect Ratio for this example core is 2 / 2 = 1, which describes a perfect square.

---

## 2. Concept of pre-placed cells

### Logic Partitioning:
The process starts with a large, abstract block of 'Combinational logic'.
This logic is then synthesized into a detailed gate-level netlist (e.g., gates A1 through A8).
This netlist can then be partitioned. In the example, cut1 and cut2 were used to logically separate the gates into two distinct groups: Block 1 (A1, A3, A2, A4) and Block 2 (A5, A7, A6, A8).

### Hierarchical Abstraction (Black Boxing):
Once partitioned, the inter-block connections (like the ones from A2 to A5 and A4 to A5) are identified.
These signals are then brought to the boundary of their respective blocks as 'Extended IO pins'.
The blocks are then converted into 'Black Boxes'. This is an abstraction that hides the internal gate-level complexity. The blocks are now defined only by their boundaries and their I/O pins.
These two Black Boxes can now be treated as two separate 'IP's (Intellectual Properties) or modules'.

### Definition and Types of Pre-placed Cells:
In a real chip, there are many other common IPs besides these user-defined blocks.
Examples include: Memory, Clock-gating cells, Comparators, and Muxes.
The arrangement of all these IPs/blocks on the chip is formally known as Floorplanning.
These blocks are called 'pre-placed cells' because they must be placed in the chip before any automated placement-and-routing tools are run.
This is because these IPs (like memories) often have user-defined locations or other specific constraints.
After these pre-placed cells are fixed in their locations, the automated tools are then used to place all the remaining logical cells (the smaller standard cells) into the available space.

---

## 3. De-coupling capacitors

### Context of Pre-placed Cells:
The floorplan was revisited, now showing the pre-placed Block a, Block b, and Block c fixed inside the core. The rest of the core is filled with standard cell rows (the blue horizontal lines) where regular logic will go.

### The Problem: Switching Current and Voltage Drop:
A complex circuit with multiple flip-flops (F) driving several outputs (Dout1, Dout2, etc.) was used to illustrate a common problem.
During a switching operation (when outputs change state), the circuit demands a large, instantaneous switching current, also known as peak current (Ipeak).
This current must be drawn from the power supply (Vdd) through the chip's power grid. This grid, however, has parasitic resistance (Rdd) and parasitic inductance (Ldd).
As this Ipeak is drawn through Rdd and Ldd, a voltage drop occurs.
The consequence is that the local voltage at the circuit's power pin (Node 'A') becomes Vdd', which is lower than the main power supply voltage (Vdd).

### Noise Margin and Signal Integrity:
This voltage drop is a serious problem because of Noise Margins.
A signal is only considered a 'logic 0' if its voltage is within the low noise margin (NML), specifically between Vol and Vil.
A signal is only considered a 'logic 1' if its voltage is within the high noise margin (NMH), specifically between Vih and Voh.
The region between Vil and Vih is an 'Undefined Region'.
If the local Vdd' drops so much that it falls below the Vih threshold, it lands in this 'undefined region'.
When this happens, a 'logic 1' being output by one circuit will not be detected as a 'logic 1' by the next circuit, causing a functional failure.

### The Solution: Decoupling Capacitors (Decaps):
The solution to this problem is the addition of Decoupling Capacitors (Decaps).
A capacitor (Cd) is added to the circuit in parallel with the switching logic, as close as possible to its Vdd and Vss pins.
This capacitor acts as a small, local charge reservoir.
When the circuit switches, it draws its high-frequency peak current directly from the local capacitor (Cd).
The main power supply then recharges the capacitor (Cd) more slowly through the RL network (Rdd/Ldd) after the switching event is over.
This prevents the large, sudden current draw from the main power grid, which in turn prevents the local voltage drop and protects the noise margin.

### Physical Implementation:
In the physical floorplan, this solution is implemented by placing DECAP1, DECAP2, and DECAP3 cells in the standard cell rows, filling the gaps between and around the pre-placed blocks (Block a, b, c).

---

## 4. Power planning

### Scaling the Problem: From Local to Global:
While decaps solve the local power issue, a larger, global problem must also be considered.
A system with a 'Driver' cell and a 'Load' cell connected by a long wire illustrates this. This path is connected to the main Vdd/Vss supply, which has parasitic resistances and inductances (Rdd, Ldd, Rss, Lss).
This problem is magnified when the connection is a wide bus, for example, a 16-bit bus.

### Simultaneous Switching Noise: Ground Bounce & Voltage Droop:
When a 16-bit bus switches, many bits can change at the same time. This is modeled as a bank of 16 capacitors. A '1' is a charged capacitor ('V' volts), and a '0' is a discharged capacitor (0 volts).
**Ground Bounce:** If many bits switch from '1' to '0' simultaneously, all their capacitors must discharge to 0 volts. If they all discharge through a single 'Ground' tap point, this massive, sudden current rush (di/dt) through the ground wire's inductance (Lss) causes the local ground potential to "bounce" up to a voltage above 0V.
**Voltage Droop:** Conversely, if many bits switch from '0' to '1' at the same time, they all must draw charging current from a single 'Vdd' tap point. This causes the local Vdd potential to "droop" or sag significantly.
Both Ground Bounce and Voltage Droop are forms of simultaneous switching noise that can cause functional failures.

### The Solution: The Power Grid:
The solution is to implement a power grid to provide multiple tap points for Vdd and Vss, distributing the current load.
This grid is an overlapping mesh of horizontal and vertical Vdd (blue) and Vss (red/grey) lines.
These lines are connected at their intersections using Contacts (yellow 'x's).
The final floorplan for "4) Power Planning" shows this implemented. A mesh of horizontal Vss lines (grey) and vertical Vdd lines (blue) runs across the entire core, on top of the pre-placed blocks and standard cell rows.

---

## 5. Pin placement and logical cell placement blockage

### Introducing the Example Design:
A complete example design was used, which expanded to four parallel data paths (for Dout1, Dout2, Dout3, Dout4).
This design also includes the three pre-placed blocks (Block a, Block b, Block c) as part of its logic.
The connectivity information for all these gates and blocks is coded in VHDL or Verilog and is known as the 'netlist'.

### Step 5: Pin Placement:
This step involves placing the I/O pins of the chip.
All the top-level I/O ports (Din1, Din2, Din3, Din4, Clk1, Clk2, Dout1, Dout2, Dout3, Dout4, Clk Out) are placed on the periphery of the die, in the orange-striped I/O area surrounding the core.

### Step 6: Logical Cell Placement Blockage:
The next step is to create 'Logical Cell Placement Blockages'.
In the floorplan, the areas already occupied by the pre-placed cells (Block a, b, c) and the decoupling capacitors (DEC P1, P2, P3) are marked with an orange-hatched pattern.
This pattern is a "keep-out" zone. It instructs the automated placement tool not to place any other logical cells (like the FFs and gates for the data paths) in these regions, because they are already occupied.

### Readiness for Next Stage:
With the core/die defined, pre-placed cells located, power grid planned, pins placed, and blockages defined, the "Floor Plan is Ready for Placement & Routing Step."

---

## 6. Steps to run floorplan using OpenLANE
(No information related to specific OpenLANE commands, scripts, or Tcl console inputs for running the floorplan stage was present in the provided set of images.)

---

## 7. Review floorplan files and steps to view floorplan
(No information related to the file structure output by the floorplan stage (e.g., .def, .lef files) or the steps to view the floorplan in a GUI tool (like magic or klayout) was present in the provided set of images.)

---

## 8. Netlist binding and initial place design

### Step 1: Bind Netlist with Physical Cells:
This process begins with the logical netlist (the schematic of the complete design).
A 'Library' is required, which contains the physical layout views for every cell used in the netlist (e.g., the physical layouts for FF1, gate 1, gate 2, FF2, and the blocks).
The "Physical View of Logic Gates" shows these physical cells arranged in rows.
'Binding' is the process where each logical component in the netlist is mapped to its corresponding physical layout from the library.

### Step 2: Placement (Initial):
The "Placement" step brings together three components: the Floorplan, the Netlist, and the Physical View of Logic Gates.
The floorplan, with its power grid, I/O pins, and pre-placed blocks, is the target.
The "initial place design" involves placing the physical cells from the library into the empty standard cell rows (blue lines) on the floorplan.
For example, the row of cells FF1, 1, 2, FF2 (corresponding to the logical Dout2 path) is placed into one of the empty rows. This is just the initial placement and has not yet been optimized.

---

## 9. Optimize placement using estimated wire-length and capacitance

### The Problem: Long Wires in Initial Placement:
This "Optimize placement" stage begins after the initial placement.
The Dout2 path is highlighted as an example. In the physical floorplan, the initial placement of the Dout2 cells (FF1, 1, 2, FF2) is spread out, resulting in long wire connections.
A long white arrow highlights a specific long-distance connection that is not ideal for timing.

### The Solution: Repeater Insertion:
At this stage, wire length and capacitance are estimated, and based on those estimations, repeaters are inserted.
To fix the long wire issue in the Dout2 path, two 'Buf' (Buffer) cells are inserted.
The cells in the physical layout are also rearranged and moved closer together into a different row (FF1, Buf, 1, Buf, 2, FF2) to optimize the connections and minimize wire length.

---

## 10. Final placement optimization

### Applying Optimization to All Paths:
This optimization process is applied iteratively to all critical paths in the design.
**Dout3 Path:** The Dout3 path is highlighted. Its physical cells are placed, and long wire connections are estimated. A 'Buf' cell is added to the path to correct for the long wire, and the cells are rearranged.
**Dout4 Path:** The Dout4 path is also analyzed. Long wires are identified (e.g., from the Din4 pin to FF1). 'Buf' cells are inserted into the physical layout to buffer these long signals and improve timing.
The core concept is that the placement tool estimates wire length and capacitance for all paths, then intelligently inserts repeaters (buffers) and rearranges cells to ensure the design can meet its timing requirements.

---

## 11. Need for libraries and characterization

### The Common Element: Gates and Cells:
"One Common Thing across all stages" of the design flow is the use of "GATES or Cells".
A 'Library' is the collection of all these fundamental building blocks. The examples listed include: AND gate, OR gate, BUFFER, INVERTER, DFF (D-Flip-Flop), LATCH, and ICG (Integrated Clock Gating cell).

### Libraries in the VLSI Flow:
This library is essential at every single step of the VLSI design flow:
* **Logic Synthesis:** The logical design is created using instances of cells from the library.
* **Floorplanning:** The floorplan is created to provide a physical space for these cells.
* **Placement:** The physical versions of the library cells are placed onto the floorplan.
* **CTS (Clock Tree Synthesis):** A balanced clock tree is built using specific library cells, like clock buffers and inverters.
* **Routing:** The final step is to create the metal wire connections between all the placed library cells.

### Library Characterization and Modelling:
This leads to the final, crucial topic: 'Library characterization and modelling'.
This is the process of analyzing every cell in the library to create detailed models for its timing (e.g., NLDM, CCS), power, and noise behavior.
This characterization is essential because the optimization tool needs these accurate models to "estimate wire length and capacitance" and decide where to "insert repeaters" (as seen in the previous topics). Without a well-characterized library, automated placement and optimization are not possible.
