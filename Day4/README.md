## 1. Introduction to delay tables
Power-Aware Clock Tree Synthesis (CTS) was introduced. In this methodology, clock gating cells are used to selectively stop the clock signal to portions of the design that are not in use, thereby saving power.

An AND gate was examined as a simple clock gating cell. Its logic was defined by a truth table:
* When the Enable (EN) pin is '0', the output 'Y' is forced to '0', regardless of the CLK input. This successfully gates (stops) the clock.
* When the Enable (EN) pin is '1', the output 'Y' follows the CLK input (Y=CLK). This allows the clock to pass through.

A diagram of a two-level buffer tree was analyzed. This tree consists of:
* **Level 1:** A single clock buffer (Cbuf1) at node 'A'.
* **Level 2:** Two parallel clock buffers (Cbuf2) at nodes 'B' and 'C'.
* The second buffer at node 'C' was shown as part of a clock-gated path, using a clock gating cell (Cand1) with an EN pin.

Several observations were made about this balanced tree structure:
* It is a 2-level buffer tree.
* At each level, every node is driving the same load.
* Identical buffers are used at the same level (e.g., both Level 2 buffers are Cbuf2).

A set of assumptions was established for capacitance calculation:
* The load capacitance of each flip-flop (C1, C2, C3, C4) is **25fF**.
* The input capacitance of the buffer cells (Cbuf1, Cbuf2) is **30fF**.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-16-42" src="https://github.com/user-attachments/assets/45f531d2-d666-437a-aac8-8bdebb1c926c" />

Based on these assumptions, the total capacitance at each driving node was calculated:
* **Total Cap at node 'B':** Drives C1 and C2. Total Cap = 25fF + 25fF = **50fF**.
* **Total Cap at node 'C':** Drives C3 and C4. Total Cap = 25fF + 25fF = **50fF**.
* **Total Cap at node 'A':** Drives the two Cbuf2 buffers. Total Cap = 30fF + 30fF = **60fF**.

A **Delay Table** was introduced as a 2-dimensional lookup table used to characterize the delay of a standard cell. The delay of a cell is dependent on:
1.  **Input Slew** (the transition time of the input signal, in picoseconds).
2.  **Output Load** (the total capacitance the cell is driving, in femtofarads).

---

## 2. Delay table usage Part 1
The concept of using the delay tables was put into practice. Full delay tables for CBUF '1' (with delay values x1 through x24) and CBUF '2' (with delay values y1 through y24) were presented.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-32-59" src="https://github.com/user-attachments/assets/11fa3555-3d75-4932-b1ba-9f56978c084b" />

* A signal with an **input slew of 40ps** was shown to be arriving at the input of the Cbuf1 buffer (node 'A').
* The **output load at node 'A'** was previously calculated to be **60fF** (from the two Cbuf2 buffers).
* To find the delay of Cbuf1, the 'Delay Table for CBUF '1'' would be referenced. The delay value, **x9'**, is found by looking at the row for 40ps input slew and finding the corresponding value for a 60fF load.
* Since 60fF is not explicitly in the table, its value would be **interpolated** between the values for 50fF (delay x9) and 70fF (delay x10).

---

## 3. Delay table usage Part 2
The delay calculation was continued to the second level of the buffer tree.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-35-17" src="https://github.com/user-attachments/assets/1d571152-3eb1-42d2-8423-1027cac04b38" />

* It was assumed that the output signal from Cbuf1 (after propagating through it) now has a **slew of 60ps**. This 60ps slew becomes the input slew for the Cbuf2 buffers at nodes 'B' and 'C'.
* The **output load at nodes 'B' and 'C'** was previously calculated to be **50fF** (from the two flip-flops each).
* To find the delay of the Cbuf2 buffers, the 'Delay Table for CBUF '2'' is used.
* The delay value is found at the intersection of the **60ps input slew** row and the **50fF output load** column. This corresponds to the delay value **y15**.

A note, "Active only under certain conditions," was highlighted. This points to the fact that one of the Cbuf2 paths is gated, and its delay contribution is only relevant when that clock path is enabled.

---

