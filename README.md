<h1 align="center">🔳 RISC-V SoC Tapeout Program — Week 3️⃣</h1>

<p align="center"><img src="./ASSETS/0.1.jpeg" width="500" alt="image 0.1"/></p>

---

<div align="center">

# 🚀 Week 3 —  Post Synthesis GLS & STA Fundamentals

🌟 

</div>

---

<details>
<summary><h2> 🌟 TASK1-Post Synthesis GLS </h2> </summary>


# 🧠 Week 3 – Post-Synthesis Simulation of VSDBabySoC (GLS)

After completing RTL-level verification in Week 2, the next milestone is to **verify the functionality of the synthesized (gate-level) design**.

👉Post-synthesis simulation (also called **Gate-Level Simulation – GLS**) is one of the **most important steps** in the ASIC design flow. After synthesis, our BabySoC RTL is converted into a **gate-level netlist** using the Sky130 standard cell library. The goal here is simple:

👉 To ensure that the design **still works exactly as intended** after synthesis, and to check if there are **any timing-related issues** that were not visible in pre-synthesis simulation.

---

## 🔨 Purpose of Gate-Level Simulation (GLS)

The **main objectives** of GLS for BabySoC are:

1. **Functionality + Timing Verification**
    - Checks whether the **gate-level netlist** still behaves as the RTL design.
    - Uses **SDF (Standard Delay Format)** for accurate timing verification.
2. **Dynamic Circuit Behavior**
    - Captures real-world issues like **glitches** or **metastability** which are invisible in RTL.
3. **Post-Synthesis Validation**
    - Confirms that modules like the **RISC-V core, PLL, and DAC** are all mapped correctly to standard cells.
    - Ensures there are **no unexpected latches, mismatches, or synthesis-induced bugs**.
4. **Final Check Before PnR**
    - This is the **last chance** to catch functional/timing problems **before moving to Physical Design (PnR)**.

![0.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/0.png)

---

## 🎯 Objective

1. Perform **logic synthesis** on the BabySoC design using Yosys.
2. Generate a **gate-level netlist** that represents the same behavior as the RTL.
3. Run a **post-synthesis simulation** using Icarus Verilog.
4. Compare **pre- and post-synthesis** simulation waveforms to confirm functional equivalence.

---

## ⚙️Step 1 –Environment Setup

Before moving into synthesis, verify that your environment is ready:

```bash
sudo apt update
sudo apt install yosys iverilog gtkwave
```

🧩 **Tools we’ll use:**

- **Yosys** – Open-source synthesis tool
- **Icarus Verilog** – Simulator for gate-level testing
- **GTKWave** – Waveform viewer

---

## 🗂️Step 2 –Organizing Directories

To keep the workflow structured, create dedicated folders inside the `output/` directory.

```bash
mkdir -p output/synth output/post_synth_sim
```

Your updated directory tree should now look like this 👇

```bash
output/
├── pre_synth_sim/
│   ├── pre_synth_sim.out
│   ├── vsdbabysoc.synth.v
│   └── pre_synth_sim.vcd
└── post_synth_sim/
```

This separation makes debugging and file tracking much easier.

---

## 🧰 Step 3 –Synthesis with Yosys

- The synthesis process converts RTL code into →  equivalent gate-level representation (netlist).

### 1️⃣ Launch Yosys

```bash
yosys
```

![1.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/1.png)

---

### 2️⃣ Read all verilog files

```bash
```
# Load RTL files
yosys> read_verilog src/module/vsdbabysoc.v
yosys> read_verilog -I src/include src/module/rvmyth.v
yosys> read_verilog -I src/include src/module/clk_gate.v
```

![2.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/2.png)

---

### **3️⃣ Load the Liberty Files for Synthesis**

```bash
# Load library files
yosys> read_liberty -lib src/lib/avsdpll.lib
yosys> read_liberty -lib src/lib/avsddac.lib
yosys> read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

![3.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/3.png)

---

### **4️⃣ Run Synthesis Targeting `vsdbabysoc`**

```bash
yosys> synth -top vsdbabysoc
```

![4.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/4.png)

---

### 5️⃣Statistics of Yosys Synthesis

![5.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/5.png)

![6.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/6.png)

![7.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/7.png)

![8.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/8.png)

![9.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/9.png)

---

### **6️⃣ Map D Flip-Flops to Standard Cells**

