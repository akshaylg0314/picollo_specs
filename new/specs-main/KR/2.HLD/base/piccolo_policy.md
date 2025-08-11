# PICCOLO PolicyManager 상세 설계 문서

**문서 번호**: PICCOLO-POLICY-2025-001  
**버전**: 1.1  
**날짜**: 2025-07-16  
**작성자**: joshua-jung_LGESDV  
**분류**: 설계 명세서

## 1. 개요

본 문서는 PICCOLO 프레임워크의 PolicyManager에 대한 상세 설계를 기술합니다. PolicyManager는 차량 상태, 안전 등급, 리소스 사용, 자율주행 레벨에 따라 서비스의 우선순위를 결정하고 이를 기반으로 서비스의 실행 허용(allow), 대기(pending), 거부(deny) 여부를 판단합니다.

## 2. 설계 목표

PolicyManager는 다음과 같은 목표를 가지고 설계되었습니다:

1. 우선순위가 높은 서비스가 우선적으로 허용되도록 함
2. 차량 상태에 따라 서비스의 우선순위를 동적으로 조정
3. 안전 등급(ASIL)에 따라 서비스의 우선순위를 차등화
4. 리소스 충돌 발생 시 상황에 맞는 deny/pending 결정
5. 자율주행 레벨에 따른 서비스 우선순위 조정

## 3. 우선순위 결정 요소

### 3.1 기본 서비스 우선순위

서비스 타입별 기본 우선순위를 1-100 범위로 정의합니다(높을수록 우선순위 높음).

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

### 3.2 차량 상태별 우선순위 조정

차량 상태에 따른 서비스 유형별 우선순위를 조정하는 값을 정의합니다.

#### 3.2.1 전원 꺼짐 상태 (poweroff)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | -50 |
| adas | -90 |
| hud | -90 |
| navigation | -90 |
| cluster | -90 |
| alert | -50 |
| voice-control | -90 |
| diagnostics | +10 |
| media | -90 |
| comfort | -90 |
| infotainment | -90 |
| connectivity | -50 |
| background | +10 |
| update | +20 |

#### 3.2.2 시동 상태 (ignition)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +10 |
| adas | +5 |
| hud | +5 |
| navigation | 0 |
| cluster | +10 |
| alert | +10 |
| voice-control | -10 |
| diagnostics | +15 |
| media | -20 |
| comfort | 0 |
| infotainment | -20 |
| connectivity | +5 |
| background | +5 |
| update | -10 |

#### 3.2.3 주차 상태 (parking)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | 0 |
| adas | -10 |
| hud | -20 |
| navigation | +5 |
| cluster | -5 |
| alert | +5 |
| voice-control | +10 |
| diagnostics | +15 |
| media | +15 |
| comfort | +10 |
| infotainment | +15 |
| connectivity | +10 |
| background | +5 |
| update | +20 |

#### 3.2.4 주행 상태 (driving)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +15 |
| navigation | +15 |
| cluster | +10 |
| alert | +10 |
| voice-control | +5 |
| diagnostics | -20 |
| media | -5 |
| comfort | 0 |
| infotainment | -10 |
| connectivity | -5 |
| background | -20 |
| update | -80 |

#### 3.2.5 중립 상태 (neutral)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +5 |
| adas | +10 |
| hud | +10 |
| navigation | +10 |
| cluster | +5 |
| alert | +5 |
| voice-control | +5 |
| diagnostics | 0 |
| media | +5 |
| comfort | +5 |
| infotainment | +5 |
| connectivity | +5 |
| background | 0 |
| update | -10 |

#### 3.2.6 후진 상태 (reverse)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +5 |
| navigation | -5 |
| cluster | +5 |
| alert | +10 |
| voice-control | -5 |
| diagnostics | -10 |
| media | -15 |
| comfort | -5 |
| infotainment | -15 |
| connectivity | -10 |
| background | -15 |
| update | -80 |

### 3.3 ASIL 안전 등급별 우선순위 조정

| ASIL 등급 | 우선순위 조정 | 설명 |
|----------|--------------|------|
| QM | 0 | 품질 관리 (Quality Managed) |
| A | +5 | ASIL A |
| B | +10 | ASIL B |
| C | +15 | ASIL C |
| D | +20 | ASIL D (최고 안전 등급) |

### 3.4 디바이스 충돌 정책

리소스 충돌 시 서비스의 deny/pending 여부를 결정하는 정책입니다.

| 디바이스 | 충돌 시 정책 | 타임아웃(초) | 재시도 횟수 | 추가 정보 |
|---------|------------|------------|-----------|---------|
| audio | pending | 30 | 3 | |
| gpu | pending | 60 | 5 | |
| speaker | pending | 15 | 2 | |
| mic | deny | - | - | 개인정보 민감 리소스 |
| display | pending | 45 | 3 | |
| npu | pending | 120 | 10 | |

### 3.5 자율주행 레벨별 우선순위 조정

#### 3.5.1 레벨 1 (운전자 지원)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +5 |
| adas | +10 |
| hud | +10 |
| alert | +10 |
| navigation | +5 |
| cluster | +5 |
| media | 0 |
| infotainment | -5 |
| update | -15 |

#### 3.5.2 레벨 2 (부분 자동화)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +10 |
| adas | +15 |
| hud | +15 |
| alert | +15 |
| navigation | +10 |
| cluster | +10 |
| voice-control | +5 |
| media | -10 |
| infotainment | -15 |
| update | -25 |

#### 3.5.3 레벨 3 (조건부 자동화)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +15 |
| adas | +20 |
| hud | +10 |
| alert | +15 |
| navigation | +5 |
| cluster | +10 |
| voice-control | +10 |
| media | +5 |
| infotainment | 0 |
| update | -30 |

#### 3.5.4 레벨 4 (고도 자동화)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +20 |
| adas | +25 |
| hud | +5 |
| alert | +15 |
| navigation | +15 |
| voice-control | +15 |
| media | +10 |
| infotainment | +5 |
| comfort | +10 |
| update | -10 |

#### 3.5.5 레벨 5 (완전 자동화)

| 서비스 유형 | 우선순위 조정 |
|------------|--------------|
| safety-critical | +25 |
| adas | +25 |
| navigation | +20 |
| alert | +15 |
| infotainment | +15 |
| media | +15 |
| comfort | +15 |
| voice-control | +15 |
| connectivity | +10 |
| update | 0 |

## 4. 결정 로직

### 4.1 최종 우선순위 계산

최종 우선순위는 다음 요소의 합으로 계산됩니다:
```
최종 우선순위 = 기본 우선순위 + 차량 상태 조정 + ASIL 등급 조정 + 자율주행 레벨 조정
```

### 4.2 결정 임계값

| 결정 | 임계값 |
|------|--------|
| 허용 (allow) | ≥ 70 |
| 대기 (pending) | ≥ 40, < 70 |
| 거부 (deny) | < 40 |

### 4.3 특별 규칙

일부 상황에서는 계산된 우선순위와 관계없이 특별 규칙이 적용됩니다:

1. 안전 중요 서비스(safety-critical)는 주행 중(driving) 항상 허용됨
2. 소프트웨어 업데이트(update)는 주행 중(driving) 항상 거부됨
3. 자율주행 레벨 4-5에서는 운전자 주의 관련 알림의 우선순위가 감소함

## 5. 서비스 유형별 정책 예시

### 5.1 안전 중요 서비스

| 서비스 이름 패턴 | 유형 | ASIL 등급 | 필요 디바이스 |
|-----------------|------|----------|--------------|
| brake-control-* | safety-critical | D | npu |
| steering-control-* | safety-critical | D | npu |

### 5.2 ADAS 서비스

| 서비스 이름 패턴 | 유형 | ASIL 등급 | 필요 디바이스 |
|-----------------|------|----------|--------------|
| lane-assist-* | adas | B | gpu, npu |
| adaptive-cruise-* | adas | C | npu |
| collision-warning-* | adas | B | gpu, npu |

### 5.3 인포테인먼트 서비스

| 서비스 이름 패턴 | 유형 | ASIL 등급 | 필요 디바이스 |
|-----------------|------|----------|--------------|
| navigation-* | navigation | QM | display, gpu, speaker |
| media-* | media | QM | audio, display, gpu |
| infotainment-* | infotainment | QM | display, gpu, audio |

### 5.4 기타 서비스

| 서비스 이름 패턴 | 유형 | ASIL 등급 | 필요 디바이스 |
|-----------------|------|----------|--------------|
| voice-* | voice-control | QM | mic, speaker, npu |
| diagnostic-* | diagnostics | QM | npu |
| update-* | update | QM | - |

## 6. 종합 서비스 우선순위 계산 예시

### 6.1 차량 상태별 서비스 우선순위 테이블

다음은 자율주행 레벨 3, ASIL B 안전 등급에서의 다양한 차량 상태별 최종 우선순위 계산 예시입니다:

| 서비스 유형 | 기본 | Poweroff | Ignition | Parking | Driving | Neutral | Reverse |
|------------|------|----------|----------|---------|---------|---------|---------|
| safety-critical | 90 | 65 | 125 | 115 | 125 | 120 | 125 |
| adas | 85 | 25 | 120 | 105 | 130 | 125 | 130 |
| hud | 80 | 10 | 105 | 80 | 115 | 110 | 105 |
| navigation | 75 | 0 | 90 | 95 | 105 | 100 | 85 |
| cluster | 70 | 0 | 100 | 85 | 100 | 95 | 95 |
| alert | 65 | 35 | 100 | 95 | 100 | 95 | 100 |
| voice-control | 60 | 0 | 70 | 90 | 85 | 85 | 75 |
| diagnostics | 55 | 75 | 80 | 80 | 45 | 65 | 55 |
| media | 50 | 0 | 45 | 80 | 60 | 70 | 50 |
| comfort | 45 | 0 | 65 | 75 | 65 | 70 | 60 |
| infotainment | 40 | 0 | 40 | 75 | 50 | 65 | 45 |
| connectivity | 35 | 0 | 60 | 65 | 50 | 60 | 45 |
| background | 30 | 50 | 45 | 45 | 20 | 40 | 25 |
| update | 25 | 55 | 25 | 55 | 0 | 25 | 0 |

