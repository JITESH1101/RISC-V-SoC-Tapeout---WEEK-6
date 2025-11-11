## 1. Introduction to Maze Routing – Lee’s algorithm
The foundational principles of maze routing were explored, specifically focusing on Lee's Algorithm, which was developed by C.Y. Lee in 1961.

* This algorithm is designed to find a path between two points, a **Source (S)** and a **Target (T)**, on a grid-based map.
* The routing area is represented as a grid of cells. This grid contains un-routable areas, or **obstacles**, which the path must navigate around. In the provided layout, these obstacles are identified as DECAP1, DECAP2, DECAP3, Block a, and Block b.
* The algorithm operates in two distinct phases: **wave expansion** (or fill) and **traceback**.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-11-04" src="https://github.com/user-attachments/assets/ad578fc4-c7e0-4f62-b04a-3cbc10673e3d" />


The first phase, **wave expansion**, was initiated. This process starts at the Source cell 'S'.
1.  All accessible, unblocked grid cells adjacent to the source are marked with the number **'1'**.
2.  This process continues iteratively, like a wave propagating outwards. All cells adjacent to the '1' cells are marked with a **'2'**, all cells adjacent to the '2's are marked with a **'3'**, and so on.
3.  This expansion wavefront propagates through the open grid cells, navigating around the defined DECAP and Block obstacles. The process continues until the wavefront reaches a cell that is adjacent to the Target 'T'.

 
---

## 2. Lee’s Algorithm Conclusion
The two phases of Lee's algorithm were completed to find the final path.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-12-45" src="https://github.com/user-attachments/assets/ad230689-c100-4ef9-85a0-eae8b4f07fe3" />

* **Wave Expansion Completion:** The wave that originated at the source 'S' was propagated until it successfully reached the target 'T'. The grid cells were filled with incrementally increasing numbers (1, 2, 3... up to 9 in the example). These numbers represent the "cost" or distance of each cell from the source. The expansion phase concludes the moment the target cell is reached.
* **Traceback Phase:** Once the target is found, the second phase, traceback, is initiated. This phase constructs the actual path.
    1.  The traceback starts from the **Target (T)**.
    2.  From the target, a path is traced backward by moving to an adjacent cell that has a numerical value **exactly one less** than the current cell.
    3.  As shown in the diagram, the path (highlighted in red) is built by stepping from 'T' to a '9', then to an '8', '7', '6', '5', '4', '3', '2', '1', and finally arriving back at the **Source (S)**.

This method guarantees that a path will be found if one exists, and the path found will be the **shortest possible path** in terms of the number of grid cells traversed.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-15-10" src="https://github.com/user-attachments/assets/72466eb8-1eb5-4870-992b-d6c632439d64" />

---

## 3. Design Rule Check (DRC)
After the routing stage, the critical verification step known as **Design Rule Check (DRC)** is performed. This step is essential to ensure the physical layout is manufacturable and reliable.

The layout, which includes I/O ports (Din1-4, Dout1-4, CLK1-2, Clk Out), flip-flops (FF1, FF2), buffers (Buf), and DECAP blocks, was checked against a set of rules.

Typical **Design Rules for metal wires** were examined. For any pair of wires, three primary rules are checked:
1.  **Wire Width:** A minimum width is enforced for every metal track. This ensures the wire is not too thin, which could lead to breaks or failures (e.g., electromigration).
2.  **Wire Pitch:** This defines the minimum allowed distance from the center of one wire to the center of an adjacent wire.
3.  **Wire Spacing:** This rule specifies the minimum clean gap or empty space required between the edges of two adjacent wires to prevent capacitive coupling and short circuits.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-21-20" src="https://github.com/user-attachments/assets/f4c52491-8c31-47df-87d9-5c1b10c443d0" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-21-25" src="https://github.com/user-attachments/assets/4d9ccbc0-2e03-4c64-8c99-703936cc8122" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-21-45" src="https://github.com/user-attachments/assets/36b6dd02-b3ab-4370-9a99-7d56498bb2a0" />


A major DRC violation, a **Signal Short**, was identified. This is a fatal error where two distinct nets (in this example, Dout3 and Clk Out) are incorrectly connected, which would cause the circuit to fail.