## 4. Setup timing analysis and introduction to flip-flop setup time
A **Timing Analysis with Ideal Clocks** was performed. This analysis assumes the clock signal CLK arrives at the Launch Flop and Capture Flop at the exact same instant.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-40-40" src="https://github.com/user-attachments/assets/c7f15fcf-29cc-4376-951a-c5f14cffb5cf" />

* The basic timing path consists of a Launch Flop, a cloud of combinational logic (with delay **$\Theta$**), and a Capture Flop.
* Given system specifications **Clock Frequency (F) = 1GHz**, the **Clock Period (T)** is 1/F = **1ns**.
* For the circuit to function correctly, data launched at time 0 must travel through the logic ($\Theta$) and be captured at time T. The data must arrive before the next clock edge. The initial, simplified equation for this is **$\Theta < T$**.

The internal structure of a D-flop was shown to be composed of multiplexers. A finite amount of time, defined as **'S' (Setup Time)**, is required for the data at the 'D' input to propagate internally (e.g., to node $Q_M$) before the active clock edge arrives.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-42-33" src="https://github.com/user-attachments/assets/0658dc61-4ab4-4502-a53a-fe0283970fe8" />

* This setup time 'S' creates a small window before the capture clock edge at time T, during which the data input must be stable.
* The setup equation was refined to account for this: **$\Theta < (T - S)$**. The data must arrive before this setup window begins.
* A numerical example was provided:
    * T = 1ns
    * Assume S = 10ps (or 0.01ns)
    * The equation becomes: $\Theta < (1ns - 0.01ns)$
    * Therefore, the maximum allowed delay for the combinational logic is **$\Theta < 0.99ns$**.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-42-40" src="https://github.com/user-attachments/assets/59d9a946-7952-4c8d-8334-be818af888a9" />

---

## 5. Introduction to clock jitter and uncertainty
The concept of an "Ideal Clock" was replaced with a real-world model. On a physical chip, the clock edge will not arrive at the exact same time every cycle.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-44-06" src="https://github.com/user-attachments/assets/09a20244-0c68-42da-a0c3-22eee14541c7" />

* **Jitter** was defined as the temporary variation of the Clock period. This is represented as a "window" of time around the ideal clock edge (at 0 and T) within which the clock can arrive.
* To safely model this behavior for timing analysis, a parameter called "**Uncertainty**" (**SU** - Setup Uncertainty) is introduced.
* This uncertainty period further subtracts from the available time for data propagation. The data must now arrive stable *before* the uncertainty window, which is *before* the setup time window.
* The setup timing equation was refined again to include uncertainty: **$\Theta < (T - S - SU)$**.
* A numerical example was used to illustrate this:
    * T = 1ns
    * S = 10ps (0.01ns)
    * Uncertainty (SU) = 90ps (0.09ns)
    * The final equation becomes: $\Theta < (1ns - 0.01ns - 0.09ns)$
    * This leaves an Expected maximum logic delay of **$\Theta < 0.9ns$**.

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-44-41" src="https://github.com/user-attachments/assets/1516d8ce-143c-4aec-ab6f-a9490b73a3c8" />

This concept was then applied to a physical chip layout, where timing paths were identified.
For a path from Din1 to Dout1, the total combinational delay $\Theta$ was broken down into its constituent parts:
> $\Theta$ = FF1(CLK-Q delay) + Wire delay estimate1 + delay of cell '1' + Wire delay estimate2 + delay of cell '2' + Wire delay estimate3

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-45-02" src="https://github.com/user-attachments/assets/5e5b36c1-f444-4403-b0e2-e6c2d25e8536" />

<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-45-14" src="https://github.com/user-attachments/assets/47f2cf57-00c5-464f-a07c-232782e58dd9" />

---

## 6. Clock tree routing and buffering using H-Tree algorithm
The process of **Clock Tree Synthesis (CTS)** was examined. The primary goal of CTS is to distribute the clock signal from its source to all the sequential elements (flops) in the design.

