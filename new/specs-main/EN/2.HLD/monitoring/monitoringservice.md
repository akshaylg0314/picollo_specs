# PICCOLO Resource Monitoring Service Design

## 1. Overview

The PICCOLO Resource Monitoring Service is designed to track and manage the status, performance, and quality of PICCOLO resources such as Scenario, Package, and Model in real time. This document defines the high-level design (HLD) of the resource monitoring service.

### 1.1 Purpose

- Real-time monitoring of PICCOLO resource status and performance
- Tracking and management of resource dependencies
- Anomaly detection and alerting
- Automatic recovery mechanisms in case of issues
- Data collection for resource usage and performance analysis

## 2. Key Metrics by Resource

### 2.1 Scenario

| Metric | Description | Unit |
|--------|-------------|------|
| Active State | Current activation state of the scenario | Active/Inactive |
| Execution Count | Total number of scenario executions | Count |
| Average Execution Time | Average time taken for scenario execution | ms |
| Failure Rate | Scenario execution failure rate | % |
| Dependency Status | Status of resources the scenario depends on | OK/Warning/Error |

### 2.2 Package

| Metric | Description | Unit |
|--------|-------------|------|
| Deployment Status | Deployment status of the package | Deployed/Undeployed/Failed |
| Version | Currently deployed package version | Version string |
| Usage Count | Number of times the package has been used | Count |
| Memory Usage | Memory used by the package | MB |
| Disk Usage | Disk space occupied by the package | MB |
| Load Time | Time taken to load the package | ms |

### 2.3 Model

| Metric | Description | Unit |
|--------|-------------|------|
| Execution State | Current execution state of the model (container) | Running/Pending/Failed/Unknown |
| Uptime | Total time the model has been running | h/m/s |
| Restart Count | Number of times the model has restarted | Count |
| CPU Usage | CPU usage of the model | % |
| Memory Usage | Memory used by the model | MB |
| Network Traffic | Amount of network traffic for the model | MB/s |
| Response Time | Average request response time for the model | ms |

## 3. Monitoring Strategy

### 3.1 State-based Monitoring

Each PICCOLO resource follows a defined state machine for state transitions. The monitoring service tracks and records these state changes:

```yaml
resource_state_monitoring:
  enabled: true
  tracking:
    state_transitions: true
    time_in_state: true
  alerting:
    unexpected_transitions: true
    stuck_states:
      enabled: true
      thresholds:
        loading: 30s
        initializing: 60s
```

### 3.2 Performance-based Monitoring

Collects and analyzes resource performance metrics to detect degradation or issues:

```yaml
performance_monitoring:
  collection_interval: 10s
  metrics:
    - name: execution_time
      threshold_warning: 500ms
      threshold_critical: 1000ms
    - name: memory_usage
      threshold_warning: 80%
      threshold_critical: 95%
    - name: failure_rate
      threshold_warning: 5%
      threshold_critical: 10%
```

### 3.3 Dependency Monitoring

Tracks dependencies between resources and analyzes the impact of state changes in related resources:

```yaml
dependency_monitoring:
  enabled: true
  dependency_types:
    - parent_child
    - uses
    - requires
  propagation:
    status_propagation: true
    impact_analysis: true
```

## 4. Implementation Approach

### 4.1 Resource Monitoring Configuration

Monitoring configurations for each resource type are defined in YAML format:

```yaml
resource_monitoring:
  scenario:
    enabled: true
    metrics:
      - active_state
      - execution_count
      - average_execution_time
      - failure_rate
    collection_interval: 5s
    
  package:
    enabled: true
    metrics:
      - deployment_status
      - version
      - usage_count
      - memory_usage
    collection_interval: 30s
    
  model:
    enabled: true
    metrics:
      - load_status
      - inference_time
      - throughput
      - accuracy
      - memory_usage
      - gpu_usage
    collection_interval: 10s
```

### 4.2 State Transition Tracking

Configuration for tracking and recording resource state changes:

```yaml
state_transition_tracking:
  enabled: true
  resources:
    - type: scenario
      states:
        - inactive
        - active
        - failed
    - type: package
      states:
        - undeployed
        - deploying
        - deployed
        - failed
    - type: model
      states:
        - unloaded
        - loading
        - loaded
        - error
  tracking:
    transition_history: true
    transition_duration: true
```

### 4.3 Dependency Monitoring

Configuration for defining and tracking dependencies between resources:

