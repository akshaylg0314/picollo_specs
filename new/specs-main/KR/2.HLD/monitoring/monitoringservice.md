# PICCOLO 리소스 모니터링 서비스 설계

## 1. 개요

PICCOLO 리소스 모니터링 서비스는 시스템 내 Scenario, Package, Model 등의 PICCOLO 리소스의 상태, 성능 및 품질을 실시간으로 추적하고 관리하기 위한 서비스입니다. 이 문서는 리소스 모니터링 서비스의 고수준 설계(HLD)를 정의합니다.

### 1.1 목적

- PICCOLO 리소스의 상태 및 성능 실시간 모니터링
- 리소스 간 종속성 추적 및 관리
- 이상 징후 감지 및 경고
- 문제 발생 시 자동 복구 메커니즘 제공
- 리소스 사용 및 성능 분석을 위한 데이터 수집

## 2. 리소스별 핵심 메트릭

### 2.1 Scenario

| 메트릭 | 설명 | 단위 |
|--------|------|------|
| 활성 상태 | 시나리오의 현재 활성화 상태 | Active/Inactive |
| 실행 횟수 | 시나리오가 실행된 총 횟수 | 횟수 |
| 평균 실행 시간 | 시나리오 실행의 평균 소요 시간 | ms |
| 실패율 | 시나리오 실행 실패 비율 | % |
| 종속성 상태 | 시나리오가 의존하는 리소스들의 상태 | OK/Warning/Error |

### 2.2 Package

| 메트릭 | 설명 | 단위 |
|--------|------|------|
| 배포 상태 | 패키지의 배포 상태 | Deployed/Undeployed/Failed |
| 버전 | 현재 배포된 패키지 버전 | 버전 문자열 |
| 사용량 | 패키지가 사용된 횟수 | 횟수 |
| 메모리 사용량 | 패키지가 사용하는 메모리 | MB |
| 디스크 사용량 | 패키지가 차지하는 디스크 공간 | MB |
| 로드 시간 | 패키지 로드에 소요되는 시간 | ms |

### 2.3 Model

| 메트릭 | 설명 | 단위 |
|--------|------|------|
| 실행 상태 | 모델(컨테이너)의 현재 실행 상태 | Running/Pending/Failed/Unknown |
| 가동 시간 | 모델이 실행된 총 시간 | 시/분/초 |
| 재시작 횟수 | 모델이 재시작된 횟수 | 횟수 |
| CPU 사용률 | 모델의 CPU 사용률 | % |
| 메모리 사용량 | 모델이 사용하는 메모리 | MB |
| 네트워크 트래픽 | 모델의 네트워크 트래픽 양 | MB/s |
| 응답 시간 | 모델의 평균 요청 응답 시간 | ms |

## 3. 모니터링 전략

### 3.1 상태 기반 모니터링

각 PICCOLO 리소스는 정의된 상태 머신을 따라 상태가 변경됩니다. 모니터링 서비스는 이러한 상태 변화를 추적하고 기록합니다:

```yaml
resource_state_monitoring:
  enabled: true
  tracking:
    state_transitions: true
    time_in_state: true
  alerting:
    unexpected_transitions: true
    stuck_states:
      enabled: true
      thresholds:
        loading: 30s
        initializing: 60s
```

### 3.2 성능 기반 모니터링

리소스의 성능 메트릭을 수집하고 분석하여 성능 저하나 문제를 감지합니다:

```yaml
performance_monitoring:
  collection_interval: 10s
  metrics:
    - name: execution_time
      threshold_warning: 500ms
      threshold_critical: 1000ms
    - name: memory_usage
      threshold_warning: 80%
      threshold_critical: 95%
    - name: failure_rate
      threshold_warning: 5%
      threshold_critical: 10%
```

### 3.3 종속성 모니터링

리소스 간 종속성을 추적하고 관련 리소스의 상태 변화가 미치는 영향을 분석합니다:

```yaml
dependency_monitoring:
  enabled: true
  dependency_types:
    - parent_child
    - uses
    - requires
  propagation:
    status_propagation: true
    impact_analysis: true
```

## 4. 구현 방법

### 4.1 리소스 모니터링 구성

각 리소스 유형에 대한 모니터링 구성은 YAML 형식으로 정의됩니다:

```yaml
resource_monitoring:
  scenario:
    enabled: true
    metrics:
      - active_state
      - execution_count
      - average_execution_time
      - failure_rate
    collection_interval: 5s
    
  package:
    enabled: true
    metrics:
      - deployment_status
      - version
      - usage_count
      - memory_usage
    collection_interval: 30s
    
  model:
    enabled: true
    metrics:
      - load_status
      - inference_time
      - throughput
      - accuracy
      - memory_usage
      - gpu_usage
    collection_interval: 10s
```

### 4.2 상태 전환 추적

리소스의 상태 변화를 추적하고 기록하는 구성:

```yaml
state_transition_tracking:
  enabled: true
  resources:
    - type: scenario
      states:
        - inactive
        - active
        - failed
    - type: package
      states:
        - undeployed
        - deploying
        - deployed
        - failed
    - type: model
      states:
        - unloaded
        - loading
        - loaded
        - error
  tracking:
    transition_history: true
    transition_duration: true
```

### 4.3 종속성 모니터링

리소스 간 종속성을 정의하고 추적하는 구성:

```yaml
dependency_monitoring:
  dependency_map:
    - resource: scenario_1
      depends_on:
        - type: package
          id: package_a
        - type: model
          id: model_x
    - resource: package_a
      depends_on:
        - type: model
          id: model_y
  impact_analysis:
    enabled: true
    propagation_rules:
      - from_state: failed
        impact: critical
      - from_state: degraded
        impact: warning
```

### 4.4 이상 탐지

정상 동작 패턴에서 벗어난 이상 징후를 감지하는 구성:

```yaml
anomaly_detection:
  enabled: true
  methods:
    - type: statistical
      metrics:
        - execution_time
        - memory_usage
      sensitivity: medium
    - type: pattern_based
      metrics:
        - state_transitions
        - error_counts
      window_size: 1h
  baseline:
    auto_learn: true
    learning_period: 7d
```

## 5. 데이터 저장

### 5.1 ETCD 기반 데이터 저장

모니터링 데이터는 ETCD에 저장되며, 키-값 스토어의 특성을 활용하여 효율적으로 관리됩니다:

```yaml
data_storage:
  type: etcd
  configuration:
    endpoints:
      - "etcd-server-1:2379"
      - "etcd-server-2:2379"
      - "etcd-server-3:2379"
    tls:
      enabled: true
      cert_file: "/etc/piccolo/certs/etcd-client.crt"
      key_file: "/etc/piccolo/certs/etcd-client.key"
      ca_file: "/etc/piccolo/certs/etcd-ca.crt"
  data_organization:
    prefix: "/piccolo/monitoring"
    key_structure:
      resource: "{resource_type}/{resource_id}"
      metric: "{resource_type}/{resource_id}/metrics/{metric_name}"
      state: "{resource_type}/{resource_id}/state"
      event: "{resource_type}/{resource_id}/events/{timestamp}"
  retention:
    metrics: 30d
    events: 90d
    state_transitions: 60d
  compaction:
    interval: 24h
    policy: "revision-based"
```

### 5.2 데이터 액세스 및 쿼리

ETCD에 저장된 모니터링 데이터에 접근하는 방법:

```yaml
data_access:
  apis:
    - name: "GetResourceMetrics"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/metrics"
      parameters:
        - name: "metric_names"
          type: "array"
          required: false
        - name: "time_range"
          type: "object"
          required: false
      response_format: "json"
      
    - name: "GetResourceState"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/state"
      parameters:
        - name: "include_history"
          type: "boolean"
          required: false
      response_format: "json"
      
    - name: "GetResourceEvents"
      path: "/api/v1/monitoring/resources/{resource_type}/{resource_id}/events"
      parameters:
        - name: "event_types"
          type: "array"
          required: false
        - name: "time_range"
          type: "object"
          required: false
        - name: "limit"
          type: "integer"
          required: false
      response_format: "json"
  
  query_capabilities:
    - prefix_search
    - range_queries
    - transaction_support
```

## 6. 확장성 및 통합

### 6.1 확장성

대규모 PICCOLO 리소스 환경을 지원하기 위한 확장성 고려 사항:

- 분산 모니터링 아키텍처
- 데이터 샘플링 및 집계 전략
- 계층적 모니터링 구조

### 6.2 외부 시스템 통합

기존 모니터링 및 관리 시스템과의 통합:

- Kubernetes 통합
- 로깅 시스템 연동
- 표준 메트릭 익스포터 제공