* A key objective of CTS is to **minimize skew**, which is the difference in arrival time of the clock signal at different flops (skew = t2 - t1). Ideally, the skew should be near 0 ps.
* A "Bad Tree" example demonstrated how inefficient, unbalanced routing can lead to large differences in wire length, causing significant and unacceptable skew.
* The **H-Tree** algorithm was presented as a superior routing methodology. By building a symmetrical, balanced, 'H'-shaped structure, it ensures that the wire lengths from the clock source to all the endpoints are identical, thus minimizing skew.
  
<img width="3526" height="1985" alt="Screenshot from 2025-11-10 22-49-45" src="https://github.com/user-attachments/assets/e2645738-8d2c-4f97-b4fa-e2739c67642f" />

A clock net on a chip is not an ideal wire; it is a distributed **RC network**, with its own resistance (R) and capacitance (C). This RC network causes propagation delay and signal degradation (poor slew).
* The **Elmore delay** model (Time constant T = RC) was shown as the model for this delay.
* To combat this RC delay and maintain signal integrity, **buffers (Buf)** are inserted at strategic points within the H-Tree. These buffers regenerate the clock signal, restoring its sharpness and strength, allowing it to travel across the chip.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 22-50-56" src="https://github.com/user-attachments/assets/46dfa02f-6dcb-47d6-811a-8975f11bf7d7" />

---

## 7. Crosstalk and clock net shielding
**Crosstalk** was identified as a major signal integrity problem. It is caused by coupling capacitance ($C_M$) between adjacent nets. A signal transition on an "aggressor" net can induce a voltage spike, or **glitch**, on a parallel "victim" net.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 22-54-23" src="https://github.com/user-attachments/assets/181f63ad-e376-414a-ba87-9465c666723f" />


* A critical failure case was presented: a glitch on an aggressor net coupling onto the **Reset (RST) line** of a memory block. This spurious reset pulse can corrupt the memory, leading to incorrect data and inaccurate chip functionality.
* Crosstalk also impacts timing by causing **delta delay**. If an aggressor net switches, it can add extra delay ($\Delta$) to the victim net's signal.
* This effect was shown to directly create skew. If two clock paths (L1, L2) were perfectly balanced (Delay = D), and an aggressor net adds a delta delay ($\Delta$) to L2, its new delay becomes D + $\Delta$. This results in a skew of $\Delta$ where none existed before.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 22-56-08" src="https://github.com/user-attachments/assets/067c2db2-ae79-4e2d-815e-8a0bdc957aa4" />

The primary solution to this is **Clock Net Shielding**. This technique involves routing sensitive nets (like the clock) "shielded" between VDD (power) and VSS (ground) wires. These shield wires absorb any coupled charge from aggressors, preventing glitches and delta delays from corrupting the clock signal.

---

## 8. Setup timing analysis using real clocks
The timing analysis was advanced to use "**real clocks**", where the clock distribution network itself has delays.

* The clock signal travels through different buffer paths to reach the Launch and Capture flops.
* **$\Delta_1$** was defined as the total delay of the clock path from the source to the Launch Flop's clock pin.
* **$\Delta_2$** was defined as the total delay of the clock path from the source to the Capture Flop's clock pin.

The **Data Arrival Time (DAT)** is the time the data is launched from the first flop and arrives at the second. This is the launch clock delay plus the logic delay:
> **DAT = $\Delta_1 + \Theta$**

The **Data Required Time (DRT)** for setup is the time the data *must* arrive by. This is the capture clock edge, adjusted for setup time and uncertainty:
> **DRT = (T + $\Delta_2) - S - SU$**

The final setup equation, **DAT < DRT**, is:
> **$(\Theta + \Delta_1) < (T + \Delta_2) - S - SU$**

**Setup Slack** was defined as the margin in the timing: **Slack = DRT - DAT**. This value must be positive or zero for the design to meet setup timing.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 22-59-58" src="https://github.com/user-attachments/assets/50861bf1-94bd-4cf3-a894-47ff83a814e5" />

---

## 9. Hold timing analysis using real clocks
**Hold Analysis** was introduced as the second critical timing check. It ensures that new data from the launch flop doesn't arrive too fast, which would corrupt the *current* data being captured by the capture flop at the *same* clock edge.

* A finite time **'H' (Hold Time)** is required *after* the clock edge for the flop to securely capture its data (this is an internal delay, e.g., of Mux2).
* With an ideal clock, the logic delay must be greater than the hold time: **$\Theta > H$**.

