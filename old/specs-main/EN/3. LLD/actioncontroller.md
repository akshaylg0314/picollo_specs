# Action Controller

## 1. Introduction

### Major features

Action Controller is a core component that manages container lifecycle by receiving requests from FilterGateway.

1. Receives and processes action execution requests from FilterGateway via gRPC.
2. Validates the executability of requested actions through gRPC communication with PolicyManager.
3. Converts Pullpiri declarative specifications (Package, Model) into container runtime commands to perform container creation, start, stop, restart, remove, and other operations.
4. Provides an abstracted interface to support various container runtimes (Podman, Docker, Containerd, Bluechi).
5. Manages resource configuration such as CPU, memory, and storage allocation through gRPC API.
6. Processes the application of TIMPANI scheduling parameters through gRPC API.
7. Directly accesses ETCD to manage state information to maintain system state consistency.
8. Processes container network configuration, volume mounts, and security settings through gRPC API.
9. Integrates control of Bluechi service management and inter-node communication.
10. Ensures secure container execution through security context and permission management.

### Main Dataflow

1. Receives action execution requests from FilterGateway via gRPC.
2. Determines necessary operations and assigns priorities based on action type (execute, stop, update, remove).
3. Validates the executability of the action through gRPC communication with PolicyManager.
4. Analyzes network and volume requirements and requests necessary resources through gRPC API.
5. Converts commands to match the specific container runtime (Podman, Docker, Containerd, Bluechi) through the runtime adapter pattern and executes them.
6. For Bluechi runtime, generates service management commands and configures service dependencies and communication between nodes.
7. For real-time applications, calls the TIMPANI scheduler gRPC API to configure CPU core isolation and real-time scheduling properties.
8. Stores the operation results in ETCD to maintain system state consistency, without performing container state monitoring or real-time updates.
9. Responds to FilterGateway with the action execution results.

## 2. File information

```text
actioncontroller
├── actioncontroller.md
├── Cargo.toml
└── src
    ├── action
    │   ├── executor.rs
    │   ├── mod.rs
    │   ├── plan.rs
    │   └── validation.rs
    ├── config
    │   ├── loader.rs
    │   ├── mod.rs
    │   └── validation.rs
    ├── grpc
    │   ├── client
    │   │   ├── mod.rs
    │   │   ├── monitoring_server.rs
    │   │   ├── policy_manager.rs
    │   │   └── state_manager.rs
    │   ├── mod.rs
    │   └── server
    │       ├── filter_gateway.rs
    │       └── mod.rs
    ├── main.rs
    ├── manager.rs
    ├── runtime
    │   ├── adapter.rs
    │   ├── containerd
    │   │   ├── client.rs
    │   │   └── mod.rs
    │   ├── docker
    │   │   ├── client.rs
    │   │   └── mod.rs
    │   ├── bluechi
    │   │   ├── client.rs
    │   │   ├── mod.rs
    │   │   └── service.rs
    │   ├── factory.rs
    │   ├── mod.rs
    │   └── podman
    │       ├── client.rs
    │       └── mod.rs
    ├── security
    │   ├── mod.rs
    │   ├── context.rs
    │   └── permissions.rs
    └── storage
        ├── etcd_state.rs
        └── mod.rs
```

- **main.rs** - ActionController initialization and execution
- **manager.rs** - Control of data flow and coordination between modules
- **action/mod.rs** - Action management and coordination
- **action/executor.rs** - Action execution and result processing
- **action/plan.rs** - Action planning and optimization
- **action/validation.rs** - Action validation
- **config/mod.rs** - Configuration management
- **config/loader.rs** - Configuration loading and parsing
- **config/validation.rs** - Configuration validation
- **grpc/mod.rs** - gRPC communication management
- **grpc/client/mod.rs** - gRPC client management
- **grpc/client/state_manager.rs** - gRPC communication with StateManager
- **grpc/client/monitoring_server.rs** - gRPC communication with MonitoringServer
- **grpc/client/policy_manager.rs** - gRPC communication with PolicyManager (validation of executability)
- **grpc/server/mod.rs** - gRPC server management
- **grpc/server/filter_gateway.rs** - gRPC communication with FilterGateway
- **runtime/mod.rs** - Container runtime abstraction
- **runtime/adapter.rs** - Runtime adapter interface and abstract factory pattern
- **runtime/factory.rs** - Runtime factory pattern implementation
- **runtime/podman/mod.rs** - Podman runtime support
- **runtime/podman/client.rs** - Podman API client
- **runtime/docker/mod.rs** - Docker runtime support
- **runtime/docker/client.rs** - Docker API client
- **runtime/containerd/mod.rs** - Containerd runtime support
- **runtime/containerd/client.rs** - Containerd API client
- **runtime/bluechi/mod.rs** - Bluechi runtime support
- **runtime/bluechi/client.rs** - Bluechi API client
- **runtime/bluechi/service.rs** - Bluechi service management
- **security/mod.rs** - Security management
- **security/context.rs** - Container security context management
- **security/permissions.rs** - Permission and access control
- **storage/mod.rs** - ETCD state management module
- **storage/etcd_state.rs** - Functions related to ETCD state reading/writing

