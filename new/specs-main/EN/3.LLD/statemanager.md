# State Manager

## 1. Introduction

### Major features

State Manager is a core component in the PICCOLO framework responsible for managing the state of systems and services.

1. Performs reconciliation tasks based on state differences (diffs) received from NodeAgent.
2. Implements a state machine that manages the complete lifecycle of services. (Pending, Initializing, Running, Degraded, Terminating, Failed)
3. Resolves state inconsistencies and performs automatic recovery in failure scenarios.
4. Stores and shares state information through ETCD with other components.
5. Tracks and processes state change events to maintain system stability.
6. Integrates with ActionController to execute necessary actions.

### Main Dataflow

1. Receives state difference (State Diff) information from NodeAgent.
2. Develops action plans based on received state differences.
3. Executes action plans through ActionController.
4. Updates states and reports results based on action outcomes.
5. Records state changes and events in ETCD.
6. Provides current state information to NodeAgent and other components.

## 2. File information

```text
statemanager
├── statemanager.md
├── Cargo.toml
└── src
    ├── etcd
    │   ├── client.rs
    │   ├── mod.rs
    │   └── watcher.rs
    ├── grpc
    │   ├── client
    │   │   ├── action_controller.rs
    │   │   ├── mod.rs
    │   │   └── node_agent.rs
    │   ├── mod.rs
    │   └── server
    │       ├── mod.rs
    │       └── service.rs
    ├── main.rs
    ├── manager.rs
    ├── reconciliation
    │   ├── mod.rs
    │   ├── planner.rs
    │   └── executor.rs
    ├── recovery
    │   ├── action.rs
    │   ├── mod.rs
    │   └── strategy.rs
    └── state
        ├── machine.rs
        ├── model.rs
        ├── mod.rs
        ├── events.rs
        └── tracker.rs
```

- **main.rs** - StateManager initialization and execution
- **manager.rs** - Control of data flow between modules
- **etcd/mod.rs** - etcd integration management
- **etcd/client.rs** - Common etcd library usage
- **etcd/watcher.rs** - Detection and processing of etcd key changes
- **grpc/mod.rs** - gRPC communication management
- **grpc/client/mod.rs** - gRPC client management
- **grpc/client/action_controller.rs** - gRPC communication with ActionController
- **grpc/client/node_agent.rs** - gRPC communication with NodeAgent and receipt of state differences (diffs)
- **grpc/server/mod.rs** - gRPC server management
- **grpc/server/service.rs** - StateManager gRPC service implementation
- **reconciliation/mod.rs** - State reconciliation functionality management
- **reconciliation/planner.rs** - Developing action plans based on differences received from NodeAgent
- **reconciliation/executor.rs** - Execution of action plans
- **recovery/mod.rs** - Failure recovery management
- **recovery/action.rs** - Recovery action implementation
- **recovery/strategy.rs** - Recovery strategy definition by state type
- **state/mod.rs** - State management
- **state/machine.rs** - State machine implementation
- **state/model.rs** - State model definition
- **state/events.rs** - State change event management
- **state/tracker.rs** - State change tracking

## 3. Function information

The function information in this section is written in the format used by rustdoc comments.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo State Manager
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize and start the StateManager
pub async fn initialize() {}

/// Start reconciliation loop
pub async fn start_reconciliation() {}

/// Stop all reconciliation tasks
pub async fn stop_reconciliation() {}

/// Handle state diff from NodeAgent
///
/// ### Parameters
/// * `diff: StateDiff` - State difference information from NodeAgent
/// ### Description
/// Process state differences and initiate reconciliation
pub async fn handle_state_diff(diff: StateDiff) -> common::Result<()> {}