With real clocks, the clock path delays $\Delta_1$ and $\Delta_2$ are included.
* The **Data Arrival Time (DAT)** is the same as before:
    > **DAT = $\Theta + \Delta_1$**
* The **Data Required Time (DRT)** for hold is the time the data must *not* cross. This is the capture clock edge plus the hold time and any Hold Uncertainty (HU):
    > **DRT = $\Delta_2 + H + HU$**

The final hold equation, **DAT > DRT**, is:
> **$(\Theta + \Delta_1) > (\Delta_2 + H + HU)$**

* Numerical values were provided for a hold check: H = 10ps, Uncertainty (HU) = 50ps.
* **Hold Slack** was defined as **Slack = DAT - DRT**. This value must also be positive or zero to avoid a hold violation.

Finally, the components of the clock delays were detailed. $\Delta_1$ and $\Delta_2$ are the sum of all real wire RC delays and buffer delays along their respective clock paths.
**Skew** was formally defined as the absolute difference between these clock path delays: **SKEW = |$\Delta_1 - \Delta_2$|**. This skew is a critical factor in both setup and hold calculations.

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-02-38" src="https://github.com/user-attachments/assets/1806aa5d-d65a-4d85-9a02-5b353510d0dc" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-03-27" src="https://github.com/user-attachments/assets/873aa324-c4b0-4ad0-8620-50cec2274214" />

<img width="3564" height="1985" alt="Screenshot from 2025-11-10 23-03-44" src="https://github.com/user-attachments/assets/3c14defb-2f79-4b07-a4ac-36ff1a3b76f1" />


# Lab task

FIrst go to home directory and enter the following command to open the inverter layout

```
# Change directory to vsdstdcelldesign
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```
The following picture shows the tacks.info of sky130_fd_sc_hd which is in libs (in pdks)

<img width="332" height="300" alt="tracks info" src="https://github.com/user-attachments/assets/8678bd3b-0edf-4239-9ab9-9c115714361c" />

Commands for tkcon window to set grid as tracks of locali layer

```
# Get syntax for grid command
help grid

# Set grid values accordingly
grid 0.46um 0.34um 0.23um 0.17um
```

<img width="1209" height="738" alt="magic_grid" src="https://github.com/user-attachments/assets/dc6ed0f3-96e2-4853-93d2-03fc1a5e80b0" />

<img width="869" height="311" alt="magic_grid_console" src="https://github.com/user-attachments/assets/2a910f55-a74e-4a7e-a5eb-22daddcb081f" />

Now , the 1st Condition Verified

<img width="1207" height="684" alt="1st_condition_verified" src="https://github.com/user-attachments/assets/4cfc4a21-723e-4bc7-9138-cfaaf9e8c23d" />

Condition 2 verified

<img width="1920" height="1080" alt="2nd_condition_Verified" src="https://github.com/user-attachments/assets/b03a69ac-8b1a-455a-8e40-46c7f87e0a13" />

 Horizontal track pitch = 0.46 um 

Condition 3 verified

<img width="1604" height="890" alt="3rd_condition_verified" src="https://github.com/user-attachments/assets/87d0a8da-ff82-49e0-a63c-a2b5b543cc69" />

   Vertical  track  pitch = 0.34 um 
   Height  of  standard cell = 2.72 um = 0.34 ∗ 8 

