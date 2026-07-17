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

### Core Logic Cells: The CMOS Inverter

The Complementary Metal-Oxide-Semiconductor (CMOS) inverter serves as the foundational element for all digital integrated circuit design. Analyzing this basic cell allows engineers to evaluate fundamental hardware parameters before scaling to larger systems.

### Layout Synthesis & Verification

**Design Metric Foundations:** Inverter analysis provides critical insights into optimal transistor sizing (adjusting the channel width-to-length ratio $W/L$), physical layout optimization techniques, and core electrical performance benchmarks.

**Geometrical Layout Production:** Engineers utilize the open-source tool Magic to draw exact physical mask layers, cross-reference geometry boundaries, and generate structural hardware representations by extracting SPICE netlists.

**Manufacturing Constraint Safeguards:** Fabrication success relies on strict adherence to foundry design rules. These mandate minimum feature widths to prevent open circuits, minimum spacing intervals between separate masks to prevent short circuits, and specific layer enclosure requirements for contact integrity.

### Performance Characterization with SPICE

Once the layout netlist is extracted, circuit simulators like ngspice execute transient and power analyses to determine the cell's real-world efficiency and operational speed.

| Performance Metric | Physical Definition |
| :--- | :--- |
| Propagation Delay | The time required for an input signal transition to cause a 50% response change at the output terminal. |
| Rise Time | The duration it takes for the output voltage to transition from 20% up to 80% of the maximum $V_{DD}$ supply. |
| Fall Time | The duration it takes for the output voltage to drop from 80% down to 20% of the maximum $V_{DD}$ supply. |

### Implementation

Section 3 tasks:-