## 3. Function information

The function information in this section is written in the comment format used in rustdoc.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo Action Controller
///
/// ### Description
/// Initializes logging, configuration, and starts the ActionController service.
/// Handles graceful shutdown on SIGINT and SIGTERM signals.
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize action controller components and start gRPC server
///
/// ### Description
/// Sets up runtime adapters, connects to required services (StateManager, PolicyManager),
/// and initializes core functionality.
pub async fn initialize() {}

/// Start all action controller services and background tasks
///
/// ### Returns
/// * `common::Result<()>` - Result of services startup
pub async fn start_services() -> common::Result<()> {}

/// Stop all action controller services and release resources
///
/// ### Description
/// Performs graceful shutdown of all services, cancels background tasks,
/// and ensures proper cleanup of resources.
pub async fn stop_services() {}

/// Execute action based on request from filter gateway
///
/// ### Parameters
/// * `request: ExecuteActionRequest` - Action execution request containing action type, target and parameters
/// ### Returns
/// * `common::Result<ExecuteActionResponse>` - Result of action execution with status and metadata
/// ### Description
/// Main entry point for executing container lifecycle operations.
/// Validates requests with PolicyManager, executes operations via appropriate runtime adapter,
/// and updates ETCD with execution results to maintain system state consistency.
pub async fn execute_action(request: ExecuteActionRequest) -> common::Result<ExecuteActionResponse> {}

/// Cancel ongoing action execution
///
/// ### Parameters
/// * `request: CancelActionRequest` - Action cancellation request
/// ### Returns
/// * `common::Result<CancelActionResponse>` - Result of action cancellation
pub async fn cancel_action(request: CancelActionRequest) -> common::Result<CancelActionResponse> {}

/// Execute batch actions in optimal order
///
/// ### Parameters
/// * `requests: Vec<ExecuteActionRequest>` - List of action execution requests
/// ### Returns
/// * `common::Result<Vec<ExecuteActionResponse>>` - Results of action executions
pub async fn execute_batch_actions(requests: Vec<ExecuteActionRequest>) -> common::Result<Vec<ExecuteActionResponse>> {}

/// Validate action with PolicyManager
///
/// ### Parameters
/// * `request: &ExecuteActionRequest` - Action request to validate
/// ### Returns
/// * `common::Result<PolicyValidationResult>` - Validation result from PolicyManager
/// ### Description
/// Sends action request to PolicyManager for validation before execution.
async fn validate_with_policy_manager(request: &ExecuteActionRequest) -> common::Result<PolicyValidationResult> {}

/// Update container state in ETCD
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// * `status: ContainerStatus` - New status of the container
/// ### Returns
/// * `common::Result<()>` - Result of state update
/// ### Description
/// Updates container state in ETCD for system state consistency. This does not 
/// send real-time notifications to StateManager, which will retrieve updates via ETCD.
async fn update_container_state(container_id: &str, status: ContainerStatus) -> common::Result<()> {}

/// Send container metrics to monitoring server
///
/// ### Parameters
/// * `metrics: ContainerMetrics` - Container performance metrics
/// ### Returns
/// * `common::Result<()>` - Result of sending metrics
async fn send_metrics(metrics: ContainerMetrics) -> common::Result<()> {}
```

### `action/executor.rs`

```rust
/// Execute planned action with specific runtime adapter
///
/// ### Parameters
/// * `action_plan: &ActionPlan` - Action plan with details
/// * `runtime: &dyn RuntimeAdapter` - Runtime adapter to use for execution
/// ### Returns
/// * `common::Result<ActionResult>` - Result of action execution
/// ### Description
/// Core function for executing planned actions against container runtimes
/// using the appropriate runtime adapter.
pub async fn execute(action_plan: &ActionPlan, runtime: &dyn RuntimeAdapter) -> common::Result<ActionResult> {}