- Now save this finalized layout with a new name and open it

  ```
  # Command to save as
  save sky130_vsdinv.mag
  ```

  To open it ,

  ```
  # Command to open custom inverter layout in magic
  magic -T sky130A.tech sky130_vsdinv.mag &
  ```

  <img width="1615" height="893" alt="new_layout" src="https://github.com/user-attachments/assets/e683659f-d621-49c2-9d4c-d815e780454b" />

  - Now generate lef file from this layout
 
  ```
  # lef command
  lef write
  ```
  <img width="1199" height="724" alt="lef_Write" src="https://github.com/user-attachments/assets/f6819334-8d58-47e3-9aad-5ef1295f7c7f" />

  this is the newly created lef file

   <img width="1615" height="893" alt="new_lef" src="https://github.com/user-attachments/assets/3fcd6be5-e368-44ec-8519-a9091cc2c940" />

  Now copy this newly generated lef file , library files to src of picorv32 design

  - Update the config.tcl file change lib file and add the new extra lef into the openlane flow.
 
  Commands to be added to config.tcl to include our custom cell in the openlane flow

  ```
  set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
  set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
  set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
  set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

  set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
  ```

  <img width="1611" height="879" alt="config tcl" src="https://github.com/user-attachments/assets/4d9c86be-7c84-4527-bc24-8a067370819a" />

  - Now invoke openlane and go further with synthesis ...

   Command to invoke open lane is as follows

   ```
   # Change directory to openlane flow directory
    cd Desktop/work/tools/openlane_working_dir/openlane
   
   docker
    ```

   Now enter the following commands

   ```
   # Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command
  ./flow.tcl -interactive

  # Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
  package require openlane 0.9

  # Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
  prep -design picorv32a

  # Adiitional commands to include newly added lef to openlane flow
  set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
  add_lefs -src $lefs

  # Now that the design is prepped and ready, we can run synthesis using following command
  run_synthesis
  ```
  
   After this , synthesis will be completed, now we see the following synthesis results

   <img width="1280" height="768" alt="synthesis" src="https://github.com/user-attachments/assets/edf22bb0-0661-43c1-b857-6a27111bc7cf" />

   the observed parameters are as follows

   <img width="435" height="102" alt="tns_wns" src="https://github.com/user-attachments/assets/dc29641b-2928-49ff-ac2d-3d67b0d1dd11" />

   <img width="1112" height="668" alt="area" src="https://github.com/user-attachments/assets/2a4f5d4e-90d3-45dc-ae0d-beddfb3f00e0" />

    The following are the commands to view and change parameters to improve timing and run synthesis
  
     ```
    # Now once again we have to prep design so as to update variables
    prep -design picorv32a -tag 24-03_10-03 -overwrite

   # Addiitional commands to include newly added lef to openlane flow merged.lef
    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
   add_lefs -src $lefs

   # Command to display current value of variable SYNTH_STRATEGY
   echo $::env(SYNTH_STRATEGY)

   # Command to set new value for SYNTH_STRATEGY
   set ::env(SYNTH_STRATEGY) "DELAY 3"

   # Command to display current value of variable SYNTH_BUFFERING to check whether it's enabled
   echo $::env(SYNTH_BUFFERING)

  # Command to display current value of variable SYNTH_SIZING
  echo $::env(SYNTH_SIZING)

  # Command to set new value for SYNTH_SIZING
  set ::env(SYNTH_SIZING) 1

  # Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not
  echo $::env(SYNTH_DRIVING_CELL)

  # Now that the design is prepped and ready, we can run synthesis using following command
  run_synthesis
     ```

     the following picture shows the new lef file we created in tmp directory

   <img width="1586" height="892" alt="merged lef" src="https://github.com/user-attachments/assets/70a4a1a2-f59e-4929-a151-e23cd4027cea" />

   Now after the above commands , we see the following changes in the parameters

   <img width="1508" height="711" alt="new_Area" src="https://github.com/user-attachments/assets/a81785d6-0213-4cfc-a842-8a8540a55cd4" />

   <img width="1508" height="124" alt="new_tns_wns" src="https://github.com/user-attachments/assets/b42c421e-d9f1-4e14-b10d-d45751fca07d" />

   Now we got the optimized synthesis , we will run floorplan

    ```
    run_floorplan
    ```

    After floorplan , we need to run placement

    ```
    run_placement
    ```

    After placement , the following commands are used to open the placement result in magic tool

   ```
   # Change directory to path containing generated placement def
   cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

   # Command to load the placement def in magic tool
   magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
   ```

    <img width="1034" height="575" alt="placement" src="https://github.com/user-attachments/assets/891f4ae8-f630-4813-b2e1-438d945c15b0" />

    by zooming in , we see the inverter and buffers we placed in the design

    <img width="983" height="561" alt="inv_inserted_placement def" src="https://github.com/user-attachments/assets/efcd86c0-5019-4936-9670-858f8fa2d9cc" />

 - To view the internal layers of the cell,

    ```
    expand
    ```

    <img width="771" height="568" alt="expand" src="https://github.com/user-attachments/assets/5d1dc52b-dfc4-4311-a9c6-e6f2cd14d107" />

    <img width="964" height="544" alt="expand1" src="https://github.com/user-attachments/assets/48c53b8a-78ce-4c44-ad89-534d155ad0d8" />

     here,  the abutment of power pins with other cell from library clearly visible

     ####  Post-Synthesis timing analysis with OpenSTA tool.

