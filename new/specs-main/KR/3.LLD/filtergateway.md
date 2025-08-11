# Filter Gateway

## 1. Introduction

### Major features

Filter Gateway는 차량 데이터를 수신하고 시나리오 조건을 평가하여, 조건 충족 시 ActionController로 조건 충족 정보를 전달하는 역할을 한다.

1. 차량 데이터 수신 - IDL 기반 프로토콜(DDS, SOMEIP)을 통한 차량 데이터 수신
2. 데이터 정규화 및 변환 - IDL 기반 데이터를 내부 데이터 구조로 변환
3. 시나리오 조건 필터링 - API Server로부터 수신한 시나리오 조건에 따라 데이터 필터링
4. 조건 평가 - 시나리오의 조건을 현재 차량 상태와 비교하여 평가
5. 데이터 버퍼링 - 부하 관리 및 네트워크 중단 시 데이터 캐싱
6. 조건 충족 통지 - 조건 충족 시 ActionController로 정보 전달

### Main Dataflow

1. gRPC를 통해 APIServer로부터 시나리오 조건 정보를 수신한다.
2. IDL 기반 프로토콜(DDS, SOMEIP)을 통해 차량 데이터를 수신한다.
3. 수신된 데이터를 버퍼링하여 부하를 관리한다.
4. IDL 정의에 따라 데이터를 내부 데이터 구조로 변환한다.
5. 변환된 데이터를 시나리오 조건에 따라 필터링한다.
6. 차량 상태와 시나리오 조건을 비교하여 조건 충족 여부를 평가한다.
7. 조건이 충족되면 gRPC를 통해 ActionController로 조건 충족 정보를 전달한다.
8. 성능 메트릭 및 필터링 통계를 MonitoringServer로 전송한다.

## 2. File information

```text
filtergateway
├── Cargo.toml
└── src
    ├── main.rs
    ├── manager.rs
    ├── grpc
    │   ├── mod.rs
    │   ├── receiver.rs
    │   └── sender.rs
    ├── filter
    │   └── mod.rs
    ├── vehicle
    │   ├── mod.rs
    │   └── dds
    │       ├── mod.rs
    │       └── listener.rs
    └── idl
        ├── mod.rs
        ├── parser.rs
        └── generator.rs
```

- **main.rs** - FilterGateway 초기화 및 실행
- **manager.rs** - 전체 모듈 관리 및 데이터 흐름 제어
- **grpc/mod.rs** - gRPC 관련 기능 모듈
- **grpc/receiver.rs** - APIServer로부터 시나리오 조건 정보 수신
- **grpc/sender.rs** - ActionController로 조건 충족 정보 전달
- **filter/mod.rs** - 시나리오 조건 필터링 및 평가
- **vehicle/mod.rs** - 차량 데이터 관리 모듈
- **vehicle/dds/mod.rs** - DDS 프로토콜 관리
- **vehicle/dds/listener.rs** - DDS 토픽 리스너 구현
- **idl/mod.rs** - IDL 정의 및 처리 관련 모듈
- **idl/parser.rs** - IDL 파일 파싱 기능
- **idl/generator.rs** - IDL 기반 코드 생성 기능

## 3. Function information