/// Execute reconciliation based on state diff
///
/// ### Parameters
/// * `diff: &StateDiff` - State difference to reconcile
/// ### Returns
/// * `ReconciliationResult` - Result of the reconciliation process
pub async fn execute_reconciliation(diff: &StateDiff) -> ReconciliationResult {}
```

### `grpc/client/node_agent.rs`

```rust
/// Connect to NodeAgent gRPC service
///
/// ### Parameters
/// * `address: &str` - NodeAgent service address
/// ### Returns
/// * `NodeAgentClient` - Connected client
pub async fn connect(address: &str) -> NodeAgentClient {}

/// State difference structure received from NodeAgent
pub struct StateDiff {
    pub resource_id: ResourceId,
    pub existence_diff: Option<ExistenceDiff>,
    pub scale_diff: Option<ScaleDiff>,
    pub version_diff: Option<VersionDiff>,
    pub config_diff: Option<ConfigDiff>,
    pub status_diff: Option<StatusDiff>,
}

/// Subscribe to state differences from NodeAgent
///
/// ### Parameters
/// * `callback: StateDiffCallback` - Callback to handle received diffs
/// ### Returns
/// * `SubscriptionId` - ID of the subscription
pub async fn subscribe_to_state_diffs(&self, callback: StateDiffCallback) -> SubscriptionId {}

/// Check if difference requires immediate action
///
/// ### Parameters
/// * `diff: &StateDiff` - Received state difference
/// ### Returns
/// * `bool` - True if requires immediate action
pub fn is_critical_diff(diff: &StateDiff) -> bool {}

/// Get priority of the difference
///
/// ### Parameters
/// * `diff: &StateDiff` - Received state difference
/// ### Returns
/// * `DiffPriority` - Priority level of the difference
pub fn get_diff_priority(diff: &StateDiff) -> DiffPriority {}
```

### `state/machine.rs`

```rust
/// Service state machine structure
pub struct StateMachine {
    pub current_state: ServiceState,
    pub transitions: Vec<StateTransition>,
    pub last_updated: DateTime<Utc>,
}

/// Service state enumeration
pub enum ServiceState {
    Pending,
    Initializing,
    Running,
    Degraded,
    Terminating,
    Failed,
}

/// Create new state machine for a service
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// ### Returns
/// * `StateMachine` - New state machine instance
pub fn new_state_machine(service_id: &str) -> StateMachine {}

/// Attempt state transition
///
/// ### Parameters
/// * `machine: &mut StateMachine` - State machine to update
/// * `event: StateEvent` - Event triggering the transition
/// ### Returns
/// * `TransitionResult` - Result of the transition attempt
pub fn try_transition(machine: &mut StateMachine, event: StateEvent) -> TransitionResult {}

/// Get allowed actions for current state
///
/// ### Parameters
/// * `state: ServiceState` - Current service state
/// ### Returns
/// * `Vec<AllowedAction>` - List of allowed actions
pub fn get_allowed_actions(state: ServiceState) -> Vec<AllowedAction> {}
```

### `etcd/client.rs`

```rust
/// Using common etcd library for client operations
///
/// ### Parameters
/// * `config: EtcdConfig` - Configuration for etcd client
/// ### Returns
/// * `EtcdClient` - Initialized etcd client from common library
pub async fn new_client(config: EtcdConfig) -> EtcdClient {}

/// Get value from etcd using common library
///
/// ### Parameters
/// * `key: &str` - Key to retrieve
/// ### Returns
/// * `Option<String>` - Value if found
pub async fn get(&self, key: &str) -> Option<String> {}

/// Put value in etcd using common library
///
/// ### Parameters
/// * `key: &str` - Key to store
/// * `value: &str` - Value to store
/// ### Returns
/// * `bool` - Success status
pub async fn put(&self, key: &str, value: &str) -> bool {}

/// Delete value from etcd using common library
///
/// ### Parameters
/// * `key: &str` - Key to delete
/// ### Returns
/// * `bool` - Success status
pub async fn delete(&self, key: &str) -> bool {}

