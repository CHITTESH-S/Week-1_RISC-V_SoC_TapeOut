# 🌟 RISC-V SoC Tapeout – Week-1: RTL Design Logic and Sysnthesis Using Sky130 PDK's

This repository covers **Week 1** of the RISC-V SoC Tapeout journey, focusing on **Verilog RTL design, simulation, waveform visualization, and logic synthesis**.

---

## 🎯 Day 1 – Learning Objectives
- 📝 Write **synthesizable Verilog RTL designs**  
- 🧪 Verify designs using **Icarus Verilog + GTKWave**  
- 🏗️ Understand **testbench structure** and simulation workflow  
- ⚡ Perform **logic synthesis using Yosys** with Sky130 PDK  
- 📚 Learn the role of **standard-cell libraries (.lib)** in timing and gate-level mapping  

---

## 📒 Day 1 – Focus Areas
- 💻 **Introduction to Verilog RTL**: role in digital systems  
- 🖥️ **Simulation workflow**: Icarus Verilog for simulation, GTKWave for waveform visualization  
- 🧩 **Testbench components**:  
  - **Stimulus Generator**: provides input signals to the design  
  - **Unit Under Test (UUT)**: the RTL module being verified  
  - **Stimulus Observer**: monitors outputs of the design  
- 🔧 **Logic synthesis using Yosys**: converts RTL into gate-level netlists  
- 🏭 **Standard-cell libraries (.lib)**: impact on timing, power, and area optimization  

---

## 🧠 Key Learnings
- 🖊️ **RTL Design** describes circuits at the register and logic level and is synthesizable  
- 🧪 **Simulation** ensures correct design behavior before hardware implementation  
- 📦 **Testbenches**: no primary inputs/outputs, drive and observe the design internally  
- ⚡ **Logic Synthesis**: maps behavioral Verilog to gates using standard cells  
- 🏎️ **Cell Flavors**:  
  - ⚡ Fast cells → low delay, higher area/power  
  - 🐢 Slow cells → prevent hold violations, may limit maximum frequency  
- ⏱️ **Timing Awareness**: setup and hold checks ensure reliable operation across different cell types  

---

>✨ **Summary Insight**  
>Day-1 built the **foundation of RTL design, simulation, and synthesis**, introducing the role of testbenches, functional verification, and standard-cell libraries. It emphasized **timing-aware design** and understanding cell trade-offs, forming the basis for more advanced synthesis and timing optimization in subsequent days.

---

## 🎯 Day 2 – Learning Objectives
- 📂 Understand the role and structure of **Liberty (.lib) timing libraries**  
- 🔄 Compare **hierarchical vs flat synthesis** flows using Yosys + Sky130 PDK  
- 🧩 Practice **submodule synthesis** for IP reuse  
- ⏱️ Explore **flip-flop coding styles** (sync vs async reset, set, enable)  
- ⚡ Learn **optimizations** (e.g., multiplications by powers of 2 → shift logic)  

---

## 📒 Day 2 – Focus Areas
- 📘 **Liberty (.lib) basics**: timing arcs, setup/hold, clk→Q, leakage/dynamic power, area, PVT corners  
- 🏗️ **Hierarchical synthesis**: preserve module boundaries → IP reuse, faster incremental builds  
- 🏭 **Flat synthesis**: global optimization → better timing/area at cost of readability  
- 🔧 **Submodule synthesis**: compile leaf blocks independently → divide & conquer large designs  
- ⏰ **Flip-flop fundamentals**: stability, glitch elimination, clocked storage, reset/set behavior  
- 🌀 **Interesting optimization**: multiplication by 2, 4, 8 synthesized as shift-left wiring  

---

## 🧠 Key Learnings
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

>✨ **Summary Insight**  
>Day-2 strengthened timing awareness by exploring **.lib files**, highlighted **hierarchical vs flat design trade-offs**, and deepened understanding of **flops + RTL optimizations**, paving the way toward efficient and reliable SoC designs.

---

## 🎯 Day 3 – Learning Objectives  
⚡ Master combinational logic optimizations (constant propagation, Boolean simplification)  
🔁 Apply sequential optimizations (constant flops removal, retiming, register cloning, FSM pruning)  
🗑 Prune unused outputs to save area, power, and complexity  
⚙️ Use Yosys optimization flows (opt_clean, abc, dfflibmap) and compare netlist stats  

---

## 📒 Day 3 – Focus Areas  
🧮 **Combinational**: constant folding, Boolean algebra optimizations, common subexpression elimination  
🔂 **Sequential**: eliminate redundant flip-flops, retime pipelines, duplicate registers for critical paths  
📉 **Unused Outputs**: remove signals/FFs that don’t impact functional outputs  
🛠 **Yosys Flow Practice**: run baseline vs optimized flows, use `stat` and `show` to compare  

---

## 🧠 Key Learnings  
📚 Optimizations at combinational and sequential levels reduce gate count, streamline logic, and boost performance  
⚖️ Retiming and register cloning help meet timing at area cost tradeoffs  
🗑 Removing unused logic is a “low-hanging fruit” for area/power savings  
🔍 Using synthesis + optimization tools reveals how your RTL style impacts final netlist  

---

> ✨ **Summary Insight**  
>On Day 3, we deepened our ability to **turn good RTL into better gate-level design**. By applying combinational and sequential optimizations and eliminating unused logic, you see firsthand how coding discipline + synthesis strategy directly affect area, timing, and power—crucial groundwork for clean, efficient RISC-V SoC tapeout.  

---

