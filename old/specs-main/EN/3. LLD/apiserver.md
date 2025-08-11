# API Server

## 1. Introduction

### Major features

API Server provides internal and external APIs and performs Piccolo Artifact registration and preparation tasks.

1. Provides REST API to communicate with Piccolo Cloud or receive artifacts directly.
2. Parses and validates the received YAML format artifacts.
3. Converts them into common module structures such as scenario, package, model, etc.
4. Uses the common ETCD library to store the converted artifacts and provides real-time monitoring (watch) functionality.
5. Transmits new artifact information to FilterGateway via gRPC.
6. Notifies subscribers of artifact registration, changes, and deletions through a real-time notification system.
7. Ensures secure API access through security context and permission verification.
8. Tracks system usage through API call logging and monitoring.
9. Provides asynchronous processing mechanism for handling large-scale artifacts.

### Main Dataflow

1. Receives artifacts from external systems (Piccolo Cloud or direct requests) through REST API using various HTTP methods.
2. Validates the received artifacts and verifies if they have an allowed security context.
3. Parses YAML format artifacts by type and converts them into common data model structures.
4. Uses the common ETCD library to store the converted artifacts and monitor changes.
5. When a new scenario is registered, it is transmitted to FilterGateway via gRPC.
6. Sends real-time notifications to systems subscribing to stored artifact information.
7. Transfers new or updated artifact information to NodeAgent to perform necessary node operations.
8. Logs API calls and processing results and transmits statistics to the monitoring system.
9. Utilizes asynchronous queues for large-scale artifact processing to ensure system stability.

## 2. File information

```text
apiserver
├── apiserver.md
├── Cargo.toml
└── src
    ├── artifact
    │   ├── data.rs
    │   ├── mod.rs
    │   ├── parser.rs
    │   ├── validation.rs
    │   └── watcher.rs
    ├── grpc
    │   ├── mod.rs
    │   └── sender
    │       ├── filtergateway.rs
    │       ├── mod.rs
    │       ├── nodeagent.rs
    │       └── notification.rs
    ├── main.rs
    ├── manager.rs
    ├── metrics
    │   ├── collector.rs
    │   └── mod.rs
    ├── route
    │   ├── api.rs
    │   ├── auth.rs
    │   ├── mod.rs
    │   └── middleware.rs
    ├── security
    │   ├── context.rs
    │   ├── mod.rs
    │   └── permissions.rs
    └── storage
        ├── etcd_client.rs
        └── mod.rs
```

- **main.rs** - API Server initialization and execution
- **manager.rs** - Control of data flow and coordination between modules
- **artifact/mod.rs** - Artifact management and coordination
- **artifact/data.rs** - Implementation of functions to store or retrieve parsed results using common ETCD library
- **artifact/parser.rs** - Parse and convert YAML format strings to structures
- **artifact/validation.rs** - Artifact validation and verification
- **artifact/watcher.rs** - Artifact change monitoring and notification through ETCD
- **grpc/mod.rs** - gRPC communication management
- **grpc/sender/mod.rs** - gRPC message transmission management
- **grpc/sender/nodeagent.rs** - gRPC communication with NodeAgent
- **grpc/sender/filtergateway.rs** - gRPC communication with FilterGateway
- **grpc/sender/notification.rs** - gRPC communication for notification service
- **metrics/mod.rs** - Metrics management and coordination
- **metrics/collector.rs** - API call and performance metrics collection
- **route/mod.rs** - REST API routing management
- **route/api.rs** - REST API handler functions
- **route/auth.rs** - API authentication and authorization
- **route/middleware.rs** - API middleware (logging, authentication, metrics, etc.)
- **security/mod.rs** - Security management
- **security/context.rs** - Security context and session management
- **security/permissions.rs** - API permissions and access control
- **storage/mod.rs** - Storage management
- **storage/etcd_client.rs** - Data storage and retrieval using common ETCD library

## 3. Function information