본 문단의 function 정보는 rustdoc 에서 사용하는 주석 형태로 작성되었다.
이에 대해서는 [링크](https://doc.rust-lang.org/stable/rustdoc/index.html) 를 참조하라.

### `main.rs`

```rust
/// Main function of Piccolo Filter Gateway
#[tokio::main]
async fn main() -> Result<()> {}

/// Initialize FilterGateway
///
/// Sets up the manager thread, gRPC services, and DDS listeners.
/// This is the main initialization function for the FilterGateway component.
///
/// # Returns
///
/// * `Result<()>` - Success or error result
async fn initialize() -> Result<()> {}
```

### `manager.rs`

```rust
/// FilterGatewayManager structure
pub struct FilterGatewayManager {
    /// Receiver for scenario information from gRPC
    rx_grpc: mpsc::Receiver<Scenario>,
    /// Sender for DDS data
    tx_dds: mpsc::Sender<DdsData>,
    /// Receiver for DDS data
    rx_dds: Arc<Mutex<mpsc::Receiver<DdsData>>>,
    /// Active filters for scenarios
    filters: Arc<Mutex<Vec<Filter>>>,
    /// gRPC sender for action controller
    sender: Arc<FilterGatewaySender>,
}

impl FilterGatewayManager {
    /// Creates a new FilterGatewayManager instance
    pub fn new(rx: mpsc::Receiver<Scenario>) -> Self {}

    /// Start the manager processing
    pub async fn run(&mut self) -> Result<()> {}

    /// Subscribe to vehicle data for a scenario
    pub async fn subscribe_vehicle_data(
        &self,
        scenario_name: String,
        vehicle_message: DdsData,
    ) -> Result<()> {}

    /// Unsubscribe from vehicle data for a scenario
    pub async fn unsubscribe_vehicle_data(
        &self,
        scenario_name: String,
        vehicle_message: DdsData,
    ) -> Result<()> {}

    /// Create and launch a filter for a scenario
    pub async fn launch_scenario_filter(&self, scenario: Scenario) -> Result<()> {}

    /// Remove a filter for a scenario
    pub async fn remove_scenario_filter(&self, scenario_name: String) -> Result<()> {}

    /// Load and initialize IDL definitions
    pub async fn initialize_idl(&self, idl_dir: &Path) -> Result<()> {}

    /// Process incoming vehicle data based on IDL definition
    pub async fn process_idl_data(&self, data: &[u8], idl_definition: &IdlDefinition, type_name: &str) -> Result<()> {}
}
```

### `grpc/mod.rs`

```rust
/// Initializes the gRPC module for FilterGateway
pub async fn init() -> common::Result<()> {}
```

### `grpc/receiver.rs`

```rust
/// FilterGateway gRPC service handler
pub struct FilterGatewayReceiver {
    tx: mpsc::Sender<Scenario>,
}

impl FilterGatewayReceiver {
    /// Create a new FilterGatewayReceiver
    pub fn new(tx: mpsc::Sender<Scenario>) -> Self {}

    /// Get the gRPC server for this receiver
    pub fn into_service(self) -> FilterGatewayConnectionServer<Self> {}

    /// Handle a scenario from API-Server
    pub async fn handle_scenario(&self, scenario_yaml_str: String, action: i32) -> Result<()> {}
}

#[tonic::async_trait]
impl FilterGatewayConnection for FilterGatewayReceiver {
    async fn handle_scenario(
        &self,
        request: Request<HandleScenarioRequest>,
    ) -> std::result::Result<Response<HandleScenarioResponse>, Status> {}
}
```

### `grpc/sender.rs`

```rust
/// Sender for making gRPC requests to ActionController
#[derive(Clone)]
pub struct FilterGatewaySender {
    client: Option<ActionControllerConnectionClient<Channel>>,
}

impl FilterGatewaySender {
    /// Create a new FilterGatewaySender
    pub fn new() -> Self {}

    /// Initialize the gRPC connection to ActionController
    pub async fn init(&mut self) -> Result<()> {}

    /// Trigger an action for a scenario
    pub async fn trigger_action(&self, scenario_name: String) -> Result<()> {}
}
```

### `filter/mod.rs`

```rust
/// Filter for evaluating scenario conditions
pub struct Filter {
    /// Name of the scenario
    pub scenario_name: String,
    /// Full scenario definition
    scenario: Scenario,
    /// Flag to indicate if the filter is active
    is_active: bool,
    /// gRPC sender for action controller
    sender: Arc<FilterGatewaySender>,
}

impl Filter {
    /// Create a new Filter
    pub fn new(
        scenario_name: String,
        scenario: Scenario,
        rx_dds: Arc<Mutex<mpsc::Receiver<DdsData>>>,
        sender: Arc<FilterGatewaySender>,
    ) -> Self {}

    /// Check if scenario conditions are met
    pub async fn meet_scenario_condition(&self, data: &DdsData) -> Result<()> {}

    /// Pause the filter processing
    pub async fn pause_scenario_filter(&mut self, scenario_name: String) -> Result<()> {}

    /// Resume the filter processing
    pub async fn resume_scenario_filter(&mut self, scenario_name: String) -> Result<()> {}
}

/// Process DDS data for filters
pub async fn handle_dds(
    arc_rx_dds: Arc<Mutex<mpsc::Receiver<DdsData>>>,
    arc_filters: Arc<Mutex<Vec<Filter>>>,
) {}
```

### `vehicle/mod.rs`

```rust
/// Vehicle data management module
pub struct VehicleManager {
    /// DDS Manager instance
    dds_manager: dds::DdsManager,
}

impl VehicleManager {
    /// Creates a new VehicleManager
    pub fn new() -> Self {}

    /// Initializes the vehicle data system
    pub async fn init(&mut self) -> Result<()> {}

    /// Subscribes to a vehicle data topic
    pub async fn subscribe_topic(
        &mut self,
        topic_name: String,
        data_type_name: String,
    ) -> Result<()> {}

    /// Unsubscribes from a vehicle data topic
    pub async fn unsubscribe_topic(&mut self, topic_name: String) -> Result<()> {}

    /// Gets the DDS data sender
    pub fn get_sender(&self) -> tokio::sync::mpsc::Sender<dds::DdsData> {}

    /// Sets the DDS domain ID
    pub fn set_domain_id(&mut self, domain_id: u32) {}
}
```

### `vehicle/dds/mod.rs`

```rust
/// DDS data structure
#[derive(Debug, Clone)]
pub struct DdsData {
    /// Name of the topic
    pub name: String,
    /// Value of the data
    pub value: String,
}

/// Initializes the DDS module
pub async fn run(tx: mpsc::Sender<DdsData>) -> Result<()> {}

/// DDS manager for handling listeners
pub struct DdsManager {
    /// Collection of active listeners
    listeners: Vec<listener::TopicListener>,
    /// Domain ID for DDS communications
    domain_id: u32,
    /// Channel sender for DDS data
    tx: mpsc::Sender<DdsData>,
}

impl DdsManager {
    /// Creates a new DDS manager
    pub fn new() -> Self {}

    /// Sets the domain ID
    pub fn set_domain_id(&mut self, domain_id: u32) {}

    /// Gets the sender channel
    pub fn get_sender(&self) -> mpsc::Sender<DdsData> {}

    /// Creates a listener for a topic
    pub async fn create_listener(
        &mut self,
        topic_name: String,
        data_type_name: String,
    ) -> Result<()> {}

    /// Removes a listener for a topic
    pub async fn remove_listener(&mut self, topic_name: &str) -> Result<()> {}

    /// Starts all listeners
    pub async fn start_all(&mut self) -> Result<()> {}

    /// Stops all listeners
    pub async fn stop_all(&mut self) -> Result<()> {}
}
```

### `vehicle/dds/listener.rs`

```rust
/// DDS topic listener
pub struct TopicListener {
    /// Name of the topic
    topic_name: String,
    /// Data type of the topic
    data_type_name: String,
    /// Channel sender for data
    tx: Sender<DdsData>,
    /// Domain ID for DDS
    domain_id: u32,
    /// Handle to the listener task
    listener_task: Option<JoinHandle<()>>,
    /// Flag indicating if the listener is running
    is_running: bool,
}

impl TopicListener {
    /// Creates a new topic listener
    pub fn new(
        topic_name: String,
        data_type_name: String,
        tx: Sender<DdsData>,
        domain_id: u32,
    ) -> Self {}

    /// Starts listening to the topic
    pub async fn start(&mut self) -> common::Result<()> {}

    /// The main listener loop for processing DDS data
    async fn listener_loop(
        topic_name: String,
        data_type_name: String,
        tx: Sender<DdsData>,
        domain_id: u32,
    ) -> common::Result<()> {}

    /// Stops the listener
    pub async fn stop(&mut self) -> common::Result<()> {}
}

/// Generic DDS Data type for dynamic data handling
#[derive(Debug, PartialEq, DdsType)]
pub struct VehicleData {
    /// Identifier for the data
    #[dust_dds(key)]
    pub id: u32,
    /// Name of the topic
    pub topic_name: String,
    /// JSON-encoded value
    pub value: String,
}

/// Creates a data reader for a specific topic type
async fn create_data_reader(
    participant: &dust_dds::dds_async::domain_participant::DomainParticipantAsync,
    topic_name: &str,
    type_name: &str,
) -> common::Result<()> {}
```

### `idl/mod.rs`

```rust
/// Export IDL functionalities
pub use parser::*;
pub use generator::*;

/// Initialize IDL subsystem
pub fn initialize() -> common::Result<()> {}

/// Load IDL definitions from file
pub fn load_idl(path: &Path) -> common::Result<IdlDefinition> {}
```

### `idl/parser.rs`

```rust
/// Parse IDL file content
pub fn parse(content: &str) -> common::Result<IdlDefinition> {}

/// Validate IDL definition
pub fn validate(definition: &IdlDefinition) -> common::Result<()> {}

/// Extract message types from IDL definition
pub fn extract_message_types(definition: &IdlDefinition) -> Vec<MessageType> {}

/// IDL definition structure
pub struct IdlDefinition {
    /// Namespace of IDL definition
    pub namespace: String,
    /// Message types defined in the IDL
    pub messages: Vec<MessageType>,
    /// Service interfaces defined in the IDL
    pub services: Vec<ServiceDefinition>,
    /// Type definitions in the IDL
    pub types: Vec<TypeDefinition>,
}
```

### `idl/generator.rs`

```rust
/// Generate Rust code from IDL definition
pub fn generate_rust_code(definition: &IdlDefinition, output_path: &Path) -> common::Result<()> {}

/// Generate serialization/deserialization code
pub fn generate_serde_code(definition: &IdlDefinition) -> String {}

/// Generate protocol-specific adapters
pub fn generate_protocol_adapter(definition: &IdlDefinition, protocol: ProtocolType) -> String {}

/// Protocol type enumeration
pub enum ProtocolType {
    /// Data Distribution Service
    DDS,
    /// SOME/IP
    SOMEIP,
}
```

## 4. Reference Information

### 4.1 Related Components

| Component | Interface | Description |
|-----------|-----------|-------------|
| APIServer | gRPC Client | 시나리오 조건 정보 수신 |
| ActionController | gRPC Client | 조건 만족 시 조건 충족 정보 전달 |
| MonitoringServer | 간접 통신 | 데이터 필터링 메트릭 로깅 |

### 4.2 Protocol / API

- **gRPC API**
  - `HandleScenarioRequest`: API Server로부터 시나리오 조건 정보 수신
  - `TriggerAction`: ActionController로 조건 충족 정보 전달

- **IDL 기반 프로토콜**
  - DDS: Data Distribution Service 사양에 따른 발행-구독 방식 데이터 통신
  - SOMEIP: Scalable service-Oriented MiddlewarE over IP 프로토콜 지원
  - IDL: Interface Definition Language를 통한 데이터 및 서비스 정의

### 4.3 Data Structures

- **DdsData**: DDS 프로토콜로부터 수신된 데이터
- **Scenario**: 시나리오 정의 및 조건 정보
- **IdlDefinition**: IDL 정의 및 메타데이터
- **Filter**: 시나리오 조건 평가 및 처리

### 4.4 Reference Materials

- `src/common/src/spec/artifact/scenario.rs` 참고한다.
- `common/proto/filtergateway.proto`의 빌드 결과물을 참고해서 grpc/receiver.rs의 import를 작성한다.
- `common/proto/actioncontroller.proto`의 빌드 결과물을 참고해서 grpc/sender.rs의 import를 작성한다.
- `example/resource/bms-performance.yaml` 은 예제 시나리오 파일이니 참고해서 DDS 토픽을 수신받도록 구독한다.
- 모듈 내부에서 통신하는 채널을 사용할때는 std::sync::mpsc 대신 tokio::sync::mpsc를 사용한다.
- DDS 코드 개발은 test-dds.rs 참고해서 작성한다.
- IDL 기반 프로토콜별 데이터 처리 로직은 해당 프로토콜 스펙에 따라 구현된다:
  - DDS: OMG(Object Management Group) DDS 사양
  - SOMEIP: AUTOSAR SOME/IP 사양
  - IDL: OMG IDL 4.2 사양
- IDL 파싱 및 생성을 위해 `src/tools/idl2rs` 모듈을 참고한다.
