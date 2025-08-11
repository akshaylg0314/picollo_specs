# PICCOLO (PULLPIRI) 개발 계획 문서

**문서 정보:**
- 문서 ID: PICCOLO-PLAN-INDEX-2025
- 버전: 1.0
- 날짜: 2025-07-28
- 분류: 개발 계획 인덱스

## 개요

이 문서는 PICCOLO(PULLPIRI) 프로젝트의 개발 계획 관련 문서들을 정리한 인덱스입니다. GitHub 저장소([eclipse-pullpiri/pullpiri](https://github.com/eclipse-pullpiri/pullpiri/tree/refactoring))의 초기 구현 상태를 기반으로 작성된 세부 개발 계획, 컴포넌트 구현 계획 및 기술 연구 항목을 포함합니다.

## 개발 계획 문서 목록

### 1. 개발 로드맵
- **파일**: [development_roadmap.md](development_roadmap.md)
- **설명**: 4단계 개발 계획, 단계별 주요 작업 항목 및 일정, 마일스톤 정의
- **주요 내용**:
  - Phase 1: 핵심 컴포넌트 완성 (2025-08 ~ 2025-09)
  - Phase 2: 확장 기능 구현 (2025-09 ~ 2025-10)
  - Phase 3: 특화 기능 및 성능 최적화 (2025-10 ~ 2025-11)
  - Phase 4: 시스템 검증 및 안정화 (2025-11 ~ 2025-12)

### 2. 컴포넌트 구현 계획
- **파일**: [component_implementation.md](component_implementation.md)
- **설명**: 각 컴포넌트별 상세 구현 계획, 현재 상태 분석, 우선순위 설정
- **주요 내용**:
  - 핵심 컴포넌트 구현 계획 (APIServer, ActionController, StateManager, FilterGateway)
  - 확장 컴포넌트 구현 계획 (PolicyManager, MonitoringServer, NodeAgent)
  - 특화 컴포넌트 구현 계획 (TIMPANI Scheduler, PHAROS Networking, SettingsService)
  - 공통 라이브러리 및 유틸리티 개발 계획

### 3. 기술 과제 및 연구 항목
- **파일**: [technical_challenges.md](technical_challenges.md)
- **설명**: 프로젝트 성공을 위한 핵심 기술 과제와 연구 항목 정의
- **주요 내용**:
  - 차량 환경 최적화 컨테이너 기술 연구
  - 실시간 스케줄링 및 리소스 관리 연구
  - 네트워크 오케스트레이션 및 eBPF 연구
  - 분산 상태 관리 및 장애 복구 연구
  - 차량 특화 서비스 통합 연구
  - 클라우드 연동 및 OTA 연구
  - 개발 및 테스트 환경 구축

## 구현 우선순위

프로젝트 개발을 위한 구현 우선순위는 다음과 같이 설정되었습니다:

### 높은 우선순위
1. 핵심 컴포넌트 기본 기능 구현 (APIServer, ActionController, StateManager)
2. 공통 ETCD 라이브러리 최적화 및 확장
3. FilterGateway 기본 시나리오 처리 구현
4. 기본 컨테이너 라이프사이클 관리 기능

### 중간 우선순위
1. PolicyManager 및 MonitoringServer 구현
2. 고급 상태 관리 및 오류 복구 기능
3. NodeAgent 및 분산 모니터링 시스템
4. SettingsService 및 관리 인터페이스

### 낮은 우선순위
1. TIMPANI Scheduler 및 실시간 성능 최적화
2. PHAROS Networking 및 eBPF 기반 네트워킹
3. 차량 특화 관리자 (DeviceManager, AudioManager 등)
4. 고급 클라우드 연동 및 OTA 기능

## 개발 일정 요약

| 기간 | 주요 작업 |
|------|----------|
| **2025-08** | 핵심 컴포넌트 구현 착수, 공통 라이브러리 최적화 |
| **2025-09** | 핵심 컴포넌트 완성, 확장 기능 구현 시작 |
| **2025-10** | 확장 기능 완성, 특화 기능 구현 시작 |
| **2025-11** | 특화 기능 구현, 성능 최적화, 테스트 확대 |
| **2025-12** | 시스템 안정화, 문서화 완성, 최종 릴리스 준비 |

## 결론

이 개발 계획은 PICCOLO(PULLPIRI) 프로젝트의 성공적인 구현을 위한 로드맵을 제공합니다. 각 단계별 세부 계획과 기술 연구 항목을 통해 체계적인 개발을 진행하고, 정기적인 검토 및 업데이트를 통해 계획의 실효성을 유지해야 합니다.

프로젝트 진행 과정에서 발생하는 변화에 따라 이 계획은 지속적으로 수정 및 보완되어야 하며, 개발 팀은 정기적인 회의를 통해 진행 상황을 점검하고 필요한 조정을 해야 합니다.