/// Get values with prefix from etcd using common library
///
/// ### Parameters
/// * `prefix: &str` - Key prefix to match
/// ### Returns
/// * `HashMap<String, String>` - Map of matching keys to values
pub async fn get_prefix(&self, prefix: &str) -> HashMap<String, String> {}
```

### `etcd/watcher.rs`

```rust
/// Start watching etcd key prefix
///
/// ### Parameters
/// * `prefix: &str` - Key prefix to watch
/// * `handler: WatchHandler` - Callback to handle changes
/// ### Returns
/// * `WatchId` - ID of the watch task
pub async fn watch_prefix(&self, prefix: &str, handler: WatchHandler) -> WatchId {}

/// Stop watching etcd key prefix
///
/// ### Parameters
/// * `watch_id: WatchId` - ID of the watch task to stop
pub async fn cancel_watch(&self, watch_id: WatchId) {}

/// Process etcd watch events
///
/// ### Parameters
/// * `events: Vec<WatchEvent>` - List of watch events
/// ### Description
/// Internal function to process watch events and call the appropriate handlers
async fn process_watch_events(events: Vec<WatchEvent>) {}
```

### `grpc/client/action_controller.rs`

```rust
/// Connect to ActionController gRPC service
///
/// ### Parameters
/// * `address: &str` - ActionController service address
/// ### Returns
/// * `ActionControllerClient` - Connected client
pub async fn connect(address: &str) -> ActionControllerClient {}

/// Request container recovery action
///
/// ### Parameters
/// * `container_id: &str` - ID of the container to recover
/// * `action: RecoveryAction` - Type of recovery action to perform
/// ### Returns
/// * `bool` - Success status
pub async fn request_recovery(&self, container_id: &str, action: RecoveryAction) -> bool {}

/// Request container lifecycle action
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// * `action: LifecycleAction` - Type of lifecycle action to perform
/// ### Returns
/// * `bool` - Success status
pub async fn request_lifecycle_action(&self, container_id: &str, action: LifecycleAction) -> bool {}
```

### `grpc/client/monitoring_server.rs`

```rust
/// Connect to MonitoringServer gRPC service
///
/// ### Parameters
/// * `address: &str` - MonitoringServer service address
/// ### Returns
/// * `MonitoringServerClient` - Connected client
pub async fn connect(address: &str) -> MonitoringServerClient {}

/// Send state change alert to MonitoringServer
///
/// ### Parameters
/// * `alert: StateAlert` - Alert information
/// ### Returns
/// * `bool` - Success status
pub async fn send_alert(&self, alert: StateAlert) -> bool {}

/// Send state metrics to MonitoringServer
///
/// ### Parameters
/// * `metrics: StateMetrics` - Metrics information
/// ### Returns
/// * `bool` - Success status
pub async fn send_metrics(&self, metrics: StateMetrics) -> bool {}
```

### `grpc/server/service.rs`

```rust
/// StateManager gRPC service implementation
pub struct StateManagerService {
    // Service fields
}

/// Handle state update request
///
/// ### Parameters
/// * `request: StateUpdateRequest` - Update request details
/// ### Returns
/// * `StateUpdateResponse` - Response with result
pub async fn update_state(&self, request: StateUpdateRequest) -> StateUpdateResponse {}

/// Handle state query request
///
/// ### Parameters
/// * `request: StateQueryRequest` - Query request details
/// ### Returns
/// * `StateQueryResponse` - Response with state information
pub async fn query_state(&self, request: StateQueryRequest) -> StateQueryResponse {}

/// Handle subscription to state changes
///
/// ### Parameters
/// * `request: StateSubscriptionRequest` - Subscription request details
/// ### Returns
/// * `Stream<StateChangeEvent>` - Stream of state change events
pub async fn subscribe_to_state_changes(&self, request: StateSubscriptionRequest) -> Stream<StateChangeEvent> {}
```

### `reconciliation/planner.rs`

```rust
/// Create action plan based on state differences from NodeAgent
///
/// ### Parameters
/// * `diff: &StateDiff` - State differences received from NodeAgent
/// ### Returns
/// * `ActionPlan` - Plan of actions to resolve differences
pub fn create_action_plan(diff: &StateDiff) -> ActionPlan {}

