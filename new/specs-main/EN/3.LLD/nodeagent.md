# Node Agent

## 1. Introduction

### Major features

Node Agent is a component in the PICCOLO framework that monitors and manages the state of each node. It is responsible for collecting all monitoring information within the system.

1. Collects Pod/container metrics to provide information necessary for state management.
   - Computing resources (CPU usage, throttling, scheduling delays, etc.)
   - Memory resources (usage, working set, OOM events, etc.)
   - Storage and I/O (filesystem usage, disk operations, etc.)
   - Network (received/transmitted bytes, errors, packet drops, etc.)
   - Process and runtime metrics (number of processes, restart count, etc.)
   - Application-level metrics (response time, error rate, etc.)
2. Collects hardware system information.
   - CPU, memory, disk, network hardware status (managed in integrated resource.rs)
   - Hardware health indicators such as temperature, voltage, etc.
3. Monitors and reports node availability.
4. Handles configuration file distribution and setting updates.
5. Performs Bluechi pre-configuration and environment setup tasks.
   - Creates and distributes Bluechi configuration files
   - Configures node environments and sets up necessary directories/files
   - Sets file permissions and default settings
6. Sends node and container state information to StateManager.
7. Sends all metrics and log data to MonitoringServer.
8. Monitors network connection status and service availability.
   - Performs health checks on core services
   - Tests network connectivity and measures quality
   - Runs automatic diagnostics when network issues occur

### Main Dataflow

1. Collects container and system metrics according to defined sampling cycles.
   - Critical metrics are collected every 1 second (CPU/memory usage, OOM events, etc.)
   - Less critical metrics are collected at 5 or 10-second intervals
   - The integrated resource.rs file handles collection of CPU, memory, disk, and network metrics
2. Filters and normalizes collected metrics and converts them into state events.
3. Compares the actual state with the desired state (loaded from ETCD) to calculate state differences (diff).
4. Sends the calculated state differences and critical events (OOM kills, container terminations, etc.) to StateManager.
5. Sends all monitoring metrics to MonitoringServer for storage in ETCD and analysis.
6. Sends log data, separated into general logs and stream logs, to MonitoringServer.
7. Receives and responds to state-related requests from StateManager.
8. Receives and applies configuration files and setting updates from API Server.
9. Performs Bluechi pre-configuration tasks.
   - Receives necessary node information and configuration parameters from API Server
   - Generates Bluechi cluster and agent configuration files
   - Deploys the generated configuration files to appropriate locations and sets up necessary directories and permissions
   - Reports the Bluechi basic configuration status to API Server
10. Performs regular network status checks and sends results to MonitoringServer.
    - Executes health checks on important services
    - Collects network quality indicators such as latency, packet loss rate, etc.
    - Diagnoses network issues and attempts recovery

## 2. File information

```text
nodeagent
├── nodeagent.md
├── Cargo.toml
└── src
    ├── config
    │   ├── loader.rs
    │   ├── mod.rs
    │   └── validator.rs
    ├── grpc
    │   ├── client
    │   │   ├── state_manager.rs
    │   │   ├── api_server.rs
    │   │   ├── mod.rs
    │   │   └── monitoring_server.rs
    │   ├── mod.rs
    │   └── server
    │       ├── mod.rs
    │       └── service.rs
    ├── metrics
    │   ├── collector
    │   │   └── mod.rs
    │   ├── mod.rs
    │   └── model.rs
    ├── hardware
    │   ├── mod.rs
    │   ├── monitor.rs
    │   └── health.rs
    ├── container
    │   ├── mod.rs
    │   ├── runtime.rs
    │   ├── status.rs
    │   └── events.rs
    ├── diff
    │   ├── mod.rs
    │   ├── calculator.rs
    │   ├── model.rs
    │   └── reporter.rs
    ├── resource.rs
    ├── bluechi
    │   ├── mod.rs
    │   ├── config_generator.rs
    │   ├── file_deployer.rs
    │   └── models.rs
    ├── event
    │   ├── mod.rs
    │   ├── filter.rs
    │   └── dispatcher.rs
    ├── main.rs
    ├── manager.rs
    ├── resource
    │   ├── cleaner.rs
    │   ├── disk.rs
    │   └── mod.rs
    ├── sync
    │   ├── distributor.rs
    │   ├── mod.rs
    │   └── receiver.rs

```

