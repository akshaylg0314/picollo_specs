# Filter Gateway

## 1. Introduction

### Major features

Filter Gateway receives vehicle data, evaluates scenario conditions, and when conditions are met, transmits condition fulfillment information to the ActionController.

1. Vehicle Data Reception - Receives vehicle data through IDL-based protocols (DDS, SOMEIP)
2. Data Normalization and Transformation - Converts IDL-based data into internal data structures
3. Scenario Condition Filtering - Filters data according to scenario conditions received from the API Server
4. Condition Evaluation - Evaluates scenario conditions against current vehicle status
5. Data Buffering - Caches data for load management and during network interruptions
6. Condition Fulfillment Notification - Transmits information to ActionController when conditions are met

### Main Dataflow

1. Receives scenario condition information from APIServer through gRPC.
2. Receives vehicle data through IDL-based protocols (DDS, SOMEIP).
3. Buffers received data to manage load.
4. Converts data into internal data structures according to IDL definitions.
5. Filters converted data according to scenario conditions.
6. Evaluates condition fulfillment by comparing vehicle status with scenario conditions.
7. When conditions are met, transmits condition fulfillment information to ActionController via gRPC.
8. Sends performance metrics and filtering statistics to MonitoringServer.

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

- **main.rs** - FilterGateway initialization and execution
- **manager.rs** - Overall module management and data flow control
- **grpc/mod.rs** - gRPC related functionality module
- **grpc/receiver.rs** - Receiving scenario condition information from APIServer
- **grpc/sender.rs** - Transmitting condition fulfillment information to ActionController
- **filter/mod.rs** - Scenario condition filtering and evaluation
- **vehicle/mod.rs** - Vehicle data management module
- **vehicle/dds/mod.rs** - DDS protocol management
- **vehicle/dds/listener.rs** - DDS topic listener implementation
- **idl/mod.rs** - IDL definition and processing related module
- **idl/parser.rs** - IDL file parsing functionality
- **idl/generator.rs** - IDL-based code generation functionality

## 3. Function information

The function information in this section is written in the comment format used in rustdoc.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

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
| APIServer | gRPC Client | Receives scenario condition information |
| ActionController | gRPC Client | Transmits condition fulfillment information when conditions are met |
| MonitoringServer | Indirect Communication | Logs data filtering metrics |

### 4.2 Protocol / API

- **gRPC API**
  - `HandleScenarioRequest`: Receives scenario condition information from API Server
  - `TriggerAction`: Transmits condition fulfillment information to ActionController

- **IDL-based Protocols**
  - DDS: Data communication using publish-subscribe pattern according to Data Distribution Service specifications
  - SOMEIP: Supports Scalable service-Oriented MiddlewarE over IP protocol
  - IDL: Data and service definition through Interface Definition Language

### 4.3 Data Structures

- **DdsData**: Data received from DDS protocol
- **Scenario**: Scenario definition and condition information
- **IdlDefinition**: IDL definition and metadata
- **Filter**: Scenario condition evaluation and processing

### 4.4 Reference Materials

- Refer to `src/common/src/spec/artifact/scenario.rs`.
- Refer to build results of `common/proto/filtergateway.proto` for imports in grpc/receiver.rs.
- Refer to build results of `common/proto/actioncontroller.proto` for imports in grpc/sender.rs.
- Refer to `example/resource/bms-performance.yaml` as an example scenario file to subscribe to DDS topics.
- Use tokio::sync::mpsc instead of std::sync::mpsc for channel communication within modules.
- Refer to test-dds.rs for developing DDS code.
- Data processing logic based on IDL protocols is implemented according to the respective protocol specifications:
  - DDS: OMG (Object Management Group) DDS specifications
  - SOMEIP: AUTOSAR SOME/IP specifications
  - IDL: OMG IDL 4.2 specifications
- Refer to the `src/tools/idl2rs` module for IDL parsing and generation.
