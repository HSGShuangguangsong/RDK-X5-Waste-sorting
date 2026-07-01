# RDK X5-Based Waste Sorting Robot with 100+ FPS YOLO Vision Tracking Optimization

> High-Performance Host-Side Vision Decision-Making + High-Real-Time MCU-Side Motion Execution — A Fully Autonomous Intelligent Waste Sorting Robot for Home and Community Scenarios

[![Platform](https://img.shields.io/badge/Platform-RDK%20X5-blue)]()
[![MCU](https://img.shields.io/badge/MCU-STM32H750-green)]()
[![Detection](https://img.shields.io/badge/Detection-YOLOv11-orange)]()
[![Tracking](https://img.shields.io/badge/Tracking-ByteTrack-yellow)]()
[![FPS](https://img.shields.io/badge/Inference-%E2%89%A590%20FPS-red)]()
[![Accuracy](https://img.shields.io/badge/Accuracy-%E2%89%A595%25-brightgreen)]()

---

## Table of Contents

- [Project Overview](#project-overview)
- [Core Features](#core-features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Modules](#software-modules)
- [Performance Metrics](#performance-metrics)
- [Key Innovations](#key-innovations)
- [Mechanical Design](#mechanical-design)
- [Communication Protocol](#communication-protocol)
- [Application Scenarios](#application-scenarios)
- [Extensibility](#extensibility)
- [Suggested Project Structure](#suggested-project-structure)
- [References](#references)
- [Acknowledgments](#acknowledgments)

---

## Project Overview

With the continuous growth of municipal solid waste generation and the deepening implementation of China's "dual carbon" strategy, intelligent and automated sorting of household waste has become a key component in building a resource-conserving, environmentally friendly society. However, most mainstream waste sorting devices on the market today suffer from prominent problems such as **slow recognition speed, weak tracking robustness, inability to handle bagged waste, and poor electromechanical coordination precision**, making it difficult to meet the practical needs of real home and community scenarios.

This project builds an intelligent waste sorting robot based on a heterogeneous dual-core architecture combining **RDK X5** and **STM32H750**. The overall design adopts a layered architecture of "high-performance host-side vision decision-making + high-real-time MCU-side motion execution," achieving a complete closed-loop workflow from "**image acquisition → intelligent recognition → multi-target tracking → path planning → precise grasping → bag-breaking and sorting → directional disposal**." The entire process can autonomously complete intelligent household waste sorting without human intervention.

**Test results show**: the system achieves a visual recognition frame rate of over **90 FPS**, with a classification accuracy above **95%**, and can reliably sort waste into four major categories — recyclables, hazardous waste, kitchen waste, and other waste.

---

## Core Features

| Feature | Description |
|------|------|
| 🎯 **Multi-Category Waste Recognition & Tracking** | Real-time recognition of four waste categories based on YOLOv11, combined with ByteTrack to assign stable IDs to each target, resolving ID-switching issues in occlusion and stacking scenarios |
| 🦾 **High-Precision Mechanical Grasping & Directional Disposal** | CoreXY kinematics driving dual stepper motors + high-torque digital servos for precise XY-plane grasping and category-based disposal |
| 🔥 **Automatic Bag-Breaking for Bagged Waste** | An innovative heating-wire thermal-fusion bag-breaking solution builds a "recognize → break bag → re-recognize → sort" closed loop, overcoming the limitation of traditional devices that can only handle unbagged waste |
| 🖥️ **Real-Time Human-Machine Interaction & Status Visualization** | A serial-port HMI screen displays operating status, sorting counts, bin capacity, and fault alerts in real time |
| ⚡ **100+ FPS Embedded Inference** | CPU multi-threaded parallel pre/post-processing combined with a BPU asynchronous inference pipeline, along with int16 quantization of DFL nodes, achieves 90+ FPS |

---

## System Architecture

The system consists of five subsystems: **Visual Perception Layer, Decision & Inference Layer, Communication Layer, Motion Execution Layer, and Human-Machine Interaction Layer**. These modules collaborate efficiently through a strictly defined serial communication protocol, forming a complete closed loop of "perception—decision—communication—execution—feedback."

```
┌─────────────────────────────┐         UART          ┌──────────────────────────────┐
│  Host (RDK X5 / Ubuntu)      │  ◄──────────────────►  │  MCU (STM32H750 / Keil)       │
│                              │  Custom Protocol        │                              │
│  ┌────────┐  ┌────────────┐  │                        │  ┌────────────┐ ┌──────────┐ │
│  │ Image   │ →│ Preprocess │  │                        │  │ UART Recv/  │ │ Motion    │ │
│  │ Capture │  │            │  │                        │  │ Parse       │ │ Control   │ │
│  └────────┘  └────────────┘  │                        │  └────────────┘ └──────────┘ │
│        ↓                     │                        │        ↓             ↓       │
│  ┌────────────┐ ┌──────────┐ │                        │  ┌────────────┐ ┌──────────┐ │
│  │ BPU         │ →│Postproc/│ │  Perception/Decision → │  │ CoreXY      │ │ Servo/    │ │
│  │ Inference   │  │Tracking │ │  Data                  │  │ Kinematics  │ │ Bag-Break │ │
│  │ (YOLOv11)   │  │(ByteTrack)│ │  ← Status/Sensor Data  │  │ Solver      │ │ Timing    │ │
│  └────────────┘ └──────────┘ │                        │  └────────────┘ └──────────┘ │
│        ↓                     │                        │        ↓                     │
│  ┌────────────────────────┐  │                        │  ┌──────────────────────────┐ │
│  │  Serial Comm Thread     │  │                        │  │  HMI Display (Serial LCD) │ │
│  └────────────────────────┘  │                        │  └──────────────────────────┘ │
└─────────────────────────────┘                        └──────────────────────────────┘
      Foreground Multi-Threaded Architecture                    Foreground/Background Scheduling
    (Latest-frame-only buffering)                          (Main loop + Interrupt service)
```

- **Visual Perception Layer**: A USB 1080P HD camera captures images of waste on the loading platform
- **Decision & Inference Layer**: The RDK X5 single-board computer handles image preprocessing, YOLOv11 inference, ByteTrack tracking, coordinate transformation, and command generation
- **Communication Layer**: Based on the UART serial protocol, reliably delivers commands from the host to the MCU
- **Motion Execution Layer**: The STM32H750 drives the CoreXY stepper motor system, the mechanical gripper servo, the sorting-tray servo, and the heating-wire bag-breaking mechanism
- **Human-Machine Interaction Layer**: A serial-port HMI screen displays real-time operating status, integrated with an infrared sensing switch for contactless triggering

---

## Hardware Components

| Module Category | Core Components / Model | Main Function |
|---------|----------------|---------|
| Host | RDK X5 (ARM Cortex-A55 × 8 @1.5 GHz, 10 TOPS BPU, 8 GB LPDDR4) | Image acquisition, YOLOv11 inference, ByteTrack tracking, coordinate computation, and command generation |
| MCU | STM32H750VBT6 (ARM Cortex-M7 @480 MHz) | Kinematics solving, motor/servo driving, sensor acquisition, and HMI scheduling |
| Vision | USB 1080P HD camera | Real-time waste image capture, wide field of view, high resolution |
| Motion | 42 stepper motors + timing pulley/belt set | High-precision coordinated XY-plane motion (CoreXY) |
| Actuation | Multiple high-torque digital servos | Gripper open/close, sorting-tray rotation, waste disposal |
| Positioning Reference | Mechanical limit switches | XY-axis origin detection and homing |
| Sensing | Infrared distance sensor + infrared sensing switch | Bin-full detection, contactless triggering |
| HMI | Serial-port HMI display module | Real-time display of operating status, sorting counts, and fault alerts |
| Bag-Breaking | Relay + heating-wire module | Thermal-fusion bag-breaking for bagged waste |
| Power Management | 24V lithium battery + voltage regulation module | Provides stable power to all components |

### Circuit Modules

The circuit system is divided into seven modules: **main control circuit / power circuit / motor drive circuit / servo drive circuit / sensor acquisition circuit / bag-breaking relay circuit / HMI circuit**, all of which undergo rigorous electromagnetic compatibility design and anti-interference processing.

---

## Software Modules

### Host (RDK X5 / Ubuntu 22.04)

A multi-threaded parallel architecture where each functional thread operates independently, using a ring-buffer (latest-frame-only) mechanism to achieve efficient CPU-BPU pipelined parallel processing:

1. **Image Acquisition & Preprocessing Module** — Captures frames from the USB camera; performs format conversion, letterbox/resize, and NV12 packaging
2. **YOLOv11 Inference Module** — Loads the BPU-quantized model, submits asynchronous tasks, and performs dequantization → DFL decoding → NMS
3. **ByteTrack Multi-Target Tracking Module** — Kalman filter prediction plus primary/secondary Hungarian-algorithm association, mitigating ID switching caused by occlusion
4. **Coordinate Transformation & Command Dispatch Module** — Converts pixel coordinates to mechanical coordinates, packages custom protocol frames, and sends them via UART

### MCU (STM32H750 / Keil + HAL)

A foreground/background scheduling model — the main loop (polling execution) handles motion control and sensor acquisition, while interrupt service routines (USART receive interrupt, timers, limit switches) prioritize handling critical events:

5. **Serial Receive & Parsing Module** — USART interrupt + DMA double-buffered reception, with frame-header/length-field/CRC validation
6. **CoreXY Kinematics Solving Module** — `ΔA = ΔX + ΔY`, `ΔB = ΔX − ΔY`, combined with dynamic ARR adjustment to implement trapezoidal speed planning
7. **Servo & Actuator Control Module** — Advanced timers output PWM to control the gripper / sorting tray / Z-axis lift
8. **Bag-Breaking Timing Control Module** — Grasp → relay power on/off → timed fusion, with built-in overheat protection and timeout exit
9. **HMI Display Module** — Independent USART for bidirectional communication with the serial HMI screen, supporting mode switching and fault reset

---

## Performance Metrics

| Test Item | Design Target | Measured Result | Status |
|---------|---------|---------|---------|
| Four-category recognition accuracy | ≥ 95% | 96.3% | ✅ Met |
| Multi-target ID switch rate | ≤ 5% | 3.2% | ✅ Exceeded target |
| XY-plane positioning error | ≤ ±1 mm | ±0.6 mm | ✅ Exceeded target |
| Mechanical grasping success rate | ≥ 95% | 97.5% | ✅ Met |
| Single-item sorting cycle | ≤ 5 s/item | ~4.2 s/item | ✅ Met |
| Bag-breaking success rate | ≥ 98% | 99.0% | ✅ Exceeded target |
| Continuous operation stability | ≥ 2 h fault-free | 4 h continuous, fault-free | ✅ Exceeded target |

**Other key metrics**:
- Visual recognition frame rate: ≥ 90 FPS (CPU-BPU asynchronous pipeline + int16 quantization)
- Overall power consumption: standby ≤ 15 W, operating ≤ 60 W

---

## Key Innovations

1. **CPU-BPU Asynchronous Pipeline Scheduling Mechanism**: CPU multi-threaded pre/post-processing works in tandem with BPU asynchronous inference, enabling YOLOv11 to exceed 90 FPS on embedded hardware
2. **int16 Quantization Deployment of DFL Nodes**: Custom quantization targeting the Softmax computation bottleneck significantly frees up BPU compute with virtually no accuracy loss
3. **YOLO + ByteTrack Stable Multi-Target Tracking**: Kalman filtering combined with secondary association resolves ID jumps caused by occlusion and missed detections among multiple targets
4. **Innovative Heating-Wire Thermal-Fusion Bag-Breaking Mechanism**: Pioneers a "recognize → break bag → re-recognize → sort" closed loop, overcoming the limitation of traditional devices that can only handle unbagged waste
5. **Modular Snap-Fit Disassembly Structure**: Quick-release snap fittings combined with magnetic transparent access doors allow tool-free disassembly and cleaning

---

## Mechanical Design

The mechanical structure is centered on "modularity, lightweight design, and ease of maintenance." 3D modeling and kinematic simulation were completed in SolidWorks, and the robot was fabricated using a combination of **CNC-machined aluminum alloy profiles, 3D-printed parts, and acrylic sheets**. The overall structure is divided into six main sections:

- **Overall Frame**: CNC-machined 4040 industrial aluminum alloy profiles, offering high rigidity and flatness
- **CoreXY 2D Motion Platform**: Dual 42 stepper motors + GT2 timing belts, providing low motion inertia and high positioning precision
- **Mechanical Gripper & Lift Mechanism**: Driven by high-torque digital servos, with a bio-inspired flexible gripper head adaptable to various waste shapes
- **Sorting Bins**: Four quick-release snap-fit bins with magnetic transparent doors for easy capacity monitoring
- **Bag-Breaking Mechanism**: Heating-wire module + mounting bracket + protective cover, safely fusing through bagged waste
- **Enclosure & Protection Structure**: A combination of acrylic panels and 3D-printed parts providing both protection and visual display

---

## Communication Protocol

### Host → MCU (Target Coordinate Frame)

| Field | Description |
|------|------|
| Frame Header | Frame start marker |
| Target ID | Stable tracking ID assigned by ByteTrack |
| Coordinate X / Coordinate Y | Pulse coordinates in the mechanical coordinate system |
| Waste Category | One of the four waste classification labels |
| CRC Checksum | Frame integrity check |

When no target is detected, an empty-target/stop status frame is sent to prevent the MCU from continuing to act on the previous target.

### MCU → Host (Status Feedback Frame)

Contains status information such as position, speed, sensor data, and alerts, received via USART interrupt + DMA double buffering, with parsing triggered by an idle-line interrupt.

---

## Application Scenarios

**Current Scenarios**: Intelligent household waste sorting in homes, communities, supermarkets, office buildings, airports, train stations, and other public and semi-public spaces.

**Transferable Scenarios** (enabled by the "high-performance embedded vision + high-real-time motion control" heterogeneous dual-core architecture):

- Industrial assembly-line parts sorting
- Electronic component inspection
- Medical supplies categorization
- Agricultural product grading
- Logistics cargo handling
- Other RDK X5 embedded AI applications such as intelligent security, autonomous navigation, and humanoid robot perception

---

## Extensibility

**Functional Level**:
- Upgrade to a larger-scale version of YOLOv11 or Transformer-based frameworks such as RT-DETR to improve small-object recognition
- Introduce semantic segmentation models to achieve pixel-level fine-grained classification of waste after bag-breaking
- Add millimeter-wave radar or depth cameras to build an RGB-D perception system, addressing recognition challenges for transparent or reflective materials

**Scenario Level**:
- Integrate 5G / Wi-Fi 6 communication modules to enable multi-robot coordination and remote maintenance
- Expand into smart city and intelligent logistics scenarios

---

## Suggested Project Structure

> The following is a recommended code repository directory structure based on the document content; it can be adjusted according to actual engineering needs.

```
.
├── upper_computer/                # Host side (RDK X5)
│   ├── capture/                   # Image acquisition & preprocessing module
│   ├── inference/                 # YOLOv11 BPU inference module (quantized .bin model)
│   ├── tracking/                  # ByteTrack multi-target tracking module
│   ├── coordinate_transform/      # Coordinate transformation & command dispatch module
│   ├── serial_comm/               # Serial communication thread
│   └── main.py
├── lower_computer/                # MCU side (STM32H750, Keil + HAL)
│   ├── Core/
│   │   ├── Src/
│   │   │   ├── usart_parse.c      # Serial receive & parsing
│   │   │   ├── corexy_kinematics.c# CoreXY kinematics solving
│   │   │   ├── servo_control.c    # Servo & actuator control
│   │   │   ├── bag_breaking.c     # Bag-breaking timing control
│   │   │   └── hmi_display.c      # HMI display
│   │   └── Inc/
│   └── MDK-ARM/
├── mechanical/                    # SolidWorks 3D models & drawings
├── pcb/                            # Circuit design files
├── docs/                          # Technical documentation, test reports
└── README.md
```

---

## References

1. G. Yang et al., "Garbage Classification System with YOLOV5 Based on Image Recognition," ICSIP, 2021.
2. B. D. Carolis, F. Ladogana, N. Macchiarulo, "YOLO TrashNet: Garbage Detection in Video Streams," EAIS, 2020.
3. F. Akar et al., "A Real-Time Carboy Tracking and Counting System Based on YOLOv8 and ByteTrack," ICAMAC, 2024.
4. J. Tian et al., "Design of Intelligent Garbage Classification and Disposal System Based on Raspberry Pi," ICCECE, 2025.
5. Y. Zhang et al., "ByteTrack: Multi-Object Tracking by Associating Every Detection Box," ECCV, 2022.
6. L. V. Ma et al., "Adaptive Confidence Threshold for ByteTrack in Multi-Object Tracking," ICCAIS, 2023.
7. Z. X. Liu, W. X. Xie, P. Wang, "Tracking a Target Using a Cubature Kalman Filter Versus Unbiased Converted Measurements," ICSP, 2012.
8. L. Wang et al., "Research on Pedestrian Tracking Algorithm in Industrial Scene Based on Improved ByteTrack," EIECS, 2023.
9. T. K. Hariadi et al., "Design of Automatic Sorting Trash Bin for Organic, Inorganic, and Metal Waste," ICE3IS, 2025.
10. I. N. T. Catacutan et al., "BINSIGHT: Waste Sorting Enhanced with Machine Learning and Reward System," ISAS, 2024.

---

## Acknowledgments

During the optimization of the BPU heterogeneous deployment and pipeline scheduling mechanism for the vision algorithms, this project referenced technical resources shared by the D-Robotics (地瓜派) developer community. Sincere thanks are extended for this support.

---

> This project deeply integrates heterogeneous computing, machine vision, and electromechanical control technologies, providing an efficient and complete engineering solution for the practical deployment of intelligent household waste sorting. It also demonstrates strong transferability and application value in scenarios such as industrial parts sorting, medical supplies categorization, and agricultural product grading.
