# PICCOLO 리소스 명세서

**문서 번호**: PICCOLO-RESOURCES-2025-001  
**버전**: 1.0  
**날짜**: 2025-08-04  
**작성자**: joshua-jung_LGESDV  
**분류**: 리소스 명세서  

## 1. 개요

본 문서는 PICCOLO 프레임워크에서 사용되는 주요 리소스 타입 (Scenario, Package, Model, Volume, Network, Node)의 명세와 상태 정의를 기술합니다. 각 리소스 타입은 차량 환경에서의 서비스 배포, 실행 및 관리를 위한 구성 요소로 사용됩니다.

## 2. Scenario 리소스

### 2.1 Scenario 정의

Scenario는 차량 상태 조건에 따라 특정 패키지에 대한 액션을 정의합니다. 차량의 특정 상태나 이벤트가 발생했을 때 어떤 서비스를 어떻게 관리할지 결정하는 규칙입니다.

#### 2.1.1 Scenario 구성 요소

| 구성 요소 | 설명 |
|---------|------|
| condition | 시나리오가 적용되는 차량 상태 조건 (예: 차량 속도, 점화 상태, 배터리 레벨 등) |
| action | 조건 충족 시 실행할 작업 (launch, update, modify, notify, rollback, pause, resume) |
| target | 작업의 대상이 되는 패키지 |

#### 2.1.2 Scenario 액션 유형

| 액션 유형 | 설명 | 주요 사용 사례 |
|----------|------|---------------|
| launch | 패키지 시작 및 배포 | 특정 상태에서 서비스 활성화 필요 시 |
| update | 패키지 업데이트 | 패키지 버전 업그레이드 필요 시 |
| modify | 패키지 구성 수정 | 실행 중인 서비스 설정 변경 필요 시 |
| notify | 패키지 상태 알림 | 특정 상태 변경 시 알림 전송 |
| rollback | 이전 버전으로 복구 | 업데이트 실패 또는 문제 발생 시 |
| pause | 패키지 일시 중지 | 임시 리소스 확보 또는 다른 작업 우선 처리 시 |
| resume | 일시 중지된 패키지 재개 | 일시 중지된 서비스 복원 시 |

#### 2.1.3 Scenario Status

| 상태 | 설명 |
|------|------|
| idle | 시나리오 준비 상태 (활성화되지 않음) |
| waiting | 조건이 충족되기를 기다리는 상태 |
| playing | 시나리오 액션이 실행 중인 상태 |
| error | 시나리오 실행 중 오류 발생 상태 |
| allowed | 정책에 의해 실행이 허용된 상태 |
| denied | 정책에 의해 실행이 거부된 상태 |

#### 2.1.4 Scenario YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: adas-activation
  annotations:
    description: "Activates ADAS features during driving"
    priority: "high"
  labels:
    function: "driver-assistance"
    category: "safety"
    asilLevel: "B"
spec:
  condition:
    vehicleSpeed: ">30"
    ignitionState: "ON"
    driverSeatOccupied: "true"
  action: launch
  target:
    packages:
      - adas-software
      - vision-processing
      - sensor-processing
status:
  currentState: playing
  policyStatus: allowed
  lastTransitionTime: "2025-08-03T14:25:32Z"
  message: "ADAS services activated successfully"
  conditions:
    - type: ConditionsMet
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:30Z"
    - type: PolicyVerified
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:31Z"
    - type: ActionExecuted
      status: "True"
      lastUpdateTime: "2025-08-03T14:25:32Z"
  activePackages:
    - name: adas-software
      status: running
    - name: vision-processing
      status: running
    - name: sensor-processing
      status: running
