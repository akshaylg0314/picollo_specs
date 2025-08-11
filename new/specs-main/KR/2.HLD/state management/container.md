# PICCOLO 컨테이너 상태관리 상세 기능 명세서

**문서 번호**: PICCOLO-STATE-2025-002  
**버전**: 1.0  
**날짜**: 2025-07-17  
**작성자**: joshua-jung_LGESDV  
**분류**: 기능 명세서  
**작성 시각**: 2025-07-17 02:22:27 UTC

## 1. 개요

본 문서는 PICCOLO 프레임워크의 상태관리 기능에 대한 상세 명세를 기술합니다. PICCOLO의 상태관리는 NodeAgent와 StateManager를 통해 시스템 및 서비스의 상태를 지속적으로 모니터링하고, 실제 상태와 원하는 상태 간의 차이를 감지하여 조정(reconciliation)하는 역할을 담당합니다. 본 명세서는 NodeAgent의 데이터 수집 및 StateManager의 상태 머신을 중심으로 상태관리 기능을 정의합니다.

## 2. NodeAgent 메트릭 수집 기능

### 2.1 Pod/컨테이너 메트릭 정의

NodeAgent는 시스템 내 모든 Pod 및 컨테이너에 대해 다음과 같은 메트릭을 수집합니다:

#### 2.1.1 컴퓨팅 리소스 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_cpu_usage_seconds | 코어-초 | 컨테이너가 사용한 CPU 시간 | 1초 | 필수 |
| container_cpu_utilization | % | CPU 사용률 (할당 대비) | 1초 | 필수 |
| container_cpu_throttling | 횟수 | CPU 제한으로 인한 스로틀링 횟수 | 1초 | 높음 |
| container_cpu_scheduling_latency | ms | CPU 스케줄링 지연 시간 | 1초 | 중간 |
| container_cpu_context_switches | 횟수 | 컨텍스트 스위치 횟수 | 5초 | 낮음 |

#### 2.1.2 메모리 리소스 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_memory_usage_bytes | 바이트 | 현재 메모리 사용량 | 1초 | 필수 |
| container_memory_working_set | 바이트 | 작업 세트 메모리 (활성 사용) | 1초 | 필수 |
| container_memory_limit | 바이트 | 메모리 제한값 | 1초 | 높음 |
| container_memory_cache | 바이트 | 페이지 캐시 메모리 | 1초 | 중간 |
| container_memory_rss | 바이트 | 실제 메모리 사용량 (RSS) | 1초 | 높음 |
| container_memory_swap | 바이트 | 스왑 메모리 사용량 | 1초 | 중간 |
| container_memory_failcnt | 횟수 | 메모리 할당 실패 횟수 | 1초 | 높음 |
| container_memory_oom_events | 횟수 | OOM 킬러 발생 이벤트 | 1초 | 필수 |

#### 2.1.3 스토리지 및 I/O 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_fs_usage_bytes | 바이트 | 파일시스템 사용량 | 5초 | 중간 |
| container_fs_limit_bytes | 바이트 | 파일시스템 제한 | 5초 | 중간 |
| container_fs_reads | 횟수 | 디스크 읽기 연산 수 | 5초 | 중간 |
| container_fs_writes | 횟수 | 디스크 쓰기 연산 수 | 5초 | 중간 |
| container_fs_read_bytes | 바이트 | 디스크에서 읽은 바이트 수 | 5초 | 중간 |
| container_fs_write_bytes | 바이트 | 디스크에 쓴 바이트 수 | 5초 | 중간 |
| container_fs_io_current | 횟수 | 현재 진행 중인 I/O 작업 수 | 5초 | 낮음 |
| container_fs_io_time | ms | I/O 작업에 소요된 시간 | 5초 | 낮음 |

#### 2.1.4 네트워크 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_network_receive_bytes | 바이트 | 수신된 바이트 수 | 1초 | 높음 |
| container_network_transmit_bytes | 바이트 | 전송된 바이트 수 | 1초 | 높음 |
| container_network_receive_packets | 패킷 | 수신된 패킷 수 | 1초 | 중간 |
| container_network_transmit_packets | 패킷 | 전송된 패킷 수 | 1초 | 중간 |
| container_network_receive_errors | 횟수 | 수신 오류 수 | 1초 | 높음 |
| container_network_transmit_errors | 횟수 | 전송 오류 수 | 1초 | 높음 |
| container_network_receive_drops | 횟수 | 드롭된 수신 패킷 수 | 1초 | 높음 |
| container_network_transmit_drops | 횟수 | 드롭된 전송 패킷 수 | 1초 | 높음 |

