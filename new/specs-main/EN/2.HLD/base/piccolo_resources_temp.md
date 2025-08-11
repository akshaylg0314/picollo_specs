# PICCOLO Resource Specification

**Document Number**: PICCOLO-RESOURCES-2025-001  
**Version**: 1.0  
**Date**: 2025-08-04  
**Author**: joshua-jung_LGESDV  
**Category**: Resource Specification  

## 1. Overview

This document provides the technical definitions and status descriptions for the main resource types (Scenario, Package, Model, Volume, Network, Node) used in the PICCOLO framework. Each resource type is used as a configuration element for service deployment, execution, and management in the vehicle environment.

## 2. Scenario Resource

### 2.1 Scenario Definition

A Scenario defines actions for specific packages according to vehicle state conditions. It determines the rules for how services are managed when certain vehicle states or events occur.

#### 2.1.1 Scenario Configuration Elements

| Element | Description |
|---------|------------|
| condition | Vehicle state condition for scenario application (e.g., vehicle speed, ignition state, battery level) |
| action | Operation to perform when the condition is met (launch, update, modify, notify, rollback, pause, resume) |
| target | The package that is the target of the operation |

#### 2.1.2 Scenario Action Types

| Action Type | Description | Main Use Case |
|-------------|-------------|---------------|
| launch | Start and deploy package | Activate service in specific state |
| update | Update package | Upgrade package version |
| modify | Modify package configuration | Change service settings during operation |
| notify | Notify package state | Send notification on state change |
| rollback | Restore to previous version | Rollback on update failure or issue |
| pause | Temporarily suspend package | For resource check or priority handling |
| resume | Resume suspended package | Restore service after temporary suspension |

#### 2.1.3 Scenario Status

| Status | Description |
|--------|------------|
| idle | Scenario ready (not yet activated) |
| waiting | Waiting for condition to be met |
| playing | Scenario action in progress |
| error | Error occurred during scenario execution |
| allowed | Action allowed by policy |
| denied | Action denied by policy |

#### 2.1.4 Scenario YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: adas-activation
  annotations:
    description: "Activates ADAS features during driving"
    priority: "high"
  labels:
    function: "driver-assistance"
    category: "safety"
    asilLevel: "B"
spec:
  condition:
    vehicleSpeed: ">30"
    ignitionState: "ON"
    driverSeatOccupied: "true"
  action: launch
  target:
    packages:
      - adas-software
      - vision-processing
      - sensor-processing
status:
  currentState: playing
  policyStatus: allowed
  lastTransitionTime: "2025-08-03T14:25:32Z"
  message: "ADAS services activated successfully"
  conditions:
    - type: ConditionsMet
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:30Z"
    - type: PolicyVerified
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:31Z"
    - type: ActionExecuted
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:32Z"
  activePackages:
    - name: adas-software
      status: running
    - name: vision-processing
      status: running
    - name: sensor-processing
      status: running
```

## 3. Package Resource

### 3.1 Package Definition

A Package is a group of Models that are deployed and managed together. It is a logical grouping unit for multiple components that work together to provide specific functionality.

#### 3.1.1 Package Configuration Elements

| Element | Description |
|---------|------------|
| pattern | Deployment pattern (plain, distributed, selector) |
| models | List of models (containers) included in the package |
| types | Package type list (multiple can be specified: plain, redundant, update, time-critical, stateful, resource-bounded, secure) |

#### 3.1.2 Package Types

| Package Type | Description | Characteristics |
|--------------|-------------|-----------------|
| plain | Basic container operation | Basic type for simple deployment and execution |
| redundant | High availability guarantee | Redundant operation, fault detection and automatic recovery |
| update | Reliable update guarantee | Updates according to defined policies, automatic rollback on failure |
| time-critical | Precise time interval guarantee | Precise timing control, priority resource allocation, real-time scheduling |
| stateful | State information maintenance | State preservation during restart/relocation, automatic persistent storage management |
| resource-bounded | Strict resource constraint compliance | Clear resource allocation and limits, resource monitoring and control |
| secure | Enhanced security requirements | Isolated network environment, encrypted storage, security policy enforcement |

#### 3.1.3 Package Status

| Status | Description |
|--------|------------|
| initializing | Package in initialization state |
| running | Package operating normally |
| degraded | Package operating with some functionality degraded |
| error | Package error state |

#### 3.1.4 Package YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: vision-processing
  annotations:
    version: "2.3"
    description: "Camera and image processing system"
  labels:
    types: ["resource-bounded", "time-critical"]
    component: "vision"
    asilLevel: "B"
spec:
  pattern:
    type: selector
    nodeSelector:
      gpu: "available"
  models:
    - name: vision-system
      image: vehicle/vision-system:2.3
      node: VisionHPC
      resources:
        requests:
          cpu: 2
          memory: 2Gi
          gpu: 1
        limits:
          cpu: 4
          memory: 4Gi
          gpu: 1
        volume: vision-data-volume
        network: sensor-network
    - name: object-detection
      image: vehicle/object-detection:1.5
      node: VisionHPC
      resources:
        requests:
          cpu: 3
          memory: 4Gi
          gpu: 1
        limits:
          cpu: 6
          memory: 8Gi
          gpu: 1
        volume: vision-data-volume
        network: sensor-network
status:
  currentState: running
  lastTransitionTime: "2025-08-03T14:25:35Z"
  message: "All models running normally"
  models:
    - name: vision-system
      status: running
      restarts: 0
      readiness: true
    - name: object-detection
      status: running
      restarts: 0
      readiness: true
  resources:
    cpuUsage: "65%"
    memoryUsage: "48%"
    gpuUsage: "72%"
```