The checks are not limited to horizontal wires. They also apply to **vias**, which are the vertical connections between different metal layers.
* A via is used to connect, for example, a wire on Metal Layer : Mn to a wire on Metal Layer : Mn+1.
* These vias have their own set of **2 New Design Rules** that must be checked:
    1.  **Via Width:** The diameter or size of the via cut itself must meet a specific requirement.
    2.  **Via Spacing:** A minimum spacing must be maintained between adjacent vias to ensure they are distinct and do not merge.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-25-19" src="https://github.com/user-attachments/assets/a1a75bd9-41ee-4f22-94ef-5835ae9d8ea6" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-25-33" src="https://github.com/user-attachments/assets/39b64389-4398-4fdc-9e3f-c16fdc5257ba" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-25-44" src="https://github.com/user-attachments/assets/eba30d3d-b8f0-45a4-b6f8-d0102129a71b" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-25-55" src="https://github.com/user-attachments/assets/dba57c65-57a8-427c-b9af-cb0344f88954" />


Once all such violations are fixed (e.g., the "No Signal Short" status is achieved), the layout is considered **DRC Clean**.

Following a DRC-clean layout, the next step is **Parasitics Extraction**. This process analyzes the final, verified geometry to extract the unintended parasitic resistors (R) and capacitors (C) that are now part of the design. These parasitic values are critical for performing accurate timing and power analysis on the final, "as-built" circuit.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-26-16" src="https://github.com/user-attachments/assets/c2bb9e8d-d79d-4a60-aa6c-0b16e1ebe408" />

---

## 4. Basics of Global and Detail Routing and Configure TritonRoute
The overall chip **Routing** process was understood to be a two-stage approach.

1.  **Fast Route (Global Routing):** This is the first, high-level stage. It divides the entire chip layout into larger regions called **Global cells**. It then finds a rough, approximate path for each net through the "Global edges" that connect these cells. This stage does not draw the exact wires but rather allocates routing resources.
2.  **Detail Route:** This is the second, low-level stage. The **TritonRoute** tool is configured to perform this step. It takes the high-level paths from the global router as a guide.
    * The detail router's job is to find the **exact physical tracks and vias** for each net. It must implement the specific connections, like the one shown moving from point A to B, then to C (using a via to change layers), and finally to point D.
    * The 3D diagram shows how these detailed routes are implemented across multiple metal layers, with vias connecting the different segments.

---

## 5. TritonRoute Feature 1 - Honors Pre-processed Route Guides
The specific capabilities of the TritonRoute detail router were introduced.

* TritonRoute performs the initial detail route for the design.
* A primary feature is that it **honors preprocessed route guides**. These guides are the output from the "fast route" stage and define the general channels where a net should be routed. TritonRoute attempts to place the final wires within these guide-regions as much as possible.
* The router's methodology is based on a proposed MILP-based (Mixed Integer Linear Programming) panel routing scheme, which involves an **intra-layer parallel and inter-layer sequential** framework.

The route guides themselves are not used directly; they are first **preprocessed** to make them more suitable for the detail router. This preprocessing involves several transformations:
* (a) Starting with the initial route guides.
* (b) **Splitting** guides that may be complex or L-shaped.
* (c) **Merging** adjacent guides to create larger, more usable routing areas.
* (d) **Bridging** to connect nearby guides.
* (e) The result is the final preprocessed guides (shown in blue for M1 and red for M2).

These preprocessed guides must meet two key requirements:
1.  They should have **unit width**.
2.  They must be in the **preferred direction** for their specific metal layer (e.g., M1 is vertical, M2 is horizontal).

---

## 6. TritonRoute Feature 2 & 3 - Inter-guide Connectivity and Intra- & Inter-layer Routing
Deeper features of TritonRoute, focusing on connectivity and routing methodology, were examined.

<img width="3520" height="1785" alt="Screenshot from 2025-11-11 23-04-13" src="https://github.com/user-attachments/assets/41c7897b-a182-45a7-8112-6c5f6843e0df" />

