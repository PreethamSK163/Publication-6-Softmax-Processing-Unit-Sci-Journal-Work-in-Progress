# Sci Journal Paper (Work In Progress) — Fixed-Point Softmax Processing Unit with LUT-Based Exponent and Mitchell Logarithmic Divider

> A fully fixed-point hardware architecture for the Softmax activation function — featuring dual-mode LUT-based exponentiation and an 8-stage pipelined Mitchell logarithmic division unit — designed for deployment in Deep Neural Network inference accelerators. Physical design completed through GDSII on TSMC 180nm using Cadence Genus and Cadence Innovus.

![Status](https://img.shields.io/badge/Status-SCI%20Journal%20Manuscript%20In%20Progress-yellow)
![Tool](https://img.shields.io/badge/Tool-Cadence%20Genus%2021.14-blue)
![Tool](https://img.shields.io/badge/Tool-Cadence%20Innovus%2021.15-blue)
![PDK](https://img.shields.io/badge/PDK-TSMC%20180nm-orange)
![Language](https://img.shields.io/badge/Language-Verilog%20HDL-purple)
![Application](https://img.shields.io/badge/Application-DNN%20Inference%20Accelerators-green)



## 📄 Publication Details

| Field | Details |
|:---|:---|
| **Title** | Low-Power High-Performance Fixed-Point Softmax Processing Unit with LUT-Based Exponent and Mitchell Logarithmic Divider |
| **Status** | SCI-Indexed Journal Manuscript — In Progress |
| **Related Patent** | Indian Patent Application — Under Review, Indian Patent Office |
| **Patent Attorney** | Khurana & Khurana, Advocates and IP Attorneys |
| **Field** | VLSI Design / AI Hardware Acceleration |
| **Research Centre** | Centre for Nanoelectronics and VLSI Design (CNVD), VIT Chennai |



## 👥 Authors

| Name | Affiliation |
|:---|:---|
| **Preetham SK** | School of Electrical Engineering, VIT Chennai |
| Sreehari R | School of Electrical Engineering, VIT Chennai |
| Dr. P. Sasipriya | Professor Grade 1, CNVD, School of Electronics Engineering, VIT Chennai |
| Dr. A. Anita Angeline | Professor, CNVD, School of Electronics Engineering, VIT Chennai |
| Prof. V. Anantha Krishnan | Assistant Professor, School of Electrical Engineering, VIT Chennai |



## 🔍 Overview

The Softmax activation function is a critical computational block in Deep Neural Networks — used in the output layer of CNNs, Transformers, and other multi-class classifiers to convert raw logits into a normalised probability distribution. Despite its mathematical simplicity, hardware implementation of Softmax is challenging due to the computational intensity of exponentiation and division operations, particularly demanding in fixed-point arithmetic where precision and dynamic range must be carefully managed.

This work presents a **fully fixed-point hardware architecture** for the Softmax function, specifically designed for resource-constrained DNN inference accelerators. The architecture eliminates the need for floating-point computation and input normalisation while maintaining high classification accuracy — enabling efficient deployment in ASICs and edge AI systems.

The complete architecture has been implemented in Verilog HDL, synthesised using **Cadence Genus** on **TSMC 180nm CMOS**, and taken through **complete physical design including GDSII layout generation** using **Cadence Innovus** — for both the Single-LUT and Dual-LUT exponent configurations.



## 🧠 Problem Addressed

Existing Softmax hardware implementations suffer from four key limitations:

**1. Hardware Complexity in Exponentiation:** Taylor series expansion and CORDIC-based methods require multiple iterative stages — resulting in high area, long delay, and increased power consumption impractical for real-time inference hardware.

**2. Precision-Area Tradeoff in LUT-Based Designs:** Coarse LUT quantisation introduces significant output error; fine-grained LUTs maintain accuracy but require impractically large memory structures for ASIC deployment.

**3. Sequential Division Bottleneck:** Conventional restoring/non-restoring array dividers require multiple clock cycles per division. For a Softmax layer with N inputs, total latency grows as O(N × cycles_per_division) — creating a severe throughput bottleneck for neural network inference pipelines.

**4. Input Normalisation Overhead:** Most existing designs require maximum-value subtraction before exponentiation — necessitating additional sorting hardware that increases system latency and area.



## 💡 Proposed Architecture

The Softmax Processing Unit comprises three coordinated hardware blocks controlled by a top-level Finite State Machine:

### 1. Exponent Block — Dual-Mode LUT-Based Design

Two architectural configurations are provided:

**Single-LUT Design** — Uses a 61-entry lookup table with linear interpolation over 1024 sub-steps per interval, operating on a 16-bit Q3.12 signed input. Achieves fine-grained precision without requiring a large table, producing a 32-bit Q14.18 fixed-point exponential output. Eliminates input normalisation entirely.

**Dual-LUT Design** — Uses two cascaded lookup tables: LUT1 with 16 entries at integer steps and LUT2 with 64 entries at 1/64 fractional steps. Integer and fractional components are retrieved independently and combined multiplicatively — achieving equivalent accuracy through a compact two-table factorisation approach.

Both configurations operate on Q3.12 format input (16-bit signed) and produce Q14.18 format output (32-bit unsigned).

### 2. Accumulator Block

A 36-bit Q22.14 format accumulator performs sequential addition of exponent outputs. The 36-bit width supports accumulation of up to N=239 worst-case exponent values before overflow — providing full support for DNN output layers with N up to 64 with significant headroom.

### 3. Division Block — 8-Stage Pipelined Mitchell Logarithmic Divider

The Mitchell binary logarithmic approximation algorithm is implemented as a fully pipelined 8-stage design:

| Stage | Operation |
|:---|:---|
| Stage 1 | Leading Zero Detection (LZD) on dividend and divisor |
| Stage 2 | Operand normalisation and mantissa extraction |
| Stage 3 | Integer logarithm computation from LZD results |
| Stage 4 | Fractional mantissa extraction |
| Stage 5 | Log-domain subtraction — log(dividend) − log(divisor) |
| Stage 6 | Antilogarithm integer reconstruction |
| Stage 7 | Fractional antilogarithm reconstruction and output assembly |
| Stage 8 | Output clamping and Q1.15 formatting |

The pipelined implementation achieves **one Softmax probability output per clock cycle** after pipeline fill — delivering approximately **8.9× throughput improvement** over a 16-cycle sequential restoring binary divider for N=10 inputs.

### Fixed-Point Datapath

| Stage | Format | Width |
|:---|:---|:---|
| Input | Q3.12 signed | 16-bit |
| Exponent output | Q14.18 unsigned | 32-bit |
| Accumulated sum | Q22.14 unsigned | 36-bit |
| Final probability output | Q1.15 unsigned | 16-bit |



## ⚙️ Implementation

| Parameter | Details |
|:---|:---|
| **HDL** | Verilog HDL |
| **Functional Verification** | Cadence NC Launch + Python reference model |
| **Logic Synthesis Tool** | Cadence Genus Synthesis Solution 21.14 |
| **Physical Design Tool** | Cadence Innovus Implementation System 21.15 |
| **Technology Node** | TSMC 180nm CMOS (tsmc18) |
| **Standard Cell Library** | TSMC 180nm slow corner |
| **Operating Conditions** | Slow (balanced_tree) |
| **Clock Frequency** | 100 MHz (10 ns period) |
| **Pipeline Depth** | 8 stages (Division Block) |
| **Supported N** | 2 to 239 input classes |



## 📊 Physical Design Results

### Single-LUT Configuration — Synthesis (Cadence Genus 21.14, TSMC 180nm)

| Metric | Value |
|:---|:---|
| **Total Cell Count** | 5,152 |
| **Exponent Block Cells** | 1,524 |
| **Accumulator Block Cells** | 385 |
| **Total Cell Area** | 132,703.4 µm² |
| **Exponent Block Area** | 30,157.1 µm² |
| **Accumulator Block Area** | 10,115.6 µm² |
| **Total Power** | 13.44 mW |
| **Register Power** | 6.40 mW (49.43%) |
| **Logic Power** | 5.98 mW (46.14%) |
| **Clock Power** | 0.57 mW (4.43%) |
| **Critical Path Slack** | 171 ps (MET) |
| **Clock Period** | 10 ns (100 MHz) |

### Dual-LUT Configuration — Synthesis (Cadence Genus 21.14, TSMC 180nm)

| Metric | Value |
|:---|:---|
| **Total Cell Count** | 5,026 |
| **Exponent Block Cells** | 1,418 |
| **Accumulator Block Cells** | 383 |
| **Total Cell Area** | 132,969.5 µm² |
| **Exponent Block Area** | 31,441.1 µm² |
| **Accumulator Block Area** | 10,065.7 µm² |
| **Total Power** | 12.14 mW |
| **Register Power** | 6.29 mW (51.79%) |
| **Logic Power** | 5.29 mW (43.55%) |
| **Clock Power** | 0.57 mW (4.66%) |
| **Critical Path Slack** | 68 ps (MET) |
| **Clock Period** | 10 ns (100 MHz) |

### Physical Design — Cadence Innovus 21.15, TSMC 180nm

Both Single-LUT and Dual-LUT configurations have been carried through complete physical design using Cadence Innovus 21.15 — including floorplanning, placement, clock tree synthesis (CTS), routing, parasitic extraction (SPEF), and GDSII layout generation.

| Stage | Single-LUT | Dual-LUT |
|:---|:---|:---|
| Floorplan | ✅ Complete | ✅ Complete |
| Placement | ✅ Complete | ✅ Complete |
| Clock Tree Synthesis | ✅ Complete | ✅ Complete |
| Routing | ✅ Complete | ✅ Complete |
| SPEF Extraction | ✅ Complete | ✅ Complete |
| GDSII Generation | ✅ Complete | ✅ Complete |

## 📈 Functional Verification Results

| Metric | Single-LUT | Dual-LUT |
|:---|:---|:---|
| **MNIST Classification Accuracy** | 97.89% | 97.89% |
| **Hardware-Software Output Match** | 100% (0 mismatches) | 100% (0 mismatches) |
| **Validation Dataset** | 10,000 MNIST samples | 10,000 MNIST samples |
| **Reference Model** | Python floating-point | Python floating-point |
| **Throughput vs Sequential Divider** | 8.9× improvement | 8.9× improvement |


## 🎯 Applications

- **DNN Inference Accelerators** — Output layer Softmax for CNN and Transformer models
- **Edge AI and IoT** — Resource-constrained ASIC inference on wearable and embedded devices
- **Real-Time Image Classification** — Hardware Softmax for mobile vision applications
- **Natural Language Processing Hardware** — Attention score normalisation in Transformer accelerators
- **Medical Imaging** — Low-power hardware inference for diagnostic classification systems


## 🔗 Related Work

| Output | Details |
|:---|:---|
| **Patent** | Indian Patent Application — Under Review, Indian Patent Office |
| **SCI Journal** | Manuscript in progress |
| **Research Supervisor** | Dr. P. Sasipriya, CNVD, VIT Chennai |
| **Co-Supervisor** | Dr. A. Anita Angeline, CNVD, VIT Chennai |
| **Research Centre** | Centre for Nanoelectronics and VLSI Design (CNVD), VIT Chennai |

## 🤝 Connect

| **Author** | Preetham SK |
|:---|:---|
| **Programme** | B.Tech. Electrical and Electronics Engineering, VIT Chennai |
| **LinkedIn** | [linkedin.com/in/preethamsk16](https://www.linkedin.com/in/preethamsk16) |
| **GitHub** | [github.com/PreethamSK163](https://github.com/PreethamSK163) |
| **Portfolio** | [preethamsk163.github.io](https://preethamsk163.github.io) |
| **Email** | preethamsk163@gmail.com |