* 주: 표의 값은 기본 우선순위 + 차량 상태 조정 + ASIL B 등급 조정(+10) + 자율주행 레벨 3 조정으로 계산되었으며, 최대값은 100으로 제한됨

### 6.2 자율주행 레벨별 주요 서비스 우선순위 비교

다음은 주행(Driving) 상태에서 ASIL B 안전 등급에 대한 자율주행 레벨별 주요 서비스 우선순위 비교입니다:

| 서비스 유형 | 기본 | 레벨 1 | 레벨 2 | 레벨 3 | 레벨 4 | 레벨 5 |
|------------|------|-------|-------|-------|-------|-------|
| safety-critical | 90 | 115 | 120 | 125 | 130 | 135 |
| adas | 85 | 120 | 125 | 130 | 135 | 135 |
| hud | 80 | 115 | 120 | 115 | 110 | 105 |
| navigation | 75 | 105 | 110 | 105 | 115 | 120 |
| alert | 65 | 95 | 100 | 100 | 100 | 100 |
| media | 50 | 55 | 45 | 60 | 65 | 70 |
| infotainment | 40 | 35 | 25 | 40 | 45 | 55 |
| update | 25 | 0 | 0 | 0 | 0 | 0 |

* 주: 표의 값은 기본 우선순위 + 주행 상태 조정 + ASIL B 등급 조정(+10) + 자율주행 레벨별 조정으로 계산되었으며, 최대값은 100으로 제한됨, 최소값은 0으로 제한됨

### 6.3 구체적 서비스 사례 분석

#### 6.3.1 주행 중 차선 이탈 경고 서비스 (레벨별 변화)

| 자율주행 레벨 | 기본 | 차량 상태 | ASIL | 자율주행 | 최종 | 결정 |
|------------|------|----------|------|---------|------|------|
| 레벨 1 | 85 | +15 | +10 | +10 | 120 | 허용 |
| 레벨 2 | 85 | +15 | +10 | +15 | 125 | 허용 |
| 레벨 3 | 85 | +15 | +10 | +20 | 130 | 허용 |
| 레벨 4 | 85 | +15 | +10 | +25 | 135 | 허용 |
| 레벨 5 | 85 | +15 | +10 | +25 | 135 | 허용 |

#### 6.3.2 주행 중 미디어 플레이어 (레벨별 변화)

| 자율주행 레벨 | 기본 | 차량 상태 | ASIL | 자율주행 | 최종 | 결정 |
|------------|------|----------|------|---------|------|------|
| 레벨 1 | 50 | -5 | 0 | 0 | 45 | 대기 |
| 레벨 2 | 50 | -5 | 0 | -10 | 35 | 거부 |
| 레벨 3 | 50 | -5 | 0 | +5 | 50 | 대기 |
| 레벨 4 | 50 | -5 | 0 | +10 | 55 | 대기 |
| 레벨 5 | 50 | -5 | 0 | +15 | 60 | 대기 |

#### 6.3.3 주차 중 소프트웨어 업데이트 (상태별 변화)

| 차량 상태 | 기본 | 차량 상태 | ASIL | 자율주행 레벨 3 | 최종 | 결정 |
|---------|------|----------|------|---------------|------|------|
| Poweroff | 25 | +20 | 0 | -30 | 15 | 거부 |
| Ignition | 25 | -10 | 0 | -30 | 0 | 거부 |
| Parking | 25 | +20 | 0 | -30 | 15 | 거부 |
| Driving | 25 | -80 | 0 | -30 | 0 | 거부 |
| Neutral | 25 | -10 | 0 | -30 | 0 | 거부 |
| Reverse | 25 | -80 | 0 | -30 | 0 | 거부 |

## 7. ISO 26262 기능 안전성 통합

### 7.1 안전 목표 및 ASIL 등급 정의

PolicyManager는 ISO 26262 자동차 기능 안전 표준을 준수하기 위해 다음과 같은 안전 목표를 정의합니다:

| 안전 목표 ID | 설명 | ASIL 등급 | 적용 대상 서비스 |
|------------|------|----------|-----------------|
| SG-01 | 차량 주행 중 안전 필수 서비스의 신뢰성 있는 실행 보장 | D | safety-critical, adas |
| SG-02 | 차량 상태에 부적합한 서비스의 실행 방지 | C | 모든 서비스 |
| SG-03 | 리소스 충돌 시 안전 필수 서비스의 우선 실행 보장 | C | safety-critical, adas, alert |

### 7.2 ASIL 등급별 안전 메커니즘

각 ASIL 등급에 따라 다음과 같은 안전 메커니즘을 적용합니다:

#### 7.2.1 ASIL D 서비스 요구사항

| 범주 | 요구사항 | 구현 방법 |
|------|----------|-----------|
| 모니터링 | 50ms 간격 모니터링 | 정기적 상태 확인 및 심박 검사 |
| 결함 처리 | 최대 3회 재시도, 500ms 내 복구 | 자동 복구 메커니즘 |
| 리소스 요구사항 | CPU 격리, 메모리 예약, 중복성 | 전용 리소스 할당 |
| 최소 우선순위 | 80 이상 보장 | 우선순위 하한선 설정 |

#### 7.2.2 ASIL C 서비스 요구사항

| 범주 | 요구사항 | 구현 방법 |
|------|----------|-----------|
| 모니터링 | 100ms 간격 모니터링 | 정기적 상태 확인 |
| 결함 처리 | 최대 2회 재시도, 1000ms 내 복구 | 자동 복구 메커니즘 |
| 리소스 요구사항 | CPU 격리, 메모리 예약 | 전용 리소스 할당 |
| 최소 우선순위 | 70 이상 보장 | 우선순위 하한선 설정 |

#### 7.2.3 ASIL B 서비스 요구사항

| 범주 | 요구사항 | 구현 방법 |
|------|----------|-----------|
| 모니터링 | 200ms 간격 모니터링 | 정기적 상태 확인 |
| 결함 처리 | 최대 2회 재시도, 2000ms 내 복구 | 자동 복구 메커니즘 |
| 리소스 요구사항 | 메모리 예약 | 메모리 할당 보장 |
| 최소 우선순위 | 60 이상 보장 | 우선순위 하한선 설정 |

#### 7.2.4 ASIL A 서비스 요구사항

| 범주 | 요구사항 | 구현 방법 |
|------|----------|-----------|
| 모니터링 | 500ms 간격 모니터링 | 정기적 상태 확인 |
| 결함 처리 | 최대 1회 재시도, 3000ms 내 복구 | 제한적 복구 시도 |
| 리소스 요구사항 | 일반 리소스 사용 | 표준 리소스 관리 |
| 최소 우선순위 | 50 이상 보장 | 우선순위 하한선 설정 |

### 7.3 안전 메커니즘 구현

#### 7.3.1 서비스 모니터링

- 실행 시간 모니터링: 서비스 실행 시간이 예상 범위를 벗어나는지 감시
- 리소스 사용 모니터링: CPU, 메모리, 네트워크 사용량 추적
- 건전성 확인: 정기적 상태 확인 및 오류 감지

#### 7.3.2 결함 감지 및 대응

- 단계적 복구 메커니즘: 경미한 문제는 자동 복구, 심각한 문제는 안전 상태로 전환
- 중요도별 대응 전략:
  - 심각: 서비스 즉시 종료 및 안전 상태 활성화
  - 주요: 서비스 일시 중단 및 복구 시도
  - 경미: 로깅 및 자동 복구 시도

#### 7.3.3 안전한 상태 전환

- 단계적 성능 저하 정의:
  - 최소 기능: 핵심 안전 기능만 유지
  - 필수 기능: 안전 및 기본 작동에 필요한 기능 유지
  - 정상 기능: 모든 기능 활성화
- 상태 전환 조건 및 시간 제약:
  - 안전 중요 서비스: 1000ms 내 완료
  - 일반 서비스: 5000ms 내 완료

### 7.4 검증 및 확인 요구사항

#### 7.4.1 테스트 요구사항

- 코드 커버리지: ASIL D 서비스는 100% 분기 커버리지 및 MC/DC 커버리지 필요
- 요구사항 추적성: 모든 안전 요구사항은 테스트 케이스와 연결되어야 함
- 장애 주입 테스트: 결함 처리 메커니즘의 효과 검증

#### 7.4.2 안전 분석

- FMEA(고장 모드 및 영향 분석): 가능한 실패 모드 식별 및 영향 평가
- FTA(결함 트리 분석): 안전 목표 위반 원인 분석
- HAZOP(위험 및 운용성 연구): 시스템 운영 중 발생 가능한 위험 식별

#### 7.4.3 독립적 검증

- 안전 중요 컴포넌트는 개발팀과 독립된 팀의 검증 필요
- 안전 평가 보고서 작성 및 승인 프로세스 정의

### 7.5 런타임 모니터링 및 진단

PolicyManager는 다음과 같은 런타임 모니터링 및 진단 기능을 제공합니다:

- 서비스 실행 시간 모니터링: 실행 시간이 임계값을 초과하는 경우 대응
- 리소스 사용량 모니터링: 메모리, CPU 사용량 추적 및 과부하 방지
- 오류 로깅: 안전 관련 오류의 상세 로깅 및 30일간 보관
- 건전성 모니터링: 1초 간격의 건전성 확인

