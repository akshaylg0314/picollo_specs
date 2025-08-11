# PICCOLO Framework Feature Specification

**Document Information:**
- Document ID: PICCOLO-SPEC-2025-FULL
- Version: 1.0
- Date: 2025-07-08 05:13:35
- Author: edo-lee_LGESDV
- Category: Integrated Feature Specification

## Table of Contents

1. [Overview](#1-overview)
2. [Core Features](#2-core-features)
3. [Component-specific Features](#3-component-specific-features)
4. [Resource Definition and Management](#4-resource-definition-and-management)
5. [Special Managers](#5-special-managers)
   - [DeviceManager](#51-devicemanager-details)
   - [AudioManager](#52-audiomanager-details)
   - [WindowManager](#53-windowmanager-details)
   - [CloudManager](#54-cloudmanager-details)
   - [NetworkManager (Pharos)](#54-networkmanager)
   - [StorageManager](#56-storagemanager-details)
6. [Operation Scenarios](#6-operation-scenarios)
7. [User Interface](#7-user-interface)
8. [Security and Safety](#8-security-and-safety)
9. [Integration and Scalability](#9-integration-and-scalability)
10. [Performance and Scalability](#10-performance-and-scalability)
11. [Implementation Requirements](#11-implementation-requirements)
12. [Conclusion](#12-conclusion)
13. [Appendix: Architecture Diagram](#13-appendix-architecture-diagram)

## 1. Overview

PICCOLO is an OS-optimized, independent container framework for vehicle environments, managing in-vehicle software services using Podman as a base and providing a Kubernetes-style declarative access method. This document describes the functional requirements and features of the PICCOLO framework.

### 1.1 Purpose

PICCOLO is designed with the following objectives:

- Provide unified service management across various operating systems
- Deploy and manage container-based services optimized for vehicle environments
- Simplify service lifecycle management through declarative configuration
- Provide a deterministic execution environment for critical systems
- Proactive monitoring of system status and performance
- Dynamic service composition based on vehicle status

### 1.2 Scope

This document covers the following features of the PICCOLO framework:

- Core component features
- Resource definition and management
- Scenario-based workflow management
- User interface and configuration management
- Monitoring and logging
- Real-time control and networking features
- Security and safety features
- Special managers (DeviceManager, AudioManager, WindowManager)

## 2. Core Features

### 2.1 Feature Summary Table

| Feature | Description | Key Characteristics |
|---------|-------------|---------------------|
| Declarative Resource Management | YAML-based resource definition and management | • Scenario, Package, Model, Volume, Network<br>• ETCD-based state storage<br>• Version control |
| Container Lifecycle Management | Podman-based container management | • Create, start, stop, restart, delete<br>• Resource limits and allocation<br>• Volume and network management |
| Vehicle State-Based Scenarios | Service management based on vehicle state | • Conditional service activation<br>• Dynamic workload adjustment<br>• Priority-based management |
| Monitoring and Remote Management | System state monitoring and management | • Container/hardware metric collection<br>• Cloud data transmission<br>• Alert and notification management |
| Configuration Management Interface | UI and CLI-based configuration management | • Structural resource navigation/editing<br>• Template-based resource creation<br>• Version management and rollback |
| Real-time Control and Networking | Deterministic execution environment | • TIMPANI deterministic scheduling<br>• PHAROS eBPF-based networking<br>• QoS guarantee |

### 2.2 Declarative Resource Management

PICCOLO defines and manages vehicle services through the following declarative resources:

- **Scenario**: Defines vehicle state conditions, target packages, and actions
- **Package**: Defines groups of models (containers) deployed together
- **Model**: Defines individual containerized services
- **Volume**: Defines persistent data storage
- **Network**: Defines network configuration and connections

These resources are defined in YAML format and stored in ETCD to maintain consistent state across the system.

### 2.3 Container Lifecycle Management

Provides container lifecycle management based on Podman:

- Container creation, starting, stopping, restarting, deletion
- Resource limitation and allocation management
- Environment variable and configuration management
- Volume mounting and data persistence
- Network configuration and connection management
- Container state monitoring and logging

### 2.4 Vehicle State-Based Scenarios

Scenario-based management that automatically starts, updates, or stops services based on the current vehicle state:

- Service activation based on conditions such as vehicle speed, ignition state, battery level, etc.
- Dynamic service configuration in response to user interaction or system events
- Automatic workload adjustment for resource optimization
- Priority-based service management in safety-critical situations

### 2.5 Monitoring and Remote Management

Comprehensive monitoring and management of container and system state:

- Monitor container resource usage (CPU, memory, disk, network)
- Monitor hardware system state (temperature, voltage, fan speed, storage)
- Log collection, aggregation, and analysis
- Data transmission to cloud services
- Alert and notification management
- Dashboard and visualization

### 2.6 Configuration Management Interface

UI and CLI interfaces for easy system configuration management:

- Structural resource navigation and editing
- Template-based new resource creation
- Real-time validation and application of changes
- Version history management and rollback
- Permission-based access control

### 2.7 Real-time Control and Networking

Deterministic execution environment for safety-critical functions:

- Deterministic task execution via TIMPANI scheduler
- CPU isolation and real-time priority assignment
- High-speed packet processing via PHAROS networking (eBPF-based)
- Packet filtering, routing, and firewall functions
- QoS (Quality of Service) guarantee

## 3. Component-specific Features

### 3.1 Component Feature Summary Table

| Component | Key Features | Interface |
|-----------|-------------|-----------|
| APIServer | • Provides REST API<br>• Authentication and authorization<br>• Request processing<br>• Monitoring and diagnostics | • REST API<br>• gRPC<br>• WebSocket |
| FilterGateway | • Multi-protocol support<br>• Data filtering<br>• Message routing<br>• Data buffering | • Vehicle communication interface<br>• Internal API |
| PolicyManager | • Safety policy management<br>• Security policy management<br>• Resource policy management<br>• Policy violation response | • Internal API<br>• ETCD watch |
| ActionController | • Container lifecycle management<br>• Command translation<br>• Resource configuration<br>• Real-time control integration | • Podman API<br>• Internal API |
| StateManager | • State monitoring<br>• State management<br>• Automatic recovery<br>• State reporting | • Internal API<br>• ETCD watch |
| NodeAgent | • File management<br>• Configuration deployment<br>• Local resource management<br>• Node state reporting | • File system API<br>• Internal API |
| MonitoringServer | • Metric collection<br>• Log management<br>• Alert management<br>• Cloud integration<br>• Visualization | • Monitoring API<br>• Cloud API |
| SettingsService | • Configuration management<br>• UI interface<br>• CLI interface<br>• Change management<br>• User management | • Web UI<br>• CLI<br>• REST API |
| TIMPANI Scheduler | • Deterministic scheduling<br>• CPU management<br>• Timing analysis<br>• Safety-critical workload management | • Kernel scheduling API<br>• Internal API |
| PHAROS Networking | • eBPF-based packet processing<br>• Packet filtering<br>• Traffic control<br>• Network telemetry | • eBPF/XDP API<br>• Network stack API |

### 3.2 APIServer

Component providing communication interface with external systems:

- **Provides REST API**
  - Endpoints for resource creation, retrieval, update, and deletion
  - Versioned API support
  - Structured response format (JSON/YAML)

- **Authentication and Authorization**
  - Token-based authentication
  - Role-based access control
  - API key management

- **Request Processing**
  - Request validation and routing
  - Request rate limiting
  - Load balancing

- **Monitoring and Diagnostics**
  - API usage statistics
  - Performance metric collection
  - Diagnostic endpoints

### 3.3 FilterGateway

Component for receiving and processing vehicle data:

- **Multi-protocol Support**
  - Supports vehicle communication protocols such as CAN, FlexRay, Ethernet
  - Protocol conversion and normalization

- **Data Filtering**
  - Rule-based data filtering
  - Duplicate data removal
  - Data transformation and enrichment

- **Message Routing**
  - Priority-based message processing
  - Destination-based message routing
  - Event-based message distribution

- **Data Buffering**
  - Buffering for peak load management
  - Data caching during network interruptions
  - Batch processing optimization

### 3.4 PolicyManager

Component for managing and applying safety and security policies:

- **Safety Policy Management**
  - ISO 26262 compliance verification
  - Safety-critical function monitoring
  - Functional safety state evaluation

- **Security Policy Management**
  - Access control policy application
  - Data protection policy management
  - Network security policy application

- **Resource Policy Management**
  - Resource allocation and limit policies
  - Ensures fair resource sharing
  - Priority-based resource management

- **Policy Violation Response**
  - Violation detection and reporting
  - Automatic recovery actions
  - Violation history management

### 3.5 ActionController

Component for managing containers by executing Podman commands:

- **Container Lifecycle Management**
  - Create, start, stop, restart, remove containers
  - Image management (pull, delete)
  - Container state monitoring

- **Command Translation**
  - Translate declarative specifications into Podman commands
  - Execute commands and process results
  - Error handling and retry

- **Resource Configuration**
  - CPU, memory, storage allocation
  - Network configuration
  - Volume mounting

- **Real-time Control Integration**
  - Apply TIMPANI scheduling parameters
  - CPU isolation for deterministic execution
  - Real-time priority configuration

### 3.6 StateManager

Component for managing container and system state:

- **State Monitoring**
  - Real-time tracking of container state
  - Resource usage monitoring
  - Event and state change detection

- **State Management**
  - Resolve state inconsistencies
  - Fault detection and recovery
  - State history tracking

- **Automatic Recovery**
  - Restart failed containers
  - Detect and handle abnormal states
  - Graceful shutdown management

- **State Reporting**
  - Generate state summaries
  - State change notifications
  - State query interface

### 3.7 NodeAgent

Component for managing node file system and resources:

- **File Management**
  - Create, modify, delete files
  - Directory structure management
  - File permission management

- **Configuration Deployment**
  - Deploy configuration files
  - Update settings
  - System environment configuration

- **Local Resource Management**
  - Disk space management
  - Local cache management
  - Temporary file cleanup

- **Node State Reporting**
  - Hardware state monitoring
  - Resource usage reporting
  - Node availability reporting

### 3.8 MonitoringServer

Component for collecting and analyzing system metrics and logs:

- **Metric Collection**
  - Container metrics (CPU, memory, disk, network)
  - System metrics (hardware state, temperature, voltage)
  - Custom metrics

- **Log Management**
  - Log collection and aggregation
  - Log filtering and search
  - Log rotation and archiving

- **Alert Management**
  - Threshold-based alerts
  - Anomaly detection alerts
  - Alert routing and escalation

- **Cloud Integration**
  - Data transmission to cloud services
  - Offline caching and synchronization
  - Data compression and optimization

- **Visualization**
  - Dashboards and graphs
  - Real-time monitoring views
  - Report generation

### 3.9 SettingsService

Component providing UI and CLI interfaces for system configuration management:

- **Configuration Management**
  - Load and save YAML configurations
  - Configuration validation and application
  - Configuration history management

- **UI Interface**
  - Hierarchical resource navigation
  - Attribute-based editor
  - Template library
  - Drag-and-drop resource management
  - Real-time validation feedback

- **CLI Interface**
  - Command-line resource management
  - Batch operations and scripting support
  - Pipeline integration
  - Remote management support
  - JSON/YAML format support

- **Change Management**
  - Change validation and conflict detection
  - Atomic change application
  - Rollback feature

- **User Management**
  - Permission-based access control
  - User authentication and authorization
  - Change audit logging

### 3.10 TIMPANI Scheduler

Component for managing deterministic task execution:

- **Deterministic Scheduling**
  - Periodic task scheduling
  - Deadline-based execution
  - Jitter minimization

- **CPU Management**
  - CPU core allocation and isolation
  - Interrupt routing optimization
  - Real-time priority management

- **Timing Analysis**
  - Execution time monitoring
  - Timing violation detection
  - Performance analysis and optimization

- **Safety-critical Workload Management**
  - Isolation of safety-critical tasks
  - Priority inheritance and ceiling
  - Guarantee deterministic behavior

### 3.11 PHAROS Networking

Component providing high-performance, low-latency networking:

- **eBPF-based Packet Processing**
  - Utilizes XDP (eXpress Data Path)
  - High-speed packet processing
  - Kernel bypass optimization

- **Packet Filtering**
  - Fine-grained packet filter rules
  - Protocol and port-based filtering
  - State-based packet inspection

- **Traffic Control**
  - QoS (Quality of Service) management
  - Bandwidth limitation and allocation
  - Priority-based packet scheduling

- **Network Telemetry**
  - Packet flow monitoring
  - Network performance measurement
  - Anomaly traffic detection

## 4. Resource Definition and Management

### 4.1 Resource Type Summary Table

| Resource | Description | Key Attributes |
|----------|-------------|---------------|
| Scenario | Service activation rules based on vehicle state conditions | • condition<br>• action<br>• target |
| Package | Group of models deployed together | • pattern<br>• models<br>• resource connections |
| Model | Individual containerized service | • container definition<br>• resource requests/limits<br>• scheduling configuration |
| Volume | Persistent data storage definition | • size<br>• accessModes<br>• reuse policy |
| Network | Container network definition | • type<br>• QoS settings<br>• packet filtering rules |

### 4.2 Scenario Resource

Defines rules for activating, updating, or removing services based on vehicle state conditions.

**Features:**
- Define vehicle state conditions (speed, ignition state, battery level, etc.)
- Specify actions to execute when conditions are met (launch, update, delete)
- Specify target Package
- Manage priorities and dependencies
- Support nested and composite conditions

**Template:**
```yaml
apiVersion: piccolo.ai/v1
kind: Scenario
metadata:
  name: "{{SCENARIO_NAME}}"        # Scenario resource name (e.g., "monitoring-scenario")
  namespace: "{{NAMESPACE}}"       # Namespace (e.g., "default", "monitoring")
  labels:
    app: "{{APP_LABEL}}"           # Application label (e.g., "monitoring")
spec:
  description: "{{DESCRIPTION}}"   # Scenario description (e.g., "CPU usage monitoring scenario")
  condition:
    type: "{{CONDITION_TYPE}}"     # Condition type (e.g., "Periodic", "Event", "Manual")
    schedule: "{{SCHEDULE}}"       # Schedule (Cron expression for Periodic type, e.g., "*/5 * * * *")
    eventSource:
      type: "{{EVENT_TYPE}}"       # Event type (e.g., "ResourceChange", "MetricThreshold")
      resource: "{{RESOURCE_TYPE}}" # Resource type (e.g., "Package", "Model")
      name: "{{RESOURCE_NAME}}"    # Resource name (e.g., "example-package")
  action:
    - type: "{{ACTION_TYPE}}"      # Action type (e.g., "Notification", "CommandExecution")
      target: "{{TARGET}}"         # Target (e.g., "email", "slack", "webhook")
      recipients: ["{{RECIPIENT}}"] # Recipient list (e.g., ["admin@example.com"])
      message: "{{MESSAGE}}"       # Message (e.g., "Scenario trigger notification")
  resources:
    packages:
      - name: "{{PACKAGE_NAME}}"   # Package name (e.g., "example-package")
        namespace: "{{NAMESPACE}}" # Namespace (e.g., "default")
    models:
      - name: "{{MODEL_NAME}}"     # Model name (e.g., "example-model")
        namespace: "{{NAMESPACE}}" # Namespace (e.g., "default")
  thresholds:
    cpu: "{{CPU_THRESHOLD}}"       # CPU threshold (e.g., "80%")
    memory: "{{MEMORY_THRESHOLD}}" # Memory threshold (e.g., "75%")
    responseTime: "{{RESPONSE_TIME_THRESHOLD}}" # Response time threshold (e.g., "500ms")
status:
  phase: "{{PHASE}}"               # Phase (e.g., "Active", "Inactive", "Failed", "Completed")
  lastExecutionTime: "{{LAST_EXECUTION_TIME}}" # Last execution time (e.g., "2023-06-15T10:30:00Z")
  executionCount: {{EXECUTION_COUNT}} # Execution count (e.g., 42)
  lastExecutionStatus: "{{LAST_STATUS}}" # Last execution status (e.g., "Succeeded", "Failed", "Pending")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # Condition type (e.g., "Ready")
      status: "{{STATUS}}"         # Status (e.g., "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # Last transition time (e.g., "2023-06-15T10:30:00Z")
      reason: "{{REASON}}"         # Reason (e.g., "ScenarioReady")
      message: "{{MESSAGE}}"       # Message (e.g., "Scenario is ready to run")
  metrics:
    executionTime: "{{EXECUTION_TIME}}" # Execution time (e.g., "120ms")
    successRate: "{{SUCCESS_RATE}}" # Success rate (e.g., "98.5%")
    triggerCount: {{TRIGGER_COUNT}} # Trigger count (e.g., 150)
```
**Example:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: driver-assistance-activate
spec:
  condition:
    vehicleSpeed: ">30"
    ignitionState: "ON"
    driverSeatOccupied: "true"
  action: launch
  target: driver-assistance-package
```

### 4.3 Package Resource

Defines a group of models deployed together.

**Features:**
- Resource (volume, network) connection
- Node allocation management
- Package metadata management

**Template:**
```yaml
apiVersion: piccolo.ai/v1
kind: Package
metadata:
  name: "{{PACKAGE_NAME}}"         # Package resource name (e.g., "example-package")
  namespace: "{{NAMESPACE}}"       # Namespace (e.g., "default")
  labels:
    app: "{{APP_LABEL}}"           # Application label (e.g., "adas")
spec:
  description: "{{DESCRIPTION}}"   # Package description (e.g., "ADAS functionality package")
  type: "{{PACKAGE_TYPE}}"         # Package type (e.g., "StandardPackage", "CustomPackage", "SystemPackage")
  version: "{{VERSION}}"           # Version (e.g., "1.0.0")
  models:
    - name: "{{MODEL_NAME}}"       # Model name (e.g., "model-a")
      namespace: "{{NAMESPACE}}"   # Namespace (e.g., "default")
      role: "{{ROLE}}"             # Role (e.g., "primary", "secondary")
  dependencies:
    - name: "{{DEPENDENCY_NAME}}"  # Dependency package name (e.g., "dependency-package")
      namespace: "{{NAMESPACE}}"   # Namespace (e.g., "default")
      version: "{{VERSION}}"       # Version (e.g., "1.0.0")
  configMaps:
    - name: "{{CONFIG_MAP_NAME}}"  # ConfigMap name (e.g., "package-config")
      namespace: "{{NAMESPACE}}"   # Namespace (e.g., "default")
  secrets:
    - name: "{{SECRET_NAME}}"      # Secret name (e.g., "package-secrets")
      namespace: "{{NAMESPACE}}"   # Namespace (e.g., "default")
status:
  phase: "{{PHASE}}"               # Phase (e.g., "Pending", "Running", "Failed", "Terminated")
  startTime: "{{START_TIME}}"      # Start time (e.g., "2023-06-15T09:00:00Z")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # Condition type (e.g., "Available")
      status: "{{STATUS}}"         # Status (e.g., "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # Last transition time (e.g., "2023-06-15T09:05:00Z")
      reason: "{{REASON}}"         # Reason (e.g., "PackageAvailable")
      message: "{{MESSAGE}}"       # Message (e.g., "Package is available and operational")
  modelStatus:
    - name: "{{MODEL_NAME}}"       # Model name (e.g., "model-a")
      status: "{{STATUS}}"         # Status (e.g., "Running", "Failed")
      startTime: "{{START_TIME}}"  # Start time (e.g., "2023-06-15T09:01:00Z")
  metrics:
    healthScore: "{{HEALTH_SCORE}}" # Health score (e.g., "95%")
    uptime: "{{UPTIME}}"           # Uptime (e.g., "99.9%")
    packageType:
      type: "{{PACKAGE_TYPE}}"     # Package type (e.g., "StandardPackage")
      count: {{COUNT}}             # Count (e.g., 1)
      operationalStatus: "{{OPERATIONAL_STATUS}}" # Operational status (e.g., "Normal", "Degraded")
    modelRelationships:
      total: {{TOTAL_MODELS}}      # Total models (e.g., 2)
      healthy: {{HEALTHY_MODELS}}  # Healthy models (e.g., 2)
```
**Example:**
```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: driver-assistance-package
  annotations:
    version: "2.1"
    priority: "high"
spec:
  pattern:
    - type: distributed
  models:
    - name: object-detection-model
      node: VisionHPC
      resources:
        volume: vision-data-volume
        network: sensor-network
    - name: lane-keeping-model
      node: ControlHPC
      resources:
        volume: control-data-volume
        network: control-network
```

### 4.4 Model Resource

Defines an individual containerized service.

**Features:**
- Define container image and configuration
- Set resource requests and limits
- Configure environment variables and volume mounts
- Set security contexts
- Define lifecycle hooks and probes
- Configure time deterministic scheduling (TIMPANI)
- Access specialized manager resources

**Template:**
```yaml
apiVersion: piccolo.ai/v1
kind: Model
metadata:
  name: "{{MODEL_NAME}}"           # Model resource name (e.g., "example-model")
  namespace: "{{NAMESPACE}}"       # Namespace (e.g., "default")
  labels:
    app: "{{APP_LABEL}}"           # Application label (e.g., "adas")
spec:
  description: "{{DESCRIPTION}}"   # Model description (e.g., "Object detection model")
  image: "{{IMAGE}}"               # Image (e.g., "registry.example.com/models/example-model:v1.0.0")
  resources:
    limits:
      cpu: "{{CPU_LIMIT}}"         # CPU limit (e.g., "2")
      memory: "{{MEMORY_LIMIT}}"   # Memory limit (e.g., "4Gi")
      gpu: "{{GPU_LIMIT}}"         # GPU limit (e.g., "1")
    requests:
      cpu: "{{CPU_REQUEST}}"       # CPU request (e.g., "1")
      memory: "{{MEMORY_REQUEST}}" # Memory request (e.g., "2Gi")
      gpu: "{{GPU_REQUEST}}"       # GPU request (e.g., "1")
  ports:
    - name: "{{PORT_NAME}}"        # Port name (e.g., "http")
      containerPort: {{CONTAINER_PORT}} # Container port (e.g., 8080)
      protocol: "{{PROTOCOL}}"     # Protocol (e.g., "TCP", "UDP")
  envFrom:
    - configMapRef:
        name: "{{CONFIG_MAP_NAME}}" # ConfigMap name (e.g., "model-config")
    - secretRef:
        name: "{{SECRET_NAME}}"    # Secret name (e.g., "model-secrets")
  volumeMounts:
    - name: "{{VOLUME_NAME}}"      # Volume name (e.g., "data-volume")
      mountPath: "{{MOUNT_PATH}}"  # Mount path (e.g., "/data")
  volumes:
    - name: "{{VOLUME_NAME}}"      # Volume name (e.g., "data-volume")
      persistentVolumeClaim:
        claimName: "{{PVC_NAME}}"  # PVC name (e.g., "model-data-pvc")
  asil:
    level: "{{ASIL_LEVEL}}"        # ASIL level (e.g., "A", "B", "C", "D")
    safetyMechanism:
      enabled: {{SAFETY_ENABLED}}  # Safety mechanism enabled (true/false)
      type: "{{SAFETY_TYPE}}"      # Safety mechanism type (e.g., "Redundancy", "Monitoring", "Isolation")
status:
  phase: "{{PHASE}}"               # Phase (e.g., "Pending", "Running", "Failed", "Terminated")
  podName: "{{POD_NAME}}"          # Pod name (e.g., "example-model-pod-xyz123")
  startTime: "{{START_TIME}}"      # Start time (e.g., "2023-06-15T08:30:00Z")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # Condition type (e.g., "Ready")
      status: "{{STATUS}}"         # Status (e.g., "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # Last transition time (e.g., "2023-06-15T08:35:00Z")
      reason: "{{REASON}}"         # Reason (e.g., "ModelReady")
      message: "{{MESSAGE}}"       # Message (e.g., "Model container is running and ready")
  metrics:
    uptime: "{{UPTIME}}"           # Uptime (e.g., "99.8%")
    restartCount: {{RESTART_COUNT}} # Restart count (e.g., 0)
    cpuUsage: "{{CPU_USAGE}}"      # CPU usage (e.g., "45%")
    memoryUsage: "{{MEMORY_USAGE}}" # Memory usage (e.g., "60%")
    gpuUsage: "{{GPU_USAGE}}"      # GPU usage (e.g., "30%")
    healthStatus: "{{HEALTH_STATUS}}" # Health status (e.g., "Healthy", "Degraded", "Unhealthy")
    asilCompliance:
      status: "{{ASIL_STATUS}}"    # ASIL compliance status (e.g., "Compliant", "Non-Compliant")
      lastVerificationTime: "{{VERIFICATION_TIME}}" # Last verification time (e.g., "2023-06-15T12:00:00Z")
```
**Example:**
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
```

### 4.5 Volume Resource

Defines volumes for persistent data storage.

**Example:**
```yaml
apiVersion: piccolo.io/v1
kind: Volume
metadata:
  name: vision-data-volume
spec:
  size: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
```

### 4.6 Network Resource

Defines networks for container-to-container and external communications.

**Features:**
- Set network type and bandwidth
- Configure security policies and QoS
- Configure eBPF-based fast path (PHAROS)
- Define packet filtering rules
- Set network telemetry

**Example:**
```yaml
apiVersion: v1
kind: Network
metadata:
  name: "{{NETWORK_NAME}}"        # Network resource name (e.g., "sensor-network")
  labels:
    app: "{{APP_LABEL}}"          # Application label (e.g., "adas")    
spec:    
  node_name: "{{NODE_NAME}}"      # Node name (e.g., "vm1", "soc2")
  bridge:
    name: "{{BRIDGE_NAME}}"       # Bridge name (e.g., "br0")
    type: "{{BRIDGE_TYPE}}"       # Bridge type (e.g., "bridge")
  ingress:
    port: "{{INGRESS_PORT}}"      # Ingress port (e.g., "5555")
    protocol: "{{PROTOCOL}}"      # Protocol (e.g., "udp", "tcp", "dds")
    topic: "{{TOPIC_NAME}}"       # Topic name (when using DDS protocol, e.g., "sensor_data")
  egress:
    port: "{{EGRESS_PORT}}"      # Egress port (e.g., "5555")
    protocol: "{{PROTOCOL}}"      # Protocol (e.g., "udp", "tcp", "dds")
    topic: "{{TOPIC_NAME}}"       # Topic name (when using DDS protocol, e.g., "sensor_data")
  qos:
    latency: "{{LATENCY_LEVEL}}"  # Latency level (e.g., "low", "medium", "ultra-low")
    priority: "{{PRIORITY}}"      # Priority (e.g., "high", "critical", "normal")
  fastPath:
    enabled: {{FAST_PATH}}        # Fast path enabled (true/false)
    optimizationLevel: "{{OPTIMIZATION_LEVEL}}" # Optimization level (e.g., "aggressive", "balanced")
  security:
    securityPolicy: "{{SECURITY_POLICY}}" # Security policy (e.g., "strict", "standard")
    encryption: {{ENCRYPTION}}    # Encryption enabled (true/false)
```

## 5. Specialized Managers

PICCOLO provides six managers for handling specialized resources in vehicle environments:

### 5.1 Specialized Manager Summary Table

| Manager | Primary Purpose | Managed Resources |
|--------|----------|-----------|
| DeviceManager | Integrated management and access control of vehicle hardware devices | Sensors, cameras, actuators, input devices |
| AudioManager | Routing, mixing, and priority management of vehicle audio resources | Speakers, microphones, audio streams |
| WindowManager | Management of display resources and user interface screens | Displays, windows, input events |
| CloudManager | Management and optimization of vehicle-cloud hybrid workloads | Cloud services, data synchronization, network connections |
| NetworkManager (Pharos) | Management of in-vehicle network resources and communication optimization (separate framework) | Container networks, packet filtering, cloud connections |
| StorageManager | Management of in-vehicle storage resources and mount optimization | Container volumes, host storage, external storage |

### 5.1 DeviceManager Details

#### 5.1.1 Feature Overview

DeviceManager provides integrated management of access, control, and monitoring for various hardware devices in the vehicle.

**Key Features:**
- Hardware device abstraction and integrated interface
- Device access permission management and conflict prevention
- Device status monitoring and anomaly detection
- Power management and resource optimization
- Policy-based management of safety-critical devices

#### 5.1.2 DeviceManager Template

```yaml
apiVersion: piccolo.io/v1
kind: DeviceManager
metadata:
  name: ${devicemanager_name}
  namespace: ${namespace}
spec:
  devices:
    - id: ${device_id}
      type: ${device_type}
      description: ${description}
      accessMode: ${access_mode}
      securityLevel: ${security_level}
      powerPolicy: ${power_policy}
      metadata:
        ${key1}: ${value1}
        ${key2}: ${value2}
  accessControl:
    policies:
      - name: ${policy_name}
        deviceSelector:
          ${selector_key}: ${selector_value}
        access: ${policy_access}
        conditions:
          ${condition_key}: ${condition_value}
```

### 5.2 AudioManager Details

#### 5.2.1 Feature Overview

AudioManager is responsible for routing, mixing, and priority management of audio resources in the vehicle.

**Key Features:**
- Audio stream and device management
- Audio routing and mixing control
- Priority-based audio policy enforcement
- Volume and mute management
- Support for various audio scenarios (media, navigation, calls, notifications)

#### 5.2.2 AudioManager Template

```yaml
apiVersion: piccolo.io/v1
kind: AudioManager
metadata:
  name: ${audiomanager_name}
  namespace: ${namespace}
spec:
  devices:
    - id: ${audio_device_id}
      type: ${audio_device_type}
      channels: ${channels}
      description: ${description}
  streams:
    - name: ${stream_name}
      source: ${source}
      sink: ${sink}
      format: ${format}
      sampleRate: ${sample_rate}
      channels: ${stream_channels}
      priority: ${stream_priority}
      policy: ${stream_policy}
  routing:
    rules:
      - condition:
          ${rule_condition_key}: ${rule_condition_value}
        action: ${routing_action}
        target: ${routing_target}
  volumeControl:
    defaultLevel: ${default_volume_level}
    maxLevel: ${max_volume_level}
    perStream: ${per_stream_volume}
  policy:
    scenarioPolicies:
      - scenario: ${scenario_name}
        streamPriorities:
          ${stream_name}: ${priority}
        muteRules:
          ${stream_name}: ${mute_policy}
```

### 5.3 WindowManager Details

#### 5.3.1 Feature Overview

WindowManager manages vehicle display resources and window layouts.

**Key Features:**
- Display device management and configuration
- Window layout and z-order management
- Input device routing management
- Layout policy application based on vehicle state
- Window access permission management

#### 5.3.2 WindowManager Template

```yaml
apiVersion: piccolo.io/v1
kind: WindowManager
metadata:
  name: ${windowmanager_name}
  namespace: ${namespace}
spec:
  displays:
    - id: ${display_id}
      resolution: ${resolution}
      orientation: ${orientation}
      brightnessControl: ${brightness_control}
      colorCalibration:
        enabled: ${color_cal_enabled}
        profile: ${color_profile}
  windows:
    - name: ${window_name}
      display: ${display_id}
      position:
        x: ${x}
        y: ${y}
      size:
        width: ${width}
        height: ${height}
      zOrder: ${z_order}
      visible: ${visible}
      focusable: ${focusable}
      input:
        touch: ${touch_input}
        gesture: ${gesture_input}
        keyboard: ${keyboard_input}
      permissions:
        apps:
          - ${app_name}
        access: ${window_access}
  policies:
    - name: ${policy_name}
      condition:
        ${policy_condition_key}: ${policy_condition_value}
      layout:
        ${window_name}:
          visible: ${policy_visible}
          focusable: ${policy_focusable}
  inputRouting:
    default: ${default_window}
    rules:
      - source: ${input_source}
        target: ${input_target}
        condition:
          ${input_condition_key}: ${input_condition_value}
  accessControl:
    rules:
      - app: ${policy_app_name}
        allow:
          - ${allow_window}
        deny:
          - ${deny_window}
```

### 5.4 NetworkManager

The NetworkManager functionality is provided as a specialized feature through the Pharos framework. Pharos is a specialized solution for managing in-vehicle network resources and optimizing communications, which is integrated with PICCOLO.

Please refer to the Pharos documentation for more details.

### 5.5 CloudManager Details

#### 5.5.1 Feature Overview

CloudManager manages and optimizes hybrid workloads between the vehicle and cloud in the PICCOLO framework. This enables offloading resource-intensive tasks to the cloud or supporting seamless transitions between vehicle and cloud.

**Key Features:**
- Hybrid workload management
- Workload splitting and distributed execution
- Automatic workload offloading
- Real-time/non-real-time task differentiation
- Priority-based execution location determination

#### 5.5.2 CloudManager Template

```yaml
apiVersion: piccolo.io/v1
kind: CloudManager
metadata:
  name: ${cloudmanager_name}
  namespace: ${namespace}
spec:
  cloudServices:
    - name: ${service_name}
      provider: ${provider}
      region: ${region}
      credentials:
        secretName: ${secret_name}
        keyPath: ${key_path}
      endpoints:
        - name: ${endpoint_name}
          url: ${endpoint_url}
          protocol: ${endpoint_protocol}
  workloads:
    - name: ${workload_name}
      type: ${workload_type}
      placement: ${placement}
      requirements:
        cpu: ${cpu}
        memory: ${memory}
        gpu: ${gpu}
        latency: ${latency}
      fallback:
        strategy: ${fallback_strategy}
        timeout: ${fallback_timeout}
  dataSync:
    - name: ${sync_name}
      source: ${sync_source}
      destination: ${sync_destination}
      priority: ${sync_priority}
      schedule: ${sync_schedule}
      strategy: ${sync_strategy}
      conflict: ${sync_conflict}
  connections:
    primary: ${primary_conn}
    fallback:
      - ${fallback_conn}
    monitoring:
      interval: ${monitor_interval}
      threshold:
        latency: ${latency_threshold}
        packetLoss: ${packet_loss_threshold}
        bandwidth: ${bandwidth_threshold}
  offlineMode:
    caching:
      enabled: ${offline_cache_enabled}
      maxSize: ${offline_cache_max_size}
      ttl: ${offline_cache_ttl}
    localServices:
      - service: ${offline_service_name}
        mode: ${offline_service_mode}
  permissions:
    - modelSelector:
        matchLabels:
          ${model_label_key}: ${model_label_value}
      services:
        - ${permission_service}
      accessLevel: ${permission_level}
```

### 5.6 StorageManager Details

#### 5.6.1 Feature Overview

StorageManager is responsible for container storage and volume management within the PICCOLO framework.

**Key Features:**
- Support for various storage modes (virtual, host, shared)
- Container volume lifecycle management
- Volume mount and permission control
- Storage quota and usage monitoring
- Optimization for high-performance data access
- Storage resource protection and isolation

#### 5.6.2 StorageManager Template

```yaml
apiVersion: piccolo.io/v1
kind: StorageManager
metadata:
  name: ${storagemanager_name}
  namespace: ${namespace}
spec:
  defaultStorageMode: ${storage_mode}
  volumes:
    virtual:
      - name: ${virtual_volume_name}
        size: ${volume_size}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        emptyDir:
          medium: ${medium}
          sizeLimit: ${size_limit}
        permissions:
          mode: ${permission_mode}
          user: ${user_id}
          group: ${group_id}
    host:
      - name: ${host_volume_name}
        hostPath: ${host_path}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        permissions:
          mode: ${permission_mode}
          user: ${user_id}
          group: ${group_id}
        pathType: ${path_type}
        readOnly: ${read_only}
    shared:
      - name: ${shared_volume_name}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        protocol: ${protocol}
        server: ${server_ip}
        path: ${server_path}
        options: ${mount_options}
        credentials:
          secretName: ${secret_name}
        readOnly: ${read_only}
  volumeClaims:
    - name: ${claim_name}
      podSelector:
        matchLabels:
          ${pod_label_key}: ${pod_label_value}
      volumes:
        - name: ${volume_name}
          mountPath: ${mount_path}
          readOnly: ${read_only}
  quotas:
    - name: ${quota_name}
      podSelector:
        matchLabels:
          ${pod_label_key}: ${pod_label_value}
      limits:
        storage: ${storage_limit}
        inodes: ${inode_limit}
      actions:
        onLimitReached: ${limit_action}
  snapshots:
    enabled: ${snapshots_enabled}
    schedule:
      - name: ${schedule_name}
        volumeName: ${volume_name}
        cronExpression: ${cron_expr}
        retention:
          count: ${snapshot_count}
          duration: ${retention_period}
  dataProtection:
    encryption:
      enabled: ${encryption_enabled}
      volumes:
        - ${encrypted_volume}
      keyManagement:
        keySource: ${key_source}
        keyRotation: ${key_rotation}
    backup:
      enabled: ${backup_enabled}
      volumes:
        - ${backup_volume}
      schedule: ${backup_schedule}
      destination: ${backup_destination}
  performance:
    ioProfile: ${io_profile}
    caching:
      enabled: ${caching_enabled}
      volumes:
        - ${cached_volume}
      size: ${cache_size}
      policy: ${cache_policy}
  monitoring:
    metrics:
      enabled: ${metrics_enabled}
      interval: ${metrics_interval}
      exporters:
        - ${metrics_exporter}
    alerts:
      - name: ${alert_name}
        condition: ${alert_condition}
        severity: ${alert_severity}
  ```

## 6. Operational Scenarios

PICCOLO supports various vehicle operational scenarios:

### 6.1 Driver Assistance System Activation

Automatically activates the driver assistance system when the vehicle is traveling above a certain speed and the driver is present.

**Activation Conditions:**
- Vehicle speed > 30km/h
- Ignition state = ON
- Driver seat occupied = true

**Activated Services:**
- Object detection (camera, radar, lidar processing)
- Lane keeping assistance
- Adaptive cruise control

### 6.2 Parking Assistance System Activation

Starts the parking assistance system when the vehicle is reversing at low speed and the parking assist button is activated.

**Activation Conditions:**
- Vehicle speed < 10km/h
- Gear position = R (reverse)
- Parking assist button = ON

**Activated Services:**
- Surround view camera system
- Parking guidance system
- Obstacle detection and warning

### 6.3 Entertainment System Update

Updates the entertainment system when the vehicle is parked, has sufficient battery, and is connected to Wi-Fi.

**Activation Conditions:**
- Ignition state = OFF
- Battery level > 50%
- Wi-Fi connection = true

**Updated Services:**
- Media player
- Navigation system
- Infotainment UI

### 6.4 Vehicle Diagnostic System Execution

Runs the vehicle diagnostic system when diagnostic mode is activated and a service request is initiated.

**Activation Conditions:**
- Diagnostic mode = true
- Service request = initiated

**Activated Services:**
- System diagnostic tools
- Error code analyzer
- Service report generator

### 6.5 Idle Resource Cleanup

Cleans up non-essential services when the vehicle has been off for an extended period and battery is low.

**Activation Conditions:**
- Ignition state = OFF
- Idle time > 30 minutes
- Battery level < 40%

**Removed Services:**
- Non-essential background services
- Idle applications
- Temporary caches and logs

### 6.6 Specialized Manager Integration Scenarios

#### 6.6.1 Navigation While Driving Scenario

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: navigation-active-scenario
spec:
  condition:
    vehicleSpeed: ">5"
    ignitionState: "ON"
  action: launch
  target: navigation-package
```

In this scenario:
1. DeviceManager provides access to GPS sensor and speedometer
2. AudioManager sets navigation voice guidance as a primary stream and reduces media volume
3. WindowManager allocates larger screen area to the navigation window

#### 6.6.2 Parking Assistance Scenario

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: parking-assist-scenario
spec:
  condition:
    gearPosition: "R"
    vehicleSpeed: "<10"
  action: launch
  target: parking-assist-package
```

In this scenario:
1. DeviceManager provides access to rear cameras and ultrasonic sensors
2. AudioManager prioritizes parking sensor warning sounds and lowers other audio
3. WindowManager switches to parking mode layout with full-screen camera view

## 7. User Interface

### 7.1 Settings UI Panel

Graphical user interface for managing system configuration:

- **Navigation Tree**
  - Displays hierarchical resource structure
  - Groups by resource type
  - Quick navigation and search

- **Property Editor**
  - Structured form-based editing
  - Inline YAML editing
  - Real-time validation feedback

- **Template Library**
  - Predefined resource templates
  - Create and share custom templates
  - Quick resource creation based on templates

- **Version Management**
  - View change history
  - Compare with previous versions
  - Roll back to specific versions

- **Dashboard**
  - System status overview
  - Key metrics visualization
  - Alerts and events display

- **Specialized Manager Panels**
  - Device management panel
  - Audio management panel
  - Window management panel

### 7.2 Settings CLI

Command-line system management:

- **Resource Management**
  - Create, read, update, delete resources
  - Check resource status
  - Import and export resources

- **System Management**
  - Check system status
  - Apply and roll back configurations
  - Diagnostics and troubleshooting

- **Batch Operations**
  - Script support
  - Automation workflows
  - Bulk operation processing

- **Format Support**
  - JSON/YAML format support
  - Custom output formats
  - Pipeline integration

- **Remote Management**
  - SSH-based remote management
  - API token authentication
  - Secure shell sessions

- **Specialized Manager Commands**
  - Device management commands
  - Audio management commands
  - Window management commands

### 7.3 API Interface

Programmatic system management:

- **RESTful API**
  - Resource CRUD operations
  - Status queries and monitoring
  - Event stream subscription

- **WebSocket API**
  - Real-time event streaming
  - State change notifications
  - Bidirectional communication

- **gRPC API**
  - High-performance inter-service communication
  - Streaming support
  - Strong type checking

## 8. Security and Safety

### 8.1 Authentication and Authorization

- **User Authentication**
  - Token-based authentication
  - Certificate-based authentication
  - External authentication system integration

- **Authorization**
  - Role-based access control (RBAC)
  - Resource-level permissions
  - Granular operation permissions

- **Auditing**
  - Logging of all administrative operations
  - Change history tracking
  - Audit report generation

### 8.2 Network Security

- **Communication Encryption**
  - TLS-based encryption
  - Service-to-service mutual authentication
  - Certificate management

- **Network Separation**
  - Security zone separation
  - Micro-segmentation
  - Traffic filtering

- **Intrusion Detection**
  - Abnormal traffic detection
  - Attack pattern recognition
  - Security event alerts

### 8.3 Container Security

- **Image Security**
  - Image signing and verification
  - Vulnerability scanning
  - Run only approved images

- **Runtime Security**
  - Privilege minimization
  - Read-only file systems
  - System call restrictions

- **Security Context**
  - Non-root user execution
  - Capability limitations
  - Secure computing mode

### 8.4 Functional Safety

- **Safety Policies**
  - ISO 26262 compliance
  - Automotive Safety Integrity Level (ASIL) support
  - Safety case management

- **Fault Detection and Handling**
  - Failure detection mechanisms
  - Transition to safe state
  - Redundancy and diversity

- **Time Determinism**
  - Deterministic execution guarantee
  - Timing violation detection
  - Real-time performance monitoring

## 9. Integration and Extensibility

### 9.1 External System Integration

- **Vehicle System Integration**
  - Vehicle bus integration (CAN, FlexRay, Automotive Ethernet)
  - Vehicle diagnostic interface
  - Sensor system integration

- **Cloud Service Integration**
  - Data synchronization
  - Remote management and monitoring
  - OTA update support

- **Development Tool Integration**
  - CI/CD pipeline integration
  - Test automation
  - Code repository connection

### 9.2 Extension Mechanisms

- **Plugin Architecture**
  - Add custom components
  - Extend existing functionality
  - Third-party integration

- **Custom Resource Definitions**
  - Add user-defined resource types
  - Extend domain-specific resource models
  - Customize resource validation rules

- **Event Hooks**
  - Hooks reacting to system events
  - Execute user-defined actions
  - Integrate automation workflows

## 10. Performance and Scalability

### 10.1 Performance Characteristics

- **Response Times**
  - API request response: <100ms
  - Container startup: <3 seconds
  - Safety-critical functions: <10ms

- **Throughput**
  - Concurrent API requests: 1000 TPS
  - Container management operations: 100/minute
  - Event processing: 10,000/second

- **Resource Usage**
  - Minimum memory footprint: 512MB
  - Minimum CPU cores: 2
  - Storage requirements: 10GB

### 10.2 Scalability Characteristics

- **Node Scalability**
  - Maximum supported nodes: 10
  - Maximum containers per node: 100
  - Dynamic support for adding/removing nodes

- **Service Scalability**
  - Maximum number of models: 500
  - Maximum number of scenarios: 200
  - Maximum number of packages: 100

- **Storage Scalability**
  - Maximum ETCD data size: 8GB
  - Maximum configuration items: 100,000
  - Maximum event history: 1,000,000

## 11. Implementation Constraints

### 11.1 Hardware Constraints

- **Minimum Hardware Requirements**
  - CPU: Multi-core processor, minimum 4 cores
  - Memory: 8GB RAM
  - Storage: 64GB
  - Network: 1Gbps Ethernet

- **Recommended Hardware Configuration**
  - CPU: 8 cores, real-time processing capability
  - Memory: 16GB RAM, ECC memory
  - Storage: 128GB SSD
  - Network: 10Gbps Ethernet

### 11.2 Software Constraints

- **Operating System Requirements**
  - Linux kernel 5.10 or higher
  - PREEMPT_RT patch recommended
  - eBPF/XDP support

- **Runtime Requirements**
  - Podman 3.0 or higher
  - ETCD 3.4 or higher
  - Go 1.18 or higher
  - Python 3.9 or higher

- **Library Dependencies**
  - eBPF libraries
  - Real-time extension libraries
  - Vehicle communication protocol libraries
  - Graphics subsystem libraries

- **Device Driver Requirements**
  - Standard Linux device drivers
  - Specific vehicle sensor drivers
  - Graphics acceleration drivers
  - Audio subsystem drivers

- **Container Compatibility**
  - OCI-compatible container images
  - Rootless container support
  - cgroup v2 support
  - seccomp profile support

### 11.3 Performance Constraints

- **Timing Constraints**
  - Real-time task jitter: <1ms
  - Network latency: <10ms
  - Boot time: <30 seconds

- **Resource Constraints**
  - Container overhead: <10%
  - Network overhead: <5%
  - Storage I/O: Minimum 50MB/s

- **Throughput Constraints**
  - Maximum concurrent container deployments: 20
  - Maximum event processing rate: 10,000/second
  - Maximum log processing rate: 5MB/second

### 11.4 Environmental Constraints

- **Operating Environment**
  - Temperature: -20°C to 70°C
  - Power consumption: <25W
  - Vibration and shock resistance

- **Network Environment**
  - Support for intermittent connectivity
  - Handling of limited bandwidth
  - Handling of high latency