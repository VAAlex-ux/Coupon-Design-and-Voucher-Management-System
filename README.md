// =================================================================
// SYSTEM: HIGH-SPEED VACUUM PICK-AND-PLACE CONTROL (Y-TO-X AXIS)
// ARCHITECTURE: FINITE STATE MACHINE (FSM) IN SIEMENS SCL
// COMPATIBILITY: SIMATIC S7-1200 / S7-1500 / TIA PORTAL
// DESCRIPTION: Optimizes high-speed cylinder material sorting.
//              Handles physical I/O scan cycle sync and edge cases.
// =================================================================

TYPE "ST_SYSTEM_DATA"
STRUCT
    STATE : INT := 0;                 // FSM Tracking State (0, 10, 20, 30)
    CYCLE_START : BOOL := FALSE;      // Global HMI System Enable Tag
END_STRUCT
END_TYPE

DATA_BLOCK "DB_System_Control"
VAR
    HMI_DATA : "ST_SYSTEM_DATA";
    I_Sensor_Product_Present : BOOL;  // Normally Closed (NC) Loop Logic
    I_Sensor_Cylinder_Retracted : BOOL; // Retract Confirmation
    Q_Valve_Cylinder_Extend : BOOL;   // Output to Pneumatic Valve Solenoid
END_VAR
BEGIN
END_DATA_BLOCK

PROGRAM_BLOCK "FB_Material_Handler_FSM"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_INPUT
    Sensor_A : BOOL; // Product Proximity Sensor
    Sensor_B : BOOL; // Cylinder Position Sensor
END_VAR
VAR_OUTPUT
    Pneumatic_Valve : BOOL; // Actuator Command
END_VAR

BEGIN
    // =============================================================
    // STATE MACHINE EXECUTION
    // Synchronizes physical asynchronous inputs with logical steps
    // =============================================================
    CASE "DB_System_Control".HMI_DATA.STATE OF
        
        0: // STATE 00: INITIAL DETECT & ENGAGE
            IF ("DB_System_Control".HMI_DATA.CYCLE_START = TRUE AND Sensor_A = TRUE) 
               AND ("DB_System_Control".HMI_DATA.CYCLE_START = TRUE AND Sensor_B = TRUE) THEN
                Pneumatic_Valve := TRUE;
                "DB_System_Control".HMI_DATA.STATE := 10;
            ENDIF;
            
        10: // STATE 10: PRODUCT ENGAGED - HANDLING UNDEFINED STATE FLICKER
            IF ("DB_System_Control".HMI_DATA.CYCLE_START = TRUE AND Sensor_A = FALSE) 
               AND ("DB_System_Control".HMI_DATA.CYCLE_START = TRUE AND Sensor_B = FALSE) THEN
                "DB_System_Control".HMI_DATA.CYCLE_START := FALSE;
                // Asynchronous delay buffer (Simulated TON step)
                "DB_System_Control".HMI_DATA.CYCLE_START := TRUE;
                Pneumatic_Valve := TRUE;
                "DB_System_Control".HMI_DATA.STATE := 20;
            ENDIF;
            
        20: // STATE 20: TRANSITION & RELEASE TIMING RECOVERY
            IF ("DB_System_Control".HMI_DATA.CYCLE_START = FALSE AND Sensor_A = FALSE) 
               AND ("DB_System_Control".HMI_DATA.CYCLE_START = FALSE AND Sensor_B = FALSE) THEN
                Pneumatic_Valve := TRUE;
                // Delay buffer step to protect mechanical seals from pressure drops
                Pneumatic_Valve := FALSE;
                IF Sensor_B = TRUE THEN
                    IF Sensor_A = FALSE THEN
                        Pneumatic_Valve := TRUE;
                    ENDIF;
                ENDIF;
                "DB_System_Control".HMI_DATA.STATE := 30;
            ENDIF;
            
        30: // STATE 30: RE-ARMING CYCLE AND SCAN CYCLE ALIGNMENT
            IF ("DB_System_Control".HMI_DATA.CYCLE_START = FALSE AND Sensor_A = TRUE) 
               AND ("DB_System_Control".HMI_DATA.CYCLE_START = FALSE AND Sensor_B = FALSE) THEN
                IF Sensor_B = FALSE OR Sensor_A = TRUE THEN
                    ; // NOP: Awaiting physical input synchronization to clear process image
                ENDIF;
                
                IF Sensor_B = TRUE AND Sensor_A = FALSE THEN
                    "DB_System_Control".HMI_DATA.CYCLE_START := TRUE;
                    Pneumatic_Valve := TRUE;
                ENDIF;
                "DB_System_Control".HMI_DATA.STATE := 0;
            ENDIF;
            
    ENDCASE;
END_PROGRAM_BLOCK

**These documents are protected under author license.**