```bash
yosys> dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

![10.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/10.png)

---

### **7️⃣ Perform Optimization and Technology Mapping**

```bash
yosys> opt
yosys> abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```

![11.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/11.png)

![12.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/12.png)

---

### **8️⃣ Perform Final Clean-Up and Renaming**

```bash
yosys> flatten
yosys> setundef -zero
yosys> clean -purge
yosys> rename -enumerate
```

![13.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/13.png)

---

### **9️⃣ Check Statistics**

```bash
yosys> stat
```

![14.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/14.png)

---

### **🔟 Write the Synthesized Netlist**

```bash
yosys> write_verilog -noattr output/post_synth_sim/vsdbabysoc.synth.v
```

![15.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/15.png)

✅ **Output:** `synth_netlist.v` – the synthesized gate-level version of our BabySoC core.

```bash
yosys> cd output/post_synth_sim/
```

![16.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/16.png)

---

## 🧪 Step 4 – Post-Synthesis Simulation

📌 Before going to next check your directory, that should now look like this 👇

```bash
output/
├── pre_synth_sim/
│   ├── pre_synth_sim.out
│   ├── vsdbabysoc.synth.v
│   └── pre_synth_sim.vcd
├── post_synth_sim/
    ├── vsdbabysoc.synth.v
    └── post_synth_sim.out
```

Now that we have the gate-level netlist, we must verify that its behavior matches the RTL simulation results.

We use the **same testbench** (`tb_mythcore_test.v`), replacing the RTL design with the synthesized netlist.

### 🔹 Compile the Gate-Level Design

```bash
iverilog -o /home/maddy/Desktop/open_source_tapout/VLSI/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/maddy/Desktop/open_source_tapout/VLSI/VSDBabySoC/src/include -I /home/maddy/Desktop/open_source_tapout/VLSI/VSDBabySoC/src/module /home/maddy/Desktop/open_source_tapout/VLSI/VSDBabySoC/src/module/testbench.v
```
🔹 Post-Synthesis Simulation – Command Breakdown
iverilog \
  -o output/post_synth_sim/post_synth_sim.out \   # Output simulation executable
  -DPOST_SYNTH_SIM \                              # Enable POST_SYNTH_SIM mode
  -DFUNCTIONAL \                                  # Use behavioral models
  -DUNIT_DELAY=#1 \                               # Assign #1 delay for all gates
  -I src/include \                                # Include path for header files
  -I src/module \                                 # Include path for modules
  src/module/testbench.v                          # Top-level testbench


Explanation of Options:

🔹 iverilog → Icarus Verilog compiler to convert Verilog into an executable.

🔹 -o <path> → Output binary path for the simulation.

🔹 -DPOST_SYNTH_SIM → Switch testbench to post-synthesis simulation mode.

🔹 -DFUNCTIONAL → Use high-level behavioral models instead of gate timing.

🔹 -DUNIT_DELAY=#1 → Assigns a unit delay of #1 for all gates.

🔹 -I <include_path> → Add include directories for modules or headers.

🔹 testbench.v → Specifies the testbench as the top-level simulation file.

### 🔹 Run the Simulation

```bash
cd output/post_synth_sim
./post_synth_sim.out
```

This will produce a new `.vcd` file (waveform dump).

![17.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/17.png)

### 🔹 Visualize the Waveform by GTKwave

```bash
gtkwave tb_mythcore_test_post.vcd
```

Observe signal transitions, clock gating, and output behavior.

![18.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/18.png)

---

## 🔬 Step 5 – Result Analysis

When you compare **pre- and post-synthesis waveforms** in GTKWave (`tb_mythcore_test.vcd` vs `tb_mythcore_test_post.vcd`):

- The **functional behavior** should be identical.
- Slight **timing variations** may appear because the synthesized design includes gate delays.
- Successful matching confirms that **Yosys synthesis preserved the RTL logic**.

---

## 🧩 Step 6 – Conclusion

🎉 **Post-synthesis verification completed!**

| Stage | Tool | Output | Verification |
| --- | --- | --- | --- |
| RTL Simulation | Icarus Verilog | `tb_mythcore_test.vcd` | Functional correctness |
| Synthesis | Yosys | `synth_netlist.v` | Logical equivalence |
| Gate-Level Simulation | Icarus Verilog | `tb_mythcore_test_post.vcd` |  |

---

## **Comparing Pre-Synthesis ⚡VS⚡ Post-Synthesis Output**

Why because the matching outputs between pre- and post-synthesis simulations mean the synthesis process has preserved your design’s functionality, while now adding real-world timing considerations. 

### 🔹 Pre-Synthesis **⚡** Post-Synthesis

| Aspect | Pre-Synthesis | Post-Synthesis |
| --- | --- | --- |
| **Purpose** | Verify RTL logic & functionality | Verify gate-level design & timing |
| **Focus** | Logical correctness, design intent | Gate delays, timing violations, glitches |
| **Simulation** | Behavioral, fast | Gate-level, includes timing info |
| **Outcome** | Confirms RTL works as intended | Confirms synthesized design behaves correctly in real-world conditions |

📌  Pre-Synthesis output

![19.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/19.png)

📌 Post-Synthesis output

![20.png](week3%202805f99c9dcb80e48e4ee8a3457c6f65/20.png)

</details>