```

## 3. Package 리소스

### 3.1 Package 정의

Package는 함께 배포되고 관리되는 Model들의 그룹입니다. 패키지는 특정 기능을 제공하기 위해 함께 작동하는 여러 컴포넌트를 논리적으로 그룹화하는 단위입니다.

#### 3.1.1 Package 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| pattern | 배포 패턴 (plain, distributed, selector) |
| models | 패키지에 포함된 모델(컨테이너) 목록 |
| types | 패키지 타입 리스트 (복수 지정 가능: plain, redundant, update, time-critical, stateful, resource-bounded, secure) |

#### 3.1.2 Package 타입

| 패키지 타입 | 설명 | 특성 |
|------------|------|------|
| plain | 기본 컨테이너 운영 | 단순한 배포 및 실행을 위한 기본 타입 |
| redundant | 고가용성 보장 | 이중화 운영, 장애 감지 및 자동 복구 기능 |
| update | 안정적인 업데이트 보장 | 정의된 정책에 따른 업데이트, 실패 시 자동 롤백 |
| time-critical | 정확한 시간 인터벌 보장 | 정밀한 타이밍 제어, 리소스 우선 할당, 실시간 스케줄링 |
| stateful | 상태 정보 유지 및 관리 | 재시작/재배치 시에도 상태 보존, 영구 스토리지 자동 관리 |
| resource-bounded | 엄격한 리소스 제약 준수 | 명확한 리소스 할당 및 제한, 리소스 모니터링 및 조절 |
| secure | 강화된 보안 요구사항 충족 | 격리된 네트워크 환경, 암호화된 스토리지, 보안 정책 적용 |

#### 3.1.3 Package Status

| 상태 | 설명 |
|------|------|
| initializing | 패키지 초기화 중인 상태 |
| running | 패키지가 정상 작동 중인 상태 |
| degraded | 패키지가 일부 기능 저하된 상태로 작동 중 |
| error | 패키지 오류 발생 상태 |

#### 3.1.4 Package YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: vision-processing
  annotations:
    version: "2.3"
    description: "카메라 및 영상 처리 시스템"
  labels:
    types: ["resource-bounded", "time-critical"]
    component: "vision"
    asilLevel: "B"
spec:
  pattern:
    type: selector
    nodeSelector:
      gpu: "available"
  models:
    - name: vision-system
      image: vehicle/vision-system:2.3
      node: VisionHPC
      resources:
        requests:
          cpu: 2
          memory: 2Gi
          gpu: 1
        limits:
          cpu: 4
          memory: 4Gi
          gpu: 1
        volume: vision-data-volume
        network: sensor-network
    - name: object-detection
      image: vehicle/object-detection:1.5
      node: VisionHPC
      resources:
        requests:
          cpu: 3
          memory: 4Gi
          gpu: 1
        limits:
          cpu: 6
          memory: 8Gi
          gpu: 1
        volume: vision-data-volume
        network: sensor-network
status:
  currentState: running
  lastTransitionTime: "2025-08-03T14:25:35Z"
  message: "All models running normally"
  models:
    - name: vision-system
      status: running
      restarts: 0
      readiness: true
    - name: object-detection
      status: running
      restarts: 0
      readiness: true
  resources:
    cpuUsage: "65%"
    memoryUsage: "48%"
    gpuUsage: "72%"
```

## 4. Model 리소스

### 4.1 Model 정의

Model은 Pod과 동일한 개념으로, 하나 이상의 컨테이너로 구성된 배포 단위입니다. Model은 실제 서비스를 제공하는 가장 기본적인 실행 단위입니다.

#### 4.1.1 Model 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| containers | 모델에 포함된 컨테이너 정의 |
| resources | 필요한 컴퓨팅 리소스 (CPU, 메모리, GPU 등) |
| volumes | 마운트할 볼륨 |
| networks | 연결할 네트워크 |
| scheduling | 스케줄링 관련 설정 (우선순위, 실시간 요구사항 등) |

#### 4.1.2 Model Status (Pod Status와 동일)

| 상태 | 설명 |
|------|------|
| Pending | 모델이 노드에 할당되기를 기다리는 상태 |
| Running | 모델이 정상적으로 실행 중인 상태 |
| Succeeded | 모델의 모든 컨테이너가 성공적으로 종료된 상태 |
| Failed | 하나 이상의 컨테이너가 실패한 상태 |
| Unknown | 모델의 상태를 확인할 수 없는 상태 |

#### 4.1.3 Model YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Model
metadata:
  name: brake-controller-model
  annotations:
    io.piccolo.annotations.package-type: safety-critical-v1.0
  labels:
    app: brake-controller
    safetyLevel: critical
    asilLevel: "D"
spec:
  hostNetwork: false
  containers:
    - name: brake-controller
      image: sdv.lge.com/vehicle/brake-controller:1.2
      resources:
        limits:
          cpu: 1
          memory: 1Gi
  scheduling:
    timeCritical: true
    priority: 95
    period: "10ms"
    deadline: "8ms"
    executionTime: "2ms"
    cpuAffinity: [2, 3]
    isolationLevel: "exclusive"
    criticality: "safety-critical"
  deviceAccess:
    - manager: vehicle-sensors
      devices: ["brake-sensor", "wheel-speed-sensor"]
  volumes:
    - name: brake-data
      mountPath: /data
      volume: brake-data-volume
  networks:
    - name: control-network