## 7. 리소스 모니터링 템플릿

아래는 Scenario와 Package 리소스에 대한 모니터링 메트릭 정보를 포함한 YAML 템플릿입니다. 각 템플릿은 사용자가 정의하는 부분(spec)과 시스템이 모니터링하고 관리하는 부분(status)을 모두 포함합니다.

### 7.1 Scenario 리소스 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: "{{SCENARIO_NAME}}"               # 시나리오 이름 (예: "adas-activation")
  annotations:
    description: "{{DESCRIPTION}}"        # 시나리오 설명 (예: "Activates ADAS features during driving")
    priority: "{{PRIORITY}}"              # 우선순위 (예: "high", "medium", "low")
  labels:
    function: "{{FUNCTION}}"              # 기능 (예: "driver-assistance")
    category: "{{CATEGORY}}"              # 카테고리 (예: "safety", "comfort")
    asilLevel: "{{ASIL_LEVEL}}"           # ASIL 레벨 (예: "A", "B", "C", "D", "QM")
spec:
  condition:
    {{CONDITION_KEY}}: "{{CONDITION_VALUE}}"   # 조건 키-값 (예: vehicleSpeed: ">30")
    # 추가 조건들...
  action: "{{ACTION_TYPE}}"               # 액션 유형 (launch, update, modify, notify, rollback, pause, resume)
  target:
    packages:
      - "{{PACKAGE_NAME_1}}"              # 대상 패키지 이름 (예: "adas-software")
      - "{{PACKAGE_NAME_2}}"              # 추가 패키지...
      # 추가 패키지들...
# 시스템이 추적/관리하는 부분
status:
  # 기본 상태 정보
  currentState: "{{CURRENT_STATE}}"           # 현재 상태 (idle/waiting/playing/error/allowed/denied)
  policyStatus: "{{POLICY_STATUS}}"           # 정책 상태 (allowed/denied)
  lastTransitionTime: "{{LAST_TRANSITION_TIME}}"  # 마지막 상태 전이 시간
  message: "{{STATUS_MESSAGE}}"               # 상태 메시지
  
  # 시나리오 조건 및 실행 상태
  conditions:
    - type: "{{CONDITION_TYPE}}"              # 조건 유형 (예: "ConditionsMet")
      status: "{{CONDITION_STATUS}}"          # 조건 상태 (True/False)
      lastUpdateTime: "{{LAST_UPDATE_TIME}}"  # 마지막 업데이트 시간
    # 추가 조건 상태들...
  
  # 대상 패키지 상태
  activePackages:
    - name: "{{PACKAGE_NAME_1}}"              # 패키지 이름
      status: "{{PACKAGE_STATUS}}"            # 패키지 상태 (running, error 등)
    # 추가 패키지 상태들...
  
  # 모니터링 메트릭
  metrics:
    # 활성 상태 메트릭
    activeState: "{{ACTIVE_STATE}}"           # Active/Inactive
    
    # 실행 관련 메트릭
    executionCount: {{EXECUTION_COUNT}}       # 실행 횟수
    averageExecutionTime: {{AVG_EXEC_TIME}}   # 평균 실행 시간 (ms)
    lastExecutionTime: {{LAST_EXEC_TIME}}     # 마지막 실행 시간 (ms)
    failureRate: {{FAILURE_RATE}}             # 실패율 (%)
    lastExecutionResult: "{{EXEC_RESULT}}"    # 마지막 실행 결과 (Success/Failure)
    
    # 종속성 상태 메트릭
    dependencyStatus: "{{DEPENDENCY_STATUS}}" # 종속성 상태 (OK/Warning/Error)
    dependencyDetails:
      - resourceType: "{{RESOURCE_TYPE}}"     # 리소스 타입 (Package, Model 등)
        resourceName: "{{RESOURCE_NAME}}"     # 리소스 이름
        status: "{{RESOURCE_STATUS}}"         # 리소스 상태 (OK/Warning/Error)
      # 추가 종속성 상태들...
    
    # 성능 메트릭
    responseTime: {{RESPONSE_TIME}}           # 응답 시간 (ms)
    resourceUtilization: {{RESOURCE_UTIL}}    # 리소스 사용률 (%)
    
    # 이벤트 메트릭
    totalEvents: {{TOTAL_EVENTS}}             # 총 이벤트 수
    lastEventTime: "{{LAST_EVENT_TIME}}"      # 마지막 이벤트 시간
    recentEvents:
      - type: "{{EVENT_TYPE}}"                # 이벤트 유형
        time: "{{EVENT_TIME}}"                # 이벤트 시간
        details: "{{EVENT_DETAILS}}"          # 이벤트 상세 정보
      # 추가 이벤트들...
    
    # ASIL 관련 메트릭 (안전 중요 시나리오인 경우)
    safetyMetrics:
      responseTimeConstraint: {{RT_CONSTRAINT}}  # 응답 시간 제약 (ms)
      responseTimeActual: {{RT_ACTUAL}}          # 실제 응답 시간 (ms)
      constraintsMet: {{CONSTRAINTS_MET}}        # 제약 충족 여부 (true/false)
