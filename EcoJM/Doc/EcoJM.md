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

<img width="996" height="581" alt="image" src="https://github.com/user-attachments/assets/9c7d3ded-be28-4b88-87e7-04cd456a5250" />

## 2.3 Main Functional Domains
- **Processing and Control**: The STM32F_dev board serves as the system's microcontroller unit (MCU). It processes measurement signals, communicates with connected peripherals, and coordinates system-level operations.
- **Electrical Measurement**: The system acquires load measurements using a high-power voltage divider and an ACS712 30A Hall-effect current sensor. These signals are scaled down to safe analog voltages (Voltage_M and Current_M) and provided to the MCU to calculate instantaneous power and total Joule consumption.
- **CAN Communication**: The SIT65HVD230DR acts as a CAN transceiver, providing the physical interface between the MCU's CAN signals and the external CAN bus.
- **Switching**: K1 is an SPDT relay acting as the main high-power switch for the system. It is driven by the MCU via an AO3400A MOSFET and includes manual bypass headers for external hardware control or emergency switching.
- **Data Storage**: The HX TF PUSH is a microSD card socket connected to the MCU via a standard SPI interface. It provides external storage capabilities, likely for logging continuous energy consumption data.
- **Voltage Reference**: The TL431DBZ shunt regulator provides a highly stable 2.5V reference signal (VREF). This feeds directly into the MCU’s analog-to-digital converter (ADC) to ensure precise, calibrated readings from the voltage and current sensors.
- **Auxiliary Power Supply**: TThe power supply section steps down a 12V external input to generate the required low-voltage supply rails. A switching regulator (MT2492) efficiently steps the 12V down to 5V, while an LDO linear regulator (AZ1117-3.3) drops the 5V to a clean 3.3V rail for the digital logic.

## 2.4 Power and Signal Flow
The EcoJM system includes power distribution, electrical measurement, processing, and communication paths.
The auxiliary power supply generates the required voltage rails for the electronic circuits. The measurement circuitry acquires voltage and current signals, which are transferred to the MCU for processing. The MCU communicates with external devices through the CAN interface and may exchange data with the microSD storage interface.
The detailed power and signal paths, including the relevant connectors, supply rails, and measurement signals, are documented in the system schematic.

