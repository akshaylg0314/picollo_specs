# PICCOLO (PULLPIRI) 소프트웨어 설계 문서 (Software Design Documents)

이 디렉토리는 PICCOLO(PULLPIRI) 프레임워크의 각 컴포넌트에 대한 상세 설계 문서를 포함합니다.

## 프로젝트 개요

PICCOLO는 Eclipse PULLPIRI 프로젝트의 내부 코드명으로, 차량 환경에 최적화된 OS 독립적 컨테이너 프레임워크입니다. PULLPIRI는 차량 서비스 오케스트레이터 프레임워크로서 클라우드 네이티브 기술의 이점을 차량 내 서비스와 애플리케이션에 적용하는 것을 목표로 합니다.

## 컴포넌트 문서

### 시스템 관리 컴포넌트
- [`statemanager.md`](statemanager.md) - 시스템 및 서비스의 상태를 관리하고 조정하는 StateManager
- [`settingsservice.md`](settingsservice.md) - 시스템 설정 관리를 위한 UI 및 CLI를 제공하는 SettingsService
- [`policymanager.md`](policymanager.md) - 서비스 우선순위 결정과 허용/대기/거부 여부를 판단하는 PolicyManager

### 실행 관리 컴포넌트
- [`actioncontroller.md`](actioncontroller.md) - 컨테이너 라이프사이클을 관리하는 ActionController
- [`apiserver.md`](apiserver.md) - API 제공 및 Artifact 등록/준비를 담당하는 APIServer
- [`filtergateway.md`](filtergateway.md) - 서비스 요청을 필터링하고 라우팅하는 FilterGateway

### 모니터링 및 노드 관리 컴포넌트
- [`monitoringserver.md`](monitoringserver.md) - 메트릭 수집 및 분석을 담당하는 MonitoringServer
- [`nodeagent.md`](nodeagent.md) - 각 노드의 상태를 모니터링하고 관리하는 NodeAgent

### 네트워킹 및 스케줄링 컴포넌트
- [`timpanischeduler.md`](timpanischeduler.md) - 실시간 애플리케이션을 위한 스케줄링을 제공하는 TIMPANI Scheduler
- [`pharosnetworking.md`](pharosnetworking.md) - eBPF 기반의 네트워크 오케스트레이션 기능을 제공하는 컴포넌트

## 최근 아키텍처 변경 사항

### 공통 ETCD 라이브러리 통합 (2025-07-25)
모든 컴포넌트에서 사용하던 개별 ETCD 클라이언트 구현을 `common/etcd` 패키지의 공통 라이브러리로 통합했습니다. 
이를 통해 코드 중복을 줄이고, 일관된 ETCD 연동 방식을 제공하며, 향후 ETCD 관련 기능 확장 및 유지보수를 용이하게 했습니다.

**주요 변경사항:**
- ETCD 클라이언트 연결, 읽기/쓰기, 감시 기능 표준화
- JSON/YAML 직렬화/역직렬화 통합 인터페이스 제공
- 트랜잭션 및 일괄 처리 기능 강화
- 연결 오류 및 재시도 처리 로직 표준화
- 성능 최적화 및 메모리 사용량 개선

**변경된 파일:**
- `statemanager` - `etcd/client.rs` → `common/etcd` 라이브러리 사용
- `settingsservice` - `etcd/client.rs` → `common/etcd` 라이브러리 사용
- `policymanager` - `etcd/client.rs` → `common/etcd` 라이브러리 사용
- `actioncontroller` - `storage/etcd.rs` → `common/etcd` 라이브러리 사용
- `apiserver` - `artifact/data.rs` ETCD 관련 함수 → `common/etcd` 라이브러리 사용
- `monitoringserver` - `metrics/store.rs` ETCD 관련 함수 → `common/etcd` 라이브러리 사용

### 클라우드 통신 아키텍처 개선 (2025-07-25)
`monitoringserver`에서 CloudManager 관련 참조를 제거하고 클라우드 통신 기능을 독립적인 서비스로 분리했습니다. 
이를 통해 각 컴포넌트의 책임을 명확히 하고, 클라우드 연결 관련 기능을 중앙화하여 관리할 수 있게 되었습니다.

**주요 변경사항:**
- MonitoringServer에서 클라우드 통신 관련 코드 제거
- CloudManager 컴포넌트로 클라우드 통신 책임 이전
- 표준화된 메트릭 전송 인터페이스 도입
- 클라우드 연결 상태에 따른 로컬 버퍼링 기능 강화

### 런타임 어댑터 패턴 도입 (2025-07-15)
ActionController에 런타임 어댑터 패턴을 도입하여 다양한 컨테이너 런타임(Podman, Docker, Containerd)을 유연하게 지원할 수 있도록 개선했습니다.

### 보안 컨텍스트 및 권한 관리 강화 (2025-07-10)
APIServer와 ActionController에 보안 컨텍스트 및 권한 관리 기능을 추가하여 시스템의 안전성을 높였습니다.

## 문서 작성 규칙

각 설계 문서는 다음 구조를 따릅니다:

1. **Introduction** - 컴포넌트 개요, 주요 기능 및 기본 데이터 흐름
2. **File information** - 디렉토리 구조 및 파일 설명
3. **Function information** - 주요 함수 및 메소드 설명 (rustdoc 형식)
4. **Reference information** - 관련 컴포넌트, 프로토콜, API 및 데이터 구조 참조

문서는 마크다운(`.md`) 형식으로 작성되며, 코드 예제는 rustdoc 주석 스타일을 따릅니다.

## 개발 환경 구성

PICCOLO(PULLPIRI) 개발 환경 설정에 관한 자세한 정보는 [공식 GitHub 저장소](https://github.com/eclipse-pullpiri/pullpiri/tree/refactoring)의 문서를 참조하세요.
