# ⚡VSDBabySoC Simulation & Synthesis Flow LAB 
- VSDBabySoC is a small SoC including PLL, DAC, and a RISCV-based processor named RVMYTH.
- Task to simulate and check the timing waveforms and later synthesise and check the GLS with RTL.
- USING THE TOOLS LIKE iverilog and GTKwave only.

# Step by step modeling walkthrough

## Setup and Prepare Project Directory
- Clone or set up the directory structure as follows:
# 🔗 Cloning 
To begin, clone the VSDBabySoC repository using the following command:
```
$ git clone https://github.com/manili/VSDBabySoC.git
$ cd ~/vcd/VSDBabySoC/
$ ls VSDBabySoC/
images  LICENSE  Makefile  README.md  src
```
## Project Structure
- **src/include/** - Contains header files (*.vh) with necessary macros or parameter definitions.
- **src/module/** - Contains Verilog files for each module in the SoC design.
- **output/** - Directory where compiled outputs and simulation files will be generated.

```
bash

├── output
│   ├── post_synth_sim
│   │   ├── post_synth_sim.out
│   │   └── post_synth_sim.vcd
│   └── pre_synth_sim
│       ├── mkdir
│       ├── post_synth_sim.out
│       └── pre_synth_sim.out
├── dv_env
│   ├── bin
│   ├── include
│   │   ├── sandpiper_gen.vh
│   │   ├── sandpiper.vh
│   │   ├── sp_default.vh
│   │   └── sp_verilog.vh
│   ├── layout_conf
│   │   ├── rvmyth
│   │   │   ├── config.tcl
│   │   │   └── pin_order.cfg
│   │   └── vsdbabysoc
│   │       ├── config.tcl
│   │       ├── macro.cfg
│   │       └── pin_order.cfg
│   ├── lef
│   │   ├── avsddac.lef
│   │   └── avsdpll.lef
│   ├── lib
│   │   ├── avsddac.lib
│   │   ├── avsdpll.lib
│   │   └── sky130_fd_sc_hd__tt_025C_1v80.lib
│   ├── module
│   │   ├── avsddac.v
│   │   ├── avsdpll.v
│   │   ├── clk_gate.v
│   │   ├── primitives.v
│   │   ├── pseudo_rand_gen.sv
│   │   ├── pseudo_rand.sv
│   │   ├── rvmyth_gen.v
│   │   ├── rvmyth.tlv
│   │   ├── rvmyth.v
│   │   ├── sky130_fd_sc_hd.v
│   │   ├── testbench.rvmyth.post-routing.v
│   │   ├── testbench.v
│   │   ├── vsdbabysoc.synth.v
│   │   └── vsdbabysoc.v
│   ├── script
│   │   ├── sta.conf
│   │   ├── verilog_to_lib.pl
│   │   └── yosys.ys
│   └── sdc
│       ├── vsdbabysoc_layout.sdc
│       └── vsdbabysoc_synthesis.sdc
```

# TLV to Verilog Conversion for RVMYTH
Initially, you will see only the rvmyth.tlv file inside src/module/, since the RVMYTH core is written in TL-Verilog.
To convert it into a .v file for simulation, follow the steps below:
🔧 TLV to Verilog Conversion Steps

```
# Step 1: Install python3-venv (if not already installed)
sudo apt update
sudo apt install python3-venv python3-pip
cd VSDBabySoC/

# Step 2: Create and activate a virtual environment
python3 -m venv dv_env
source sp_env/bin/activate

# Step 3: Install SandPiper-SaaS inside the virtual environment
pip install pyyaml click sandpiper-saas

# Step 4: Convert rvmyth.tlv to Verilog
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

# VSDBabySoC RTL to Gate-Level Simulation Flow

# 1️⃣ Pre-Synthesis Simulation:
   - Compile RTL with Icarus Verilog
   - **Macro** requied: -DPRE_SYNTH_SIM
   - **Purpose**: Verify RTL functionality before synthesis
   - **Output**: output/pre_synth_sim/pre_synth_sim.out and waveform .vcd
 
Run the following command to perform a pre-synthesis simulation:
```
$ iverilog -o output/pre_synth_sim/pre_synth_sim.out   -DPRE_SYNTH_SIM   -I src/include   -I src/module   src/module/testbench.v
```

<img width="1920" height="482" alt="PRE_SYNTH_SCREENSHOT" src="https://github.com/user-attachments/assets/49168a05-ab4d-4c3c-adfe-49c63eb38e8c" />

```
DINESH@UBUNTU:~/Desktop/my_project/VSDBabySoC/output$ cd pre_synth_sim/
DINESH@UBUNTU:~/Desktop/my_project/VSDBabySoC/output/pre_synth_sim$ ls
post_synth_sim.out  pre_synth_sim.out
```
<img width="1917" height="906" alt="PRE_SYNTH_SCREENSHOT_1" src="https://github.com/user-attachments/assets/476f7abe-fa56-47c0-8a1c-9f80fc2059c4" />

<img width="1902" height="878" alt="PRE_SYNTH_SCREENSHOT_2" src="https://github.com/user-attachments/assets/003d7024-f521-4cb8-81b4-5ac5592db595" />

- Output: output/pre_synth_sim/pre_synth_sim.vcd (waveform if $dumpfile is used in testbench).
- This is your reference RTL behavior.

# 2️⃣ RTL Synthesis using Yosys
  - Tool: Yosys
  - Generate synthesized netlist: vsdbabysoc.synth.v
  -  Main steps:
  -  Read RTL modules
      - Synthesize top-level module
      - Write synthesized gate-level Verilog
```
DINESH@UBUNTU:~/Desktop/my_project$ cd VSDBabySoC/
DINESH@UBUNTU:~/Desktop/my_project/VSDBabySoC$ yosys
```
```
yosys> read_verilog  -sv -I src/include/ -I src/module/ src/module/vsdbabysoc.v src/module/clk_gate.v src/module/rvmyth.v

yosys> read_liberty -lib /home/DINESH/Desktop/Open_Source_EDA_Tool/yosys/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_liberty -lib src/lib/avsddac.lib
 
read_liberty -lib src/lib/avsdpll.lib

read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

write_verilog vsdbabysoc.synth.v

abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show 
```
<img width="1917" height="830" alt="image" src="https://github.com/user-attachments/assets/64c29aa6-3b43-4851-b258-ca2044340033" />

# 3️⃣ Post-Synthesis Simulation:
   - Compile synthesized netlist + standard cells with Icarus Verilog
   - Macro: -DPOST_SYNTH_SIM
   - Purpose: Verify gate-level behavior matches RTL
   - Output: output/post_synth_sim/post_synth_sim.out and waveform .vcd
```
iverilog -o output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM  -I src/include/ -I src/module/ src/module/testbench.v
```
<img width="1893" height="461" alt="POST_SYNTH_SIM" src="https://github.com/user-attachments/assets/8f1531d5-04d1-4e6b-9bfd-22833d452493" />
```
DINESH@UBUNTU:~/Desktop/my_project/VSDBabySoC/output$ cd post_synth_sim/
DINESH@UBUNTU:~/Desktop/my_project/VSDBabySoC/output/post_synth_sim$ ls
post_synth_sim.out  post_synth_sim.vcd
```
<img width="1910" height="903" alt="POST_SYNTH_SIM_2" src="https://github.com/user-attachments/assets/abfdae8c-c282-471f-bb89-62d01b9c8b80" />


# Why Pre-Synthesis and Post-Synthesis?
**1️⃣Pre-Synthesis Simulation:**

  - Focuses only on verifying functionality based on the RTL code.
  - Zero-delay environment, with events occurring on the active clock edge.

    
**2️⃣Post-Synthesis Simulation (GLS):**

  - Uses the synthesized netlist (gate-level) to simulate both functionality and timing.
  - Identifies timing violations and potential mismatches (e.g., unintended latches).
  - Helps verify dynamic circuit behavior that static methods may miss.

