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

* A signal with an **input slew of 40ps** was shown to be arriving at the input of the Cbuf1 buffer (node 'A').
* The **output load at node 'A'** was previously calculated to be **60fF** (from the two Cbuf2 buffers).
* To find the delay of Cbuf1, the 'Delay Table for CBUF '1'' would be referenced. The delay value, **x9'**, is found by looking at the row for 40ps input slew and finding the corresponding value for a 60fF load.
* Since 60fF is not explicitly in the table, its value would be **interpolated** between the values for 50fF (delay x9) and 70fF (delay x10).

---

## 3. Delay table usage Part 2
The delay calculation was continued to the second level of the buffer tree.

* It was assumed that the output signal from Cbuf1 (after propagating through it) now has a **slew of 60ps**. This 60ps slew becomes the input slew for the Cbuf2 buffers at nodes 'B' and 'C'.
* The **output load at nodes 'B' and 'C'** was previously calculated to be **50fF** (from the two flip-flops each).
* To find the delay of the Cbuf2 buffers, the 'Delay Table for CBUF '2'' is used.
* The delay value is found at the intersection of the **60ps input slew** row and the **50fF output load** column. This corresponds to the delay value **y15**.

A note, "Active only under certain conditions," was highlighted. This points to the fact that one of the Cbuf2 paths is gated, and its delay contribution is only relevant when that clock path is enabled.

---

## 4. Setup timing analysis and introduction to flip-flop setup time
A **Timing Analysis with Ideal Clocks** was performed. This analysis assumes the clock signal CLK arrives at the Launch Flop and Capture Flop at the exact same instant.

* The basic timing path consists of a Launch Flop, a cloud of combinational logic (with delay **$\Theta$**), and a Capture Flop.
* Given system specifications **Clock Frequency (F) = 1GHz**, the **Clock Period (T)** is 1/F = **1ns**.
* For the circuit to function correctly, data launched at time 0 must travel through the logic ($\Theta$) and be captured at time T. The data must arrive before the next clock edge. The initial, simplified equation for this is **$\Theta < T$**.

The internal structure of a D-flop was shown to be composed of multiplexers. A finite amount of time, defined as **'S' (Setup Time)**, is required for the data at the 'D' input to propagate internally (e.g., to node $Q_M$) before the active clock edge arrives.

* This setup time 'S' creates a small window before the capture clock edge at time T, during which the data input must be stable.
* The setup equation was refined to account for this: **$\Theta < (T - S)$**. The data must arrive before this setup window begins.
* A numerical example was provided:
    * T = 1ns
    * Assume S = 10ps (or 0.01ns)
    * The equation becomes: $\Theta < (1ns - 0.01ns)$
    * Therefore, the maximum allowed delay for the combinational logic is **$\Theta < 0.99ns$**.

---

## 5. Introduction to clock jitter and uncertainty
The concept of an "Ideal Clock" was replaced with a real-world model. On a physical chip, the clock edge will not arrive at the exact same time every cycle.

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

This concept was then applied to a physical chip layout, where timing paths were identified.
For a path from Din1 to Dout1, the total combinational delay $\Theta$ was broken down into its constituent parts:
> $\Theta$ = FF1(CLK-Q delay) + Wire delay estimate1 + delay of cell '1' + Wire delay estimate2 + delay of cell '2' + Wire delay estimate3

---

## 6. Clock tree routing and buffering using H-Tree algorithm
The process of **Clock Tree Synthesis (CTS)** was examined. The primary goal of CTS is to distribute the clock signal from its source to all the sequential elements (flops) in the design.

* A key objective of CTS is to **minimize skew**, which is the difference in arrival time of the clock signal at different flops (skew = t2 - t1). Ideally, the skew should be near 0 ps.
* A "Bad Tree" example demonstrated how inefficient, unbalanced routing can lead to large differences in wire length, causing significant and unacceptable skew.
* The **H-Tree** algorithm was presented as a superior routing methodology. By building a symmetrical, balanced, 'H'-shaped structure, it ensures that the wire lengths from the clock source to all the endpoints are identical, thus minimizing skew.

A clock net on a chip is not an ideal wire; it is a distributed **RC network**, with its own resistance (R) and capacitance (C). This RC network causes propagation delay and signal degradation (poor slew).
* The **Elmore delay** model (Time constant T = RC) was shown as the model for this delay.
* To combat this RC delay and maintain signal integrity, **buffers (Buf)** are inserted at strategic points within the H-Tree. These buffers regenerate the clock signal, restoring its sharpness and strength, allowing it to travel across the chip.

---

## 7. Crosstalk and clock net shielding
**Crosstalk** was identified as a major signal integrity problem. It is caused by coupling capacitance ($C_M$) between adjacent nets. A signal transition on an "aggressor" net can induce a voltage spike, or **glitch**, on a parallel "victim" net.

* A critical failure case was presented: a glitch on an aggressor net coupling onto the **Reset (RST) line** of a memory block. This spurious reset pulse can corrupt the memory, leading to incorrect data and inaccurate chip functionality.
* Crosstalk also impacts timing by causing **delta delay**. If an aggressor net switches, it can add extra delay ($\Delta$) to the victim net's signal.
* This effect was shown to directly create skew. If two clock paths (L1, L2) were perfectly balanced (Delay = D), and an aggressor net adds a delta delay ($\Delta$) to L2, its new delay becomes D + $\Delta$. This results in a skew of $\Delta$ where none existed before.

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