#### 2.1.5 프로세스 및 런타임 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_processes | 개수 | 컨테이너 내 프로세스 수 | 5초 | 중간 |
| container_threads | 개수 | 컨테이너 내 스레드 수 | 5초 | 중간 |
| container_file_descriptors | 개수 | 열린 파일 디스크립터 수 | 5초 | 중간 |
| container_uptime | 초 | 컨테이너 가동 시간 | 5초 | 높음 |
| container_restart_count | 횟수 | 컨테이너 재시작 횟수 | 5초 | 필수 |
| container_exit_code | 코드 | 마지막 종료 코드 | 이벤트 기반 | 필수 |
| container_oom_killed | 불리언 | OOM으로 종료 여부 | 이벤트 기반 | 필수 |

#### 2.1.6 애플리케이션 수준 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| container_app_response_time | ms | 애플리케이션 응답 시간 | 10초 | 중간 |
| container_app_error_rate | % | 오류율 | 10초 | 높음 |
| container_app_request_rate | rps | 초당 요청 수 | 10초 | 중간 |
| container_app_health_status | 열거형 | 상태 검사 결과 (정상/경고/오류) | 10초 | 필수 |
| container_app_garbage_collection | ms | GC에 소요된 시간 | 10초 | 낮음 |

### 2.2 하드웨어 시스템 정보 정의

NodeAgent는 호스트 시스템의 하드웨어 상태에 대한 다음 정보를 수집합니다:

#### 2.2.1 시스템 CPU 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_cpu_utilization | % | 전체 CPU 사용률 | 1초 | 필수 |
| node_cpu_core_utilization | % | 코어별 CPU 사용률 | 1초 | 높음 |
| node_cpu_user | % | 사용자 모드 CPU 사용률 | 1초 | 중간 |
| node_cpu_system | % | 시스템 모드 CPU 사용률 | 1초 | 중간 |
| node_cpu_iowait | % | I/O 대기 시간 비율 | 1초 | 중간 |
| node_cpu_irq | % | 인터럽트 처리 시간 비율 | 1초 | 낮음 |
| node_cpu_softirq | % | 소프트 인터럽트 처리 시간 비율 | 1초 | 낮음 |
| node_cpu_idle | % | 유휴 상태 시간 비율 | 1초 | 중간 |
| node_cpu_temp | °C | CPU 온도 | 5초 | 높음 |
| node_cpu_freq | MHz | CPU 주파수 | 5초 | 중간 |
| node_load_1m | 부하 | 1분 평균 시스템 부하 | 5초 | 높음 |
| node_load_5m | 부하 | 5분 평균 시스템 부하 | 5초 | 중간 |
| node_load_15m | 부하 | 15분 평균 시스템 부하 | 5초 | 중간 |

#### 2.2.2 시스템 메모리 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_memory_total | 바이트 | 총 메모리 크기 | 5초 | 필수 |
| node_memory_used | 바이트 | 사용 중인 메모리 | 1초 | 필수 |
| node_memory_free | 바이트 | 사용 가능한 메모리 | 1초 | 필수 |
| node_memory_cached | 바이트 | 캐시된 메모리 | 1초 | 중간 |
| node_memory_buffers | 바이트 | 버퍼 메모리 | 1초 | 중간 |
| node_memory_swap_total | 바이트 | 총 스왑 크기 | 5초 | 중간 |
| node_memory_swap_used | 바이트 | 사용 중인 스왑 | 1초 | 중간 |
| node_memory_swap_free | 바이트 | 사용 가능한 스왑 | 1초 | 중간 |
| node_memory_pagefaults | 횟수 | 페이지 폴트 발생 수 | 1초 | 중간 |
| node_memory_major_pagefaults | 횟수 | 메이저 페이지 폴트 수 | 1초 | 높음 |

#### 2.2.3 시스템 디스크 및 I/O 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_disk_total | 바이트 | 총 디스크 공간 | 30초 | 필수 |
| node_disk_used | 바이트 | 사용 중인 디스크 공간 | 30초 | 필수 |
| node_disk_free | 바이트 | 사용 가능한 디스크 공간 | 30초 | 필수 |
| node_disk_reads | 횟수 | 디스크 읽기 연산 수 | 5초 | 중간 |
| node_disk_writes | 횟수 | 디스크 쓰기 연산 수 | 5초 | 중간 |
| node_disk_read_bytes | 바이트 | 읽은 바이트 수 | 5초 | 중간 |
| node_disk_write_bytes | 바이트 | 쓴 바이트 수 | 5초 | 중간 |
| node_disk_io_time | ms | I/O 작업에 소요된 시간 | 5초 | 중간 |
| node_disk_io_weighted | ms | 가중치가 적용된 I/O 시간 | 5초 | 낮음 |
| node_disk_temp | °C | 디스크 온도 | 30초 | 높음 |
| node_disk_smart_status | 열거형 | S.M.A.R.T 상태 | 1분 | 높음 |
| node_disk_inode_total | 개수 | 총 inode 수 | 1분 | 중간 |
| node_disk_inode_used | 개수 | 사용 중인 inode 수 | 1분 | 중간 |

