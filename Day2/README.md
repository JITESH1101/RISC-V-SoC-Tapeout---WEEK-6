## DAY2

## 1. Utilization factor and aspect ratio

### Core vs. Die Concepts:
First, the fundamental structure of a chip was understood. A silicon wafer is shown to contain multiple individual circuits. Each of these individual circuits is known as a 'die'.
A 'die' is defined as the small semiconductor material specimen on which the fundamental circuit is fabricated.
The 'die' itself was shown to be composed of two main parts: the 'core' and the surrounding I/O (Input/Output) area.
The core is the inner, central area of the die where all the logical cells and circuitry are to be placed. The core's dimensions are labeled as 'H' (Height) and 'W' (Width).
The I/O area is the region between the core and the die edge, which is reserved for pads and I/O cells.

### Netlist as a Logical Starting Point:
The entire design process was shown to begin with a 'netlist'. A netlist is formally defined as the description of the connectivity within an electronic design.
An example netlist was provided, consisting of two flip-flops (FFs), which are also referred to as latches or registers, and two standard cells (A1, an AND gate, and O1, an OR gate).
The connectivity was clearly illustrated: The clock (Clk) drives both FFs. The output (Q) of the first FF feeds into input 'a' of the AND gate (A1) and input 'a' of the OR gate (O1). Input 'b' of A1 and input 'b' of O1 are also shown. The output 'y' of A1 and output 'y' of O1 are combined (implying further logic) and fed into the 'D' input of the second FF.

### From Logical to Physical Dimensions:
The next step illustrated was the conversion of these logical, symbolic netlist components into physical dimensions.
The symbols for the two FFs and two gates were highlighted with dashed yellow boxes, representing the physical area, or "footprint," that each cell will occupy on the die.

### Area Calculation:
A simplified calculation for the area occupied by the netlist was demonstrated.
It was assumed that a standard cell (like the gates A1 and O1) has dimensions of 1 unit by 1 unit, giving it an area of 1 square unit.
Similarly, a flip-flop (FF) was also assumed to have dimensions of 1 unit by 1 unit, resulting in an area of 1 square unit.
Based on the example netlist (2 FFs and 2 gates), the total area occupied by the netlist would be 4 square units.

### Defining Utilization Factor:
The concept of placing these logical cells inside the 'core' was shown. The four example cells were placed in a 2x2 grid within the core boundary.
Utilization Factor was formally defined with the equation:
> Utilization Factor = (Area Occupied by Netlist) / (Total Area of the Core)

A scenario of 100% Utilization was explained. This occurs when the logical cells (the netlist) occupy the complete area of the core. In this case, the Area Occupied by the Netlist is equal to the Total Area of the Core, making the Utilization Factor equal to 1.

### Defining Aspect Ratio:
Alongside the utilization factor, the Aspect Ratio was also defined.
The formula was given as:
> Aspect Ratio = Height / Width

Using the 2x2 grid example, where each cell is 1x1 unit, the total core was shown to be 2 units wide and 2 units high.
Therefore, the Aspect Ratio for this example core is 2 / 2 = 1, which describes a perfect square.

---

## 2. Concept of pre-placed cells

### Logic Partitioning:
The process was initiated by considering a large, abstract block of 'Combinational logic', represented as a cloud.
This logic was then synthesized into a detailed gate-level netlist (composed of gates A1 through A8).
A key step shown was the partitioning of this netlist using "cuts." The diagram showed cut1 and cut2, which logically separated the gates into two distinct groups: Block 1 (A1, A3, A2, A4) and Block 2 (A5, A7, A6, A8).

### Hierarchical Abstraction (Black Boxing):
Once partitioned, the inter-block connections (e.g., from A2 to A5, and A4 to A5) were highlighted in yellow.
The concept of 'Extend IO pins' was introduced, where these inter-block signals are brought to the boundary of their respective blocks.
Following this, the blocks were turned into 'Black Boxes'. This abstraction hides the internal gate-level complexity, and the blocks are now defined only by their boundaries and their I/O pins.
These two Black Boxes were then shown as two separate 'IP's (Intellectual Properties) or modules'. Block 1 is shown with 4 inputs (a, b, c, d) and 4 outputs (o1, o2, o3, o4), while Block 2 has 4 inputs and 1 output.