/// Execute batch actions with optimized ordering
///
/// ### Parameters
/// * `action_plans: &[ActionPlan]` - Multiple action plans to execute
/// * `runtime: &dyn RuntimeAdapter` - Runtime adapter to use
/// ### Returns
/// * `common::Result<Vec<ActionResult>>` - Results of action executions
/// ### Description
/// Optimizes the execution order of multiple actions for maximum parallelism
/// and dependency satisfaction.
pub async fn execute_batch(action_plans: &[ActionPlan], runtime: &dyn RuntimeAdapter) -> common::Result<Vec<ActionResult>> {}

/// Track action execution progress
///
/// ### Parameters
/// * `action_id: &str` - ID of the action to track
/// ### Returns
/// * `common::Result<ActionProgress>` - Progress information
/// ### Description
/// Provides real-time tracking of action execution with completion percentage and status.
pub async fn track_progress(action_id: &str) -> common::Result<ActionProgress> {}

/// Update ETCD with execution result
///
/// ### Parameters
/// * `action_result: &ActionResult` - Result of action execution
/// ### Returns
/// * `common::Result<()>` - Update result
/// ### Description
/// Updates ETCD with action execution result to maintain system state consistency.
/// This does not send real-time updates to StateManager, which will retrieve state updates from ETCD.
async fn update_etcd_state(action_result: &ActionResult) -> common::Result<()> {}
```

### `action/plan.rs`

```rust
/// Create an action plan from an execution request
///
/// ### Parameters
/// * `request: &ExecuteActionRequest` - Request from FilterGateway
/// ### Returns
/// * `common::Result<ActionPlan>` - Generated action plan
/// ### Description
/// Transforms high-level action request into executable steps with dependencies.
pub fn create_plan(request: &ExecuteActionRequest) -> common::Result<ActionPlan> {}

/// Validate action plan before execution
///
/// ### Parameters
/// * `plan: &ActionPlan` - Action plan to validate
/// ### Returns
/// * `common::Result<ValidationResult>` - Validation result with warnings or errors
/// ### Description
/// Performs comprehensive validation of action plan for executability and constraints.
pub fn validate_plan(plan: &ActionPlan) -> common::Result<ValidationResult> {}

/// Optimize action plan for efficient execution
///
/// ### Parameters
/// * `plan: &mut ActionPlan` - Action plan to optimize
/// ### Returns
/// * `common::Result<()>` - Optimization result
/// ### Description
/// Reorders steps and identifies parallelizable operations to improve efficiency.
pub fn optimize_plan(plan: &mut ActionPlan) -> common::Result<()> {}

/// Merge multiple action plans for batch execution
///
/// ### Parameters
/// * `plans: &[ActionPlan]` - Plans to merge
/// ### Returns
/// * `common::Result<ActionPlan>` - Merged action plan
/// ### Description
/// Creates a composite action plan that satisfies all dependencies efficiently.
pub fn merge_plans(plans: &[ActionPlan]) -> common::Result<ActionPlan> {}
```

### `action/validation.rs`

```rust
/// Validate action request against system constraints
///
/// ### Parameters
/// * `request: &ExecuteActionRequest` - Action request to validate
/// ### Returns
/// * `common::Result<ValidationResult>` - Validation result with details
/// ### Description
/// Initiates validation with PolicyManager and processes the response.
pub fn validate_request(request: &ExecuteActionRequest) -> common::Result<ValidationResult> {}

/// Convert validation result from PolicyManager to local format
///
/// ### Parameters
/// * `policy_result: &PolicyValidationResult` - Result from PolicyManager
/// ### Returns
/// * `ValidationResult` - Converted validation result
/// ### Description
/// Transforms PolicyManager's validation format to ActionController's internal format.
fn convert_policy_validation(policy_result: &PolicyValidationResult) -> ValidationResult {}

/// Validate security context and permissions
///
/// ### Parameters
/// * `security_context: &SecurityContext` - Security context to validate
/// ### Returns
/// * `common::Result<SecurityValidation>` - Security validation result
/// ### Description
/// Verifies security context against system policies and restrictions.
pub fn validate_security(security_context: &SecurityContext) -> common::Result<SecurityValidation> {}
```

### `application/lifecycle.rs`

```rust
/// Start application with all dependencies
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `config: &AppConfig` - Application configuration
/// ### Returns
/// * `common::Result<AppInstance>` - Started application instance
/// ### Description
/// Manages the complete start sequence including dependency resolution.
pub async fn start_application(app_id: &str, config: &AppConfig) -> common::Result<AppInstance> {}