#### 2.2.4 시스템 네트워크 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_network_receive_bytes | 바이트 | 수신된 바이트 수 | 1초 | 높음 |
| node_network_transmit_bytes | 바이트 | 전송된 바이트 수 | 1초 | 높음 |
| node_network_receive_packets | 패킷 | 수신된 패킷 수 | 1초 | 중간 |
| node_network_transmit_packets | 패킷 | 전송된 패킷 수 | 1초 | 중간 |
| node_network_receive_errors | 횟수 | 수신 오류 수 | 1초 | 높음 |
| node_network_transmit_errors | 횟수 | 전송 오류 수 | 1초 | 높음 |
| node_network_receive_drops | 횟수 | 드롭된 수신 패킷 수 | 1초 | 높음 |
| node_network_transmit_drops | 횟수 | 드롭된 전송 패킷 수 | 1초 | 높음 |
| node_network_collisions | 횟수 | 충돌 수 | 1초 | 중간 |
| node_network_up | 불리언 | 네트워크 인터페이스 상태 | 5초 | 필수 |
| node_network_mtu | 바이트 | 네트워크 인터페이스 MTU | 1분 | 낮음 |
| node_network_speed | Mbps | 네트워크 인터페이스 속도 | 1분 | 중간 |

#### 2.2.5 시스템 전력 및 온도 메트릭

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_power_consumption | 와트 | 총 전력 소비량 | 5초 | 높음 |
| node_power_voltage | 볼트 | 전압 | 5초 | 중간 |
| node_power_current | 암페어 | 전류 | 5초 | 중간 |
| node_power_battery_level | % | 배터리 잔량 (해당 시) | 5초 | 높음 |
| node_power_battery_time_remaining | 분 | 배터리 잔여 시간 (해당 시) | 5초 | 중간 |
| node_temp_system | °C | 시스템 온도 | 5초 | 높음 |
| node_temp_ambient | °C | 주변 온도 | 5초 | 중간 |
| node_fan_speed | RPM | 팬 속도 | 5초 | 중간 |
| node_thermal_throttling | 불리언 | 발열로 인한 성능 제한 여부 | 5초 | 높음 |

#### 2.2.6 시스템 일반 정보

| 메트릭 이름 | 단위 | 설명 | 샘플링 주기 | 우선순위 |
|------------|------|------|------------|---------|
| node_uptime | 초 | 시스템 가동 시간 | 1분 | 높음 |
| node_boot_time | 타임스탬프 | 부팅 시간 | 부팅 시 | 중간 |
| node_kernel_version | 문자열 | 커널 버전 | 부팅 시 | 중간 |
| node_os_version | 문자열 | OS 버전 | 부팅 시 | 중간 |
| node_hostname | 문자열 | 호스트명 | 부팅 시 | 중간 |
| node_processes | 개수 | 총 프로세스 수 | 10초 | 중간 |
| node_users | 개수 | 로그인한 사용자 수 | 1분 | 낮음 |
| node_time | 타임스탬프 | 현재 시스템 시간 | 1분 | 중간 |
| node_time_synchronized | 불리언 | 시간 동기화 상태 | 1분 | 중간 |

### 2.3 데이터 수집 아키텍처

#### 2.3.1 수집 메커니즘

NodeAgent는 다음과 같은 메커니즘을 통해 데이터를 수집합니다:

1. **컨테이너 런타임 API**:
   - containerd/Docker API를 사용하여 컨테이너 메트릭 수집
   - CRI(Container Runtime Interface) 호출을 통한 데이터 수집

2. **cgroup 파일시스템**:
   - `/sys/fs/cgroup/` 경로의 cgroup 파일시스템 직접 모니터링
   - CPU, 메모리, I/O 등의 리소스 사용량 추적

3. **procfs 및 sysfs**:
   - `/proc` 및 `/sys` 파일시스템을 통한 시스템 정보 수집
   - 커널 수준의 상세 정보 수집

4. **하드웨어 모니터링 인터페이스**:
   - IPMI(Intelligent Platform Management Interface) 활용
   - lm-sensors와 같은 온도 및 팬 모니터링 도구 통합
   - S.M.A.R.T 디스크 상태 정보 수집

5. **애플리케이션 메트릭 엔드포인트**:
   - 애플리케이션이 제공하는 `/metrics` 엔드포인트 스크래핑
   - Prometheus 형식 메트릭 지원

#### 2.3.2 수집 스케줄링

NodeAgent는 TIMPANI 스케줄러를 활용하여 정확한 간격으로 데이터를 수집합니다:

1. **우선순위별 수집 태스크**:

   | 우선순위 | 태스크 이름 | 스케줄링 주기 | 실행 시간 제약 | CPU 친화성 |
   |---------|------------|--------------|--------------|-----------|
   | 95 | critical_metrics_collector | 1초 | 50ms | 코어 0 |
   | 90 | high_priority_metrics_collector | 1초 | 100ms | 코어 0-1 |
   | 80 | medium_priority_metrics_collector | 5초 | 200ms | 코어 0-3 |
   | 70 | low_priority_metrics_collector | 30초 | 500ms | 코어 0-7 |