### Definition and Types of Pre-placed Cells:
It was explained that, similar to these user-defined blocks, other common IPs are also available in a design.
Examples of these other IPs were given, including: Memory, Clock-gating cells, Comparators, and Muxes.
The process of arranging all these IPs/blocks on the chip is formally defined as Floorplanning.
These blocks are designated as 'pre-placed cells' for a critical reason: they must be placed in the chip before the automated placement-and-routing tools run.
This is because these IPs (like memories or user-defined modules) often have user-defined locations and specific requirements that must be honored.
After these pre-placed cells are fixed, the automated placement and routing tools are then used to place all the remaining logical cells (the individual standard cells like AND, OR, etc.) into the available space on the chip.

---

## 3. De-coupling capacitors

### Context of Pre-placed Cells:
The floorplan was shown again, this time with the pre-placed Block a, Block b, and Block c fixed inside the core area, which is filled with standard cell rows (the blue horizontal lines).

### The Problem: Switching Current and Voltage Drop:
A complex circuit was considered to illustrate the need for decoupling capacitors. This circuit contained multiple flip-flops (F) driving multiple outputs (Dout1, Dout2, etc.).
It was explained that during a switching operation (e.g., when the clock signals, and the outputs change state), the circuit demands a large, instantaneous switching current, also known as peak current (Ipeak).
This current is drawn from the power supply (Vdd) through the chip's power grid. This grid is not ideal and has parasitic resistance (Rdd) and parasitic inductance (Ldd).
Due to the presence of Rdd and Ldd, a voltage drop occurs as this Ipeak is drawn.
The consequence is that the local voltage at the circuit's power pin (Node 'A') becomes Vdd', which is lower than the main power supply voltage (Vdd).

### Noise Margin and Signal Integrity:
The concept of Noise Margin was introduced to explain why this voltage drop is a problem.
A signal (measured in Volts) is only considered 'logic 0' if it is within the low noise margin (NML), specifically between Vol and Vil.
A signal is only considered 'logic 1' if it is within the high noise margin (NMH), specifically between Vih and Voh.
The region between Vil and Vih is an 'Undefined Region'. A signal in this region is not a valid logic level.
The diagrams of "noise induced bump characteristics" showed that as long as noise bumps stay within their respective margins, the logic level is correctly read.
The problem occurs when the local Vdd' drops below the Vih threshold (i.e., it falls out of the 'logic 1' range and into the 'undefined region').
If this happens, a 'logic 1' being output by one circuit will not be detected as a 'logic 1' by the input of the next circuit, leading to a functional failure of the chip.

### The Solution: Decoupling Capacitors (Decaps):
The solution to this problem was presented as the addition of Decoupling Capacitors (Decaps).
A capacitor (Cd) is added to the circuit in parallel with the switching logic, placed as close as possible to its Vdd and Vss pins.
The function of this capacitor was explained:
* When the circuit switches, it draws its high-frequency peak current directly from the local capacitor (Cd), which acts as a small, nearby charge reservoir.
* The main power supply, through the slower RL network (Rdd/Ldd), is then used to replenish the charge on the capacitor (Cd) after the switching event is complete.
This prevents the large, sudden current draw from the main power grid, thus eliminating the significant voltage drop at the local Vdd' pin and preserving the noise margin.

### Physical Implementation:
Finally, the physical implementation of this solution was shown in the floorplan.
The directive "Surround pre-placed cells with Decoupling Capacitors" was given.
The floorplan diagram was updated to show DECAP1, DECAP2, and DECAP3 cells placed in the standard cell rows, filling the gaps between and around the pre-placed blocks (Block a, b, c).

---

## 4. Power planning

### Scaling the Problem: From Local to Global:
It was stated that while decaps have taken care of local communication, a larger scenario must be considered.
A system was shown with a 'Driver' cell and a 'Load' cell connected by a long wire. This was then expanded to show the full power supply connection (Vdd, Vss) with its associated parasitic resistances and inductances (Rdd, Ldd, Rss, Lss).
It was assumed that the connection path is a 16-bit bus.