The function information in this section is written in the comment format used in rustdoc.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo API Server
///
/// ### Description
/// Initializes logging, configuration, and starts the API Server service.
/// Sets up signal handlers for graceful shutdown on SIGINT and SIGTERM signals.
/// Configures and launches the REST API server and other supporting services.
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize API server components and start REST API server
///
/// ### Description
/// Sets up security context, artifact processors, storage connections,
/// metrics collectors, and other core services.
/// Configures CORS, authentication, and API middleware.
pub async fn initialize() {}

/// Start all API server services and background tasks
///
/// ### Returns
/// * `common::Result<()>` - Result of services startup
/// ### Description
/// Launches REST API listener, ETCD watchers, metric collectors,
/// and other required background services.
pub async fn start_services() -> common::Result<()> {}

/// Stop all API server services and release resources
///
/// ### Description
/// Performs graceful shutdown of all services, cancels background tasks,
/// and ensures proper cleanup of resources.
pub async fn stop_services() {}

/// Reload all scenario data using common ETCD library
///
/// ### Description
/// Reads all stored scenarios from ETCD using the common library.
/// Sets up watchers for artifact changes.
/// This function is called once when the API server starts.
async fn reload() {}

/// Apply downloaded artifact
///
/// ### Parameters
/// * `body: &str` - YAML string of Piccolo artifact
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<()>` - Result of artifact application
/// ### Description
/// Validates, parses and stores artifact using common ETCD library.
/// Notifies NodeAgent about new artifact via gRPC.
/// Sends notifications to FilterGateway via gRPC.
/// Records metrics for the operation.
pub async fn apply_artifact(
    body: &str,
    security_context: Option<&SecurityContext>
) -> common::Result<()> {}

/// Withdraw downloaded artifact
///
/// ### Parameters
/// * `body: &str` - YAML string of Piccolo artifact
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<()>` - Result of artifact withdrawal
/// ### Description
/// Validates permissions, then deletes the artifact using common ETCD library.
/// Notifies NodeAgent about artifact removal via gRPC.
/// Notifies FilterGateway of artifact removal via gRPC.
/// Records metrics for the operation.
pub async fn withdraw_artifact(
    body: &str,
    security_context: Option<&SecurityContext>
) -> common::Result<()> {}

/// Update existing artifact
///
/// ### Parameters
/// * `artifact_name: &str` - Name of the artifact to update
/// * `body: &str` - Updated YAML string
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<()>` - Result of artifact update
/// ### Description
/// Validates update permissions, then updates the artifact using common ETCD library.
/// Notifies NodeAgent about artifact updates via gRPC.
/// Notifies FilterGateway of artifact changes via gRPC.
pub async fn update_artifact(
    artifact_name: &str,
    body: &str,
    security_context: Option<&SecurityContext>
) -> common::Result<()> {}
```

### `artifact/mod.rs`

```rust
/// Apply downloaded artifact using common ETCD library
///
/// ### Parameters
/// * `body: &str` - YAML string of Piccolo artifact
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<(String, String)>` - Scenario and package YAML in downloaded artifact
/// ### Description
/// Validates and stores artifact using common ETCD library with proper versioning.
/// Performs security checks if security context is provided.
pub async fn apply(
    body: &str, 
    security_context: Option<&SecurityContext>
) -> common::Result<(String, String)> {}

/// Update existing artifact using common ETCD library
///
/// ### Parameters
/// * `artifact_name: &str` - Name of the artifact to update
/// * `body: &str` - Updated YAML string
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<(String, String)>` - Updated scenario and package YAML
/// ### Description
/// Validates and updates existing artifact using common ETCD library.
/// Performs security checks and maintains versioning history.
pub async fn update(
    artifact_name: &str,
    body: &str,
    security_context: Option<&SecurityContext>
) -> common::Result<(String, String)> {}

/// Delete downloaded artifact using common ETCD library
///
/// ### Parameters
/// * `body: &str` - YAML string of Piccolo artifact
/// * `security_context: Option<&SecurityContext>` - Optional security context for validation
/// ### Returns
/// * `common::Result<String>` - Scenario YAML in downloaded artifact
/// ### Description
/// Removes scenario YAML only, as other scenarios might use the same package.
/// Performs security checks if security context is provided.
pub async fn withdraw(
    body: &str, 
    security_context: Option<&SecurityContext>
) -> common::Result<String> {}
```

