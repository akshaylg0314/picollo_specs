# PICCOLO PolicyManager Detailed Design Document

**Document Number**: PICCOLO-POLICY-2025-001  
**Version**: 1.1  
**Date**: 2025-07-16  
**Author**: joshua-jung_LGESDV  
**Classification**: Design Specification

## 1. Overview

This document describes the detailed design of the PolicyManager for the PICCOLO framework. The PolicyManager determines the priority of services based on vehicle state, safety level, resource usage, and autonomous driving level, and makes decisions on whether to allow, pend, or deny service execution.

## 2. Design Objectives

The PolicyManager is designed with the following objectives:

1. Ensure high-priority services are allowed first
2. Dynamically adjust service priorities according to vehicle state
3. Differentiate service priorities according to safety level (ASIL)
4. Make appropriate deny/pending decisions in case of resource conflicts
5. Adjust service priorities according to autonomous driving level

## 3. Priority Decision Factors

### 3.1 Basic Service Priorities

Basic priorities for service types are defined in the range of 1-100 (higher means higher priority).

| Service Type | Base Priority | Description |
|------------|--------------|------|
| safety-critical | 90 | Safety critical services (brake control, steering control) |
| adas | 85 | Advanced Driver Assistance Systems |
| hud | 80 | Head-Up Display |
| navigation | 75 | Navigation |
| cluster | 70 | Instrument Cluster |
| alert | 65 | Warnings and Alerts |
| voice-control | 60 | Voice Control |
| diagnostics | 55 | Vehicle Diagnostics |
| media | 50 | Media (audio/video) |
| comfort | 45 | Comfort Features (temperature control, seat adjustment) |
| infotainment | 40 | Infotainment |
| connectivity | 35 | Connectivity Services |
| background | 30 | Background Services |
| update | 25 | Software Updates |

### 3.2 Priority Adjustments by Vehicle State

Priority adjustment values by service type according to vehicle state.

#### 3.2.1 Power Off State (poweroff)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | -50 |
| adas | -90 |
| hud | -90 |
| navigation | -90 |
| cluster | -90 |
| alert | -50 |
| voice-control | -90 |
| diagnostics | +10 |
| media | -90 |
| comfort | -90 |
| infotainment | -90 |
| connectivity | -50 |
| background | +10 |
| update | +20 |

#### 3.2.2 Ignition State (ignition)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +10 |
| adas | +5 |
| hud | +5 |
| navigation | 0 |
| cluster | +10 |
| alert | +10 |
| voice-control | -10 |
| diagnostics | +15 |
| media | -20 |
| comfort | 0 |
| infotainment | -20 |
| connectivity | +5 |
| background | +5 |
| update | -10 |

#### 3.2.3 Parking State (parking)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | 0 |
| adas | -10 |
| hud | -20 |
| navigation | +5 |
| cluster | -5 |
| alert | +5 |
| voice-control | +10 |
| diagnostics | +15 |
| media | +15 |
| comfort | +10 |
| infotainment | +15 |
| connectivity | +10 |
| background | +5 |
| update | +20 |

#### 3.2.4 Driving State (driving)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +15 |
| navigation | +15 |
| cluster | +10 |
| alert | +10 |
| voice-control | +5 |
| diagnostics | -20 |
| media | -5 |
| comfort | 0 |
| infotainment | -10 |
| connectivity | -5 |
| background | -20 |
| update | -80 |

#### 3.2.5 Neutral State (neutral)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +5 |
| adas | +10 |
| hud | +10 |
| navigation | +10 |
| cluster | +5 |
| alert | +5 |
| voice-control | +5 |
| diagnostics | 0 |
| media | +5 |
| comfort | +5 |
| infotainment | +5 |
| connectivity | +5 |
| background | 0 |
| update | -10 |

#### 3.2.6 Reverse State (reverse)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +5 |
| navigation | -5 |
| cluster | +5 |
| alert | +10 |
| voice-control | -5 |
| diagnostics | -10 |
| media | -15 |
| comfort | -5 |
| infotainment | -15 |
| connectivity | -10 |
| background | -15 |
| update | -80 |

### 3.3 Priority Adjustments by ASIL Safety Level

| ASIL Level | Priority Adjustment | Description |
|----------|--------------|------|
| QM | 0 | Quality Managed |
| A | +5 | ASIL A |
| B | +10 | ASIL B |
| C | +15 | ASIL C |
| D | +20 | ASIL D (Highest safety level) |

### 3.4 Device Conflict Policy

Policy for determining service deny/pending status in case of resource conflicts.

