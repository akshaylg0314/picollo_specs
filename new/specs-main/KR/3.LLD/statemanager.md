# State Manager

## 1. Introduction

### Major features

State Manager는 PICCOLO 프레임워크 내에서 시스템 및 서비스의 상태를 관리하는 핵심 컴포넌트이다.

1. NodeAgent로부터 수신한 상태 차이(diff)를 기반으로 조정(reconciliation) 작업을 수행한다.
2. 서비스의 전체 생명주기를 관리하는 상태 머신을 구현한다. (Pending, Initializing, Running, Degraded, Terminating, Failed)
3. 상태 불일치를 해결하며 장애 상황에서 자동 복구를 수행한다.
4. ETCD를 통해 상태 정보를 저장하고 다른 컴포넌트와 공유한다.
5. 상태 변경 이벤트를 추적하고 처리하여 시스템 안정성을 유지한다.
6. ActionController와 연동하여 필요한 조치를 실행한다.

### Main Dataflow

1. NodeAgent로부터 상태 차이(State Diff) 정보를 수신한다.
2. 수신된 상태 차이에 따라 조치 계획(Action Planning)을 수립한다.
3. 조치 계획을 ActionController를 통해 실행한다.
4. 조치 결과에 따라 상태를 업데이트하고 결과를 보고한다.
5. 상태 변화 및 이벤트를 ETCD에 기록한다.
6. 최신 상태 정보를 NodeAgent와 다른 컴포넌트에 제공한다.

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

- **main.rs** - StateManager 초기화 및 실행
- **manager.rs** - 각 모듈 간의 데이터 흐름 제어
- **etcd/mod.rs** - etcd 연동 관리
- **etcd/client.rs** - 공통 etcd 라이브러리 사용
- **etcd/watcher.rs** - etcd 키 변화 감지 및 처리
- **grpc/mod.rs** - gRPC 통신 관리
- **grpc/client/mod.rs** - gRPC 클라이언트 관리
- **grpc/client/action_controller.rs** - ActionController와의 gRPC 통신
- **grpc/client/node_agent.rs** - NodeAgent와의 gRPC 통신 및 상태 차이(diff) 수신
- **grpc/server/mod.rs** - gRPC 서버 관리
- **grpc/server/service.rs** - StateManager gRPC 서비스 구현
- **reconciliation/mod.rs** - 상태 조정 기능 관리
- **reconciliation/planner.rs** - NodeAgent로부터 받은 차이 정보를 기반으로 조치 계획 수립
- **reconciliation/executor.rs** - 조치 계획 실행
- **recovery/mod.rs** - 장애 복구 관리
- **recovery/action.rs** - 복구 액션 구현
- **recovery/strategy.rs** - 상태 유형별 복구 전략 정의
- **state/mod.rs** - 상태 관리
- **state/machine.rs** - 상태 머신 구현
- **state/model.rs** - 상태 모델 정의
- **state/events.rs** - 상태 변경 이벤트 관리
- **state/tracker.rs** - 상태 변화 추적

## 3. Function information

본 문단의 function 정보는 rustdoc 에서 사용하는 주석 형태로 작성되었다.
이에 대해서는 [링크](https://doc.rust-lang.org/stable/rustdoc/index.html) 를 참조하라.

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

### 4.1 관련 컴포넌트
- **NodeAgent**: 각 노드의 상태를 모니터링하고 실제 상태와 원하는 상태 간 차이(diff)를 계산하여 StateManager에 제공
- **ActionController**: 서비스 라이프사이클 관리를 담당하며 StateManager의 조치 계획을 실행
- **ETCD**: 상태 정보 저장 및 공유를 위한 분산 키-값 저장소

### 4.2 프로토콜 및 API
- **gRPC API**:
  - `StateUpdateRequest`: 상태 업데이트 요청
  - `StateQueryRequest`: 상태 조회 요청
  - `StateSubscriptionRequest`: 상태 변경 구독 요청
  - `StateDiffRequest`: NodeAgent로부터 상태 차이 요청
  - `StateDiffSubscription`: NodeAgent로부터 상태 차이 구독
  - `ActionExecutionRequest`: ActionController에 조치 실행 요청
- **ETCD API**: 상태 데이터 저장 및 조회를 위한 공통 ETCD 라이브러리 인터페이스

### 4.3 데이터 구조
- **ServiceState**: 서비스 상태 정보 (spec, status 포함)
- **StateDiff**: NodeAgent로부터 수신한 실제 상태와 원하는 상태 간의 차이점
- **ActionPlan**: 상태 차이 해결을 위한 조치 계획
- **StateEvent**: 상태 변경 이벤트
- **StateMachine**: 서비스 상태 전이를 관리하는 상태 머신
- **RecoveryStrategy**: 이슈 유형별 복구 전략
- **ResourceState**: 리소스(서비스, 컨테이너 등)의 최신 상태 정보

### 4.4 참고 자료
- [ETCD API 문서](https://etcd.io/docs/v3.5/learning/api/)
- [gRPC 문서](https://grpc.io/docs/)
- [Kubernetes Reconciliation 패턴](https://kubernetes.io/docs/concepts/architecture/controller/)
- [상태 머신 이론](https://en.wikipedia.org/wiki/Finite-state_machine)
- [NodeAgent 상태 모니터링 인터페이스](internal://nodeagent.md)