/// Action plan structure
pub struct ActionPlan {
    pub resource_id: ResourceId,
    pub actions: Vec<PlannedAction>,
    pub priority: ActionPriority,
    pub estimated_duration: Duration,
    pub impact_assessment: ImpactAssessment,
}

/// Sort action plans by priority
///
/// ### Parameters
/// * `plans: &mut Vec<ActionPlan>` - Action plans to sort
pub fn sort_plans_by_priority(plans: &mut Vec<ActionPlan>) {}

/// Optimize action plan to minimize disruption
///
/// ### Parameters
/// * `plan: &mut ActionPlan` - Action plan to optimize
/// ### Returns
/// * `bool` - True if plan was optimized
pub fn optimize_plan(plan: &mut ActionPlan) -> bool {}
```

### `reconciliation/executor.rs`

```rust
/// Execute action plan
///
/// ### Parameters
/// * `plan: &ActionPlan` - Action plan to execute
/// ### Returns
/// * `ExecutionResult` - Result of execution
pub async fn execute_plan(plan: &ActionPlan) -> ExecutionResult {}

/// Execute single action using ActionController
///
/// ### Parameters
/// * `action: &PlannedAction` - Action to execute
/// ### Returns
/// * `ActionResult` - Result of the action
pub async fn execute_action(action: &PlannedAction) -> ActionResult {}

/// Check if action execution was successful
///
/// ### Parameters
/// * `result: &ActionResult` - Result of action execution
/// ### Returns
/// * `bool` - True if action was successful
pub fn is_action_successful(result: &ActionResult) -> bool {}

/// Handle action execution failure
///
/// ### Parameters
/// * `action: &PlannedAction` - Failed action
/// * `result: &ActionResult` - Execution result
/// ### Returns
/// * `RecoveryAction` - Recovery action to take
pub fn handle_execution_failure(action: &PlannedAction, result: &ActionResult) -> RecoveryAction {}
```

### `grpc/client/node_agent.rs`

```rust
/// Connect to NodeAgent gRPC service
///
/// ### Parameters
/// * `address: &str` - NodeAgent service address
/// ### Returns
/// * `NodeAgentClient` - Connected client
pub async fn connect(address: &str) -> NodeAgentClient {}

/// Request state diff from NodeAgent
///
/// ### Parameters
/// * `resource_id: &ResourceId` - Resource ID to check
/// ### Returns
/// * `StateDiff` - State differences
pub async fn get_state_diff(&self, resource_id: &ResourceId) -> StateDiff {}

/// Subscribe to state diff events from NodeAgent
///
/// ### Parameters
/// * `resource_filter: Option<ResourceFilter>` - Optional filter for specific resources
/// ### Returns
/// * `Stream<StateDiff>` - Stream of state diff events
pub async fn subscribe_state_diffs(&self, resource_filter: Option<ResourceFilter>) -> Stream<StateDiff> {}

/// Send updated state to NodeAgent
///
/// ### Parameters
/// * `resource_id: &ResourceId` - Resource ID
/// * `updated_state: &ResourceState` - Updated state information
/// ### Returns
/// * `bool` - Success status
pub async fn update_resource_state(&self, resource_id: &ResourceId, updated_state: &ResourceState) -> bool {}
```

### `recovery/action.rs`

```rust
/// Recovery action types
pub enum RecoveryAction {
    Restart,
    Recreate,
    Relocate,
    RollbackVersion,
    ResetConfig,
    Notify,
    Isolate,
}