### `artifact/data.rs`

```rust
/// Read YAML string of artifacts using common ETCD library
///
/// ### Parameters
/// * `artifact_name: &str` - Name of the artifact to read
/// ### Returns
/// * `common::Result<String>` - YAML string of the artifact
/// ### Description
/// Retrieves artifact data using common ETCD library with automatic retries.
pub async fn read_from_etcd(artifact_name: &str) -> common::Result<String> {}

/// Read all scenario YAML strings using common ETCD library
///
/// ### Parameters
/// * None
/// ### Returns
/// * `common::Result<Vec<String>>` - Vector of scenario YAML strings
/// ### Description
/// Retrieves all scenario data using common ETCD library with pagination support.
pub async fn read_all_scenario_from_etcd() -> common::Result<Vec<String>> {}

/// Write YAML string of artifacts using common ETCD library
///
/// ### Parameters
/// * `key: &str` - ETCD key for the artifact
/// * `artifact_str: &str` - YAML string to store
/// ### Returns
/// * `common::Result<()>` - Result of write operation
/// ### Description
/// Stores artifact data using common ETCD library with versioning support.
pub async fn write_to_etcd(key: &str, artifact_str: &str) -> common::Result<()> {}

/// Delete YAML string of artifacts using common ETCD library
///
/// ### Parameters
/// * `key: &str` - ETCD key to delete
/// ### Returns
/// * `common::Result<()>` - Result of delete operation
/// ### Description
/// Removes artifact data using common ETCD library with proper cleanup.
pub async fn delete_at_etcd(key: &str) -> common::Result<()> {}

/// Watch for changes in artifacts using common ETCD library
///
/// ### Parameters
/// * `prefix: &str` - Key prefix to watch
/// * `handler: F` - Handler function for change events
/// ### Returns
/// * `common::Result<WatchHandle>` - Handle to manage the watcher
/// ### Description
/// Sets up a watcher for changes to artifacts with the given prefix.
pub async fn watch_artifacts<F>(prefix: &str, handler: F) -> common::Result<WatchHandle>
where
    F: Fn(WatchEvent<String>) + Send + Sync + 'static,
{
    // Implementation using common ETCD library
}

/// Get artifact version history using common ETCD library
///
/// ### Parameters
/// * `artifact_name: &str` - Name of the artifact
/// ### Returns
/// * `common::Result<Vec<ArtifactVersion>>` - Version history
/// ### Description
/// Retrieves the version history of an artifact using common ETCD library.
pub async fn get_artifact_versions(artifact_name: &str) -> common::Result<Vec<ArtifactVersion>> {}
```

### `artifact/parser.rs`

```rust
/// Parse YAML string into artifact structure
///
/// ### Parameters
/// * `yaml_str: &str` - YAML string to parse
/// ### Returns
/// * `common::Result<ArtifactType>` - Parsed artifact
/// ### Description
/// Parses YAML string into appropriate artifact structure based on content.
pub fn parse_yaml(yaml_str: &str) -> common::Result<ArtifactType> {}

/// Serialize artifact structure to YAML string
///
/// ### Parameters
/// * `artifact: &ArtifactType` - Artifact to serialize
/// ### Returns
/// * `common::Result<String>` - YAML string
/// ### Description
/// Converts artifact structure to YAML string with proper formatting.
pub fn serialize_to_yaml(artifact: &ArtifactType) -> common::Result<String> {}

/// Detect artifact type from YAML content
///
/// ### Parameters
/// * `yaml_str: &str` - YAML string to analyze
/// ### Returns
/// * `ArtifactKind` - Type of artifact detected
/// ### Description
/// Determines the kind of artifact from YAML content for proper parsing.
pub fn detect_artifact_type(yaml_str: &str) -> ArtifactKind {}
```