Since we are having 0 wns after improved timing run we are going to do timing analysis on initial run of synthesis which has lots of violations and no parameters were added to improve timing

the following are the Commands to invoke the OpenLANE flow include new lef and perform synthesis 

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

# Adiitional commands to include newly added lef to openlane flow
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis
```

![Screenshot from 2024-03-26 05-52-18](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/790d4852-7f1d-47b5-a64f-5735f6064b61)

Newly created `pre_sta.conf` for STA analysis in `openlane` directory

![Screenshot from 2024-03-26 05-53-06](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/0a02d055-f012-44bd-a750-4508bbfc771e)

Newly created `my_base.sdc` for STA analysis in `openlane/designs/picorv32a/src` directory based on the file `openlane/scripts/base.sdc`

![Screenshot from 2024-03-26 05-55-17](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/8c917d87-5507-4079-ba6b-facbf18c238f)
![Screenshot from 2024-03-26 05-55-38](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/e07d05db-2f06-47ab-bff7-2af88c39d001)

Commands to run STA in another terminal

```bash
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Command to invoke OpenSTA tool with script
sta pre_sta.conf
```

![Screenshot from 2024-03-26 06-04-28](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/32def177-5173-4dd7-ba3b-44e37846644c)
![Screenshot from 2024-03-26 06-05-07](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/d8b8c46a-3a8b-458c-8c75-03f6577f5d17)
![Screenshot from 2024-03-26 06-05-53](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/ab396c91-430f-41ca-b9a4-dda72a91c4d5)
![Screenshot from 2024-03-26 06-08-51](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/aa1d50b0-9cb1-45bf-a641-b64842166b48)

Since more fanout is causing more delay we can add parameter to reduce fanout and do synthesis again

the following are the Commands to include new lef and perform synthesis 

```tcl
# Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
prep -design picorv32a -tag 25-03_18-52 -overwrite

# Adiitional commands to include newly added lef to openlane flow
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Command to set new value for SYNTH_MAX_FANOUT
set ::env(SYNTH_MAX_FANOUT) 4

# Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not
echo $::env(SYNTH_DRIVING_CELL)

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis
```


![Screenshot from 2024-03-26 06-20-29](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c5869cf5-ab95-46f9-9fd1-dc91e92455d8)

the following are Commands to run STA in another terminal

```bash
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Command to invoke OpenSTA tool with script
sta pre_sta.conf
```

![Screenshot from 2024-03-26 06-22-31](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/0f3247fc-aaad-4e98-b23e-bdc9a361d08c)
![Screenshot from 2024-03-26 06-22-41](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/9fcc3975-0e03-4515-aee5-04b7c1c6a315)
![Screenshot from 2024-03-26 06-22-50](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/e19db773-3995-4af4-b0a3-188b02fdf454)
![Screenshot from 2024-03-26 06-23-01](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c22c85fb-1a94-4639-a731-da4c364d1a78)

#### Make timing ECO fixes to remove all violations.

OR gate of drive strength 2 is driving 4 fanouts

![Screenshot from 2024-03-26 06-55-46](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/73c58332-a212-4a24-b853-e8cae3b385a6)

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4

```tcl
# Reports all the connections to a net
report_net -connections _11672_

# Checking command syntax
help replace_cell

# Replacing cell
replace_cell _14510_ sky130_fd_sc_hd__or3_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

Result - slack reduced