### Simultaneous Switching Noise: Ground Bounce & Voltage Droop:
The problem of a 16-bit bus was examined. An example showed a 16-bit value (11100101...) being fed into a 16-bit inverter, producing an inverted output (00011010...).
This switching event was modeled as a bank of 16 capacitors. A '1' is a capacitor charged to 'V' volts, and a '0' is a capacitor at 0 volts.
Ground Bounce was explained: When many bits switch from '1' to '0' simultaneously (as in the inverter example), all their corresponding capacitors must discharge to 0 volts. If they all discharge through a single 'Ground' tap point, this massive, sudden current rush (di/dt) through the ground wire's inductance (Lss) causes the local ground potential to rise, or "bounce," above 0V.
Voltage Droop was also explained: Conversely, when many bits switch from '0' to '1' simultaneously, all their capacitors must charge to 'V' volts. If they all draw this charging current from a single 'Vdd' tap point, it causes the local Vdd potential to drop, or "droop," significantly.

### The Solution: The Power Grid:
Both ground bounce and voltage droop are types of simultaneous switching noise that can cause functional failures.
The solution presented was to implement a power grid, which provides multiple tap points for Vdd and Vss.
A diagram showed the cells (each with its local decap, Cd) being connected to an overlapping grid of horizontal and vertical Vdd (blue) and Vss (red/grey) lines.
This grid is connected at many intersections via Contacts (yellow 'x's).
The final floorplan diagram, labeled "4) Power Planning," showed this implemented. A mesh of horizontal Vss lines (grey) and vertical Vdd lines (blue) is created across the entire core.
This grid distributes the current load, ensuring that no single tap point is overwhelmed, thus stabilizing the Vdd and Vss levels across the chip. The pre-placed blocks and decaps are shown to lie underneath this power grid.

---

## 5. Pin placement and logical cell placement blockage

### Introducing the Example Design:
An example design to be implemented was introduced, starting with two parallel data paths:
Path 1: Din1 -> FF1 -> inverter -> AND gate -> FF2 -> Dout1
Path 2: Din2 -> FF1 -> AND gate -> FF2 -> Dout2
This design was then expanded into the 'Complete design', featuring four parallel data paths (for Dout1, Dout2, Dout3, Dout4) and including the three pre-placed blocks (Block a, Block b, Block c) which are also part of the logic.
It was noted that the connectivity information for all these gates and blocks is coded using a language like VHDL or Verilog, and this complete description is the 'netlist'.

### Step 5: Pin Placement:
The floorplan diagram, which already included the core, die, pre-placed blocks, and power grid, was updated with 'Pin Placement'.
This step involves placing the I/O pins of the chip.
The diagram clearly shows all the top-level I/O ports (Din1, Din2, Din3, Din4, Clk1, Clk2, Dout1, Dout2, Dout3, Dout4, Clk Out) being placed on the periphery of the die, in the orange-striped I/O area that surrounds the core.

### Step 6: Logical Cell Placement Blockage:
The next step shown was 'Logical Cell Placement Blockage'.
In the floorplan diagram, the areas already occupied by the pre-placed cells (Block a, b, c) and the decoupling capacitors (DEC P1, P2, P3) were overlaid with an orange-hatched pattern.
This pattern represents a placement blockage or a "keep-out" zone.
This blockage serves as an instruction to the automated placement tool, forbidding it from placing any other logical cells (like the standard FFs and gates from the four data paths) in these regions, as they are already occupied.

### Readiness for Next Stage:
With the core and die defined, pre-placed cells located, power grid planned, pins placed, and blockages defined, the final statement was made: "Floor Plan is Ready for Placement & Routing Step."

---

## 6. Steps to run floorplan using OpenLANE
(No information related to specific OpenLANE commands, scripts, or Tcl console inputs for running the floorplan stage was present in the provided set of images.)

---

## 7. Review floorplan files and steps to view floorplan
(No information related to the file structure output by the floorplan stage (e.g., .def, .lef files) or the steps to view the floorplan in a GUI tool (like magic or klayout) was present in the provided set of images.)

---

## 8. Netlist binding and initial place design

### Step 1: Bind Netlist with Physical Cells:
The process was shown to start with the logical netlist (the schematic view of the "Complete design" with its four data paths).
A 'Library' was introduced. This library was shown to contain the physical layout views of all the cells used in the netlist (e.g., the physical layouts for FF1, gate 1, gate 2, FF2, and Blocks a, b, c).
The "Physical View of Logic Gates" was depicted as rows of these physical cells, ready to be placed.
The act of 'binding' is the process where each logical component in the netlist is mapped to its corresponding physical layout from the library.