### 7.6 안전 문서화 요구사항

ISO 26262 준수를 위한 다음 문서를 관리합니다:

- 안전 계획: 안전 활동, 일정, 담당자 정의
- 안전 요구사항 명세: 기능 안전 요구사항 상세 정의
- 안전 분석 보고서: FMEA, FTA 등 안전 분석 결과
- 테스트 명세 및 보고서: 안전 검증 테스트 절차 및 결과
- 안전 케이스: 안전 목표 달성 근거 제시
- 안전 평가 보고서: 독립적인 안전 평가 결과

## 8. RecoveryPolicy 템플릿

### 8.1 RecoveryPolicy 개요

RecoveryPolicy는 PICCOLO 프레임워크에서 서비스의 장애 감지, 상태 평가, 복구 전략을 정의하는 중요한 정책입니다. 서비스의 ASIL 등급, 차량 상태, 자율주행 등급에 따라 적응적으로 복구 전략을 적용할 수 있도록 설계되었으며, PolicyManager와 연계하여 서비스의 안정성과 가용성을 보장합니다.

본 문서는 PICCOLO 프레임워크에서 사용되는 RecoveryPolicy의 ASIL 등급별, 차량 상태별, 자율주행 등급별 템플릿을 제공합니다. 각 템플릿은 구체적인 데이터 타입과 유효 범위를 명시하며, 차량의 동적 상태에 따라 적응적으로 복구 전략을 적용할 수 있도록 설계되었습니다.

PICCOLO 프레임워크에서는 서비스의 안전성과 가용성을 보장하기 위해 ASIL 등급별 RecoveryPolicy 템플릿을 제공합니다. 이 템플릿들은 ISO 26262 표준에 따라 각 ASIL 등급에 적합한 복구 전략과 모니터링 요구사항을 정의하며, 차량 상태와 자율주행 등급에 따라 동적으로 조정됩니다.

### 8.2 RecoveryPolicy 기본 구조 및 데이터 타입

RecoveryPolicy는 다음과 같은 기본 구조를 가집니다:

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: string          # 서비스명 기반 정책 이름 (필수, 문자열, 최대 63자)
  annotations:
    description: string  # 정책 설명 (옵션, 문자열, 최대 256자)
    asilLevel: string    # ASIL 등급 (필수, 열거형: "QM", "A", "B", "C", "D")
spec:
  target:
    package: string      # 대상 패키지명 (필수, 문자열, 최대 63자)
    models: [string]     # 대상 모델 리스트 (옵션, 문자열 배열)
  stateAssessment:
    interval: string     # 모니터링 주기 (필수, 문자열 시간 형식, 범위: "5ms" ~ "10s")
    healthEndpoints:     # 상태 확인 엔드포인트 목록 (조건부 필수)
      - model: string       # 모델명 (필수, 문자열)
        endpoint: string    # 엔드포인트 경로 (필수, URI 형식)
        timeout: string     # 타임아웃 (필수, 문자열 시간 형식, 범위: "1ms" ~ "10s")
    metricThresholds:    # 상태 지표 임계값 목록 (옵션)
      - name: string        # 지표명 (필수, 문자열)
        threshold: string   # 임계값 (필수, 문자열, 숫자 또는 백분율)
        action: string      # 임계값 초과 시 액션 (필수, 열거형: "restart", "failover", "degradeMode", "alert", "notify")
    watchdogConfig:      # 워치독 설정 (ASIL C 이상 필수)
      enabled: boolean      # 활성화 여부 (필수, 불리언)
      interval: string      # 하트비트 주기 (필수, 문자열 시간 형식)
      missedThreshold: int  # 최대 누락 허용 횟수 (필수, 정수, 범위: 1-10)
  recoveryStrategies:    # 복구 전략 목록 (필수, 최소 1개 이상)
    - type: string          # 복구 전략 유형 (필수, 열거형: "failover", "restart", "degradedMode", "resourceAdjustment")
      timeout: string       # 복구 제한 시간 (필수, 문자열 시간 형식)
      # 전략별 추가 설정 (type에 따라 달라짐)
  stateTransitionValidation:  # 상태 전이 검증 설정 (ASIL B 이상 필수)
    required: boolean          # 검증 필수 여부 (필수, 불리언)
    timeout: string            # 검증 제한 시간 (필수, 문자열 시간 형식)
    validationSteps: []        # 검증 단계 (옵션)
  vehicleStateAdjustments:     # 차량 상태별 조정 설정 (옵션)
    - vehicleState: string     # 차량 상태 (필수, 열거형: "poweroff", "ignition", "parking", "driving", "neutral", "reverse")
      interval: string         # 해당 상태일 때 모니터링 주기 (옵션)
      recoveryTimeout: string  # 해당 상태일 때 복구 제한 시간 (옵션)
      priorityModifier: int    # 우선순위 수정자 (옵션, 정수, 범위: -90 ~ +20)
  autonomyLevelAdjustments:    # 자율주행 등급별 조정 설정 (옵션)
    - autonomyLevel: string    # 자율주행 등급 (필수, 열거형: "level1", "level2", "level3", "level4", "level5")
      interval: string         # 해당 등급일 때 모니터링 주기 (옵션)
      recoveryTimeout: string  # 해당 등급일 때 복구 제한 시간 (옵션)
      priorityModifier: int    # 우선순위 수정자 (옵션, 정수, 범위: -25 ~ +25)
  loggingConfig:            # 로깅 설정 (옵션)
    detailedStateTransitions: boolean  # 상세 상태 전이 로깅 (옵션, 불리언)
    persistenceLevel: string           # 로그 지속성 수준 (옵션, 열거형: "minimal", "basic", "system", "blackbox")
    storeDuration: string              # 저장 기간 (옵션, 문자열 시간 형식, 범위: "1d" ~ "30d")
  safetyMeasures:           # 안전 조치 설정 (ASIL B 이상 필수)
    fallbackMode: string       # 폴백 모드 (필수, 문자열)
    emergencyStop: {}          # 비상 정지 설정 (ASIL D 필수)
    driverNotification: {}     # 운전자 알림 설정 (ASIL A 이상 필수)