### `artifact/validation.rs`

```rust
/// Validate artifact schema against defined rules
///
/// ### Parameters
/// * `artifact: &ArtifactType` - Artifact to validate
/// ### Returns
/// * `common::Result<ValidationResult>` - Validation results
/// ### Description
/// Checks if artifact structure conforms to schema requirements.
pub fn validate_schema(artifact: &ArtifactType) -> common::Result<ValidationResult> {}

/// Validate artifact semantics and relationships
///
/// ### Parameters
/// * `artifact: &ArtifactType` - Artifact to validate
/// ### Returns
/// * `common::Result<ValidationResult>` - Validation results
/// ### Description
/// Verifies semantic correctness and relationship integrity in artifact.
pub fn validate_semantics(artifact: &ArtifactType) -> common::Result<ValidationResult> {}

/// Validate security permissions for artifact operation
///
/// ### Parameters
/// * `artifact: &ArtifactType` - Target artifact
/// * `operation: ArtifactOperation` - Operation being performed
/// * `context: &SecurityContext` - Security context for validation
/// ### Returns
/// * `common::Result<bool>` - Whether operation is permitted
/// ### Description
/// Checks if the security context allows the operation on the artifact.
pub fn validate_permissions(
    artifact: &ArtifactType,
    operation: ArtifactOperation,
    context: &SecurityContext
) -> common::Result<bool> {}
```

### `artifact/watcher.rs`

```rust
/// Start watching artifacts in ETCD
///
/// ### Parameters
/// * `config: &WatchConfig` - Configuration for watch behavior
/// ### Returns
/// * `common::Result<WatchManager>` - Manager for artifact watches
/// ### Description
/// Sets up watchers for artifacts with notification capabilities.
pub async fn start_watching(config: &WatchConfig) -> common::Result<WatchManager> {}

/// Register notification handler for artifact changes
///
/// ### Parameters
/// * `artifact_type: ArtifactKind` - Type of artifact to watch
/// * `handler: F` - Handler function for changes
/// ### Returns
/// * `common::Result<SubscriptionId>` - ID for managing subscription
/// ### Description
/// Registers a handler function to be called when artifacts change.
pub fn register_notification_handler<F>(
    artifact_type: ArtifactKind,
    handler: F
) -> common::Result<SubscriptionId>
where
    F: Fn(ArtifactChangeEvent) + Send + Sync + 'static,
{}

/// Get artifact change history
///
/// ### Parameters
/// * `artifact_name: &str` - Name of the artifact
/// * `limit: Option<usize>` - Maximum number of changes to retrieve
/// ### Returns
/// * `common::Result<Vec<ChangeRecord>>` - Change history
/// ### Description
/// Retrieves the change history for an artifact with optional limit.
pub async fn get_change_history(
    artifact_name: &str,
    limit: Option<usize>
) -> common::Result<Vec<ChangeRecord>> {}
```

### `route/mod.rs`

```rust
/// Serve Piccolo HTTP API service
///
/// ### Parameters
/// * `config: &ServerConfig` - Server configuration options
/// ### Returns
/// * `common::Result<ServerHandle>` - Handle to the server
/// ### Description
/// Configures and starts the HTTP API server with CORS, authentication,
/// and other middleware. Returns a handle for managing the server.
pub async fn launch_tcp_listener(config: &ServerConfig) -> common::Result<ServerHandle> {}

/// Generate appropriate API response based on handler execution result
///
/// ### Parameters
/// * `result: common::Result<T>` - Result of API handler logic
/// * `ctx: &RequestContext` - Request context information
/// ### Returns
/// * `Response` - Properly formatted API response
/// ### Description
/// Creates a standardized API response with appropriate status codes,
/// headers, and response format based on result and context.
pub fn create_response<T: Serialize>(result: common::Result<T>, ctx: &RequestContext) -> Response {}

/// Setup API middleware stack
///
/// ### Parameters
/// * `config: &MiddlewareConfig` - Middleware configuration
/// ### Returns
/// * `MiddlewareStack` - Configured middleware stack
/// ### Description
/// Creates a middleware stack with logging, authentication, metrics,
/// and other request processing components.
pub fn setup_middleware(config: &MiddlewareConfig) -> MiddlewareStack {}
```