| Device | Conflict Policy | Timeout (sec) | Retry Count | Additional Info |
|---------|------------|------------|-----------|---------|
| audio | pending | 30 | 3 | |
| gpu | pending | 60 | 5 | |
| speaker | pending | 15 | 2 | |
| mic | deny | - | - | Privacy-sensitive resource |
| display | pending | 45 | 3 | |
| npu | pending | 120 | 10 | |

### 3.5 Priority Adjustments by Autonomous Driving Level

#### 3.5.1 Level 1 (Driver Assistance)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +5 |
| adas | +10 |
| hud | +10 |
| alert | +10 |
| navigation | +5 |
| cluster | +5 |
| media | 0 |
| infotainment | -5 |
| update | -15 |

#### 3.5.2 Level 2 (Partial Automation)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +15 |
| alert | +15 |
| navigation | +10 |
| cluster | +10 |
| voice-control | +5 |
| media | -10 |
| infotainment | -15 |
| update | -25 |

#### 3.5.3 Level 3 (Conditional Automation)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +15 |
| adas | +20 |
| hud | +10 |
| alert | +15 |
| navigation | +5 |
| cluster | +10 |
| voice-control | +10 |
| media | +5 |
| infotainment | 0 |
| update | -30 |

#### 3.5.4 Level 4 (High Automation)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +20 |
| adas | +25 |
| hud | +5 |
| alert | +15 |
| navigation | +15 |
| voice-control | +15 |
| media | +10 |
| infotainment | +5 |
| comfort | +10 |
| update | -10 |

#### 3.5.5 Level 5 (Full Automation)

| Service Type | Priority Adjustment |
|------------|--------------|
| safety-critical | +25 |
| adas | +25 |
| navigation | +20 |
| alert | +15 |
| infotainment | +15 |
| media | +15 |
| comfort | +15 |
| voice-control | +15 |
| connectivity | +10 |
| update | 0 |

## 4. Decision Logic

### 4.1 Final Priority Calculation

The final priority is calculated as the sum of the following factors:
```
Final Priority = Base Priority + Vehicle State Adjustment + ASIL Level Adjustment + Autonomous Driving Level Adjustment
```

### 4.2 Decision Thresholds

| Decision | Threshold |
|------|--------|
| Allow | ≥ 70 |
| Pending | ≥ 40, < 70 |
| Deny | < 40 |

### 4.3 Special Rules

In some situations, special rules apply regardless of the calculated priority:

1. Safety-critical services are always allowed while driving
2. Software updates are always denied while driving
3. In autonomous driving levels 4-5, the priority of driver attention-related alerts is reduced

## 5. Service Type Policy Examples

### 5.1 Safety-Critical Services

| Service Name Pattern | Type | ASIL Level | Required Devices |
|-----------------|------|----------|--------------|
| brake-control-* | safety-critical | D | npu |
| steering-control-* | safety-critical | D | npu |

### 5.2 ADAS Services

| Service Name Pattern | Type | ASIL Level | Required Devices |
|-----------------|------|----------|--------------|
| lane-assist-* | adas | B | gpu, npu |
| adaptive-cruise-* | adas | C | npu |
| collision-warning-* | adas | B | gpu, npu |

### 5.3 Infotainment Services

| Service Name Pattern | Type | ASIL Level | Required Devices |
|-----------------|------|----------|--------------|
| navigation-* | navigation | QM | display, gpu, speaker |
| media-* | media | QM | audio, display, gpu |
| infotainment-* | infotainment | QM | display, gpu, audio |

### 5.4 Other Services

| Service Name Pattern | Type | ASIL Level | Required Devices |
|-----------------|------|----------|--------------|
| voice-* | voice-control | QM | mic, speaker, npu |
| diagnostic-* | diagnostics | QM | npu |
| update-* | update | QM | - |

## 6. Comprehensive Service Priority Calculation Examples

### 6.1 Service Priority Table by Vehicle State

The following is an example of final priority calculations for various vehicle states at autonomous driving level 3 with ASIL B safety level:

| Service Type | Base | Poweroff | Ignition | Parking | Driving | Neutral | Reverse |
|------------|------|----------|----------|---------|---------|---------|---------|
| safety-critical | 90 | 65 | 125 | 115 | 125 | 120 | 125 |
| adas | 85 | 25 | 120 | 105 | 130 | 125 | 130 |
| hud | 80 | 10 | 105 | 80 | 115 | 110 | 105 |
| navigation | 75 | 0 | 90 | 95 | 105 | 100 | 85 |
| cluster | 70 | 0 | 100 | 85 | 100 | 95 | 95 |
| alert | 65 | 35 | 100 | 95 | 100 | 95 | 100 |
| voice-control | 60 | 0 | 70 | 90 | 85 | 85 | 75 |
| diagnostics | 55 | 75 | 80 | 80 | 45 | 65 | 55 |
| media | 50 | 0 | 45 | 80 | 60 | 70 | 50 |
| comfort | 45 | 0 | 65 | 75 | 65 | 70 | 60 |
| infotainment | 40 | 0 | 40 | 75 | 50 | 65 | 45 |
| connectivity | 35 | 0 | 60 | 65 | 50 | 60 | 45 |
| background | 30 | 50 | 45 | 45 | 20 | 40 | 25 |
| update | 25 | 55 | 25 | 55 | 0 | 25 | 0 |