- **main.rs** - NodeAgent initialization and execution
- **manager.rs** - Controls data flow between modules
- **config/mod.rs** - Configuration related functionality module
- **config/loader.rs** - Configuration file loader
- **config/validator.rs** - Configuration validator
- **grpc/mod.rs** - gRPC communication management
- **grpc/client/mod.rs** - gRPC client management
- **grpc/client/state_manager.rs** - gRPC communication with StateManager
- **grpc/client/api_server.rs** - gRPC communication with APIServer
- **grpc/client/monitoring_server.rs** - gRPC communication with MonitoringServer
- **grpc/server/mod.rs** - gRPC server management
- **grpc/server/service.rs** - NodeAgent gRPC service implementation
- **metrics/mod.rs** - Metrics collection functionality module
- **metrics/model.rs** - Metrics data model definition
- **metrics/collector/mod.rs** - Metrics collector functionality module (see resource.rs)
- **hardware/mod.rs** - Hardware related functionality module (see resource.rs)
- **hardware/monitor.rs** - Hardware monitoring functionality
- **hardware/health.rs** - Hardware health indicator collection
- **resource.rs** - Integrated resource-related functionality module (CPU, memory, disk, network metrics collection and hardware status monitoring)
- **container/mod.rs** - Container related functionality module
- **container/runtime.rs** - Container runtime integration
- **container/status.rs** - Container status management
- **container/events.rs** - Container event handling
- **diff/mod.rs** - State difference (diff) management module
- **diff/calculator.rs** - Calculates differences between actual state and desired state
- **diff/model.rs** - State difference model definition
- **diff/reporter.rs** - Sends state difference information to StateManager
- **event/mod.rs** - Event handling functionality module
- **event/filter.rs** - Event filtering
- **event/dispatcher.rs** - Event dispatcher
- **bluechi/mod.rs** - Bluechi pre-configuration functionality module
- **bluechi/config_generator.rs** - Bluechi configuration file generator
- **bluechi/file_deployer.rs** - Bluechi file deployer
- **bluechi/models.rs** - Bluechi configuration model definition
- **resource/mod.rs** - Resource management
- **resource/cleaner.rs** - Temporary file and cache cleanup
- **resource/disk.rs** - Disk space management
- **sync/mod.rs** - Node synchronization
- **sync/distributor.rs** - Configuration distribution
- **sync/receiver.rs** - Configuration reception and application

## 3. Function information

The function information in this section is written in the format used by rustdoc comments.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo Node Agent
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize and start the NodeAgent
pub async fn initialize() {}

/// Start all node agent services
pub async fn start_services() {}

/// Stop all node agent services
pub async fn stop_services() {}

/// Collect and process metrics
///
/// ### Returns
/// * `Result<CollectedMetrics, Error>` - Collected metrics or error
pub async fn collect_metrics() -> Result<CollectedMetrics, Error> {}

/// Collect and process logs
///
/// ### Returns
/// * `Result<CollectedLogs, Error>` - Collected logs or error
pub async fn collect_logs() -> Result<CollectedLogs, Error> {}

/// Send metrics to MonitoringServer
///
/// ### Parameters
/// * `metrics: &CollectedMetrics` - Metrics to send
/// ### Returns
/// * `Result<(), Error>` - Send result
pub async fn send_metrics_to_monitoring(&self, metrics: &CollectedMetrics) -> Result<(), Error> {}

/// Send logs to MonitoringServer
///
/// ### Parameters
/// * `logs: &CollectedLogs` - Logs to send
/// ### Returns
/// * `Result<(), Error>` - Send result
pub async fn send_logs_to_monitoring(&self, logs: &CollectedLogs) -> Result<(), Error> {}

/// Send state update to StateManager
///
/// ### Parameters
/// * `update: &StateUpdate` - State update to send
/// ### Returns
/// * `Result<(), Error>` - Send result
pub async fn send_state_update(&self, update: &StateUpdate) -> Result<(), Error> {}

/// Stream container logs to MonitoringServer
///
/// ### Parameters
/// * `container_id: &str` - Container ID
/// ### Returns
/// * `Result<(), Error>` - Stream setup result
pub async fn stream_container_logs(&self, container_id: &str) -> Result<(), Error> {}

/// Process configuration update
///
/// ### Parameters
/// * `config: ConfigUpdate` - Configuration update
/// ### Returns
/// * `Result<(), Error>` - Processing result
pub async fn process_config_update(&self, config: ConfigUpdate) -> Result<(), Error> {}

/// Handle hardware monitoring events
///
/// ### Parameters
/// * `event: HardwareEvent` - Hardware event
/// ### Returns
/// * `Result<(), Error>` - Handling result
pub async fn handle_hardware_event(&self, event: HardwareEvent) -> Result<(), Error> {}