/// Execute recovery action for a service
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// * `action: RecoveryAction` - Type of action to perform
/// ### Returns
/// * `RecoveryResult` - Result of the recovery action
pub async fn execute_recovery(&self, service_id: &str, action: RecoveryAction) -> RecoveryResult {}

/// Attempt minor recovery
///
/// ### Parameters
/// * `service_id: &str` - ID of the service to recover
/// ### Returns
/// * `bool` - Success status
async fn try_minor_recovery(service_id: &str) -> bool {}

/// Reset service configuration
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// ### Returns
/// * `bool` - Success status
async fn reset_service_config(service_id: &str) -> bool {}

/// Rollback to previous version
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// ### Returns
/// * `bool` - Success status
async fn rollback_to_previous_version(service_id: &str) -> bool {}

/// Log recovery attempt
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// * `action: &RecoveryAction` - Recovery action attempted
/// * `success: bool` - Whether the recovery was successful
async fn log_recovery_attempt(service_id: &str, action: &RecoveryAction, success: bool) {}
```

### `recovery/strategy.rs`

```rust
/// Recovery strategy configuration
pub struct RecoveryStrategy {
    pub max_retry_count: u32,
    pub backoff_delay: Duration,
    pub escalation_threshold: u32,
    pub actions: Vec<RecoveryAction>,
}

/// Get recovery strategy for a specific issue type
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// * `issue_type: IssueType` - Type of issue to recover from
/// ### Returns
/// * `RecoveryStrategy` - Recovery strategy
pub async fn get_recovery_strategy(service_id: &str, issue_type: IssueType) -> RecoveryStrategy {}

/// Determine next recovery action
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// * `issue_type: IssueType` - Type of issue
/// * `failure_count: u32` - Number of previous failures
/// ### Returns
/// * `RecoveryAction` - Next action to take
pub async fn determine_next_action(service_id: &str, issue_type: IssueType, failure_count: u32) -> RecoveryAction {}

/// Apply staged recovery process
///
/// ### Parameters
/// * `service_id: &str` - ID of the service
/// * `issue: &ServiceIssue` - Issue to recover from
/// ### Returns
/// * `RecoveryResult` - Result of recovery attempt
pub async fn attempt_staged_recovery(service_id: &str, issue: &ServiceIssue) -> RecoveryResult {}
```

### `state/events.rs`

```rust
/// State change event
pub struct StateEvent {
    pub resource_id: ResourceId,
    pub event_type: EventType,
    pub timestamp: DateTime<Utc>,
    pub changes: Vec<FieldChange>,
    pub source: EventSource,
    pub metadata: HashMap<String, String>,
}

/// Field change information
pub struct FieldChange {
    pub field: String,
    pub old_value: Option<Value>,
    pub new_value: Value,
}

/// Process state change event
///
/// ### Parameters
/// * `event: StateEvent` - Event to process
/// ### Returns
/// * `ProcessingResult` - Result of event processing
pub async fn process_event(event: StateEvent) -> ProcessingResult {}

/// Filter events by criteria
///
/// ### Parameters
/// * `events: &[StateEvent]` - Events to filter
/// * `criteria: &EventFilter` - Filter criteria
/// ### Returns
/// * `Vec<StateEvent>` - Filtered events
pub fn filter_events(events: &[StateEvent], criteria: &EventFilter) -> Vec<StateEvent> {}
```

### `state/model.rs`

```rust
/// Service state resource definition
pub struct ServiceState {
    pub metadata: ResourceMetadata,
    pub spec: ServiceSpec,
    pub status: ServiceStatus,
}

/// Service specification
pub struct ServiceSpec {
    pub replicas: u32,
    pub version: String,
    pub resources: ResourceRequirements,
    pub config: HashMap<String, String>,
    pub dependencies: Vec<ServiceDependency>,
    pub state: ServiceStateType,
}