status:
  phase: Running
  conditions:
    - type: Ready
      status: "True"
      lastProbeTime: "2025-08-03T14:25:40Z"
  containerStatuses:
    - name: brake-controller
      state:
        running:
          startedAt: "2025-08-03T14:25:38Z"
      ready: true
      restartCount: 0
      image: sdv.lge.com/vehicle/brake-controller:1.2
      imageID: sha256:a1b2c3d4...
  startTime: "2025-08-03T14:25:35Z"
  hostIP: "192.168.1.10"
  podIP: "10.0.0.5"
```

## 5. Volume 리소스

### 5.1 Volume 정의

Volume은 컨테이너에 마운트되어 영구적인 데이터 저장을 제공하는 스토리지 리소스입니다.

#### 5.1.1 Volume 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| size | 볼륨 크기 |
| accessModes | 접근 모드 (ReadWriteOnce, ReadOnlyMany, ReadWriteMany) |
| storageClass | 스토리지 클래스 |
| reclaimPolicy | 재사용 정책 |

#### 5.1.2 Volume YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Volume
metadata:
  name: vision-data-volume
  annotations:
    backup: "enabled"
  labels:
    application: "adas"
spec:
  size: 10Gi
  accessModes:
    - ReadWriteMany
  storageClass: "fast-ssd"
  persistentVolumeReclaimPolicy: Retain
  backup:
    schedule: "0 0 * * *"
    retention: 7
status:
  phase: Bound
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  claimRef:
    kind: Model
    name: vision-system
    namespace: default
  message: ""
```

## 6. Network 리소스

### 6.1 Network 정의

Network는 컨테이너 간 통신을 위한 네트워크 리소스입니다. QoS 관리, 보안 설정, 최적화 등을 정의합니다.

#### 6.1.1 Network 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| bridge | 네트워크 브릿지 설정 |
| ingress/egress | 인그레스/이그레스 포트 및 프로토콜 설정 |
| qos | 서비스 품질 설정 |
| security | 보안 설정 |

#### 6.1.2 Network YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Network
metadata:
  name: sensor-network
  labels:
    app: "adas"
spec:
  node_name: "VisionHPC"
  bridge:
    name: "br0"
    type: "bridge"
  ingress:
    port: "5555"
    protocol: "udp"
  egress:
    port: "5556"
    protocol: "udp"
  qos:
    latency: "ultra-low"
    priority: "high"
  fastPath:
    enabled: true
    optimizationLevel: "aggressive"
  security:
    securityPolicy: "strict"
    encryption: true
status:
  state: active
  connectedModels: 5
  bandwidthUsage: "45Mbps"
  packetLoss: "0.01%"
  averageLatency: "1.2ms"
```

## 7. Node 리소스

### 7.1 Node 정의

Node는 컨테이너가 실행되는 물리적 또는 가상의 컴퓨팅 리소스입니다. 차량에서는 일반적으로 다양한 ECU나 컴퓨팅 하드웨어를 나타냅니다.

#### 7.1.1 Node 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| capacity | 노드가 제공하는 리소스 용량 (CPU, 메모리, GPU 등) |
| conditions | 노드의 상태 조건 |
| addresses | 노드의 네트워크 주소 |
| nodeInfo | 노드의 시스템 정보 |

#### 7.1.2 Node YAML 예시

```yaml
apiVersion: piccolo.io/v1
kind: Node
metadata:
  name: VisionHPC
  labels:
    role: "vision-processing"
    gpu: "available"
    zone: "front"
spec:
  unschedulable: false
  taints:
    - key: "dedicated"
      value: "vision"
      effect: "NoSchedule"
  podCIDR: "10.0.0.0/24"
  powerManagement:
    mode: "performance"
status:
  capacity:
    cpu: "8"
    memory: "16Gi"
    gpu: "2"
    pods: "32"
  allocatable:
    cpu: "7"
    memory: "14Gi"
    gpu: "2"
    pods: "28"
  conditions:
    - type: Ready
      status: "True"
      lastHeartbeatTime: "2025-08-04T08:30:45Z"
      lastTransitionTime: "2025-08-01T06:20:10Z"
      reason: "KubeletReady"
      message: "kubelet is posting ready status"
  addresses:
    - type: InternalIP
      address: "192.168.1.10"
    - type: Hostname
      address: "VisionHPC"
  nodeInfo:
    machineID: "abc123def456"
    systemUUID: "ABF23CD4-5678-90EF-GHIJ-KLMNOPQRSTUV"
    bootID: "zyx987wvu654"
    kernelVersion: "5.15.0-rt10"
    osImage: "Automotive Linux 8.5"
    containerRuntimeVersion: "podman://4.5.1"
    kubeletVersion: "v1.27.0"
    kubeProxyVersion: "v1.27.0"
    operatingSystem: "linux"
    architecture: "amd64"
