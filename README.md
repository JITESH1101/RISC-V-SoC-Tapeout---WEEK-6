# RISC-V-SoC-Tapeout---WEEK-6

This week's learning was focused on a comprehensive, end-to-end exploration of the open-source digital ASIC design flow, from high-level architecture down to the specifics of physical layout and verification. The journey began with an introduction to the RISC-V instruction set architecture and the physical context of a QFN-48 package, bridging the gap between software applications and the hardware they run on. A complete overview of the open-source digital ASIC design ecosystem was provided, setting the stage for a deep dive into the simplified RTL2GDS flow using tools like OpenLANE. The core principles of physical design were introduced, starting with floorplanning concepts such as utilization factor, aspect ratio, pre-placed cells, and power planning strategies involving de-coupling capacitors. This foundational knowledge was then extended to cell placement, timing analysis, and the critical role of standard cell libraries.

<img width="1736" height="1042" alt="Screenshot from 2025-11-11 23-44-28" src="https://github.com/user-attachments/assets/2d0ddc9e-b510-4d1d-a9c2-25524f7b82a4" />


- Day1- Inception of open source EDA,OpenLANE and Sky130 PDK -https://github.com/JITESH1101/RISC-V-SoC-Tapeout---WEEK-6/blob/main/Day1/README.md

- Day2- Goodfloorplan vs bad floorplan and introduction to library cells - https://github.com/JITESH1101/RISC-V-SoC-Tapeout---WEEK-6/blob/main/Day2/README.md

- Day3- Design library cell using Magic Layout and ngspice characterization - https://github.com/JITESH1101/RISC-V-SoC-Tapeout---WEEK-6/blob/main/Day3/README.md

- Day4- Pre-layout timing analysis and importance of good clock tree - https://github.com/JITESH1101/RISC-V-SoC-Tapeout---WEEK-6/blob/main/Day4/README.md

- Day5- Final steps for RTL2GDS using tritonRoute and openSTA - https://github.com/JITESH1101/RISC-V-SoC-Tapeout---WEEK-6/blob/main/Day5/README.md

# Conclusion

In summary, this week's study successfully documented the entire path from a high-level RISC-V concept to a fully routed and verified physical design. A detailed examination of the OpenLANE flow was completed, covering floorplanning, placement optimization, clock tree synthesis using H-Tree algorithms, and final routing with TritonRoute. A significant portion of the learning was also dedicated to the fundamentals behind the tools: how standard cells are designed, laid out, and characterized. This included hands-on SPICE simulations of a CMOS inverter to understand timing models, propagation delays, and power consumption. The fundamental fabrication steps, from N-well formation to multi-level metal interconnects, were also covered, providing a complete picture. The process concluded with the final verification steps, including Design Rule Check (DRC) and analyzing the final routed files, solidifying the entire RTL2GDS learning experience.
With this week6 is done and week7 is coming soon
