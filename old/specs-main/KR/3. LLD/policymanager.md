# Policy Manager

## 1. Introduction

### Major features

Policy Manager는 PICCOLO 프레임워크 내에서 서비스의 실행 허용(allow), 대기(pending), 거부(deny) 여부를 판단하는 핵심 컴포넌트이다.

1. ETCD에서 정책 데이터를 로드하여 정책 평가를 수행한다.
2. ActionController로부터 받은 차량 상태와 서비스 정보를 기반으로 우선순위를 계산한다.
3. 계산된 우선순위에 따라 서비스 실행 허용, 대기, 거부 결정을 내린다.
4. 차량 상태(주행 중, 주차됨 등)에 따라 서비스 우선순위를 동적으로 조정한다.
5. 자율주행 레벨에 따른 서비스 우선순위를 동적으로 조정한다.

### Main Dataflow

1. 시작 시 ETCD에서 모든 서비스의 정책 데이터를 로드하고 캐싱한다.
2. ActionController로부터 정책 평가 요청(service_id, action_type, 현재 차량 상태 등)을 수신한다.
3. ETCD에서 로드된 정책 데이터와 ActionController에서 받은 동적 데이터를 조합하여 우선순위를 계산한다.
4. 계산된 우선순위와 임계값을 비교하여 허용(allow), 대기(pending), 거부(deny) 결정을 내린다.
5. 정책 평가 결과를 ActionController에게 반환한다.

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

- **main.rs** - PolicyManager 초기화 및 실행
- **manager.rs** - 각 모듈 간의 데이터 흐름 제어, ETCD에서 정책 로드 및 ActionController 요청 처리
- **etcd/mod.rs** - etcd 연동 관리 (정책 데이터 로드)
- **etcd/client.rs** - etcd 클라이언트 구현 (정책 조회 및 캐싱)
- **grpc/mod.rs** - gRPC 통신 관리
- **grpc/client/mod.rs** - gRPC 클라이언트 관리
- **grpc/client/action_controller.rs** - ActionController와의 gRPC 통신 (요청 처리 및 응답)
- **grpc/server/mod.rs** - gRPC 서버 관리
- **grpc/server/service.rs** - PolicyManager gRPC 서비스 구현 (정책 평가 API)
- **policy/mod.rs** - 정책 관리 핵심 모듈
- **policy/evaluator.rs** - 정책 조건 평가 (차량 상태 및 액션 요청에 따른 정책 평가)
- **policy/types.rs** - 정책 및 결정 관련 타입 정의 (허용/대기/거부 등)
- **priority/mod.rs** - 우선순위 관리 모듈
- **priority/calculator.rs** - 서비스 우선순위 계산 (ETCD에 저장된 정책 기반)

## 3. Function information

