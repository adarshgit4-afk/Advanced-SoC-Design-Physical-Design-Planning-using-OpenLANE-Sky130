# Advanced-SoC-Design-Physical-Design-Planning-using-OpenLANE-Sky130
This repository documents my hands-on journey through the 5-Day Advanced Physical Design workshop using the open-source OpenLANE flow and Google's Sky130 PDK. The project focuses on executing a complete RTL-to-GDSII flow on the picorv32a RISC-V core.

# Day 1: Open-Source EDA, OpenLANE Flow, and Sky130 PDK
# The ASIC Design Flow: RTL to GDSII
Transforming a microarchitectural description (RTL) into a final physical layout format (GDSII) requires a highly orchestrated, step-by-step pipeline.

**1.Specification & RTL Design:** Phase 1.Define the chip's features and architecture. The logic is then implemented using a Hardware Description Language (HDL) like Verilog or VHDL.

**2.Functional Verification:** Phase 2.Simulate the RTL design extensively using testbenches to verify that the logic behaves exactly as specified before touching physical components.

**3.Logic Synthesis:** Phase 3.Translate the abstract RTL code into a structural netlist of discrete logic gates (AND, OR, NOT) mapped from a target library.

**4.Floorplanning & Placement** :Phase 4.Define the physical boundaries of the chip, place the I/O pads, allocate power grids, and position the macro blocks and individual standard cells onto the silicon canvas.

**5.Clock Tree Synthesis (CTS):** Phase 5.Build a balanced distribution network for the clock signal to ensure it reaches every sequential element (flip-flops) across the chip simultaneously, minimizing clock skew.

**6.Routing:** Phase 6.Use the chip's metal layers to physically interconnect the placed standard cells and clock networks according to the structural netlist.

**7.Sign-off Checks & GDSII:** Phase 7.Run crucial physical and electrical verifications: Design Rule Checking (DRC) for manufacturing limits, Layout Versus Schematic (LVS) for structural accuracy, and Static Timing Analysis (STA) for speed requirements. Once clean, export the final GDSII file for foundry manufacturing.

# The Open-Source EDA & Hardware Ecosystem

| Tool Class | Tool Name | Core Functionality |
| :--- | :--- | :--- |
| Logic Synthesis | Yosys | RTL synthesis and RTL-to-gate mapping |
| RTL-to-GDSII Automated Flow | OpenLANE | Autonomous digital ASIC flow wrapper |
| Layout & Physical Verification | Magic | Schematic layout viewing and DRC checking |
| Static Timing Analysis | OpenSTA | Verifies design meets speed and timing constraints |
| Circuit Simulation | ngspice | SPICE engine for analog performance verification |

# The Foundation: Sky130 PDK

Distributed by SkyWater Technology, Sky130 is a fully open-source Process Design Kit (PDK). It provides the critical link to physical manufacturing by delivering open-source device models, strict layout design rules, and pre-characterized standard cell libraries targeted for a 130nm manufacturing node.

**Implementation**

Section 1 tasks:-

   1. Run 'picorv32a' design synthesis using OpenLANE flow and generate necessary outputs.
   2. Calculate the flop ratio.

   $$ \text{Flop Ratio} = \frac{\text{Number of D Flip Flops}}{\text{Total Number of Cells}} $$

$$ \text{Percentage of DFF's} = \text{Flop Ratio} * 100 $$

**1. Run 'picorv32a' design synthesis using OpenLANE flow and generate necessary outputs.**

<img width="1920" height="923" alt="Screenshot from 2026-07-12 18-30-48" src="https://github.com/user-attachments/assets/7be2645b-5ee7-4845-8b48-d5d4d2700fb4" />


<img width="1920" height="923" alt="Screenshot from 2026-07-12 18-32-43" src="https://github.com/user-attachments/assets/44d0a17b-cf0f-46b5-aa69-58d2b0b60157" />

**2. Calculate the flop ratio.**

Screenshots of synthesis statistics report file with required values attached below:

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 18-37-58" src="https://github.com/user-attachments/assets/210af7db-3f72-43ec-8246-4b0b4d9b1fdf" />

Calculation of Flop Ratio and DFF % from synthesis statistics report file

$$ \text{Flop Ratio} = \frac{1613}{14876} = 0.108429685 $$

$$ \text{Percentage of DFF's} = 0.108429685 * 100 = 10.84296854\ \% $$