![Screenshot from 2024-03-26 07-02-44](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c50c6023-1836-468d-a8c3-955b02e4224c)
![Screenshot from 2024-03-26 07-04-08](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/6059a511-9977-4d9e-8dde-8939f0e1b8f7)
![Screenshot from 2024-03-26 09-42-15](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/df3dba3c-9445-4d51-9842-bf4410f05cf5)
![Screenshot from 2024-03-26 07-07-50](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/f141d1ff-4e21-4f9a-ab47-19579a686ac4)

OR gate of drive strength 2 is driving 4 fanouts

![Screenshot from 2024-03-26 09-46-23](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/3f66eed9-c43d-483e-ab85-21d70452b81f)

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4

```tcl
# Reports all the connections to a net
report_net -connections _11675_

# Replacing cell
replace_cell _14514_ sky130_fd_sc_hd__or3_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

Result - slack reduced

![Screenshot from 2024-03-26 09-49-29](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/d896c1fa-078e-4fe1-8e4d-60fae870d672)
![Screenshot from 2024-03-26 09-50-13](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/a21abd89-b92a-4d28-b065-6b8b523026de)
![Screenshot from 2024-03-26 09-50-33](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/9b961126-3de7-450e-9789-5009ca7f6a16)

OR gate of drive strength 2 driving OA gate has more delay

![Screenshot from 2024-03-26 10-22-10](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/7669eeef-7b93-4cfd-a152-17eec772496a)

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4

```tcl
# Reports all the connections to a net
report_net -connections _11643_

# Replacing cell
replace_cell _14481_ sky130_fd_sc_hd__or4_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

Result - slack reduced

![Screenshot from 2024-03-26 10-29-31](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c63cf6a1-582d-43e5-907f-e292b94cc086)
![Screenshot from 2024-03-26 10-29-55](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/6495c2e8-78c8-41d2-b783-628f36b982c5)

OR gate of drive strength 2 driving OA gate has more delay

![Screenshot from 2024-03-26 10-32-27](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/4185dc98-a065-49d3-a92f-fb4b02f23ee5)

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4

```tcl
# Reports all the connections to a net
report_net -connections _11668_

# Replacing cell
replace_cell _14506_ sky130_fd_sc_hd__or4_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

Result - slack reduced

![Screenshot from 2024-03-26 10-36-59](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/be71b3a4-efe1-4239-be5a-28ccebd2b7f4)
![Screenshot from 2024-03-26 10-37-14](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/2ae759b3-f9a7-478f-a34a-3f8706347eca)

Commands to verify instance `_14506_`  is replaced with `sky130_fd_sc_hd__or4_4`

```tcl
# Generating custom timing report
report_checks -from _29043_ -to _30440_ -through _14506_
```

the below is the Screenshot of replaced instance

![Screenshot from 2024-03-26 10-43-04](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/970b3cb7-fe10-4b5e-99c9-85059714f8f2)

*We started ECO fixes at wns -23.9000 and now we stand at wns -22.6173 we reduced around 1.2827 ns of violation*

#### Replace the old netlist with the new netlist generated after timing ECO fix and implement the floorplan, placement and cts.

Now to insert this updated netlist to PnR flow and we can use `write_verilog` and overwrite the synthesis netlist but before that we are going to make a copy of the old old netlist

Commands to make copy of netlist

```bash
# Change from home directory to synthesis results directory
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/

# List contents of the directory
ls

# Copy and rename the netlist
cp picorv32a.synthesis.v picorv32a.synthesis_old.v

# List contents of the directory
ls
```

![Screenshot from 2024-03-26 10-54-15](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/90c90853-0664-44d7-8ff2-573621935870)

Commands to write verilog

```tcl
# Check syntax
help write_verilog

# Overwriting current synthesis netlist
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/picorv32a.synthesis.v

# Exit from OpenSTA since timing analysis is done
exit
```

![Screenshot from 2024-03-26 11-02-19](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/c6c8ce7f-5219-41bc-a5a8-47b0e1c62c7c)

Verified that the netlist is overwritten by checking that instance `_14506_`  is replaced with `sky130_fd_sc_hd__or4_4`

![Screenshot from 2024-03-26 11-01-25](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/1ddb66da-64cb-426d-8d78-30370c63dacb)

Since we confirmed that netlist is replaced and will be loaded in PnR but since we want to follow up on the earlier 0 violation design we are continuing with the clean design to further stages

Commands load the design and run necessary stages

```tcl
# Now once again we have to prep design so as to update variables
prep -design picorv32a -tag 24-03_10-03 -overwrite

