# Policy Manager

## 1. Introduction

### Major features

Policy Manager is a core component in the PICCOLO framework that determines whether a service should be allowed, put in pending state, or denied execution.

1. Loads policy data from ETCD and performs policy evaluation.
2. Calculates priorities based on vehicle state and service information received from ActionController.
3. Makes service execution decisions (allow, pending, deny) according to calculated priorities.
4. Dynamically adjusts service priorities based on vehicle state (driving, parked, etc.).
5. Dynamically adjusts service priorities based on autonomous driving level.

### Main Dataflow

1. Loads and caches policy data for all services from ETCD at startup.
2. Receives policy evaluation requests (service_id, action_type, current vehicle state, etc.) from ActionController.
3. Calculates priorities by combining policy data loaded from ETCD with dynamic data received from ActionController.
4. Compares calculated priorities with thresholds to make allow, pending, or deny decisions.
5. Returns policy evaluation results to ActionController.

## 2. File information

```text
policymanager
├── policymanager.md
├── Cargo.toml
└── src
    ├── etcd
    │   ├── client.rs
    │   └── mod.rs
    ├── grpc
    │   ├── client
    │   │   ├── action_controller.rs
    │   │   └── mod.rs
    │   ├── mod.rs
    │   └── server
    │       ├── mod.rs
    │       └── service.rs
    ├── main.rs
    ├── manager.rs
    ├── policy
    │   ├── evaluator.rs
    │   ├── mod.rs
    │   └── types.rs
    └── priority
        ├── calculator.rs
        └── mod.rs
```

- **main.rs** - PolicyManager initialization and execution
- **manager.rs** - Controls data flow between modules, loads policies from ETCD and processes ActionController requests
- **etcd/mod.rs** - Manages ETCD integration (policy data loading)
- **etcd/client.rs** - Implements ETCD client (policy querying and caching)
- **grpc/mod.rs** - gRPC communication management
- **grpc/client/mod.rs** - gRPC client management
- **grpc/client/action_controller.rs** - gRPC communication with ActionController (handling requests and responses)
- **grpc/server/mod.rs** - gRPC server management
- **grpc/server/service.rs** - PolicyManager gRPC service implementation (policy evaluation API)
- **policy/mod.rs** - Core policy management module
- **policy/evaluator.rs** - Policy condition evaluation (based on vehicle state and action requests)
- **policy/types.rs** - Policy and decision type definitions (allow/pending/deny etc.)
- **priority/mod.rs** - Priority management module
- **priority/calculator.rs** - Service priority calculation (based on policies stored in ETCD)

## 3. Function information