```

#### 8.2.1 주요 필드 데이터 타입 및 범위

| 필드 | 데이터 타입 | 유효 범위/값 | 비고 |
|-----|------------|-------------|-----|
| metadata.name | 문자열 | 최대 63자, 영숫자와 '-' | 패키지명 기반 명명 |
| metadata.annotations.asilLevel | 열거형 | "QM", "A", "B", "C", "D" | 서비스의 ASIL 등급 |
| spec.stateAssessment.interval | 시간 문자열 | ASIL D: "5ms"~"10ms"<br>ASIL C: "50ms"~"100ms"<br>ASIL B: "100ms"~"200ms"<br>ASIL A: "500ms"~"1s"<br>QM: "1s"~"5s" | 모니터링 주기 |
| spec.recoveryStrategies[].timeout | 시간 문자열 | ASIL D: "20ms"~"50ms"<br>ASIL C: "50ms"~"100ms"<br>ASIL B: "100ms"~"500ms"<br>ASIL A: "500ms"~"1s"<br>QM: "1s"~"5s" | 복구 제한 시간 |
| spec.recoveryStrategies[].type | 열거형 | "failover", "restart", "degradedMode", "resourceAdjustment" | 복구 전략 유형 |
| spec.vehicleStateAdjustments[].vehicleState | 열거형 | "poweroff", "ignition", "parking", "driving", "neutral", "reverse" | 차량 상태 |
| spec.autonomyLevelAdjustments[].autonomyLevel | 열거형 | "level1", "level2", "level3", "level4", "level5" | 자율주행 등급 |
| spec.loggingConfig.persistenceLevel | 열거형 | "minimal", "basic", "system", "blackbox" | 로그 지속성 수준 |
| spec.stateTransitionValidation.validationSteps[].checks[] | 문자열 배열 | "resource", "dependencies", "security", "atomicity", "consistency", "functionality", "performance", "data-integrity", "user-notification" | 검증 항목 |

### 8.3 ASIL 등급별 RecoveryPolicy 템플릿

#### 8.3.1 ASIL D RecoveryPolicy 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: <service-name>-recovery            # 문자열, 최대 63자
  annotations:
    description: "ASIL D recovery policy"  # 문자열, 최대 256자
    asilLevel: "D"                         # 열거형: "D"
spec:
  target:
    package: <package-name>                # 문자열, 최대 63자
    models:                                # 문자열 배열, 옵션
      - <primary-component>                # 주 컴포넌트 명
      - <secondary-component>              # 백업 컴포넌트 명
  stateAssessment:
    interval: "5ms"                        # 시간 문자열, 범위: "5ms"~"10ms"
    healthEndpoints:                       # 객체 배열, 필수
      - model: <primary-component>         # 문자열
        endpoint: "/health"                # URI 형식
        timeout: "2ms"                     # 시간 문자열, 범위: "1ms"~"5ms"
      - model: <secondary-component>       # 문자열
        endpoint: "/health"                # URI 형식
        timeout: "2ms"                     # 시간 문자열, 범위: "1ms"~"5ms"
    watchdogConfig:                        # 객체, 필수
      enabled: true                        # 불리언
      interval: "5ms"                      # 시간 문자열, 범위: "5ms"~"10ms"
      missedThreshold: 1                   # 정수, 범위: 1-2
    metricThresholds:                      # 객체 배열, 옵션
      - name: "responseTime"               # 문자열
        threshold: "3ms"                   # 문자열, 숫자 또는 백분율
        action: "failover"                 # 열거형: "failover"
      - name: "brakeActuationDelay"        # 문자열
        threshold: "4ms"                   # 문자열, 숫자 또는 백분율
        action: "failover"                 # 열거형: "failover"
  recoveryStrategies:                      # 객체 배열, 필수
    - type: "failover"                     # 열거형: "failover"
      timeout: "20ms"                      # 시간 문자열, 범위: "10ms"~"25ms"
      maxRetries: 0                        # 정수, 범위: 0-1
      backupInstance: true                 # 불리언
      validationRequired: true             # 불리언
      stateSync: true                      # 불리언
      dataSyncRequired: true               # 불리언
    - type: "restart"                      # 열거형: "restart"
      timeout: "50ms"                      # 시간 문자열, 범위: "40ms"~"50ms"
      maxRetries: 1                        # 정수, 범위: 0-1
      gracePeriod: "10ms"                  # 시간 문자열
      coldRestart: false                   # 불리언
      statePreservation: true              # 불리언
    - type: "degradedMode"                 # 열거형: "degradedMode"
      activationCondition: "failover-failed AND restart-failed"  # 문자열, 조건식
      notificationLevel: "critical"        # 열거형: "critical"
      functionalityReduction:              # 객체, 필수
        maintain: []                       # 문자열 배열, 필수
        reduce: []                         # 문자열 배열, 필수
      timeLimit: "1m"                      # 시간 문자열
      continuousReassessment: true         # 불리언
  isolationStrategy:                       # 객체, 필수
    enabled: true                          # 불리언
    scope: "container"                     # 열거형: "container", "process", "thread"
    faultContainment: true                 # 불리언
    resourceIsolation: true                # 불리언
  resourceGuarantee:                       # 객체, 필수
    preemptive: true                       # 불리언
    reservedResources:                     # 객체, 필수
      cpu: <cpu-value>                     # 문자열, CPU 자원량
      memory: <memory-value>               # 문자열, 메모리 자원량
  stateTransitionValidation:               # 객체, 필수
    required: true                         # 불리언
    timeout: "5ms"                         # 시간 문자열, 범위: "5ms"~"10ms"
    validationSteps:                       # 객체 배열, 필수
      - step: "pre-transition"             # 열거형: "pre-transition"
        checks:                            # 문자열 배열, 필수
          - "resource"                     # 문자열, 검증 항목
          - "dependencies"                 # 문자열, 검증 항목
          - "security"                     # 문자열, 검증 항목
      - step: "during-transition"          # 열거형: "during-transition"
        checks:                            # 문자열 배열, 필수
          - "atomicity"                    # 문자열, 검증 항목
          - "consistency"                  # 문자열, 검증 항목
      - step: "post-transition"            # 열거형: "post-transition"
        checks:                            # 문자열 배열, 필수
          - "functionality"                # 문자열, 검증 항목
          - "performance"                  # 문자열, 검증 항목
          - "data-integrity"               # 문자열, 검증 항목
  vehicleStateAdjustments:                 # 객체 배열, 옵션
    - vehicleState: "driving"              # 열거형: "driving"
      interval: "5ms"                      # 시간 문자열
      recoveryTimeout: "15ms"              # 시간 문자열
      priorityModifier: 10                 # 정수, 범위: -90~+20
    - vehicleState: "parking"              # 열거형: "parking"
      interval: "10ms"                     # 시간 문자열
      recoveryTimeout: "30ms"              # 시간 문자열
      priorityModifier: 0                  # 정수, 범위: -90~+20
  autonomyLevelAdjustments:                # 객체 배열, 옵션
    - autonomyLevel: "level3"              # 열거형: "level3"
      interval: "5ms"                      # 시간 문자열
      recoveryTimeout: "15ms"              # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -25~+25
    - autonomyLevel: "level4"              # 열거형: "level4"
      interval: "5ms"                      # 시간 문자열
      recoveryTimeout: "10ms"              # 시간 문자열
      priorityModifier: 20                 # 정수, 범위: -25~+25
  loggingConfig:                           # 객체, 필수
    detailedStateTransitions: true         # 불리언
    persistenceLevel: "blackbox"           # 열거형: "blackbox"
    storeDuration: "30d"                   # 시간 문자열, 범위: "7d"~"30d"
    criticalEvents:                        # 문자열 배열, 필수
      - "failover-trigger"                 # 문자열, 이벤트명
      - "recovery-failed"                  # 문자열, 이벤트명
      - "degraded-mode-activated"          # 문자열, 이벤트명
  safetyMeasures:                          # 객체, 필수
    fallbackMode: <fallback-mode>          # 문자열, 서비스별 안전 모드
    emergencyStop:                         # 객체, 필수
      enabled: true                        # 불리언
      conditions: "all-recovery-failed AND <service-condition>"  # 문자열, 조건식
    driverNotification:                    # 객체, 필수
      level: "critical"                    # 열거형: "critical"
      visualAlert: true                    # 불리언
      audioAlert: true                     # 불리언
```

#### 8.3.2 ASIL D 서비스별 커스터마이징 필드

ASIL D 템플릿에서 서비스별로 반드시 커스터마이징해야 하는 필드:

1. **metadata.name**: 서비스명 기반 정책 이름 (예: "emergency-brake-recovery")
2. **spec.target.package**: 대상 패키지명 (예: "emergency-brake-system")
3. **spec.target.models**: 서비스 컴포넌트 목록 (예: ["primary-brake-controller", "secondary-brake-controller"])
4. **spec.recoveryStrategies[degradedMode].functionalityReduction**: 서비스별 핵심/감소 기능 (예: maintain: ["emergency-braking", "abs"], reduce: ["comfort-features"])
5. **spec.resourceGuarantee.reservedResources**: 서비스별 필요 리소스 (예: cpu: "2", memory: "1Gi")
6. **spec.safetyMeasures.fallbackMode**: 서비스별 안전 모드 (예: "mechanical-backup")
7. **spec.safetyMeasures.emergencyStop.conditions**: 서비스별 비상 정지 조건 (예: "all-recovery-failed AND vehicle-moving")

#### 8.4 ASIL C RecoveryPolicy 템플릿

#### 8.4.1 기본 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: <service-name>-recovery            # 문자열, 최대 63자
  annotations:
    description: "ASIL C recovery policy"  # 문자열, 최대 256자
    asilLevel: "C"                         # 열거형: "C"
spec:
  target:
    package: <package-name>                # 문자열, 최대 63자
    models:                                # 문자열 배열, 옵션
      - <primary-component>                # 주 컴포넌트 명
      - <backup-component>                 # 백업 컴포넌트 명 (옵션)
  stateAssessment:
    interval: "50ms"                       # 시간 문자열, 범위: "50ms"~"100ms"
    healthEndpoints:                       # 객체 배열, 필수
      - model: <primary-component>         # 문자열
        endpoint: "/health"                # URI 형식
        timeout: "20ms"                    # 시간 문자열, 범위: "10ms"~"30ms"
    metricThresholds:                      # 객체 배열, 옵션
      - name: "responseTime"               # 문자열
        threshold: "40ms"                  # 문자열, 숫자 또는 백분율
        action: "restart"                  # 열거형: "restart", "failover", "degradeMode", "alert", "notify"
      - name: "qualityMetric"              # 문자열
        threshold: "80%"                   # 문자열, 숫자 또는 백분율
        action: "degradeMode"              # 열거형: "degradeMode"
    watchdogConfig:                        # 객체, 필수
      enabled: true                        # 불리언
      interval: "50ms"                     # 시간 문자열, 범위: "50ms"~"100ms"
      missedThreshold: 2                   # 정수, 범위: 1-3
  recoveryStrategies:                      # 객체 배열, 필수
    - type: "failover"                     # 열거형: "failover"
      timeout: "50ms"                      # 시간 문자열, 범위: "40ms"~"70ms"
      maxRetries: 1                        # 정수, 범위: 0-2
      backupInstance: true                 # 불리언
      validationRequired: true             # 불리언
      stateSync: true                      # 불리언
    - type: "degradedMode"                 # 열거형: "degradedMode"
      timeout: "100ms"                     # 시간 문자열, 범위: "70ms"~"200ms"
      levels:                              # 객체 배열, 필수
        - name: "reduced-precision"        # 문자열, 저하 모드 이름
          conditions: "qualityMetric < 80% AND qualityMetric >= 60%"  # 문자열, 조건식
          features:                        # 객체, 필수
            reduced: ["high-precision"]    # 문자열 배열, 감소할 기능 목록
          stateMapping: "Degraded"         # 문자열, 상태 매핑
        - name: "minimal-functionality"    # 문자열, 저하 모드 이름
          conditions: "qualityMetric < 60%" # 문자열, 조건식
          features:                        # 객체, 필수
            disabled: ["advanced-features"] # 문자열 배열, 비활성화할 기능 목록
            reduced: ["update-frequency"]  # 문자열 배열, 감소할 기능 목록
          stateMapping: "ReducedFunctionality" # 문자열, 상태 매핑
      notificationRequired: true           # 불리언
    - type: "restart"                      # 열거형: "restart"
      timeout: "100ms"                     # 시간 문자열, 범위: "80ms"~"150ms"
      maxRetries: 2                        # 정수, 범위: 1-3
      gracePeriod: "50ms"                  # 시간 문자열, 범위: "20ms"~"80ms"
      statePreservation: true              # 불리언
    - type: "resourceAdjustment"           # 열거형: "resourceAdjustment"
      timeout: "80ms"                      # 시간 문자열, 범위: "60ms"~"120ms"
      adjustments:                         # 객체 배열, 필수
        - resource: "cpu"                  # 문자열, 조정 대상 리소스
          action: "increase"               # 열거형: "increase", "decrease", "reset"
          limit: "available"               # 문자열, 리소스 한도
  stateTransitionValidation:               # 객체, 필수
    required: true                         # 불리언
    timeout: "20ms"                        # 시간 문자열, 범위: "10ms"~"30ms"
    validationSteps:                       # 객체 배열, 필수
      - step: "pre-transition"             # 열거형: "pre-transition"
        checks:                            # 문자열 배열, 필수
          - "resource"                     # 문자열, 검증 항목
          - "dependencies"                 # 문자열, 검증 항목
      - step: "post-transition"            # 열거형: "post-transition"
        checks:                            # 문자열 배열, 필수
          - "functionality"                # 문자열, 검증 항목
          - "user-notification"            # 문자열, 검증 항목
  vehicleStateAdjustments:                 # 객체 배열, 옵션
    - vehicleState: "driving"              # 열거형: "driving"
      interval: "50ms"                     # 시간 문자열
      recoveryTimeout: "90ms"              # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -90~+20
    - vehicleState: "parking"              # 열거형: "parking"
      interval: "80ms"                     # 시간 문자열
      recoveryTimeout: "150ms"             # 시간 문자열
      priorityModifier: -10                # 정수, 범위: -90~+20
  autonomyLevelAdjustments:                # 객체 배열, 옵션
    - autonomyLevel: "level2"              # 열거형: "level2"
      interval: "50ms"                     # 시간 문자열
      recoveryTimeout: "90ms"              # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -25~+25
    - autonomyLevel: "level3"              # 열거형: "level3"
      interval: "50ms"                     # 시간 문자열
      recoveryTimeout: "90ms"              # 시간 문자열
      priorityModifier: 20                 # 정수, 범위: -25~+25
    - autonomyLevel: "level4"              # 열거형: "level4"
      interval: "40ms"                     # 시간 문자열
      recoveryTimeout: "80ms"              # 시간 문자열
      priorityModifier: 25                 # 정수, 범위: -25~+25
  loggingConfig:                           # 객체, 필수
    detailedStateTransitions: true         # 불리언
    persistenceLevel: "system"             # 열거형: "minimal", "basic", "system", "blackbox"
    storeDuration: "15d"                   # 시간 문자열, 범위: "7d"~"30d"
  safetyMeasures:                          # 객체, 필수
    fallbackMode: <fallback-mode>          # 문자열, 서비스별 안전 모드
    driverNotification:                    # 객체, 필수
      level: "warning"                     # 열거형: "info", "warning", "critical"
      visualAlert: true                    # 불리언
      audioAlert: true                     # 불리언
    handoverStrategy:                      # 객체, 옵션
      enabled: true                        # 불리언
      warningTime: "3s"                    # 시간 문자열