/// Stop application and release resources
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `options: &StopOptions` - Stop options (graceful, timeout)
/// ### Returns
/// * `common::Result<()>` - Stop result
/// ### Description
/// Performs graceful shutdown with configurable timeout before forced termination.
pub async fn stop_application(app_id: &str, options: &StopOptions) -> common::Result<()> {}

/// Update running application
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `config: &AppConfig` - New application configuration
/// ### Returns
/// * `common::Result<UpdateResult>` - Update result with status
/// ### Description
/// Performs in-place or rolling update based on configuration and constraints.
pub async fn update_application(app_id: &str, config: &AppConfig) -> common::Result<UpdateResult> {}

/// Perform application lifecycle transition
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `transition: LifecycleTransition` - Requested transition
/// ### Returns
/// * `common::Result<TransitionResult>` - Transition result
/// ### Description
/// Manages application state transitions with validation and events.
pub async fn transition_state(app_id: &str, transition: LifecycleTransition) -> common::Result<TransitionResult> {}

/// Get current lifecycle status of application
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// ### Returns
/// * `common::Result<LifecycleStatus>` - Current lifecycle status
/// ### Description
/// Provides detailed application lifecycle status including substates and health.
pub async fn get_lifecycle_status(app_id: &str) -> common::Result<LifecycleStatus> {}
```

### `application/monitor.rs`

```rust
/// Start monitoring an application
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `config: &MonitorConfig` - Monitoring configuration
/// ### Returns
/// * `common::Result<MonitorHandle>` - Handle for monitor control
/// ### Description
/// Starts comprehensive application monitoring including resource usage and health.
pub async fn start_monitoring(app_id: &str, config: &MonitorConfig) -> common::Result<MonitorHandle> {}

/// Collect point-in-time application metrics
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `metric_types: &[MetricType]` - Types of metrics to collect
/// ### Returns
/// * `common::Result<ApplicationMetrics>` - Collected application metrics
/// ### Description
/// Gathers specified metrics for application without continuous monitoring.
pub async fn collect_metrics(app_id: &str, metric_types: &[MetricType]) -> common::Result<ApplicationMetrics> {}

/// Register alert condition for application
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `condition: AlertCondition` - Alert trigger condition
/// * `handler: AlertHandler` - Alert handling function
/// ### Returns
/// * `common::Result<AlertId>` - Identifier for registered alert
/// ### Description
/// Configures alerting based on application metrics or status conditions.
pub async fn register_alert(
    app_id: &str,
    condition: AlertCondition,
    handler: AlertHandler,
) -> common::Result<AlertId> {}

/// Send application metrics to monitoring server
///
/// ### Parameters
/// * `metrics: &ApplicationMetrics` - Metrics to send
/// ### Returns
/// * `common::Result<()>` - Send result
/// ### Description
/// Forwards collected application metrics to the monitoring server.
async fn send_to_monitoring_server(metrics: &ApplicationMetrics) -> common::Result<()> {}
```

### `application/status.rs`

```rust
/// Get detailed application status
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// ### Returns
/// * `common::Result<ApplicationStatus>` - Comprehensive application status
/// ### Description
/// Provides detailed status including runtime metrics, health, and dependencies.
pub async fn get_status(app_id: &str) -> common::Result<ApplicationStatus> {}

/// Subscribe to application status changes
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `status_handler: F` - Status change handler function
/// ### Returns
/// * `common::Result<Subscription>` - Status subscription handle
/// ### Description
/// Provides real-time notifications of application status changes.
pub async fn subscribe_status<F>(
    app_id: &str,
    status_handler: F,
) -> common::Result<Subscription>
where
    F: Fn(ApplicationStatus) + Send + Sync + 'static,
{}

/// Wait for specific application condition
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `condition: ApplicationCondition` - Target condition
/// * `timeout: Duration` - Maximum wait time
/// ### Returns
/// * `common::Result<bool>` - Whether condition was satisfied
/// ### Description
/// Waits until application reaches specified condition or timeout expires.
pub async fn wait_for_condition(
    app_id: &str,
    condition: ApplicationCondition,
    timeout: Duration,
) -> common::Result<bool> {}

