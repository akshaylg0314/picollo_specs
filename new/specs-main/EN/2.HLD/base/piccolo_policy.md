# PICCOLO PolicyManager Detailed Design Document

**Document Number**: PICCOLO-POLICY-2025-001  
**Version**: 1.1  
**Date**: 2025-07-16  
**Author**: joshua-jung_LGESDV  
**Category**: Design Specification

## 1. Overview

This document provides a technical detailed design for the PolicyManager of the PICCOLO framework. The PolicyManager determines the service execution priority (allow, pending, deny) based on vehicle status, safety grade, resource usage, and autonomy level.

## 2. Design Goals

The PolicyManager is designed with the following goals:

1. Ensure that high-priority services are allowed preferentially
2. Dynamically adjust service priorities based on vehicle status
3. Prioritize services according to safety grade (ASIL)
4. Decide deny/pending actions in case of resource collision
5. Adjust service priorities according to autonomy level

## 3. Priority Determination Elements

### 3.1 Default Service Priority

Service types are defined with a base priority range of 1-100 (higher means higher priority).

| Service Type | Base Priority | Description |
|-------------|--------------|-------------|
| safety-critical | 90 | Essential safety services (brake control, steering control) |
| adas | 85 | Driver assistance systems |
| hud | 80 | Head-up display |
| navigation | 75 | Navigation |
| cluster | 70 | Instrument cluster |
| alert | 65 | Alerts and notifications |
| voice-control | 60 | Voice control |
| diagnostics | 55 | Vehicle diagnostics |
| media | 50 | Media (audio/video) |
| comfort | 45 | Comfort features (temperature, seat control) |
| infotainment | 40 | Infotainment |
| connectivity | 35 | Connectivity services |
| background | 30 | Background services |
| update | 25 | Software update |

### 3.2 Vehicle State-based Priority Adjustment

Service priorities are adjusted according to vehicle state.

#### 3.2.1 Poweroff State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

#### 3.2.2 Ignition State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

#### 3.2.3 Parking State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

#### 3.2.4 Driving State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

#### 3.2.5 Neutral State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

#### 3.2.6 Reverse State

| Service Type | Priority Adjustment |
|-------------|--------------------|
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

### 3.3 ASIL Safety Grade Priority Adjustments

| ASIL Grade | Priority Adjustment | Description |
|------------|-------------------|-------------|
| QM | 0 | Quality Managed |
| A | +5 | ASIL A |
| B | +10 | ASIL B |
| C | +15 | ASIL C |
| D | +20 | ASIL D (highest safety grade) |

### 3.4 Device Collision Policy

Policy for determining service deny/pending status in case of resource conflicts.

| Device | Collision Policy | Timeout(s) | Retry Count | Additional Info |
|--------|-----------------|------------|------------|----------------|
| audio | pending | 30 | 3 | |
| gpu | pending | 60 | 5 | |
| speaker | pending | 15 | 2 | |
| mic | deny | - | - | Privacy-sensitive resource |
| display | pending | 45 | 3 | |
| npu | pending | 120 | 10 | |

### 3.5 Autonomous Driving Level Priority Adjustments

#### 3.5.1 Level 1 (Driver Assistance)

| Service Type | Priority Adjustment |
|-------------|--------------------|
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
|-------------|--------------------|
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
|-------------|--------------------|
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
|-------------|--------------------|
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
|-------------|--------------------|
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

The final priority is calculated as the sum of the following components:
```
Final Priority = Base Priority + Vehicle State Adjustment + ASIL Grade Adjustment + Autonomous Driving Level Adjustment
```

### 4.2 Decision Thresholds

| Decision | Threshold |
|----------|-----------|
| Allow | ≥ 70 |
| Pending | ≥ 40, < 70 |
| Deny | < 40 |

### 4.3 Special Rules

In certain situations, special rules apply regardless of the calculated priority:

1. Safety-critical services are always allowed when driving
2. Software updates are always denied when driving
3. Driver attention-related alerts have decreased priority in autonomous driving levels 4-5

## 5. Service Type Policy Examples

### 5.1 Safety-Critical Services

| Service Name Pattern | Type | ASIL Grade | Required Devices |
|----------------------|------|-----------|-----------------|
| lane-assist-* | adas | B | gpu, npu |
| adaptive-cruise-* | adas | C | npu |
| collision-warning-* | adas | B | gpu, npu |

### 5.3 Infotainment Services

| Service Name Pattern | Type | ASIL Grade | Required Devices |
|----------------------|------|-----------|-----------------|
| navigation-* | navigation | QM | display, gpu, speaker |
| media-* | media | QM | audio, display, gpu |
| infotainment-* | infotainment | QM | display, gpu, audio |

### 5.4 Other Services

| Service Name Pattern | Type | ASIL Grade | Required Devices |
|----------------------|------|-----------|-----------------|
| voice-* | voice-control | QM | mic, speaker, npu |
| diagnostic-* | diagnostics | QM | npu |
| update-* | update | QM | - |

## 6. Comprehensive Service Priority Calculation Examples

### 6.1 Service Priority Table by Vehicle State

The following table shows final priority calculations for various vehicle states with autonomous driving level 3 and ASIL B safety grade:

| Service Type | Base | Poweroff | Ignition | Parking | Driving | Neutral | Reverse |
|--------------|------|----------|----------|---------|---------|---------|---------|
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

* Note: Values are calculated as Base Priority + Vehicle State Adjustment + ASIL B Grade Adjustment(+10) + Autonomous Driving Level 3 Adjustment, with maximum value limited to 100

### 6.2 Key Service Priority Comparison by Autonomous Driving Level