* Note: Values in the table are calculated as Base Priority + Vehicle State Adjustment + ASIL B Level Adjustment (+10) + Autonomous Driving Level 3 Adjustment, with maximum value limited to 100

### 6.2 Key Service Priority Comparison by Autonomous Driving Level

The following is a comparison of key service priorities by autonomous driving level for driving state with ASIL B safety level:

| Service Type | Base | Level 1 | Level 2 | Level 3 | Level 4 | Level 5 |
|------------|------|-------|-------|-------|-------|-------|
| safety-critical | 90 | 115 | 120 | 125 | 130 | 135 |
| adas | 85 | 120 | 125 | 130 | 135 | 135 |
| hud | 80 | 115 | 120 | 115 | 110 | 105 |
| navigation | 75 | 105 | 110 | 105 | 115 | 120 |
| alert | 65 | 95 | 100 | 100 | 100 | 100 |
| media | 50 | 55 | 45 | 60 | 65 | 70 |
| infotainment | 40 | 35 | 25 | 40 | 45 | 55 |
| update | 25 | 0 | 0 | 0 | 0 | 0 |

* Note: Values in the table are calculated as Base Priority + Driving State Adjustment + ASIL B Level Adjustment (+10) + Autonomous Driving Level Adjustment, with maximum value limited to 100 and minimum value limited to 0

### 6.3 Specific Service Case Analysis

#### 6.3.1 Lane Departure Warning Service While Driving (By Level)

| Autonomous Driving Level | Base | Vehicle State | ASIL | Autonomous Driving | Final | Decision |
|------------|------|----------|------|---------|------|------|
| Level 1 | 85 | +15 | +10 | +10 | 120 | Allow |
| Level 2 | 85 | +15 | +10 | +15 | 125 | Allow |
| Level 3 | 85 | +15 | +10 | +20 | 130 | Allow |
| Level 4 | 85 | +15 | +10 | +25 | 135 | Allow |
| Level 5 | 85 | +15 | +10 | +25 | 135 | Allow |

#### 6.3.2 Media Player While Driving (By Level)

| Autonomous Driving Level | Base | Vehicle State | ASIL | Autonomous Driving | Final | Decision |
|------------|------|----------|------|---------|------|------|
| Level 1 | 50 | -5 | 0 | 0 | 45 | Pending |
| Level 2 | 50 | -5 | 0 | -10 | 35 | Deny |
| Level 3 | 50 | -5 | 0 | +5 | 50 | Pending |
| Level 4 | 50 | -5 | 0 | +10 | 55 | Pending |
| Level 5 | 50 | -5 | 0 | +15 | 60 | Pending |

#### 6.3.3 Software Update While Parked (By State)

| Vehicle State | Base | Vehicle State | ASIL | Autonomous Driving Level 3 | Final | Decision |
|---------|------|----------|------|---------------|------|------|
| Poweroff | 25 | +20 | 0 | -30 | 15 | Deny |
| Ignition | 25 | -10 | 0 | -30 | 0 | Deny |
| Parking | 25 | +20 | 0 | -30 | 15 | Deny |
| Driving | 25 | -80 | 0 | -30 | 0 | Deny |
| Neutral | 25 | -10 | 0 | -30 | 0 | Deny |
| Reverse | 25 | -80 | 0 | -30 | 0 | Deny |

## 7. ISO 26262 Functional Safety Integration

### 7.1 Safety Goals and ASIL Level Definitions

PolicyManager defines the following safety goals to comply with the ISO 26262 automotive functional safety standard:

| Safety Goal ID | Description | ASIL Level | Applicable Services |
|------------|------|----------|-----------------|
| SG-01 | Ensuring reliable execution of safety-critical services while driving | D | safety-critical, adas |
| SG-02 | Preventing execution of services inappropriate for vehicle state | C | All services |
| SG-03 | Ensuring priority execution of safety-critical services in resource conflicts | C | safety-critical, adas, alert |

### 7.2 Safety Mechanisms by ASIL Level

The following safety mechanisms are applied according to each ASIL level:

#### 7.2.1 ASIL D Service Requirements

