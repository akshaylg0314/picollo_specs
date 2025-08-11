# Monitoring Server

## 1. Introduction

### Major features

Monitoring Server is a component that collects and analyzes system metrics and logs in the PICCOLO framework.

1. Receives and processes metrics and log data from NodeAgent.
2. Stores collected metrics and logs in ETCD.
3. Compresses stream log data and saves it as files.
4. Provides threshold-based and anomaly detection alerting functionality.
5. Monitors system status in real-time through visualization tools such as dashboards and graphs.
6. Generates status reports and provides analysis functionality.

### Main Dataflow

1. Receives all metrics and log data from NodeAgent via gRPC.
2. Processes and stores received metrics data in ETCD.
3. Compresses received log streams and stores them in the file system.
4. Analyzes collected data and generates alerts when thresholds are exceeded.
5. Provides real-time monitoring data and analysis results through a web interface.

## 2. File information

```text
monitoringserver
├── monitoringserver.md
├── Cargo.toml
└── src
    ├── alert
    │   ├── detector.rs
    │   ├── manager.rs
    │   ├── mod.rs
    │   └── notifier.rs
    ├── grpc
    │   ├── client
    │   │   └── mod.rs
    │   ├── mod.rs
    │   └── server
    │       ├── mod.rs
    │       └── node_agent.rs
    ├── log
    │   ├── aggregator.rs
    │   ├── compressor.rs
    │   ├── mod.rs
    │   ├── parser.rs
    │   └── storage.rs
    ├── main.rs
    ├── manager.rs
    ├── metrics
    │   ├── mod.rs
    │   ├── processor.rs
    └── └── store.rs
     
```

- **main.rs** - MonitoringServer initialization and execution
- **manager.rs** - Data flow control between modules
- **alert/mod.rs** - Alert management
- **alert/detector.rs** - Anomaly detection
- **alert/manager.rs** - Alert management and routing
- **alert/notifier.rs** - Alert sending
- **grpc/mod.rs** - gRPC communication management
- **grpc/client/mod.rs** - gRPC client management
- **grpc/server/mod.rs** - gRPC server management
- **grpc/server/node_agent.rs** - gRPC communication with NodeAgent
- **log/mod.rs** - Log management
- **log/aggregator.rs** - Log aggregation
- **log/compressor.rs** - Log compression
- **log/parser.rs** - Log parsing
- **log/storage.rs** - Compressed log file storage management
- **metrics/mod.rs** - Metrics management
- **metrics/processor.rs** - Metrics processing and analysis
- **metrics/store.rs** - Metrics storage management (using common ETCD library)

## 3. Function information

The function information in this section is written in the comment format used in rustdoc.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo Monitoring Server
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize and start the MonitoringServer
pub async fn initialize() {}

/// Start all monitoring services
pub async fn start_services() {}

/// Stop all monitoring services
pub async fn stop_services() {}

/// Handle incoming metrics data from NodeAgent
///
/// ### Parameters
/// * `metrics: MetricsData` - Incoming metrics data
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn handle_metrics(metrics: MetricsData) -> Result<(), Error> {}

/// Handle incoming log data from NodeAgent
///
/// ### Parameters
/// * `logs: LogData` - Incoming log data
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn handle_logs(logs: LogData) -> Result<(), Error> {}

/// Compress and store log data
///
/// ### Parameters
/// * `logs: LogData` - Log data to compress and store
/// ### Returns
/// * `Result<StorageInfo, Error>` - Storage information or error
pub async fn compress_and_store_logs(logs: LogData) -> Result<StorageInfo, Error> {}

/// Store metrics in ETCD using common library
///
/// ### Parameters
/// * `metrics: ProcessedMetrics` - Processed metrics to store
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn store_metrics(metrics: ProcessedMetrics) -> Result<(), Error> {}

/// Generate system status report
///
/// ### Parameters
/// * `report_type: ReportType` - Type of report to generate
/// ### Returns
/// * `SystemReport` - Generated system report
pub async fn generate_report(report_type: ReportType) -> SystemReport {}

/// Get metrics from ETCD using common library
///
/// ### Parameters
/// * `query: MetricsQuery` - Query parameters
/// ### Returns
/// * `Result<MetricsData, Error>` - Retrieved metrics or error
pub async fn get_metrics(query: MetricsQuery) -> Result<MetricsData, Error> {}