```yaml
dependency_monitoring:
  dependency_map:
    - resource: scenario_1
      depends_on:
        - type: package
          id: package_a
        - type: model
          id: model_x
    - resource: package_a
      depends_on:
        - type: model
          id: model_y
  impact_analysis:
    enabled: true
    propagation_rules:
      - from_state: failed
        impact: critical
      - from_state: degraded
        impact: warning
```

### 4.4 Anomaly Detection

Configuration for detecting abnormal behavior that deviates from normal operation patterns:

```yaml
anomaly_detection:
  enabled: true
  methods:
    - type: statistical
      metrics:
        - execution_time
        - memory_usage
      sensitivity: medium
    - type: pattern_based
      metrics:
        - state_transitions
        - error_counts
      window_size: 1h
  baseline:
    auto_learn: true
    learning_period: 7d
```

## 5. Data Storage

### 5.1 ETCD-based Data Storage

Monitoring data is stored in ETCD, leveraging the characteristics of key-value store for efficient management:

```yaml
data_storage:
  type: etcd
  configuration:
    endpoints:
      - "etcd-server-1:2379"
      - "etcd-server-2:2379"
      - "etcd-server-3:2379"
    tls:
      enabled: true
      cert_file: "/etc/piccolo/certs/etcd-client.crt"
      key_file: "/etc/piccolo/certs/etcd-client.key"
      ca_file: "/etc/piccolo/certs/etcd-ca.crt"
  data_organization:
    prefix: "/piccolo/monitoring"
    key_structure:
      resource: "{resource_type}/{resource_id}"
      metric: "{resource_type}/{resource_id}/metrics/{metric_name}"
      state: "{resource_type}/{resource_id}/state"
      event: "{resource_type}/{resource_id}/events/{timestamp}"
  retention:
    metrics: 30d
    events: 90d
    state_transitions: 60d
  compaction:
    interval: 24h
    policy: "revision-based"
```

### 5.2 Data Access and Querying

Methods for accessing monitoring data stored in ETCD:

```yaml
data_access:
  apis:
    - name: "GetResourceMetrics"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/metrics"
      parameters:
        - name: "metric_names"
          type: "array"
          required: false
        - name: "time_range"
          type: "object"
          required: false
      response_format: "json"
      
    - name: "GetResourceState"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/state"
      parameters:
        - name: "include_history"
          type: "boolean"
          required: false
      response_format: "json"
      
    - name: "GetResourceEvents"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/events"
      parameters:
        - name: "event_types"
          type: "array"
          required: false
        - name: "time_range"
          type: "object"
          required: false
        - name: "limit"
          type: "integer"
          required: false
      response_format: "json"
  
  query_capabilities:
    - prefix_search
    - range_queries
    - transaction_support
```

## 6. Scalability and Integration

### 6.1 Scalability

Scalability considerations for supporting large-scale PICCOLO resource environments:

- Distributed monitoring architecture
- Data sampling and aggregation strategies
- Hierarchical monitoring structure

### 6.2 External System Integration

Integration with existing monitoring and management systems:

- Kubernetes integration
- Logging system connection
- Standard metrics exporter provision

## 7. Resource Monitoring Templates

Below are YAML templates that include monitoring metric information for Scenario and Package resources. Each template includes both the user-defined part (spec) and the system-monitored and managed part (status).

### 7.1 Scenario Resource Template

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: "{{SCENARIO_NAME}}"               # Scenario name (e.g., "adas-activation")
  annotations:
    description: "{{DESCRIPTION}}"        # Scenario description (e.g., "Activates ADAS features during driving")
    priority: "{{PRIORITY}}"              # Priority (e.g., "high", "medium", "low")
  labels:
    function: "{{FUNCTION}}"              # Function (e.g., "driver-assistance")
    category: "{{CATEGORY}}"              # Category (e.g., "safety", "comfort")
    asilLevel: "{{ASIL_LEVEL}}"           # ASIL level (e.g., "A", "B", "C", "D", "QM")
spec:
  condition:
    {{CONDITION_KEY}}: "{{CONDITION_VALUE}}"   # Condition key-value (e.g., vehicleSpeed: ">30")
    # Additional conditions...
  action: "{{ACTION_TYPE}}"               # Action type (launch, update, modify, notify, rollback, pause, resume)
  target:
    packages:
      - "{{PACKAGE_NAME_1}}"              # Target package name (e.g., "adas-software")
      - "{{PACKAGE_NAME_2}}"              # Additional package...
      # Additional packages...