| Category | Requirement | Implementation Method |
|------|----------|-----------|
| Monitoring | 50ms interval monitoring | Regular status check and heartbeat verification |
| Fault Handling | Maximum 3 retries, recovery within 500ms | Automatic recovery mechanism |
| Resource Requirements | CPU isolation, memory reservation, redundancy | Dedicated resource allocation |
| Minimum Priority | Guarantee at least 80 | Setting priority lower limit |

#### 7.2.2 ASIL C Service Requirements

| Category | Requirement | Implementation Method |
|------|----------|-----------|
| Monitoring | 100ms interval monitoring | Regular status check |
| Fault Handling | Maximum 2 retries, recovery within 1000ms | Automatic recovery mechanism |
| Resource Requirements | CPU isolation, memory reservation | Dedicated resource allocation |
| Minimum Priority | Guarantee at least 70 | Setting priority lower limit |

#### 7.2.3 ASIL B Service Requirements

| Category | Requirement | Implementation Method |
|------|----------|-----------|
| Monitoring | 200ms interval monitoring | Regular status check |
| Fault Handling | Maximum 2 retries, recovery within 2000ms | Automatic recovery mechanism |
| Resource Requirements | Memory reservation | Memory allocation guarantee |
| Minimum Priority | Guarantee at least 60 | Setting priority lower limit |

#### 7.2.4 ASIL A Service Requirements

| Category | Requirement | Implementation Method |
|------|----------|-----------|
| Monitoring | 500ms interval monitoring | Regular status check |
| Fault Handling | Maximum 1 retry, recovery within 3000ms | Limited recovery attempt |
| Resource Requirements | General resource usage | Standard resource management |
| Minimum Priority | Guarantee at least 50 | Setting priority lower limit |

### 7.3 Safety Mechanism Implementation

#### 7.3.1 Service Monitoring

- Execution time monitoring: Monitoring whether service execution time exceeds expected range
- Resource usage monitoring: Tracking CPU, memory, network usage
- Health check: Regular status checks and error detection

#### 7.3.2 Fault Detection and Response

- Gradual recovery mechanism: Automatic recovery for minor issues, transition to safe state for serious issues
- Response strategy by severity:
  - Critical: Immediate service termination and safe state activation
  - Major: Service suspension and recovery attempt
  - Minor: Logging and automatic recovery attempt

#### 7.3.3 Safe State Transition

- Gradual performance degradation definition:
  - Minimum functionality: Maintain only core safety functions
  - Essential functionality: Maintain functions needed for safety and basic operation
  - Normal functionality: All functions activated
- State transition conditions and time constraints:
  - Safety-critical services: Complete within 1000ms
  - General services: Complete within 5000ms

### 7.4 Verification and Validation Requirements

#### 7.4.1 Test Requirements

- Code coverage: ASIL D services require 100% branch coverage and MC/DC coverage
- Requirement traceability: All safety requirements must be linked to test cases
- Fault injection tests: Verification of fault handling mechanism effectiveness

#### 7.4.2 Safety Analysis

- FMEA (Failure Mode and Effects Analysis): Identifying possible failure modes and evaluating impacts
- FTA (Fault Tree Analysis): Analyzing causes of safety goal violations
- HAZOP (Hazard and Operability Study): Identifying possible hazards during system operation

#### 7.4.3 Independent Verification

- Safety-critical components require verification by a team independent from the development team
- Definition of safety assessment report creation and approval process

### 7.5 Runtime Monitoring and Diagnostics

PolicyManager provides the following runtime monitoring and diagnostic functions:

- Service execution time monitoring: Response when execution time exceeds threshold
- Resource usage monitoring: Memory, CPU usage tracking and overload prevention
- Error logging: Detailed logging of safety-related errors and retention for 30 days
- Health monitoring: Health check at 1-second intervals

### 7.6 Safety Documentation Requirements

The following documents are managed for ISO 26262 compliance:

- Safety plan: Defining safety activities, schedule, and responsible persons
- Safety requirements specification: Detailed definition of functional safety requirements
- Safety analysis report: Results of safety analyses such as FMEA, FTA
- Test specification and report: Safety verification test procedures and results
- Safety case: Presenting evidence for achieving safety goals
- Safety assessment report: Results of independent safety assessment

## 8. Conclusion

Through the priority tables and decision logic defined in this document, the PICCOLO PolicyManager can make optimal service management decisions based on the vehicle's current state, service safety criticality, resource usage, and autonomous driving level. By integrating ISO 26262 functional safety requirements, as the autonomous driving level increases, the priority of safety-critical services continues to be maintained, while the priority of infotainment and convenience functions gradually increases as vehicle control becomes automated. This ensures vehicle safety while optimizing user experience in autonomous driving situations.