/// Get compressed logs
///
/// ### Parameters
/// * `query: LogQuery` - Query parameters
/// ### Returns
/// * `Result<LogData, Error>` - Retrieved logs or error
pub async fn get_compressed_logs(query: LogQuery) -> Result<LogData, Error> {}
```

### `alert/detector.rs`

```rust
/// Initialize anomaly detector
///
/// ### Parameters
/// * `config: DetectorConfig` - Detector configuration
/// ### Returns
/// * `AnomalyDetector` - Initialized anomaly detector
pub fn new(config: DetectorConfig) -> AnomalyDetector {}

/// Start anomaly detection
pub async fn start_detection(&self) {}

/// Stop anomaly detection
pub async fn stop_detection(&self) {}

/// Check metrics for anomalies
///
/// ### Parameters
/// * `metrics: &MetricsData` - Metrics data to check
/// ### Returns
/// * `Vec<Anomaly>` - Detected anomalies
pub async fn check_metrics(&self, metrics: &MetricsData) -> Vec<Anomaly> {}

/// Check logs for anomalies
///
/// ### Parameters
/// * `logs: &LogData` - Log data to check
/// ### Returns
/// * `Vec<Anomaly>` - Detected anomalies
pub async fn check_logs(&self, logs: &LogData) -> Vec<Anomaly> {}

/// Update detection thresholds
///
/// ### Parameters
/// * `thresholds: DetectionThresholds` - New threshold settings
pub async fn update_thresholds(&self, thresholds: DetectionThresholds) {}
```

### `alert/manager.rs`

```rust
/// Initialize alert manager
///
/// ### Parameters
/// * `config: AlertManagerConfig` - Alert manager configuration
/// ### Returns
/// * `AlertManager` - Initialized alert manager
pub fn new(config: AlertManagerConfig) -> AlertManager {}

/// Start alert manager
pub async fn start(&self) {}

/// Stop alert manager
pub async fn stop(&self) {}

/// Process new alert
///
/// ### Parameters
/// * `alert: Alert` - Alert to process
/// ### Returns
/// * `AlertId` - ID of the processed alert
pub async fn process_alert(&self, alert: Alert) -> AlertId {}

/// Resolve alert
///
/// ### Parameters
/// * `alert_id: AlertId` - ID of the alert to resolve
/// ### Returns
/// * `bool` - Success status
pub async fn resolve_alert(&self, alert_id: AlertId) -> bool {}

/// Get active alerts
///
/// ### Parameters
/// * `filter: Option<AlertFilter>` - Optional filter for alerts
/// ### Returns
/// * `Vec<Alert>` - List of active alerts
pub async fn get_active_alerts(&self, filter: Option<AlertFilter>) -> Vec<Alert> {}

/// Set alert routing rules
///
/// ### Parameters
/// * `rules: AlertRoutingRules` - New routing rules
pub async fn set_routing_rules(&self, rules: AlertRoutingRules) {}
```

### `alert/notifier.rs`

```rust
/// Initialize alert notifier
///
/// ### Parameters
/// * `config: NotifierConfig` - Notifier configuration
/// ### Returns
/// * `AlertNotifier` - Initialized alert notifier
pub fn new(config: NotifierConfig) -> AlertNotifier {}

/// Send alert notification
///
/// ### Parameters
/// * `alert: &Alert` - Alert to send
/// * `channels: &[NotificationChannel]` - Channels to use
/// ### Returns
/// * `Vec<NotificationResult>` - Results of sending notifications
pub async fn send_notification(&self, alert: &Alert, channels: &[NotificationChannel]) -> Vec<NotificationResult> {}

/// Add notification channel
///
/// ### Parameters
/// * `channel: NotificationChannel` - Channel to add
/// ### Returns
/// * `ChannelId` - ID of the added channel
pub async fn add_channel(&self, channel: NotificationChannel) -> ChannelId {}

/// Remove notification channel
///
/// ### Parameters
/// * `channel_id: ChannelId` - ID of the channel to remove
/// ### Returns
/// * `bool` - Success status
pub async fn remove_channel(&self, channel_id: ChannelId) -> bool {}

/// Test notification channel
///
/// ### Parameters
/// * `channel: &NotificationChannel` - Channel to test
/// ### Returns
/// * `TestResult` - Test result
pub async fn test_channel(&self, channel: &NotificationChannel) -> TestResult {}
```

### `grpc/server/node_agent.rs`

```rust
/// Initialize NodeAgent gRPC server
///
/// ### Parameters
/// * `config: ServerConfig` - Server configuration
/// ### Returns
/// * `Result<NodeAgentServer, Error>` - Initialized NodeAgent server or error
pub async fn new(config: ServerConfig) -> Result<NodeAgentServer, Error> {}