# System-tracked/managed part
status:
  # Basic status information
  currentState: "{{CURRENT_STATE}}"           # Current state (idle/waiting/playing/error/allowed/denied)
  policyStatus: "{{POLICY_STATUS}}"           # Policy status (allowed/denied)
  lastTransitionTime: "{{LAST_TRANSITION_TIME}}"  # Last state transition time
  message: "{{STATUS_MESSAGE}}"               # Status message
  
  # Scenario condition and execution status
  conditions:
    - type: "{{CONDITION_TYPE}}"              # Condition type (e.g., "ConditionsMet")
      status: "{{CONDITION_STATUS}}"          # Condition status (True/False)
      lastUpdateTime: "{{LAST_UPDATE_TIME}}"  # Last update time
    # Additional condition statuses...
  
  # Target package status
  activePackages:
    - name: "{{PACKAGE_NAME_1}}"              # Package name
      status: "{{PACKAGE_STATUS}}"            # Package status (running, error, etc.)
    # Additional package statuses...
  
  # Monitoring metrics
  metrics:
    # Active state metrics
    activeState: "{{ACTIVE_STATE}}"           # Active/Inactive
    
    # Execution-related metrics
    executionCount: {{EXECUTION_COUNT}}       # Execution count
    averageExecutionTime: {{AVG_EXEC_TIME}}   # Average execution time (ms)
    lastExecutionTime: {{LAST_EXEC_TIME}}     # Last execution time (ms)
    failureRate: {{FAILURE_RATE}}             # Failure rate (%)
    lastExecutionResult: "{{EXEC_RESULT}}"    # Last execution result (Success/Failure)
    
    # Dependency status metrics
    dependencyStatus: "{{DEPENDENCY_STATUS}}" # Dependency status (OK/Warning/Error)
    dependencyDetails:
      - resourceType: "{{RESOURCE_TYPE}}"     # Resource type (Package, Model, etc.)
        resourceName: "{{RESOURCE_NAME}}"     # Resource name
        status: "{{RESOURCE_STATUS}}"         # Resource status (OK/Warning/Error)
      # Additional dependency statuses...
    
    # Performance metrics
    responseTime: {{RESPONSE_TIME}}           # Response time (ms)
    resourceUtilization: {{RESOURCE_UTIL}}    # Resource utilization (%)
    
    # Event metrics
    totalEvents: {{TOTAL_EVENTS}}             # Total event count
    lastEventTime: "{{LAST_EVENT_TIME}}"      # Last event time
    recentEvents:
      - type: "{{EVENT_TYPE}}"                # Event type
        time: "{{EVENT_TIME}}"                # Event time
        details: "{{EVENT_DETAILS}}"          # Event details
      # Additional events...
    
    # ASIL-related metrics (for safety-critical scenarios)
    safetyMetrics:
      responseTimeConstraint: {{RT_CONSTRAINT}}  # Response time constraint (ms)
      responseTimeActual: {{RT_ACTUAL}}          # Actual response time (ms)
      constraintsMet: {{CONSTRAINTS_MET}}        # Constraints met (true/false)
```

### 7.2 Package Resource Template

```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: "{{PACKAGE_NAME}}"               # Package name (e.g., "vision-processing")
  annotations:
    version: "{{VERSION}}"               # Version (e.g., "2.3")
    description: "{{DESCRIPTION}}"       # Description (e.g., "Camera and video processing system")
  labels:
    types: ["{{TYPE_1}}", "{{TYPE_2}}"]  # Package types (e.g., ["resource-bounded", "time-critical"])
    component: "{{COMPONENT}}"           # Component (e.g., "vision")
    asilLevel: "{{ASIL_LEVEL}}"          # ASIL level (e.g., "B")
spec:
  pattern:
    type: "{{PATTERN_TYPE}}"             # Pattern type (e.g., "selector", "plain", "distributed")
    nodeSelector:
      "{{SELECTOR_KEY}}": "{{SELECTOR_VALUE}}"  # Node selector (e.g., gpu: "available")
  packageType: "{{PACKAGE_TYPE}}"        # Package type (e.g., "StandardPackage", "CustomPackage", "SystemPackage")
  typeProperties:
    category: "{{TYPE_CATEGORY}}"        # Type category (e.g., "application", "system", "infrastructure")
    criticality: "{{CRITICALITY}}"       # Criticality (e.g., "high", "medium", "low")
    lifecycle: "{{LIFECYCLE}}"           # Lifecycle (e.g., "development", "testing", "production")
  models:
    - name: "{{MODEL_NAME_1}}"           # Model name (e.g., "vision-system")
      image: "{{IMAGE_PATH}}"            # Image path (e.g., "vehicle/vision-system:2.3")
      node: "{{NODE_NAME}}"              # Node name (e.g., "VisionHPC")