```

#### 8.4.2 서비스별 커스터마이징 필드

ASIL C 템플릿에서 서비스별로 반드시 커스터마이징해야 하는 필드:

1. **metadata.name**: 서비스명 기반 정책 이름 (예: "lane-keeping-recovery")
2. **spec.target.package**: 대상 패키지명 (예: "lane-keeping-assist")
3. **spec.target.models**: 서비스 컴포넌트 목록 (예: ["lane-detection", "lane-controller"])
4. **stateAssessment.metricThresholds**: 서비스별 품질 지표 임계값 (예: detectionQuality: "75%")
5. **recoveryStrategies[degradedMode].levels**: 서비스별 저하 모드 정의 (예: name: "reduced-precision", features: { reduced: ["detection-precision"] })
6. **safetyMeasures.fallbackMode**: 서비스별 안전 모드 (예: "driver-control")
7. **safetyMeasures.handoverStrategy**: 운전자 제어 전환 설정 (예: warningTime: "3s")

#### 8.5 ASIL B RecoveryPolicy 템플릿

#### 8.5.1 기본 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: <service-name>-recovery            # 문자열, 최대 63자
  annotations:
    description: "ASIL B recovery policy"  # 문자열, 최대 256자
    asilLevel: "B"                         # 열거형: "B"
spec:
  target:
    package: <package-name>                # 문자열, 최대 63자
    models:                                # 문자열 배열, 옵션
      - <main-component>                   # 주 컴포넌트 명
  stateAssessment:
    interval: "100ms"                      # 시간 문자열, 범위: "100ms"~"200ms"
    healthEndpoints:                       # 객체 배열, 필수
      - model: <main-component>            # 문자열
        endpoint: "/health"                # URI 형식
        timeout: "50ms"                    # 시간 문자열, 범위: "30ms"~"80ms"
    metricThresholds:                      # 객체 배열, 옵션
      - name: "responseTime"               # 문자열
        threshold: "80ms"                  # 문자열, 숫자 또는 백분율
        action: "restart"                  # 열거형: "restart", "degradeMode", "alert"
      - name: "accuracyMetric"             # 문자열
        threshold: "80%"                   # 문자열, 숫자 또는 백분율
        action: "degradeMode"              # 열거형: "degradeMode"
    watchdogConfig:                        # 객체, 옵션
      enabled: true                        # 불리언
      interval: "100ms"                    # 시간 문자열, 범위: "100ms"~"200ms"
      missedThreshold: 2                   # 정수, 범위: 1-3
  recoveryStrategies:                      # 객체 배열, 필수
    - type: "restart"                      # 열거형: "restart"
      timeout: "500ms"                     # 시간 문자열, 범위: "300ms"~"800ms"
      maxRetries: 3                        # 정수, 범위: 1-5
      gracePeriod: "100ms"                 # 시간 문자열, 범위: "50ms"~"150ms"
      statePreservation: true              # 불리언
    - type: "degradedMode"                 # 열거형: "degradedMode"
      timeout: "500ms"                     # 시간 문자열, 범위: "300ms"~"800ms"
      activationCondition: "accuracyMetric < 80% OR restart-failed"  # 문자열, 조건식
      notificationLevel: "warning"         # 열거형: "info", "warning", "critical"
      functionalityReduction:              # 객체, 필수
        maintain: ["basic-functions"]      # 문자열 배열, 유지할 기능 목록
        reduce: ["advanced-features"]      # 문자열 배열, 감소할 기능 목록
  stateTransitionValidation:               # 객체, 필수
    required: true                         # 불리언
    timeout: "50ms"                        # 시간 문자열, 범위: "30ms"~"80ms"
    validationSteps:                       # 객체 배열, 필수
      - step: "post-transition"            # 열거형: "post-transition"
        checks:                            # 문자열 배열, 필수
          - "basic-functionality"          # 문자열, 검증 항목
          - "user-notification"            # 문자열, 검증 항목
  vehicleStateAdjustments:                 # 객체 배열, 옵션
    - vehicleState: "driving"              # 열거형: "driving"
      interval: "100ms"                    # 시간 문자열
      recoveryTimeout: "400ms"             # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -90~+20
    - vehicleState: "reverse"              # 열거형: "reverse"
      interval: "100ms"                    # 시간 문자열
      recoveryTimeout: "400ms"             # 시간 문자열
      priorityModifier: 10                 # 정수, 범위: -90~+20
  autonomyLevelAdjustments:                # 객체 배열, 옵션
    - autonomyLevel: "level2"              # 열거형: "level2"
      interval: "100ms"                    # 시간 문자열
      recoveryTimeout: "400ms"             # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -25~+25
    - autonomyLevel: "level3"              # 열거형: "level3"
      interval: "90ms"                     # 시간 문자열
      recoveryTimeout: "350ms"             # 시간 문자열
      priorityModifier: 20                 # 정수, 범위: -25~+25
  loggingConfig:                           # 객체, 옵션
    detailedStateTransitions: true         # 불리언
    persistenceLevel: "system"             # 열거형: "minimal", "basic", "system", "blackbox"
    storeDuration: "7d"                    # 시간 문자열, 범위: "3d"~"15d"
  safetyMeasures:                          # 객체, 필수
    fallbackMode: <fallback-mode>          # 문자열, 서비스별 안전 모드
    driverNotification:                    # 객체, 필수
      level: "warning"                     # 열거형: "info", "warning", "critical"
      visualAlert: true                    # 불리언
      audioAlert: true                     # 불리언
```

#### 8.5.2 서비스별 커스터마이징 필드

ASIL B 템플릿에서 서비스별로 반드시 커스터마이징해야 하는 필드:

1. **metadata.name**: 서비스명 기반 정책 이름 (예: "collision-warning-recovery")
2. **spec.target.package**: 대상 패키지명 (예: "collision-warning-system")
3. **spec.target.models**: 서비스 컴포넌트 목록 (예: ["collision-detection"])
4. **stateAssessment.metricThresholds**: 서비스별 품질 지표 임계값 (예: detectionAccuracy: "80%")
5. **recoveryStrategies[degradedMode].functionalityReduction**: 서비스별 감소 기능 목록 (예: maintain: ["warnings"], reduce: ["automatic-braking"])
6. **safetyMeasures.fallbackMode**: 서비스별 안전 모드 (예: "manual-operation")

#### 8.6 ASIL A RecoveryPolicy 템플릿

#### 8.6.1 기본 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: <service-name>-recovery            # 문자열, 최대 63자
  annotations:
    description: "ASIL A recovery policy"  # 문자열, 최대 256자
    asilLevel: "A"                         # 열거형: "A"