/// Start gRPC server
///
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn start(&self) -> Result<(), Error> {}

/// Stop gRPC server
///
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn stop(&self) -> Result<(), Error> {}

/// Handle metrics data from NodeAgent
///
/// Implementation of the MonitoringMetricsService.SendMetrics RPC
///
/// ### Parameters
/// * `request: Request<SendMetricsRequest>` - gRPC request
/// ### Returns
/// * `Result<Response<SendMetricsResponse>, Status>` - gRPC response
pub async fn send_metrics(
    &self,
    request: Request<SendMetricsRequest>,
) -> Result<Response<SendMetricsResponse>, Status> {}

/// Handle log data from NodeAgent
///
/// Implementation of the MonitoringLogService.SendLogs RPC
///
/// ### Parameters
/// * `request: Request<SendLogsRequest>` - gRPC request
/// ### Returns
/// * `Result<Response<SendLogsResponse>, Status>` - gRPC response
pub async fn send_logs(
    &self,
    request: Request<SendLogsRequest>,
) -> Result<Response<SendLogsResponse>, Status> {}

/// Receive streaming log data from NodeAgent
///
/// Implementation of the MonitoringLogService.StreamLogs RPC
///
/// ### Parameters
/// * `request: Request<tonic::Streaming<StreamLogsRequest>>` - gRPC streaming request
/// ### Returns
/// * `Result<Response<StreamLogsResponse>, Status>` - gRPC response
pub async fn stream_logs(
    &self,
    request: Request<tonic::Streaming<StreamLogsRequest>>,
) -> Result<Response<StreamLogsResponse>, Status> {}

/// Get server status
///
/// ### Returns
/// * `ServerStatus` - Current server status
pub fn status(&self) -> ServerStatus {}
```

### `grpc/server/state_manager.rs`

```rust
/// Handle state alert submission
///
/// ### Parameters
/// * `request: StateAlertRequest` - State alert data
/// ### Returns
/// * `StateAlertResponse` - Response with result
pub async fn handle_state_alert(&self, request: StateAlertRequest) -> StateAlertResponse {}

/// Handle state metrics submission
///
/// ### Parameters
/// * `request: StateMetricsRequest` - State metrics data
/// ### Returns
/// * `StateMetricsResponse` - Response with result
pub async fn handle_state_metrics(&self, request: StateMetricsRequest) -> StateMetricsResponse {}

/// Get alert configuration
///
/// ### Parameters
/// * `request: AlertConfigRequest` - Config request
/// ### Returns
/// * `AlertConfigResponse` - Response with configuration
pub async fn get_alert_config(&self, request: AlertConfigRequest) -> AlertConfigResponse {}
```

### `log/aggregator.rs`

```rust
/// Initialize log aggregator
///
/// ### Parameters
/// * `config: AggregatorConfig` - Aggregator configuration
/// ### Returns
/// * `LogAggregator` - Initialized log aggregator
pub fn new(config: AggregatorConfig) -> LogAggregator {}

/// Start log aggregation
pub async fn start(&self) {}

/// Stop log aggregation
pub async fn stop(&self) {}

/// Aggregate logs
///
/// ### Parameters
/// * `logs: &[LogEntry]` - Logs to aggregate
/// ### Returns
/// * `AggregatedLogs` - Aggregated log data
pub async fn aggregate(&self, logs: &[LogEntry]) -> AggregatedLogs {}

/// Set aggregation rules
///
/// ### Parameters
/// * `rules: AggregationRules` - New aggregation rules
pub async fn set_rules(&self, rules: AggregationRules) {}

/// Get aggregation statistics
///
/// ### Returns
/// * `AggregationStats` - Current aggregation statistics
pub async fn get_stats(&self) -> AggregationStats {}
```

### `log/compressor.rs`

```rust
/// Initialize log compressor
///
/// ### Parameters
/// * `config: CompressorConfig` - Compressor configuration
/// ### Returns
/// * `LogCompressor` - Initialized log compressor
pub fn new(config: CompressorConfig) -> LogCompressor {}

/// Compress log data
///
/// ### Parameters
/// * `logs: &LogData` - Log data to compress
/// ### Returns
/// * `Result<CompressedData, Error>` - Compressed data or error
pub async fn compress(&self, logs: &LogData) -> Result<CompressedData, Error> {}