/// Check if application is healthy
///
/// ### Parameters
/// * `app_id: &str` - Application identifier
/// * `probe_type: ProbeType` - Type of health check to perform
/// ### Returns
/// * `common::Result<HealthStatus>` - Health check result
/// ### Description
/// Performs specified health check against application endpoints.
pub async fn check_health(
    app_id: &str,
    probe_type: ProbeType,
) -> common::Result<HealthStatus> {}
```

### `grpc/server/filter_gateway.rs`

```rust
/// Implement Action Controller Service for Filter Gateway
#[derive(Debug, Default)]
pub struct ActionControllerService {}

#[tonic::async_trait]
impl ActionController for ActionControllerService {
    /// Execute action requested by filter gateway
    ///
    /// ### Parameters
    /// * `request: Request<ExecuteActionRequest>` - Action execution request
    /// ### Returns
    /// * `Result<Response<ExecuteActionResponse>, Status>` - Action execution response
    async fn execute_action(
        &self,
        request: Request<ExecuteActionRequest>,
    ) -> Result<Response<ExecuteActionResponse>, Status> {
        let request = request.into_inner();
        
        match manager::execute_action(request).await {
            Ok(response) => Ok(Response::new(response)),
            Err(e) => Err(Status::internal(format!("Failed to execute action: {}", e))),
        }
    }

    /// Cancel action requested by filter gateway
    ///
    /// ### Parameters
    /// * `request: Request<CancelActionRequest>` - Action cancellation request
    /// ### Returns
    /// * `Result<Response<CancelActionResponse>, Status>` - Action cancellation response
    async fn cancel_action(
        &self,
        request: Request<CancelActionRequest>,
    ) -> Result<Response<CancelActionResponse>, Status> {
        let request = request.into_inner();
        
        match manager::cancel_action(request).await {
            Ok(response) => Ok(Response::new(response)),
            Err(e) => Err(Status::internal(format!("Failed to cancel action: {}", e))),
        }
    }
}
```

### `grpc/client/state_manager.rs`

```rust
/// [DEPRECATED] Update container state in ETCD directly
///
/// ### Parameters
/// * `container_id: &str` - Container ID
/// * `status: ContainerStatus` - Container status
/// ### Returns
/// * `common::Result<()>` - Update result
/// ### Description
/// Updates container state directly in ETCD without sending real-time updates to StateManager.
/// StateManager will independently retrieve state information from ETCD as needed.
pub async fn update_container_state_in_etcd(
    container_id: &str,
    status: ContainerStatus,
) -> common::Result<()> {
    // Implementation using common ETCD library
}

/// [DEPRECATED] Update package state in ETCD directly
///
/// ### Parameters
/// * `package_id: &str` - Package ID
/// * `status: PackageStatus` - Package status
/// ### Returns
/// * `common::Result<()>` - Update result
/// ### Description
/// Updates package state directly in ETCD without sending real-time updates to StateManager.
/// StateManager will independently retrieve state information from ETCD as needed.
pub async fn update_package_state_in_etcd(
    package_id: &str,
    status: PackageStatus,
) -> common::Result<()> {
    // Implementation using common ETCD library
}
```

### `grpc/client/policy_manager.rs`

```rust
/// Connect to PolicyManager gRPC service
///
/// ### Parameters
/// * `address: &str` - PolicyManager service address
/// ### Returns
/// * `PolicyManagerClient` - Connected client
pub async fn connect(address: &str) -> PolicyManagerClient {}

/// Validate action against policies
///
/// ### Parameters
/// * `request: &ActionValidationRequest` - Action to validate
/// ### Returns
/// * `Result<ActionValidationResponse, Status>` - Validation response
pub async fn validate_action(request: &ActionValidationRequest) -> Result<ActionValidationResponse, Status> {
    let mut client = PolicyManagerClient::connect(connect_server()).await?;
    client.validate_action(Request::new(request.clone())).await
}

/// Check resource conflicts
///
/// ### Parameters
/// * `resource_request: &ResourceRequest` - Resources being requested
/// ### Returns
/// * `Result<ResourceConflictResponse, Status>` - Conflict check result
pub async fn check_resource_conflicts(
    resource_request: &ResourceRequest
) -> Result<ResourceConflictResponse, Status> {
    let mut client = PolicyManagerClient::connect(connect_server()).await?;
    client.check_resource_conflicts(Request::new(resource_request.clone())).await
}
```

### `grpc/client/timpani_scheduler.rs`

```rust
/// Connect to TIMPANI Scheduler gRPC service
///
/// ### Parameters
/// * `address: &str` - TIMPANI Scheduler service address
/// ### Returns
/// * `TimpaniSchedulerClient` - Connected client
pub async fn connect(address: &str) -> TimpaniSchedulerClient {}