```

### 7.2 Package 리소스 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: "{{PACKAGE_NAME}}"               # 패키지 이름 (예: "vision-processing")
  annotations:
    version: "{{VERSION}}"               # 버전 (예: "2.3")
    description: "{{DESCRIPTION}}"       # 설명 (예: "카메라 및 영상 처리 시스템")
  labels:
    types: ["{{TYPE_1}}", "{{TYPE_2}}"]  # 패키지 타입 (예: ["resource-bounded", "time-critical"])
    component: "{{COMPONENT}}"           # 컴포넌트 (예: "vision")
    asilLevel: "{{ASIL_LEVEL}}"          # ASIL 레벨 (예: "B")
spec:
  pattern:
    type: "{{PATTERN_TYPE}}"             # 패턴 타입 (예: "selector", "plain", "distributed")
    nodeSelector:
      "{{SELECTOR_KEY}}": "{{SELECTOR_VALUE}}"  # 노드 선택자 (예: gpu: "available")
  packageType: "{{PACKAGE_TYPE}}"        # 패키지 타입 (예: "StandardPackage", "CustomPackage", "SystemPackage")
  typeProperties:
    category: "{{TYPE_CATEGORY}}"        # 타입 카테고리 (예: "application", "system", "infrastructure")
    criticality: "{{CRITICALITY}}"       # 중요도 (예: "high", "medium", "low")
    lifecycle: "{{LIFECYCLE}}"           # 라이프사이클 (예: "development", "testing", "production")
  models:
    - name: "{{MODEL_NAME_1}}"           # 모델 이름 (예: "vision-system")
      image: "{{IMAGE_PATH}}"            # 이미지 경로 (예: "vehicle/vision-system:2.3")
      node: "{{NODE_NAME}}"              # 노드 이름 (예: "VisionHPC")
# 시스템이 추적/관리하는 부분
status:
  # 기본 상태 정보
  currentState: "{{CURRENT_STATE}}"      # 현재 상태 (initializing/running/degraded/error)
  lastTransitionTime: "{{LAST_TRANSITION_TIME}}"  # 마지막 상태 전이 시간
  message: "{{STATUS_MESSAGE}}"          # 상태 메시지
  
  # 모델 상태 정보
  models:
    - name: "{{MODEL_NAME_1}}"           # 모델 이름
      status: "{{MODEL_STATUS}}"         # 모델 상태 (running, error 등)
      restarts: {{RESTART_COUNT}}        # 재시작 횟수
      readiness: {{READINESS}}           # 준비 상태 (true/false)
    # 추가 모델 상태들...
  
  # 패턴 및 타입 모니터링 상태
  patternStatus:
    type: "{{PATTERN_TYPE}}"             # 패턴 타입 (selector, plain, distributed)
    health: "{{PATTERN_HEALTH}}"         # 패턴 상태 (healthy, degraded, failed)
    nodesMatched: {{NODES_MATCHED}}      # 매칭된 노드 수
    nodeList:
      - name: "{{NODE_NAME_1}}"          # 노드 이름
        status: "{{NODE_STATUS}}"        # 노드 상태
      # 추가 노드들...
    lastVerified: "{{LAST_VERIFIED}}"    # 마지막 검증 시간
  
  packageTypeStatus:
    type: "{{PACKAGE_TYPE}}"             # 패키지 타입
    complianceStatus: "{{COMPLIANCE}}"   # 규정 준수 상태 (compliant, non-compliant)
    typeConstraints:
      - name: "{{CONSTRAINT_NAME}}"      # 제약 조건 이름
        status: "{{CONSTRAINT_STATUS}}"  # 제약 조건 상태 (met, not-met)
      # 추가 제약 조건들...
    propertiesStatus:
      lifecycle: "{{LIFECYCLE_STATUS}}"  # 라이프사이클 상태
      criticality: "{{CRITICALITY_LEVEL}}" # 중요도 레벨
  
  # 모니터링 메트릭
  metrics:
    # 배포 상태 메트릭
    deploymentStatus: "{{DEPLOYMENT_STATUS}}"  # 배포 상태 (Deployed/Undeployed/Failed)
    deploymentTime: "{{DEPLOYMENT_TIME}}"      # 배포 시간
    
    # 버전 정보
    version: "{{VERSION}}"                     # 버전
    lastUpdateTime: "{{LAST_UPDATE_TIME}}"     # 마지막 업데이트 시간
    
    # 패키지 타입 모니터링 메트릭
    packageType:
      type: "{{PACKAGE_TYPE}}"                 # 패키지 타입
      count: {{COUNT}}                         # 동일 타입 패키지 수
      operationalStatus: "{{OP_STATUS}}"       # 운영 상태
      typicalResponseTime: {{RESPONSE_TIME}}   # 일반적인 응답 시간
      healthScore: "{{HEALTH_SCORE}}"          # 상태 점수 (0-100%)
      typeSpecificMetrics:
        - name: "{{METRIC_NAME}}"              # 타입별 메트릭 이름
          value: "{{METRIC_VALUE}}"            # 타입별 메트릭 값
          threshold: "{{METRIC_THRESHOLD}}"    # 타입별 메트릭 임계값
    
    # 패턴 모니터링 메트릭
    patternMetrics:
      type: "{{PATTERN_TYPE}}"                 # 패턴 타입
      effectiveDeploymentRate: {{DEPLOY_RATE}} # 효과적 배포율 (%)
      patternComplianceScore: {{COMPLIANCE}}   # 패턴 준수 점수 (0-100%)
      nodeDistribution:                        # 노드 분포
        matched: {{MATCHED_NODES}}             # 매칭된 노드 수
        healthy: {{HEALTHY_NODES}}             # 정상 노드 수
        degraded: {{DEGRADED_NODES}}           # 성능 저하 노드 수
        failed: {{FAILED_NODES}}               # 실패 노드 수
      balanceScore: {{BALANCE_SCORE}}          # 부하 분산 점수 (0-100%)
    
    # 사용 메트릭
    usageCount: {{USAGE_COUNT}}                # 사용 횟수
    activeScenarios: {{ACTIVE_SCENARIOS}}      # 활성 시나리오 수
    uptime: "{{UPTIME}}"                       # 가동 시간
    
    # 모델 관계 메트릭
    modelRelationships:
      total: {{TOTAL_MODELS}}                  # 총 모델 수
      healthy: {{HEALTHY_MODELS}}              # 정상 모델 수
      unhealthy: {{UNHEALTHY_MODELS}}          # 비정상 모델 수
      dependencies:
        - source: "{{SOURCE_MODEL}}"           # 소스 모델
          target: "{{TARGET_MODEL}}"           # 타겟 모델
          status: "{{DEPENDENCY_STATUS}}"      # 종속성 상태
        # 추가 종속성들...                    # 가동 시간
   
```