/// Handle container events
///
/// ### Parameters
/// * `event: ContainerEvent` - Container event
/// ### Returns
/// * `Result<(), Error>` - Handling result
pub async fn handle_container_event(&self, event: ContainerEvent) -> Result<(), Error> {}
```

### `config/loader.rs`

```rust
/// Load configuration from file
///
/// ### Parameters
/// * `path: &Path` - Path to the configuration file
/// ### Returns
/// * `Result<Config, Error>` - Parsed configuration or error
pub fn load_config(path: &Path) -> Result<Config, Error> {}

/// Parse YAML configuration
///
/// ### Parameters
/// * `yaml_str: &str` - YAML string to parse
/// ### Returns
/// * `Result<Config, Error>` - Parsed configuration or error
pub fn parse_config(yaml_str: &str) -> Result<Config, Error> {}

/// Save configuration to file
///
/// ### Parameters
/// * `config: &Config` - Configuration to save
/// * `path: &Path` - Path to save the configuration
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub fn save_config(config: &Config, path: &Path) -> Result<(), Error> {}

/// Load configuration from a remote source
///
/// ### Parameters
/// * `source: RemoteSource` - Remote configuration source
/// ### Returns
/// * `Result<Config, Error>` - Loaded configuration or error
pub async fn load_remote_config(source: RemoteSource) -> Result<Config, Error> {}
```

### `config/validator.rs`

```rust
/// Validate configuration structure
///
/// ### Parameters
/// * `config: &Config` - Configuration to validate
/// ### Returns
/// * `Result<(), ValidationError>` - Validation result
pub fn validate_config(config: &Config) -> Result<(), ValidationError> {}

/// Check configuration compatibility
///
/// ### Parameters
/// * `config: &Config` - Configuration to check
/// * `node_info: &NodeInfo` - Node information for compatibility check
/// ### Returns
/// * `bool` - True if configuration is compatible
pub fn check_compatibility(config: &Config, node_info: &NodeInfo) -> bool {}

/// Validate configuration dependencies
///
/// ### Parameters
/// * `config: &Config` - Configuration to check dependencies
/// ### Returns
/// * `Vec<DependencyIssue>` - List of dependency issues found
pub fn validate_dependencies(config: &Config) -> Vec<DependencyIssue> {}

/// Check configuration security
///
/// ### Parameters
/// * `config: &Config` - Configuration to check
/// ### Returns
/// * `SecurityReport` - Security check report
pub fn check_security(config: &Config) -> SecurityReport {}
```

### `resource.rs`

```rust
// CPU Resources

/// CPU hardware metrics structure
pub struct CpuHardwareMetrics {
    /// Overall CPU utilization percentage
    pub utilization_percent: f64,
    /// Per core CPU utilization
    pub core_utilization: Vec<f64>,
    /// User mode CPU utilization
    pub user_percent: f64,
    /// System mode CPU utilization
    pub system_percent: f64,
    /// IO wait time percentage
    pub iowait_percent: f64,
    /// IRQ handling time percentage
    pub irq_percent: f64,
    /// Soft IRQ handling time percentage
    pub softirq_percent: f64,
    /// Idle time percentage
    pub idle_percent: f64,
    /// CPU temperature
    pub temperature_celsius: Option<f64>,
    /// CPU frequency in MHz
    pub frequency_mhz: Option<f64>,
    /// Load average for 1 minute
    pub load_1m: f64,
    /// Load average for 5 minutes
    pub load_5m: f64,
    /// Load average for 15 minutes
    pub load_15m: f64,
    /// Timestamp of the metrics collection
    pub timestamp: DateTime<Utc>,
}

/// Container CPU metrics structure
pub struct CpuMetrics {
    pub usage_seconds: f64,
    pub utilization_percent: f64,
    pub throttling_count: u64,
    pub scheduling_latency_ms: u64,
    pub context_switches: u64,
    pub timestamp: DateTime<Utc>,
}

/// Monitor CPU hardware status
///
/// ### Returns
/// * `Result<CpuHardwareMetrics, Error>` - CPU hardware metrics or error
pub async fn monitor_cpu_hardware() -> Result<CpuHardwareMetrics, Error> {}

