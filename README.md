# Advanced-SoC-Design-Physical-Design-Planning-using-OpenLANE-Sky130
This repository documents my hands-on journey through the 5-Day Advanced Physical Design workshop using the open-source OpenLANE flow and Google's Sky130 PDK. The project focuses on executing a complete RTL-to-GDSII flow on the picorv32a RISC-V core.

# Day 1: Open-Source EDA, OpenLANE Flow, and Sky130 PDK
## The ASIC Design Flow: RTL to GDSII
Transforming a microarchitectural description (RTL) into a final physical layout format (GDSII) requires a highly orchestrated, step-by-step pipeline.

**1.Specification & RTL Design:** Phase 1.Define the chip's features and architecture. The logic is then implemented using a Hardware Description Language (HDL) like Verilog or VHDL.

**2.Functional Verification:** Phase 2.Simulate the RTL design extensively using testbenches to verify that the logic behaves exactly as specified before touching physical components.

**3.Logic Synthesis:** Phase 3.Translate the abstract RTL code into a structural netlist of discrete logic gates (AND, OR, NOT) mapped from a target library.

**4.Floorplanning & Placement** :Phase 4.Define the physical boundaries of the chip, place the I/O pads, allocate power grids, and position the macro blocks and individual standard cells onto the silicon canvas.

**5.Clock Tree Synthesis (CTS):** Phase 5.Build a balanced distribution network for the clock signal to ensure it reaches every sequential element (flip-flops) across the chip simultaneously, minimizing clock skew.

**6.Routing:** Phase 6.Use the chip's metal layers to physically interconnect the placed standard cells and clock networks according to the structural netlist.

**7.Sign-off Checks & GDSII:** Phase 7.Run crucial physical and electrical verifications: Design Rule Checking (DRC) for manufacturing limits, Layout Versus Schematic (LVS) for structural accuracy, and Static Timing Analysis (STA) for speed requirements. Once clean, export the final GDSII file for foundry manufacturing.

## The Open-Source EDA & Hardware Ecosystem

| Tool Class | Tool Name | Core Functionality |
| :--- | :--- | :--- |
| Logic Synthesis | Yosys | RTL synthesis and RTL-to-gate mapping |
| RTL-to-GDSII Automated Flow | OpenLANE | Autonomous digital ASIC flow wrapper |
| Layout & Physical Verification | Magic | Schematic layout viewing and DRC checking |
| Static Timing Analysis | OpenSTA | Verifies design meets speed and timing constraints |
| Circuit Simulation | ngspice | SPICE engine for analog performance verification |

## The Foundation: Sky130 PDK

Distributed by SkyWater Technology, Sky130 is a fully open-source Process Design Kit (PDK). It provides the critical link to physical manufacturing by delivering open-source device models, strict layout design rules, and pre-characterized standard cell libraries targeted for a 130nm manufacturing node.

## **Implementation**

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

# Section 2 - Good floorplan vs bad floorplan and introduction to library cells

## **Theory**

Floorplanning establishes the initial physical boundaries and geometric structure of an integrated circuit. In this stage, designers define both the overall die area (the total footprint of the physical silicon chip) and the inner core area (the specific zone allocated for placing standard logic cells).

Two critical metrics guide this layout optimization:

**Aspect Ratio:** Calculated as the chip's height divided by its width ($Aspect\ Ratio = \frac{Height}{Width}$), this ratio dictates the physical shape of the die. An aspect ratio of 1.0 represents a perfect square, while other values create rectangular forms optimized for specific packaging constraints.

**Core Utilization:** Determined by dividing the total area of the active standard cells by the total available core area, this percentage measures how densely the silicon is packed.

### **Evaluating Floorplan Quality**

The success of the subsequent placement and routing phases depends heavily on the quality of the initial floorplan execution. In modern EDA flows, this step is typically initiated via automation scripts using commands like **run_floorplan**.

### **The Impact of Layout Architecture**

**Characteristics of a Highly Optimized Floorplan:** A well-executed floorplan maintains a balanced utilization rate across the die and leaves adequate channels to prevent routing congestion. By keeping critical paths physically close together, it minimizes wire lengths, optimizes signal timing, and ensures uniform power distribution.

**Consequences of a Poorly Executed Floorplan:** High local utilization leads to severe routing congestion, leaving automated tools unable to connect the logic gates. This density triggers significant voltage drops (known as IR drop issues) across the power grid and creates long wire paths that result in terminal timing violations.

### Power Planning & Cell Distribution

Once the boundaries are established, designers implement power planning to build a robust power distribution network (PDN). This grid supplies stable voltage ($V_{DD}$) and ground ($V_{SS}$) connections across the entire chip using a system of peripheral power rings, vertical and horizontal distribution straps, and standard cell rails. This structure prevents electrical instability during high-frequency operation.