본 문단의 function 정보는 rustdoc 에서 사용하는 주석 형태로 작성되었다.
이에 대해서는 [링크](https://doc.rust-lang.org/stable/rustdoc/index.html) 를 참조하라.

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

### 4.1 우선순위 계산 시스템

#### 4.1.1 기본 서비스 우선순위

서비스 타입별 기본 우선순위(1-100 범위, 높을수록 우선순위가 높음):

| 서비스 유형 | 기본 우선순위 | 설명 |
|------------|--------------|------|
| safety-critical | 90 | 안전 필수 서비스 (브레이크 제어, 조향 제어) |
| adas | 85 | 운전자 지원 시스템 |
| hud | 80 | 헤드업 디스플레이 |
| navigation | 75 | 내비게이션 |
| cluster | 70 | 계기판 |
| alert | 65 | 경고 및 알림 |
| voice-control | 60 | 음성 제어 |
| diagnostics | 55 | 차량 진단 |
| media | 50 | 미디어 (오디오/비디오) |
| comfort | 45 | 편의 기능 (온도 조절, 시트 조절) |
| infotainment | 40 | 인포테인먼트 |
| connectivity | 35 | 연결성 서비스 |
| background | 30 | 백그라운드 서비스 |
| update | 25 | 소프트웨어 업데이트 |

#### 4.1.2 차량 상태별 우선순위 조정

주행 상태에 따른 서비스 타입별 우선순위 조정값:

| 서비스 유형 | Poweroff | Ignition | Parking | Driving | Neutral | Reverse |
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

#### 4.1.3 ASIL 안전 등급별 우선순위 조정

안전 중요도에 따른 우선순위 조정값:

| ASIL 등급 | 우선순위 조정 | 설명 |
|----------|--------------|------|
| QM | 0 | 품질 관리 (Quality Managed) |
| A | +5 | ASIL A |
| B | +10 | ASIL B |
| C | +15 | ASIL C |
| D | +20 | ASIL D (최고 안전 등급) |

#### 4.1.4 자율주행 레벨별 우선순위 조정

자율주행 레벨에 따른 주요 서비스 우선순위 조정값:

| 서비스 유형 | 레벨 1 | 레벨 2 | 레벨 3 | 레벨 4 | 레벨 5 |
|------------|-------|-------|-------|-------|-------|
| safety-critical | +5 | +10 | +15 | +20 | +25 |
| adas | +10 | +15 | +20 | +25 | +25 |
| hud | +10 | +15 | +10 | +5 | - |
| navigation | +5 | +10 | +5 | +15 | +20 |
| alert | +10 | +15 | +15 | +15 | +15 |
| media | 0 | -10 | +5 | +10 | +15 |
| infotainment | -5 | -15 | 0 | +5 | +15 |
| update | -15 | -25 | -30 | -10 | 0 |

#### 4.1.5 디바이스 충돌 정책

리소스 충돌 시 서비스의 대기 또는 거부 여부를 결정하는 정책:

| 디바이스 | 충돌 시 정책 | 타임아웃(초) | 재시도 횟수 | 추가 정보 |
|---------|------------|------------|-----------|---------|
| audio | pending | 30 | 3 | |
| gpu | pending | 60 | 5 | |
| speaker | pending | 15 | 2 | |
| mic | deny | - | - | 개인정보 민감 리소스 |
| display | pending | 45 | 3 | |
| npu | pending | 120 | 10 | |

### 4.2 PolicyManager gRPC 인터페이스

```proto
service PolicyManagerService {
  // 정책 평가 - ActionController에서 호출
  rpc EvaluatePolicy(PolicyRequest) returns (PolicyResponse);
}

message PolicyRequest {
  string request_id = 1;
  string service_id = 2;
  string service_type = 3;
  string action_type = 4;
  
  // 동적 데이터 (ActionController에서 전달)
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

### 4.3 결정 로직 및 임계값

#### 4.3.1 최종 우선순위 계산

PolicyManager는 ETCD에서 읽어온 정책 데이터를 기반으로 우선순위를 계산합니다:

```
최종 우선순위 = 기본 우선순위 + 차량 상태 조정 + 자율주행 레벨 조정
```

- 기본 우선순위: ETCD에서 로드된 서비스 유형별 기본값
- 차량 상태 조정: 현재 차량 상태(주행 중, 주차됨 등)에 따른 조정값
- 자율주행 레벨 조정: 현재 자율주행 레벨에 따른 조정값

#### 4.3.2 결정 임계값

| 결정 | 임계값 |
|------|--------|
| 허용 (allow) | ≥ 70 |
| 대기 (pending) | ≥ 40, < 70 |
| 거부 (deny) | < 40 |

#### 4.3.3 특별 규칙

1. 안전 중요 서비스(safety-critical)는 주행 중(driving) 항상 허용됨
2. 소프트웨어 업데이트(update)는 주행 중(driving) 항상 거부됨

### 4.4 ETCD에 저장되는 정책 데이터 구조

정책 데이터는 다음과 같은 구조로 ETCD에 저장됩니다:

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

이러한 정책 데이터는 ActionController에서 제공하는 동적 데이터(차량 상태, 필요한 디바이스 등)와 함께 평가되어 최종 결정을 내립니다.