/// Apply real-time scheduling properties to container
///
/// ### Parameters
/// * `container_id: &str` - Container ID
/// * `rt_properties: &RealtimeProperties` - Real-time scheduling properties
/// ### Returns
/// * `Result<Response<ApplySchedulingResponse>, Status>` - Apply result
/// ### Description
/// Configures CPU core isolation and real-time scheduling properties for containers
/// that require deterministic execution.
pub async fn apply_realtime_scheduling(
    container_id: &str,
    rt_properties: &RealtimeProperties,
) -> Result<Response<ApplySchedulingResponse>, Status> {
    let mut client = TimpaniSchedulerClient::connect(connect_server()).await?;
    let request = ApplySchedulingRequest {
        container_id: container_id.to_string(),
        properties: Some(rt_properties.clone()),
    };
    client.apply_scheduling(Request::new(request)).await
}

/// Request CPU cores for container
///
/// ### Parameters
/// * `container_id: &str` - Container ID
/// * `core_request: &CoreRequest` - CPU core requirements
/// ### Returns
/// * `Result<Response<CoreAllocationResponse>, Status>` - Allocation result
/// ### Description
/// Requests specific or isolated CPU cores for container execution based on
/// performance and isolation requirements.
pub async fn allocate_cores(
    container_id: &str,
    core_request: &CoreRequest,
) -> Result<Response<CoreAllocationResponse>, Status> {
    let mut client = TimpaniSchedulerClient::connect(connect_server()).await?;
    let request = CoreAllocationRequest {
        container_id: container_id.to_string(),
        requirements: Some(core_request.clone()),
    };
    client.allocate_cores(Request::new(request)).await
}

/// Release allocated CPU cores
///
/// ### Parameters
/// * `container_id: &str` - Container ID
/// ### Returns
/// * `Result<Response<ReleaseResourcesResponse>, Status>` - Release result
/// ### Description
/// Releases CPU cores previously allocated to a container when it's stopped or removed.
pub async fn release_cores(
    container_id: &str,
) -> Result<Response<ReleaseResourcesResponse>, Status> {
    let mut client = TimpaniSchedulerClient::connect(connect_server()).await?;
    let request = ReleaseResourcesRequest {
        container_id: container_id.to_string(),
    };
    client.release_resources(Request::new(request)).await
}
```

### `podman/api.rs`

```rust
/// Podman API interface
pub trait PodmanApi {
    /// Create container
    ///
    /// ### Parameters
    /// * `config: ContainerConfig` - Container configuration
    /// ### Returns
    /// * `common::Result<String>` - ID of created container
    fn create_container(&self, config: ContainerConfig) -> BoxFuture<'_, common::Result<String>>;

    /// Start container
    ///
    /// ### Parameters
    /// * `container_id: &str` - Container ID
    /// ### Returns
    /// * `common::Result<()>` - Start result
    fn start_container(&self, container_id: &str) -> BoxFuture<'_, common::Result<()>>;

    /// Stop container
    ///
    /// ### Parameters
    /// * `container_id: &str` - Container ID
    /// ### Returns
    /// * `common::Result<()>` - Stop result
    fn stop_container(&self, container_id: &str) -> BoxFuture<'_, common::Result<()>>;

    /// Remove container
    ///
    /// ### Parameters
    /// * `container_id: &str` - Container ID
    /// ### Returns
    /// * `common::Result<()>` - Remove result
    fn remove_container(&self, container_id: &str) -> BoxFuture<'_, common::Result<()>>;

    /// Restart container
    ///
    /// ### Parameters
    /// * `container_id: &str` - Container ID
    /// ### Returns
    /// * `common::Result<()>` - Restart result
    fn restart_container(&self, container_id: &str) -> BoxFuture<'_, common::Result<()>>;

    /// Get container status
    ///
    /// ### Parameters
    /// * `container_id: &str` - Container ID
    /// ### Returns
    /// * `common::Result<ContainerStatus>` - Container status
    fn get_container_status(&self, container_id: &str) -> BoxFuture<'_, common::Result<ContainerStatus>>;

