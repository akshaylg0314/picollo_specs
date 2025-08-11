# TIMPANI Scheduler (SW Center is in charge of )

## 1. Introduction

### Major features

TIMPANI Scheduler는 PICCOLO 프레임워크에서 시간 결정적 태스크 실행을 관리하는 컴포넌트이다.

1. 주기적 태스크 스케줄링, 데드라인 기반 실행, 지터(jitter) 최소화를 포함한 결정적 스케줄링을 제공한다.
2. CPU 코어 할당 및 격리, 인터럽트 라우팅 최적화, 실시간 우선순위 관리를 통해 CPU 리소스를 관리한다.
3. 실행 시간 모니터링, 타이밍 위반 감지, 성능 분석 및 최적화를 위한 타이밍 분석 기능을 제공한다.
4. 안전 중요 태스크를 격리하고, 우선순위 상속 및 천장 메커니즘을 적용하며, 결정적 동작을 보장한다.
5. 다양한 실시간 스케줄링 알고리즘을 지원하여 다양한 워크로드 요구 사항을 충족한다.
6. Linux 커널의 실시간 스케줄링 기능을 활용하고 필요한 경우 확장한다.

### Main Dataflow

1. ActionController로부터 컨테이너 스케줄링 매개변수를 수신한다.
2. CPU 분리 및 인터럽트 라우팅을 설정한다.
3. 주기적 태스크의 실행 일정을 계획하고 관리한다.
4. 태스크 실행 시간 및 데드라인 준수 여부를 모니터링한다.
5. 성능 데이터를 수집하여 MonitoringServer에 제공한다.
6. 타이밍 위반 시 ApiServer에 알림을 전송한다.

## 2. Reference information

### 2.1 관련 컴포넌트
- **ActionController**: TIMPANI 스케줄러에 태스크 실행 요청 전달
- **APIServer**: 타이밍 위반 및 스케줄러 상태 알림 수신
- **MonitoringServer**: 스케줄러 성능 메트릭 수집

### 2.2 프로토콜 및 API
- **gRPC API**: ActionController 및 APIServer와의 통신을 위한 gRPC 서비스
- **커널 스케줄링 API**: Linux 실시간 스케줄링 인터페이스
- **cgroup API**: CPU 리소스 제어를 위한 cgroups 인터페이스
- **모니터링 API**: 성능 메트릭 수집을 위한 인터페이스


### 4.4 참고 자료
- [Linux 실시간 스케줄링 문서](https://man7.org/linux/man-pages/man7/sched.7.html)
- [SCHED_DEADLINE 문서](https://www.kernel.org/doc/html/latest/scheduler/sched-deadline.html)
- [cgroups v2 문서](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)
- [CPU 격리 기법](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-cpu-isolating_cpus)
- [태스크 스케줄링 이론](https://en.wikipedia.org/wiki/Scheduling_(computing))
