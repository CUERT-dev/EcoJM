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



## 3. Hardware Architecture and Module Documentation

The proposed system is composed of several functional modules responsible for power switching, electrical measurement, communication, processing, and data storage. The STM32F development board acts as the central processing unit, receiving analogue measurement signals from the current and voltage measurement circuits and controlling the main power switching circuit. A CAN transceiver provides communication between the microcontroller and the vehicle CAN bus, while the auxiliary power supply generates the required 5 V and 3.3 V supply rails.



## 3.1 Microcontroller Unit
## 3.1.1 Purpose
The Microcontroller Unit (MCU) serves as the central control and data-processing element of the system. It receives measurement signals from the current and voltage sensing circuits through its analog-to-digital converter (ADC) inputs, controls the main power switching circuit, communicates with the vehicle network through the CAN interface, and manages data storage through the SPI interface.
The system uses an STM32F development board based on an STM32 microcontroller. The MCU operates from the regulated 3.3 V supply generated by the auxiliary power supply.

## 3.1.2 Architecture
The MCU is connected to the current measurement circuit through an analog input, where the conditioned output of the ACS712 current sensor is sampled by the ADC. A second ADC channel is used to acquire the scaled voltage measurement signal generated by the voltage measurement circuit.
A digital control signal is provided to the main relay switching circuit. This signal controls the relay driver and consequently determines whether the high-voltage path is connected to the external load.
For communication, the MCU uses its CAN transmit and receive signals to interface with the SIT65HVD230DR CAN transceiver. The MCU also communicates with the MicroSD card through the SPI peripheral for data logging.
The MCU is supplied from the 3.3 V regulated rail. The 5 V rail is used as an intermediate supply generated by the DC-DC converter and is subsequently regulated to 3.3V.
| Interface | MCU connection       | Function                  |
| --------- | -------------------- | ------------------------- |
| ADC       | Current_M            | Current measurement       |
| ADC       | Voltage_M            | Voltage measurement       |
| GPIO      | SSR0 / relay control | Main relay control        |
| CAN TX/RX | CAN transceiver      | Vehicle communication     |
| SPI       | MOSI, MISO, SCK, CS  | MicroSD data logging      |
| VREF      | TL431 reference      | ADC reference             |
| 3.3 V     | Power input          | MCU supply                |




## 3.2 CAN Communication
## 3.2.1 Purpose
The CAN communication module provides an interface between the microcontroller and the vehicle CAN network. It allows the system to transmit measured electrical parameters and receive relevant vehicle information through the CAN bus.

## 3.2.2 Architecture

The CAN communication path is divided into two interfaces. The MCU-side interface consists of the CAN transmit (CAN_TX) and CAN receive (CAN_RX) signals. These signals are connected to the SIT65HVD230DR CAN transceiver.
The transceiver converts the logic-level CAN signals from the MCU into the differential CANH and CANL signals required by the physical CAN bus. The CANH and CANL lines are then connected to the external vehicle CAN network.
Protection components are placed at the external interface to reduce the risk of transient or fault conditions propagating into the communication circuitry.

## 3.2.3 Data flow

MCU → CAN_TX → transceiver → CANH/CANL → vehicle

and:

vehicle → CANH/CANL → transceiver → CAN_RX → MCU



## 3.3 Main Power Switch
## 3.3.1 Purpose
The main power switch controls the connection between the vehicle high-voltage power source and the external load. The switching function is implemented using a relay controlled by the microcontroller through an appropriate driver circuit, with secondary provisions for hardware-level manual override and transient voltage protection.

## 3.3.2 Architecture

EV HV Power → K1 Relay SPDT → Switched HV Output - External Load

STM32F MCU → Relay Control → MOSFET → K1 Relay

When the MCU determines that the main power path should be enabled, it generates the relay-control signal. The MOSFET driver circuit energizes the relay coil, causing K1 to change state. The HV path is then connected to the switched output. When the control signal is removed, the relay returns to its default open state. A flyback diode is connected across the relay coil to suppress the voltage transient generated when the relay coil is de-energized, which protects the switching transistor from excessive voltage stress.

Manual Bypass Interface: Connectors J1 and J10 are wired directly in parallel with the control MOSFET. This allows external hardware, such as a physical toggle switch or an emergency stop button, to ground the relay coil and engage the main power path completely independent of MCU intervention.   
Transient Voltage Protection: A Transient Voltage Suppression (TVS) diode (D1) is placed across the switched high-voltage output lines (VDCH+_OUT and VDCH-). If a sudden transient voltage spike or surge occurs on the power lines, this diode instantly shunts the excess energy, protecting the external load and preventing damage to the system. 



## 3.4 Current Measurement — Joule Meter
## 3.4.1 Purpose
The current measurement module measures the current flowing through the monitored electrical path. The measured current is converted into an analogue voltage by the ACS712 Hall-effect current sensor and sampled by the STM32 ADC. The measured current is subsequently used together with the measured voltage to calculate electrical power and accumulated energy.

## 3.4.2 ACS712 operation
The ACS712 provides electrical isolation between the high-current conductor and the low-voltage measurement circuitry using Hall-effect sensing. The output of the sensor is an analog voltage whose value varies according to the current flowing through the monitored conductor.

## 3.4.3 ADC interface
The 5V analog output of the ACS712 is stepped down through a resistor voltage divider (R6 and R7) to safely scale the signal below 3.3V, creating the Current_M signal that is sampled by the STM32 ADC.



##### UNDER DEVELOPMENT #####

## 4. Instrumentation and Measurement

## 4.1 Instrumentation Objectives
## 4.2 Required Equipment
## 4.3 Test Points and Measurement Nodes
## 4.4 Initial Power-Up Procedure
## 4.5 Power Rail Measurements
## 4.6 Current Measurement Testing
## 4.7 Voltage Measurement Testing
## 4.8 CAN Communication Testing
## 4.9 MicroSD Interface Testing
## 4.10 Main Switch Testing


## 5. Calibration

## 5.1 Calibration Objectives
## 5.2 Calibration Requirements
## 5.3 Current Sensor Calibration
## 5.4 Voltage Measurement Calibration
## 5.5 Calibration Procedure
## 5.6 Calibration Data
## 5.7 Calibration Error and Uncertainty
## 5.8 Calibration Results


## 6. PCB Implementation and Mechanical Integration

## 6.1 PCB Overview
## 6.2 Component Placement
## 6.3 Routing and Layout Considerations
## 6.4 Power and Signal Separation
## 6.5 Connectors and Accessibility
## 6.6 Mounting Holes and Mechanical Features
## 6.7 3D Model Generation

## 7. Validation and Bring-Up

## 7.1 Initial Bring-Up
## 7.2 Low-Voltage Testing
## 7.3 Power Supply Validation
## 7.4 MCU and Communication Validation
## 7.5 ADC and Measurement Validation
## 7.6 Main Switching Validation
## 7.7 MicroSD Validation
## 7.8 Integrated Hardware Testing


## 8. Known Issues and Limitations

## 8.1 Known Hardware Issues
## 8.2 Measurement Limitations
## 8.3 Calibration Limitations
## 8.4 Documentation Gaps
## 8.5 Mechanical and PCB Limitations


## 9. Future Development

## 9.1 Hardware Improvements
## 9.2 Measurement Improvements
## 9.3 Protection Improvements
## 9.4 Communication and Data Logging Improvements
## 9.5 Mechanical Improvements


## 10. Revision History