## 4. Model Resource

### 4.1 Model Definition

A Model is the same concept as a Pod, a deployment unit consisting of one or more containers. It is the most basic execution unit that provides actual services.

#### 4.1.1 Model Configuration Elements

| Element | Description |
|---------|------------|
| containers | Container definitions included in the model |
| resources | Required computing resources (CPU, memory, GPU, etc.) |
| volumes | Volumes to mount |
| networks | Networks to connect |
| scheduling | Scheduling-related settings (priority, real-time requirements, etc.) |

#### 4.1.2 Model Status (Same as Pod Status)

| Status | Description |
|--------|------------|
| Pending | Model waiting to be assigned to a node |
| Running | Model running normally |
| Succeeded | All containers in the model successfully terminated |
| Failed | One or more containers failed |
| Unknown | Model state cannot be determined |

#### 4.1.3 Model YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Model
metadata:
  name: brake-controller-model
  annotations:
    io.piccolo.annotations.package-type: safety-critical-v1.0
  labels:
    app: brake-controller
    safetyLevel: critical
    asilLevel: "D"
spec:
  hostNetwork: false
  containers:
    - name: brake-controller
      image: sdv.lge.com/vehicle/brake-controller:1.2
      resources:
        limits:
          cpu: 1
          memory: 1Gi
  scheduling:
    timeCritical: true
    priority: 95
    period: "10ms"
    deadline: "8ms"
    executionTime: "2ms"
    cpuAffinity: [2, 3]
    isolationLevel: "exclusive"
    criticality: "safety-critical"
  deviceAccess:
    - manager: vehicle-sensors
      devices: ["brake-sensor", "wheel-speed-sensor"]
  volumes:
    - name: brake-data
      mountPath: /data
      volume: brake-data-volume
  networks:
    - name: control-network
status:
  phase: Running
  conditions:
    - type: Ready
      status: "True"
      lastProbeTime: "2025-08-03T14:25:40Z"
  containerStatuses:
    - name: brake-controller
      state:
        running:
          startedAt: "2025-08-03T14:25:38Z"
      ready: true
      restartCount: 0
      image: sdv.lge.com/vehicle/brake-controller:1.2
      imageID: sha256:a1b2c3d4...
  startTime: "2025-08-03T14:25:35Z"
  hostIP: "192.168.1.10"
  podIP: "10.0.0.5"
```

## 5. Volume Resource

### 5.1 Volume Definition

A Volume is a storage resource that is mounted to containers to provide persistent data storage.

#### 5.1.1 Volume Configuration Elements

| Element | Description |
|---------|------------|
| size | Volume size |
| accessModes | Access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany) |
| storageClass | Storage class |
| reclaimPolicy | Reclaim policy |

#### 5.1.2 Volume YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Volume
metadata:
  name: vision-data-volume
  annotations:
    backup: "enabled"
  labels:
    application: "adas"
spec:
  size: 10Gi
  accessModes:
    - ReadWriteMany
  storageClass: "fast-ssd"
  persistentVolumeReclaimPolicy: Retain
  backup:
    schedule: "0 0 * * *"
    retention: 7
status:
  phase: Bound
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  claimRef:
    kind: Model
    name: vision-system
    namespace: default
  message: ""
```

## 6. Network Resource

### 6.1 Network Definition

A Network is a networking resource for communication between containers. It defines QoS management, security settings, and optimizations.

#### 6.1.1 Network Configuration Elements

| Element | Description |
|---------|------------|
| bridge | Network bridge settings |
| ingress/egress | Ingress/egress port and protocol settings |
| qos | Quality of service settings |
| security | Security settings |

