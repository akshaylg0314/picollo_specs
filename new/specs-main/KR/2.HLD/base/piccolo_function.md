# PICCOLO 프레임워크 기능 명세서

**문서 정보:**
- 문서 ID: PICCOLO-SPEC-2025-FULL
- 버전: 1.0
- 날짜: 2025-07-08 05:13:35
- 작성자: edo-lee_LGESDV
- 분류: 기능 통합 명세서

## 목차

1. [개요](#1-개요)
2. [핵심 기능](#2-핵심-기능)
3. [컴포넌트별 기능](#3-컴포넌트별-기능)
4. [리소스 정의 및 관리](#4-리소스-정의-및-관리)
5. [특화 관리자](#5-특화-관리자)
   - [DeviceManager](#51-devicemanager-상세)
   - [AudioManager](#52-audiomanager-상세)
   - [WindowManager](#53-windowmanager-상세)
   - [CloudManager](#54-cloudmanager-상세)
   - [NetworkManager (Pharos)](#54-networkmanager)
   - [StorageManager](#56-storagemanager-상세)
6. [운영 시나리오](#6-운영-시나리오)
7. [사용자 인터페이스](#7-사용자-인터페이스)
8. [보안 및 안전성](#8-보안-및-안전성)
9. [통합 및 확장성](#9-통합-및-확장성)
10. [성능 및 확장성](#10-성능-및-확장성)
11. [구현 제약사항](#11-구현-제약사항)
12. [결론](#12-결론)
13. [부록: 아키텍처 다이어그램](#13-부록-아키텍처-다이어그램)

## 1. 개요

PICCOLO는 차량 환경에 최적화된 OS 독립적 컨테이너 프레임워크로, Podman을 기반으로 하여 Kubernetes 스타일의 선언적 접근 방식을 통해 차량 소프트웨어 서비스를 관리합니다. 본 문서는 PICCOLO 프레임워크의 기능적 요구사항과 기능을 설명합니다.

### 1.1 목적

PICCOLO는 다음과 같은 목적을 가지고 설계되었습니다:

- 다양한 운영체제에서 일관된 서비스 관리 제공
- 차량 환경에 최적화된 컨테이너 기반 서비스 배포 및 관리
- 선언적 구성을 통한 서비스 라이프사이클 관리 간소화
- 안전 중요 시스템을 위한 결정적 실행 환경 제공
- 시스템 상태 및 성능에 대한 포괄적인 모니터링
- 차량 상태에 따른 동적 서비스 구성 제공

### 1.2 적용 범위

본 문서는 PICCOLO 프레임워크의 다음 기능을 다룹니다:

- 핵심 컴포넌트 기능
- 리소스 정의 및 관리
- 시나리오 기반 워크로드 관리
- 사용자 인터페이스 및 설정 관리
- 모니터링 및 로깅
- 실시간 제어 및 네트워킹 기능
- 보안 및 안전성 기능
- 특화 관리자 (DeviceManager, AudioManager, WindowManager)

## 2. 핵심 기능

### 2.1 기능 요약 테이블

| 기능 | 설명 | 주요 특징 |
|------|------|-----------|
| 선언적 리소스 관리 | YAML 기반 리소스 정의 및 관리 | • Scenario, Package, Model, Volume, Network<br>• ETCD 기반 상태 저장<br>• 버전 관리 |
| 컨테이너 라이프사이클 관리 | Podman 기반 컨테이너 관리 | • 생성, 시작, 중지, 재시작, 삭제<br>• 리소스 제한 및 할당<br>• 볼륨 및 네트워크 관리 |
| 차량 상태 기반 시나리오 | 차량 상태에 따른 서비스 관리 | • 조건부 서비스 활성화<br>• 동적 워크로드 조정<br>• 우선순위 기반 관리 |
| 모니터링 및 원격 관리 | 시스템 상태 모니터링 및 관리 | • 컨테이너/하드웨어 메트릭 수집<br>• 클라우드 데이터 전송<br>• 알림 및 경고 관리 |
| 설정 관리 인터페이스 | UI 및 CLI 기반 설정 관리 | • 구조적 리소스 탐색/편집<br>• 템플릿 기반 리소스 생성<br>• 버전 관리 및 롤백 |
| 실시간 제어 및 네트워킹 | 시간 결정적 실행 환경 제공 | • TIMPANI 결정적 스케줄링<br>• PHAROS eBPF 기반 네트워킹<br>• QoS 보장 |

### 2.2 선언적 리소스 관리

PICCOLO는 다음과 같은 선언적 리소스를 통해 차량 서비스를 정의하고 관리합니다:

- **Scenario**: 차량 상태 조건, 대상 패키지, 액션을 정의
- **Package**: 함께 배포되는 모델(컨테이너) 그룹 정의
- **Model**: 개별 컨테이너화된 서비스 정의
- **Volume**: 영구 데이터 저장소 정의
- **Network**: 네트워크 구성 및 연결 정의

이러한 리소스는 YAML 형식으로 정의되며, ETCD에 저장되어 시스템 전체에서 일관된 상태를 유지합니다.

### 2.3 컨테이너 라이프사이클 관리

Podman을 기반으로 한 컨테이너 라이프사이클 관리 기능을 제공합니다:

- 컨테이너 생성, 시작, 중지, 재시작, 삭제
- 리소스 제한 및 할당 관리
- 환경 변수 및 구성 관리
- 볼륨 마운트 및 데이터 지속성
- 네트워크 구성 및 연결 관리
- 컨테이너 상태 모니터링 및 로깅

### 2.4 차량 상태 기반 시나리오

차량의 현재 상태에 따라 자동으로 서비스를 시작, 업데이트 또는 중지할 수 있는 시나리오 기반 관리:

- 차량 속도, 점화 상태, 배터리 수준 등의 조건에 따른 서비스 활성화
- 사용자 상호작용 또는 시스템 이벤트에 대응하는 동적 서비스 구성
- 리소스 최적화를 위한 자동 워크로드 조정
- 안전 중요 상황에서의 우선순위 기반 서비스 관리

### 2.5 모니터링 및 원격 관리

컨테이너와 시스템 상태를 종합적으로 모니터링하고 관리하는 기능:

- 컨테이너 리소스 사용량 모니터링 (CPU, 메모리, 디스크, 네트워크)
- 하드웨어 시스템 상태 모니터링 (온도, 전압, 팬 속도, 스토리지)
- 로그 수집, 집계 및 분석
- 클라우드 서비스로의 데이터 전송
- 알림 및 경고 관리
- 대시보드 및 시각화

### 2.6 설정 관리 인터페이스

시스템 구성을 쉽게 관리할 수 있는 UI 및 CLI 인터페이스:

- 리소스 구조적 탐색 및 편집
- 템플릿 기반 새 리소스 생성
- 변경 사항 실시간 검증 및 적용
- 버전 이력 관리 및 롤백
- 권한 기반 접근 제어

### 2.7 실시간 제어 및 네트워킹

안전 중요 기능을 위한 시간 결정적 실행 환경:

- TIMPANI 스케줄러를 통한 결정적 태스크 실행
- CPU 분리 및 실시간 우선순위 지정
- PHAROS 네트워킹을 통한 eBPF 기반 고속 패킷 처리
- 패킷 필터링, 라우팅 및 방화벽 기능
- QoS(Quality of Service) 보장

## 3. 컴포넌트별 기능

### 3.1 컴포넌트 기능 요약 테이블

| 컴포넌트 | 주요 기능 | 인터페이스 |
|----------|-----------|------------|
| APIServer | • REST API 제공<br>• 인증 및 권한 부여<br>• 요청 처리<br>• 모니터링 및 진단 | • REST API<br>• gRPC<br>• WebSocket |
| FilterGateway | • 다중 프로토콜 지원<br>• 데이터 필터링<br>• 메시지 라우팅<br>• 데이터 버퍼링 | • 차량 통신 인터페이스<br>• 내부 API |
| PolicyManager | • 안전 정책 관리<br>• 보안 정책 관리<br>• 리소스 정책 관리<br>• 정책 위반 대응 | • 내부 API<br>• ETCD 감시 |
| ActionController | • 컨테이너 라이프사이클 관리<br>• 명령 변환<br>• 리소스 구성<br>• 실시간 제어 통합 | • Podman API<br>• 내부 API |
| StateManager | • 상태 모니터링<br>• 상태 관리<br>• 자동 복구<br>• 상태 보고 | • 내부 API<br>• ETCD 감시 |
| NodeAgent | • 파일 관리<br>• 구성 배포<br>• 로컬 리소스 관리<br>• 노드 상태 보고 | • 파일 시스템 API<br>• 내부 API |
| MonitoringServer | • 메트릭 수집<br>• 로그 관리<br>• 알림 관리<br>• 클라우드 통합<br>• 시각화 | • 모니터링 API<br>• 클라우드 API |
| SettingsService | • 설정 관리<br>• UI 인터페이스<br>• CLI 인터페이스<br>• 변경 관리<br>• 사용자 관리 | • Web UI<br>• CLI<br>• REST API |
| TIMPANI 스케줄러 | • 결정적 스케줄링<br>• CPU 관리<br>• 타이밍 분석<br>• 안전 중요 워크로드 관리 | • 커널 스케줄링 API<br>• 내부 API |
| PHAROS 네트워킹 | • eBPF 기반 패킷 처리<br>• 패킷 필터링<br>• 트래픽 제어<br>• 네트워크 텔레메트리 | • eBPF/XDP API<br>• 네트워크 스택 API |

### 3.2 APIServer

외부 시스템과의 통신 인터페이스를 제공하는 컴포넌트:

- **REST API 제공**
  - 리소스 생성, 조회, 수정, 삭제를 위한 엔드포인트
  - 버전 관리된 API 지원
  - 구조화된 응답 형식 (JSON/YAML)

- **인증 및 권한 부여**
  - 토큰 기반 인증
  - 역할 기반 접근 제어
  - API 키 관리

- **요청 처리**
  - 요청 검증 및 라우팅
  - 요청 속도 제한
  - 로드 밸런싱

- **모니터링 및 진단**
  - API 사용량 통계
  - 성능 지표 수집
  - 진단 엔드포인트

### 3.3 FilterGateway

차량 데이터를 수신하고 처리하는 컴포넌트:

- **다중 프로토콜 지원**
  - CAN, FlexRay, Ethernet 등 차량 통신 프로토콜 지원
  - 프로토콜 변환 및 정규화

- **데이터 필터링**
  - 규칙 기반 데이터 필터링
  - 중복 데이터 제거
  - 데이터 변환 및 보강

- **메시지 라우팅**
  - 우선순위 기반 메시지 처리
  - 목적지 기반 메시지 라우팅
  - 이벤트 기반 메시지 배포

- **데이터 버퍼링**
  - 피크 로드 관리를 위한 버퍼링
  - 네트워크 중단 시 데이터 캐싱
  - 배치 처리 최적화

### 3.4 PolicyManager

안전 및 보안 정책을 관리하고 적용하는 컴포넌트:

- **안전 정책 관리**
  - ISO 26262 준수 검증
  - 안전 중요 기능 모니터링
  - 기능 안전 상태 평가

- **보안 정책 관리**
  - 접근 제어 정책 적용
  - 데이터 보호 정책 관리
  - 네트워크 보안 정책 적용

- **리소스 정책 관리**
  - 리소스 할당 및 제한 정책
  - 공정한 리소스 공유 보장
  - 우선순위 기반 리소스 관리

- **정책 위반 대응**
  - 위반 감지 및 보고
  - 자동 복구 조치 실행
  - 위반 이력 관리

### 3.5 ActionController

Podman 명령을 실행하여 컨테이너를 관리하는 컴포넌트:

- **컨테이너 라이프사이클 관리**
  - 컨테이너 생성, 시작, 중지, 재시작, 제거
  - 이미지 관리 (가져오기, 삭제)
  - 컨테이너 상태 모니터링

- **명령 변환**
  - 선언적 명세를 Podman 명령으로 변환
  - 명령 실행 및 결과 처리
  - 오류 처리 및 재시도

- **리소스 구성**
  - CPU, 메모리, 스토리지 할당
  - 네트워크 구성
  - 볼륨 마운트

- **실시간 제어 통합**
  - TIMPANI 스케줄링 매개변수 적용
  - 결정적 실행을 위한 CPU 격리
  - 실시간 우선순위 구성

### 3.6 StateManager

컨테이너 및 시스템 상태를 관리하는 컴포넌트:

- **상태 모니터링**
  - 컨테이너 상태 실시간 추적
  - 리소스 사용량 모니터링
  - 이벤트 및 상태 변경 감지

- **상태 관리**
  - 상태 불일치 해결
  - 장애 감지 및 복구
  - 상태 이력 추적

- **자동 복구**
  - 장애 컨테이너 재시작
  - 비정상 상태 감지 및 조치
  - 그레이스풀 종료 관리

- **상태 보고**
  - 상태 요약 생성
  - 상태 변경 알림
  - 상태 쿼리 인터페이스

### 3.7 NodeAgent

노드 파일 시스템 및 리소스를 관리하는 컴포넌트:

- **파일 관리**
  - 파일 생성, 수정, 삭제
  - 디렉토리 구조 관리
  - 파일 권한 관리

- **구성 배포**
  - 구성 파일 배포
  - 설정 업데이트
  - 시스템 환경 구성

- **로컬 리소스 관리**
  - 디스크 공간 관리
  - 로컬 캐시 관리
  - 임시 파일 정리

- **노드 상태 보고**
  - 하드웨어 상태 모니터링
  - 리소스 사용량 보고
  - 노드 가용성 보고

### 3.8 MonitoringServer

시스템 메트릭 및 로그를 수집하고 분석하는 컴포넌트:

- **메트릭 수집**
  - 컨테이너 메트릭 (CPU, 메모리, 디스크, 네트워크)
  - 시스템 메트릭 (하드웨어 상태, 온도, 전압)
  - 사용자 정의 메트릭

- **로그 관리**
  - 로그 수집 및 집계
  - 로그 필터링 및 검색
  - 로그 회전 및 보관

- **알림 관리**
  - 임계값 기반 알림
  - 이상 감지 알림
  - 알림 라우팅 및 에스컬레이션

- **클라우드 통합**
  - 클라우드 서비스로 데이터 전송
  - 오프라인 캐싱 및 동기화
  - 데이터 압축 및 최적화

- **시각화**
  - 대시보드 및 그래프
  - 실시간 모니터링 뷰
  - 보고서 생성

### 3.9 SettingsService

시스템 설정 관리를 위한 UI 및 CLI 인터페이스를 제공하는 컴포넌트:

- **설정 관리**
  - YAML 구성 로드 및 저장
  - 설정 검증 및 적용
  - 설정 이력 관리

- **UI 인터페이스**
  - 계층적 리소스 탐색
  - 속성 기반 편집기
  - 템플릿 라이브러리
  - 드래그 앤 드롭 리소스 관리
  - 실시간 검증 피드백

- **CLI 인터페이스**
  - 명령줄 기반 리소스 관리
  - 배치 작업 및 스크립팅 지원
  - 파이프라인 통합
  - 원격 관리 지원
  - JSON/YAML 포맷 지원

- **변경 관리**
  - 변경 검증 및 충돌 감지
  - 원자적 변경 적용
  - 롤백 기능

- **사용자 관리**
  - 권한 기반 접근 제어
  - 사용자 인증 및 권한 부여
  - 변경 감사 로깅

### 3.10 TIMPANI 스케줄러

시간 결정적 태스크 실행을 관리하는 컴포넌트:

- **결정적 스케줄링**
  - 주기적 태스크 스케줄링
  - 데드라인 기반 실행
  - 지터(jitter) 최소화

- **CPU 관리**
  - CPU 코어 할당 및 격리
  - 인터럽트 라우팅 최적화
  - 실시간 우선순위 관리

- **타이밍 분석**
  - 실행 시간 모니터링
  - 타이밍 위반 감지
  - 성능 분석 및 최적화

- **안전 중요 워크로드 관리**
  - 안전 중요 태스크 격리
  - 우선순위 상속 및 천장
  - 결정적 동작 보장

### 3.11 PHAROS 네트워킹

고성능, 저지연 네트워킹을 제공하는 컴포넌트:

- **eBPF 기반 패킷 처리**
  - XDP(eXpress Data Path) 활용
  - 고속 패킷 처리
  - 커널 바이패스 최적화

- **패킷 필터링**
  - 정밀한 패킷 필터 규칙
  - 프로토콜 및 포트 기반 필터링
  - 상태 기반 패킷 검사

- **트래픽 제어**
  - QoS(Quality of Service) 관리
  - 대역폭 제한 및 할당
  - 우선순위 기반 패킷 스케줄링

- **네트워크 텔레메트리**
  - 패킷 흐름 모니터링
  - 네트워크 성능 측정
  - 이상 트래픽 감지

## 4. 리소스 정의 및 관리

### 4.1 리소스 유형 요약 테이블

| 리소스 | 설명 | 주요 속성 |
|--------|------|-----------|
| Scenario | 차량 상태 조건에 따른 서비스 활성화 규칙 | • 조건(condition)<br>• 액션(action)<br>• 대상(target) |
| Package | 함께 배포되는 Model 그룹 | • 패턴(pattern)<br>• 모델(models)<br>• 리소스 연결 |
| Model | 개별 컨테이너화된 서비스 | • 컨테이너 정의<br>• 리소스 요청/제한<br>• 스케줄링 구성 |
| Volume | 영구 데이터 저장 정의 | • 크기(size)<br>• 접근 모드(accessModes)<br>• 재사용 정책 |
| Network | 컨테이너 네트워크 정의 | • 유형(type)<br>• QoS 설정<br>• 패킷 필터링 규칙 |

### 4.2 Scenario 리소스

차량 상태 조건에 따라 서비스를 활성화, 업데이트 또는 제거하는 규칙을 정의합니다.

**기능:**
- 차량 상태 조건 정의 (속도, 점화 상태, 배터리 수준 등)
- 조건 충족 시 실행할 액션 지정 (launch, update, delete)
- 타겟 Package 지정
- 우선순위 및 의존성 관리
- 중첩 조건 및 복합 조건 지원

**템플릿:**
```yaml
apiVersion: piccolo.ai/v1
kind: Scenario
metadata:
  name: "{{SCENARIO_NAME}}"        # 시나리오 리소스 이름 (예: "monitoring-scenario")
  namespace: "{{NAMESPACE}}"       # 네임스페이스 (예: "default", "monitoring")
  labels:
    app: "{{APP_LABEL}}"           # 애플리케이션 라벨 (예: "monitoring")
spec:
  description: "{{DESCRIPTION}}"   # 시나리오 설명 (예: "CPU 사용량 모니터링 시나리오")
  condition:
    type: "{{CONDITION_TYPE}}"     # 조건 유형 (예: "Periodic", "Event", "Manual")
    schedule: "{{SCHEDULE}}"       # 스케줄 (Periodic 유형에 대한 Cron 표현식, 예: "*/5 * * * *")
    eventSource:
      type: "{{EVENT_TYPE}}"       # 이벤트 유형 (예: "ResourceChange", "MetricThreshold")
      resource: "{{RESOURCE_TYPE}}" # 리소스 유형 (예: "Package", "Model")
      name: "{{RESOURCE_NAME}}"    # 리소스 이름 (예: "example-package")
  action:
    - type: "{{ACTION_TYPE}}"      # 작업 유형 (예: "Notification", "CommandExecution")
      target: "{{TARGET}}"         # 대상 (예: "email", "slack", "webhook")
      recipients: ["{{RECIPIENT}}"] # 수신자 목록 (예: ["admin@example.com"])
      message: "{{MESSAGE}}"       # 메시지 (예: "시나리오 트리거 알림")
  resources:
    packages:
      - name: "{{PACKAGE_NAME}}"   # 패키지 이름 (예: "example-package")
        namespace: "{{NAMESPACE}}" # 네임스페이스 (예: "default")
    models:
      - name: "{{MODEL_NAME}}"     # 모델 이름 (예: "example-model")
        namespace: "{{NAMESPACE}}" # 네임스페이스 (예: "default")
  thresholds:
    cpu: "{{CPU_THRESHOLD}}"       # CPU 임계값 (예: "80%")
    memory: "{{MEMORY_THRESHOLD}}" # 메모리 임계값 (예: "75%")
    responseTime: "{{RESPONSE_TIME_THRESHOLD}}" # 응답 시간 임계값 (예: "500ms")
status:
  phase: "{{PHASE}}"               # 단계 (예: "Active", "Inactive", "Failed", "Completed")
  lastExecutionTime: "{{LAST_EXECUTION_TIME}}" # 마지막 실행 시간 (예: "2023-06-15T10:30:00Z")
  executionCount: {{EXECUTION_COUNT}} # 실행 횟수 (예: 42)
  lastExecutionStatus: "{{LAST_STATUS}}" # 마지막 실행 상태 (예: "Succeeded", "Failed", "Pending")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # 조건 유형 (예: "Ready")
      status: "{{STATUS}}"         # 상태 (예: "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # 마지막 전환 시간 (예: "2023-06-15T10:30:00Z")
      reason: "{{REASON}}"         # 이유 (예: "ScenarioReady")
      message: "{{MESSAGE}}"       # 메시지 (예: "시나리오가 실행 준비됨")
  metrics:
    executionTime: "{{EXECUTION_TIME}}" # 실행 시간 (예: "120ms")
    successRate: "{{SUCCESS_RATE}}" # 성공률 (예: "98.5%")
    triggerCount: {{TRIGGER_COUNT}} # 트리거 횟수 (예: 150)
```
**예시:**
```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: driver-assistance-activate
spec:
  condition:
    vehicleSpeed: ">30"
    ignitionState: "ON"
    driverSeatOccupied: "true"
  action: launch
  target: driver-assistance-package
```

### 4.3 Package 리소스

함께 배포되는 Model들의 그룹을 정의합니다.

**기능:**
- 다수의 Model 참조
- 배포 패턴 정의 (plain, distributed, selector)
- 리소스(볼륨, 네트워크) 연결
- 노드 할당 관리
- 패키지 메타데이터 관리

**템플릿:**
```yaml
apiVersion: piccolo.ai/v1
kind: Package
metadata:
  name: "{{PACKAGE_NAME}}"         # 패키지 리소스 이름 (예: "example-package")
  namespace: "{{NAMESPACE}}"       # 네임스페이스 (예: "default")
  labels:
    app: "{{APP_LABEL}}"           # 애플리케이션 라벨 (예: "adas")
spec:
  description: "{{DESCRIPTION}}"   # 패키지 설명 (예: "ADAS 기능 패키지")
  type: "{{PACKAGE_TYPE}}"         # 패키지 유형 (예: "StandardPackage", "CustomPackage", "SystemPackage")
  version: "{{VERSION}}"           # 버전 (예: "1.0.0")
  models:
    - name: "{{MODEL_NAME}}"       # 모델 이름 (예: "model-a")
      namespace: "{{NAMESPACE}}"   # 네임스페이스 (예: "default")
      role: "{{ROLE}}"             # 역할 (예: "primary", "secondary")
  dependencies:
    - name: "{{DEPENDENCY_NAME}}"  # 종속성 패키지 이름 (예: "dependency-package")
      namespace: "{{NAMESPACE}}"   # 네임스페이스 (예: "default")
      version: "{{VERSION}}"       # 버전 (예: "1.0.0")
  configMaps:
    - name: "{{CONFIG_MAP_NAME}}"  # ConfigMap 이름 (예: "package-config")
      namespace: "{{NAMESPACE}}"   # 네임스페이스 (예: "default")
  secrets:
    - name: "{{SECRET_NAME}}"      # Secret 이름 (예: "package-secrets")
      namespace: "{{NAMESPACE}}"   # 네임스페이스 (예: "default")
status:
  phase: "{{PHASE}}"               # 단계 (예: "Pending", "Running", "Failed", "Terminated")
  startTime: "{{START_TIME}}"      # 시작 시간 (예: "2023-06-15T09:00:00Z")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # 조건 유형 (예: "Available")
      status: "{{STATUS}}"         # 상태 (예: "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # 마지막 전환 시간 (예: "2023-06-15T09:05:00Z")
      reason: "{{REASON}}"         # 이유 (예: "PackageAvailable")
      message: "{{MESSAGE}}"       # 메시지 (예: "패키지가 사용 가능하고 작동 중")
  modelStatus:
    - name: "{{MODEL_NAME}}"       # 모델 이름 (예: "model-a")
      status: "{{STATUS}}"         # 상태 (예: "Running", "Failed")
      startTime: "{{START_TIME}}"  # 시작 시간 (예: "2023-06-15T09:01:00Z")
  metrics:
    healthScore: "{{HEALTH_SCORE}}" # 상태 점수 (예: "95%")
    uptime: "{{UPTIME}}"           # 가동 시간 (예: "99.9%")
    packageType:
      type: "{{PACKAGE_TYPE}}"     # 패키지 유형 (예: "StandardPackage")
      count: {{COUNT}}             # 개수 (예: 1)
      operationalStatus: "{{OPERATIONAL_STATUS}}" # 운영 상태 (예: "Normal", "Degraded")
    modelRelationships:
      total: {{TOTAL_MODELS}}      # 총 모델 수 (예: 2)
      healthy: {{HEALTHY_MODELS}}  # 정상 모델 수 (예: 2)
```
**예시:**
```yaml
apiVersion: piccolo.io/v1
kind: Package
metadata:
  name: driver-assistance-package
  annotations:
    version: "2.1"
    priority: "high"
spec:
  pattern:
    - type: distributed
  models:
    - name: object-detection-model
      node: VisionHPC
      resources:
        volume: vision-data-volume
        network: sensor-network
    - name: lane-keeping-model
      node: ControlHPC
      resources:
        volume: control-data-volume
        network: control-network
```

### 4.4 Model 리소스

개별 컨테이너화된 서비스를 정의합니다.

**기능:**
- 컨테이너 이미지 및 구성 정의
- 리소스 요청 및 제한 설정
- 환경 변수 및 볼륨 마운트 구성
- 보안 컨텍스트 설정
- 라이프사이클 훅 및 프로브 정의
- 시간 결정적 스케줄링 구성 (TIMPANI)
- 특화 관리자 리소스 액세스

**템플릿:**
```yaml
apiVersion: piccolo.ai/v1
kind: Model
metadata:
  name: "{{MODEL_NAME}}"           # 모델 리소스 이름 (예: "example-model")
  namespace: "{{NAMESPACE}}"       # 네임스페이스 (예: "default")
  labels:
    app: "{{APP_LABEL}}"           # 애플리케이션 라벨 (예: "adas")
spec:
  description: "{{DESCRIPTION}}"   # 모델 설명 (예: "객체 감지 모델")
  image: "{{IMAGE}}"               # 이미지 (예: "registry.example.com/models/example-model:v1.0.0")
  resources:
    limits:
      cpu: "{{CPU_LIMIT}}"         # CPU 제한 (예: "2")
      memory: "{{MEMORY_LIMIT}}"   # 메모리 제한 (예: "4Gi")
      gpu: "{{GPU_LIMIT}}"         # GPU 제한 (예: "1")
    requests:
      cpu: "{{CPU_REQUEST}}"       # CPU 요청 (예: "1")
      memory: "{{MEMORY_REQUEST}}" # 메모리 요청 (예: "2Gi")
      gpu: "{{GPU_REQUEST}}"       # GPU 요청 (예: "1")
  ports:
    - name: "{{PORT_NAME}}"        # 포트 이름 (예: "http")
      containerPort: {{CONTAINER_PORT}} # 컨테이너 포트 (예: 8080)
      protocol: "{{PROTOCOL}}"     # 프로토콜 (예: "TCP", "UDP")
  envFrom:
    - configMapRef:
        name: "{{CONFIG_MAP_NAME}}" # ConfigMap 이름 (예: "model-config")
    - secretRef:
        name: "{{SECRET_NAME}}"    # Secret 이름 (예: "model-secrets")
  volumeMounts:
    - name: "{{VOLUME_NAME}}"      # 볼륨 이름 (예: "data-volume")
      mountPath: "{{MOUNT_PATH}}"  # 마운트 경로 (예: "/data")
  volumes:
    - name: "{{VOLUME_NAME}}"      # 볼륨 이름 (예: "data-volume")
      persistentVolumeClaim:
        claimName: "{{PVC_NAME}}"  # PVC 이름 (예: "model-data-pvc")
  asil:
    level: "{{ASIL_LEVEL}}"        # ASIL 레벨 (예: "A", "B", "C", "D")
    safetyMechanism:
      enabled: {{SAFETY_ENABLED}}  # 안전 메커니즘 활성화 (true/false)
      type: "{{SAFETY_TYPE}}"      # 안전 메커니즘 유형 (예: "Redundancy", "Monitoring", "Isolation")
status:
  phase: "{{PHASE}}"               # 단계 (예: "Pending", "Running", "Failed", "Terminated")
  podName: "{{POD_NAME}}"          # Pod 이름 (예: "example-model-pod-xyz123")
  startTime: "{{START_TIME}}"      # 시작 시간 (예: "2023-06-15T08:30:00Z")
  conditions:
    - type: "{{CONDITION_TYPE}}"   # 조건 유형 (예: "Ready")
      status: "{{STATUS}}"         # 상태 (예: "True", "False")
      lastTransitionTime: "{{TRANSITION_TIME}}" # 마지막 전환 시간 (예: "2023-06-15T08:35:00Z")
      reason: "{{REASON}}"         # 이유 (예: "ModelReady")
      message: "{{MESSAGE}}"       # 메시지 (예: "모델 컨테이너가 실행 중이고 준비됨")
  metrics:
    uptime: "{{UPTIME}}"           # 가동 시간 (예: "99.8%")
    restartCount: {{RESTART_COUNT}} # 재시작 횟수 (예: 0)
    cpuUsage: "{{CPU_USAGE}}"      # CPU 사용량 (예: "45%")
    memoryUsage: "{{MEMORY_USAGE}}" # 메모리 사용량 (예: "60%")
    gpuUsage: "{{GPU_USAGE}}"      # GPU 사용량 (예: "30%")
    healthStatus: "{{HEALTH_STATUS}}" # 상태 (예: "Healthy", "Degraded", "Unhealthy")
    asilCompliance:
      status: "{{ASIL_STATUS}}"    # ASIL 준수 상태 (예: "Compliant", "Non-Compliant")
      lastVerificationTime: "{{VERIFICATION_TIME}}" # 마지막 검증 시간 (예: "2023-06-15T12:00:00Z")
```
**예시:**
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
```

### 4.5 Volume 리소스

영구 데이터 저장을 위한 볼륨을 정의합니다.

**기능:**
- 스토리지 크기 및 유형 정의
- 접근 모드 설정
- 스토리지 클래스 지정
- 볼륨 재사용 정책 설정
- 백업 및 복구 구성

**예시:**
```yaml
apiVersion: piccolo.io/v1
kind: Volume
metadata:
  name: vision-data-volume
spec:
  size: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
```

### 4.6 Network 리소스

컨테이너 간 및 외부 통신을 위한 네트워크를 정의합니다.

**기능:**
- 네트워크 유형 및 대역폭 설정
- 보안 정책 및 QoS 구성
- eBPF 기반 고속 경로 구성 (PHAROS)
- 패킷 필터링 규칙 정의
- 네트워크 텔레메트리 설정

**예시:**
```yaml
apiVersion: v1
kind: Network
metadata:
  name: "{{NETWORK_NAME}}"        # 네트워크 리소스 이름 (예: "sensor-network")
  labels:
    app: "{{APP_LABEL}}"          # 애플리케이션 라벨 (예: "adas")    
spec:    
  node_name: "{{NODE_NAME}}"      # 노드 이름 (예: "vm1", "soc2")
  bridge:
    name: "{{BRIDGE_NAME}}"       # 브릿지 이름 (예: "br0")
    type: "{{BRIDGE_TYPE}}"       # 브릿지 타입 (예: "bridge")
  ingress:
    port: "{{INGRESS_PORT}}"      # 인그레스 포트 (예: "5555")
    protocol: "{{PROTOCOL}}"      # 프로토콜 (예: "udp", "tcp", "dds")
    topic: "{{TOPIC_NAME}}"       # 토픽 이름 (DDS 프로토콜 사용 시, 예: "sensor_data")
  egress:
    port: "{{EGRESS_PORT}}"      # 인그레스 포트 (예: "5555")
    protocol: "{{PROTOCOL}}"      # 프로토콜 (예: "udp", "tcp", "dds")
    topic: "{{TOPIC_NAME}}"       # 토픽 이름 (DDS 프로토콜 사용 시, 예: "sensor_data")
  qos:
    latency: "{{LATENCY_LEVEL}}"  # 지연 시간 레벨 (예: "low", "medium", "ultra-low")
    priority: "{{PRIORITY}}"      # 우선순위 (예: "high", "critical", "normal")
  fastPath:
    enabled: {{FAST_PATH}}        # 고속 경로 활성화 (true/false)
    optimizationLevel: "{{OPTIMIZATION_LEVEL}}" # 최적화 수준 (예: "aggressive", "balanced")
  security:
    securityPolicy: "{{SECURITY_POLICY}}" # 보안 정책 (예: "strict", "standard")
    encryption: {{ENCRYPTION}}    # 암호화 활성화 (true/false)
```

## 5. 특화 관리자

PICCOLO는 차량 환경의 특화된 리소스를 관리하기 위한 여섯 가지 관리자를 제공합니다:

### 5.1 특화 관리자 요약 테이블

| 관리자 | 주요 목적 | 관리 대상 |
|--------|----------|-----------|
| DeviceManager | 차량 하드웨어 디바이스의 통합 관리 및 접근 제어 | 센서, 카메라, 액추에이터, 입력 장치 |
| AudioManager | 차량 오디오 리소스의 라우팅, 믹싱, 우선순위 관리 | 스피커, 마이크, 오디오 스트림 |
| WindowManager | 디스플레이 리소스 및 사용자 인터페이스 화면 관리 | 디스플레이, 윈도우, 입력 이벤트 |
| CloudManager | 차량-클라우드 하이브리드 워크로드 관리 및 최적화 | 클라우드 서비스, 데이터 동기화, 네트워크 연결 |
| NetworkManager (Pharos) | 차량 내 네트워크 자원 관리 및 통신 최적화 (별도 프레임워크) | 컨테이너 네트워크, 패킷 필터링, 클라우드 연결 |
| StorageManager | 차량 내 스토리지 자원 관리 및 마운트 최적화 | 컨테이너 볼륨, 호스트 스토리지, 외부 스토리지 |

### 5.1 DeviceManager 상세

#### 5.1.1 기능 개요

DeviceManager는 차량 내 다양한 하드웨어 장치의 접근, 제어, 모니터링을 통합적으로 관리합니다.

**주요 기능:**
- 하드웨어 장치 추상화 및 통합 인터페이스 제공
- 디바이스 액세스 권한 관리 및 충돌 방지
- 장치 상태 모니터링 및 이상 감지
- 전력 관리 및 자원 최적화
- 안전 중요 장치의 정책 기반 관리

#### 5.1.2 DeviceManager 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: DeviceManager
metadata:
  name: ${devicemanager_name}
  namespace: ${namespace}
spec:
  devices:
    - id: ${device_id}
      type: ${device_type}
      description: ${description}
      accessMode: ${access_mode}
      securityLevel: ${security_level}
      powerPolicy: ${power_policy}
      metadata:
        ${key1}: ${value1}
        ${key2}: ${value2}
  accessControl:
    policies:
      - name: ${policy_name}
        deviceSelector:
          ${selector_key}: ${selector_value}
        access: ${policy_access}
        conditions:
          ${condition_key}: ${condition_value}
```

### 5.2 AudioManager 상세

#### 5.2.1 기능 개요

AudioManager는 차량 내 오디오 리소스의 라우팅, 믹싱, 우선순위 관리를 담당합니다.

**주요 기능:**
- 오디오 스트림 및 장치 관리
- 오디오 라우팅 및 믹싱 제어
- 우선순위 기반 오디오 정책 시행
- 볼륨 및 음소거 관리
- 다양한 오디오 시나리오 지원 (미디어, 내비게이션, 통화, 알림)

#### 5.2.2 AudioManager 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: AudioManager
metadata:
  name: ${audiomanager_name}
  namespace: ${namespace}
spec:
  devices:
    - id: ${audio_device_id}
      type: ${audio_device_type}
      channels: ${channels}
      description: ${description}
  streams:
    - name: ${stream_name}
      source: ${source}
      sink: ${sink}
      format: ${format}
      sampleRate: ${sample_rate}
      channels: ${stream_channels}
      priority: ${stream_priority}
      policy: ${stream_policy}
  routing:
    rules:
      - condition:
          ${rule_condition_key}: ${rule_condition_value}
        action: ${routing_action}
        target: ${routing_target}
  volumeControl:
    defaultLevel: ${default_volume_level}
    maxLevel: ${max_volume_level}
    perStream: ${per_stream_volume}
  policy:
    scenarioPolicies:
      - scenario: ${scenario_name}
        streamPriorities:
          ${stream_name}: ${priority}
        muteRules:
          ${stream_name}: ${mute_policy}
```

### 5.3 WindowManager 상세

#### 5.3.1 기능 개요

WindowManager는 차량 디스플레이 리소스와 윈도우 레이아웃을 관리합니다.

**주요 기능:**
- 디스플레이 장치 관리 및 구성
- 윈도우 레이아웃 및 z-순서 관리
- 입력 장치 라우팅 관리
- 차량 상태에 따른 레이아웃 정책 적용
- 윈도우 접근 권한 관리

#### 5.3.2 WindowManager 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: WindowManager
metadata:
  name: ${windowmanager_name}
  namespace: ${namespace}
spec:
  displays:
    - id: ${display_id}
      resolution: ${resolution}
      orientation: ${orientation}
      brightnessControl: ${brightness_control}
      colorCalibration:
        enabled: ${color_cal_enabled}
        profile: ${color_profile}
  windows:
    - name: ${window_name}
      display: ${display_id}
      position:
        x: ${x}
        y: ${y}
      size:
        width: ${width}
        height: ${height}
      zOrder: ${z_order}
      visible: ${visible}
      focusable: ${focusable}
      input:
        touch: ${touch_input}
        gesture: ${gesture_input}
        keyboard: ${keyboard_input}
      permissions:
        apps:
          - ${app_name}
        access: ${window_access}
  policies:
    - name: ${policy_name}
      condition:
        ${policy_condition_key}: ${policy_condition_value}
      layout:
        ${window_name}:
          visible: ${policy_visible}
          focusable: ${policy_focusable}
  inputRouting:
    default: ${default_window}
    rules:
      - source: ${input_source}
        target: ${input_target}
        condition:
          ${input_condition_key}: ${input_condition_value}
  accessControl:
    rules:
      - app: ${policy_app_name}
        allow:
          - ${allow_window}
        deny:
          - ${deny_window}
```

### 5.4 NetworkManager

네트워크 관리자(NetworkManager) 기능은 특화된 기능으로 Pharos 프레임워크를 통해 별도로 구성됩니다. Pharos는 차량 내 네트워크 자원 관리 및 통신 최적화를 위한 전문화된 솔루션으로, PICCOLO와 통합되어 사용됩니다.

자세한 내용은 Pharos 관련 문서를 참조하십시오.


### 5.5 CloudManager 상세

#### 5.5.1 기능 개요

CloudManager는 PICCOLO 프레임워크에서 차량과 클라우드 간의 하이브리드 워크로드를 관리하고 최적화합니다. 이를 통해 자원 집약적인 작업을 클라우드로 오프로드하거나, 차량과 클라우드 간의 원활한 전환을 지원합니다.

**주요 기능:**
- 하이브리드 워크로드 관리
- 워크로드 분할 및 분산 실행
- 자동 워크로드 오프로딩
- 실시간/비실시간 작업 구분
- 우선순위 기반 실행 위치 결정

#### 5.5.2 CloudManager 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: CloudManager
metadata:
  name: ${cloudmanager_name}
  namespace: ${namespace}
spec:
  cloudServices:
    - name: ${service_name}
      provider: ${provider}
      region: ${region}
      credentials:
        secretName: ${secret_name}
        keyPath: ${key_path}
      endpoints:
        - name: ${endpoint_name}
          url: ${endpoint_url}
          protocol: ${endpoint_protocol}
  workloads:
    - name: ${workload_name}
      type: ${workload_type}
      placement: ${placement}
      requirements:
        cpu: ${cpu}
        memory: ${memory}
        gpu: ${gpu}
        latency: ${latency}
      fallback:
        strategy: ${fallback_strategy}
        timeout: ${fallback_timeout}
  dataSync:
    - name: ${sync_name}
      source: ${sync_source}
      destination: ${sync_destination}
      priority: ${sync_priority}
      schedule: ${sync_schedule}
      strategy: ${sync_strategy}
      conflict: ${sync_conflict}
  connections:
    primary: ${primary_conn}
    fallback:
      - ${fallback_conn}
    monitoring:
      interval: ${monitor_interval}
      threshold:
        latency: ${latency_threshold}
        packetLoss: ${packet_loss_threshold}
        bandwidth: ${bandwidth_threshold}
  offlineMode:
    caching:
      enabled: ${offline_cache_enabled}
      maxSize: ${offline_cache_max_size}
      ttl: ${offline_cache_ttl}
    localServices:
      - service: ${offline_service_name}
        mode: ${offline_service_mode}
  permissions:
    - modelSelector:
        matchLabels:
          ${model_label_key}: ${model_label_value}
      services:
        - ${permission_service}
      accessLevel: ${permission_level}
```

### 5.6 StorageManager 상세

#### 5.6.1 기능 개요

StorageManager는 PICCOLO 프레임워크 내에서 컨테이너 스토리지 및 볼륨 관리 기능을 담당합니다.

**주요 기능:**
- 다양한 스토리지 모드 지원 (virtual, host, shared)
- 컨테이너 볼륨 생명주기 관리
- 볼륨 마운트 및 권한 제어
- 스토리지 할당량 및 사용량 모니터링
- 고성능 데이터 접근을 위한 최적화
- 스토리지 리소스 보호 및 격리

#### 5.6.2 StorageManager 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: StorageManager
metadata:
  name: ${storagemanager_name}
  namespace: ${namespace}
spec:
  defaultStorageMode: ${storage_mode}
  volumes:
    virtual:
      - name: ${virtual_volume_name}
        size: ${volume_size}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        emptyDir:
          medium: ${medium}
          sizeLimit: ${size_limit}
        permissions:
          mode: ${permission_mode}
          user: ${user_id}
          group: ${group_id}
    host:
      - name: ${host_volume_name}
        hostPath: ${host_path}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        permissions:
          mode: ${permission_mode}
          user: ${user_id}
          group: ${group_id}
        pathType: ${path_type}
        readOnly: ${read_only}
    shared:
      - name: ${shared_volume_name}
        mountPath: ${mount_path}
        accessMode: ${access_mode}
        protocol: ${protocol}
        server: ${server_ip}
        path: ${server_path}
        options: ${mount_options}
        credentials:
          secretName: ${secret_name}
        readOnly: ${read_only}
  volumeClaims:
    - name: ${claim_name}
      podSelector:
        matchLabels:
          ${pod_label_key}: ${pod_label_value}
      volumes:
        - name: ${volume_name}
          mountPath: ${mount_path}
          readOnly: ${read_only}
  quotas:
    - name: ${quota_name}
      podSelector:
        matchLabels:
          ${pod_label_key}: ${pod_label_value}
      limits:
        storage: ${storage_limit}
        inodes: ${inode_limit}
      actions:
        onLimitReached: ${limit_action}
  snapshots:
    enabled: ${snapshots_enabled}
    schedule:
      - name: ${schedule_name}
        volumeName: ${volume_name}
        cronExpression: ${cron_expr}
        retention:
          count: ${snapshot_count}
          duration: ${retention_period}
  dataProtection:
    encryption:
      enabled: ${encryption_enabled}
      volumes:
        - ${encrypted_volume}
      keyManagement:
        keySource: ${key_source}
        keyRotation: ${key_rotation}
    backup:
      enabled: ${backup_enabled}
      volumes:
        - ${backup_volume}
      schedule: ${backup_schedule}
      destination: ${backup_destination}
  performance:
    ioProfile: ${io_profile}
    caching:
      enabled: ${caching_enabled}
      volumes:
        - ${cached_volume}
      size: ${cache_size}
      policy: ${cache_policy}
  monitoring:
    metrics:
      enabled: ${metrics_enabled}
      interval: ${metrics_interval}
      exporters:
        - ${metrics_exporter}
    alerts:
      - name: ${alert_name}
        condition: ${alert_condition}
        severity: ${alert_severity}
        action: ${alert_action}
    logging:
      level: ${log_level}
```

## 6. 운영 시나리오

PICCOLO는 다양한 차량 운영 시나리오를 지원합니다:

### 6.1 운전 보조 시스템 활성화

차량이 특정 속도 이상으로 주행하고 운전자가 탑승했을 때 자동으로 운전 보조 시스템을 활성화합니다.

**활성화 조건:**
- 차량 속도 > 30km/h
- 점화 상태 = ON
- 운전석 탑승 = true

**활성화되는 서비스:**
- 객체 감지 (카메라, 레이더, 라이다 처리)
- 차선 유지 보조
- 적응형 크루즈 컨트롤

### 6.2 주차 보조 시스템 활성화

차량이 저속으로 후진하고 주차 보조 버튼이 활성화되었을 때 주차 보조 시스템을 시작합니다.

**활성화 조건:**
- 차량 속도 < 10km/h
- 기어 위치 = R (후진)
- 주차 보조 버튼 = ON

**활성화되는 서비스:**
- 주변 시야 카메라 시스템
- 주차 안내 시스템
- 장애물 감지 및 경고

### 6.3 엔터테인먼트 시스템 업데이트

차량이 주차되고 충분한 배터리 잔량이 있으며 Wi-Fi에 연결되었을 때 엔터테인먼트 시스템을 업데이트합니다.

**활성화 조건:**
- 점화 상태 = OFF
- 배터리 수준 > 50%
- Wi-Fi 연결 = true

**업데이트되는 서비스:**
- 미디어 플레이어
- 내비게이션 시스템
- 인포테인먼트 UI

### 6.4 차량 진단 시스템 실행

진단 모드가 활성화되고 서비스 요청이 시작되었을 때 차량 진단 시스템을 실행합니다.

**활성화 조건:**
- 진단 모드 = true
- 서비스 요청 = initiated

**활성화되는 서비스:**
- 시스템 진단 도구
- 오류 코드 분석기
- 서비스 보고서 생성기

### 6.5 유휴 리소스 정리

차량이 장시간 꺼져 있고 배터리 잔량이 부족할 때 비필수 서비스를 정리합니다.

**활성화 조건:**
- 점화 상태 = OFF
- 유휴 시간 > 30분
- 배터리 수준 < 40%

**삭제되는 서비스:**
- 비필수 백그라운드 서비스
- 유휴 상태의 애플리케이션
- 임시 캐시 및 로그

### 6.6 특화 관리자 통합 시나리오

#### 6.6.1 주행 중 내비게이션 시나리오

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: navigation-active-scenario
spec:
  condition:
    vehicleSpeed: ">5"
    ignitionState: "ON"
  action: launch
  target: navigation-package
```

이 시나리오에서:
1. DeviceManager는 GPS 센서 및 속도계 접근을 제공
2. AudioManager는 내비게이션 음성 안내를 주요 스트림으로 설정하고 미디어 볼륨을 줄임
3. WindowManager는 내비게이션 윈도우에 더 큰 화면 영역을 할당

#### 6.6.2 주차 보조 시나리오

```yaml
apiVersion: piccolo.io/v1
kind: Scenario
metadata:
  name: parking-assist-scenario
spec:
  condition:
    gearPosition: "R"
    vehicleSpeed: "<10"
  action: launch
  target: parking-assist-package
```

이 시나리오에서:
1. DeviceManager는 후방 카메라, 초음파 센서에 대한 접근을 제공
2. AudioManager는 주차 센서 경고음을 우선시하고 다른 오디오를 낮춤
3. WindowManager는 주차 모드 레이아웃으로 전환하여 카메라 화면을 전체 화면으로 표시

## 7. 사용자 인터페이스

### 7.1 Settings UI 패널

시스템 구성을 관리하기 위한 그래픽 사용자 인터페이스:

- **네비게이션 트리**
  - 계층적 리소스 구조 표시
  - 리소스 유형별 그룹화
  - 빠른 탐색 및 검색

- **속성 편집기**
  - 구조화된 양식 기반 편집
  - 인라인 YAML 편집
  - 실시간 검증 피드백

- **템플릿 라이브러리**
  - 사전 정의된 리소스 템플릿
  - 커스텀 템플릿 생성 및 공유
  - 템플릿 기반 빠른 리소스 생성

- **버전 관리**
  - 변경 이력 조회
  - 이전 버전과의 비교
  - 특정 버전으로 롤백

- **대시보드**
  - 시스템 상태 개요
  - 주요 지표 시각화
  - 알림 및 이벤트 표시

- **특화 관리자 패널**
  - 디바이스 관리 패널
  - 오디오 관리 패널
  - 윈도우 관리 패널

### 7.2 Settings CLI

명령줄을 통한 시스템 관리:

- **리소스 관리**
  - 리소스 생성, 조회, 수정, 삭제
  - 리소스 상태 확인
  - 리소스 가져오기 및 내보내기

- **시스템 관리**
  - 시스템 상태 조회
  - 구성 적용 및 롤백
  - 진단 및 문제 해결

- **배치 작업**
  - 스크립트 지원
  - 자동화 워크플로우
  - 대량 작업 처리

- **포맷 지원**
  - JSON/YAML 형식 지원
  - 사용자 정의 출력 형식
  - 파이프라인 통합

- **원격 관리**
  - SSH 기반 원격 관리
  - API 토큰 인증
  - 보안 셸 세션

- **특화 관리자 명령어**
  - 디바이스 관리 명령어
  - 오디오 관리 명령어
  - 윈도우 관리 명령어

### 7.3 API 인터페이스

프로그래밍 방식으로 시스템 관리:

- **RESTful API**
  - 리소스 CRUD 작업
  - 상태 쿼리 및 모니터링
  - 이벤트 스트림 구독

- **WebSocket API**
  - 실시간 이벤트 스트리밍
  - 상태 변경 알림
  - 양방향 통신

- **gRPC API**
  - 고성능 서비스 간 통신
  - 스트리밍 지원
  - 강력한 타입 검사

## 8. 보안 및 안전성

### 8.1 인증 및 권한 부여

- **사용자 인증**
  - 토큰 기반 인증
  - 인증서 기반 인증
  - 외부 인증 시스템 통합

- **권한 부여**
  - 역할 기반 접근 제어 (RBAC)
  - 리소스 수준 권한
  - 세분화된 작업 권한

- **감사**
  - 모든 관리 작업 로깅
  - 변경 이력 추적
  - 감사 보고서 생성

### 8.2 네트워크 보안

- **통신 암호화**
  - TLS 기반 암호화
  - 서비스 간 상호 인증
  - 인증서 관리

- **네트워크 분리**
  - 보안 영역 분리
  - 마이크로세그먼테이션
  - 트래픽 필터링

- **침입 탐지**
  - 비정상 트래픽 감지
  - 공격 패턴 인식
  - 보안 이벤트 알림

### 8.3 컨테이너 보안

- **이미지 보안**
  - 이미지 서명 및 검증
  - 취약점 스캔
  - 승인된 이미지만 실행

- **런타임 보안**
  - 권한 최소화
  - 읽기 전용 파일 시스템
  - 시스템 콜 제한

- **보안 컨텍스트**
  - 비루트 사용자 실행
  - 기능 제한
  - 시큐어 컴퓨팅 모드

### 8.4 기능 안전

- **안전 정책**
  - ISO 26262 준수
  - 안전 무결성 수준 (ASIL) 지원
  - 안전 사례 관리

- **결함 감지 및 처리**
  - 장애 감지 메커니즘
  - 안전한 상태로의 전환
  - 중복성 및 다양성

- **시간 결정성**
  - 결정적 실행 보장
  - 타이밍 위반 감지
  - 실시간 성능 모니터링

## 9. 통합 및 확장성

### 9.1 외부 시스템 통합

- **차량 시스템 통합**
  - 차량 버스 통합 (CAN, FlexRay, Automotive Ethernet)
  - 차량 진단 인터페이스
  - 센서 시스템 통합

- **클라우드 서비스 통합**
  - 데이터 동기화
  - 원격 관리 및 모니터링
  - OTA 업데이트 지원

- **개발 도구 통합**
  - CI/CD 파이프라인 통합
  - 테스트 자동화
  - 코드 리포지토리 연동

### 9.2 확장 메커니즘

- **플러그인 아키텍처**
  - 사용자 정의 컴포넌트 추가
  - 기존 기능 확장
  - 서드파티 통합

- **커스텀 리소스 정의**
  - 사용자 정의 리소스 유형 추가
  - 도메인별 리소스 모델 확장
  - 리소스 검증 규칙 사용자화

- **이벤트 훅**
  - 시스템 이벤트에 반응하는 훅
  - 사용자 정의 작업 실행
  - 자동화 워크플로우 통합

## 10. 성능 및 확장성

### 10.1 성능 특성

- **응답 시간**
  - API 요청 응답: <100ms
  - 컨테이너 시작: <3초
  - 안전 중요 기능: <10ms

- **처리량**
  - 동시 API 요청: 1000 TPS
  - 컨테이너 관리 작업: 100/분
  - 이벤트 처리: 10,000/초

- **리소스 사용**
  - 최소 메모리 공간: 512MB
  - 최소 CPU 코어: 2
  - 스토리지 요구사항: 10GB

### 10.2 확장성 특성

- **노드 확장성**
  - 지원되는 최대 노드 수: 10
  - 노드당 최대 컨테이너: 100
  - 노드 추가/제거 동적 지원

- **서비스 확장성**
  - 최대 모델 수: 500
  - 최대 시나리오 수: 200
  - 최대 패키지 수: 100

- **저장소 확장성**
  - 최대 ETCD 데이터 크기: 8GB
  - 최대 구성 항목 수: 100,000
  - 최대 이벤트 이력: 1,000,000

## 11. 구현 제약사항

### 11.1 하드웨어 제약

- **최소 하드웨어 요구사항**
  - CPU: 멀티코어 프로세서, 최소 4코어
  - 메모리: 8GB RAM
  - 스토리지: 64GB
  - 네트워크: 1Gbps 이더넷

- **권장 하드웨어 구성**
  - CPU: 8코어, 실시간 처리 기능 지원
  - 메모리: 16GB RAM, ECC 메모리
  - 스토리지: 128GB SSD
  - 네트워크: 10Gbps 이더넷

### 11.2 소프트웨어 제약

- **운영체제 요구사항**
  - Linux 커널 5.10 이상
  - PREEMPT_RT 패치 적용 권장
  - eBPF/XDP 지원

- **런타임 요구사항**
  - Podman 3.0 이상
  - ETCD 3.4 이상
  - Go 1.18 이상
  - Python 3.9 이상

- **라이브러리 의존성**
  - eBPF 라이브러리
  - 실시간 확장 라이브러리
  - 차량 통신 프로토콜 라이브러리
  - 그래픽 하위 시스템 라이브러리

- **디바이스 드라이버 요구사항**
  - 표준 리눅스 디바이스 드라이버
  - 특정 차량 센서 드라이버
  - 그래픽 가속 드라이버
  - 오디오 서브시스템 드라이버

- **컨테이너 호환성**
  - OCI 호환 컨테이너 이미지
  - rootless 컨테이너 지원
  - cgroup v2 지원
  - seccomp 프로파일 지원

### 11.3 성능 제약

- **타이밍 제약**
  - 실시간 작업 지터: <1ms
  - 네트워크 지연시간: <10ms
  - 부팅 시간: <30초

- **리소스 제약**
  - 컨테이너 오버헤드: <10%
  - 네트워크 오버헤드: <5%
  - 스토리지 I/O: 최소 50MB/s

- **처리량 제약**
  - 최대 동시 컨테이너 배포: 20
  - 최대 이벤트 처리량: 10,000/초
  - 최대 로그 처리량: 5MB/초

### 11.4 환경 제약

- **작동 환경**
  - 온도: -20°C ~ 70°C
  - 전력 소비: <25W
  - 진동 및 충격 내성

- **네트워크 환경**
  - 간헐적 연결 지원
  - 제한된 대역폭 처리
  - 높은 지연 시간 처리