2. **TIMPANI 설정 예시**:
   ```
   task_config {
     name: "critical_metrics_collector"
     priority: 95
     policy: "SCHED_FIFO"
     period_ns: 1000000000  # 1초
     deadline_ns: 50000000  # 50ms
     cpu_set: [0]
     allowed_deadline_misses: 0
   }
   
   task_config {
     name: "high_priority_metrics_collector"
     priority: 90
     policy: "SCHED_FIFO"
     period_ns: 1000000000  # 1초
     deadline_ns: 100000000  # 100ms
     cpu_set: [0, 1]
     allowed_deadline_misses: 1
   }
   ```

#### 2.3.3 데이터 전처리 및 검증

수집된 데이터는 전송 전 다음과 같은 처리 과정을 거칩니다:

1. **이상치 감지 및 필터링**:
   - 통계적 방법을 통한 이상치 감지
   - 물리적으로 불가능한 값 필터링
   - 급격한 변화 감지 및 표시

2. **데이터 압축 및 최적화**:
   - 델타 인코딩을 통한 시계열 데이터 압축
   - 중복 제거 및 배치 처리
   - 우선순위 기반 데이터 송신

3. **메타데이터 추가**:
   - 타임스탬프 및 노드 식별자 추가
   - 수집 소스 및 방법 태깅
   - 데이터 신뢰도 지표 포함

#### 2.3.4 데이터 전송 및 저장

1. **전송 프로토콜**:
   - gRPC 기반 스트리밍 전송
   - 프로토콜 버퍼(Protobuf) 직렬화
   - TLS 1.3 암호화

2. **백업 및 복구 메커니즘**:
   - 네트워크 중단 시 로컬 저장
   - 연결 복구 시 자동 동기화
   - 순환 버퍼를 통한 제한된 로컬 저장

3. **저장 정책**:
   - 시계열 데이터베이스(TSDB) 기반 저장
   - 고해상도 데이터: 7일 보관
   - 중해상도 집계 데이터: 30일 보관
   - 저해상도 집계 데이터: 1년 보관

## 3. StateManager 상태 관리 머신

### 3.1 상태 관리 모델

StateManager는 선언적 상태 관리 모델을 구현하여 시스템의 실제 상태와 원하는 상태 간의 차이를 지속적으로 조정합니다.

#### 3.1.1 상태 표현 모델

1. **상태 리소스 정의**:
   ```yaml
   apiVersion: piccolo.io/v1
   kind: ServiceState
   metadata:
     name: service-name
     namespace: system
     generation: 123
   spec:
     # 원하는 상태 정의
     replicas: 2
     version: "1.2.3"
     resources:
       cpu: "2"
       memory: "4Gi"
     config:
       key1: value1
       key2: value2
     dependencies:
       - name: other-service
         minimumVersion: "1.0.0"
     state: Running
   status:
     # 현재 상태 (시스템에 의해 업데이트됨)
     observedGeneration: 122
     replicas: 1
     availableReplicas: 1
     conditions:
       - type: Available
         status: "True"
         lastTransitionTime: "2025-07-16T10:45:02Z"
         reason: "ServiceRunning"
         message: "Service is running normally"
     state: Running
     lastReconcileTime: "2025-07-17T02:15:33Z"
   ```

2. **상태 변경 이벤트**:
   ```yaml
   apiVersion: piccolo.io/v1
   kind: StateEvent
   metadata:
     name: service-name-scale-up
     namespace: system
     timestamp: "2025-07-17T02:20:15Z"
   spec:
     target:
       kind: ServiceState
       name: service-name
       namespace: system
     changes:
       - field: spec.replicas
         oldValue: 1
         newValue: 2
     reason: "ManualScaling"
     source: "admin-dashboard"
     user: "system-operator"
   ```

#### 3.1.2 상태 머신 정의

StateManager는 다음과 같은 상태 머신을 통해 리소스의 생명주기를 관리합니다:

1. **서비스 상태 머신**:

   ```
                 +------------+
                 |            |
          +----->+   Pending  +-----+
          |      |            |     |
          |      +------------+     |
          |                         |
          |                         v
   +------+-------+          +------+-------+
   |              |  scale   |              |
   |  Initializing+--------->+   Running    +<------+
   |              |          |              |       |
   +------+-------+          +------+-------+       |
          |                         |               |
          |                         |               |
          |                         v               |
          |                  +------+-------+       |
          |                  |              |       |
          |                  |   Degraded   +-------+
          |                  |              |
          |                  +------+-------+
          |                         |
          |                         |
          |                         v
   +------+-------+          +------+-------+
   |              |          |              |
   |    Failed    |<---------+  Terminating |
   |              |          |              |
   +--------------+          +--------------+
   ```