    /// List containers
    ///
    /// ### Parameters
    /// * `filters: HashMap<String, String>` - Filters for container list
    /// ### Returns
    /// * `common::Result<Vec<ContainerInfo>>` - List of containers
    fn list_containers(&self, filters: HashMap<String, String>) -> BoxFuture<'_, common::Result<Vec<ContainerInfo>>>;
}
```

### `podman/client.rs`

```rust
/// Podman client implementation
pub struct PodmanClient {
    /// Podman API endpoint
    endpoint: String,
    /// HTTP client
    client: Client,
}

impl PodmanClient {
    /// Create new Podman client
    ///
    /// ### Parameters
    /// * `endpoint: String` - Podman API endpoint
    /// ### Returns
    /// * `Self` - Podman client
    pub fn new(endpoint: String) -> Self {
        Self {
            endpoint,
            client: Client::new(),
        }
    }

    /// Build API URL
    ///
    /// ### Parameters
    /// * `path: &str` - API path
    /// ### Returns
    /// * `String` - Full API URL
    fn api_url(&self, path: &str) -> String {
        format!("{}{}", self.endpoint, path)
    }
}

#[async_trait]
impl PodmanApi for PodmanClient {
    /// Create container implementation
    async fn create_container(&self, config: ContainerConfig) -> common::Result<String> {
        // Implementation details
    }

    /// Start container implementation
    async fn start_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }

    /// Stop container implementation
    async fn stop_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }

    /// Remove container implementation
    async fn remove_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }

    /// Restart container implementation
    async fn restart_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }

    /// Get container status implementation
    async fn get_container_status(&self, container_id: &str) -> common::Result<ContainerStatus> {
        // Implementation details
    }

    /// List containers implementation
    async fn list_containers(&self, filters: HashMap<String, String>) -> common::Result<Vec<ContainerInfo>> {
        // Implementation details
    }
}
```

### `runtime/docker/client.rs`

```rust
/// Docker runtime adapter implementation
pub struct DockerAdapter {
    /// Docker API endpoint
    endpoint: String,
    /// HTTP client
    client: Client,
    /// API version
    api_version: String,
}

impl DockerAdapter {
    /// Create new Docker adapter
    ///
    /// ### Parameters
    /// * `config: &DockerConfig` - Docker configuration
    /// ### Returns
    /// * `common::Result<Self>` - Created Docker adapter
    pub fn new(config: &DockerConfig) -> common::Result<Self> {
        // Implementation details
    }
    
    /// Build Docker API URL
    ///
    /// ### Parameters
    /// * `path: &str` - API path
    /// ### Returns
    /// * `String` - Full API URL
    fn api_url(&self, path: &str) -> String {
        // Implementation details
    }
}

#[async_trait]
impl RuntimeAdapter for DockerAdapter {
    /// Create container using Docker runtime
    async fn create_container(&self, config: &ContainerConfig) -> common::Result<ContainerInstance> {
        // Implementation details
    }
    
    /// Start container with Docker runtime
    async fn start_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }
    
    /// Stop container with Docker runtime
    async fn stop_container(&self, container_id: &str, options: &StopOptions) -> common::Result<()> {
        // Implementation details
    }
    
    /// Remove container with Docker runtime
    async fn remove_container(&self, container_id: &str, force: bool) -> common::Result<()> {
        // Implementation details
    }
    
    /// Get Docker runtime capabilities
    fn capabilities(&self) -> RuntimeCapabilities {
        // Implementation details
    }
}
```

### `runtime/containerd/client.rs`

```rust
/// Containerd runtime adapter implementation
pub struct ContainerdAdapter {
    /// Containerd API endpoint
    endpoint: String,
    /// gRPC client
    client: ContainerdClient,
    /// Namespace
    namespace: String,
}

impl ContainerdAdapter {
    /// Create new Containerd adapter
    ///
    /// ### Parameters
    /// * `config: &ContainerdConfig` - Containerd configuration
    /// ### Returns
    /// * `common::Result<Self>` - Created Containerd adapter
    pub async fn new(config: &ContainerdConfig) -> common::Result<Self> {
        // Implementation details
    }
}

#[async_trait]
impl RuntimeAdapter for ContainerdAdapter {
    /// Create container using Containerd runtime
    async fn create_container(&self, config: &ContainerConfig) -> common::Result<ContainerInstance> {
        // Implementation details
    }
    
