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
source dv_env/bin/activate

# Step 3: Install SandPiper-SaaS inside the virtual environment
pip install pyyaml click sandpiper-saas

# Step 4: Convert rvmyth.tlv to Verilog
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```
<img width="806" height="509" alt="Image" src="https://github.com/user-attachments/assets/88cad2b0-90e3-4a75-aaf3-550dfae9e141" />

---

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

<img width="1483" height="692" alt="Image" src="https://github.com/user-attachments/assets/59b6efd5-c627-4895-b86f-c0eabda7a263" />

- Output: output/pre_synth_sim/pre_synth_sim.vcd (waveform if $dumpfile is used in testbench).

<img width="1473" height="692" alt="Image" src="https://github.com/user-attachments/assets/61bf176b-0e5c-42bc-b05c-f4b45061dc79" />

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


# 3️⃣ Post-Synthesis Simulation:
   - Compile synthesized netlist + standard cells with Icarus Verilog
   - Macro: -DPOST_SYNTH_SIM
   - Purpose: Verify gate-level behavior matches RTL
   - Output: output/post_synth_sim/post_synth_sim.out and waveform .vcd
```
iverilog -o output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM  -I src/include/ -I src/module/ src/module/testbench.v
```
>[!TIP] 
-Landed in error, No worry need to change the line number where the error occured just comment after `endif completely whatever it is in the library

<img width="820" height="280" alt="Image" src="https://github.com/user-attachments/assets/e82edb2d-4962-4eb1-a8cc-4ab94d10549e" />

- After solution
<img width="935" height="873" alt="Image" src="https://github.com/user-attachments/assets/b0a27d02-b301-431a-9cb9-e168556a0ffa" />

---

<img width="818" height="311" alt="Image" src="https://github.com/user-attachments/assets/c53688d2-c7fd-4ac6-8758-eec826c9e95f" />

- here your Post simutaion GLS

<img width="1002" height="442" alt="Image" src="https://github.com/user-attachments/assets/9b25b535-681b-4730-bf37-329c639c0b1e" />


# Why Pre-Synthesis and Post-Synthesis?
**1️⃣Pre-Synthesis Simulation:**

  - Focuses only on verifying functionality based on the RTL code.
  - Zero-delay environment, with events occurring on the active clock edge.

    
**2️⃣Post-Synthesis Simulation (GLS):**

  - Uses the synthesized netlist (gate-level) to simulate both functionality and timing.
  - Identifies timing violations and potential mismatches (e.g., unintended latches).
  - Helps verify dynamic circuit behavior that static methods may miss.

