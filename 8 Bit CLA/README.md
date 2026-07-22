# 8-Bit Custom Carry-Lookahead Adder (CLA)

## Project Overview
This project involves the full-custom design of an 8-bit Carry-Lookahead Adder (CLA) built from scratch using a 45nm CMOS process In high-speed processors, the adder is usually one of the biggest speed bottlenecks because the carry bit must sequentially pass through every stage. To overcome this and improve upon a standard synthesized baseline, this design utilizes architectural parallelism, custom transistor sizing, and low-threshold voltage (low-Vth) devices. 

*   **Objective:** Maximize operational frequency under a 1.1V supply voltage, exceeding the 1.8 GHz project requirement.
*   **Result:** Achieved a maximum frequency of 2.8 GHz, delivering a 27.3% speed boost (600 MHz) over the automated place-and-route (PnR) baseline.

---

## Architecture & Design Choices

### Hierarchical Carry-Lookahead Architecture
To mitigate the inherent propagation delay of ripple-carry designs, the architecture cascaded two 4-bit CLA blocks together. 
*   The lookahead logic computes the propagate and generate signals across 4-bit boundaries ahead of time.
*   The propagate signal is calculated as $P_{i} = A_{i} \oplus B_{i}$.
*   The generate signal is calculated as $G_{i} = A_{i} \cdot B_{i}$.
*   This approach enables the first 4-bit block to rapidly calculate its carry-out and instantly pass it to the second block as the carry-in.

### Custom Gate Design & Sizing
Rather than relying on pre-made basic gates inside Cadence, every single gate on the critical path was hand-designed. 
*   **Low-Vth Devices:** The design exclusively uses low-Vth devices for all gates to lower the channel threshold.
*   This specific design choice yields significantly more drive current to switch the nodes faster.
*   While this increases power consumption, it was necessary to prioritize meeting the frequency criteria.

### 1-Bit Mirror Adder Cell
Standard combinations of XOR, AND, and OR gates were avoided because stacking multiple stages adds too much propagation delay.
*   The 1-bit full adder was implemented as a single-stage, static complementary CMOS mirror adder.
*   This structure features symmetric NMOS and PMOS networks for the carry-out (Cout).
*   The single-stage layout minimizes total transistor count and reduces internal node capacitance compared to a traditional gate-level implementation, significantly reducing propagation delay.

---

## Simulation & Testing Setup
The custom 8-bit adder was simulated and tested using Cadence. 
*   **Baseline Comparison:** The design was benchmarked against an automated PnR ripple-carry adder synthesized from the standard gsclib045 library at VDD=1.1V.
*   **Pipeline Simulation:** Both adders were driven by positive-edge triggered D-flip-flops and loaded with flip-flop inputs to simulate a real pipeline environment.
*   **Worst-Case Delay Vectors:** Performance was characterized using critical input pairs. The ripple-carry baseline was tested with A=11111111 and B=00000001, while the custom carry-lookahead design used A=01111111 and B=00000001.
*   **Power Measurement:** Power consumption at maximum frequency was calculated by averaging the transient current waveform over 10 full switching cycles, making sure to include the clock network and the D-flip-flops.

---

## Performance Results

The custom sizing and low-Vth strategy successfully produced a 600 MHz speed jump over the synthesized baseline, which justifies the slight bump in power consumption. 

| Metric | Synthesized Baseline (Ripple-Carry) | Custom Design (Carry-Lookahead) |
| :--- | :--- | :--- |
| **Max Frequency (fmax)** | 2.2 GHz | 2.8 GHz |
| **Power Consumption (at fmax)** | 322.142 µW | 428.846 µW |
| **Energy per Operation (at fmax)** | 146.43 fJ | 151.37 fJ |
| **Transistor Types** | Standard VT Cell (SVT) | Low VT Cell (VTL) |

*(Note: When tested at a 1GHz clock speed and VDD=1.1V, the custom design consumed 143.159 µW with an energy efficiency of 143.2 fJ, outperforming the baseline's 146.658 µW and 146.66 fJ)*.

---

## Future Optimizations
While hand-sizing transistors and optimizing the architecture was effective, low-Vth devices have higher subthreshold leakage, which increased static power. If a second iteration were to be done, the following optimizations could be applied:
*   **Dual-Vth Strategy:** Keep the low-Vth devices on the critical path to S(7) to maintain the 2.8 GHz speed, but use standard or high-Vth devices for the non-critical side paths to cut down on leakage.
*   **Dynamic Logic Implementation:** Try implementing the lookahead logic with dynamic multi-output domino logic instead of static CMOS to fully remove the PMOS sizing load, potentially allowing the design to run at frequencies over 3.0 GHz.