2. **상태 전이 정의**:

   | 현재 상태 | 이벤트 | 다음 상태 | 조건 | 액션 |
   |----------|-------|----------|------|------|
   | - | 초기 생성 | Pending | 항상 | 리소스 생성 초기화 |
   | Pending | 초기화 완료 | Initializing | 의존성 충족 | 서비스 컨테이너 생성 |
   | Initializing | 준비 완료 | Running | 모든 컨테이너 실행 중 | 서비스 가용성 확인 |
   | Running | 상태 악화 | Degraded | 부분 가용, 성능 저하 | 복구 절차 시작 |
   | Degraded | 복구 성공 | Running | 정상 성능 회복 | 모니터링 강화 |
   | Degraded | 복구 실패 | Failed | 여러 복구 시도 실패 | 오류 로깅, 알림 생성 |
   | Running | 삭제 요청 | Terminating | 삭제 명령 수신 | 정상 종료 절차 |
   | Terminating | 종료 실패 | Failed | 정상 종료 실패 | 강제 종료, 리소스 정리 |
   | Terminating | 종료 성공 | - | 모든 리소스 정리 완료 | 상태 객체 제거 |
   | Failed | 재시작 요청 | Initializing | 수동 재시작 명령 | 모든 리소스 재생성 |

3. **상태별 제약 조건**:

   | 상태 | 허용된 작업 | 금지된 작업 | 타임아웃 | 최대 재시도 |
   |-----|------------|------------|---------|------------|
   | Pending | 구성 변경, 취소 | 스케일링, 버전 변경 | 5분 | - |
   | Initializing | 취소 | 스케일링, 구성 변경 | 10분 | 3회 |
   | Running | 모든 작업 | - | - | - |
   | Degraded | 재시작, 구성 변경, 취소 | 스케일 아웃 | 30분 | 5회 |
   | Failed | 재시작, 삭제 | 스케일링, 구성 변경 | - | - |
   | Terminating | 강제 종료 | 기타 모든 작업 | 5분 | 1회 |

### 3.2 Reconciliation 프로세스

#### 3.2.1 Reconciliation 루프

StateManager는 다음과 같은 Reconciliation 루프를 통해 시스템의 실제 상태를 원하는 상태로 조정합니다:

1. **Reconciliation 흐름도**:

   ```
   +-------------------+
   | Watch Events      |
   | (변경 감지)        |
   +--------+----------+
            |
            v
   +--------+----------+
   | State Diff        |
   | (상태 차이 계산)   |
   +--------+----------+
            |
            v
   +--------+----------+
   | Action Planning   |
   | (조치 계획 수립)   |
   +--------+----------+
            |
            v
   +--------+----------+     +------------------+
   | Policy Validation +---->+ PolicyManager    |
   | (정책 검증)       |     | (허용/거부/대기)  |
   +--------+----------+     +------------------+
            |
            v
   +--------+----------+
   | Action Execution  |
   | (조치 실행)        |
   +--------+----------+
            |
            v
   +--------+----------+
   | Status Update     |
   | (상태 업데이트)    |
   +--------+----------+
            |
            v
   +--------+----------+
   | Result Reporting  |
   | (결과 보고)        |
   +--------+----------+
   ```

2. **Reconciliation 주기**:

   | 조건 | 재조정 주기 | 최대 지연 | 우선순위 |
   |-----|------------|----------|---------|
   | 상태 변경 이벤트 | 즉시 | 100ms | 높음 |
   | 정기 재조정 (중요 서비스) | 30초 | 1초 | 높음 |
   | 정기 재조정 (일반 서비스) | 5분 | 10초 | 중간 |
   | 실패 후 재시도 | 지수 백오프 (10s, 30s, 1m, 5m) | 30초 | 중간 |
   | 수동 트리거 | 즉시 | 100ms | 최고 |

3. **Reconciliation 동시성 제어**:

   - 서비스별 Reconciliation 락 메커니즘
   - 의존성 기반 Reconciliation 순서 결정
   - 충돌 해결 전략:
     - 최신 세대(generation) 우선
     - 관리자 변경 우선
     - 충돌 시 보수적 접근(안전 우선)

#### 3.2.2 조치 계획 수립

실제 상태와 원하는 상태 간의 차이가 감지되면 다음 단계로 조치 계획을 수립합니다:

1. **상태 차이 유형**:

   | 차이 유형 | 설명 | 우선순위 | 예시 |
   |----------|------|---------|------|
   | 존재 차이 | 리소스 존재/부재 불일치 | 최고 | 서비스가 없지만 있어야 함 |
   | 스케일 차이 | 복제본 수 불일치 | 높음 | 2개 복제본이 필요하나 1개만 실행 중 |
   | 버전 차이 | 소프트웨어 버전 불일치 | 높음 | v1.2가 필요하나 v1.1이 실행 중 |
   | 구성 차이 | 설정값 불일치 | 중간 | 설정 파라미터 값 불일치 |
   | 리소스 차이 | 할당된 리소스 불일치 | 중간 | CPU/메모리 할당량 불일치 |
   | 상태 차이 | 서비스 상태 불일치 | 높음 | Running이어야 하나 Stopped 상태임 |