| Service Type | Base | Level 1 | Level 2 | Level 3 | Level 4 | Level 5 |
|--------------|------|---------|---------|---------|---------|---------|
| safety-critical | 90 | 115 | 120 | 125 | 130 | 135 |
| adas | 85 | 120 | 125 | 130 | 135 | 135 |
| hud | 80 | 115 | 120 | 115 | 110 | 105 |
| navigation | 75 | 105 | 110 | 105 | 115 | 120 |
| alert | 65 | 95 | 100 | 100 | 100 | 100 |
| media | 50 | 55 | 45 | 60 | 65 | 70 |
| infotainment | 40 | 35 | 25 | 40 | 45 | 55 |
| update | 25 | 0 | 0 | 0 | 0 | 0 |

* Note: Values are calculated as Base Priority + Driving State Adjustment + ASIL B Grade Adjustment(+10) + Autonomous Driving Level Adjustment, with maximum value limited to 100 and minimum to 0

### 6.3 Specific Service Case Analysis

#### 6.3.1 Lane Departure Warning Service While Driving (Changes by Level)

| Autonomous Level | Base | Vehicle State | ASIL | Autonomous | Final | Decision |
|-----------------|------|--------------|------|------------|------|----------|
| Level 1 | 85 | +15 | +10 | +10 | 120 | Allow |
| Level 2 | 85 | +15 | +10 | +15 | 125 | Allow |
| Level 3 | 85 | +15 | +10 | +20 | 130 | Allow |
| Level 4 | 85 | +15 | +10 | +25 | 135 | Allow |
| Level 5 | 85 | +15 | +10 | +25 | 135 | Allow |

#### 6.3.2 Media Player While Driving (Changes by Level)

| Autonomous Level | Base | Vehicle State | ASIL | Autonomous | Final | Decision |
|-----------------|------|--------------|------|------------|------|----------|
| Level 1 | 50 | -5 | 0 | 0 | 45 | Pending |
| Level 2 | 50 | -5 | 0 | -10 | 35 | Deny |
| Level 3 | 50 | -5 | 0 | +5 | 50 | Pending |
| Level 4 | 50 | -5 | 0 | +10 | 55 | Pending |
| Level 5 | 50 | -5 | 0 | +15 | 60 | Pending |

#### 6.3.3 Software Update While Parked (Changes by State)

| Vehicle State | Base | Vehicle State | ASIL | Autonomous Level 3 | Final | Decision |
|---------------|------|--------------|------|-------------------|------|----------|
| Poweroff | 25 | +20 | 0 | -30 | 15 | Deny |
| Ignition | 25 | -10 | 0 | -30 | 0 | Deny |
| Parking | 25 | +20 | 0 | -30 | 15 | Deny |
| Driving | 25 | -80 | 0 | -30 | 0 | Deny |
| Neutral | 25 | -10 | 0 | -30 | 0 | Deny |
| Reverse | 25 | -80 | 0 | -30 | 0 | Deny |

## 7. ISO 26262 Functional Safety Integration

The PolicyManager defines the following safety goals to comply with the ISO 26262 automotive functional safety standard:

| Safety Goal ID | Description | ASIL Grade | Applicable Services |
|----------------|-------------|------------|---------------------|
| SG-01 | Ensure reliable execution of safety-critical services during vehicle operation | D | safety-critical, adas |
| SG-02 | Prevent execution of services inappropriate for vehicle state | C | All services |
| SG-03 | Ensure priority execution of safety-critical services during resource conflicts | C | safety-critical, adas, alert |

### 7.2 Safety Mechanisms by ASIL Grade

The following safety mechanisms are applied according to each ASIL grade:

#### 7.2.1 ASIL D Service Requirements

| Category | Requirement | Implementation Method |
|----------|-------------|----------------------|
| Monitoring | 50ms interval monitoring | Regular status checks and heartbeat monitoring |
| Fault Handling | Maximum 3 retries, recovery within 500ms | Automatic recovery mechanism |
| Resource Requirements | CPU isolation, memory reservation, redundancy | Dedicated resource allocation |
| Minimum Priority | Guarantee of 80 or higher | Priority lower limit setting |

#### 7.2.2 ASIL C Service Requirements

| Category | Requirement | Implementation Method |
|----------|-------------|----------------------|
| Monitoring | 100ms interval monitoring | Regular status checks |
| Fault Handling | Maximum 2 retries, recovery within 1000ms | Automatic recovery mechanism |
| Resource Requirements | CPU isolation, memory reservation | Dedicated resource allocation |
| Minimum Priority | Guarantee of 70 or higher | Priority lower limit setting |

#### 7.2.3 ASIL B Service Requirements

| Category | Requirement | Implementation Method |
|----------|-------------|----------------------|
| Monitoring | 200ms interval monitoring | Regular status checks |
| Fault Handling | Maximum 2 retries, recovery within 2000ms | Automatic recovery mechanism |
| Resource Requirements | Memory reservation | Guaranteed memory allocation |
| Minimum Priority | Guarantee of 60 or higher | Priority lower limit setting |

#### 7.2.4 ASIL A Service Requirements

| Category | Requirement | Implementation Method |
|----------|-------------|----------------------|
| Monitoring | 500ms interval monitoring | Regular status checks |
| Fault Handling | Maximum 1 retry, recovery within 3000ms | Limited recovery attempts |
| Resource Requirements | General resource usage | Standard resource management |
| Minimum Priority | Guarantee of 50 or higher | Priority lower limit setting |

### 7.3 Safety Mechanism Implementation

#### 7.3.1 Service Monitoring