1. Clone custom inverter standard cell design from github repository: [Standard cell design and characterization using OpenLANE flow](https://github.com/nickson-jose/vsdstdcelldesign)
2. Loading the custom inverter layout in magic.
3. Spice extraction of inverter in magic.
4. Editing the spice model file for analysis through simulation.
5. Post-layout ngspice simulations.
6. Find problem in the DRC section of the old magic tech file for the skywater process and fix them.

**1. Clone custom inverter standard cell design from github repository**

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 02-37-04" src="https://github.com/user-attachments/assets/2df603b1-cc51-4e55-b124-454dc9b0fcbe" />

**2. Loading the custom inverter layout in magic.**

Attached below is the screenshot of custom inverter layout in magic.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 03-35-09" src="https://github.com/user-attachments/assets/cc9077f5-a3d8-4b24-b514-19a1f41ec2b8" />

NMOS and PMOS are identified from the inverter layout.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-25-44" src="https://github.com/user-attachments/assets/386cc6ac-019f-4467-9db3-6224ce03fe72" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-27-19" src="https://github.com/user-attachments/assets/88d6280d-7412-4cfd-9bce-6e9f620b50e0" />

Output Y connectivity to PMOS and NMOS drain is verified.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-27-35" src="https://github.com/user-attachments/assets/37f68583-3f11-44ff-8127-db1395716291" />

PMOS source connectivity to VDD (here VPWR) is verified.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-27-42" src="https://github.com/user-attachments/assets/5a9756a2-ffe2-4718-ac05-a00ea0f84348" />

NMOS source connectivity to VSS (here VGND) is verified.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-27-49" src="https://github.com/user-attachments/assets/c986d1b3-0caa-494d-81a3-75a2fc8d0d41" />

Deleting necessary layout part to see DRC error.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 19-56-46" src="https://github.com/user-attachments/assets/67b14b7f-8c58-4331-9f69-ffa601547b22" />

**3. Spice extraction of inverter in magic.**

Attached below is the screenshot of tkcon window after running the spice extraction commands.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 21-17-19" src="https://github.com/user-attachments/assets/0b4a0594-cea6-4b98-b925-6408709cf21b" />

Attached below is the screenshot of created spice file.

<img width="1920" height="1080" alt="Screenshot from 2026-07-14 21-20-43" src="https://github.com/user-attachments/assets/bec10d81-eee5-4636-b6a5-43ab2f036162" />

**4. Editing the spice model file for analysis through simulation.**

Measuring unit distance in layout grid.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 00-51-25" src="https://github.com/user-attachments/assets/8dc2ea14-6767-4f82-9eb5-f4a158e3ef00" />

Attached below is the final edited spice file ready for ngspice simulation.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-15-52" src="https://github.com/user-attachments/assets/fc9892a3-82f8-483d-9529-ab478ad798a8" />

**5. Post-layout ngspice simulations.**

Attached below are the screenshots of ngspice run.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-16-26" src="https://github.com/user-attachments/assets/d24b7da2-ee60-420a-a977-6b70f62f0726" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-18-01" src="https://github.com/user-attachments/assets/d1d91043-2048-47ba-ad3c-f515413d2990" />

Below is the screenshot of generated plot.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-19-51" src="https://github.com/user-attachments/assets/fd3f05d4-03ee-4820-a9bf-157fb20ce7a4" />

**Rise transition time calculation**

*`Rise_transition_time = time_taken_for_output_to_rise_to_80_percent - time_taken_for_output_to_rise_to_20_percent`*

*`output_20_percent = 660_mV`*

*`output_80_percent = 2.64_V`*

20% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-36-01" src="https://github.com/user-attachments/assets/1f0f97ef-611f-4921-9cd0-1199ebe90f45" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-41-35" src="https://github.com/user-attachments/assets/92917db6-b12a-467a-891a-dc17ceb9480a" />

80% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-42-23" src="https://github.com/user-attachments/assets/cb6d6f07-3ba1-4227-8f04-ba18a73f7950" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-42-30" src="https://github.com/user-attachments/assets/2ce58ee9-7dc0-4175-a3eb-3a52a24e8fca" />

Rise Transition time = 2.24494 − 2.18181 = 0.06313 ns = 63.13 ps 

**Fall Transition time calculation**

*`Fall_transition_time = time_taken_for_output_to_fall_to_20_percent - time_taken_for_output_to_fall_to_80_percent`*

*`output_20_percent = 660_mV`*

*`output_80_percent = 2.64_V`*

20% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-47-53" src="https://github.com/user-attachments/assets/c2eaebac-ea0d-461a-9dc4-545057309169" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-47-59" src="https://github.com/user-attachments/assets/60ea0454-e163-49e8-9762-0a4eedc12dfd" />

80% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-49-06" src="https://github.com/user-attachments/assets/16d0ceef-cb89-4043-b599-a12ed6c983b3" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-49-14" src="https://github.com/user-attachments/assets/e4b32ae0-46c7-405d-b9ba-53437c9e2678" />

Fall Transition time = 4.09495 - 4.05246 = 0.04249 ns = 42.49 ps

**Rise Cell Delay Calculation**

*`Rise_Cell_delay = time_taken_for_output_to_rise_to_50_percent - time_taken_for_input_to_fall_to_50_percent`*

*`50%_of_3.3V = 1.65V`*

50% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-53-06" src="https://github.com/user-attachments/assets/3f56e36f-d4fb-457a-a7ee-c46685c86f4d" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-54-31" src="https://github.com/user-attachments/assets/c5f605d2-0b92-4870-8c13-722b3a7fd66f" />

Rise cell delay = 2.21045 − 2.15 = 0.06045 ns = 60.45 ps

**Fall Cell Delay Calculation**

*`Fall_Cell_delay = time_taken_for_output_to_fall_to_50_percent - time_taken_for_input_to_rise_to_50_percent`*

*`50%_of_3.3V = 1.65V`*

50% Screenshots

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-58-00" src="https://github.com/user-attachments/assets/11cc09c3-98c6-4e55-8b02-a22b05171577" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 01-58-10" src="https://github.com/user-attachments/assets/27bffcd9-07d7-4da8-b861-d3e92f37a27e" />

Fall cell delay = 4.07737 - 4.05 = 0.02737 ns = 27.37 ps

**6. Find problem in the DRC section of the old magic tech file for the skywater process and fix them.**

Link to Sky130 Periphery rules: [Sky 130 Periphery rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html)

Attached below is the screenshot of commands to download and view the corrupted skywater process magic tech file and associated files to perform drc corrections.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-07-44" src="https://github.com/user-attachments/assets/5dbc3052-a387-487f-8b9b-09e75ca8d459" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-07-54" src="https://github.com/user-attachments/assets/ebc4e520-eac0-48e3-8f96-5dea047f3cdc" />

Attached below is the screenshot of .magicrc file

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-08-14" src="https://github.com/user-attachments/assets/5e4d6d13-4c8c-4456-9641-b5d1b044337f" />

**Incorrectly implemented poly.9 simple rule correction**

Attached below is the screenshot of poly rules.

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-13-38" src="https://github.com/user-attachments/assets/2adf7436-71b2-47be-915d-4483dbd0df22" />

Incorrectly implemented poly.9 rule no drc violation even though spacing < 0.48u

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-33-17" src="https://github.com/user-attachments/assets/80207056-8e49-46ae-99ed-557c1fcf848e" />

New commands inserted in sky130A.tech file to update drc

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-47-24" src="https://github.com/user-attachments/assets/a8485f86-2c93-435a-8c25-70295cd1121a" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 03-51-14" src="https://github.com/user-attachments/assets/0c11e591-b3e1-42be-a25b-060e6bc52614" />

Commands to run in tkcon window

Loading updated tech file:
tech load sky130A.tech

Must re-run drc check to see updated drc errors:
drc check

Selecting region displaying the new errors and getting the error messages:
drc why

Attached below is the screenshot of magic window with rule implemented

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-35-33" src="https://github.com/user-attachments/assets/a9c233e8-dc9e-4ffe-9ef9-b67c09f4d838" />

**Incorrectly implemented difftap.2 simple rule correction**

Screenshot of difftap rules

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-37-33" src="https://github.com/user-attachments/assets/3a8ccdbb-2d21-48a6-88c5-a79d3b18e63f" />

Incorrectly implemented difftap.2 rule no drc violation even though spacing < 0.42u

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-39-03" src="https://github.com/user-attachments/assets/43db8313-12f9-4299-a7a8-2f64f8aa0aa0" />

New commands inserted in sky130A.tech file to update drc

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-43-52" src="https://github.com/user-attachments/assets/a19524a7-a520-49f0-845f-f968ac655f69" />

Attached below is the screenshot of magic window with rule implemented

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-45-34" src="https://github.com/user-attachments/assets/dad5924f-796e-4fb0-954a-17cebc68aa87" />

**Incorrectly implemented nwell.4 complex rule correction**

Attached below is the screenshot of nwell rules

<img width="1920" height="1080" alt="Screenshot from 2026-07-18 01-00-01" src="https://github.com/user-attachments/assets/14af007e-2236-41fd-922b-5d72fc436843" />


Incorrectly implemented nwell.4 rule no drc violation even though no tap present in nwell

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-52-32" src="https://github.com/user-attachments/assets/e0be9653-5a2d-422e-aaa1-4e2a96b6b6b5" />

New commands inserted in sky130A.tech file to update drc

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 04-56-50" src="https://github.com/user-attachments/assets/4dd70044-474b-473a-9b9f-d23d2af055a6" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 05-03-12" src="https://github.com/user-attachments/assets/6d62a75b-2a15-4218-aa7c-687a2c47c129" />

Attached below is the screenshot of magic window with rule implemented

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 05-14-00" src="https://github.com/user-attachments/assets/292edf4d-7635-4d9a-9b3e-0e4f4547bbce" />

# Section 3 - Design library cell using Magic Layout and ngspice characterization

## Theory

### Structural Overview: Timing Analysis & Clock Networks

Static Timing Analysis (STA) verifies that a digital circuit functions correctly at a specified clock frequency without running full dynamic simulations. This validation pipeline spans pre-layout estimations, physical distribution design, and final path verification.

### Core Timing Parameters

Timing analysis ensures data propagates reliably between sequential elements within each clock period.

**Setup Time ($T_{su}$):** The minimum duration that data must remain stable before the active clock edge arrives. Slow data paths cause setup violations.

**Hold Time ($T_h$):** The minimum duration that data must remain stable after the active clock edge arrives. Excessively fast data paths cause hold violations.

**Clock-to-Q Delay ($T_{cq}$):** The time delay between the arriving active clock edge and the resulting change at the flip-flop output terminal. This value limits subsequent logic path speed.

**Slack:** The algebraic difference between the required arrival time and the actual arrival time of a signal.

 **Positive Slack:** The signal arrived early. Timing requirements are successfully met.

 **Negative Slack:** The signal arrived late. A structural timing violation has occurred.

 ### Pre-Layout Verification vs. Clock Tree Synthesis

 Timing budgets are enforced in two separate development phases: initial structural analysis and physical implementation.

 #### Pre-Layout Timing Analysis

This phase evaluates the gate-level netlist before physical design by utilizing calculated estimations for wire delays.

**Early Isolation:** Identifies critical, slow logic paths early in the design cycle.

**RTL Evaluation:** Assesses baseline circuit performance constraints at the register-transfer level.

**Pre-emptive Optimization:** Resolves systemic logic bugs before initiating physical placement and routing steps.

**Sign-off Insurance:** Lowers the risk of encountering catastrophic timing failures late in the tape-out process.

#### Clock Tree Synthesis (CTS)

CTS constructs a balanced physical network to deliver the clock signal to every sequential element simultaneously.

**1.Buffer Insertion:**

Inject clock buffers and inverters strategically along the distribution path to maintain clean signal transitions and drive capacity.

**2.Path Balancing:**

Match the physical lengths and loading conditions of parallel clock branches to ensure concurrent signal propagation.

**3.Skew Minimization:**

Actively tune the network to reduce structural delay discrepancies across all terminating registers.

### Distribution Metrics: Skew & Latency

Two critical parameters determine the efficiency and reliability of a physical clock distribution tree.

| Variable | Engineering Definition | Architectural Impact |
| :--- | :--- | :--- |
| Clock Skew | The spatial difference in clock arrival times between any two interconnected registers. | High skew causes unexpected setup or hold violations, creating unstable logic states. |
| Clock Latency | The total absolute time delay required for a clock pulse to travel from the source pin to a target register. | Excessive insertion latency restricts maximum system operating frequency and increases dynamic power consumption. |

### The Role of CTS in Sign-off

Successful Clock Tree Synthesis directly improves timing closure. By minimizing skew and reducing layout-induced setup and hold violations, it guarantees stable, high-speed synchronous chip operations.

## Implementation

**1. Fix up small DRC errors and verify the design is ready to be inserted into our flow.**

Conditions to be verified before moving forward with custom designed cell layout:

**Condition 1:** The input and output ports of the standard cell should lie on the intersection of the vertical and horizontal tracks.

**Condition 2:** Width of the standard cell should be odd multiples of the horizontal track pitch.

**Condition 3:** Height of the standard cell should be even multiples of the vertical track pitch.

Below are the commands to open the custom inverter layout

**Change directory to vsdstdcelldesign:**
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign

**Command to open custom inverter layout in magic:**
magic -T sky130A.tech sky130_inv.mag &

Attached below is the screenshot of tracks.info of sky130_fd_sc_hd

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-11-43" src="https://github.com/user-attachments/assets/076c85fc-e629-4539-8c04-20ea42df6130" />

Commands for tkcon window to set grid as tracks of locali layer

**Get syntax for grid command:**
help grid

**Set grid values accordingly:**
grid 0.46um 0.34um 0.23um 0.17um

Screenshot of commands been run

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-14-38" src="https://github.com/user-attachments/assets/dba4617a-8d32-42dc-93b9-d83b952ce407" />

Condition 1 is verified

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-14-46" src="https://github.com/user-attachments/assets/05c03b87-be43-4bfd-b89e-ba6c883afb31" />

Condition 2 is verified

Horizontal Track Pitch = 0.46 um

Width of Standard cell = 1.38 um = 0.46 ∗ 3

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-24-19" src="https://github.com/user-attachments/assets/b74dcf88-5a53-4932-ac13-5ba14e85fa53" />

Condition 3 is verified

Vertical Track Pitch = 0.46 um

Height of Standard cell = 2.72 um = 0.34 ∗ 8

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-25-55" src="https://github.com/user-attachments/assets/7044f208-9a8c-4a4c-9f1f-d6b55445cd58" />

**2. Save the finalized layout with custom name and open it.**

Command for tkcon window to save the layout with custom name:
save sky130_vsdinv.mag

Command to open the newly saved layout in magic:
magic -T sky130A.tech sky130_vsdinv.mag &

Attached below is the screenshot of newly saved layout

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-28-50" src="https://github.com/user-attachments/assets/4c934df3-ad35-4995-bc2d-6d4b47d1709e" />

**3. Generate lef from the layout.**

Command for tkcon window to write lef: lef write

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-29-55" src="https://github.com/user-attachments/assets/ac13df98-ea03-4c0b-9c33-ef9c3a8b1966" />

Attached below is the screenshot of newly created lef file

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-30-55" src="https://github.com/user-attachments/assets/dfe80180-3923-4b2c-91cb-fbd1864990b0" />

**4. Copy the newly generated lef and associated required lib files to 'picorv32a' design 'src' directory.**

Commands to copy necessary files to 'picorv32a' design 'src' directory:

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-41-39" src="https://github.com/user-attachments/assets/bd193cc7-60a9-4c54-9a08-981efc4ef7db" />

**5. Edit 'config.tcl' to change lib file and add the new extra lef into the openlane flow.**

Below are the commands to be added to config.tcl to include our custom cell in the openlane flow:

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"

set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"

set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

Attached below is the screenshot of config.tcl after adding above commands

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-47-14" src="https://github.com/user-attachments/assets/f937278a-5918-42ed-b326-1906504c9684" />

**6. Run openlane flow synthesis with newly inserted custom inverter cell.**

Commands to invoke the OpenLANE flow include new lef and perform synthesis:

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-51-19" src="https://github.com/user-attachments/assets/774a064a-b4f4-46a2-8c73-5fa8b507736f" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-15 23-52-45" src="https://github.com/user-attachments/assets/d103a94a-cb3f-4797-b022-3c0bb4767f38" />

**7. Remove/reduce the newly introduced violations with the introduction of custom inverter cell by modifying design parameters.**

Noting down current design values generated before modifying parameters to improve timing

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-09-37" src="https://github.com/user-attachments/assets/930263e7-57b9-4835-bcfb-84d4bafe7d71" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-09-46" src="https://github.com/user-attachments/assets/e5704cbc-0a46-411d-b578-3a7785733ec5" />

Screenshot of merged.lef in tmp directory with our custom inverter as macro

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-39-30" src="https://github.com/user-attachments/assets/8ef37d08-6c14-4f60-a192-cf1bca084659" />

Commands to view and change parameters to improve timing and run synthesis:

**Now once again we have to prep design so as to update variables:**
prep -design picorv32a -tag 15-07_18-19 -overwrite

**Addiitional commands to include newly added lef to openlane flow merged.lef:**
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

**Command to display current value of variable SYNTH_STRATEGY:**
echo $::env(SYNTH_STRATEGY)

**Command to set new value for SYNTH_STRATEGY:**
set ::env(SYNTH_STRATEGY) "DELAY 3"

**Command to display current value of variable SYNTH_BUFFERING to check whether it's enabled:**
echo $::env(SYNTH_BUFFERING)

**Command to display current value of variable SYNTH_SIZING:**
echo $::env(SYNTH_SIZING)

**Command to set new value for SYNTH_SIZING:**
set ::env(SYNTH_SIZING) 1

**Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not:**
echo $::env(SYNTH_DRIVING_CELL)

**Now that the design is prepped and ready, we can run synthesis using following command:**
run_synthesis

Attached below is the screenshots of above commands been run

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-44-21" src="https://github.com/user-attachments/assets/6da41a4f-b311-47fb-b6b0-90e6cec56757" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-46-55" src="https://github.com/user-attachments/assets/be42048d-be62-444b-9472-a57245106890" />

Comparing to previously noted run values area has increased and worst negative slack has become 0

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-47-25" src="https://github.com/user-attachments/assets/b9024fb8-62c6-4259-8f5a-8e4fd087f43e" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-47-33" src="https://github.com/user-attachments/assets/915aac52-a3a4-43bd-ba30-2b6dedbffa08" />

**8. Once synthesis has accepted our custom inverter we can now run floorplan and placement and verify the cell is accepted in PnR flow.**

Now that our custom inverter is properly accepted in synthesis we can now run floorplan using following command: run_floorplan

But due to an unexpected error, we instead use these three commands for Floorplan:

**init_floorplan**

**place_io**

**tap_decap_or**

based on floorplan Commands section in **Desktop/work/tools/openlane_working_dir/openlane/docs/source/OpenLANE_commands.md**

Attached below are the screenshots of above commands been run

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-57-24" src="https://github.com/user-attachments/assets/861b1f21-65a3-478c-9823-3320db36a30a" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-58-22" src="https://github.com/user-attachments/assets/c5e3f83a-08fd-45b9-9136-1d4c6255b359" />

Now that floorplan is done we can do placement using following command: run_placement

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 00-58-42" src="https://github.com/user-attachments/assets/d898e039-73b9-451a-8e6b-15a050ad2882" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-03-22" src="https://github.com/user-attachments/assets/02cb05eb-5fcf-4606-b6a6-66a57d8b5b76" />

Below are the commands to load placement def in magic in another terminal:

**Change directory to path containing generated placement def:**
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-07_18-19/results/placement/

**Command to load the placement def in magic tool:**
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &

Attached below is the screenshot of placement def in magic

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-08-34" src="https://github.com/user-attachments/assets/f9aab9d6-d67b-4c5a-b601-54cdb0007acb" />

Screenshot of custom inverter inserted in placement def with proper abutment

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-12-02" src="https://github.com/user-attachments/assets/b7dcd763-7cde-4ce8-9811-b4218bf32628" />

Command for tkcon window to view internal layers of cells: expand

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-13-25" src="https://github.com/user-attachments/assets/d5ef03e7-b1a6-4dd9-8eac-466e9d57fe1f" />

**9. Do Post-Synthesis timing analysis with OpenSTA tool.**

Since we are having 0 wns after improved timing run we are going to do timing analysis on initial run of synthesis which has lots of violations and no parameters were added to improve timing

**Change directory to openlane flow directory:**
cd Desktop/work/tools/openlane_working_dir/openlane

**Since we have aliased the long command to 'docker' we can invoke the OpenLANE flow docker sub-system by just running this command:**
docker

**Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command:**
./flow.tcl -interactive

**Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow:**
package require openlane 0.9

**Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a':**
prep -design picorv32a

**Adiitional commands to include newly added lef to openlane flow:**
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

**Command to set new value for SYNTH_SIZING:**
set ::env(SYNTH_SIZING) 1

**Now that the design is prepped and ready, we can run synthesis using following command:**
run_synthesis

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-26-41" src="https://github.com/user-attachments/assets/aba48f84-a177-4d5b-9393-12e5ca6d3591" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 01-27-57" src="https://github.com/user-attachments/assets/6e6eeee2-2521-4610-aac1-56ca9e8f064b" />

Newly created pre_sta.conf for STA analysis in openlane directory

<img width="1920" height="1080" alt="Screenshot from 2026-07-18 02-11-21" src="https://github.com/user-attachments/assets/148ccc04-64cc-46df-91d0-7d02f08d9895" />

Newly created my_base.sdc for STA analysis in openlane/designs/picorv32a/src directory based on the file openlane/scripts/base.sdc

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-05-22" src="https://github.com/user-attachments/assets/813dad0d-092b-4355-a7f6-89bf95ef0a24" />

Below are the commands to run STA in another terminal:

**Change directory to openlane:**
cd Desktop/work/tools/openlane_working_dir/openlane

**Command to invoke OpenSTA tool with script:**
sta pre_sta.conf

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-08-15" src="https://github.com/user-attachments/assets/5092762f-a902-49e9-bd53-a22d5883f0b5" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-08-24" src="https://github.com/user-attachments/assets/12d2ef93-7bcd-428e-9af1-2f1d351d68ef" />

Since more fanout is causing more delay we can add parameter to reduce fanout and do synthesis again

Commands to include new lef and perform synthesis:

**Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a':**
prep -design picorv32a -tag 15-07_18-19 -overwrite

**Adiitional commands to include newly added lef to openlane flow:**
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

**Command to set new value for SYNTH_SIZING:**
set ::env(SYNTH_SIZING) 1

**Command to set new value for SYNTH_MAX_FANOUT:**
set ::env(SYNTH_MAX_FANOUT) 4

**Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not:**
echo $::env(SYNTH_DRIVING_CELL)

**Now that the design is prepped and ready, we can run synthesis using following command:**
run_synthesis

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-17-46" src="https://github.com/user-attachments/assets/cb5b2803-9390-4da7-9f37-3cd29291c16a" />

Command to run STA in another terminal: sta pre_sta.conf

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-20-38" src="https://github.com/user-attachments/assets/6cd53b45-a14a-4f7e-b570-073e17e839c9" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-20-43" src="https://github.com/user-attachments/assets/724b9880-306e-4f27-a3e6-2f0d39532ced" />

**10. Make timing ECO fixes to remove all violations.**

OR gate of drive strength 2 is driving 4 fanouts

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-22-28" src="https://github.com/user-attachments/assets/e45c8b11-b22d-4920-a4b6-15f9d93365b7" />

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4:

**Reports all the connections to a net:**
report_net -connections _11672_

**Checking command syntax:**
help replace_cell

**Replacing cell:**
replace_cell _14510_ sky130_fd_sc_hd__or3_4

**Generating custom timing report:**
report_checks -fields {net cap slew input_pins} -digits 4

Result - Slack reduced

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-26-31" src="https://github.com/user-attachments/assets/cb13fa41-509e-4fce-8ccf-564b89a3f9c4" />

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-29-38" src="https://github.com/user-attachments/assets/bd1d34c8-3547-4ea6-838a-41ded1114170" />

OR gate of drive strength 2 is driving 4 fanouts

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-33-34" src="https://github.com/user-attachments/assets/74172ab5-3956-41aa-810d-03ea76ac39da" />

Result - Slack reduced

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-36-14" src="https://github.com/user-attachments/assets/13870520-a1d8-491c-b2d1-767bcc136918" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-36-36" src="https://github.com/user-attachments/assets/4cb77609-97e4-4fd5-bbe0-92603a9dc86e" />

OR gate of drive strength 2 driving OA gate has more delay

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-39-16" src="https://github.com/user-attachments/assets/9c5d9506-bace-44b8-9b9a-cc04a8cf3b8b" />

Commands to perform analysis and optimize timing by replacing with OR gate of drive strength 4:

**Reports all the connections to a net:**
report_net -connections _11643_

**Replacing cell:**
replace_cell _14481_ sky130_fd_sc_hd__or4_4

**Generating custom timing report:**
report_checks -fields {net cap slew input_pins} -digits 4

Result - Slack reduced

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-41-40" src="https://github.com/user-attachments/assets/78de701b-7e94-48d0-bdbf-17f655b091e8" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-41-52" src="https://github.com/user-attachments/assets/36af4dba-41ae-4476-b5d5-58e363d632a3" />

OR gate of drive strength 2 driving OA gate has more delay

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-45-25" src="https://github.com/user-attachments/assets/de14c56d-4272-4ca0-836f-3b6306f57bfe" />

Result - Slack reduced

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-47-05" src="https://github.com/user-attachments/assets/51569a63-690c-4a86-8efc-6bbb32369f55" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-47-31" src="https://github.com/user-attachments/assets/4effee62-39ed-457b-b76b-ea2c7a7706c8" />

Command to verify instance _14506_ is replaced with sky130_fd_sc_hd__or4_4: report_checks -from _29043_ -to _30440_ -through _14506_

Attached below is the screenshot of replaced instance

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 02-51-08" src="https://github.com/user-attachments/assets/5194a4a4-4bbe-4a26-875c-5679710dc98b" />

We started ECO fixes at wns -23.9000 and now we stand at wns -22.6173 we reduced around 1.2827 ns of violation.

**11. Replace the old netlist with the new netlist generated after timing ECO fix and implement the floorplan, placement and cts.**

Now to insert this updated netlist to PnR flow and we can use write_verilog and overwrite the synthesis netlist but before that we are going to make a copy of the old old netlist

Commands to make copy of netlist:

**Change from home directory to synthesis results directory:**
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-07_18-19/results/synthesis/

**List contents of the directory:**
ls -ltr

**Copy and rename the netlist:**
cp picorv32a.synthesis.v picorv32a.synthesis_old.v

**List contents of the directory:**
ls -ltr

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-06-50" src="https://github.com/user-attachments/assets/62f6cc00-57d3-4b87-897c-4a69ce58973f" />

Commands to write verilog:

**Overwriting current synthesis netlist:**
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-07_18-19/results/synthesis/picorv32a.synthesis.v

# Exit from OpenSTA since timing analysis is done
exit

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-19-53" src="https://github.com/user-attachments/assets/177bb8a3-6e9f-4608-92d2-0ac20b465a8a" />

Verified that the netlist is overwritten by checking that instance _14506_ is replaced with sky130_fd_sc_hd__or4_4

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-23-42" src="https://github.com/user-attachments/assets/c92624b6-e770-4336-8b7b-bb86a5fc8035" />

Since we confirmed that netlist is replaced and will be loaded in PnR but since we want to follow up on the earlier 0 violation design we are continuing with the clean design to further stages

Commands load the design and run necessary stages:

**Now once again we have to prep design so as to update variables:**
prep -design picorv32a -tag 15-07_18-19 -overwrite

**Addiitional commands to include newly added lef to openlane flow merged.lef:**
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

**Command to set new value for SYNTH_STRATEGY:**
set ::env(SYNTH_STRATEGY) "DELAY 3"

**Command to set new value for SYNTH_SIZING:**
set ::env(SYNTH_SIZING) 1

**Now that the design is prepped and ready, we can run synthesis using following command:**
run_synthesis

**Follwing commands are alltogather sourced in "run_floorplan" command:**
init_floorplan
place_io
tap_decap_or

**Now we are ready to run placement:**
run_placement

**With placement done we are now ready to run CTS:**
run_cts

<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-30-02" src="https://github.com/user-attachments/assets/917acec2-3027-4230-92bf-35528ecca39b" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-31-44" src="https://github.com/user-attachments/assets/7e6a9b40-7bcd-4ebc-83e9-1023cee9a3d6" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-34-07" src="https://github.com/user-attachments/assets/6f7cee3f-2fea-4bdd-8833-4c5b1ad3d530" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 03-34-23" src="https://github.com/user-attachments/assets/bfc1c43b-db07-4b63-a696-d2663cba2874" />
<img width="1920" height="1080" alt="Screenshot from 2026-07-16 13-21-39" src="https://github.com/user-attachments/assets/393f39ec-aa7b-4305-b395-4ca5587c0ee5" />

















































