The function information in this section is written in the format used by rustdoc comments.
For more information, refer to [this link](https://doc.rust-lang.org/stable/rustdoc/index.html).

### `main.rs`

```rust
/// Main function of Piccolo Policy Manager
#[tokio::main]
async fn main() {}
```

### `manager.rs`

```rust
/// Initialize and start the PolicyManager
pub async fn initialize() {}

/// Start policy evaluation service
pub async fn start_service() {}

/// Stop policy evaluation service
pub async fn stop_service() {}

/// Evaluate policy for a given action request
///
/// ### Parameters
/// * `request: PolicyRequest` - The policy evaluation request from ActionController
/// ### Returns
/// * `PolicyResponse` - Policy evaluation response (ALLOW, DENY, PENDING)
pub async fn evaluate_policy(request: PolicyRequest) -> PolicyResponse {}

/// Load policies from ETCD
///
/// ### Returns
/// * `Result<(), Error>` - Success or failure of policy loading
pub async fn load_policies_from_etcd() -> Result<(), Error> {}
```

### `etcd/client.rs`

```rust
/// ETCD Client for policy data
pub struct EtcdClient {
    client: Client,
    policy_cache: Arc<RwLock<HashMap<String, PolicyData>>>,
}

/// Create a new ETCD client
///
/// ### Parameters
/// * `endpoints: &[String]` - ETCD server endpoints
/// ### Returns
/// * `Result<EtcdClient, Error>` - New ETCD client or error
pub async fn new(endpoints: &[String]) -> Result<EtcdClient, Error> {}

/// Load policy data from ETCD
///
/// ### Returns
/// * `Result<HashMap<String, PolicyData>, Error>` - Policy data or error
pub async fn load_policies(&self) -> Result<HashMap<String, PolicyData>, Error> {}

/// Get policy data for a specific service
///
/// ### Parameters
/// * `service_id: &str` - Service identifier
/// ### Returns
/// * `Option<PolicyData>` - Policy data if found
pub async fn get_policy(&self, service_id: &str) -> Option<PolicyData> {}
```

### `policy/evaluator.rs`

```rust
/// Policy evaluator for service requests
pub struct PolicyEvaluator {
    policy_data: Arc<RwLock<HashMap<String, PolicyData>>>,
}

/// Create a new policy evaluator
///
/// ### Parameters
/// * `policy_data: Arc<RwLock<HashMap<String, PolicyData>>>` - Policy data from ETCD
/// ### Returns
/// * `PolicyEvaluator` - New policy evaluator instance
pub fn new(policy_data: Arc<RwLock<HashMap<String, PolicyData>>>) -> PolicyEvaluator {}

/// Evaluate policy for service request
///
/// ### Parameters
/// * `request: &PolicyRequest` - The policy request from ActionController
/// ### Returns
/// * `PolicyDecision` - Policy decision (ALLOW, PENDING, DENY)
pub async fn evaluate(&self, request: &PolicyRequest) -> PolicyDecision {}

/// Check if action is allowed in current vehicle state
///
/// ### Parameters
/// * `service_type: ServiceType` - Type of service
/// * `vehicle_state: &VehicleState` - Current vehicle state
/// * `action_type: ActionType` - Type of action
/// ### Returns
/// * `bool` - Whether action is allowed
pub fn is_action_allowed(&self, service_type: ServiceType, vehicle_state: &VehicleState, action_type: ActionType) -> bool {}
```

### `policy/types.rs`

```rust
/// Policy decision enum
pub enum PolicyDecision {
    /// Allow the service to run
    Allow,
    /// Put the service request on hold
    Pending(u32), // with retry after seconds
    /// Deny the service request
    Deny(String), // with reason
}

/// Service type enum
pub enum ServiceType {
    SafetyCritical,
    Adas,
    Hud,
    Navigation,
    Cluster,
    Alert,
    VoiceControl,
    Diagnostics,
    Media,
    Comfort,
    Infotainment,
    Connectivity,
    Background,
    Update,
}

/// Policy request from ActionController
pub struct PolicyRequest {
    pub request_id: String,
    pub service_id: String,
    pub service_type: String,
    pub action_type: String,
    pub vehicle_state: VehicleState,
    pub required_devices: Vec<String>,
}

/// Policy response to ActionController
pub struct PolicyResponse {
    pub request_id: String,
    pub decision: PolicyDecision,
    pub calculated_priority: i32,
}

/// Policy data stored in ETCD
pub struct PolicyData {
    pub service_id: String,
    pub service_type: ServiceType,
    pub base_priority: i32,
    pub asil_level: String,
    pub required_devices: Vec<String>,
    pub state_adjustments: HashMap<String, i32>,
    pub autonomy_adjustments: HashMap<String, i32>,
}
```

### `priority/calculator.rs`

```rust
/// Priority calculator for services
pub struct PriorityCalculator {
    policy_data: Arc<RwLock<HashMap<String, PolicyData>>>,
}

/// Create a new priority calculator
///
/// ### Parameters
/// * `policy_data: Arc<RwLock<HashMap<String, PolicyData>>>` - Policy data from ETCD
/// ### Returns
/// * `PriorityCalculator` - New priority calculator instance
pub fn new(policy_data: Arc<RwLock<HashMap<String, PolicyData>>>) -> PriorityCalculator {}

/// Calculate final priority for a service
///
/// ### Parameters
/// * `service_id: &str` - Service identifier
/// * `vehicle_state: &VehicleState` - Current vehicle state
/// ### Returns
/// * `i32` - Calculated priority value (0-100)
pub fn calculate_final_priority(&self, service_id: &str, vehicle_state: &VehicleState) -> i32 {}

/// Get decision based on calculated priority
///
/// ### Parameters
/// * `priority: i32` - Calculated priority
/// ### Returns
/// * `PolicyDecision` - Policy decision (ALLOW, PENDING, DENY)
pub fn get_decision(&self, priority: i32) -> PolicyDecision {
    if priority >= 70 {
        PolicyDecision::Allow
    } else if priority >= 40 {
        PolicyDecision::Pending(30) // Retry after 30 seconds
    } else {
        PolicyDecision::Deny("Insufficient priority".to_string())
    }
}
```

## 4. Reference Information

### 4.1 Priority Calculation System

#### 4.1.1 Base Service Priority

Base priorities by service type (range 1-100, higher is more important):

| Service Type | Base Priority | Description |
|------------|--------------|------|
| safety-critical | 90 | Safety critical services (brake control, steering control) |
| adas | 85 | Advanced Driver Assistance Systems |
| hud | 80 | Head-Up Display |
| navigation | 75 | Navigation |
| cluster | 70 | Instrument Cluster |
| alert | 65 | Warnings and notifications |
| voice-control | 60 | Voice control |
| diagnostics | 55 | Vehicle diagnostics |
| media | 50 | Media (audio/video) |
| comfort | 45 | Comfort features (temperature control, seat adjustment) |
| infotainment | 40 | Infotainment |
| connectivity | 35 | Connectivity services |
| background | 30 | Background services |
| update | 25 | Software updates |

#### 4.1.2 Vehicle State Priority Adjustments

Priority adjustment values by service type and driving state:

| Service Type | Poweroff | Ignition | Parking | Driving | Neutral | Reverse |
|------------|----------|----------|---------|---------|---------|---------|
| safety-critical | -50 | +10 | 0 | +10 | +5 | +10 |
| adas | -90 | +5 | -10 | +15 | +10 | +15 |
| hud | -90 | +5 | -20 | +15 | +10 | +5 |
| navigation | -90 | 0 | +5 | +15 | +10 | -5 |
| cluster | -90 | +10 | -5 | +10 | +5 | +5 |
| alert | -50 | +10 | +5 | +10 | +5 | +10 |
| voice-control | -90 | -10 | +10 | +5 | +5 | -5 |
| diagnostics | +10 | +15 | +15 | -20 | 0 | -10 |
| media | -90 | -20 | +15 | -5 | +5 | -15 |
| comfort | -90 | 0 | +10 | 0 | +5 | -5 |
| infotainment | -90 | -20 | +15 | -10 | +5 | -15 |
| connectivity | -50 | +5 | +10 | -5 | +5 | -10 |
| background | +10 | +5 | +5 | -20 | 0 | -15 |
| update | +20 | -10 | +20 | -80 | -10 | -80 |

#### 4.1.3 ASIL Safety Level Priority Adjustments

Priority adjustments based on safety criticality:

| ASIL Level | Priority Adjustment | Description |
|----------|--------------|------|
| QM | 0 | Quality Managed |
| A | +5 | ASIL A |
| B | +10 | ASIL B |
| C | +15 | ASIL C |
| D | +20 | ASIL D (highest safety level) |

#### 4.1.4 Autonomous Driving Level Priority Adjustments

Priority adjustments for key services based on autonomous driving level:

| Service Type | Level 1 | Level 2 | Level 3 | Level 4 | Level 5 |
|------------|-------|-------|-------|-------|-------|
| safety-critical | +5 | +10 | +15 | +20 | +25 |
| adas | +10 | +15 | +20 | +25 | +25 |
| hud | +10 | +15 | +10 | +5 | - |
| navigation | +5 | +10 | +5 | +15 | +20 |
| alert | +10 | +15 | +15 | +15 | +15 |
| media | 0 | -10 | +5 | +10 | +15 |
| infotainment | -5 | -15 | 0 | +5 | +15 |
| update | -15 | -25 | -30 | -10 | 0 |

#### 4.1.5 Device Conflict Policy

Policy for determining pending or denial of services during resource conflicts:

| Device | Conflict Policy | Timeout (seconds) | Retry Count | Additional Info |
|---------|------------|------------|-----------|---------|
| audio | pending | 30 | 3 | |
| gpu | pending | 60 | 5 | |
| speaker | pending | 15 | 2 | |
| mic | deny | - | - | Privacy-sensitive resource |
| display | pending | 45 | 3 | |
| npu | pending | 120 | 10 | |

### 4.2 PolicyManager gRPC Interface

```proto
service PolicyManagerService {
  // Policy evaluation - called from ActionController
  rpc EvaluatePolicy(PolicyRequest) returns (PolicyResponse);
}

message PolicyRequest {
  string request_id = 1;
  string service_id = 2;
  string service_type = 3;
  string action_type = 4;
  
  // Dynamic data (passed from ActionController)
  VehicleState vehicle_state = 5;
  repeated string required_devices = 6;
}

message VehicleState {
  string ignition_state = 1;
  float vehicle_speed = 2;
  string driving_mode = 3;
  string autonomy_level = 4;
  string power_state = 5;
}

message PolicyResponse {
  string request_id = 1;
  PolicyDecision decision = 2;
  string reason = 3;
  int32 calculated_priority = 4;
  int32 retry_after_seconds = 5;
}

enum PolicyDecision {
  ALLOW = 0;
  PENDING = 1;
  DENY = 2;
}
```

### 4.3 Decision Logic and Thresholds

#### 4.3.1 Final Priority Calculation

PolicyManager calculates priorities based on policy data loaded from ETCD:

```
Final Priority = Base Priority + Vehicle State Adjustment + Autonomy Level Adjustment
```

- Base Priority: Default value for each service type loaded from ETCD
- Vehicle State Adjustment: Adjustment based on current vehicle state (driving, parked, etc.)
- Autonomy Level Adjustment: Adjustment based on current autonomous driving level

#### 4.3.2 Decision Thresholds

| Decision | Threshold |
|------|--------|
| Allow | ≥ 70 |
| Pending | ≥ 40, < 70 |
| Deny | < 40 |

#### 4.3.3 Special Rules

1. Safety-critical services are always allowed while driving
2. Software updates are always denied while driving

### 4.4 Policy Data Structure in ETCD

Policy data is stored in ETCD with the following structure:

```json
{
  "service_id": "navigation-service",
  "service_type": "navigation",
  "base_priority": 75,
  "required_devices": ["display", "gpu", "speaker"],
  "state_adjustments": {
    "driving": 15,
    "parking": 5,
    "poweroff": -90,
    "ignition": 0,
    "neutral": 10,
    "reverse": -5
  },
  "autonomy_adjustments": {
    "level1": 5,
    "level2": 10,
    "level3": 5,
    "level4": 15,
    "level5": 20
  }
}
```

This policy data is evaluated alongside dynamic data provided by ActionController (vehicle state, required devices, etc.) to make the final decision.