### 7.3 Model 리소스 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: Model
metadata:
  name: "{{MODEL_NAME}}"                  # 모델 이름 (예: "vision-system")
  annotations:
    version: "{{VERSION}}"                # 버전 (예: "1.2")
    description: "{{DESCRIPTION}}"        # 설명 (예: "카메라 비전 처리 시스템")
  labels:
    component: "{{COMPONENT}}"            # 컴포넌트 (예: "vision")
    asilLevel: "{{ASIL_LEVEL}}"           # ASIL 레벨 (예: "B", "D")
spec:
  image: "{{IMAGE_PATH}}"                 # 이미지 경로 (예: "vehicle/vision-system:1.2")
  node: "{{NODE_NAME}}"                   # 노드 이름 (예: "VisionHPC")
  hostNetwork: {{HOST_NETWORK}}           # 호스트 네트워크 사용 여부 (true/false)
  containers:
    - name: "{{CONTAINER_NAME}}"          # 컨테이너 이름
      image: "{{CONTAINER_IMAGE}}"        # 컨테이너 이미지
      resources:
        limits:
          cpu: {{CPU_LIMIT}}              # CPU 제한 (예: 1)
          memory: "{{MEMORY_LIMIT}}"      # 메모리 제한 (예: "1Gi")
          gpu: {{GPU_LIMIT}}              # GPU 제한 (예: 1)
  scheduling:
    timeCritical: {{TIME_CRITICAL}}       # 시간 중요도 (true/false)
    priority: {{PRIORITY}}                # 우선순위 (예: 95)
    period: "{{PERIOD}}"                  # 주기 (예: "10ms")
    deadline: "{{DEADLINE}}"              # 데드라인 (예: "8ms")
    executionTime: "{{EXECUTION_TIME}}"   # 실행 시간 (예: "2ms")
    cpuAffinity: [{{CPU_AFFINITY}}]       # CPU 친화성 (예: [2, 3])
    isolationLevel: "{{ISOLATION_LEVEL}}" # 격리 수준 (예: "exclusive")
    criticality: "{{CRITICALITY}}"        # 중요도 (예: "safety-critical")
  deviceAccess:
    - manager: "{{DEVICE_MANAGER}}"       # 장치 관리자 (예: "vehicle-sensors")
      devices: ["{{DEVICE_1}}", "{{DEVICE_2}}"] # 장치 목록 (예: ["camera", "lidar"])
  volumes:
    - name: "{{VOLUME_NAME}}"             # 볼륨 이름
      mountPath: "{{MOUNT_PATH}}"         # 마운트 경로 (예: "/data")
      volume: "{{VOLUME_REF}}"            # 볼륨 참조 (예: "vision-data-volume")
  networks:
    - name: "{{NETWORK_NAME}}"            # 네트워크 이름 (예: "sensor-network")