### `route/api.rs`

```rust
/// Create router for Piccolo API service
///
/// ### Parameters
/// * `auth_service: AuthService` - Authentication service
/// * `metrics_collector: MetricsCollector` - Metrics collection service
/// ### Returns
/// * `Router` - Configured API router
/// ### Description
/// Creates a router with all API endpoints, middleware, and services.
pub fn router(auth_service: AuthService, metrics_collector: MetricsCollector) -> Router {}

/// Notify of new artifact release in the cloud
///
/// ### Parameters
/// * `Path(artifact_name): Path<String>` - Name of the artifact
/// * `Extension(security_ctx): Extension<SecurityContext>` - Security context
/// ### Returns
/// * `Response` - API response
/// ### Description
/// Handles notification of artifact availability in cloud storage.
/// Validates security context before processing.
async fn notify(Path(artifact_name): Path<String>, Extension(security_ctx): Extension<SecurityContext>) -> Response {}

/// Apply new artifacts (scenario, package, etc...)
///
/// ### Parameters
/// * `body: String` - YAML format artifact content
/// * `Extension(security_ctx): Extension<SecurityContext>` - Security context
/// ### Returns
/// * `Response` - API response
/// ### Description
/// Processes and stores new artifacts with security validation.
/// Records metrics and sends notifications of changes.
async fn apply_artifact(body: String, Extension(security_ctx): Extension<SecurityContext>) -> Response {}

/// Update existing artifact
///
/// ### Parameters
/// * `Path(artifact_name): Path<String>` - Name of the artifact to update
/// * `body: String` - Updated YAML content
/// * `Extension(security_ctx): Extension<SecurityContext>` - Security context
/// ### Returns
/// * `Response` - API response
/// ### Description
/// Updates an existing artifact with new content after validation.
async fn update_artifact(
    Path(artifact_name): Path<String>,
    body: String,
    Extension(security_ctx): Extension<SecurityContext>
) -> Response {}

/// Withdraw applied scenario
///
/// ### Parameters
/// * `body: String` - YAML of artifact to delete
/// * `Extension(security_ctx): Extension<SecurityContext>` - Security context
/// ### Returns
/// * `Response` - API response
/// ### Description
/// Removes an artifact after security validation.
/// Sends notifications of removal to subscribed services.
async fn withdraw_artifact(body: String, Extension(security_ctx): Extension<SecurityContext>) -> Response {}

/// List all artifacts of specified type
///
/// ### Parameters
/// * `Query(params): Query<ListParams>` - Query parameters
/// * `Extension(security_ctx): Extension<SecurityContext>` - Security context
/// ### Returns
/// * `Response` - API response with artifact list
/// ### Description
/// Returns a list of artifacts filtered by type and other criteria.
async fn list_artifacts(
    Query(params): Query<ListParams>,
    Extension(security_ctx): Extension<SecurityContext>
) -> Response {}
```

### `route/auth.rs`