/// Decompress log data
///
/// ### Parameters
/// * `compressed: &CompressedData` - Compressed data
/// ### Returns
/// * `Result<LogData, Error>` - Decompressed log data or error
pub async fn decompress(&self, compressed: &CompressedData) -> Result<LogData, Error> {}

/// Set compression level
///
/// ### Parameters
/// * `level: CompressionLevel` - Compression level
pub fn set_compression_level(&mut self, level: CompressionLevel) {}

/// Get compression statistics
///
/// ### Returns
/// * `CompressionStats` - Compression statistics
pub fn get_stats(&self) -> CompressionStats {}
```

### `log/storage.rs`

```rust
/// Initialize log storage
///
/// ### Parameters
/// * `config: LogStorageConfig` - Storage configuration
/// ### Returns
/// * `Result<LogStorage, Error>` - Initialized log storage or error
pub fn new(config: LogStorageConfig) -> Result<LogStorage, Error> {}

/// Store compressed log data
///
/// ### Parameters
/// * `compressed: &CompressedData` - Compressed log data
/// * `metadata: &LogMetadata` - Log metadata
/// ### Returns
/// * `Result<StorageInfo, Error>` - Storage information or error
pub async fn store(&self, compressed: &CompressedData, metadata: &LogMetadata) -> Result<StorageInfo, Error> {}

/// Retrieve compressed log data
///
/// ### Parameters
/// * `id: &str` - Log ID
/// ### Returns
/// * `Result<CompressedData, Error>` - Retrieved compressed data or error
pub async fn retrieve(&self, id: &str) -> Result<CompressedData, Error> {}

/// List available log archives
///
/// ### Parameters
/// * `filter: Option<LogFilter>` - Optional filter
/// ### Returns
/// * `Result<Vec<LogMetadata>, Error>` - List of log metadata or error
pub async fn list(&self, filter: Option<LogFilter>) -> Result<Vec<LogMetadata>, Error> {}

/// Delete log archive
///
/// ### Parameters
/// * `id: &str` - Log ID
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn delete(&self, id: &str) -> Result<(), Error> {}

/// Rotate log archives
///
/// ### Parameters
/// * `retention_policy: RetentionPolicy` - Retention policy
/// ### Returns
/// * `Result<RotationStats, Error>` - Rotation statistics or error
pub async fn rotate(&self, retention_policy: RetentionPolicy) -> Result<RotationStats, Error> {}

/// Get storage statistics
///
/// ### Returns
/// * `Result<StorageStats, Error>` - Storage statistics or error
pub async fn get_stats(&self) -> Result<StorageStats, Error> {}
```

### `metrics/collector.rs`

```rust
/// Initialize metrics collector
///
/// ### Parameters
/// * `config: CollectorConfig` - Collector configuration
/// ### Returns
/// * `MetricsCollector` - Initialized metrics collector
pub fn new(config: CollectorConfig) -> MetricsCollector {}

/// Start metrics collection
pub async fn start(&self) {}

/// Stop metrics collection
pub async fn stop(&self) {}

/// Collect container metrics
///
/// ### Parameters
/// * `container_id: &str` - ID of the container
/// ### Returns
/// * `Result<ContainerMetrics, Error>` - Container metrics or error
pub async fn collect_container_metrics(&self, container_id: &str) -> Result<ContainerMetrics, Error> {}

/// Collect system metrics
///
/// ### Returns
/// * `Result<SystemMetrics, Error>` - System metrics or error
pub async fn collect_system_metrics(&self) -> Result<SystemMetrics, Error> {}

/// Collect hardware metrics
///
/// ### Returns
/// * `Result<HardwareMetrics, Error>` - Hardware metrics or error
pub async fn collect_hardware_metrics(&self) -> Result<HardwareMetrics, Error> {}

/// Set collection interval
///
/// ### Parameters
/// * `interval: Duration` - New collection interval
pub async fn set_collection_interval(&self, interval: Duration) {}
```

### `metrics/processor.rs`

```rust
/// Initialize metrics processor
///
/// ### Parameters
/// * `config: ProcessorConfig` - Processor configuration
/// ### Returns
/// * `MetricsProcessor` - Initialized metrics processor
pub fn new(config: ProcessorConfig) -> MetricsProcessor {}

/// Process metrics data
///
/// ### Parameters
/// * `metrics: &RawMetrics` - Raw metrics to process
/// ### Returns
/// * `ProcessedMetrics` - Processed metrics data
pub async fn process(&self, metrics: &RawMetrics) -> ProcessedMetrics {}