#### 6.1.2 Network YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Network
metadata:
  name: sensor-network
  labels:
    app: "adas"
spec:
  node_name: "VisionHPC"
  bridge:
    name: "br0"
    type: "bridge"
  ingress:
    port: "5555"
    protocol: "udp"
  egress:
    port: "5556"
    protocol: "udp"
  qos:
    latency: "ultra-low"
    priority: "high"
  fastPath:
    enabled: true
    optimizationLevel: "aggressive"
  security:
    securityPolicy: "strict"
    encryption: true
status:
  state: active
  connectedModels: 5
  bandwidthUsage: "45Mbps"
  packetLoss: "0.01%"
  averageLatency: "1.2ms"
```

## 7. Node Resource

### 7.1 Node Definition

A Node is a physical or virtual computing resource where containers run. In vehicles, it typically represents various ECUs or computing hardware.

#### 7.1.1 Node Configuration Elements

| Element | Description |
|---------|------------|
| capacity | Resource capacity provided by the node (CPU, memory, GPU, etc.) |
| conditions | Node status conditions |
| addresses | Network addresses of the node |
| nodeInfo | System information of the node |

#### 7.1.2 Node YAML Example

```yaml
apiVersion: piccolo.io/v1
kind: Node
metadata:
  name: VisionHPC
  labels:
    role: "vision-processing"
    gpu: "available"
    zone: "front"
spec:
  unschedulable: false
  taints:
    - key: "dedicated"
      value: "vision"
      effect: "NoSchedule"
  podCIDR: "10.0.0.0/24"
  powerManagement:
    mode: "performance"
status:
  capacity:
    cpu: "8"
    memory: "16Gi"
    gpu: "2"
    pods: "32"
  allocatable:
    cpu: "7"
    memory: "14Gi"
    gpu: "2"
    pods: "28"
  conditions:
    - type: Ready
      status: "True"
      lastHeartbeatTime: "2025-08-04T08:30:45Z"
      lastTransitionTime: "2025-08-01T06:20:10Z"
      reason: "KubeletReady"
      message: "kubelet is posting ready status"
  addresses:
    - type: InternalIP
      address: "192.168.1.10"
    - type: Hostname
      address: "VisionHPC"
  nodeInfo:
    machineID: "abc123def456"
    systemUUID: "ABF23CD4-5678-90EF-GHIJ-KLMNOPQRSTUV"
    bootID: "zyx987wvu654"
    kernelVersion: "5.15.0-rt10"
    osImage: "Automotive Linux 8.5"
    containerRuntimeVersion: "podman://4.5.1"
    kubeletVersion: "v1.27.0"
    kubeProxyVersion: "v1.27.0"
    operatingSystem: "linux"
    architecture: "amd64"
```

## 8. ASIL Service Operation Considerations

Considerations for Scenario actions for ASIL (Automotive Safety Integrity Level) service operation:

### 8.1 Action Constraints by ASIL Grade

| ASIL Grade | Real-time Constraints | Resource Requirements | Allowed Actions | Restricted Actions |
|------------|----------------------|----------------------|-----------------|-------------------|
| ASIL D | Strict real-time guarantee | Dedicated resources required | launch, modify(limited), rollback | update, pause |
| ASIL C | Semi-real-time guarantee | High priority resources | launch, modify, rollback | pause(long duration) |
| ASIL B | General real-time | Sufficient resource allocation | launch, update, modify, rollback, pause(short duration) | - |
| ASIL A | Low real-time | Standard resource allocation | All actions allowed | - |
| QM | No real-time requirement | Standard resource allocation | All actions allowed | - |

### 8.2 Package Type Recommendations for ASIL Services

| ASIL Grade | Recommended Package Types | Notes |
|------------|---------------------------|-------|
| ASIL D | time-critical + redundant | Most strict safety requirements, deterministic execution time and redundancy essential |
| ASIL C | time-critical + redundant or either one | High safety requirements, deterministic execution and redundancy both or individually applied depending on situation |
| ASIL B | time-critical or resource-bounded | Medium level safety requirements, real-time or resource isolation needed |
| ASIL A | resource-bounded or secure | Low safety requirements, resource guarantee or enhanced security needed |
| QM | plain or update | General quality management level, basic package types sufficient |

### 8.3 ASIL Service Scenario Example

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: emergency-brake-activation
  annotations:
    description: "Emergency brake system activation"
    priority: "critical"
    safetyLevel: "ASIL-D"
  labels:
    function: "safety-critical"
    category: "braking"
spec:
  condition:
    vehicleStatus: "MOVING"
    emergencyBrakeRequest: "ACTIVE"
  action: launch
  target:
    packages:
      - emergency-brake-controller
      - collision-detection-system
      - safety-monitor
  executionConstraints:
    maxStartupTime: "50ms"
    resourceGuarantee: true
    preemptLowerPriority: true
status:
  currentState: playing
  policyStatus: allowed
  lastTransitionTime: "2025-08-04T09:12:30Z"
  message: "Emergency brake system activated"
  executionMetrics:
    actualStartupTime: "42ms"
    resourceUtilization: "90%"
    responseTime: "5ms"
```

