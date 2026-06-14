# Hardware-Accelerated Real-Time Object Detection using CNN on SoC

## 📌 Overview
This repository contains the complete hardware/software co-design implementation of a Convolutional Neural Network (CNN) accelerator targeted at a System-on-Chip (SoC) FPGA architecture (such as the Xilinx Zynq-7000 series). 

The primary objective of this project is to achieve real-time object detection at the edge by offloading compute-intensive convolutional operations to custom Programmable Logic (PL) while the Processing System (ARM Cortex cores) handles high-level control, pre-processing, and AXI data transfers. 

## 🚀 Key Features
* **Custom 3x3 MAC Array**: A deeply pipelined Multiply-Accumulate block designed in SystemVerilog to execute 9 parallel multiplications for rapid spatial convolution.
* **On-the-Fly Window Generation**: Utilizes custom Line Buffers (`line_buffer.sv`) to stream standard pixel input into 3x3 matrices (`window_generator.sv`) without stalling the datapath or relying heavily on external memory.
* **Hardware Activation & Pooling**: Integrates zero-latency ReLU activation and high-throughput 2x2 Max Pooling directly into the RTL datapath.
* **Standardized IP Integration**: A dedicated AXI wrapper (`cnn_axi_wrapper.v`) memory-maps the custom accelerator, enabling seamless high-throughput DMA transfers between the PS and PL.
* **Cycle-Accurate Control**: An FSM-based controller (`cnn_controller.sv`) manages datapath routing, ensuring deterministic latency across all network layers.

## 🛠️ Technology Stack
* **Hardware Description Language**: SystemVerilog, Verilog
* **Design, Synthesis & Implementation**: Xilinx Vivado Design Suite
* **IP Packaging & Integration**: Vivado IP Integrator (`.bd` block designs)
* **Target Architecture**: ARM-based SoC FPGAs (e.g., Xilinx Zynq)
* **Bus Protocols**: AMBA AXI4-Lite (Control), AXI4-Stream (Data)

## 📂 Project Structure (RTL & Block Design)

### Core Computation Datapath
* `cnn_accelerator_top.sv` - The top-level module instantiating and wiring all CNN sub-components.
* `mac_array_3x3.sv` - The arithmetic heart of the accelerator performing dot products for the convolutional layers.
* `relu_activation.sv` - Combinational logic module applying the non-linear Rectified Linear Unit function.
* `max_pool_2x2.sv` - Reduces spatial dimensions of feature maps by outputting the maximum value in a 2x2 sliding window.

### Memory & Data Management
* `line_buffer.sv` - FIFO-based line buffers to cache image rows for 2D filtering operations.
* `window_generator.sv` - Synchronizes with the line buffers to continuously output valid 3x3 pixel grids to the MAC array.

### Control & Interconnect
* `config.sv` - Global parameter definitions (bit-widths, matrix dimensions, thresholds).
* `cnn_controller.sv` - The central Finite State Machine managing pipeline stalls, data validation signals, and layer transitions.
* `cnn_axi_wrapper.v` - Translates AXI4 transactions from the ARM processor into native signals for the CNN Top module.

### SoC Export Files
* `cnn_soc.bd` / `cnn_soc.bda` - The Vivado Block Design files defining the complete PS-PL system topology.
* `cnn_soc_wrapper.xsa` - The exported hardware handoff file containing the bitstream and hardware definitions for Vitis/Petalinux software development.

## ⚙️ Architecture Flow
1. The **Processing System (ARM)** loads the image data and pre-trained CNN weights into the DDR memory.
2. The PS initiates an AXI transaction to send a stream of image pixels to the **Programmable Logic (FPGA)**.
3. The data enters the `cnn_axi_wrapper` and flows into the `window_generator` and `line_buffer`.
4. As valid 3x3 windows form, they are pushed into the `mac_array_3x3` along with the corresponding weights.
5. The accumulated convolution output is passed through `relu_activation` and then downsampled via `max_pool_2x2`.
6. The final processed feature maps are sent back through the AXI interface to the PS for classification (bounding box generation).

## 👨‍💻 Authors & Team
* **Vemana Venkata Pujithram** (Project Lead / RTL Designer)
* Developed as part of the academic System-on-Chip Design coursework for real-time edge AI optimization.

## 🔮 Future Work
* Transitioning to dynamic 8-bit Integer Quantization for the weights to double the MAC array density.
* Implementing parallel MAC arrays to process multiple input channels simultaneously (depthwise convolution).
* Developing a custom Vitis Linux application to handle live camera feeds directly into the PL via VDMA.
