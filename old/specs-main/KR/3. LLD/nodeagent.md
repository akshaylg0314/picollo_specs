# Node Agent

## 1. Introduction

### Major features

Node Agent는 PICCOLO 프레임워크에서 각 노드의 상태를 모니터링하고 관리하는 컴포넌트이다. 시스템 내 모든 모니터링 정보 수집을 담당한다.

1. Pod/컨테이너 메트릭을 수집하여 상태관리에 필요한 정보를 제공한다.
   - 컴퓨팅 리소스 (CPU 사용률, 스로틀링, 스케줄링 지연 등)
   - 메모리 리소스 (사용량, 작업 세트, OOM 이벤트 등)
   - 스토리지 및 I/O (파일 시스템 사용량, 디스크 연산 등)
   - 네트워크 (수신/전송 바이트, 오류, 패킷 드롭 등)
   - 프로세스 및 런타임 메트릭 (프로세스 수, 재시작 횟수 등)
   - 애플리케이션 수준 메트릭 (응답 시간, 오류율 등)
2. 하드웨어 시스템 정보를 수집한다.
   - CPU, 메모리, 디스크, 네트워크 하드웨어 상태 (통합된 resource.rs에서 관리)
   - 온도, 전압 등의 하드웨어 건전성 지표
3. 노드 가용성을 모니터링하고 보고한다.
4. 구성 파일 배포 및 설정 업데이트를 처리한다.
5. Bluechi 사전 설정 및 환경 구성 작업을 수행한다.
   - Bluechi 구성 파일 생성 및 배포
   - 노드 환경 구성 및 필요한 디렉토리/파일 설정
   - 파일 권한 및 기본 설정
6. StateManager에 노드 및 컨테이너 상태 정보를 전송한다.
7. MonitoringServer에 모든 메트릭 및 로그 데이터를 전송한다.
8. 네트워크 연결 상태 및 서비스 가용성을 모니터링한다.
   - 핵심 서비스에 대한 상태 점검 수행
   - 네트워크 연결성 테스트 및 품질 측정
   - 네트워크 문제 발생 시 자동 진단 실행

### Main Dataflow

1. 정해진 샘플링 주기에 따라 컨테이너 및 시스템 메트릭을 수집한다.
   - 필수 메트릭은 1초 주기로 수집 (CPU/메모리 사용률, OOM 이벤트 등)
   - 중요도가 낮은 메트릭은 5초 또는 10초 주기로 수집
   - 통합된 resource.rs 파일이 CPU, 메모리, 디스크, 네트워크 메트릭 수집을 담당
2. 수집된 메트릭을 필터링하고 정규화하여 상태 이벤트로 변환한다.
3. 실제 상태와 원하는 상태(ETCD에서 로드)를 비교하여 상태 차이(diff)를 계산한다.
4. 계산된 상태 차이와 중요 이벤트(OOM 킬, 컨테이너 종료 등)를 StateManager로 전송한다.
5. 모든 모니터링 메트릭은 MonitoringServer로 전송하여 ETCD에 저장하고 분석할 수 있게 한다.
6. 로그 데이터는 일반 로그와 스트림 로그로 구분하여 MonitoringServer에 전송한다.
7. StateManager로부터 상태 관련 요청을 수신하고 필요한 정보를 응답한다.
8. API Server로부터 구성 파일 및 설정 업데이트를 수신하여 적용한다.
8. Bluechi 사전 설정 작업을 수행한다.
   - API Server로부터 필요한 노드 정보와 구성 파라미터를 수신한다.
   - Bluechi 클러스터 및 에이전트 구성 파일을 생성한다.
   - 생성된 구성 파일을 적절한 위치에 배포하고 필요한 디렉토리 및 권한을 설정한다.
   - Bluechi 기본 설정 상태를 API Server에 보고한다.
9. 정기적인 네트워크 상태 점검을 수행하고 결과를 MonitoringServer에 전송한다.
   - 중요 서비스에 대한 상태 점검 실행
   - 네트워크 지연 시간, 패킷 손실률 등 품질 지표 수집
   - 네트워크 문제 진단 및 복구 시도

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