* The router operates under the assumption that the preprocessed route guides for a net already satisfy **inter-guide connectivity**.
* This connectivity between two guides is defined by two conditions:
    1.  They are connected if they are on the **same metal layer** and have **touching edges**.
    2.  They are connected if they are on **neighboring metal layers** and have a **nonzero vertically overlapped area** (which allows a via to be placed).
* A critical rule for ensuring a routable design is that every unconnected terminal, such as a pin on a standard-cell, must have its **pin shape overlapped by a route guide**.

The routing method itself is called **Intra-layer parallel & Inter-layer sequential panel routing**.
This flow is executed in a specific sequence:
* (a) First, **parallel routing of panels on M2** is performed. The router works on the "Current routing panel" (red) while being aware of the "Previous routed panel" (white).
* (b) Next, **parallel routing of even-index panels on M3** is completed.
* (c) Finally, **parallel routing of odd panels on M3** is done. This sequential handling of layers (M2, then M3) and panels (even, then odd) breaks the complex problem into smaller, manageable steps.

---

## 7. TritonRoute Method to Handle Connectivity
The formal **Problem Statement** for the TritonRoute detail router was established.

* **INPUTS:** The router requires three main inputs: the **LEF file** (which defines technology rules and cell macros), the **DEF file** (which defines the design's placement and netlist), and the **preprocessed route guides** from the global router.
* **OUTPUT:** The router's goal is to produce a detailed routing solution (the final wires and vias) that is optimized for **minimal wire-length and via count**.
* **CONSTRAINTS:** The solution is not valid unless it adheres to three strict constraints: **route guide honoring**, **connectivity constraints** (all nets must be fully connected), and all **design rules (DRC)**.

To manage this, a specific method for **Handling Connectivity** is used, which relies on two definitions:
1.  **Access Point (AP):** An on-grid point on a metal layer within a route guide. It acts as a valid "port" for connecting to other segments, either on lower layers, upper layers, pins, or I/O ports.
2.  **Access Point Cluster (APC):** A collection of all the APs that belong to the same object (e.g., all the APs on a single pin, or all the APs connecting to a specific lower-layer guide).

<img width="3520" height="1785" alt="Screenshot from 2025-11-11 23-05-12" src="https://github.com/user-attachments/assets/b8b6629d-d192-4520-8d99-fbc7e03d520a" />


These concepts are shown in the illustrations:
* (a) An AP (blue dot) connects a routing track to a lower-layer segment.
* (b) An AP connects to a pin shape.
* (c) An AP connects to an upper layer, which is also described as an "Access point, aka Via (M1/M2) attached with AP".

---

## 8. Routing Topology Algorithm and Final Files List Post-route
The high-level algorithm used to determine the optimal way to connect all the pins within a single net was presented. This is **Algorithm 1: Optimization of Routing Topology**.

This algorithm is used to find the best tree-like structure to connect all the **Access Point Clusters (APCs)** that belong to a net.

1.  First, the algorithm uses nested `for` loops to iterate through every possible pair of APCs (`i` and `j`) in the net.
2.  For each pair, it calculates the distance between them and stores it as a cost:
    `cost(i, j) <- dist(APC(i), APC(j))`
3.  After all pair-wise costs are known, a **Minimum Spanning Tree (MST)** is constructed.
4.  The MST algorithm is given the set of all APCs and all the calculated COSTs. It finds the set of connections (edges, `e(i, j)`) that connects all APCs together with the **minimum possible total cost** (e.g., shortest total wire length) and without creating any loops.
5.  The resulting set of edges from the MST is returned as the final routing topology, `T`. This topology is then used as the blueprint for the detail router to implement the actual wires.

## 9. Lab task

- Perform generation of Power Distribution Network (PDN) and explore the PDN layout.

Commands to perform all necessary stages up until now

```bash
# Change directory to openlane flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'
# Since we have aliased the long command to 'docker' we can invoke the OpenLANE flow docker sub-system by just running this command
docker
```
```tcl
# Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command
./flow.tcl -interactive

# Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
package require openlane 0.9

# Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
prep -design picorv32a

# Addiitional commands to include newly added lef to openlane flow merged.lef
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Following commands are alltogather sourced in "run_floorplan" command
init_floorplan
place_io
tap_decap_or

# Now we are ready to run placement
run_placement

# Incase getting error
unset ::env(LIB_CTS)

# With placement done we are now ready to run CTS
run_cts

# Now that CTS is done we can do power distribution network
gen_pdn 
```

Screenshots of power distribution network run

![Screenshot from 2024-03-26 14-22-34](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/dd916806-6688-4c96-b1af-156b2d4acfe6)
![Screenshot from 2024-03-26 14-22-46](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/1f6ade75-93c2-4b76-bc46-77d1d532a84c)

Commands to load PDN def in magic in another terminal

```
# Change directory to path containing generated PDN def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/tmp/floorplan/

# Command to load the PDN def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read 14-pdn.def &
```

Screenshots of PDN def

![Screenshot from 2024-03-26 14-30-52](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/b13997fd-296c-4213-b4f9-8f66a7375e47)
![Screenshot from 2024-03-26 14-32-24](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/79b5f158-acf4-4065-a0ec-61007ab465d0)
![Screenshot from 2024-03-26 14-34-03](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/bee921ce-03d5-49fb-a9fc-bcc6e3402f8c)

- To Perfrom detailed routing using TritonRoute and explore the routed layout.

Command to perform routing

```tcl
# Check value of 'CURRENT_DEF'
echo $::env(CURRENT_DEF)

# Check value of 'ROUTING_STRATEGY'
echo $::env(ROUTING_STRATEGY)

# Command for detailed route using TritonRoute
run_routing
```

Screenshots of routing run

![Screenshot from 2024-03-26 14-48-29](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/f166be26-f49a-4001-abee-ce395857990f)
![Screenshot from 2024-03-26 15-38-39](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c0c8f372-0293-4fdd-a0a3-691f164e7bed)
![Screenshot from 2024-03-26 15-29-38](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/70a99289-06ea-4eb8-b3b0-4147395c6f9c)

the following are the Commands to load routed def in magic in another terminal

```
# Change directory to path containing routed def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/results/routing/

# Command to load the routed def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
```

Screenshots of routed def

![Screenshot from 2024-03-26 15-33-12](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/6eade230-eb96-4d7b-b7a9-7a7db9c2c8b7)
![Screenshot from 2024-03-26 15-30-36](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/b1900c55-7470-41b2-8b4f-3af871494d99)
![Screenshot from 2024-03-26 15-31-29](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/5faaca5b-fb6e-4abd-946d-6531b35489b8)
![Screenshot from 2024-03-26 15-32-20](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/594ef79b-4755-4934-a087-33fb24996526)

Screenshot of fast route guide present in `openlane/designs/picorv32a/runs/26-03_08-45/tmp/routing` directory

![Screenshot from 2024-03-26 15-41-18](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/1dc38a57-03c9-45c3-acdb-063731a86433)

- Post-Route parasitic extraction using SPEF extractor.

the following are the commands for SPEF extraction using external tool

```bash
# Change directory
cd Desktop/work/tools/SPEF_EXTRACTOR

# Command extract spef
python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/tmp/merged.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.def
```

- Post-Route OpenSTA timing analysis with the extracted parasitics of the route.

Commands to be run in OpenLANE flow to do OpenROAD timing analysis with integrated OpenSTA in OpenROAD

```tcl
# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/26-03_08-45/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.def

# Creating an OpenROAD database to work with
write_db pico_route.db

# Loading the created database in OpenROAD
read_db pico_route.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/synthesis/picorv32a.synthesis_preroute.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Read SPEF
read_spef /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.spef

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE flow
exit
```

Screenshots of commands run and timing report generated

![Screenshot from 2024-03-26 23-16-16](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/72ef3e8d-7ca2-4b60-89ea-de053f9c2902)
![Screenshot from 2024-03-26 23-17-09](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/5cac9ce5-420a-4eaa-b5f4-09286701e550)
![Screenshot from 2024-03-26 23-17-32](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/7d809f14-66b6-4dd6-8161-2ad8371cfaf9)
![Screenshot from 2024-03-26 23-17-56](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/64ccb1d8-74aa-42b0-88d4-a0f9588d2ca2)

