# 1. Introduction



<img width="635" height="367" alt="image" src="https://github.com/user-attachments/assets/05d3be40-322f-48dd-a830-78df48c4a0e6" />



## 1.1 Purpose
This document describes the architecture, design, implementation, and validation of the **EcoJM** vehicle energy measurement and safety control platform. It documents the hardware architecture used to realize the EcoJM implementation, including its modular functional partitioning, electrical interfaces, power distribution, automated switching key, measurement infrastructure, grounding strategy, PCB implementation, mechanical integration, and protection mechanisms.

The purpose of this document is to provide a technical reference for understanding how the EcoJM architectural principles are realized in the physical hardware, as well as to record the engineering decisions, constraints, and trade-offs that shaped the design. This document is intended to support continued development, maintenance, debugging, reproduction, and future evolution of the hardware.

---

## 1.2 EcoJM Scope
The EcoJM system implements an onboard Joulemeter intended to measure the total energy consumed by an Electric Vehicle (EV), while providing real-time hardware safety mechanisms. The system is composed of integrated hardware modules responsible for low-voltage signal conditioning, energy processing, data logging, power regulation, and automated high-power load isolation.

The EcoJM implementation provides the hardware infrastructure required for high-precision energy measurement and automated protection, including:
* **Automated Switching & Isolation Key:** High-power relay and driver stage for physical power-bus isolation.
* **Current & Voltage Measurement:** Differential signal paths and analog front-end circuitry for real-time power sensing.
* **Control Infrastructure:** Microcontroller processing stage for calculation of energy metrics and safety logic execution.
* **Power Supply Unit (PSU):** Low-voltage auxiliary power regulation and rail generation.
* **CAN Communication:** Vehicle bus interface for streaming live telemetry and diagnostic data.
* **Onboard Data Storage:** MicroSD interface for continuous high-rate energy logging and event recording.
* **Fault & Protection Mechanisms:** Inline fuses, inductive filtering, and transient suppression components.

The scope of this document is limited to the EcoJM hardware implementation. Firmware algorithms, software state machines, and higher-level vehicle control strategies are discussed only where they directly affect hardware operation, switching logic, or electrical interfaces.

---

## 1.3 Design Objectives
The EcoJM hardware was developed to satisfy the following primary engineering objectives:

* **Real-time Vehicle Energy Tracking:** Maintain accurate, low-noise monitoring and accumulation of total energy usage ($\text{Wh}/\text{kWh}$) drawn by the vehicle's primary motor driver and auxiliary systems under dynamic operating conditions.
* **Automated Fault Isolation:** Provide immediate hardware power cutoff via the primary relay (`K1`) when operating parameters exceed defined current, voltage, or short-circuit thresholds.
* **High EMI Immunity & Noise Suppression:** Achieve signal measurement accuracy within harsh electromagnetic environments by integrating low-pass filtering networks (`L2`, `C5`, `C8–C11`, `R9–R12`), differential trace routing, and transient suppression diodes (`D1–D2`).
* **Mechanical & Thermal Reliability:** Enhance physical board durability against vehicle chassis vibration and thermal fatigue through full teardrop reinforcement across all vias and component pads (`td_onvia`, `td_onsmdpad`, `td_onpthpad`).
* **Maintainability & Debug Access:** Provide accessible test points (`TP3`), standardized mounting features (`H1–H4`), and clean interface layouts to simplify probing, validation, and field servicing.

These objectives are treated as primary engineering requirements and trade-offs. Where requirements conflict, system safety, noise immunity, electrical performance, and circuit protection take strict precedence.



## 2. EcoJM System Architecture



## 2.1 System Overview
EcoJM is a Joule meter designed to measure the total energy consumed by an Electric Vehicle (EV). The system acquires electrical measurements, processes the measurement data through an STM32-based controller, and supports communication and data storage for energy monitoring and analysis.

## 2.2 Functional Block Diagram

The functional block diagram illustrates the main hardware modules of the EcoJM system, their interconnections, and the flow of power, measurement signals, and communication data.

<img width="1078" height="636" alt="image" src="https://github.com/user-attachments/assets/aaa3919e-68aa-4c2b-8df8-05408c47b7e9" />

## 2.3 Main Functional Domains
- **Processing and Control**: The STM32F_dev board serves as the system's microcontroller unit (MCU). It processes measurement signals, communicates with connected peripherals, and coordinates system-level operations.
- **Electrical Measurement**: The system acquires voltage and current measurements through its sensing circuits. The measurement signals are provided to the MCU for processing and energy monitoring.
- **CAN Communication**: The SIT65HVD230DR acts as a CAN transceiver, providing the physical interface between the MCU's CAN signals and the external CAN bus.
- **Switching** — K1 is an SPDT relay used for switching within the system's power circuit. Its exact switching function and control behavior should be verified from the schematic and system requirements.
- **Data Storage**: The HX TF PUSH is a microSD card socket connected to the MCU through the implemented data interface. It provides an interface for external storage and data logging.
- **Voltage Reference**: The TL431DBZ is used as a voltage reference in the measurement circuit. Its exact function and reference voltage should be verified from the schematic and component specifications.
- **Auxiliary Power Supply**: The power supply section generates the required low-voltage supply rails. The MT2492 is used in the power conversion stage, while the AZ1117-3.3 generates the 3.3 V supply rail.

## 2.4 Power and Signal Flow
The EcoJM system includes power distribution, electrical measurement, processing, and communication paths.
The auxiliary power supply generates the required voltage rails for the electronic circuits. The measurement circuitry acquires voltage and current signals, which are transferred to the MCU for processing. The MCU communicates with external devices through the CAN interface and may exchange data with the microSD storage interface.
The detailed power and signal paths, including the relevant connectors, supply rails, and measurement signals, are documented in the system schematic.