spec:
  target:
    package: <package-name>                # 문자열, 최대 63자
    models:                                # 문자열 배열, 옵션
      - <main-component>                   # 주 컴포넌트 명
  stateAssessment:
    interval: "500ms"                      # 시간 문자열, 범위: "500ms"~"1s"
    healthEndpoints:                       # 객체 배열, 필수
      - model: <main-component>            # 문자열
        endpoint: "/health"                # URI 형식
        timeout: "200ms"                   # 시간 문자열, 범위: "100ms"~"300ms"
    metricThresholds:                      # 객체 배열, 옵션
      - name: "responseTime"               # 문자열
        threshold: "400ms"                 # 문자열, 숫자 또는 백분율
        action: "restart"                  # 열거형: "restart", "degradeMode", "alert", "notify"
  recoveryStrategies:                      # 객체 배열, 필수
    - type: "restart"                      # 열거형: "restart"
      timeout: "1s"                        # 시간 문자열, 범위: "800ms"~"2s"
      maxRetries: 3                        # 정수, 범위: 1-5
      gracePeriod: "500ms"                 # 시간 문자열, 범위: "300ms"~"800ms"
      statePreservation: false             # 불리언
    - type: "degradedMode"                 # 열거형: "degradedMode"
      timeout: "1s"                        # 시간 문자열, 범위: "800ms"~"2s"
      activationCondition: "restart-failed OR responseTime > 400ms"  # 문자열, 조건식
      notificationLevel: "info"            # 열거형: "info", "warning"
      functionalityReduction:              # 객체, 필수
        reduce: ["non-critical-features"]  # 문자열 배열, 감소할 기능 목록
  stateTransitionValidation:               # 객체, 옵션
    required: false                        # 불리언
    timeout: "300ms"                       # 시간 문자열, 범위: "200ms"~"500ms"
  vehicleStateAdjustments:                 # 객체 배열, 옵션
    - vehicleState: "driving"              # 열거형: "driving"
      interval: "500ms"                    # 시간 문자열
      recoveryTimeout: "1.5s"              # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -90~+20
    - vehicleState: "parking"              # 열거형: "parking"
      interval: "800ms"                    # 시간 문자열
      recoveryTimeout: "2s"                # 시간 문자열
      priorityModifier: 5                  # 정수, 범위: -90~+20
  autonomyLevelAdjustments:                # 객체 배열, 옵션
    - autonomyLevel: "level2"              # 열거형: "level2"
      interval: "500ms"                    # 시간 문자열
      recoveryTimeout: "1.5s"              # 시간 문자열
      priorityModifier: 15                 # 정수, 범위: -25~+25
    - autonomyLevel: "level3"              # 열거형: "level3"
      interval: "400ms"                    # 시간 문자열
      recoveryTimeout: "1.2s"              # 시간 문자열
      priorityModifier: 20                 # 정수, 범위: -25~+25
  loggingConfig:                           # 객체, 옵션
    detailedStateTransitions: false        # 불리언
    persistenceLevel: "basic"              # 열거형: "minimal", "basic", "system", "blackbox"
    storeDuration: "3d"                    # 시간 문자열, 범위: "1d"~"7d"
  safetyMeasures:                          # 객체, 필수
    fallbackMode: <fallback-mode>          # 문자열, 서비스별 안전 모드
    driverNotification:                    # 객체, 필수
      level: "info"                        # 열거형: "info", "warning", "critical"
      visualAlert: true                    # 불리언
      audioAlert: false                    # 불리언
```

#### 8.6.2 서비스별 커스터마이징 필드

ASIL A 템플릿에서 서비스별로 반드시 커스터마이징해야 하는 필드:

1. **metadata.name**: 서비스명 기반 정책 이름 (예: "driver-monitoring-recovery")
2. **spec.target.package**: 대상 패키지명 (예: "driver-monitoring-system")
3. **spec.target.models**: 서비스 컴포넌트 목록 (예: ["attention-monitoring"])
4. **stateAssessment.metricThresholds**: 서비스별 품질 지표 임계값 (예: attentionScore: "70%")
5. **recoveryStrategies[degradedMode].functionalityReduction**: 서비스별 기능 제한 목록 (예: reduce: ["advanced-alerts"])
6. **safetyMeasures.fallbackMode**: 서비스별 안전 모드 (예: "basic-monitoring")

### 8.7 QM RecoveryPolicy 템플릿

#### 8.7.1 기본 템플릿

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: <service-name>-recovery            # 문자열, 최대 63자
  annotations:
    description: "QM recovery policy"      # 문자열, 최대 256자
    asilLevel: "QM"                        # 열거형: "QM"
spec:
  target:
    package: <package-name>                # 문자열, 최대 63자
  stateAssessment:
    interval: "1s"                         # 시간 문자열, 범위: "1s"~"5s"
    basicCheck:                            # 객체, 필수
      enabled: true                        # 불리언
      timeout: "500ms"                     # 시간 문자열, 범위: "300ms"~"1s"
    healthEndpoints:                       # 객체 배열, 옵션
      - endpoint: "/health"                # URI 형식
        timeout: "500ms"                   # 시간 문자열, 범위: "300ms"~"1s"
  recoveryStrategies:                      # 객체 배열, 필수
    - type: "restart"                      # 열거형: "restart"
      timeout: "5s"                        # 시간 문자열, 범위: "3s"~"10s"
      maxRetries: 5                        # 정수, 범위: 1-10
      backoffSeconds: 30                   # 정수, 범위: 10-60
      gracePeriod: "2s"                    # 시간 문자열, 범위: "1s"~"5s"
  vehicleStateAdjustments:                 # 객체 배열, 옵션
    - vehicleState: "driving"              # 열거형: "driving"
      interval: "2s"                       # 시간 문자열, 범위: "1s"~"5s"
      recoveryTimeout: "8s"                # 시간 문자열, 범위: "5s"~"15s"
      priorityModifier: -20                # 정수, 범위: -90~+10
    - vehicleState: "parking"              # 열거형: "parking"
      interval: "1s"                       # 시간 문자열, 범위: "1s"~"3s"
      recoveryTimeout: "4s"                # 시간 문자열, 범위: "3s"~"10s"
      priorityModifier: 15                 # 정수, 범위: -10~+20
  autonomyLevelAdjustments:                # 객체 배열, 옵션
    - autonomyLevel: "level4"              # 열거형: "level4"
      interval: "1s"                       # 시간 문자열, 범위: "1s"~"3s"
      recoveryTimeout: "4s"                # 시간 문자열, 범위: "3s"~"8s"
      priorityModifier: 5                  # 정수, 범위: -10~+15
    - autonomyLevel: "level5"              # 열거형: "level5"
      interval: "1s"                       # 시간 문자열, 범위: "1s"~"3s"
      recoveryTimeout: "3s"                # 시간 문자열, 범위: "2s"~"6s"
      priorityModifier: 15                 # 정수, 범위: -5~+25
  loggingConfig:                           # 객체, 옵션
    detailedStateTransitions: false        # 불리언
    persistenceLevel: "minimal"            # 열거형: "minimal", "basic"
    storeDuration: "1d"                    # 시간 문자열, 범위: "1d"~"7d"
```

#### 8.7.2 서비스별 커스터마이징 필드

QM 템플릿에서 서비스별로 반드시 커스터마이징해야 하는 필드:

1. **metadata.name**: 서비스명 기반 정책 이름 (예: "media-player-recovery")
2. **spec.target.package**: 대상 패키지명 (예: "media-playback-system")
3. **vehicleStateAdjustments**: 차량 상태별 조정 설정 (주행 중 미디어 플레이어는 우선순위 낮게, 주차 중에는 높게)
4. **autonomyLevelAdjustments**: 자율주행 등급별 조정 설정 (높은 자율주행 레벨에서는 인포테인먼트 우선순위 상향)

### 8.8 차량 상태별 RecoveryPolicy 조정

차량의 동적 상태에 따라 복구 정책을 최적화하기 위한 조정 방법입니다.

#### 8.8.1 주요 차량 상태별 조정 전략

| 차량 상태 | 조정 방향 | ASIL D | ASIL C | ASIL B | ASIL A | QM |
|----------|---------|-------|-------|-------|-------|-----|
| 주행 중(driving) | 안전 우선, 응답 시간 단축 | interval: "5ms"<br>priority: +10 | interval: "50ms"<br>priority: +15 | interval: "100ms"<br>priority: +15 | interval: "500ms"<br>priority: +15 | interval: "2s"<br>priority: -20 |
| 주차 중(parking) | 사용성 우선, 일부 시간 여유 | interval: "10ms"<br>priority: 0 | interval: "80ms"<br>priority: -10 | interval: "150ms"<br>priority: -10 | interval: "800ms"<br>priority: +5 | interval: "1s"<br>priority: +15 |
| 시동 중(ignition) | 기본 서비스 빠른 시작 | interval: "5ms"<br>priority: +10 | interval: "50ms"<br>priority: +5 | interval: "100ms"<br>priority: +5 | interval: "500ms"<br>priority: +5 | interval: "1s"<br>priority: -10 |
| 중립(neutral) | 안전 서비스 유지 | interval: "5ms"<br>priority: +5 | interval: "50ms"<br>priority: +5 | interval: "100ms"<br>priority: +5 | interval: "500ms"<br>priority: +5 | interval: "1s"<br>priority: 0 |
| 후진(reverse) | 안전 서비스 우선 | interval: "5ms"<br>priority: +10 | interval: "50ms"<br>priority: +10 | interval: "100ms"<br>priority: +10 | interval: "500ms"<br>priority: +10 | interval: "1s"<br>priority: -15 |
| 전원 꺼짐(poweroff) | 최소 서비스만 유지 | interval: "10ms"<br>priority: -50 | interval: "100ms"<br>priority: -50 | interval: "200ms"<br>priority: -70 | interval: "1s"<br>priority: -90 | interval: "5s"<br>priority: -90 |

#### 8.8.2 서비스 타입별 차량 상태 조정 우선순위

차량 상태에 따른 서비스 유형별 우선순위 조정 권장값은 다음과 같습니다:

| 서비스 유형 | 주행(driving) | 주차(parking) | 시동(ignition) | 중립(neutral) | 후진(reverse) | 전원 꺼짐(poweroff) |
|------------|--------------|--------------|--------------|--------------|--------------|------------------|
| 안전 필수(safety-critical) | +10 | 0 | +10 | +5 | +10 | -50 |
| 운전자 지원(adas) | +15 | -10 | +5 | +10 | +15 | -90 |
| 헤드업 디스플레이(hud) | +15 | -20 | +5 | +10 | +5 | -90 |
| 네비게이션(navigation) | +15 | +5 | 0 | +10 | -5 | -90 |
| 클러스터(cluster) | +10 | -5 | +10 | +5 | +5 | -90 |
| 경고(alert) | +10 | +5 | +10 | +5 | +10 | -50 |
| 음성 제어(voice-control) | +5 | +10 | -10 | +5 | -5 | -90 |
| 미디어(media) | -5 | +15 | -20 | +5 | -15 | -90 |
| 인포테인먼트(infotainment) | -10 | +15 | -20 | +5 | -15 | -90 |
| 업데이트(update) | -80 | +20 | -10 | -10 | -80 | +20 |

### 8.9 자율주행 등급별 RecoveryPolicy 조정

자율주행 등급에 따라 복구 정책을 최적화하기 위한 조정 방법입니다.

#### 8.9.1 자율주행 등급별 조정 전략

| 자율주행 등급 | 조정 방향 | ASIL D | ASIL C | ASIL B | ASIL A | QM |
|--------------|---------|-------|-------|-------|-------|-----|
| 레벨 1 | 기본 운전자 지원 | interval: "5ms"<br>priority: +5 | interval: "50ms"<br>priority: +10 | interval: "100ms"<br>priority: +10 | interval: "500ms"<br>priority: +5 | interval: "2s"<br>priority: 0 |
| 레벨 2 | 부분 자동화 | interval: "5ms"<br>priority: +10 | interval: "50ms"<br>priority: +15 | interval: "100ms"<br>priority: +15 | interval: "500ms"<br>priority: +15 | interval: "1.5s"<br>priority: -5 |
| 레벨 3 | 조건부 자동화 | interval: "5ms"<br>priority: +15 | interval: "50ms"<br>priority: +20 | interval: "90ms"<br>priority: +20 | interval: "400ms"<br>priority: +20 | interval: "1.2s"<br>priority: 0 |
| 레벨 4 | 고도 자동화 | interval: "5ms"<br>priority: +20 | interval: "40ms"<br>priority: +25 | interval: "80ms"<br>priority: +25 | interval: "300ms"<br>priority: +20 | interval: "1s"<br>priority: +5 |
| 레벨 5 | 완전 자동화 | interval: "5ms"<br>priority: +25 | interval: "40ms"<br>priority: +25 | interval: "80ms"<br>priority: +25 | interval: "300ms"<br>priority: +20 | interval: "1s"<br>priority: +15 |

#### 8.9.2 서비스 타입별 자율주행 등급 조정 우선순위

자율주행 등급에 따른 서비스 유형별 우선순위 조정 권장값은 다음과 같습니다:

| 서비스 유형 | 레벨 1 | 레벨 2 | 레벨 3 | 레벨 4 | 레벨 5 |
|------------|-------|-------|-------|-------|-------|
| 안전 필수(safety-critical) | +5 | +10 | +15 | +20 | +25 |
| 운전자 지원(adas) | +10 | +15 | +20 | +25 | +25 |
| 헤드업 디스플레이(hud) | +10 | +15 | +10 | +5 | 0 |
| 네비게이션(navigation) | +5 | +10 | +5 | +15 | +20 |
| 클러스터(cluster) | +5 | +10 | +10 | +10 | +10 |
| 경고(alert) | +10 | +15 | +15 | +15 | +15 |
| 음성 제어(voice-control) | 0 | +5 | +10 | +15 | +15 |
| 미디어(media) | 0 | -10 | +5 | +10 | +15 |
| 인포테인먼트(infotainment) | -5 | -15 | 0 | +5 | +15 |
| 업데이트(update) | -15 | -25 | -30 | -10 | 0 |

#### 8.9.3 조정 적용 가이드

1. **서비스 유형 식별**: 서비스가 어떤 유형(안전 필수, 운전자 지원, 미디어 등)에 해당하는지 결정
2. **기본 템플릿 선택**: 서비스의 ASIL 등급에 맞는 기본 템플릿 선택
3. **차량 상태 조정 적용**: 서비스 유형과 ASIL 등급에 맞는 차량 상태별 interval, timeout, priorityModifier 값 적용
4. **자율주행 등급 조정 적용**: 서비스 유형과 ASIL 등급에 맞는 자율주행 등급별 interval, timeout, priorityModifier 값 적용
5. **조정 값 검증**: 모든 값이 유효 범위 내에 있는지 확인

### 8.11 구체적 구현 가이드

#### 8.11.1 차량 상태별 커스터마이징 예시

제동 제어 시스템(ASIL D)의 차량 상태별 설정 예시:

```yaml
vehicleStateAdjustments:
  - vehicleState: "driving"
    interval: "5ms"
    recoveryTimeout: "15ms"
    priorityModifier: 10
  - vehicleState: "parking"
    interval: "10ms"
    recoveryTimeout: "30ms"
    priorityModifier: 0
  - vehicleState: "reverse"
    interval: "5ms"
    recoveryTimeout: "15ms"
    priorityModifier: 10
```

#### 8.11.2 자율주행 등급별 커스터마이징 예시

차선 유지 보조(ASIL C)의 자율주행 등급별 설정 예시:

```yaml
autonomyLevelAdjustments:
  - autonomyLevel: "level2"
    interval: "50ms"
    recoveryTimeout: "90ms"
    priorityModifier: 15
  - autonomyLevel: "level3"
    interval: "50ms"
    recoveryTimeout: "90ms"
    priorityModifier: 20
  - autonomyLevel: "level4"
    interval: "40ms"
    recoveryTimeout: "80ms"
    priorityModifier: 25
```

#### 8.11.3 복구 전략 타입별 구현 세부 사항

**Failover 전략**:
- maxRetries: ASIL D는 0-1, ASIL C는 1-2, ASIL B는 1-3, ASIL A는 2-3
- backupInstance: ASIL D,C는 반드시 true, ASIL B,A는 서비스에 따라 선택
- stateSync: ASIL D,C는 반드시 true, ASIL B,A는 선택적 true

**Restart 전략**:
- gracePeriod: ASIL D: "10ms", ASIL C: "50ms", ASIL B: "100ms", ASIL A: "500ms"
- coldRestart: 일반적으로 false, 심각한 오류 발생 시에만 true
- statePreservation: ASIL D,C,B는 true, ASIL A,QM은 선택적 true

**DegradedMode 전략**:
- notificationLevel: ASIL D: "critical", ASIL C: "warning", ASIL B: "warning", ASIL A,QM: "info"
- timeLimit: ASIL D: "1m", ASIL C: "5m", ASIL B: "10m", ASIL A,QM: "30m"
- continuousReassessment: ASIL D,C는 true, 나머지는 서비스에 따라 선택

#### 8.11.4 템플릿 적용 예시

##### ASIL D 제동 시스템 예시

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: emergency-brake-recovery
  annotations:
    description: "ASIL D emergency brake system recovery policy"
    asilLevel: "D"
spec:
  target:
    package: emergency-brake-system
    models:
      - primary-brake-controller
      - secondary-brake-controller
  stateAssessment:
    interval: "5ms"
    healthEndpoints:
      - model: "primary-brake-controller"
        endpoint: "/health"
        timeout: "2ms"
      - model: "secondary-brake-controller"
        endpoint: "/health"
        timeout: "2ms"
    watchdogConfig:
      enabled: true
      interval: "5ms"
      missedThreshold: 1
    metricThresholds:
      - name: "responseTime"
        threshold: "3ms"
        action: "failover"
      - name: "brakeActuationDelay"
        threshold: "4ms"
        action: "failover"
  recoveryStrategies:
    - type: failover
      timeout: 20ms
      maxRetries: 0
      backupInstance: true
      validationRequired: true
      stateSync: true
    - type: degradedMode
      activationCondition: "failover-failed"
      notificationLevel: "critical"
      functionalityReduction:
        maintain: ["emergency-braking", "abs"]
        reduce: ["comfort-features"]
      timeLimit: "1m"
      continuousReassessment: true
  stateTransitionValidation:
    required: true
    timeout: "5ms"
    validationSteps:
      - step: "pre-transition"
        checks: ["resource", "dependencies", "security"]
      - step: "post-transition"
        checks: ["functionality", "performance", "data-integrity"]
  vehicleStateAdjustments:
    - vehicleState: "driving"
      interval: "5ms"
      recoveryTimeout: "15ms"
      priorityModifier: 10
    - vehicleState: "parking"
      interval: "10ms"
      recoveryTimeout: "30ms"
      priorityModifier: 0
    - vehicleState: "reverse"
      interval: "5ms"
      recoveryTimeout: "15ms"
      priorityModifier: 10
  autonomyLevelAdjustments:
    - autonomyLevel: "level3"
      interval: "5ms"
      recoveryTimeout: "15ms"
      priorityModifier: 15
    - autonomyLevel: "level4"
      interval: "5ms"
      recoveryTimeout: "10ms"
      priorityModifier: 20
  loggingConfig:
    detailedStateTransitions: true
    persistenceLevel: "blackbox"
    storeDuration: "30d"
    criticalEvents:
      - "failover-trigger"
      - "recovery-failed"
  safetyMeasures:
    fallbackMode: "mechanical-backup"
    emergencyStop:
      enabled: true
      conditions: "all-recovery-failed AND vehicle-moving"
    driverNotification:
      level: "critical"
      visualAlert: true
      audioAlert: true