### Step 2: Placement (Initial):
The "Placement" step was shown as the synthesis of three components: the Floorplan, the Netlist, and the Physical View of Logic Gates.
The floorplan (left side), complete with its power grid, I/O pins, and pre-placed blocks (DECAP1-3, Blocks a, b, c), was shown as the target.
The "initial place design" was demonstrated by showing the physical cells from the library being placed into the standard cell rows (blue lines) on the floorplan.
As an example, the row of cells FF1, 1, 2, FF2 (corresponding to the logical Dout2 data path) was explicitly shown being placed into one of the empty rows in the core. This is the initial placement, which has not yet been optimized.

---

## 9. Optimize placement using estimated wire-length and capacitance

### The Problem: Long Wires in Initial Placement:
This stage, titled "Optimize placement," was shown to begin after the initial placement.
A specific path, the Dout2 path, was highlighted with a white dotted circle in the netlist.
In the corresponding physical floorplan, the initial placement of the Dout2 cells (FF1, 1, 2, FF2) was shown to be spread out, resulting in long wire connections.
A long white arrow specifically highlighted a long-distance connection from the output of gate 1 back to the input of FF1, which is not ideal.

### The Solution: Repeater Insertion:
It was explicitly stated: "This is the stage where we estimate wire length and capacitance and, based on that, insert repeaters."
To fix the long wire issue, the diagram showed two 'Buf' (Buffer) cells being inserted into the Dout2 data path.
The cells in the physical layout were also rearranged and moved closer together to optimize the connections. The Dout2 path cells (FF1, Buf, 1, Buf, 2, FF2) are now shown in a different row, placed more compactly to minimize wire length.

---

## 10. Final placement optimization

### Applying Optimization to All Paths:
This step, "3) Optimize Placement," was shown to be an iterative process applied to all critical paths in the design, not just the Dout2 path.
Dout3 Path: The Dout3 path was highlighted in the netlist. The corresponding physical cells were shown placed in the core. White arrows indicated long wire connections that were estimated. A 'Buf' cell was shown being added to the path between gate 1 and FF2 to correct for the long wire. The cells (1, 2) were also placed in different rows to optimize.
Dout4 Path: The Dout4 path was highlighted. Again, long wires were identified (e.g., from the Din4 pin to FF1 and from gate 1 to gate 2). 'Buf' cells were inserted into the physical layout to buffer these long signals and improve timing.
The core concept was reinforced: The placement tool estimates wire length and capacitance for all paths and intelligently inserts repeaters (buffers) and rearranges cells to ensure the design will meet its timing requirements.

---

## 11. Need for libraries and characterization

### The Common Element: Gates and Cells:
It was explained that there is "One Common Thing across all stages" of the design flow: "GATES or Cells".
A 'Library' was defined as the collection of these fundamental building blocks. The examples listed included: AND gate, OR gate, BUFFER, INVERTER, DFF (D-Flip-Flop), LATCH, and ICG (Integrated Clock Gating cell).

### Libraries in the VLSI Flow:
The entire VLSI design flow was summarized visually to show how this library is essential at every single step:
* Logic Synthesis: The logical design is first created using instances of cells from the library (FFs, gates).
* Floorplanning: The floorplan is created to provide a physical space for these cells.
* Placement: The physical versions of the library cells (F, 1, 2, etc.) are placed onto the floorplan.
* CTS (Clock Tree Synthesis): A balanced clock tree is built using specific library cells, such as clock buffers and inverters (shown as triangles).
* Routing: The final step is to create the metal wire connections between all the placed library cells.

### Library Characterization and Modelling:
This led to the final, overarching topic: 'Library characterization and modelling'.
This was described as "Part 1 – Concepts and Theory."
Characterization is the process of analyzing each cell in the library to create detailed models for its timing (e.g., NLDM, CCS), power, and noise behavior.
The need for this was implicitly explained: for the optimization tool to "estimate wire length and capacitance" and "insert repeaters" (as seen in the previous topics), it must have these accurate, pre-characterized models for every cell in the library. Without characterization, optimization is not possible.
