# RISC-V Reconfigurable Hardware Accelerator for Edge Anomaly Detection

[Fill: one-line tagline, e.g. "An FPGA-based accelerator for real-time, on-device video anomaly detection using a MobileNet + GRU pipeline, targeting a Xilinx ZCU104."]

---

## 📌 Overview

This project implements an **INT8-quantized, tiled convolution hardware accelerator** for computer-vision-based anomaly detection, developed under India's **MeitY Chips-to-Startup (C2S) programme**, targeting deployment on a **Xilinx ZCU104 FPGA**.

The pipeline pairs a **MobileNet** backbone (spatial feature extraction) with a **GRU** temporal head (detecting anomalies across frames, not just within a single one), with the convolutional layers accelerated in custom hardware.

This work is documented in a peer-reviewed paper: *[paper title]*, published at ICSPCI 2025 (IEEE Xplore, DOI: 10.1109/ICCSPCI68199.2025.11451603).

---

## 🎯 Objectives

- Accelerate INT8 convolution (standard + depthwise) for real-time, on-device inference
- Support a MobileNet + GRU pipeline for temporal (video-based) anomaly detection
- Minimize dependence on external memory bandwidth via wide, parallel BRAM access
- Build toward a low-cost, domestically-producible accelerator architecture

---

## 🧩 Architecture

### RTL Architecture (Hardware)

The accelerator is a network of hand-written Verilog modules coordinated by a master FSM (**CCU**):

- **CCU** — master control FSM; latches configuration, drives downstream start/reset
- **MCU** — walks the convolution window, generates pixel/kernel addresses
- **MemoryBundle** — unpacks/packs 1024-bit (128-lane) BRAM words, address translation
- **Fmap_Router** — selects active channel lane(s) for the MAC array
- **UniversalACC** — accumulates partial products, adds bias, routes standard vs. depthwise paths
- **ChannelAcc** — cross-channel running sum for standard convolution
- **BiasReader** — reads bias values from the shared KernelMem region
- **DWM** — requantization stage (scale, round, shift, saturate, optional ReLU)
- **SumMux** — selects between depthwise and standard-conv accumulation paths

[Insert block diagram — Project_diagrams/]

### Memory Architecture

- **1024-bit (128-lane) BRAM words** — every compute module operates on all 128 lanes in parallel across a tile
- Kernel weights and bias share a time-multiplexed KernelMem region
- Supports both **standard convolution** (cross-channel accumulation) and **depthwise convolution** (independent per-channel) via a single mode bit

### Host-Side Firmware (PYNQ / Python)

- `pack_layer()` — packs MobileNet layer weights + bias into the 128-lane BRAM format (once per layer, at setup)
- `pack_fmap()` — packs input activation tiles into the same format (every layer, every inference)
- Output arrays are DMA-ready (`V128` dtype), written via `write_kernel_to_cdma()` / `write_fmap_to_cdma()`

[Insert diagrams from the pack_layer/pack_fmap deep-dive doc]

---

## 🔁 Supported Operations

- Standard convolution
- Depthwise convolution
- INT8 quantized inference
- [Add: ReLU / pooling / GRU-specific ops if applicable]

---

## 📊 Results & Performance

[Fill: latency comparison table — CPU vs. FPGA, resource utilization (LUTs/BRAM/DSP), clock frequency, timing closure — same format as your earlier project]

| Platform | Method | Time |
|---|---|---|
| CPU | Software inference | |
| FPGA | [DMA method] | |

---

## ⚠️ Known Limitations / Work in Progress

- [From the architecture doc: note any disabled features relevant to share publicly — e.g. per-channel scale factor support is implemented but not yet enabled]
- Not yet deployed as a full industry-ready application — current stage is a working prototype (see [funding/next-steps context if you want to include it])

---

## 🚀 Future Work

- Full GRU temporal head integration in hardware (currently [software-only / in progress — fill in])
- Scale to larger models / additional layer types
- [Domestic/low-cost sourcing angle, if you want it public]

---

## 📄 Publication

[Paper title], ICSPCI 2025 (IEEE Xplore) — DOI: [10.1109/ICCSPCI68199.2025.11451603](https://doi.org/10.1109/ICCSPCI68199.2025.11451603)

---

## 📬 Contact

Open for collaboration and discussion.