# 시스템이 추적/관리하는 부분
status:
  # 기본 상태 정보
  phase: "{{PHASE}}"                      # 단계 (Running/Pending/Succeeded/Failed/Unknown)
  conditions:
    - type: "{{CONDITION_TYPE}}"          # 조건 유형 (예: "Ready")
      status: "{{CONDITION_STATUS}}"      # 조건 상태 (True/False)
      lastProbeTime: "{{LAST_PROBE_TIME}}" # 마지막 검사 시간
  startTime: "{{START_TIME}}"             # 시작 시간
  hostIP: "{{HOST_IP}}"                   # 호스트 IP
  podIP: "{{POD_IP}}"                     # Pod IP
  
  # 컨테이너 상태 정보
  containerStatuses:
    - name: "{{CONTAINER_NAME}}"          # 컨테이너 이름
      state:
        running:
          startedAt: "{{STARTED_AT}}"     # 시작 시간
      ready: {{READY}}                    # 준비 상태 (true/false)
      restartCount: {{RESTART_COUNT}}     # 재시작 횟수
      image: "{{IMAGE}}"                  # 이미지
      imageID: "{{IMAGE_ID}}"             # 이미지 ID
  
  # 모니터링 메트릭
  metrics:
    # 로드 상태 메트릭
    loadStatus: "{{LOAD_STATUS}}"         # 로드 상태 (Loaded/Unloaded/Error)
    loadTime: {{LOAD_TIME}}               # 로드 시간 (ms)
    
    # 성능 메트릭
    inferenceTime: {{INFERENCE_TIME}}     # 추론 시간 (ms)
    throughput: {{THROUGHPUT}}            # 처리량 (요청/초)
    latency: {{LATENCY}}                  # 지연 시간 (ms)
    accuracy: {{ACCURACY}}                # 정확도 (%)
    
    # 리소스 사용 메트릭
    memoryUsage: {{MEMORY_USAGE}}         # 메모리 사용량 (MB)
    memoryUsagePercent: {{MEMORY_PCT}}    # 메모리 사용률 (%)
    cpuUsage: {{CPU_USAGE}}               # CPU 사용률 (%)
    gpuUsage: {{GPU_USAGE}}               # GPU 사용률 (%)
    gpuMemoryUsage: {{GPU_MEMORY}}        # GPU 메모리 사용량 (MB)
    
    # 상태 메트릭
    uptime: "{{UPTIME}}"                  # 가동 시간
    healthStatus: "{{HEALTH_STATUS}}"     # 건강 상태 (Healthy/Unhealthy)
    lastError: "{{LAST_ERROR}}"           # 마지막 오류
    errorCount: {{ERROR_COUNT}}           # 오류 횟수
    
    # 이벤트 메트릭
    events:
      - type: "{{EVENT_TYPE}}"            # 이벤트 유형
        time: "{{EVENT_TIME}}"            # 이벤트 시간
        message: "{{EVENT_MESSAGE}}"      # 이벤트 메시지
        severity: "{{EVENT_SEVERITY}}"    # 이벤트 심각도
    
    # ASIL 관련 메트릭 (안전 중요 모델인 경우)
    safetyMetrics:
      asilLevel: "{{ASIL_LEVEL}}"            # ASIL 레벨 (A/B/C/D)
      deadlineMissCount: {{DEADLINE_MISSES}} # 데드라인 미스 횟수
      worstCaseExecutionTime: {{WCET}}       # 최악 실행 시간 (ms)
      averageExecutionTime: {{AVG_EXEC_TIME}} # 평균 실행 시간 (ms)
      executionTimeVariance: {{EXEC_VARIANCE}} # 실행 시간 분산
      schedulingLatency: {{SCHED_LATENCY}}   # 스케줄링 지연 시간 (ms)
      safetyMonitorStatus: "{{SAFETY_STATUS}}" # 안전 모니터 상태 (Active/Inactive)
      safetyChecks:
        - name: "{{CHECK_NAME}}"            # 체크 이름 (예: "Timing Constraint")
          status: "{{CHECK_STATUS}}"        # 체크 상태 (Pass/Fail)
          lastCheckTime: "{{CHECK_TIME}}"   # 마지막 체크 시간
      functionalSafetyMetrics:
        errorDetectionCoverage: {{ERROR_DETECTION}} # 오류 탐지 커버리지 (%)
        errorRecoveryCoverage: {{ERROR_RECOVERY}}   # 오류 복구 커버리지 (%)
        diagnosticsCoverage: {{DIAGNOSTICS}}        # 진단 커버리지 (%)
      runtimeVerification:
        enabled: {{RUNTIME_VERIFY}}              # 런타임 검증 활성화 (true/false)
        constraintsChecked: {{CONSTRAINTS_CHECKED}} # 검증된 제약 조건 수
        constraintViolations: {{CONSTRAINT_VIOLATIONS}} # 제약 조건 위반 횟수
