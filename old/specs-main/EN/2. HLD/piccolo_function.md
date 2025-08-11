# PICCOLO Framework Function Specification

**Document Information:**
- Document ID: PICCOLO-SPEC-2025-FULL
- Version: 1.0
- Date: 2025-07-08 05:13:35
- Author: edo-lee_LGESDV
- Classification: Functional Integration Specification

## Table of Contents

1. [Overview](#1-overview)
2. [Core Functions](#2-core-functions)
3. [Component Functions](#3-component-functions)
4. [Resource Definition and Management](#4-resource-definition-and-management)
5. [Specialized Managers](#5-specialized-managers)
   - [DeviceManager](#51-devicemanager-details)
   - [AudioManager](#52-audiomanager-details)
   - [WindowManager](#53-windowmanager-details)
   - [CloudManager](#54-cloudmanager-details)
   - [NetworkManager (Pharos)](#54-networkmanager)
   - [StorageManager](#56-storagemanager-details)
6. [Operational Scenarios](#6-operational-scenarios)
7. [User Interface](#7-user-interface)
8. [Security and Safety](#8-security-and-safety)
9. [Integration and Extensibility](#9-integration-and-extensibility)
10. [Performance and Scalability](#10-performance-and-scalability)
11. [Implementation Constraints](#11-implementation-constraints)
12. [Conclusion](#12-conclusion)
13. [Appendix: Architecture Diagrams](#13-appendix-architecture-diagrams)

## 1. Overview

PICCOLO is an OS-independent container framework optimized for vehicle environments, based on Podman with a Kubernetes-style declarative approach to manage vehicle software services. This document describes the functional requirements and capabilities of the PICCOLO framework.

### 1.1 Purpose

PICCOLO is designed with the following objectives:

- Provide consistent service management across diverse operating systems
- Deploy and manage container-based services optimized for vehicle environments
- Simplify service lifecycle management through declarative configuration
- Provide deterministic execution environments for safety-critical systems
- Enable comprehensive monitoring of system state and performance
- Support dynamic service configuration based on vehicle state

### 1.2 Scope

This document covers the following functions of the PICCOLO framework:

- Core component functions
- Resource definition and management
- Scenario-based workload management
- User interface and configuration management
- Monitoring and logging
- Real-time control and networking functions
- Security and safety features
- Specialized managers (DeviceManager, AudioManager, WindowManager)

## 2. Core Functions

### 2.1 Function Summary Table

| Function | Description | Key Features |
|------|------|-----------|
| Declarative Resource Management | YAML-based resource definition and management | • Scenario, Package, Model, Volume, Network<br>• ETCD-based state storage<br>• Version management |
| Container Lifecycle Management | Podman-based container management | • Create, start, stop, restart, delete<br>• Resource limitation and allocation<br>• Volume and network management |
| Vehicle State-based Scenarios | Service management based on vehicle state | • Conditional service activation<br>• Dynamic workload adjustment<br>• Priority-based management |
| Monitoring and Remote Management | System state monitoring and management | • Container/hardware metric collection<br>• Cloud data transmission<br>• Notification and alert management |
| Configuration Management Interface | UI and CLI-based configuration management | • Structured resource exploration/editing<br>• Template-based resource creation<br>• Version management and rollback |
| Real-time Control and Networking | Providing time-deterministic execution environment | • TIMPANI deterministic scheduling<br>• PHAROS eBPF-based networking<br>• QoS guarantees |

### 2.2 Declarative Resource Management

PICCOLO defines and manages vehicle services through the following declarative resources:

- **Scenario**: Defines vehicle state conditions, target packages, and actions
- **Package**: Defines groups of models (containers) deployed together
- **Model**: Defines individual containerized services
- **Volume**: Defines persistent data storage
- **Network**: Defines network configuration and connections

These resources are defined in YAML format and stored in ETCD to maintain a consistent state throughout the system.

### 2.3 Container Lifecycle Management

Provides container lifecycle management based on Podman:

- Container creation, start, stop, restart, deletion
- Resource limitation and allocation management
- Environment variable and configuration management
- Volume mounting and data persistence
- Network configuration and connection management
- Container state monitoring and logging

### 2.4 Vehicle State-based Scenarios

Scenario-based management that can automatically start, update, or stop services according to the vehicle's current state:

- Service activation based on vehicle speed, ignition state, battery level, etc.
- Dynamic service configuration in response to user interactions or system events
- Automatic workload adjustment for resource optimization
- Priority-based service management in safety-critical situations

### 2.5 Monitoring and Remote Management

Comprehensive monitoring and management of container and system state:

- Container resource usage monitoring (CPU, memory, disk, network)
- Hardware system state monitoring (temperature, voltage, fan speed, storage)
- Log collection, aggregation, and analysis
- Data transmission to cloud services
- Notification and alert management
- Dashboard and visualization

### 2.6 Configuration Management Interface

UI and CLI interfaces for easy system configuration management:

- Structured resource exploration and editing
- Template-based new resource creation
- Real-time validation and application of changes
- Version history management and rollback
- Permission-based access control

### 2.7 Real-time Control and Networking

Time-deterministic execution environment for safety-critical functions:

- Deterministic task execution through TIMPANI scheduler
- CPU isolation and real-time priority assignment
- Fast packet processing through PHAROS networking with eBPF
- Packet filtering, routing, and firewall functions
- Quality of Service (QoS) guarantees

## 3. Component Functions

### 3.1 Component Function Summary Table

| Component | Key Functions | Interfaces |
|----------|-----------|------------|
| APIServer | • Provide REST API<br>• Authentication and authorization<br>• Request processing<br>• Monitoring and diagnostics | • REST API<br>• gRPC<br>• WebSocket |
| FilterGateway | • Multi-protocol support<br>• Data filtering<br>• Message routing<br>• Data buffering | • Vehicle communication interface<br>• Internal API |
| PolicyManager | • Safety policy management<br>• Security policy management<br>• Resource policy management<br>• Policy violation response | • Internal API<br>• ETCD watch |
| ActionController | • Container lifecycle management<br>• Command transformation<br>• Resource configuration<br>• Real-time control integration | • Podman API<br>• Internal API |
| StateManager | • State monitoring<br>• State management<br>• Automatic recovery<br>• State reporting | • Internal API<br>• ETCD watch |
| NodeAgent | • File management<br>• Configuration deployment<br>• Local resource management<br>• Node state reporting | • File system API<br>• Internal API |
| MonitoringServer | • Metric collection<br>• Log management<br>• Alert management<br>• Cloud integration<br>• Visualization | • Monitoring API<br>• Cloud API |
| SettingsService | • Settings management<br>• UI interface<br>• CLI interface<br>• Change management<br>• User management | • Web UI<br>• CLI<br>• REST API |
| TIMPANI Scheduler | • Deterministic scheduling<br>• CPU management<br>• Timing analysis<br>• Safety-critical workload management | • Kernel scheduling API<br>• Internal API |
| PHAROS Networking | • eBPF-based packet processing<br>• Packet filtering<br>• Traffic control<br>• Network telemetry | • eBPF/XDP API<br>• Network stack API |

### 3.2 APIServer

Component providing communication interface with external systems:

- **REST API Provision**
  - Endpoints for resource creation, retrieval, modification, deletion
  - Support for versioned APIs
  - Structured response formats (JSON/YAML)

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
  - Performance metrics collection
  - Diagnostic endpoints

### 3.3 FilterGateway

Component for receiving and processing vehicle data:

- **Multi-protocol Support**
  - Support for vehicle communication protocols (CAN, FlexRay, Ethernet, etc.)
  - Protocol conversion and normalization

- **Data Filtering**
  - Rule-based data filtering
  - Duplicate data elimination
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
  - Functional safety state assessment

- **Security Policy Management**
  - Access control policy enforcement
  - Data protection policy management
  - Network security policy application

- **Resource Policy Management**
  - Resource allocation and limitation policies
  - Ensuring fair resource sharing
  - Priority-based resource management

- **Policy Violation Response**
  - Violation detection and reporting
  - Automatic recovery action execution
  - Violation history management

### 3.5 ActionController

Component for executing Podman commands to manage containers:

- **Container Lifecycle Management**
  - Container creation, start, stop, restart, removal
  - Image management (import, deletion)
  - Container state monitoring

- **Command Transformation**
  - Converting declarative specifications to Podman commands
  - Command execution and result processing
  - Error handling and retry

- **Resource Configuration**
  - CPU, memory, storage allocation
  - Network configuration
  - Volume mounting

- **Real-time Control Integration**
  - Applying TIMPANI scheduling parameters
  - CPU isolation for deterministic execution
  - Real-time priority configuration

### 3.6 StateManager

Component for managing container and system states:

- **State Monitoring**
  - Real-time tracking of container states
  - Resource usage monitoring
  - Event and state change detection

- **State Management**
  - Resolving state discrepancies
  - Failure detection and recovery
  - State history tracking

- **Automatic Recovery**
  - Restarting failed containers
  - Detecting and addressing abnormal states
  - Managing graceful termination

- **State Reporting**
  - Generating state summaries
  - State change notifications
  - State query interface

### 3.7 NodeAgent

Component for managing node file systems and resources:

- **File Management**
  - File creation, modification, deletion
  - Directory structure management
  - File permission management

- **Configuration Deployment**
  - Configuration file deployment
  - Setting updates
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
  - User-defined metrics

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

Component providing UI and CLI interfaces for system settings management:

- **Settings Management**
  - YAML configuration loading and saving
  - Settings validation and application
  - Settings history management

- **UI Interface**
  - Hierarchical resource navigation
  - Property-based editor
  - Template library
  - Drag-and-drop resource management
  - Real-time validation feedback

- **CLI Interface**
  - Command-line resource management
  - Batch job and scripting support
  - Pipeline integration
  - Remote management support
  - JSON/YAML format support

- **Change Management**
  - Change validation and conflict detection
  - Atomic change application
  - Rollback functionality

- **User Management**
  - Permission-based access control
  - User authentication and authorization
  - Change audit logging

### 3.10 TIMPANI Scheduler

Component for managing time-deterministic task execution:

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

- **Safety-Critical Workload Management**
  - Safety-critical task isolation
  - Priority inheritance and ceiling
  - Deterministic behavior guarantee

### 3.11 PHAROS Networking

Component providing high-performance, low-latency networking:

- **eBPF-based Packet Processing**
  - XDP (eXpress Data Path) utilization
  - Fast packet processing
  - Kernel bypass optimization

- **Packet Filtering**
  - Precise packet filter rules
  - Protocol and port-based filtering
  - Stateful packet inspection

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

| Resource | Description | Key Properties |
|--------|------|-----------|
| Scenario | Service activation rules based on vehicle state conditions | • condition<br>• action<br>• target |
| Package | Group of Models deployed together | • pattern<br>• models<br>• resource connections |
| Model | Individual containerized service | • Container definition<br>• Resource requests/limits<br>• Scheduling configuration |
| Volume | Persistent data storage definition | • size<br>• accessModes<br>• reclaim policy |
| Network | Container network definition | • type<br>• QoS settings<br>• Packet filtering rules |

### 4.2 Scenario Resource

Defines rules to activate, update, or remove services based on vehicle state conditions.

**Functions:**
- Define vehicle state conditions (speed, ignition state, battery level, etc.)
- Specify actions to execute when conditions are met (launch, update, delete)
- Specify target Packages
- Manage priorities and dependencies
- Support nested and composite conditions

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

Defines groups of Models that are deployed together.

**Functions:**
- Reference multiple Models
- Define deployment patterns (plain, distributed, selector)
- Connect resources (volumes, networks)
- Manage node assignments
- Manage package metadata

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

Defines individual containerized services.

**Functions:**
- Define container image and configuration
- Set resource requests and limits
- Configure environment variables and volume mounts
- Set security contexts
- Define lifecycle hooks and probes
- Configure time-deterministic scheduling (TIMPANI)
- Access specialized manager resources

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

**Functions:**
- Define storage size and type
- Set access modes
- Specify storage class
- Set volume reclaim policy
- Configure backup and recovery

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

Defines networks for communication between containers and externally.

**Functions:**
- Set network type and bandwidth
- Configure security policies and QoS
- Configure fast paths based on eBPF (PHAROS)
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
    port: "{{EGRESS_PORT}}"       # Egress port (e.g., "5555")
    protocol: "{{PROTOCOL}}"      # Protocol (e.g., "udp", "tcp", "dds")
    topic: "{{TOPIC_NAME}}"       # Topic name (when using DDS protocol, e.g., "sensor_data")
  qos:
    latency: "{{LATENCY_LEVEL}}"  # Latency level (e.g., "low", "medium", "ultra-low")
    priority: "{{PRIORITY}}"      # Priority (e.g., "high", "critical", "normal")
  fastPath:
    enabled: {{FAST_PATH}}        # Fast path activation (true/false)
    optimizationLevel: "{{OPTIMIZATION_LEVEL}}" # Optimization level (e.g., "aggressive", "balanced")
  security:
    securityPolicy: "{{SECURITY_POLICY}}" # Security policy (e.g., "strict", "standard")
    encryption: {{ENCRYPTION}}    # Encryption activation (true/false)
```

## 5. Specialized Managers

PICCOLO provides six managers for managing specialized resources in the vehicle environment:

### 5.1 Specialized Manager Summary Table

| Manager | Main Purpose | Managed Resources |
|--------|----------|-----------|
| DeviceManager | Integrated management and access control of vehicle hardware devices | Sensors, cameras, actuators, input devices |
| AudioManager | Routing, mixing, priority management of vehicle audio resources | Speakers, microphones, audio streams |
| WindowManager | Display resource and user interface screen management | Displays, windows, input events |
| CloudManager | Vehicle-cloud hybrid workload management and optimization | Cloud services, data synchronization, network connections |
| NetworkManager (Pharos) | Vehicle network resource management and communication optimization (separate framework) | Container networks, packet filtering, cloud connections |
| StorageManager | Vehicle storage resource management and mount optimization | Container volumes, host storage, external storage |

### 5.1 DeviceManager Details

#### 5.1.1 Function Overview

DeviceManager provides integrated management of access, control, and monitoring of various hardware devices in vehicles.

**Key Functions:**
- Hardware device abstraction and integrated interface provision
- Device access permission management and conflict prevention
- Device state monitoring and anomaly detection
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

#### 5.2.1 Function Overview

AudioManager is responsible for routing, mixing, and priority management of audio resources in the vehicle.

**Key Functions:**
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

#### 5.3.1 Function Overview

WindowManager manages vehicle display resources and window layouts.

**Key Functions:**
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

NetworkManager functionality is provided as a specialized feature through the Pharos framework. Pharos is a specialized solution for vehicle network resource management and communication optimization that integrates with PICCOLO.

Please refer to the Pharos documentation for details.

### 5.5 CloudManager Details

#### 5.5.1 Function Overview

CloudManager manages and optimizes hybrid workloads between vehicle and cloud in the PICCOLO framework. This allows offloading resource-intensive tasks to the cloud or supports seamless transitions between vehicle and cloud.

**Key Functions:**
- Hybrid workload management
- Workload partitioning and distributed execution
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

#### 5.6.1 Function Overview

StorageManager is responsible for container storage and volume management functions within the PICCOLO framework.

**Key Functions:**
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
        action: ${alert_action}
    logging:
      level: ${log_level}
```

## 6. Operational Scenarios

PICCOLO supports various vehicle operational scenarios:

### 6.1 Driver Assistance System Activation

Automatically activates driver assistance systems when the vehicle is driving above a certain speed and the driver is present.

**Activation Conditions:**
- Vehicle speed > 30km/h
- Ignition state = ON
- Driver seat occupied = true

**Scenario Definition:**
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

### 6.2 Battery Conservation Mode

Optimizes system resource usage when the vehicle battery level is low.

**Activation Conditions:**
- Battery level < 20%
- Ignition state = ON

**Scenario Definition:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: battery-conservation
spec:
  condition:
    batteryLevel: "<20"
    ignitionState: "ON"
  action: update
  target: system-resources
  parameters:
    resourceProfile: "low-power"
```

### 6.3 Parking Assistance

Activates parking assistance features when the vehicle is in reverse gear and moving slowly.

**Activation Conditions:**
- Gear position = REVERSE
- Vehicle speed < 10km/h

**Scenario Definition:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: parking-assistance-activate
spec:
  condition:
    gearPosition: "REVERSE"
    vehicleSpeed: "<10"
  action: launch
  target: parking-assistance-package
```

### 6.4 Highway Driving Mode

Activates advanced driver assistance features when highway driving is detected.

**Activation Conditions:**
- Vehicle speed > 80km/h
- GPS location = highway
- Ignition state = ON

**Scenario Definition:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: highway-mode
spec:
  condition:
    vehicleSpeed: ">80"
    gpsLocation: "HIGHWAY"
    ignitionState: "ON"
  action: launch
  target: highway-assistance-package
```

### 6.5 Infotainment System Management

Adjusts infotainment system features based on vehicle state.

**Activation Conditions:**
- Vehicle in motion (speed > 0)
- Driver seat occupied

**Scenario Definition:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: infotainment-safety-mode
spec:
  condition:
    vehicleSpeed: ">0"
    driverSeatOccupied: "true"
  action: update
  target: infotainment-package
  parameters:
    safetyMode: "enabled"
    videoPlayback: "disabled"
```

## 7. User Interface

PICCOLO provides comprehensive user interfaces for configuration management and monitoring:

### 7.1 Web-based Management Console

A browser-based interface for system configuration and monitoring.

**Key Features:**
- Resource visualization and management
- System state dashboards
- Configuration editing with real-time validation
- Log and metric visualization
- User and permission management

### 7.2 Command-line Interface (CLI)

A powerful terminal-based interface for scripting and automation.

**Key Features:**
- Complete resource management
- Batch operations
- Scripting support
- Remote management
- Pipeline integration

### 7.3 REST API

A comprehensive REST API for programmatic access to all system functions.

**Key Features:**
- CRUD operations for all resources
- Authentication and authorization
- Filtering and pagination
- Versioning
- WebHooks for event notifications

### 7.4 Vehicle HMI Integration

Integration points for vehicle HMI to display system status and notifications.

**Key Features:**
- Status indicators
- Alerts and notifications
- Basic control functions
- Diagnostic information

## 8. Security and Safety

PICCOLO implements comprehensive security and safety measures:

### 8.1 Container Isolation

- Strong container isolation
- Resource usage limitations
- Controlled inter-container communication
- Security contexts and capabilities

### 8.2 Authentication and Authorization

- Role-based access control
- Token-based authentication
- API key management
- Auditing and logging

### 8.3 Safety Features

- ISO 26262 compliance considerations
- Deterministic execution for safety-critical functions
- Health monitoring and automatic recovery
- Redundancy and fallback mechanisms

### 8.4 Network Security

- Network isolation
- Traffic encryption
- Firewall and packet filtering
- Access control lists

### 8.5 Data Protection

- Encryption at rest
- Secure storage for sensitive data
- Secure communications
- Data integrity verification

## 9. Integration and Extensibility

PICCOLO is designed for integration with various vehicle systems and extensibility:

### 9.1 Vehicle Integration

- Support for standard vehicle communication protocols
- Integration with vehicle state and diagnostics
- Adapters for various vehicle platforms

### 9.2 Cloud Integration

- Hybrid execution between vehicle and cloud
- Telemetry and remote monitoring
- Over-the-air updates
- Fleet management integration

### 9.3 Extensibility

- Plugin architecture
- Custom resource types
- Specialized manager extensions
- API extensions

### 9.4 Third-party Integration

- Integration with third-party services
- SDK for extension development
- Open API specifications
- Standard protocols support

## 10. Performance and Scalability

PICCOLO is optimized for vehicle embedded systems:

### 10.1 Resource Efficiency

- Low memory footprint
- Minimal CPU overhead
- Optimized disk usage
- Power-aware operations

### 10.2 Scalability

- Support for multi-node deployments
- Distributed state management
- Load balancing
- Resource pooling

### 10.3 Performance Optimization

- Fast container startup
- Optimized communication paths
- Efficient state management
- Cache-friendly design

### 10.4 Real-time Performance

- Deterministic execution for safety-critical functions
- Predictable latency
- Priority-based scheduling
- Resource isolation

## 11. Implementation Constraints

The following constraints must be considered in PICCOLO implementation:

### 11.1 Hardware Constraints

- Support for various SoCs (Snapdragon, NVIDIA, etc.)
- Minimum hardware requirements:
  - CPU: 4 cores, 1.5GHz+
  - RAM: 2GB+
  - Storage: 8GB+

### 11.2 Software Constraints

- Support for multiple operating systems (QNX, Linux, AGL, Android Automotive)
- Kernel requirements:
  - Linux: Kernel 5.10+
  - Features: cgroups, namespaces, seccomp
- Container runtime requirements:
  - Podman 3.0+
  - Rootless mode support

### 11.3 Network Constraints

- Support for various network topologies
- Bandwidth efficiency
- Operation in partially connected environments
- Support for network quality variability

### 11.4 Safety and Security Constraints

- Compliance with automotive safety standards
- Security certifications
- Privacy regulations compliance
- Secure boot and trusted execution

## 12. Conclusion

The PICCOLO framework provides a comprehensive platform for managing containerized applications in vehicle environments. By leveraging Podman for container management and providing a Kubernetes-like declarative approach, PICCOLO offers a familiar yet specialized solution for automotive software deployment.

Key strengths of PICCOLO include:

1. Vehicle-optimized container management
2. Scenario-based service activation
3. Specialized resource managers for automotive use cases
4. Deterministic execution for safety-critical applications
5. Integration with vehicle systems and cloud services

The framework's modular architecture allows for flexible deployment across diverse vehicle platforms while maintaining a consistent application model. This ensures that vehicle software can be developed once and deployed across multiple vehicle lines and generations.

## 13. Appendix: Architecture Diagrams

### 13.1 Overall System Architecture

```
+------------------------------------------+
|               PICCOLO System             |
+------------------------------------------+
|                                          |
|  +---------------+    +---------------+  |
|  | API Server    |<-->| Settings      |  |
|  |               |    | Service       |  |
|  +---------------+    +---------------+  |
|          ^                   ^           |
|          |                   |           |
|  +---------------+    +---------------+  |
|  | Filter        |    | State         |  |
|  | Gateway       |<-->| Manager       |  |
|  +---------------+    +---------------+  |
|          ^                   ^           |
|          |                   |           |
|  +---------------+    +---------------+  |
|  | Action        |<-->| Policy        |  |
|  | Controller    |    | Manager       |  |
|  +---------------+    +---------------+  |
|          ^                   ^           |
|          |                   |           |
|  +---------------+    +---------------+  |
|  | Node          |<-->| Monitoring    |  |
|  | Agent         |    | Server        |  |
|  +---------------+    +---------------+  |
|                                          |
+------------------------------------------+
             |                |
             v                v
+------------------------+  +---------------------+
|  Specialized Managers  |  | External Systems    |
+------------------------+  +---------------------+
| • DeviceManager        |  | • Vehicle Systems   |
| • AudioManager         |  | • Cloud Services    |
| • WindowManager        |  | • Edge Computing    |
| • CloudManager         |  | • Developer Tools   |
| • NetworkManager       |  +---------------------+
| • StorageManager       |
+------------------------+
```

### 13.2 Component Interaction Diagram

```
+-------------+      +------------+      +-------------+
| External    |      | APIServer  |      | Settings    |
| Systems     |<---->|            |<---->| Service     |
+-------------+      +------------+      +-------------+
                           ^
                           |
                           v
+--------------+      +------------+      +------------+
| Filter       |<---->| State      |<---->| Node       |
| Gateway      |      | Manager    |      | Agent      |
+--------------+      +------------+      +------------+
      ^                     ^                   ^
      |                     |                   |
      v                     v                   v
+--------------+      +------------+      +------------+
| Action       |<---->| Policy     |<---->| Monitoring |
| Controller   |      | Manager    |      | Server     |
+--------------+      +------------+      +------------+
      ^
      |
      v
+--------------+
| Container    |
| Runtime      |
+--------------+
```

### 13.3 Resource Hierarchy

```
+------------------+
|     Scenario     |
+------------------+
        |
        v
+------------------+
|     Package      |
+------------------+
        |
        v
+------------------+
|      Model       |
+------------------+
        |
        v
+------------------+     +------------------+
|   Container      |<--->|    Resources     |
+------------------+     +------------------+
                              |        |
                              v        v
                        +--------+ +--------+
                        | Volume | | Network|
                        +--------+ +--------+
```