- **main.rs** - NodeAgent 초기화 및 실행
- **manager.rs** - 각 모듈 간의 데이터 흐름 제어
- **config/mod.rs** - 구성 관련 기능 모듈
- **config/loader.rs** - 구성 파일 로더
- **config/validator.rs** - 구성 검증기
- **grpc/mod.rs** - gRPC 통신 관리
- **grpc/client/mod.rs** - gRPC 클라이언트 관리
- **grpc/client/state_manager.rs** - StateManager와의 gRPC 통신
- **grpc/client/api_server.rs** - APIServer와의 gRPC 통신
- **grpc/client/monitoring_server.rs** - MonitoringServer와의 gRPC 통신
- **grpc/server/mod.rs** - gRPC 서버 관리
- **grpc/server/service.rs** - NodeAgent gRPC 서비스 구현
- **metrics/mod.rs** - 메트릭 수집 기능 모듈
- **metrics/model.rs** - 메트릭 데이터 모델 정의
- **metrics/collector/mod.rs** - 메트릭 수집기 기능 모듈 (resource.rs 참조)
- **hardware/mod.rs** - 하드웨어 관련 기능 모듈 (resource.rs 참조)
- **hardware/monitor.rs** - 하드웨어 모니터링 기능
- **hardware/health.rs** - 하드웨어 건전성 지표 수집
- **resource.rs** - 통합된 리소스 관련 기능 모듈 (CPU, 메모리, 디스크, 네트워크 메트릭 수집 및 하드웨어 상태 모니터링)
- **container/mod.rs** - 컨테이너 관련 기능 모듈
- **container/runtime.rs** - 컨테이너 런타임 연동
- **container/status.rs** - 컨테이너 상태 관리
- **container/events.rs** - 컨테이너 이벤트 처리
- **diff/mod.rs** - 상태 차이(diff) 관리 모듈
- **diff/calculator.rs** - 실제 상태와 원하는 상태 간의 차이 계산
- **diff/model.rs** - 상태 차이 모델 정의
- **diff/reporter.rs** - 상태 차이 정보 StateManager로 전송
- **event/mod.rs** - 이벤트 처리 기능 모듈
- **event/filter.rs** - 이벤트 필터링
- **event/dispatcher.rs** - 이벤트 디스패처
- **bluechi/mod.rs** - Bluechi 사전 설정 기능 모듈
- **bluechi/config_generator.rs** - Bluechi 구성 파일 생성기
- **bluechi/file_deployer.rs** - Bluechi 파일 배포기
- **bluechi/models.rs** - Bluechi 구성 모델 정의
- **resource/mod.rs** - 리소스 관리
- **resource/cleaner.rs** - 임시 파일 및 캐시 정리
- **resource/disk.rs** - 디스크 공간 관리
- **sync/mod.rs** - 노드 간 동기화
- **sync/distributor.rs** - 설정 배포
- **sync/receiver.rs** - 설정 수신 및 적용

## 3. Function information

본 문단의 function 정보는 rustdoc 에서 사용하는 주석 형태로 작성되었다.
이에 대해서는 [링크](https://doc.rust-lang.org/stable/rustdoc/index.html) 를 참조하라.

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

### 4.1 관련 컴포넌트
- **StateManager**: NodeAgent로부터 상태 차이(diff)를 수신하고 조정 작업 수행
- **MonitoringServer**: NodeAgent로부터 메트릭을 수신하여 모니터링 및 알림 시스템에 통합
- **APIServer**: 구성 파일 및 설정 업데이트를 제공하고 NodeAgent 등록을 처리
- **ActionController**: NodeAgent가 생성한 Bluechi 구성 파일을 활용하여 Bluechi 서비스 관리

### 4.2 프로토콜 및 API
- **gRPC API**: 
  - `StateDiff`: StateManager로 상태 차이 보고
  - `MetricsReport`: MonitoringServer로 메트릭 보고
  - `StateEvent`: 중요 이벤트 (OOM, 컨테이너 종료 등) 통지
  - `NodeRegistration`: APIServer에 노드 등록
  - `ConfigUpdate`: 구성 업데이트 수신
  - `BluechiConfigFileRequest`: Bluechi 구성 파일 생성 요청 
  - `BluechiFileDeployRequest`: Bluechi 파일 배포 요청
  - `NetworkStatusReport`: 네트워크 상태 및 연결성 보고
  - `ServiceHealthReport`: 서비스 상태 점검 결과 보고
- **컨테이너 모니터링 API**: cgroups, procfs 등을 통한 컨테이너 메트릭 수집
- **하드웨어 모니터링 API**: sysfs, hwmon 등을 통한 하드웨어 정보 수집
- **파일 시스템 API**: Bluechi 구성 파일 생성 및 배포
- **네트워크 진단 API**: 네트워크 상태 테스트 및 문제 진단

### 4.3 데이터 구조
- **CpuMetrics**: CPU 사용량, 스로틀링 등 메트릭
- **MemoryMetrics**: 메모리 사용량, OOM 이벤트 등 메트릭
- **StorageMetrics**: 파일 시스템 사용량, I/O 통계 등 메트릭
- **NetworkMetrics**: 네트워크 트래픽, 패킷 손실 등 메트릭
- **ProcessMetrics**: 프로세스 수, 재시작 횟수 등 메트릭
- **StateDiff**: 실제 상태와 원하는 상태 간의 차이
- **ResourceState**: 리소스의 현재 상태 정보
- **ContainerState**: 컨테이너의 현재 상태 정보
- **ContainerEvent**: 컨테이너 이벤트 (생성, 시작, 종료 등)
- **StateEvent**: 상태 변경 이벤트
- **BluechiControllerConfig**: Bluechi 컨트롤러 구성 정보
- **BluechiAgentConfig**: Bluechi 에이전트 구성 정보
- **DeploymentResult**: 파일 배포 결과
- **ServiceHealth**: 서비스 상태 정보
- **NetworkTestResult**: 네트워크 테스트 결과
- **DiagnosticsResult**: 네트워크 진단 결과

### 4.4 참고 자료
- [Linux cgroups 문서](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)
- [gRPC 문서](https://grpc.io/docs/)
- [Linux 시스템 모니터링 API](https://www.kernel.org/doc/html/latest/admin-guide/sysinfo.html)
- [Bluechi 프로젝트](https://github.com/eclipse-bluechi/bluechi)
- [Kubernetes Reconciliation 패턴](https://kubernetes.io/docs/concepts/architecture/controller/)
- [StateManager 인터페이스](internal://statemanager.md)
- [Bluechi 구성 파일 형식](https://eclipse-bluechi.github.io/bluechi/user-guide/config.html)
- [네트워크 모니터링 및 진단 도구](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/monitoring-performance-with-the-net-snmp-suite_monitoring-and-managing-system-status-and-performance)