Within this grid, the system places components from the standard cell libraries. These libraries consist of pre-designed, fixed-height layouts categorized into two primary types:

**Combinational Cells:** Logic gates that respond instantly to inputs, such as AND, OR, and MUX cells.

**Sequential Cells:** Memory and state-tracking elements that depend on a clock signal, such as flip-flops and latches.

## Implementation

Section 2 tasks:-

1. Run 'picorv32a' design floorplan using OpenLANE flow and generate necessary outputs.
2. Calculate the die area in microns from the values in floorplan def.
3. Load generated floorplan def in magic tool and explore the floorplan.
4. Run 'picorv32a' design congestion aware placement using OpenLANE flow and generate necessary outputs.
5. Load generated placement def in magic tool and explore the placement.

   *`die_area_microns = die_width_microns * die_height_microns`*

**1. Run 'picorv32a' design floorplan using OpenLANE flow and generate necessary outputs.**


<img width="1920" height="923" alt="Screenshot from 2026-07-12 18-53-26" src="https://github.com/user-attachments/assets/16cbc39a-cb93-4af3-ac40-675b5c5433b4" />

<img width="1920" height="923" alt="Screenshot from 2026-07-12 18-54-51" src="https://github.com/user-attachments/assets/54c424cb-2de0-4c67-b5cf-f0b8c82aca90" />


**2. Calculate the die area in microns from the values in floorplan def.**

Screenshot of contents of Floorplan def is attached below:

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 18-59-31" src="https://github.com/user-attachments/assets/8a5de35d-671d-4301-8281-9983135f5984" />

According to floorplan def

$$ 1000\ \text{Unit Distance} = 1\ \text{Micron} $$

$$ \text{Die width in unit distance} = 660685 - 0 = 660685 $$

$$ \text{Die height in unit distance} = 671405 - 0 = 671405 $$

$$ \text{Distance in microns} = \frac{\text{Value in Unit Distance}}{1000} $$

$$ \text{Die width in microns} = \frac{660685}{1000} = 660.685\ \text{Microns} $$

$$ \text{Die height in microns} = \frac{671405}{1000} = 671.405\ \text{Microns} $$

$$ \text{Area of die in microns} = 660.685 * 671.405 = 443587.212425\ \text{Square Microns} $$

**3. Load generated floorplan def in magic tool and explore the floorplan.**

Below are the commands to load floorplan def in magic in another terminal

**Change directory to path containing generated floorplan def:**
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-07_18-19/results/floorplan/

**Command to load the floorplan def in magic tool:**
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

Screenshots of floorplan def in magic

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-00-51" src="https://github.com/user-attachments/assets/1f021e07-4bd7-48b9-a78b-9d7a9bfda07b" />

Equidistant placement of ports

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-02-16" src="https://github.com/user-attachments/assets/36dd3183-facd-449f-92d3-3bd8e14bdf2f" />

Port layer as set through config.tcl

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-02-39" src="https://github.com/user-attachments/assets/846d6ce0-96cf-408e-8769-f896e0afa37b" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-03-40" src="https://github.com/user-attachments/assets/2f477f2d-68ed-4286-a6fe-9b687d6b2d1a" />

Unplaced standard cells at the origin

<img width="1920" height="923" alt="Screenshot from 2026-07-12 19-06-36" src="https://github.com/user-attachments/assets/dfe33a65-bc37-4509-9555-74c5a6bba688" />

**4. Run 'picorv32a' design congestion aware placement using OpenLANE flow and generate necessary outputs.**

**Command to run placement:** run_placement

Attached below are the screenshots of placement run

<img width="1920" height="923" alt="Screenshot from 2026-07-12 19-08-24" src="https://github.com/user-attachments/assets/952a1b0c-de53-4d01-974e-6f1ed4f25f50" />

<img width="1920" height="923" alt="Screenshot from 2026-07-12 19-09-04" src="https://github.com/user-attachments/assets/41072535-2028-4a45-9bbf-099473e1dbf8" />

**5. Load generated placement def in magic tool and explore the placement.**

**Change directory to path containing generated placement def:**
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-07_18-19/results/placement/

**Command to load the placement def in magic tool:**
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &

Attached below is the screenshot of Floorplan def in magic

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-11-15" src="https://github.com/user-attachments/assets/1540918e-4582-490c-a067-da6db309d077" />

Screenshot of legally placed Standard cells

<img width="1920" height="1080" alt="Screenshot from 2026-07-12 19-11-42" src="https://github.com/user-attachments/assets/e945ee61-c74e-4fab-b323-bcb602a917b7" />

**Commands to exit from currnt run:** exit

# Section 3 - Design library cell using Magic Layout and ngspice characterization

## Theory



