```rust
/// Authenticate API request
///
/// ### Parameters
/// * `req: &Request` - Incoming HTTP request
/// ### Returns
/// * `common::Result<SecurityContext>` - Security context if authenticated
/// ### Description
/// Validates authentication tokens and creates security context.
pub async fn authenticate(req: &Request) -> common::Result<SecurityContext> {}

/// Authorize API operation
///
/// ### Parameters
/// * `context: &SecurityContext` - Security context
/// * `operation: ApiOperation` - Requested operation
/// ### Returns
/// * `common::Result<bool>` - Whether operation is authorized
/// ### Description
/// Checks if the security context permits the requested operation.
pub async fn authorize(context: &SecurityContext, operation: ApiOperation) -> common::Result<bool> {}

/// Create authentication middleware
///
/// ### Parameters
/// * `config: &AuthConfig` - Authentication configuration
/// ### Returns
/// * `AuthMiddleware` - Authentication middleware
/// ### Description
/// Creates middleware for request authentication and authorization.
pub fn create_auth_middleware(config: &AuthConfig) -> AuthMiddleware {}
```

### `route/middleware.rs`

```rust
/// Create logging middleware
///
/// ### Parameters
/// * `config: &LoggingConfig` - Logging configuration
/// ### Returns
/// * `LoggingMiddleware` - Logging middleware
/// ### Description
/// Creates middleware for API request and response logging.
pub fn create_logging_middleware(config: &LoggingConfig) -> LoggingMiddleware {}

/// Create metrics middleware
///
/// ### Parameters
/// * `collector: MetricsCollector` - Metrics collector
/// ### Returns
/// * `MetricsMiddleware` - Metrics middleware
/// ### Description
/// Creates middleware for collecting API metrics.
pub fn create_metrics_middleware(collector: MetricsCollector) -> MetricsMiddleware {}

/// Create CORS middleware
///
/// ### Parameters
/// * `config: &CorsConfig` - CORS configuration
/// ### Returns
/// * `CorsMiddleware` - CORS middleware
/// ### Description
/// Creates middleware for Cross-Origin Resource Sharing.
pub fn create_cors_middleware(config: &CorsConfig) -> CorsMiddleware {}

/// Apply middleware to router
///
/// ### Parameters
/// * `router: Router` - API router
/// * `middleware: &[Middleware]` - Middleware to apply
/// ### Returns
/// * `Router` - Router with middleware
/// ### Description
/// Applies multiple middleware components to the API router.
pub fn apply_middleware(router: Router, middleware: &[Middleware]) -> Router {}
```

### `grpc/sender/filtergateway.rs`

```rust
/// Send scenario information to FilterGateway via gRPC
///
/// ### Parameters
/// * `scenario: HandleScenarioRequest` - Scenario information
/// * `metadata: Option<Metadata>` - Optional request metadata
/// ### Returns
/// * `Result<Response<HandleScenarioResponse>, Status>` - Response from FilterGateway
/// ### Description
/// Sends scenario information to FilterGateway using the common gRPC client
/// Handles connection management and retries automatically
/// Includes security context and tracing information when available
pub async fn send_scenario(
    scenario: HandleScenarioRequest,
    metadata: Option<Metadata>
) -> Result<Response<HandleScenarioResponse>, Status> {
    let mut client = FilterGatewayConnectionClient::connect(connect_server())
        .await?;
    
    let request = if let Some(md) = metadata {
        Request::from_parts(md, scenario)
    } else {
        Request::new(scenario)
    };
    
    client.handle_scenario(request).await
}

/// Send batch scenario operations to FilterGateway
///
/// ### Parameters
/// * `batch: BatchScenarioRequest` - Batch of scenario operations
/// ### Returns
/// * `Result<Response<BatchScenarioResponse>, Status>` - Batch operation results
/// ### Description
/// Sends multiple scenario operations in a single request
/// Optimized for performance with large batches
pub async fn send_batch(
    batch: BatchScenarioRequest
) -> Result<Response<BatchScenarioResponse>, Status> {
    let mut client = FilterGatewayConnectionClient::connect(connect_server())
        .await?;
    client.handle_batch_scenario(Request::new(batch)).await
}

/// Check FilterGateway connection health
///
/// ### Returns
/// * `bool` - Whether connection is healthy
/// ### Description
/// Verifies connection to FilterGateway is working properly
pub async fn check_connection_health() -> bool {
    if let Ok(mut client) = FilterGatewayConnectionClient::connect(connect_server()).await {
        client.health_check(Request::new(HealthCheckRequest {})).await.is_ok()
    } else {
        false
    }
}
```