2. **조치 유형**:

   | 조치 유형 | 설명 | 위험도 | 영향 범위 |
   |----------|------|-------|----------|
   | 생성 | 새 리소스 생성 | 낮음 | 해당 서비스 |
   | 삭제 | 리소스 제거 | 높음 | 해당 서비스 및 의존 서비스 |
   | 스케일 업 | 복제본 증가 | 낮음 | 해당 서비스 |
   | 스케일 다운 | 복제본 감소 | 중간 | 해당 서비스 및 사용자 |
   | 재시작 | 서비스 재시작 | 중간 | 해당 서비스 및 사용자 |
   | 업그레이드 | 버전 변경 | 중간-높음 | 해당 서비스 및 의존 서비스 |
   | 구성 변경 | 설정 업데이트 | 낮음-중간 | 해당 서비스 |
   | 강제 종료 | 서비스 강제 중지 | 매우 높음 | 해당 서비스 및 의존 서비스 |

3. **조치 계획 알고리즘**:

   ```
   function createActionPlan(currentState, desiredState):
     differences = calculateDiff(currentState, desiredState)
     actions = []
     
     # 존재 차이 처리
     if desiredState.exists && !currentState.exists:
       actions.push(createAction("Create", priority=10))
     else if !desiredState.exists && currentState.exists:
       actions.push(createAction("Delete", priority=10))
       return actions  # 삭제면 다른 차이는 무시
     
     # 스케일 차이 처리
     if currentState.replicas < desiredState.replicas:
       actions.push(createAction("ScaleUp", 
                                delta=desiredState.replicas-currentState.replicas, 
                                priority=8))
     else if currentState.replicas > desiredState.replicas:
       actions.push(createAction("ScaleDown", 
                                delta=currentState.replicas-desiredState.replicas, 
                                priority=7))
     
     # 버전 차이 처리
     if currentState.version != desiredState.version:
       actions.push(createAction("Upgrade", 
                                fromVersion=currentState.version, 
                                toVersion=desiredState.version, 
                                priority=8))
     
     # 구성 차이 처리
     configDiffs = findConfigDifferences(currentState.config, desiredState.config)
     if configDiffs.length > 0:
       actions.push(createAction("UpdateConfig", 
                                changes=configDiffs, 
                                priority=6))
     
     # 상태 차이 처리
     if currentState.state != desiredState.state:
       if desiredState.state == "Running" && currentState.state == "Stopped":
         actions.push(createAction("Start", priority=9))
       else if desiredState.state == "Stopped" && currentState.state == "Running":
         actions.push(createAction("Stop", priority=9))
     
     # 우선순위에 따라 정렬
     return sortByPriority(actions)
   ```

#### 3.2.3 정책 검증 및 ActionController 연동

조치 계획이 수립되면 PolicyManager를 통해 정책 검증을 수행합니다:

1. **PolicyManager 연동 흐름**:

   ```
   +-------------------+     +----------------------+
   | StateManager      |     | PolicyManager        |
   | (조치 계획 수립)   +---->+ (정책 기반 평가)      |
   +--------+----------+     +-----------+----------+
            |                            |
            |                            |
            v                            v
   +--------+----------+     +----------------------+
   | ActionController  |<----+ 평가 결과            |
   | (조치 실행)        |     | (허용/거부/대기)     |
   +-------------------+     +----------------------+
   ```

2. **정책 검증 요청 구조**:

   ```json
   {
     "requestId": "reconcile-service-xyz-12345",
     "serviceName": "service-xyz",
     "actionType": "RESTART",
     "vehicleState": "driving",
     "autonomyLevel": "level3",
     "requiredDevices": ["gpu", "display"],
     "urgencyLevel": 8,
     "metadata": {
       "reason": "configuration_change",
       "changeId": "config-update-78901",
       "affectedUsers": 5
     }
   }
   ```

3. **정책 검증 결과 처리**:

   | 결정 | 처리 방법 | 재시도 전략 |
   |------|----------|------------|
   | ALLOW | 즉시 조치 실행 | - |
   | PENDING | 지정된 시간 후 재시도 | 지수 백오프 (최대 5회) |
   | DENY | 조치 취소 및 로그 | 상태 불일치 지속, 알림 생성 |