/// Collect container CPU metrics
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<CpuMetrics, Error>` - Collected CPU metrics or error
pub async fn collect_cpu_metrics(container_id: &str) -> Result<CpuMetrics, Error> {}

/// Calculate CPU utilization percentage
///
/// ### Parameters
/// * `usage: f64` - Current CPU usage
/// * `limit: f64` - CPU limit
/// ### Returns
/// * `f64` - CPU utilization percentage
pub fn calculate_cpu_utilization(usage: f64, limit: f64) -> f64 {}

// Memory Resources

/// Memory hardware metrics structure
pub struct MemoryHardwareMetrics {
    pub total_bytes: u64,
    pub free_bytes: u64,
    pub available_bytes: u64,
    pub used_bytes: u64,
    pub cached_bytes: u64,
    pub buffers_bytes: u64,
    pub swap_total_bytes: u64,
    pub swap_free_bytes: u64,
    pub utilization_percent: f64,
    pub timestamp: DateTime<Utc>,
}

/// Container memory metrics structure
pub struct MemoryMetrics {
    pub usage_bytes: u64,
    pub working_set_bytes: u64,
    pub limit_bytes: u64,
    pub cache_bytes: u64,
    pub rss_bytes: u64,
    pub swap_bytes: u64,
    pub failure_count: u64,
    pub oom_events: u64,
    pub timestamp: DateTime<Utc>,
}

/// Monitor memory hardware status
///
/// ### Returns
/// * `Result<MemoryHardwareMetrics, Error>` - Memory hardware metrics or error
pub async fn monitor_memory_hardware() -> Result<MemoryHardwareMetrics, Error> {}

/// Collect container memory metrics
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<MemoryMetrics, Error>` - Collected memory metrics or error
pub async fn collect_memory_metrics(container_id: &str) -> Result<MemoryMetrics, Error> {}

/// Calculate memory utilization percentage
///
/// ### Parameters
/// * `usage: u64` - Memory usage in bytes
/// * `limit: u64` - Memory limit in bytes
/// ### Returns
/// * `f64` - Memory utilization percentage
pub fn calculate_memory_utilization(usage: u64, limit: u64) -> f64 {}

// Storage/Disk Resources

/// Disk hardware metrics structure
pub struct DiskHardwareMetrics {
    pub mount_utilization: HashMap<String, MountPointUtilization>,
    pub io_stats: HashMap<String, DiskIoStats>,
    pub overall_utilization_percent: f64,
    pub timestamp: DateTime<Utc>,
}

/// Container storage metrics structure
pub struct StorageMetrics {
    pub fs_usage_bytes: u64,
    pub fs_limit_bytes: u64,
    pub reads_count: u64,
    pub writes_count: u64,
    pub read_bytes: u64,
    pub write_bytes: u64,
    pub io_current: u64,
    pub io_time_ms: u64,
    pub timestamp: DateTime<Utc>,
}

/// Monitor disk hardware status
///
/// ### Returns
/// * `Result<DiskHardwareMetrics, Error>` - Disk hardware metrics or error
pub async fn monitor_disk_hardware() -> Result<DiskHardwareMetrics, Error> {}

/// Collect container storage metrics
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<StorageMetrics, Error>` - Collected storage metrics or error
pub async fn collect_storage_metrics(container_id: &str) -> Result<StorageMetrics, Error> {}

/// Calculate I/O utilization
///
/// ### Parameters
/// * `io_time_ms: u64` - Time spent on I/O in milliseconds
/// * `period_ms: u64` - Period in milliseconds
/// ### Returns
/// * `f64` - I/O utilization percentage
pub fn calculate_io_utilization(io_time_ms: u64, period_ms: u64) -> f64 {}

// Network Resources

/// Network hardware metrics structure
pub struct NetworkHardwareMetrics {
    pub interfaces: HashMap<String, NetworkInterfaceStats>,
    pub overall_utilization_percent: f64,
    pub timestamp: DateTime<Utc>,
}

/// Container network metrics structure
pub struct NetworkMetrics {
    pub receive_bytes: u64,
    pub transmit_bytes: u64,
    pub receive_packets: u64,
    pub transmit_packets: u64,
    pub receive_errors: u64,
    pub transmit_errors: u64,
    pub receive_drops: u64,
    pub transmit_drops: u64,
    pub timestamp: DateTime<Utc>,
}

/// Monitor network hardware status
///
/// ### Returns
/// * `Result<NetworkHardwareMetrics, Error>` - Network hardware metrics or error
pub async fn monitor_network_hardware() -> Result<NetworkHardwareMetrics, Error> {}

/// Collect container network metrics
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<NetworkMetrics, Error>` - Collected network metrics or error
pub async fn collect_network_metrics(container_id: &str) -> Result<NetworkMetrics, Error> {}