```

## 8. ASIL 서비스 운영 고려사항

ASIL(Automotive Safety Integrity Level) 서비스 운영을 위한 Scenario 액션 고려사항:

### 8.1 ASIL 등급별 액션 제약사항

| ASIL 등급 | 실시간 제약 | 리소스 요구사항 | 허용 액션 | 제한 액션 |
|----------|------------|--------------|----------|---------|
| ASIL D | 엄격한 실시간성 보장 | 전용 리소스 필요 | launch, modify(제한적), rollback | update, pause |
| ASIL C | 준실시간성 보장 | 높은 우선순위 리소스 | launch, modify, rollback | pause(장시간) |
| ASIL B | 일반 실시간성 | 충분한 리소스 할당 | launch, update, modify, rollback, pause(짧은 시간) | - |
| ASIL A | 낮은 실시간성 | 표준 리소스 할당 | 모든 액션 허용 | - |
| QM | 실시간성 불필요 | 표준 리소스 할당 | 모든 액션 허용 | - |

### 8.2 ASIL 서비스를 위한 패키지 타입 권장사항

| ASIL 등급 | 권장 패키지 타입 | 비고 |
|----------|----------------|------|
| ASIL D | time-critical + redundant | 가장 엄격한 안전 요구사항, 결정적 실행 시간과 이중화 필수 |
| ASIL C | time-critical + redundant 또는 둘 중 하나 | 높은 안전 요구사항, 상황에 따라 결정적 실행 및 이중화 모두 또는 개별 적용 |
| ASIL B | time-critical 또는 resource-bounded | 중간 수준의 안전 요구사항, 실시간성 또는 리소스 격리 필요 |
| ASIL A | resource-bounded 또는 secure | 낮은 안전 요구사항, 리소스 보장 또는 보안 강화 필요 |
| QM | plain 또는 update | 일반적인 품질 관리 수준, 기본 패키지 타입으로 충분 |

### 8.3 ASIL 서비스 Scenario 예시

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: emergency-brake-activation
  annotations:
    description: "Emergency brake system activation"
    priority: "critical"
    safetyLevel: "ASIL-D"
  labels:
    function: "safety-critical"
    category: "braking"
spec:
  condition:
    vehicleStatus: "MOVING"
    emergencyBrakeRequest: "ACTIVE"
  action: launch
  target:
    packages:
      - emergency-brake-controller
      - collision-detection-system
      - safety-monitor
  executionConstraints:
    maxStartupTime: "50ms"
    resourceGuarantee: true
    preemptLowerPriority: true
status:
  currentState: playing
  policyStatus: allowed
  lastTransitionTime: "2025-08-04T09:12:30Z"
  message: "Emergency brake system activated"
  executionMetrics:
    actualStartupTime: "42ms"
    resourceUtilization: "90%"
    responseTime: "5ms"
```

## 9. 리소스 관계도

다음은 PICCOLO 리소스 간의 관계를 보여주는 다이어그램입니다:

```
Scenario1     Scenario2     Scenario3
   |              |             |
   v              v             v
Package1 <-------Package2-----> Package3
   |                |             |
   v                v             v
Model1,Model2    Model3,Model4  Model5,Model6
   |                |             |
   v                v             v
Volume1,Volume2   Volume3      Volume4,Volume5
Network1          Network2     Network3
```

### 9.1 리소스 관계 정의

- **Scenario**: 차량 상태 조건에 따라 Package에 대한 액션 정의
  - 하나의 Scenario는 여러 Package를 대상으로 지정할 수 있음
  - 여러 Scenario가 동일한 Package를 대상으로 지정할 수 있음 (이 경우 하나의 Package에 여러 Scenario가 적용됨)

- **Package**: 여러 Model을 그룹화하여 함께 관리
  - 하나의 Package는 여러 Model을 포함할 수 있음
  - Package에 선언된 각 Model은 독립적으로 실행됨
  - 동일한 Model 정의가 여러 Package에 포함되어도 각각 독립적인 인스턴스로 실행됨