# System-tracked/managed part
status:
  # Basic status information
  currentState: "{{CURRENT_STATE}}"      # Current state (initializing/running/degraded/error)
  lastTransitionTime: "{{LAST_TRANSITION_TIME}}"  # Last state transition time
  message: "{{STATUS_MESSAGE}}"          # Status message
  
  # Model status information
  models:
    - name: "{{MODEL_NAME_1}}"           # Model name
      status: "{{MODEL_STATUS}}"         # Model status (running, error, etc.)
      restarts: {{RESTART_COUNT}}        # Restart count
      readiness: {{READINESS}}           # Readiness (true/false)
    # Additional model statuses...
  
  # Pattern and type monitoring status
  patternStatus:
    type: "{{PATTERN_TYPE}}"             # Pattern type (selector, plain, distributed)
    health: "{{PATTERN_HEALTH}}"         # Pattern health (healthy, degraded, failed)
    nodesMatched: {{NODES_MATCHED}}      # Number of matched nodes
    nodeList:
      - name: "{{NODE_NAME_1}}"          # Node name
        status: "{{NODE_STATUS}}"        # Node status
      # Additional nodes...
    lastVerified: "{{LAST_VERIFIED}}"    # Last verification time
  
  packageTypeStatus:
    type: "{{PACKAGE_TYPE}}"             # Package type
    complianceStatus: "{{COMPLIANCE}}"   # Compliance status (compliant, non-compliant)
    typeConstraints:
      - name: "{{CONSTRAINT_NAME}}"      # Constraint name
        status: "{{CONSTRAINT_STATUS}}"  # Constraint status (met, not-met)
      # Additional constraints...
    propertiesStatus:
      lifecycle: "{{LIFECYCLE_STATUS}}"  # Lifecycle status
      criticality: "{{CRITICALITY_LEVEL}}" # Criticality level
  
  # Monitoring metrics
  metrics:
    # Deployment status metrics
    deploymentStatus: "{{DEPLOYMENT_STATUS}}"  # Deployment status (Deployed/Undeployed/Failed)
    deploymentTime: "{{DEPLOYMENT_TIME}}"      # Deployment time
    
    # Version information
    version: "{{VERSION}}"                     # Version
    lastUpdateTime: "{{LAST_UPDATE_TIME}}"     # Last update time
    
    # Package type monitoring metrics
    packageType:
      type: "{{PACKAGE_TYPE}}"                 # Package type
      count: {{COUNT}}                         # Count of same type packages
      operationalStatus: "{{OP_STATUS}}"       # Operational status
      typicalResponseTime: {{RESPONSE_TIME}}   # Typical response time
      healthScore: "{{HEALTH_SCORE}}"          # Health score (0-100%)
      typeSpecificMetrics:
        - name: "{{METRIC_NAME}}"              # Type-specific metric name
          value: "{{METRIC_VALUE}}"            # Type-specific metric value
          threshold: "{{METRIC_THRESHOLD}}"    # Type-specific metric threshold
    
    # Pattern monitoring metrics
    patternMetrics:
      type: "{{PATTERN_TYPE}}"                 # Pattern type
      effectiveDeploymentRate: {{DEPLOY_RATE}} # Effective deployment rate (%)
      patternComplianceScore: {{COMPLIANCE}}   # Pattern compliance score (0-100%)
      nodeDistribution:                        # Node distribution
        matched: {{MATCHED_NODES}}             # Number of matched nodes
        healthy: {{HEALTHY_NODES}}             # Number of healthy nodes
        degraded: {{DEGRADED_NODES}}           # Number of degraded nodes
        failed: {{FAILED_NODES}}               # Number of failed nodes
      balanceScore: {{BALANCE_SCORE}}          # Load balance score (0-100%)
    
    # Usage metrics
    usageCount: {{USAGE_COUNT}}                # Usage count
    activeScenarios: {{ACTIVE_SCENARIOS}}      # Number of active scenarios
    uptime: "{{UPTIME}}"                       # Uptime
    
    # Model relationship metrics
    modelRelationships:
      total: {{TOTAL_MODELS}}                  # Total number of models
      healthy: {{HEALTHY_MODELS}}              # Number of healthy models
      unhealthy: {{UNHEALTHY_MODELS}}          # Number of unhealthy models
      dependencies:
        - source: "{{SOURCE_MODEL}}"           # Source model
          target: "{{TARGET_MODEL}}"           # Target model
          status: "{{DEPENDENCY_STATUS}}"      # Dependency status
        # Additional dependencies...