    /// Start container with Containerd runtime
    async fn start_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }
    
    /// Stop container with Containerd runtime
    async fn stop_container(&self, container_id: &str, options: &StopOptions) -> common::Result<()> {
        // Implementation details
    }
    
    /// Remove container with Containerd runtime
    async fn remove_container(&self, container_id: &str, force: bool) -> common::Result<()> {
        // Implementation details
    }
    
    /// Get Containerd runtime capabilities
    fn capabilities(&self) -> RuntimeCapabilities {
        // Implementation details
    }
}
```

### `runtime/bluechi/client.rs`

```rust
/// Bluechi runtime adapter implementation
pub struct BluechiAdapter {
    /// Bluechi API endpoint
    endpoint: String,
    /// D-Bus client
    client: DBusClient,
    /// Controller address
    controller_address: String,
}

impl BluechiAdapter {
    /// Create new Bluechi adapter
    ///
    /// ### Parameters
    /// * `config: &BluechiConfig` - Bluechi configuration
    /// ### Returns
    /// * `common::Result<Self>` - Created Bluechi adapter
    pub async fn new(config: &BluechiConfig) -> common::Result<Self> {
        // Implementation details
    }
    
    /// Connect to Bluechi D-Bus API
    ///
    /// ### Returns
    /// * `common::Result<()>` - Connection result
    async fn connect_dbus(&self) -> common::Result<()> {
        // Implementation details
    }
}

#[async_trait]
impl RuntimeAdapter for BluechiAdapter {
    /// Create container using Bluechi runtime
    async fn create_container(&self, config: &ContainerConfig) -> common::Result<ContainerInstance> {
        // Implementation details
    }
    
    /// Start container with Bluechi runtime
    async fn start_container(&self, container_id: &str) -> common::Result<()> {
        // Implementation details
    }
    
    /// Stop container with Bluechi runtime
    async fn stop_container(&self, container_id: &str, options: &StopOptions) -> common::Result<()> {
        // Implementation details
    }
    
    /// Remove container with Bluechi runtime
    async fn remove_container(&self, container_id: &str, force: bool) -> common::Result<()> {
        // Implementation details
    }
    
    /// Get Bluechi runtime capabilities
    fn capabilities(&self) -> RuntimeCapabilities {
        RuntimeCapabilities {
            supports_realtime_scheduling: true,
            supports_node_isolation: true,
            supports_service_management: true,
            supports_remote_execution: true,
            supports_dependencies: true,
            // Additional capability details
        }
    }
}
```

### `runtime/bluechi/service.rs`

```rust
/// Start Bluechi controller service
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// ### Returns
/// * `common::Result<()>` - Start result
pub async fn start_controller(adapter: &BluechiAdapter) -> common::Result<()> {
    // Implementation details
}

/// Start Bluechi agent service
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// * `node_name: &str` - Agent node name
/// ### Returns
/// * `common::Result<()>` - Start result
pub async fn start_agent(adapter: &BluechiAdapter, node_name: &str) -> common::Result<()> {
    // Implementation details
}

/// Stop Bluechi service
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// * `service_type: BluechiServiceType` - Type of service to stop
/// ### Returns
/// * `common::Result<()>` - Stop result
pub async fn stop_service(adapter: &BluechiAdapter, service_type: BluechiServiceType) -> common::Result<()> {
    // Implementation details
}

/// Get Bluechi service status
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// * `service_type: BluechiServiceType` - Type of service
/// ### Returns
/// * `common::Result<ServiceStatus>` - Service status or error
pub async fn get_service_status(adapter: &BluechiAdapter, service_type: BluechiServiceType) -> common::Result<ServiceStatus> {
    // Implementation details
}

/// Manage Bluechi service lifecycle
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// * `command: ServiceCommand` - Service command to execute
/// ### Returns
/// * `common::Result<CommandResult>` - Command result or error
pub async fn manage_service(adapter: &BluechiAdapter, command: ServiceCommand) -> common::Result<CommandResult> {
    // Implementation details
}

/// Monitor Bluechi services
///
/// ### Parameters
/// * `adapter: &BluechiAdapter` - Bluechi adapter
/// ### Returns
/// * `common::Result<ServiceMonitor>` - Service monitor or error
pub async fn monitor_services(adapter: &BluechiAdapter) -> common::Result<ServiceMonitor> {
    // Implementation details
}
```
