# ⚡ Power-Aware INT8 Systolic Array Accelerator


## 📌 Overview
This repository contains a highly optimized, sparsity-aware 4x4 INT8 Systolic Array designed specifically to minimize dynamic power consumption during matrix multiplication workloads. 

Unlike standard matrix multipliers, this architecture explicitly attacks both combinational toggling and sequential clock-tree power waste. It features real-time, hardware-level performance monitoring and is fully verified for synthesis and power extraction in EDA tools like Cadence Genus.

## 🚀 Architectural Innovations

To maximize power efficiency ($P_{dynamic} = \alpha \cdot C \cdot V_{DD}^2 \cdot f$), this design implements aggressive RTL-level gating techniques at both the micro and macro levels:

* **Operand Isolation (Combinational Power Saver):** Each Processing Element (PE) continuously monitors incoming data. [cite_start]If a zero is detected on either the activation or weight input, the PE triggers a `skip_mac` flag and clamps the inputs to the combinational multiplier to absolute zero[cite: 4, 5]. [cite_start]This physically prevents the internal logic gates from toggling, completely eliminating wasted active power[cite: 4].
* **Integrated Clock Gating / ICG (Sequential Power Saver):** The PE is split into two clock domains. [cite_start]The data-forwarding registers run on a continuous clock to maintain the systolic wavefront[cite: 10, 11]. [cite_start]However, the heavy MAC accumulator registers are placed behind a glitch-free Integrated Clock Gating latch[cite: 8, 9, 12]. [cite_start]When sparse data enters, the clock signal to the accumulator flatlines, cutting sequential power to zero[cite: 12].
* [cite_start]**Global Sleep Mode (Macro Power Saver):** The top-level module includes a master `global_en` switch[cite: 33]. [cite_start]When the array finishes its workload, this switch disables the master clock gate, putting all 16 PEs into a deep sleep state to prevent power drain between tasks[cite: 37, 38].
* **Global Performance Monitor:** The design does not just save power; it proves it. [cite_start]A centralized hardware monitor aggregates the `skip_flag` and `reuse_flag` signals from all 16 PEs[cite: 36, 44, 45]. [cite_start]It tracks exact zero-skips and temporal reuses in real-time, outputting the totals to `global_skip_count` and `global_reuse_count` registers[cite: 46, 47].

## 📂 Repository Structure

* [cite_start]`pe.v` - The core Processing Element featuring Operand Isolation and safe synchronous-reset ICG[cite: 1, 7, 8].
* [cite_start]`systolic_4x4.v` - The top-level structural mesh connecting 16 PEs, the master clock gate, and the Global Performance Monitor[cite: 33, 38, 43].
* [cite_start]`pe_tb.v` - The dual-phase testbench that injects a staggered diagonal data wavefront[cite: 14, 20, 21].
* [cite_start]`systolic_power_activity.vcd` - The generated Value Change Dump (VCD) file containing the exact switching activity of the hardware during dense and sparse workloads[cite: 17, 18]. 

## 📊 Verification & Synthesis

[cite_start]The testbench is designed to simulate a real-world SoC memory bus, pushing dense matrices followed by high-sparsity matrices[cite: 19, 24]. [cite_start]During verification, the array successfully executed mathematical operations with 100% accuracy while dynamically logging hundreds of clock-gated zero-skips during the sparse phase[cite: 26, 31]. 

[cite_start]The resulting `.vcd` file is Cadence-ready[cite: 17]. By feeding this switching activity history into a synthesis tool, it proves a massive drop in dynamic power when the workload shifts from active to idle.