4. **ActionController 연동 인터페이스**:

   ```
   interface ActionExecution {
     // 조치 실행 요청
     ActionResult executeAction(Action action);
     
     // 실행 중인 조치 상태 조회
     ActionStatus getActionStatus(string actionId);
     
     // 실행 중인 조치 취소
     boolean cancelAction(string actionId);
   }
   
   // 조치 정보
   struct Action {
     string actionId;            // 조치 고유 ID
     string serviceName;         // 대상 서비스 이름
     ActionType type;            // 조치 유형
     map<string, string> params; // 조치 매개변수
     int priority;               // 우선순위 (1-10)
     int timeoutSeconds;         // 실행 타임아웃
     string reason;              // 조치 이유
   }
   
   // 조치 결과
   struct ActionResult {
     string actionId;            // 조치 ID
     ActionStatus status;        // 조치 상태
     string message;             // 상세 메시지
     int completionPercent;      // 완료 백분율
     map<string, string> details;// 상세 결과 정보
   }
   
   enum ActionStatus {
     PENDING,   // 대기 중
     RUNNING,   // 실행 중
     SUCCEEDED, // 성공
     FAILED,    // 실패
     CANCELED   // 취소됨
   }
   ```

### 3.3 오류 처리 및 복구 전략

#### 3.3.1 상태 불일치 유형 및 복구 전략

| 불일치 유형 | 감지 방법 | 복구 전략 | 최대 재시도 |
|------------|----------|----------|------------|
| 종료된 컨테이너 | NodeAgent 메트릭 | 자동 재시작 | 제한 없음 |
| 리소스 부족 | 리소스 메트릭 모니터링 | 리소스 재할당 또는 낮은 우선순위 서비스 축소 | 5회 |
| 응답 없음 | 상태 점검 실패 | 서비스 재시작 | 3회 |
| 구성 불일치 | 구성 검증 | 구성 재적용 | 3회 |
| 버전 불일치 | 버전 확인 | 롤백 또는 강제 업그레이드 | 2회 |
| 정지된 업데이트 | 타임아웃 모니터링 | 업데이트 재개 또는 롤백 | 1회 |
| 순환 재시작 | 패턴 감지 | 이전 안정 버전으로 롤백 | 0회 (즉시 롤백) |

#### 3.3.2 자동 복구 메커니즘

1. **단계적 복구 프로세스**:

   ```
   function attemptRecovery(service, issue):
     # 1단계: 경미한 복구 시도
     if tryMinorRecovery(service, issue):
       return SUCCESS
     
     # 2단계: 서비스 재시작
     if tryServiceRestart(service):
       return SUCCESS
     
     # 3단계: 구성 리셋 및 재시작
     if tryConfigResetAndRestart(service):
       return SUCCESS
     
     # 4단계: 이전 버전으로 롤백
     if tryRollbackToPreviousVersion(service):
       return SUCCESS
     
     # 5단계: 격리 모드로 전환
     enableIsolationMode(service)
     notifyAdministrator(service, issue, "Critical recovery failure")
     return FAILURE
   ```

2. **서비스 유형별 복구 전략**:

   | 서비스 유형 | 1차 복구 | 2차 복구 | 3차 복구 | 최종 조치 |
   |------------|---------|---------|---------|----------|
   | 안전 중요 | 즉시 복제본 전환 | 빠른 재시작 | 구성 롤백 | 안전 모드 전환 |
   | 시스템 핵심 | 소프트 재시작 | 하드 재시작 | 버전 롤백 | 최소 기능 모드 |
   | 사용자 중요 | 구성 리로드 | 소프트 재시작 | 하드 재시작 | 대체 서비스 제공 |
   | 일반 | 구성 리로드 | 재시작 | 일시 중단 | 서비스 비활성화 |
   | 백그라운드 | 일시 중단 후 재시도 | 재시작 | 낮은 우선순위로 재시작 | 비활성화 |

3. **복구 시간 목표 (RTO)**:

   | 서비스 중요도 | 최대 허용 중단 시간 | 복구 우선순위 |
   |--------------|-------------------|-------------|
   | 안전 필수 | <500ms | 10 (최고) |
   | 시스템 핵심 | <2s | 9 |
   | 사용자 중요 | <5s | 8 |
   | 일반 | <30s | 5 |
   | 백그라운드 | <5m | 3 |

#### 3.3.3 수동 개입 시나리오

일부 상황에서는 자동 복구가 불가능하여 수동 개입이 필요합니다:

1. **수동 개입 트리거 조건**:

   | 조건 | 임계값 | 알림 수준 |
   |------|-------|----------|
   | 연속 복구 실패 | 5회 | 심각 |
   | 24시간 내 반복 문제 | 10회 | 높음 |
   | 비정상 패턴 감지 | 패턴 기반 | 중간-높음 |
   | 리소스 누수 감지 | 패턴 기반 | 높음 |
   | 보안 관련 문제 | 즉시 | 심각 |

2. **수동 개입 인터페이스**:

   ```
   interface ManualIntervention {
     // 수동 복구 명령
     ActionResult performRecovery(string serviceId, RecoveryType type, map<string, string> params);
     
     // 서비스 강제 재설정
     ActionResult forceReset(string serviceId, boolean preserveData);
     
     // 우선순위 오버라이드
     boolean overridePriority(string serviceId, int newPriority, int durationMinutes);
     
     // 유지보수 모드 설정
     ActionResult setMaintenanceMode(string serviceId, boolean enabled, int durationMinutes);
   }
   
   enum RecoveryType {
     RESTART,
     ROLLBACK,
     RECREATE,
     RESET_CONFIG,
     SCALE_ADJUST,
     ISOLATE,
     CUSTOM
   }
   ```