```

### 7.3 Model Resource Template

```yaml
apiVersion: piccolo.io/v1
kind: Model
metadata:
  name: "{{MODEL_NAME}}"                  # Model name (e.g., "vision-system")
  annotations:
    version: "{{VERSION}}"                # Version (e.g., "1.2")
    description: "{{DESCRIPTION}}"        # Description (e.g., "Camera vision processing system")
  labels:
    component: "{{COMPONENT}}"            # Component (e.g., "vision")
    asilLevel: "{{ASIL_LEVEL}}"           # ASIL level (e.g., "B", "D")
spec:
  image: "{{IMAGE_PATH}}"                 # Image path (e.g., "vehicle/vision-system:1.2")
  node: "{{NODE_NAME}}"                   # Node name (e.g., "VisionHPC")
  hostNetwork: {{HOST_NETWORK}}           # Use host network (true/false)
  containers:
    - name: "{{CONTAINER_NAME}}"          # Container name
      image: "{{CONTAINER_IMAGE}}"        # Container image
      resources:
        limits:
          cpu: {{CPU_LIMIT}}              # CPU limit (e.g., 1)
          memory: "{{MEMORY_LIMIT}}"      # Memory limit (e.g., "1Gi")
          gpu: {{GPU_LIMIT}}              # GPU limit (e.g., 1)
  scheduling:
    timeCritical: {{TIME_CRITICAL}}       # Time critical (true/false)
    priority: {{PRIORITY}}                # Priority (e.g., 95)
    period: "{{PERIOD}}"                  # Period (e.g., "10ms")
    deadline: "{{DEADLINE}}"              # Deadline (e.g., "8ms")
    executionTime: "{{EXECUTION_TIME}}"   # Execution time (e.g., "2ms")
    cpuAffinity: [{{CPU_AFFINITY}}]       # CPU affinity (e.g., [2, 3])
    isolationLevel: "{{ISOLATION_LEVEL}}" # Isolation level (e.g., "exclusive")
    criticality: "{{CRITICALITY}}"        # Criticality (e.g., "safety-critical")
  deviceAccess:
    - manager: "{{DEVICE_MANAGER}}"       # Device manager (e.g., "vehicle-sensors")
      devices: ["{{DEVICE_1}}", "{{DEVICE_2}}"] # Device list (e.g., ["camera", "lidar"])
  volumes:
    - name: "{{VOLUME_NAME}}"             # Volume name
      mountPath: "{{MOUNT_PATH}}"         # Mount path (e.g., "/data")
      volume: "{{VOLUME_REF}}"            # Volume reference (e.g., "vision-data-volume")
  networks:
    - name: "{{NETWORK_NAME}}"            # Network name (e.g., "sensor-network")