- **Model**: 실제 서비스를 제공하는 컨테이너 실행 단위
  - 각 Model은 하나 이상의 Volume과 Network에 연결될 수 있음
  - Model은 특정 Node에서 실행됨

- **Volume**: Model에 마운트되어 데이터 저장 제공
  - 하나의 Volume은 여러 Model에서 공유될 수 있음 (accessMode에 따라 다름)

- **Network**: Model 간의 통신을 위한 네트워크 리소스
  - 여러 Model이 동일한 Network에 연결될 수 있음

- **Node**: Model이 실행되는 물리적/가상 컴퓨팅 리소스
  - 하나의 Node는 여러 Model을 호스팅할 수 있음

### 9.2 리소스 관계 예시

- **다중 Package 실행 시나리오**:
  - `emergency-response` 시나리오는 `brake-controller`, `alert-system`, `sensor-fusion` 세 개의 패키지를 대상으로 지정할 수 있음
  - 각 패키지는 독립적으로 관리되며, 시나리오에 따라 함께 실행됨

- **다중 Model 패키지**:
  - `sensor-fusion` 패키지는 `camera-processor`, `lidar-processor`, `fusion-algorithm` 세 개의 모델을 포함할 수 있음
  - 세 모델은 같은 패키지에 속하지만 각각 독립적으로 실행됨
  - 각 모델은 자체 리소스 요구사항과 설정을 가짐

- **중복 대상 패키지**:
  - `night-driving` 시나리오와 `highway-driving` 시나리오가 모두 `vision-enhancement` 패키지를 대상으로 지정할 수 있음
  - 두 시나리오의 조건이 모두 충족되면, 패키지는 두 시나리오의 영향을 모두 받게 됨
  - 충돌하는 액션이 있을 경우 우선순위 규칙에 따라 처리됨

## 10. 리소스 상태 관리 참조

리소스 상태 관리는 PICCOLO 프레임워크의 핵심 기능으로, 각 리소스 타입(Scenario, Package, Model, Volume, Network, Node)의 생명주기와 상태 전이를 관리하는 역할을 담당합니다. 리소스 상태 관리의 상세 내용은 별도 문서 [`PICCOLO_Resource_StateManagement.md`](/home/edo/2025/projects/docs/workspace/specs/KR/PICCOLO_Resource_StateManagement.md)에 정의되어 있으며, 다음 내용을 포함합니다:

1. **리소스 상태 머신 정의**: 각 리소스 타입별 상태 정의, 상태 전이 다이어그램, 상태 전이 규칙
2. **리소스 모니터링 방법**: 각 리소스 타입별 모니터링 항목, 방법, 주기, 중요도
3. **상태 조정(Reconciliation) 프로세스**: 상태 조정 단계, 리소스 타입별 조정 전략, 충돌 해결 방법
4. **오류 복구(Recovery) 방법**: 각 리소스 타입별 오류 감지 및 복구 방법, 복구 시간 목표
5. **ASIL 서비스 상태 관리 특화 기능**: ASIL 등급별 모니터링 및 복구 전략, 상태 전이 규칙
6. **StateManager 구현 방식**: StateManager 아키텍처, 작동 방식, API

리소스 상태 관리는 StateManager 컴포넌트에 의해 중앙 집중적으로 수행되며, 지속적인 모니터링, 상태 조정, 오류 복구 메커니즘을 통해 안정적이고 안전한 차량 소프트웨어 플랫폼 운영을 보장합니다.

## 11. 결론

PICCOLO 프레임워크의 리소스 모델은 차량 환경에 최적화된 컨테이너 관리 방식을 제공합니다. Scenario, Package, Model, Volume, Network, Node의 6가지 핵심 리소스 타입을 통해 차량 상태에 따른 동적 서비스 관리, 리소스 할당, 서비스 격리 및 보안을 구현할 수 있습니다.

특히 ASIL 서비스 운영을 위한 time-critical, redundant 패키지 타입을 동시에 적용하는 등 여러 패키지 타입을 조합하여 차량의 안전 중요 서비스를 위한 강화된 관리 기능을 제공합니다.

각 리소스의 상태 관리는 별도 문서 [`PICCOLO_Resource_StateManagement.md`](/home/edo/2025/projects/docs/workspace/specs/KR/PICCOLO_Resource_StateManagement.md)에 정의된 상태 머신에 따라 이루어지며, StateManager 컴포넌트에 의해 중앙 집중적으로 모니터링, 조정, 복구됩니다. 이를 통해 안정적이고 안전한 차량 소프트웨어 플랫폼을 구축할 수 있습니다.