## 9. Resource Relationship Diagram

The following diagram shows the relationships between PICCOLO resources:

```
Scenario1     Scenario2     Scenario3
   |              |             |
   v              v             v
Package1 <-------Package2-----> Package3
   |                |             |
   v                v             v
Model1,Model2    Model3,Model4  Model5,Model6
   |                |             |
   v                v             v
Volume1,Volume2   Volume3      Volume4,Volume5
Network1          Network2     Network3
```

### 9.1 Resource Relationship Definitions

- **Scenario**: Defines actions for Packages based on vehicle state conditions
  - One Scenario can target multiple Packages
  - Multiple Scenarios can target the same Package (in this case, multiple Scenarios apply to one Package)

- **Package**: Groups multiple Models for joint management
  - One Package can include multiple Models
  - Each Model declared in a Package runs independently
  - The same Model definition can be included in multiple Packages, but each runs as an independent instance

- **Model**: Container execution unit that provides actual services
  - Each Model can connect to one or more Volumes and Networks
  - Models run on specific Nodes

- **Volume**: Provides data storage mounted to Models
  - One Volume can be shared by multiple Models (depending on accessMode)

- **Network**: Network resource for communication between Models
  - Multiple Models can connect to the same Network

- **Node**: Physical/virtual computing resource where Models run
  - One Node can host multiple Models

### 9.2 Resource Relationship Examples

- **Multi-Package Execution Scenario**:
  - The `emergency-response` scenario can target three packages: `brake-controller`, `alert-system`, and `sensor-fusion`
  - Each package is managed independently but executed together according to the scenario

- **Multi-Model Package**:
  - The `sensor-fusion` package can include three models: `camera-processor`, `lidar-processor`, and `fusion-algorithm`
  - The three models belong to the same package but run independently
  - Each model has its own resource requirements and configurations

- **Duplicate Target Package**:
  - Both the `night-driving` scenario and the `highway-driving` scenario can target the `vision-enhancement` package
  - If both scenarios' conditions are met, the package is affected by both scenarios
  - Conflicting actions are handled according to priority rules

## 10. Resource State Management Reference

Resource state management is a core function of the PICCOLO framework, responsible for managing the lifecycle and state transitions of each resource type (Scenario, Package, Model, Volume, Network, Node). Detailed information on resource state management is defined in the separate document [`PICCOLO_Resource_StateManagement.md`](/home/edo/2025/projects/docs/workspace/specs/KR/PICCOLO_Resource_StateManagement.md), which includes:

1. **Resource State Machine Definitions**: State definitions, state transition diagrams, and transition rules for each resource type
2. **Resource Monitoring Methods**: Monitoring items, methods, cycles, and importance for each resource type
3. **State Reconciliation Process**: Reconciliation steps, reconciliation strategies by resource type, conflict resolution methods
4. **Error Recovery Methods**: Error detection and recovery methods for each resource type, recovery time objectives
5. **ASIL Service State Management Specialized Features**: Monitoring and recovery strategies by ASIL grade, state transition rules
6. **StateManager Implementation**: StateManager architecture, operation method, API

Resource state management is performed centrally by the StateManager component, ensuring stable and safe vehicle software platform operation through continuous monitoring, state reconciliation, and error recovery mechanisms.

## 11. Conclusion

The PICCOLO framework's resource model provides a container management method optimized for the vehicle environment. Through six core resource types—Scenario, Package, Model, Volume, Network, and Node—it enables dynamic service management, resource allocation, service isolation, and security according to vehicle state.

In particular, it provides enhanced management capabilities for vehicle safety-critical services by combining various package types, such as applying time-critical and redundant package types simultaneously for ASIL service operation.

State management for each resource follows the state machine defined in the separate document [`PICCOLO_Resource_StateManagement.md`](/home/edo/2025/projects/docs/workspace/specs/KR/PICCOLO_Resource_StateManagement.md) and is centrally monitored, reconciled, and recovered by the StateManager component. This enables the construction of a stable and safe vehicle software platform.
