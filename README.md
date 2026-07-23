**A - Context & Problem Statement**

To address operational bottlenecks on a high-speed packaging conveyor system. The client's existing third-party sequence logic was causing frequent product jams, inconsistent product singulation, and lacked mechanical diagnostics, which resulted in extended downtime when pneumatic components fatigued. The project establishes a standardized control framework for a pneumatic material handling escapement system integrated into a manufacturing conveyor line. The system regulates product flow by detecting incoming packages and utilizing a single-acting pneumatic stopper cylinder to momentarily halt or release items based on adjustable operational parameters.

**B - Objective**

Redesign the control algorithm from scratch using clean-room Structured Control Language (SCL). The target architecture required a modular, timer-controlled escapement sequence capable of direct adjustment via a centralized Siemens HMI, alongside robust fault-handling routines to catch mechanical failures before they caused system pile-ups.

<img width="350" height="450" alt="image" src="https://github.com/user-attachments/assets/f38c4940-4946-4a87-ac75-170b4971c9d9" /> <img width="357" height="450" alt="image" src="https://github.com/user-attachments/assets/a67d76ac-11cb-4616-b908-3a806d5125eb" />



**Project Type** 
Independent Engineering Contract / Technical Subcontract

**Role** 
Automation & Controls Contractor

**Objective** 
Refine, debug, and optimize material handling escapement sequences for an industrial packaging line.

**Deliverables** 
PLC Control Logic (SCL), HMI Monitoring Screens, and Commissioning Documentation.

**Control Platform** 
Programmable Logic Controller (PLC): Siemens SIMATIC S7-1500F (CPU 1511TF-1 PN)

**Development Environment** 
Siemens TIA Portal (v15 or higher)

**Programming Language**
SCL (Structured Control Language, IEC 61131-3 compliant)

**PE_Sensor_Infeed (Infeed Detection)**
Proximity photoelectric sensor stationed upstream to register the presence of incoming packages.

**Prox_Cylinder_Retracted (Home Position Verification)** Inductive proximity switch mounted on the rear-end of the pneumatic cylinder body to confirm full retraction.

**Valve_Stopper_Control (Pneumatic Actuator)** A 5/2-way single-solenoid directional control valve managing air distribution to a linear pneumatic cylinder acting as a physical singulation barrier.

**C - Functional Requirements & Control Logic**

**Primary Sequence (Interlocking Logic) / Default State (Resting)** The pneumatic stopper cylinder is configured to be normally extended (Blocking Position) to prevent unregulated product bypass during system idleness or power-up.

**Product Detection** When PE_Sensor_Infeed (1.1A1) transitions to TRUE (detecting a package), the control valve Valve_Stopper_Control (3.1Y1) energizes, venting/pressurizing the appropriate cylinder chambers to extend the cylinder forcefully and lock the package in queue.

**Clearing Transition** When PE_Sensor_Infeed transitions to FALSE (the trail edge of the package clears the optical beam), the control logic triggers the retraction cycle to release the package downstream.

**Retraction Timeout** When a release sequence initiates, the cylinder is commanded to retract. The system monitors the feedback loop from Prox_Cylinder_Retracted (1.1A2).

**Fault Triggers** If Prox_Cylinder_Retracted (1.1A2) fails to transition to TRUE within a predefined structural safety window (e.g., 2.5 seconds), a Cylinder Retraction Timeout Fault (4.1Y2) is latched.

**Fault Interlock** The active alarm flags a critical alert to the HMI screen and can be used to interlock upstream feed conveyors to prevent catastrophic line tracking jams.

**D - Schematics**

<p align="center">
<img width="564" height="203" alt="image" src="https://github.com/user-attachments/assets/a471f7ee-7bca-48de-a2ee-bd0ee69b6696" /> <img width="274" height="350" alt="image" src="https://github.com/user-attachments/assets/a8bd354b-acfc-4350-a61e-ad09f0622eea" />
</p>
<p align="center">
Sensors 1.1A1 / 1.1A2
</p>

<p align="center">
<img width="450" height="557" alt="image" src="https://github.com/user-attachments/assets/fe079851-e950-4386-90b5-ac6ad8ad9a67" /> <img width="550" height="350" alt="image" src="https://github.com/user-attachments/assets/8f64217e-df2f-44f9-a327-11cd9d7d1e8a" />
</p>
<p align="center">
Electrical Connection
</p>





**These documents are protected under author license (Velikov, Aleksandar)**