## 🎯 Day 4 – Learning Objectives
🧩 Understand Gate-Level Simulation (GLS) and why it’s needed  
⚡ Differentiate blocking (`=`) vs non-blocking (`<=`) assignments in Verilog  
⚠️ Identify and prevent synthesis–simulation mismatches  
🧪 Validate RTL vs synthesized netlist behavior through labs  

---

## 📒 Day 4 – Focus Areas
🏗️ **GLS Flow**: RTL → Synthesized Netlist → iverilog → VCD → GTKWave  
🔍 **Mismatch Pitfalls**: non-synth constructs, bad sensitivity lists, wrong assignment usage  
⚡ **Blocking vs Non-Blocking**: coding style → determines correctness of hardware mapping  
🧪 **Hands-On**: MUX examples, blocking caveats, and mismatch debugging  

---

## 🧠 Key Learnings
📚 GLS ensures synthesis preserved functionality and timing  
⚠️ Common mismatch sources:  
- Missing `@(*)` → latch inference  
- Using `initial` / `#delay` → not synthesizable  
- Mixed blocking/non-blocking → simulation vs hardware misalignments  
⚖️ Correct coding discipline = fewer surprises at GLS  
🔁 Blocking → sequential execution, good for **combinational** logic  
⏳ Non-blocking → concurrent scheduling, good for **sequential** logic  

---

> ✨ **Summary Insight**  
> Day-4 emphasized the **bridge between RTL and hardware reality**. By running GLS, exploring blocking vs non-blocking, and debugging synthesis–simulation mismatches, we learned that **coding style directly dictates silicon correctness**. These practices are vital for reliable SoC design and avoiding late-stage surprises.  

---

## 🎯 Day 5 – Learning Objectives
🔎 Understand how common Verilog constructs map to hardware after synthesis  
⚡ Identify pitfalls that cause unintended latches or inefficient logic  
✅ Learn synthesis-friendly `if/else` and `case` coding patterns  
🔁 Distinguish procedural `for` (unrolled behavior) vs `generate` (structural replication)  
🛡️ Apply techniques that avoid synthesis–simulation mismatches and produce predictable nets  

---

## 📒 Day 5 – Focus Areas
🔀 **If / else**: priority logic vs complete combinational specification — how missing `else` causes latch inference  
🎚️ **Case statements**: parallel mux-like selection, need `default` and full coverage; beware overlaps & partial assignments  
🔁 **Looping constructs**: `for` inside `always` ⇒ behavioral unrolling; `generate` ⇒ replicated instances (scalable structural design)  
🧪 **Labs**: incomplete-if, incomplete-case, overlapping-case, blocking-assignment caveat, generate vs procedural loop comparisons  
⚙️ **Synthesis mapping**: inspect netlist (`show`), use `stat` and `abc/dfflibmap` to observe hardware translation

---

## 🧠 Key Learnings
📝 Every Verilog construct implies real hardware — write with hardware intent in mind  
⚠️ Incomplete combinational specifications produce latches (unwanted state) — always assign outputs on every branch or use `default`  
📊 Case statements synthesize into clean multiplexers when fully specified; overlapping/wildcard cases are ambiguous and risky  
🔁 Procedural `for` loops are unrolled — leads to wide combinational logic; `generate` creates reusable structural instances and is preferable for parameterized hardware  
🔍 Good RTL practices (complete branches, consistent assignments, clear reset semantics) drastically reduce synthesis surprises and ease verification

---

> ✨ **Summary Insight**  
> Day-5 reinforced the core truth: **Verilog is a hardware description language**, and synthesis tools faithfully implement what you write — including mistakes. Mastering `if`/`case` patterns, understanding loop semantics, and using structural `generate` constructs lead to predictable, efficient hardware. These practices are essential to avoid late-stage issues and produce clean, tapeout-ready netlists.

---

## 🙌 Acknowledgements  

👨‍🏫 **Kunal Ghosh** – VSD SoC Program Mentor  

🧑‍🤝‍🧑 **Open-source EDA Community** – Tool contributors & maintainers  

🌍 **RISC-V & Efabless Ecosystem** – Enabling open tapeout programs  

---

## 🔗 References & Resources

💡 I sincerely thank all the organizations and their key members for making this program possible:

- 🧑‍🏫 **VLSI System Design (VSD)** – [Kunal Ghosh](https://www.linkedin.com/in/kunal-ghosh-vlsisystemdesign-com-28084836/) for mentorship and vision.
- 🤝 **Efabless** – [Michael Wishart](https://www.linkedin.com/in/mike-wishart-81480612/) & [Mohamed Kassem](https://www.linkedin.com/in/mkkassem/) for enabling collaborative open-source chip design.
- 🏭 [**Semiconductor Laboratory (SCL)**](https://www.scl.gov.in/) – for PDK & foundry support.
- 🎓 [**IIT Gandhinagar (IITGN)**](https://www.linkedin.com/school/indian-institute-of-technology-gandhinagar-iitgn-/?originalSubdomain=in) – for on-site training & project facilitation.
- 🛠️ **Synopsys** – [Sassine Ghazi](https://www.linkedin.com/in/sassine-ghazi/) for providing industry-grade EDA tools under C2S program.

---

## 🔗 Repository Links  

👉 **Week-0 Repository Link:** https://github.com/CHITTESH-S/Week-0_RISC-V_SoC_TapeOut

👉 **Week-2 Repository Link:** https://github.com/CHITTESH-S/Week-2_RISC-V_SoC_TapeOut

👉 **Main Repository Link:** https://github.com/CHITTESH-S/RISC-V_SoC_TapeOut_VSD

👨‍💻 **Contributor**: [Chittesh S](https://github.com/CHITTESH-S)  

---