3. **수동 개입 로깅 및 감사**:

   모든 수동 개입은 다음 정보와 함께 상세히 기록됩니다:
   - 개입 시간 및 사용자
   - 개입 유형 및 대상
   - 개입 이유
   - 수행된 조치
   - 결과 및 영향
   - 복구 시간

### 3.4 상태 모니터링 및 보고

#### 3.4.1 상태 대시보드

StateManager는 다음과 같은 상태 대시보드를 제공합니다:

1. **시스템 상태 개요**:
   - 전체 서비스 상태 요약
   - 주요 시스템 메트릭 실시간 그래프
   - 활성 알림 및 이벤트
   - 최근 Reconciliation 활동

2. **서비스별 상태 보기**:
   - 서비스 상태 및 건전성
   - 리소스 사용 추세
   - 구성 상태 및 변경 이력
   - 관련 알림 및 이벤트

3. **Reconciliation 활동 로그**:
   - 최근 조정 활동 목록
   - 성공/실패 상태 및 이유
   - 소요 시간 및 영향
   - 관련 조치 상세 정보

#### 3.4.2 알림 및 이벤트

1. **알림 수준**:

   | 수준 | 설명 | 대응 시간 | 알림 방법 |
   |------|------|----------|----------|
   | 심각 | 시스템 기능 심각한 손상 | 즉시 | 푸시, SMS, 이메일, 경보 |
   | 높음 | 주요 기능 손상 | <15분 | 푸시, SMS, 이메일 |
   | 중간 | 서비스 성능 저하 | <1시간 | 푸시, 이메일 |
   | 낮음 | 경미한 이슈, 정보성 | <24시간 | 이메일, 대시보드 |

2. **이벤트 유형**:

   | 유형 | 설명 | 로깅 수준 | 보존 기간 |
   |------|------|----------|----------|
   | 상태 변경 | 서비스 상태 전이 | INFO | 30일 |
   | 스케일 변경 | 서비스 규모 조정 | INFO | 30일 |
   | 구성 변경 | 설정 변경 | INFO | 90일 |
   | 버전 변경 | 소프트웨어 버전 변경 | INFO | 90일 |
   | 오류 | 서비스 오류 발생 | ERROR | 1년 |
   | 복구 | 복구 작업 수행 | INFO | 90일 |
   | 보안 | 보안 관련 이벤트 | WARN/ERROR | 1년 |
   | 사용자 작업 | 관리자 조치 | INFO | 1년 |

#### 3.4.3 상태 보고서 및 추세 분석

1. **정기 보고서**:
   - 일일/주간/월간 상태 요약
   - 서비스 가용성 및 성능 통계
   - 주요 이벤트 및 문제 요약
   - 리소스 사용 추세

2. **추세 분석**:
   - 서비스 성능 장기 추세
   - 오류 패턴 및 빈도 분석
   - 리소스 사용 예측
   - 상태 전이 패턴 분석

## 4. 결론

본 문서는 PICCOLO 프레임워크의 상태관리 기능에 대한 상세 명세를 제공합니다. NodeAgent는 컨테이너 및 하드웨어 시스템 메트릭을 수집하여 시스템의 실제 상태를 모니터링하고, StateManager는 상태 머신을 통해 실제 상태와 원하는 상태 간의 차이를 감지하고 reconciliation을 수행합니다. 이 두 컴포넌트의 유기적인 연동을 통해 PICCOLO 프레임워크는 안정적이고 자가 복구 가능한 시스템 운영을 보장합니다.

## 부록

### A. 용어 정의

| 용어 | 정의 |
|------|------|
| NodeAgent | 개별 노드에서 실행되어 시스템 메트릭과 컨테이너 상태를 수집하는 컴포넌트 |
| StateManager | 시스템의 상태를 관리하고 reconciliation을 수행하는 중앙 컴포넌트 |
| TIMPANI | 결정적 태스크 스케줄링을 제공하는 실시간 스케줄러 |
| Reconciliation | 실제 상태와 원하는 상태의 차이를 조정하는 프로세스 |
| PolicyManager | 시스템 정책을 관리하고 조치의 허용 여부를 판단하는 컴포넌트 |
| ActionController | StateManager의 결정에 따라 실제 시스템 조치를 실행하는 컴포넌트 |

### B. 관련 문서

1. PICCOLO-POLICY-2025-001: PolicyManager 상세 설계 문서
2. PICCOLO-ACTION-2025-003: ActionController 기능 명세서
3. PICCOLO-MONITOR-2025-004: 모니터링 시스템 아키텍처
4. PICCOLO-SECURITY-2025-005: 보안 아키텍처 문서