/// Calculate statistics
///
/// ### Parameters
/// * `metrics: &[MetricsPoint]` - Metrics points
/// * `window: Duration` - Time window for calculation
/// ### Returns
/// * `MetricsStats` - Calculated statistics
pub fn calculate_stats(metrics: &[MetricsPoint], window: Duration) -> MetricsStats {}

/// Detect metrics trends
///
/// ### Parameters
/// * `metrics: &[MetricsPoint]` - Metrics points
/// * `window: Duration` - Time window for trend detection
/// ### Returns
/// * `Vec<MetricsTrend>` - Detected trends
pub fn detect_trends(metrics: &[MetricsPoint], window: Duration) -> Vec<MetricsTrend> {}

/// Generate metrics summary
///
/// ### Parameters
/// * `metrics: &ProcessedMetrics` - Processed metrics data
/// ### Returns
/// * `MetricsSummary` - Generated summary
pub fn generate_summary(metrics: &ProcessedMetrics) -> MetricsSummary {}
```

### `metrics/store.rs`

```rust
/// Initialize metrics store
///
/// ### Parameters
/// * `config: MetricsStoreConfig` - Store configuration
/// ### Returns
/// * `Result<MetricsStore, Error>` - Initialized metrics store or error
pub async fn new(config: MetricsStoreConfig) -> Result<MetricsStore, Error> {}

/// Store metrics using common ETCD library
///
/// ### Parameters
/// * `metrics: &ProcessedMetrics` - Processed metrics to store
/// ### Returns
/// * `Result<(), Error>` - Result indicating success or failure
pub async fn store(&self, metrics: &ProcessedMetrics) -> Result<(), Error> {}

/// Retrieve metrics using common ETCD library
///
/// ### Parameters
/// * `query: &MetricsQuery` - Query parameters
/// ### Returns
/// * `Result<MetricsData, Error>` - Retrieved metrics or error
pub async fn retrieve(&self, query: &MetricsQuery) -> Result<MetricsData, Error> {}

/// Generate metrics key
///
/// ### Parameters
/// * `metric_type: MetricType` - Type of metric
/// * `source: &str` - Source identifier
/// * `timestamp: i64` - Unix timestamp
/// ### Returns
/// * `String` - Generated key
fn generate_key(&self, metric_type: MetricType, source: &str, timestamp: i64) -> String {}

/// Parse metrics key
///
/// ### Parameters
/// * `key: &str` - Storage key
/// ### Returns
/// * `Result<MetricKeyInfo, Error>` - Parsed key information or error
fn parse_key(&self, key: &str) -> Result<MetricKeyInfo, Error> {}

/// Set retention policy
///
/// ### Parameters
/// * `policy: RetentionPolicy` - Retention policy
pub fn set_retention_policy(&mut self, policy: RetentionPolicy) {}

/// Cleanup old metrics based on retention policy
///
/// ### Returns
/// * `Result<CleanupStats, Error>` - Cleanup statistics or error
pub async fn cleanup_old_metrics(&self) -> Result<CleanupStats, Error> {}

/// Get store statistics
///
/// ### Returns
/// * `Result<StoreStats, Error>` - Store statistics or error
pub async fn get_stats(&self) -> Result<StoreStats, Error> {}
```

## 4. Reference information

### 4.1 Related Components
- **StateManager**: Sends state alerts and metrics to MonitoringServer
- **NodeAgent**: Sends hardware status and node metrics to MonitoringServer

### 4.2 Protocol and API
- **gRPC API**: gRPC service definitions for communication with StateManager and NodeAgent
- **REST API**: REST API for web interface and integration with external systems
- **WebSocket API**: WebSocket interface for real-time dashboard updates
- **Podman API**: Podman interface for collecting container status and logs

### 4.3 Data Structures
- **MetricsData**: Structure for collected metrics data
- **LogData**: Structure for collected log data
- **Alert**: Structure for alert information
- **SystemReport**: Structure for system status reports
- **DashboardPanel**: Structure for dashboard panel configuration

### 4.4 Reference Materials
- [Prometheus Metrics Format](https://prometheus.io/docs/concepts/data_model/)
- [OpenTelemetry Standard](https://opentelemetry.io/docs/)
- [REST API Design Guide](https://restfulapi.net/)
- [WebSocket Protocol](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Podman API Documentation](https://docs.podman.io/en/latest/Reference.html)
