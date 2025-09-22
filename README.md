## 🌟 RISC-V SoC Tapeout – Week-1: RTL Design Logic and Sysnthesis Using Sky130 PDK's

This repository covers **Week 1** of the RISC-V SoC Tapeout journey, focusing on **Verilog RTL design, simulation, waveform visualization, and logic synthesis**.

---

## 🎯 Learning Objectives

- Write **synthesizable Verilog RTL** designs  
- Verify designs using **Icarus Verilog** + **GTKWave**  
- Understand **testbench structure** and simulation workflow  
- Perform **logic synthesis** using **Yosys** with Sky130 PDK  
- Learn the role of **standard-cell libraries (.lib)** in timing and gate-level mapping  

---

## 📒 Day 1 - Focus Areas

- Introduction to **Verilog RTL design** and its role in digital systems.  
- Understanding **simulation workflow** using Icarus Verilog and waveform visualization with GTKWave.  
- Learning **testbench components**:  
  - **Stimulus Generator**: Provides input signals to the design.  
  - **Unit Under Test (UUT)**: The RTL module being verified.  
  - **Stimulus Observer**: Monitors outputs of the design.  
- Understanding **logic synthesis** using Yosys to convert RTL into gate-level netlists.  
- Exploring **standard-cell libraries (.lib)** and their impact on timing, power, and area optimization.

## 🧠 Key Learnings

- **RTL Design** describes circuits at the register and logic level and is synthesizable.  
- **Simulation** ensures that the design behaves correctly before moving to hardware implementation.  
- **Testbenches** do not have primary inputs or outputs but drive and observe the design internally.  
- **Logic Synthesis** maps behavioral Verilog into gates using standard cells from a library.  
- **Cell Flavors**:  
  - Fast cells → low delay, higher area/power.  
  - Slow cells → prevent hold violations, may limit maximum frequency.  
- **Timing Awareness**: Setup and hold checks ensure reliable operation across different cell types.  

---

## 🙌 Acknowledgements

- Kunal Ghosh – VSD SoC Program
- Open-source tools
  
---

👉 Main Repository Link : https://github.com/CHITTESH-S/RISC-V_SoC_TapeOut_VSD
👉 Week-0 Repository Link : https://github.com/CHITTESH-S/Week-0_RISC-V_SoC_TapeOut