/// Calculate network bandwidth
///
/// ### Parameters
/// * `bytes: u64` - Bytes transferred
/// * `period_ms: u64` - Period in milliseconds
/// ### Returns
/// * `f64` - Bandwidth in bytes per second
pub fn calculate_bandwidth(bytes: u64, period_ms: u64) -> f64 {}

// Resource Manager

/// Resource manager for monitoring and collecting metrics
pub struct ResourceManager {
    /// Configuration for the resource manager
    config: ResourceConfig,
    /// Hardware and container metrics cache
    metrics_cache: Arc<RwLock<MetricsCache>>,
}

/// Initialize and start the ResourceManager
///
/// ### Parameters
/// * `config: ResourceConfig` - Configuration for resource monitoring
/// ### Returns
/// * `Result<ResourceManager, Error>` - Initialized resource manager
pub fn new_resource_manager(config: ResourceConfig) -> Result<ResourceManager, Error> {}

/// Get latest hardware metrics
///
/// ### Returns
/// * `HardwareMetrics` - Combined hardware metrics
pub async fn get_hardware_metrics(&self) -> HardwareMetrics {}

/// Collect metrics for a container
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<ContainerMetrics, Error>` - Container metrics
pub async fn collect_container_metrics(&self, container_id: &str) -> Result<ContainerMetrics, Error> {}
```

// Process-related functions now included in resource.rs

### `grpc/client/api_server.rs`

```rust
/// Connect to APIServer gRPC service
///
/// ### Parameters
/// * `address: &str` - APIServer service address
/// ### Returns
/// * `ApiServerClient` - Connected client
pub async fn connect(address: &str) -> ApiServerClient {}

/// Register node with APIServer
///
/// ### Parameters
/// * `node_info: NodeInfo` - Information about this node
/// ### Returns
/// * `Result<NodeRegistration, Error>` - Registration result or error
pub async fn register_node(&self, node_info: NodeInfo) -> Result<NodeRegistration, Error> {}

/// Update node status
///
/// ### Parameters
/// * `status: NodeStatus` - Current node status
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn update_node_status(&self, status: NodeStatus) -> Result<(), Error> {}

/// Get configuration updates
///
/// ### Parameters
/// * `version: &str` - Current configuration version
/// ### Returns
/// * `Result<Option<ConfigUpdate>, Error>` - Configuration update if available
pub async fn get_config_updates(&self, version: &str) -> Result<Option<ConfigUpdate>, Error> {}
```

### `grpc/client/monitoring_server.rs`