```

### 7.4 실제 사용 예시

아래는 Scenario와 Package 리소스의 YAML 사용 예시입니다:

```yaml
# Scenario 예시
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: "adas-activation"
  annotations:
    description: "차량 주행 중 ADAS 기능 활성화"
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
  action: "launch"
  target:
    packages:
      - "adas-software"
      - "vision-processing"
status:
  currentState: "playing"
  policyStatus: "allowed"
  lastTransitionTime: "2025-08-07T14:25:32Z"
  message: "ADAS 서비스가 성공적으로 활성화됨"
  metrics:
    activeState: "Active"
    executionCount: 127
    averageExecutionTime: 235
    failureRate: 0.5
    dependencyStatus: "OK"
```

```yaml
# Package 예시
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: "vision-processing"
  annotations:
    version: "2.3"
    description: "카메라 및 영상 처리 시스템"
  labels:
    types: ["resource-bounded", "time-critical"]
    component: "vision"
    asilLevel: "B"
spec:
  pattern:
    type: "selector"
    nodeSelector:
      gpu: "available"
  models:
    - name: "vision-system"
      image: "vehicle/vision-system:2.3"
      node: "VisionHPC"
status:
  currentState: "running"
  lastTransitionTime: "2025-08-07T14:25:35Z"
  message: "모든 모델이 정상 실행 중"
  metrics:
    deploymentStatus: "Deployed"
    version: "2.3"
    memoryUsage: 5.6
    cpuUsage: 65
    loadTime: 4250
    healthStatus: "Healthy"
```

## 8. 결론

PICCOLO 리소스 모니터링 서비스는 시스템 내 모든 리소스의 상태와 성능을 종합적으로 추적하고 관리합니다. ETCD를 기반으로 한 데이터 저장 방식을 통해 안정적이고 확장 가능한 모니터링 인프라를 제공하며, 이를 통해 문제를 조기에 발견하고 시스템의 안정성과 신뢰성을 향상시킵니다. 또한 이 문서에서 제공하는 상세한 메트릭 템플릿을 통해 다양한 리소스 유형에 대한 모니터링을 효과적으로 구현할 수 있습니다.