/// Service status
pub struct ServiceStatus {
    pub observed_generation: u64,
    pub replicas: u32,
    pub available_replicas: u32,
    pub conditions: Vec<ServiceCondition>,
    pub state: ServiceStateType,
    pub last_reconcile_time: DateTime<Utc>,
}

/// Create new service state
///
/// ### Parameters
/// * `name: &str` - Service name
/// * `namespace: &str` - Service namespace
/// ### Returns
/// * `ServiceState` - New service state
pub fn new_service_state(name: &str, namespace: &str) -> ServiceState {}

/// Update service status
///
/// ### Parameters
/// * `state: &mut ServiceState` - Service state to update
/// * `status: ServiceStatus` - New status
pub fn update_service_status(state: &mut ServiceState, status: ServiceStatus) {}

/// Check if service is available
///
/// ### Parameters
/// * `state: &ServiceState` - Service state to check
/// ### Returns
/// * `bool` - True if service is available
pub fn is_service_available(state: &ServiceState) -> bool {}
```

### `state/tracker.rs`

```rust
/// Track and analyze state changes
pub struct StateTracker {
    pub resource_id: ResourceId,
    pub state_history: VecDeque<ServiceState>,
    pub transition_times: HashMap<(ServiceState, ServiceState), Vec<Duration>>,
}

/// Add state change to tracker
///
/// ### Parameters
/// * `tracker: &mut StateTracker` - Tracker to update
/// * `new_state: ServiceState` - New state to record
/// * `timestamp: DateTime<Utc>` - Time of state change
pub fn record_state_change(tracker: &mut StateTracker, new_state: ServiceState, timestamp: DateTime<Utc>) {}

/// Detect problematic state transition patterns
///
/// ### Parameters
/// * `tracker: &StateTracker` - State tracker to analyze
/// ### Returns
/// * `Vec<TransitionIssue>` - Detected issues
pub fn detect_transition_issues(tracker: &StateTracker) -> Vec<TransitionIssue> {}

/// Calculate state stability metrics
///
/// ### Parameters
/// * `tracker: &StateTracker` - State tracker to analyze
/// * `window: Duration` - Time window for analysis
/// ### Returns
/// * `StabilityMetrics` - Calculated stability metrics
pub fn calculate_stability_metrics(tracker: &StateTracker, window: Duration) -> StabilityMetrics {}
```

## 4. Reference information

### 4.1 Related Components
- **NodeAgent**: Monitors the state of each node, calculates differences (diffs) between actual state and desired state, and provides these to StateManager
- **ActionController**: Responsible for service lifecycle management and executes action plans from StateManager
- **ETCD**: Distributed key-value store for storing and sharing state information

### 4.2 Protocols and APIs
- **gRPC API**:
  - `StateUpdateRequest`: State update request
  - `StateQueryRequest`: State query request
  - `StateSubscriptionRequest`: State change subscription request
  - `StateDiffRequest`: State difference request from NodeAgent
  - `StateDiffSubscription`: State difference subscription from NodeAgent
  - `ActionExecutionRequest`: Action execution request to ActionController
- **ETCD API**: Common ETCD library interface for storing and retrieving state data

### 4.3 Data Structures
- **ServiceState**: Service state information (including spec, status)
- **StateDiff**: Differences between actual state and desired state received from NodeAgent
- **ActionPlan**: Action plan for resolving state differences
- **StateEvent**: State change event
- **StateMachine**: State machine managing service state transitions
- **RecoveryStrategy**: Recovery strategies by issue type
- **ResourceState**: Current state information of resources (services, containers, etc.)

### 4.4 References
- [ETCD API Documentation](https://etcd.io/docs/v3.5/learning/api/)
- [gRPC Documentation](https://grpc.io/docs/)
- [Kubernetes Reconciliation Pattern](https://kubernetes.io/docs/concepts/architecture/controller/)
- [State Machine Theory](https://en.wikipedia.org/wiki/Finite-state_machine)
- [NodeAgent State Monitoring Interface](internal://nodeagent.md)
