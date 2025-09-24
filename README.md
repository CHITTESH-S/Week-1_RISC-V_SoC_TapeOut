## 🌟 RISC-V SoC Tapeout – Week-1: RTL Design Logic and Sysnthesis Using Sky130 PDK's

This repository covers **Week 1** of the RISC-V SoC Tapeout journey, focusing on **Verilog RTL design, simulation, waveform visualization, and logic synthesis**.

---

# 🎯 Day 1 – Learning Objectives
- 📝 Write **synthesizable Verilog RTL designs**  
- 🧪 Verify designs using **Icarus Verilog + GTKWave**  
- 🏗️ Understand **testbench structure** and simulation workflow  
- ⚡ Perform **logic synthesis using Yosys** with Sky130 PDK  
- 📚 Learn the role of **standard-cell libraries (.lib)** in timing and gate-level mapping  

---

# 📒 Day 1 – Focus Areas
- 💻 **Introduction to Verilog RTL**: role in digital systems  
- 🖥️ **Simulation workflow**: Icarus Verilog for simulation, GTKWave for waveform visualization  
- 🧩 **Testbench components**:  
  - **Stimulus Generator**: provides input signals to the design  
  - **Unit Under Test (UUT)**: the RTL module being verified  
  - **Stimulus Observer**: monitors outputs of the design  
- 🔧 **Logic synthesis using Yosys**: converts RTL into gate-level netlists  
- 🏭 **Standard-cell libraries (.lib)**: impact on timing, power, and area optimization  

---

# 🧠 Key Learnings
- 🖊️ **RTL Design** describes circuits at the register and logic level and is synthesizable  
- 🧪 **Simulation** ensures correct design behavior before hardware implementation  
- 📦 **Testbenches**: no primary inputs/outputs, drive and observe the design internally  
- ⚡ **Logic Synthesis**: maps behavioral Verilog to gates using standard cells  
- 🏎️ **Cell Flavors**:  
  - ⚡ Fast cells → low delay, higher area/power  
  - 🐢 Slow cells → prevent hold violations, may limit maximum frequency  
- ⏱️ **Timing Awareness**: setup and hold checks ensure reliable operation across different cell types  

---

✨ **Summary Insight**  
Day-1 built the **foundation of RTL design, simulation, and synthesis**, introducing the role of testbenches, functional verification, and standard-cell libraries. It emphasized **timing-aware design** and understanding cell trade-offs, forming the basis for more advanced synthesis and timing optimization in subsequent days.

---

# 🎯 Day 2 – Learning Objectives
- 📂 Understand the role and structure of **Liberty (.lib) timing libraries**  
- 🔄 Compare **hierarchical vs flat synthesis** flows using Yosys + Sky130 PDK  
- 🧩 Practice **submodule synthesis** for IP reuse  
- ⏱️ Explore **flip-flop coding styles** (sync vs async reset, set, enable)  
- ⚡ Learn **optimizations** (e.g., multiplications by powers of 2 → shift logic)  

---

# 📒 Day 2 – Focus Areas
- 📘 **Liberty (.lib) basics**: timing arcs, setup/hold, clk→Q, leakage/dynamic power, area, PVT corners  
- 🏗️ **Hierarchical synthesis**: preserve module boundaries → IP reuse, faster incremental builds  
- 🏭 **Flat synthesis**: global optimization → better timing/area at cost of readability  
- 🔧 **Submodule synthesis**: compile leaf blocks independently → divide & conquer large designs  
- ⏰ **Flip-flop fundamentals**: stability, glitch elimination, clocked storage, reset/set behavior  
- 🌀 **Interesting optimization**: multiplication by 2, 4, 8 synthesized as shift-left wiring  

---

# 🧠 Key Learnings
- 📚 `.lib` = the *timing/power/area truth* for synthesis/STA; `.v` = functional model  
- 🌡️ PVT corners + multiple drive strengths → trade speed vs power/area  
- ⚖️ **Hier vs Flat**:  
  - ✅ Hier → readable, reusable, incremental  
  - ✅ Flat → globally optimized, but heavier to debug  
- ⏳ Flops = predictable operation: sample only on clock edges, block glitches, enable pipelining  
- 🔁 **Resets/sets**:  
  - ⏩ Async → immediate, clock-independent  
  - ⏳ Sync → controlled, clock-dependent  
- 🧮 Synthesis optimizes power-of-two multiplications into **bit shifts**, saving hardware  

---

✨ **Summary Insight**  
Day-2 strengthened timing awareness by exploring **.lib files**, highlighted **hierarchical vs flat design trade-offs**, and deepened understanding of **flops + RTL optimizations**, paving the way toward efficient and reliable SoC designs.

---

## 🙌 Acknowledgements

- Kunal Ghosh – VSD SoC Program
- Open-source tools
  
---

👉 **Week-0 Repository Link:** https://github.com/CHITTESH-S/Week-0_RISC-V_SoC_TapeOut

👉 **Week-1 Repository Link:** https://github.com/CHITTESH-S/Week-1_RISC-V_SoC_TapeOut

👉 **Main Repository Link:** https://github.com/CHITTESH-S/RISC-V_SoC_TapeOut_VSD

👨‍💻 **Contributor:** Chittesh S