```

##### ASIL C 차선 유지 시스템 예시

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: lane-keeping-recovery
  annotations:
    description: "ASIL C lane keeping assist recovery policy"
    asilLevel: "C"
spec:
  target:
    package: lane-keeping-assist
    models:
      - lane-detection
      - lane-controller
  stateAssessment:
    interval: "50ms"
    healthEndpoints:
      - model: "lane-detection"
        endpoint: "/health"
        timeout: "20ms"
      - model: "lane-controller"
        endpoint: "/health"
        timeout: "20ms"
    metricThresholds:
      - name: "detectionQuality"
        threshold: "75%"
        action: "degradeMode"
      - name: "responseTime"
        threshold: "40ms"
        action: "restart"
    watchdogConfig:
      enabled: true
      interval: "50ms"
      missedThreshold: 2
  recoveryStrategies:
    - type: degradedMode
      timeout: "100ms"
      levels:
        - name: "reduced-precision"
          conditions: "detectionQuality < 75% AND detectionQuality >= 50%"
          features:
            reduced: ["detection-precision"]
          stateMapping: "Degraded"
        - name: "warning-only"
          conditions: "detectionQuality < 50%"
          features:
            disabled: ["automatic-steering"]
            reduced: ["detection-frequency"]
          stateMapping: "ReducedFunctionality"
      notificationRequired: true
    - type: restart
      timeout: "100ms"
      maxRetries: 2
      gracePeriod: "50ms"
      statePreservation: true
  stateTransitionValidation:
    required: true
    timeout: "20ms"
    validationSteps:
      - step: "pre-transition"
        checks: ["resource", "dependencies"]
      - step: "post-transition"
        checks: ["functionality", "user-notification"]
  vehicleStateAdjustments:
    - vehicleState: "driving"
      interval: "50ms"
      recoveryTimeout: "90ms"
      priorityModifier: 15
    - vehicleState: "parking"
      interval: "80ms"
      recoveryTimeout: "150ms"
      priorityModifier: -10
  autonomyLevelAdjustments:
    - autonomyLevel: "level2"
      interval: "50ms"
      recoveryTimeout: "90ms"
      priorityModifier: 15
    - autonomyLevel: "level3"
      interval: "50ms"
      recoveryTimeout: "90ms"
      priorityModifier: 20
    - autonomyLevel: "level4"
      interval: "40ms"
      recoveryTimeout: "80ms"
      priorityModifier: 25
  loggingConfig:
    detailedStateTransitions: true
    persistenceLevel: "system"
    storeDuration: "15d"
  safetyMeasures:
    fallbackMode: "driver-control"
    driverNotification:
      level: "warning"
      visualAlert: true
      audioAlert: true
    handoverStrategy:
      enabled: true
      warningTime: "3s"
```

##### ASIL B 충돌 경고 시스템 예시

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: collision-warning-recovery
  annotations:
    description: "ASIL B collision warning system recovery policy"
    asilLevel: "B"
spec:
  target:
    package: collision-warning-system
    models:
      - collision-detection
  stateAssessment:
    interval: "100ms"
    healthEndpoints:
      - model: "collision-detection"
        endpoint: "/health"
        timeout: "50ms"
    metricThresholds:
      - name: "detectionAccuracy"
        threshold: "80%"
        action: "degradeMode"
      - name: "responseTime"
        threshold: "80ms"
        action: "restart"
    watchdogConfig:
      enabled: true
      interval: "100ms"
      missedThreshold: 2
  recoveryStrategies:
    - type: restart
      timeout: "500ms"
      maxRetries: 3
      gracePeriod: "100ms"
      statePreservation: true
    - type: degradedMode
      timeout: "500ms"
      activationCondition: "detectionAccuracy < 80% OR restart-failed"
      notificationLevel: "warning"
      functionalityReduction:
        maintain: ["warnings"]
        reduce: ["automatic-braking"]
  stateTransitionValidation:
    required: true
    timeout: "50ms"
    validationSteps:
      - step: "post-transition"
        checks: ["basic-functionality", "user-notification"]
  vehicleStateAdjustments:
    - vehicleState: "driving"
      interval: "100ms"
      recoveryTimeout: "400ms"
      priorityModifier: 15
    - vehicleState: "reverse"
      interval: "100ms"
      recoveryTimeout: "400ms"
      priorityModifier: 10
  autonomyLevelAdjustments:
    - autonomyLevel: "level2"
      interval: "100ms"
      recoveryTimeout: "400ms"
      priorityModifier: 15
    - autonomyLevel: "level3"
      interval: "90ms"
      recoveryTimeout: "350ms"
      priorityModifier: 20
  loggingConfig:
    detailedStateTransitions: true
    persistenceLevel: "system"
    storeDuration: "7d"
  safetyMeasures:
    fallbackMode: "manual-operation"
    driverNotification:
      level: "warning"
      visualAlert: true
      audioAlert: true
```

##### QM 미디어 플레이어 예시

```yaml
apiVersion: piccolo.io/v1
kind: RecoveryPolicy
metadata:
  name: media-player-recovery
  annotations:
    description: "Media player recovery policy"
    asilLevel: "QM"
spec:
  target:
    package: media-playback-system
  stateAssessment:
    interval: "1s"
    basicCheck:
      enabled: true
      timeout: "500ms"
    healthEndpoints:
      - endpoint: "/health"
        timeout: "500ms"
  recoveryStrategies:
    - type: restart
      timeout: "5s"
      maxRetries: 5
      backoffSeconds: 30
      gracePeriod: "2s"
  vehicleStateAdjustments:
    - vehicleState: "driving"
      interval: "2s"
      recoveryTimeout: "8s"
      priorityModifier: -20
    - vehicleState: "parking"
      interval: "1s"
      recoveryTimeout: "4s"
      priorityModifier: 15
  autonomyLevelAdjustments:
    - autonomyLevel: "level4"
      interval: "1s"
      recoveryTimeout: "4s"
      priorityModifier: 5
    - autonomyLevel: "level5"
      interval: "1s"
      recoveryTimeout: "3s"
      priorityModifier: 15
  loggingConfig:
    detailedStateTransitions: false
    persistenceLevel: "minimal"
    storeDuration: "1d"
```

### 8.12 RecoveryPolicy 템플릿 적용 방법

RecoveryPolicy 템플릿을 개발 및 배포 과정에 적용하는 방법은 다음과 같습니다:

1. **서비스 유형 및 ASIL 등급 식별**:
   - 서비스의 기능 안전 요구사항에 따른 ASIL 등급 결정
   - 서비스 유형(액추에이터 제어, 센서 기반, 알림 등)에 따른 특화 요구사항 확인

2. **기본 템플릿 선택**:
   - 해당 ASIL 등급의 기본 템플릿 선택
   - 서비스 유형에 맞는 세부 설정 가이드 참고

3. **필수 커스터마이징 필드 설정**:
   - 서비스 관련 필드 수정 (이름, 패키지, 모델 등)
   - 서비스 특성에 맞는 메트릭 임계값 조정
   - 기능 저하 모드 및 안전 조치 정의

4. **차량 상태 및 자율주행 등급별 조정**:
   - 서비스 유형과 ASIL 등급에 따른 차량 상태별 조정값 적용
   - 자율주행 등급별 우선순위 조정 적용

5. **검증 및 적용**:
   - 모든 필드가 유효 범위 내에 있는지 확인
   - 템플릿 내용 검증 및 PolicyManager에 적용

### 8.13 템플릿 활용 시 고려사항

1. **안전성과 가용성 균형**:
   - 높은 ASIL 등급의 서비스는 안전성에 중점을 두고 짧은 감지 및 복구 시간 설정
   - 낮은 ASIL 등급의 서비스는 가용성과 사용자 경험에 중점을 두고 설정

2. **리소스 효율성**:
   - 차량 상태에 따른 모니터링 주기 조정으로 리소스 사용 최적화
   - 필요한 경우에만 복잡한 검증 단계 활성화

3. **실패 모드 분석**:
   - 서비스의 가능한 실패 모드를 분석하고 적절한 대응 전략 수립
   - 다양한 상황에 대응할 수 있는 다단계 복구 전략 설계

4. **운전자 경험 고려**:
   - 운전자 알림은 중요한 정보만 제공하여 과도한 알림 방지
   - 자율주행 레벨에 따라 운전자 개입 필요성을 고려한 설정

5. **테스트 및 검증**:
   - 다양한 시나리오에서 RecoveryPolicy의 효과 테스트
   - 극단적인 상황에서도 안전한 상태로 전환되는지 확인

### 8.14 결론

ASIL 등급별 RecoveryPolicy 템플릿을 사용하면 안전 요구사항을 일관되게 충족하면서도 각 서비스의 특성에 맞게 효율적으로 복구 정책을 구성할 수 있습니다. 특히 차량 상태와 자율주행 등급별 조정을 통해 현재 차량의 동적 상황에 최적화된 복구 전략을 제공할 수 있습니다.

템플릿은 최소한의 요구사항과 구체적인 데이터 타입 및 유효 범위를 제공하므로, 개발자는 명확한 가이드라인을 따라 안전하고 효과적인 복구 정책을 쉽게 구현할 수 있습니다. 또한 각 서비스의 특성에 맞는 세부 설정을 통해 더욱 정교한 복구 전략을 구현할 수 있습니다.

RecoveryPolicy 템플릿을 활용하면 PICCOLO 프레임워크에서 서비스의 안전성, 가용성 및 신뢰성을 체계적으로 관리할 수 있습니다. 특히 차량 상태와 자율주행 레벨에 따른 동적 조정을 통해 다양한 운행 조건에서도 최적의 서비스 성능을 보장할 수 있습니다.