# System-tracked/managed part
status:
  # Basic status information
  phase: "{{PHASE}}"                      # Phase (Running/Pending/Succeeded/Failed/Unknown)
  conditions:
    - type: "{{CONDITION_TYPE}}"          # Condition type (e.g., "Ready")
      status: "{{CONDITION_STATUS}}"      # Condition status (True/False)
      lastProbeTime: "{{LAST_PROBE_TIME}}" # Last probe time
  startTime: "{{START_TIME}}"             # Start time
  hostIP: "{{HOST_IP}}"                   # Host IP
  podIP: "{{POD_IP}}"                     # Pod IP
  
  # Container status information
  containerStatuses:
    - name: "{{CONTAINER_NAME}}"          # Container name
      state:
        running:
          startedAt: "{{STARTED_AT}}"     # Start time
      ready: {{READY}}                    # Ready status (true/false)
      restartCount: {{RESTART_COUNT}}     # Restart count
      image: "{{IMAGE}}"                  # Image
      imageID: "{{IMAGE_ID}}"             # Image ID
  
  # Monitoring metrics
  metrics:
    # Load status metrics
    loadStatus: "{{LOAD_STATUS}}"         # Load status (Loaded/Unloaded/Error)
    loadTime: {{LOAD_TIME}}               # Load time (ms)
    
    # Performance metrics
    inferenceTime: {{INFERENCE_TIME}}     # Inference time (ms)
    throughput: {{THROUGHPUT}}            # Throughput (requests/second)
    latency: {{LATENCY}}                  # Latency (ms)
    accuracy: {{ACCURACY}}                # Accuracy (%)
    
    # Resource usage metrics
    memoryUsage: {{MEMORY_USAGE}}         # Memory usage (MB)
    memoryUsagePercent: {{MEMORY_PCT}}    # Memory usage percentage (%)
    cpuUsage: {{CPU_USAGE}}               # CPU usage (%)
    gpuUsage: {{GPU_USAGE}}               # GPU usage (%)
    gpuMemoryUsage: {{GPU_MEMORY}}        # GPU memory usage (MB)
    
    # Status metrics
    uptime: "{{UPTIME}}"                  # Uptime
    healthStatus: "{{HEALTH_STATUS}}"     # Health status (Healthy/Unhealthy)
    lastError: "{{LAST_ERROR}}"           # Last error
    errorCount: {{ERROR_COUNT}}           # Error count
    
    # Event metrics
    events:
      - type: "{{EVENT_TYPE}}"            # Event type
        time: "{{EVENT_TIME}}"            # Event time
        message: "{{EVENT_MESSAGE}}"      # Event message
        severity: "{{EVENT_SEVERITY}}"    # Event severity
    
    # ASIL-related metrics (for safety-critical models)
    safetyMetrics:
      asilLevel: "{{ASIL_LEVEL}}"            # ASIL level (A/B/C/D)
      deadlineMissCount: {{DEADLINE_MISSES}} # Deadline miss count
      worstCaseExecutionTime: {{WCET}}       # Worst-case execution time (ms)
      averageExecutionTime: {{AVG_EXEC_TIME}} # Average execution time (ms)
      executionTimeVariance: {{EXEC_VARIANCE}} # Execution time variance
      schedulingLatency: {{SCHED_LATENCY}}   # Scheduling latency (ms)
      safetyMonitorStatus: "{{SAFETY_STATUS}}" # Safety monitor status (Active/Inactive)
      safetyChecks:
        - name: "{{CHECK_NAME}}"            # Check name (e.g., "Timing Constraint")
          status: "{{CHECK_STATUS}}"        # Check status (Pass/Fail)
          lastCheckTime: "{{CHECK_TIME}}"   # Last check time
      functionalSafetyMetrics:
        errorDetectionCoverage: {{ERROR_DETECTION}} # Error detection coverage (%)
        errorRecoveryCoverage: {{ERROR_RECOVERY}}   # Error recovery coverage (%)
        diagnosticsCoverage: {{DIAGNOSTICS}}        # Diagnostics coverage (%)
      runtimeVerification:
        enabled: {{RUNTIME_VERIFY}}              # Runtime verification enabled (true/false)
        constraintsChecked: {{CONSTRAINTS_CHECKED}} # Number of constraints checked
        constraintViolations: {{CONSTRAINT_VIOLATIONS}} # Number of constraint violations
```

### 7.4 Usage Examples

Below are examples of Scenario and Package resources in YAML:

```yaml
# Scenario Example
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: "adas-activation"
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
  action: "launch"
  target:
    packages:
      - "adas-software"
      - "vision-processing"
status:
  currentState: "playing"
  policyStatus: "allowed"
  lastTransitionTime: "2025-08-07T14:25:32Z"
  message: "ADAS services successfully activated"
  metrics:
    activeState: "Active"
    executionCount: 127
    averageExecutionTime: 235
    failureRate: 0.5
    dependencyStatus: "OK"
```

```yaml
# Package Example
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: "vision-processing"
  annotations:
    version: "2.3"
    description: "Camera and video processing system"
  labels:
    types: ["resource-bounded", "time-critical"]
    component: "vision"
    asilLevel: "B"
spec:
  pattern:
    type: "selector"
    nodeSelector:
      gpu: "available"
  models:
    - name: "vision-system"
      image: "vehicle/vision-system:2.3"
      node: "VisionHPC"
status:
  currentState: "running"
  lastTransitionTime: "2025-08-07T14:25:35Z"
  message: "All models running normally"
  metrics:
    deploymentStatus: "Deployed"
    version: "2.3"
    memoryUsage: 5.6
    cpuUsage: 65
    loadTime: 4250
    healthStatus: "Healthy"
```

## 8. Conclusion

The PICCOLO resource monitoring service comprehensively tracks and manages the status and performance of all resources in the system. Through ETCD-based data storage, it provides a stable and scalable monitoring infrastructure that helps detect problems early and improves system stability and reliability. Additionally, the detailed metric templates provided in this document enable effective implementation of monitoring for various resource types.