### `grpc/sender/notification.rs`

```rust
/// Send artifact change notification
///
/// ### Parameters
/// * `notification: ArtifactChangeNotification` - Change notification
/// ### Returns
/// * `Result<Response<NotificationResponse>, Status>` - Notification result
/// ### Description
/// Notifies subscribers about artifact changes
pub async fn send_change_notification(
    notification: ArtifactChangeNotification
) -> Result<Response<NotificationResponse>, Status> {
    let mut client = NotificationClient::connect(get_notification_server())
        .await?;
    client.notify_artifact_change(Request::new(notification)).await
}

/// Register artifact change subscriber
///
/// ### Parameters
/// * `subscriber: SubscriberInfo` - Subscriber information
/// ### Returns
/// * `Result<Response<SubscriptionResponse>, Status>` - Subscription result
/// ### Description
/// Registers a new subscriber for artifact change notifications
pub async fn register_subscriber(
    subscriber: SubscriberInfo
) -> Result<Response<SubscriptionResponse>, Status> {
    let mut client = NotificationClient::connect(get_notification_server())
        .await?;
    client.register_subscriber(Request::new(subscriber)).await
}

/// Unregister artifact change subscriber
///
/// ### Parameters
/// * `subscriber_id: String` - Subscriber ID
/// ### Returns
/// * `Result<Response<UnsubscriptionResponse>, Status>` - Unsubscription result
/// ### Description
/// Removes a subscriber from the notification list
pub async fn unregister_subscriber(
    subscriber_id: String
) -> Result<Response<UnsubscriptionResponse>, Status> {
    let mut client = NotificationClient::connect(get_notification_server())
        .await?;
    client.unregister_subscriber(Request::new(SubscriberId { id: subscriber_id })).await
}
```

### `grpc/sender/nodeagent.rs`

```rust
/// Send artifact information to NodeAgent via gRPC
///
/// ### Parameters
/// * `artifact: ArtifactInfo` - Artifact information
/// * `metadata: Option<Metadata>` - Optional request metadata
/// ### Returns
/// * `Result<Response<ArtifactResponse>, Status>` - Response from NodeAgent
/// ### Description
/// Sends artifact information to NodeAgent using the gRPC client
/// Handles connection management and retries automatically
/// Includes security context and tracing information when available
pub async fn send_artifact(
    artifact: ArtifactInfo,
    metadata: Option<Metadata>
) -> Result<Response<ArtifactResponse>, Status> {
    let mut client = NodeAgentClient::connect(connect_nodeagent())
        .await?;
    
    let request = if let Some(md) = metadata {
        Request::from_parts(md, artifact)
    } else {
        Request::new(artifact)
    };
    
    client.handle_artifact(request).await
}

/// Notify NodeAgent of artifact removal
///
/// ### Parameters
/// * `artifact_id: String` - ID of the artifact to remove
/// ### Returns
/// * `Result<Response<RemoveResponse>, Status>` - Response from NodeAgent
/// ### Description
/// Notifies NodeAgent that an artifact has been removed
pub async fn notify_artifact_removal(
    artifact_id: String
) -> Result<Response<RemoveResponse>, Status> {
    let mut client = NodeAgentClient::connect(connect_nodeagent())
        .await?;
    client.remove_artifact(Request::new(RemoveRequest { artifact_id })).await
}

/// Check NodeAgent connection health
///
/// ### Returns
/// * `bool` - Whether connection is healthy
/// ### Description
/// Verifies connection to NodeAgent is working properly
pub async fn check_nodeagent_connection() -> bool {
    if let Ok(mut client) = NodeAgentClient::connect(connect_nodeagent()).await {
        client.health_check(Request::new(HealthCheckRequest {})).await.is_ok()
    } else {
        false
    }
}
```
