## 🌟 RISC-V SoC Tapeout – Week 1

Welcome to **Week 1** of the RISC-V SoC Tapeout journey!  
This week, we focus on **Verilog RTL design, simulation using Icarus Verilog, waveform visualization with GTKWave, and logic synthesis using Yosys**.

---

## 🖥️ 1️⃣ Introduction and Simulation with Icarus Verilog (iverilog)

Before implementing hardware, **simulation verifies RTL designs**.

## 🧩 Simulator

- A **simulator** evaluates outputs based on input changes.
- RTL Design is checked for adherence to the specifications.
- ❗ No input change → No output change

## 🧩 Design

- A **Design** is a verilog code or set of verilog codes.
- It has the functionality with the required specifications.

## 🧩 Testbench (TB) Components

- **Stimulus Generator:** Provides inputs to the design.  
- **Unit Under Test (UUT):** The RTL module being verified.  
- **Stimulus Observer:** Monitors outputs from UUT.  

> **Note:** TB itself has no primary inputs or outputs.

**Simulation Workflow:**  

<div align="center">

```text
RTL Design + TB → Icarus Verilog → .vcd File → GTKWave → Waveforms
```
</div>

---

## 🛠️ 2️⃣ Lab Environment Setup

## Step 1: Create workspace

```bash
chittesh@msd-76:-$ pwd
/home/chittesh
chittesh@msd-76: $ cd vlsi
chittesh@msd-76:-/vlst$ ls
week 1
chittesh@msd-76:-/vlsi$ cd week_1
chittesh@msd-76:-/vlsi/week_1$ pwd
Π
/home/chittesh/vlsi/week_1
chittesh@msd-76:-/vl.st/week 1$ mkdir rtl_design_and_synth
chittesh@msd-76:-/vlst/week_1$ ls rtl_design_and_synth
chittesh@msd-76:-/vlst/week_1$ cd rtl_design_and_synth/
```

## Step 2: Clone Workshop Repository

```bash
chittesh@msd-76:-/vlst/week_1/rtl_design_and_synth$ git clone https://github.com /kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop/
Cloning into 'sky130RTLDesignAndSynthesisWorkshop'...
remote: Enumerating objects: 417, done.
remote: Counting objects: 100% (69/69), done.
remote: Compressing objects: 100% (52/52), done.
remote: Total 417 (delta 19), reused 47 (delta 12), pack-reused 348 (from 1)
Receiving objects: 100% (417/417), 7.79 MB | 4.16 MiB/s, done.
Resolving deltas: 100% (242/242), done.
chittesh@msd-76:-/vlsi/week_1/rtl_design_and_synth/sky130RTLDesignAndSynthesisWorkshop$
```
لنا
```
sky130RTLDesignAndSynthesisWorkshop/
├── DC_WORKSHOP
├── my_lib
│   ├── lib
│   │   └── sky130_fd_sc_hd__tt_025C_1v80.lib
│   └── verilog_model
├── verilog_files
├── yosys_run.sh
└── README.md
```

## 🔹 3️⃣ Lab 1 – Multiplexer (MUX)

## Understanding a MUX

- Combinational circuit with multiple inputs and one output.

- Number of inputs = 2^n (n = number of select lines).

>The verilog_files folder contains all standard cell Verilog models needed for labs.

**Running Simulation**
```bash
cd verilog_files
iverilog good_mux.v tb_good_mux.v
./a.out
gtkwave tb_good_mux.vcd
```
- TB has no primary I/O.
  
- Custom inputs are generated internally using stimulus blocks.

---

## ⚡ 4️⃣ Logic Synthesis with Yosys

**What is Yosys?**

- Open-source RTL synthesis tool.

- Converts behavioral RTL → gate-level netlist using .lib standard cells.

**Synthesizer - RTL to Gate-Level Mapping**

```bash
RTL Code → Yosys Synthesis → Gate-Level Netlist
```

**Example**

```bash
module good_mux (
    input i0, i1, sel,
    output reg y
);
  always @(*) begin
    if (sel)
      y = i1;
    else
      y = i0;
  end
endmodule
``` 

## 📚 4.1 Standard-Cell Libraries

- .lib files contain logic gates (AND, OR, NOT, DFF).

- Multiple flavors: Slow 🐢, Medium 🚶, Fast 🚀 to balance timing, power, and area.

**Timing & Hold Equations:**

```bash
TCLK > TCQ_A + TCOMBI + TSETUP_B    → setup time check
THOLD_B < TCQ_A + TCOMBI            → hold time check
``` 
>⚡ Faster cells → reduce delay, increase power/area.

>⚡ Slower cells → prevent hold violations, may reduce max frequency.

## 🛠️ 4.2 Synthesis Steps

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog good_mux_netlist.v
show
```
- synth -top → defines top-level module

- abc → maps RTL to standard-cell gates

- write_verilog → generates gate-level netlist

- show → visualizes synthesized netlist

---

## 📁 5️⃣ Lab Folder Structure

```bash
/home/chittesh/vlsi/week_1/
└─ rtl_design_and_synth/
   └─ sky130RTLDesignAndSynthesisWorkshop/
    ├─ DC WORKSHOP
    ├─ my_lib/
       └─ lib/ # Standard cell .lib files for synthesis
       └─ verilog_model/ # Standard-cell Verilog files
    ├─ verilog_files/ # RTL and Testbench files
    ├─ README.md
    ├─ yosys_run.sh
```
---

## ⚡ 6️⃣ Quick Reference Commands

| Task              | Command                                 |
| ----------------- | --------------------------------------- |
| Compile RTL       | `iverilog -g2012 -o simv design.v tb.v` |
| Run Simulation    | `vvp simv`                              |
| View Waveforms    | `gtkwave tb.vcd`                        |
| Start Yosys       | `yosys`                                 |
| Read Library      | `read_liberty -lib path/to/lib.lib`     |
| Synthesize RTL    | `synth -top module_name`                |
| Map to Gates      | `abc -liberty path/to/lib.lib`          |
| Generate Netlist  | `write_verilog netlist.v`               |
| Visualize Netlist | `show`                                  |

---

## ✅ 7️⃣ Key Concepts Recap

- RTL Design: Behavioral description using Verilog.

- Testbench: Verifies RTL; no primary I/O.

- Simulation: Icarus Verilog + GTKWave → waveform visualization.

- Logic Synthesis: RTL → netlist using Yosys + .lib.

- Standard Cells: Multiple flavors for timing, area, power optimization.

- Cell Selection: Guided by timing constraints for optimal trade-offs.

---