```rust
/// Initialize MonitoringServer gRPC client
///
/// ### Parameters
/// * `config: ClientConfig` - Client configuration
/// ### Returns
/// * `Result<MonitoringServerClient, Error>` - Initialized client or error
pub async fn new(config: ClientConfig) -> Result<MonitoringServerClient, Error> {}

/// Connect to MonitoringServer
///
/// ### Returns
/// * `Result<(), Error>` - Connection result
pub async fn connect(&self) -> Result<(), Error> {}

/// Disconnect from MonitoringServer
///
/// ### Returns
/// * `Result<(), Error>` - Disconnection result
pub async fn disconnect(&self) -> Result<(), Error> {}

/// Send metrics to MonitoringServer
///
/// ### Parameters
/// * `metrics: &MetricsData` - Metrics data to send
/// ### Returns
/// * `Result<(), Error>` - Send result
pub async fn send_metrics(&self, metrics: &MetricsData) -> Result<(), Error> {}

/// Send logs to MonitoringServer
///
/// ### Parameters
/// * `logs: &LogData` - Log data to send
/// ### Returns
/// * `Result<(), Error>` - Send result
pub async fn send_logs(&self, logs: &LogData) -> Result<(), Error> {}

/// Stream logs to MonitoringServer
///
/// ### Parameters
/// * `stream_id: &str` - Stream identifier
/// * `logs: impl Stream<Item = LogEntry> + Send + 'static` - Log stream
/// ### Returns
/// * `Result<(), Error>` - Stream setup result
pub async fn stream_logs(&self, stream_id: &str, logs: impl Stream<Item = LogEntry> + Send + 'static) -> Result<(), Error> {}

/// Check MonitoringServer connection
///
/// ### Returns
/// * `bool` - Connection status
pub fn is_connected(&self) -> bool {}

/// Get connection statistics
///
/// ### Returns
/// * `ConnectionStats` - Connection statistics
pub fn get_connection_stats(&self) -> ConnectionStats {}
```

### `grpc/server/service.rs`

```rust
/// NodeAgent gRPC service implementation
pub struct NodeAgentService {
    // Service fields
}

/// Handle file update request
///
/// ### Parameters
/// * `request: FileUpdateRequest` - File update request details
/// ### Returns
/// * `FileUpdateResponse` - Response with result
pub async fn update_file(&self, request: FileUpdateRequest) -> FileUpdateResponse {}

/// Handle configuration deployment request
///
/// ### Parameters
/// * `request: ConfigDeployRequest` - Config deployment request details
/// ### Returns
/// * `ConfigDeployResponse` - Response with result
pub async fn deploy_config(&self, request: ConfigDeployRequest) -> ConfigDeployResponse {}

/// Handle node status query
///
/// ### Parameters
/// * `request: NodeStatusRequest` - Status request details
/// ### Returns
/// * `NodeStatusResponse` - Response with node status
pub async fn get_node_status(&self, request: NodeStatusRequest) -> NodeStatusResponse {}

/// Handle resource cleanup request
///
/// ### Parameters
/// * `request: ResourceCleanupRequest` - Cleanup request details
/// ### Returns
/// * `ResourceCleanupResponse` - Response with cleanup results
pub async fn cleanup_resources(&self, request: ResourceCleanupRequest) -> ResourceCleanupResponse {}

/// Handle Bluechi configuration file creation request
///
/// ### Parameters
/// * `request: BluechiConfigFileRequest` - Config file request details
/// ### Returns
/// * `BluechiConfigFileResponse` - Response with file creation result
pub async fn create_bluechi_config_files(&self, request: BluechiConfigFileRequest) -> BluechiConfigFileResponse {}

/// Handle Bluechi file deployment request
///
/// ### Parameters
/// * `request: BluechiFileDeployRequest` - File deploy request details
/// ### Returns
/// * `BluechiFileDeployResponse` - Response with deployment result
pub async fn deploy_bluechi_files(&self, request: BluechiFileDeployRequest) -> BluechiFileDeployResponse {}

/// Handle Bluechi environment setup request
///
/// ### Parameters
/// * `request: BluechiEnvSetupRequest` - Environment setup request details
/// ### Returns
/// * `BluechiEnvSetupResponse` - Response with setup result
pub async fn setup_bluechi_environment(&self, request: BluechiEnvSetupRequest) -> BluechiEnvSetupResponse {}
```

### `hardware/monitor.rs`

```rust
/// Start hardware monitoring
///
/// ### Parameters
/// * `config: MonitorConfig` - Monitoring configuration
/// ### Returns
/// * `HardwareMonitor` - Initialized hardware monitor
pub async fn start_monitoring(config: MonitorConfig) -> HardwareMonitor {}

/// Stop hardware monitoring
///
/// ### Parameters
/// * `monitor: &HardwareMonitor` - Monitor to stop
pub async fn stop_monitoring(monitor: &HardwareMonitor) {}

/// Get current hardware metrics
///
/// ### Returns
/// * `HardwareMetrics` - Current hardware metrics
pub async fn get_hardware_metrics() -> HardwareMetrics {}

/// Check if hardware meets requirements
///
/// ### Parameters
/// * `requirements: HardwareRequirements` - Hardware requirements
/// ### Returns
/// * `bool` - True if hardware meets requirements
pub fn check_hardware_requirements(requirements: HardwareRequirements) -> bool {}

/// Set monitoring thresholds
///
/// ### Parameters
/// * `monitor: &HardwareMonitor` - Hardware monitor
/// * `thresholds: MonitoringThresholds` - New threshold settings
pub async fn set_monitoring_thresholds(monitor: &HardwareMonitor, thresholds: MonitoringThresholds) {}
```

### `hardware/reporter.rs`

```rust
/// Initialize hardware reporter
///
/// ### Parameters
/// * `config: ReporterConfig` - Reporter configuration
/// ### Returns
/// * `HardwareReporter` - Initialized hardware reporter
pub fn new(config: ReporterConfig) -> HardwareReporter {}

/// Start reporting hardware status
///
/// ### Description
/// Begin reporting hardware status at configured intervals
pub async fn start(&self) {}

/// Stop reporting hardware status
pub async fn stop(&self) {}

/// Generate hardware report
///
/// ### Returns
/// * `HardwareReport` - Generated hardware report
pub async fn generate_report(&self) -> HardwareReport {}

/// Send hardware alert
///
/// ### Parameters
/// * `alert: HardwareAlert` - Alert to send
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn send_alert(&self, alert: HardwareAlert) -> Result<(), Error> {}
```

### `resource/cleaner.rs`

```rust
/// Initialize resource cleaner
///
/// ### Parameters
/// * `config: CleanerConfig` - Cleaner configuration
/// ### Returns
/// * `ResourceCleaner` - Initialized resource cleaner
pub fn new(config: CleanerConfig) -> ResourceCleaner {}

/// Start scheduled cleanup tasks
///
/// ### Description
/// Begin scheduled cleanup tasks at configured intervals
pub async fn start_scheduled_cleanup(&self) {}

/// Stop scheduled cleanup tasks
pub async fn stop_scheduled_cleanup(&self) {}

/// Clean temporary files
///
/// ### Parameters
/// * `older_than: Duration` - Clean files older than this duration
/// ### Returns
/// * `CleanupResult` - Cleanup operation results
pub async fn clean_temp_files(&self, older_than: Duration) -> CleanupResult {}

/// Clean cache files
///
/// ### Parameters
/// * `older_than: Duration` - Clean files older than this duration
/// ### Returns
/// * `CleanupResult` - Cleanup operation results
pub async fn clean_cache_files(&self, older_than: Duration) -> CleanupResult {}

/// Clean old log files
///
/// ### Parameters
/// * `older_than: Duration` - Clean files older than this duration
/// ### Returns
/// * `CleanupResult` - Cleanup operation results
pub async fn clean_old_logs(&self, older_than: Duration) -> CleanupResult {}
```

### `resource/disk.rs`

```rust
/// Get disk usage information
///
/// ### Parameters
/// * `path: &Path` - Path to check
/// ### Returns
/// * `Result<DiskUsage, Error>` - Disk usage information or error
pub async fn get_disk_usage(path: &Path) -> Result<DiskUsage, Error> {}

/// Check available disk space
///
/// ### Parameters
/// * `path: &Path` - Path to check
/// * `required: u64` - Required space in bytes
/// ### Returns
/// * `bool` - True if enough space is available
pub async fn has_enough_space(path: &Path, required: u64) -> bool {}

/// Reserve disk space for critical operations
///
/// ### Parameters
/// * `path: &Path` - Path where space needs to be reserved
/// * `size: u64` - Size to reserve in bytes
/// ### Returns
/// * `Result<ReservedSpace, Error>` - Reserved space handle or error
pub async fn reserve_space(path: &Path, size: u64) -> Result<ReservedSpace, Error> {}

/// Release reserved disk space
///
/// ### Parameters
/// * `reserved: ReservedSpace` - Previously reserved space
pub async fn release_reserved_space(reserved: ReservedSpace) {}

/// Find largest files in directory
///
/// ### Parameters
/// * `path: &Path` - Directory path to scan
/// * `limit: usize` - Maximum number of files to return
/// ### Returns
/// * `Vec<(PathBuf, u64)>` - List of paths and sizes of largest files
pub async fn find_largest_files(path: &Path, limit: usize) -> Vec<(PathBuf, u64)> {}
```

### `sync/distributor.rs`

```rust
/// Initialize configuration distributor
///
/// ### Parameters
/// * `config: DistributorConfig` - Distributor configuration
/// ### Returns
/// * `ConfigDistributor` - Initialized configuration distributor
pub fn new(config: DistributorConfig) -> ConfigDistributor {}

/// Distribute configuration to nodes
///
/// ### Parameters
/// * `config_data: &ConfigData` - Configuration data to distribute
/// * `nodes: &[NodeInfo]` - List of target nodes
/// ### Returns
/// * `DistributionResult` - Distribution results
pub async fn distribute_config(&self, config_data: &ConfigData, nodes: &[NodeInfo]) -> DistributionResult {}

/// Check distribution status
///
/// ### Parameters
/// * `distribution_id: &str` - ID of the distribution
/// ### Returns
/// * `DistributionStatus` - Current status of the distribution
pub async fn check_distribution_status(&self, distribution_id: &str) -> DistributionStatus {}

/// Cancel ongoing distribution
///
/// ### Parameters
/// * `distribution_id: &str` - ID of the distribution to cancel
/// ### Returns
/// * `bool` - Success status
pub async fn cancel_distribution(&self, distribution_id: &str) -> bool {}

/// Verify distributed configuration
///
/// ### Parameters
/// * `distribution_id: &str` - ID of the distribution to verify
/// ### Returns
/// * `VerificationResult` - Verification results
pub async fn verify_distributed_config(&self, distribution_id: &str) -> VerificationResult {}
```

### `sync/receiver.rs`

```rust
/// Initialize configuration receiver
///
/// ### Parameters
/// * `config: ReceiverConfig` - Receiver configuration
/// ### Returns
/// * `ConfigReceiver` - Initialized configuration receiver
pub fn new(config: ReceiverConfig) -> ConfigReceiver {}

/// Start configuration receiver
///
/// ### Description
/// Start listening for configuration updates
pub async fn start(&self) {}

/// Stop configuration receiver
pub async fn stop(&self) {}

/// Process received configuration
///
/// ### Parameters
/// * `config_data: ConfigData` - Received configuration data
/// ### Returns
/// * `Result<(), Error>` - Processing result
pub async fn process_config(&self, config_data: ConfigData) -> Result<(), Error> {}

/// Verify configuration integrity
///
/// ### Parameters
/// * `config_data: &ConfigData` - Configuration data to verify
/// ### Returns
/// * `bool` - True if configuration is valid
pub fn verify_integrity(&self, config_data: &ConfigData) -> bool {}

/// Acknowledge configuration receipt
///
/// ### Parameters
/// * `config_id: &str` - ID of the received configuration
/// * `status: ReceiptStatus` - Status of the receipt
/// ### Returns
/// * `Result<(), Error>` - Result of the acknowledgment
pub async fn acknowledge_receipt(&self, config_id: &str, status: ReceiptStatus) -> Result<(), Error> {}
```


## 4. Reference information

### 4.1 Related Components
- **StateManager**: Receives state differences (diffs) from NodeAgent and performs reconciliation tasks
- **MonitoringServer**: Receives metrics from NodeAgent for integration into monitoring and alerting systems
- **APIServer**: Provides configuration files and setting updates, and handles NodeAgent registration
- **ActionController**: Uses Bluechi configuration files created by NodeAgent to manage Bluechi services

### 4.2 Protocols and APIs
- **gRPC API**: 
  - `StateDiff`: Reports state differences to StateManager
  - `MetricsReport`: Reports metrics to MonitoringServer
  - `StateEvent`: Notifies important events (OOM, container termination, etc.)
  - `NodeRegistration`: Registers node with APIServer
  - `ConfigUpdate`: Receives configuration updates
  - `BluechiConfigFileRequest`: Request for Bluechi configuration file creation
  - `BluechiFileDeployRequest`: Request for Bluechi file deployment
  - `NetworkStatusReport`: Reports network status and connectivity
  - `ServiceHealthReport`: Reports service health check results
- **Container Monitoring API**: Collection of container metrics through cgroups, procfs, etc.
- **Hardware Monitoring API**: Collection of hardware information through sysfs, hwmon, etc.
- **File System API**: Creation and deployment of Bluechi configuration files
- **Network Diagnostic API**: Testing network status and diagnosing problems

### 4.3 Data Structures
- **CpuMetrics**: CPU usage, throttling, and other metrics
- **MemoryMetrics**: Memory usage, OOM events, and other metrics
- **StorageMetrics**: File system usage, I/O statistics, and other metrics
- **NetworkMetrics**: Network traffic, packet loss, and other metrics
- **ProcessMetrics**: Process count, restart count, and other metrics
- **StateDiff**: Difference between actual and desired states
- **ResourceState**: Current state information of resources
- **ContainerState**: Current state information of containers
- **ContainerEvent**: Container events (creation, start, termination, etc.)
- **StateEvent**: State change events
- **BluechiControllerConfig**: Bluechi controller configuration information
- **BluechiAgentConfig**: Bluechi agent configuration information
- **DeploymentResult**: File deployment results
- **ServiceHealth**: Service status information
- **NetworkTestResult**: Network test results
- **DiagnosticsResult**: Network diagnostics results

### 4.4 References
- [Linux cgroups Documentation](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)
- [gRPC Documentation](https://grpc.io/docs/)
- [Linux System Monitoring API](https://www.kernel.org/doc/html/latest/admin-guide/sysinfo.html)
- [Bluechi Project](https://github.com/eclipse-bluechi/bluechi)
- [Kubernetes Reconciliation Pattern](https://kubernetes.io/docs/concepts/architecture/controller/)
- [StateManager Interface](internal://statemanager.md)
- [Bluechi Configuration File Format](https://eclipse-bluechi.github.io/bluechi/user-guide/config.html)
- [Network Monitoring and Diagnostic Tools](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/monitoring-performance-with-the-net-snmp-suite_monitoring-and-managing-system-status-and-performance)