# Addiitional commands to include newly added lef to openlane flow merged.lef
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Follwing commands are alltogather sourced in "run_floorplan" command
init_floorplan
place_io
tap_decap_or

# Now we are ready to run placement
run_placement

# Incase getting error
unset ::env(LIB_CTS)

# With placement done we are now ready to run CTS
run_cts
```

![Screenshot from 2024-03-26 11-33-14](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/a0f223bc-3cbb-4bb1-8c7e-d800f1a4efd7)
![Screenshot from 2024-03-26 11-33-22](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/9bd610a8-fd5e-452b-94b2-5a5fc76db5e9)
![Screenshot from 2024-03-26 11-35-40](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/283d9b5e-fc48-45a3-8548-c68fc8966f46)
![Screenshot from 2024-03-26 11-36-16](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/af866818-af6f-4bc4-96c0-cf3c99839003)
![Screenshot from 2024-03-26 11-37-36](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/55aa5130-8a9f-48d6-abc8-7aace68b58c8)
![Screenshot from 2024-03-26 11-38-26](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/755fc79a-2a77-402b-aa11-33799e31ee09)
![Screenshot from 2024-03-26 11-39-53](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/45fdc85e-cc32-4b08-954e-bd78dcec0891)
![Screenshot from 2024-03-26 12-00-48](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/5deb6b89-c81e-4b4d-a225-badc1f7c7299)

#### Post-CTS OpenROAD timing analysis.

Commands to be run in OpenLANE flow to do OpenROAD timing analysis with integrated OpenSTA in OpenROAD

```tcl
# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Check syntax of 'report_checks' command
help report_checks

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE flow
exit
```

the following is the Screenshots of commands run and timing report generated

![Screenshot from 2024-03-26 12-55-00](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/ee26dfa7-715a-4df7-97e7-4c6e54d16522)
![Screenshot from 2024-03-26 12-57-40](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/e04a4dfd-000c-406b-bdaa-f5314c4eedef)
![Screenshot from 2024-03-26 12-58-12](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/ac27d567-1b72-444d-a638-9be2db677ae2)
![Screenshot from 2024-03-26 13-09-57](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/e63cac70-072d-4453-992e-076b94f8f1a2)

#### Explore post-CTS OpenROAD timing analysis by removing 'sky130_fd_sc_hd__clkbuf_1' cell from clock buffer list variable 'CTS_CLK_BUFFER_LIST'.

Commands to be run in OpenLANE flow to do OpenROAD timing analysis after changing `CTS_CLK_BUFFER_LIST`

```tcl
# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Removing 'sky130_fd_sc_hd__clkbuf_1' from the list
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Checking current value of 'CURRENT_DEF'
echo $::env(CURRENT_DEF)

# Setting def as placement def
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/placement/picorv32a.placement.def

# Run CTS again
run_cts

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts1.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Report hold skew
report_clock_skew -hold

# Report setup skew
report_clock_skew -setup

# Exit to OpenLANE flow
exit

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Inserting 'sky130_fd_sc_hd__clkbuf_1' to first index of list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)
```

Screenshots of commands run and timing report generated

![Screenshot from 2024-03-26 13-42-03](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/2191f13b-ff76-4281-9d3c-52cbf12f9142)
![Screenshot from 2024-03-26 13-45-28](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/3dcd6efe-d31a-4f72-aede-be1cdd7b078f)
![Screenshot from 2024-03-26 13-48-01](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/2cb2c6fd-1e8c-4921-8553-7dcfb51258e5)
![Screenshot from 2024-03-26 13-48-13](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/70353613-86f2-432c-a1d8-821083e8c209)
![Screenshot from 2024-03-26 13-50-12](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/bf6c116b-e31c-4dce-b04f-a75430b1d03b)
![Screenshot from 2024-03-26 13-53-30](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/assets/63997454/a26e9d23-d448-4512-8676-8a2b3fb22572)




























  

 
