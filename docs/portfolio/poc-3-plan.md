# SOAR 패키지 3차 PoC 계획서

## 1. 문서 개요

| 항목 | 내용 |
|---|---|
| PoC | 3차 PoC — Subscriber 사용자 확장 및 Global 계약 검증 |
| 상태 | 계획 단계 |
| 선행 문서 | [1차 PoC 결과서](./poc-1-results.md) |
| 선행 PoC | 2차 Teams·외부 Inbound PoC 완료 후 진행 |
| 대상 환경 | Subscriber Sandbox와 별도 확장 테스트 harness |
| 핵심 범위 | Global Apex 계약, Platform Event, Flow, Invocable, 권한·사용자 확장 |
| 제외 범위 | 패키지 내부 구현 직접 호출, Production 무승인 배포 |

이 문서는 Subscriber 조직의 업무 객체·Apex·Flow·Platform Event·커스텀 액션이 SOAR 공개 계약을 통해 연결되는지 검증하기 위한 계획이다. 실제 Global 접근 수준과 파라미터는 설치 버전의 공식 계약 및 Salesforce Describe 결과로 확정하고, 내부 클래스는 직접 호출하지 않는다.

## 2. 목표

- 업무 객체와 업무 로직을 SOAR 표준 보안 이벤트로 연결
- `global`/`public` 확장 계약의 실제 접근 수준과 버전 호환성 확인
- Platform Event, Flow, Apex subscriber 간 이벤트 흐름 검증
- `SecurityInvocableLogger`의 선언형 자동화 계약 검증
- `eventKey`·`idempotencyKey` 기반 중복 방지 검증
- Subscriber 커스텀 액션·티켓·승인·내부 알림 확장 검증
- 결과 화면 Renderer 확장과 인증·승인 로직의 분리 검증
- Admin·Operator·일반 사용자·Inbound Guest 권한 경계 검증
- 실패·재시도·롤백·킬 스위치·감사 원장 연계 검증

## 3. 검증 대상 계약

| 영역 | 공개 계약/기능 | PoC에서 확인할 내용 |
|---|---|---|
| 센서 | `soarpkg.ISecuritySensor` | 업무 변경을 최소 보안 신호로 변환 |
| 센서 어댑터 | `soarpkg.SecuritySensorAdapter` | Subscriber 문맥 생성 및 표준 이벤트 발행 |
| 이벤트 문맥 | `soarpkg.SecuritySubscriberEventContext` | 사용자·리소스·정책·이벤트 키 정규화 |
| 정책 평가 | `soarpkg.Sec` | `log`, `triggerAction`, `evaluateEvent` 결과 |
| 표준 이벤트 | `soarpkg__SecurityAlert__e` | Flow·Trigger·Queueable 구독 |
| Invocable | `soarpkg.SecurityInvocableLogger` | `Send Security Log` 입력·출력·오류 분기 |
| 결과 표현 | `soarpkg.ISecurityActionHtmlRenderer` | 화면 표현 확장과 보안 로직 분리 |
| 운영 확장 | Route/Profile·정책·액션 계약 | 조직별 라우팅·dedupe·retry·fallback 연결 |
| 권한 확장 | Admin·Operator·Inbound Guest | 사용자별 조회·변경·실행 범위 |

## 4. 사전조건

### 4.1 고정 설정값과 테스트 식별자

아래 값은 3차 PoC의 기준값이다. 사용자·레코드·승인자 ID는 실제 Sandbox 값으로 치환하고, `<...>` 값은 문서나 로그에 원문을 남기지 않는다.

| 구분 | 키/필드 | 기준값 또는 형식 | 비고 |
|---|---|---|---|
| Package | Package | `SOAR_Operations_Core_Next 0.1.0.1` | 설치 버전 기록 |
| Package | Namespace | `soarpkg` | API/계약 호출 기준 |
| Event | Platform Event | `soarpkg__SecurityAlert__e` | Flow/Trigger 구독 대상 |
| Flow | Invocable Action | `Send Security Log` | `SecurityInvocableLogger` |
| Flow | 테스트 Flow 이름 | `POC3_Send_Security_Log` | 권장 이름 |
| Event subscriber | 테스트 Flow/Trigger 이름 | `POC3_SecurityAlert_Consumer` | 권장 이름 |
| Test user | Admin | `<poc3-admin-user>` + `soarpkg__SOAR_Admin` | 운영 설정 검증 |
| Test user | Operator | `<poc3-operator-user>` + `soarpkg__SOAR_Operator` | 운영 제한 검증 |
| Test user | 일반 사용자 | `<poc3-standard-user>` + SOAR 권한 없음 | 접근 거부 검증 |
| Test user | Inbound Guest | `<poc3-guest-user>` + `soarpkg__SOAR_Inbound_Guest` | Zero-Login 연계 시 |

### 4.2 표준 이벤트·Invocable 입력값

| 입력/필드 | 기준값 또는 형식 |
|---|---|
| `policyCode` | `POC3_CUSTOM_EVENT` |
| `severity` | `INFO` 정상 경로, `HIGH` 승인 경로 |
| `message` | `POC3 subscriber contract test` |
| `details` | 민감정보 없는 최소 JSON 문맥 |
| `recordId` | `<dedicated-test-record-id>` |
| `targetUserId` | `<dedicated-sandbox-user-id>` |
| `eventKey` | `POC3-EVENT-001` |
| `idempotencyKey` | `POC3-IDEMP-001` |
| `ActionName__c` | 테스트 정책에 매핑된 비파괴 액션부터 시작 |
| `ResourceType__c` | `<test-object-api-name>` |
| `ResourceName__c` | `<test-record-label>` |

Platform Event의 표준 필드는 `EventKey__c`, `PolicyCode__c`, `ActionName__c`, `Severity__c`, `UserId__c`, `UserName__c`, `ResourceId__c`, `ResourceName__c`, `ResourceType__c`, `Payload__c`를 기준으로 확인한다. 실제 필드 접근 수준과 API 버전은 Describe 결과로 확정한다.

### 4.3 계약 확인용 Describe 결과

실행 전에 다음 항목을 표로 캡처한다.

| 대상 | 확인할 값 |
|---|---|
| `soarpkg.ISecuritySensor` | 접근 수준, 메서드, 파라미터 |
| `soarpkg.SecuritySensorAdapter` | `buildSubscriberContext`, `publishSubscriberAlert` 계약 |
| `soarpkg.SecuritySubscriberEventContext` | 문맥 필드와 정규화 규칙 |
| `soarpkg.Sec` | `log`, `triggerAction`, `evaluateEvent` 계약 |
| `soarpkg.SecurityInvocableLogger` | Invocable 입력·출력·오류 구조 |
| `soarpkg__SecurityAlert__e` | 이벤트 필드, publish/subscribe 조건 |
| `soarpkg.ISecurityActionHtmlRenderer` | 렌더링 메서드와 결과 계약 |

### Subscriber 테스트 harness

- 테스트용 업무 객체 또는 최소한의 테스트 레코드
- Apex Trigger/Queueable 또는 Platform Event subscriber 샘플
- Screen Flow 또는 Record-Triggered Flow 샘플
- `Send Security Log` Invocable Action을 호출하는 Flow
- 테스트 결과를 저장할 내부 티켓·승인·알림 mock
- 중복 요청과 재시도를 재현할 수 있는 고정 `recordId`/업무 요청 ID

### 권한과 역할

- SOAR Admin 테스트 사용자
- SOAR Operator 테스트 사용자
- SOAR 권한이 없는 일반 사용자
- 필요 시 제한된 Inbound Guest 사용자
- 요청자와 승인자를 분리한 테스트 승인자

### 계약·품질 기준

- 설치 패키지 버전과 namespace 기록
- Apex/Platform Event Describe 결과 보관
- 동기·비동기·governor limit 영향 검토
- 승인·중복·롤백·재시도·비상 중지 기준 사전 정의
- 민감정보가 이벤트 문맥·Flow 입력·로그에 포함되지 않는지 검토

## 5. 검증 시나리오

| ID | 영역 | 시나리오 | 예상 확인 결과 |
|---|---|---|---|
| T3-01 | Sensor | 업무 객체 변경→`ISecuritySensor` | 최소 문맥의 표준 보안 이벤트 생성 |
| T3-02 | Adapter | `SecuritySensorAdapter` 문맥 생성 | policy/action/user/resource/event key 정규화 |
| T3-03 | Sec | `Sec.log` 및 `evaluateEvent` | 정책 평가와 감사 기록 연결 |
| T3-04 | Sec | `Sec.triggerAction` | 승인·정책·대상 검증 경계 확인 |
| T3-05 | Event | `SecurityAlert__e` subscriber | Flow/Trigger/Queueable 후속 처리 성공 |
| T3-06 | Flow | `Send Security Log` 정상 입력 | `isProcessed`, `policyCode`, `statusMessage` 확인 |
| T3-07 | Flow | 잘못된 severity·policy·record 입력 | 안전한 오류 반환과 감사 기록 |
| T3-08 | Idempotency | 동일 요청 ID 반복 실행 | 중복 티켓·알림·액션이 생성되지 않음 |
| T3-09 | Async | Subscriber 후속 처리 실패 | 패키지 탐지 결과와 업무 실패가 분리됨 |
| T3-10 | Retry | 재시도 가능한 후속 업무 | 제한된 재시도와 최종 실패 상태 확인 |
| T3-11 | Action | 커스텀 승인·티켓·내부 알림 | Subscriber 후속 조치만 수행하고 원장 추적 유지 |
| T3-12 | Renderer | `ISecurityActionHtmlRenderer` 확장 | 결과 표현만 변경되고 인증·승인 로직은 불변 |
| T3-13 | Permission | Admin/Operator/일반 사용자/Guest | 문서화된 최소 권한과 접근 경계 일치 |
| T3-14 | Rollback | 승인 거절·실패·킬 스위치 | 실행·거절·실패·롤백 상태가 감사 원장에 기록 |
| T3-15 | Compatibility | namespace·버전·라이선스 조합 | 설치 버전 계약과 실제 Describe 결과 일치 |

## 6. 단계별 진행 순서

### Phase A — 계약과 Describe 확정

1. 설치 버전의 공개 Global 클래스·인터페이스·Platform Event를 Describe한다.
2. 문서의 계약 목록과 실제 접근 수준·파라미터를 비교한다.
3. 테스트 harness에서 사용할 최소 문맥과 권한을 확정한다.

### Phase B — 이벤트·Flow 연결

1. 업무 객체 변경을 테스트 센서로 변환한다.
2. 표준 보안 이벤트를 발행한다.
3. Flow·Trigger·Queueable에서 읽기 전용으로 처리한다.
4. Invocable 결과를 성공·실패 분기로 연결한다.

### Phase C — 사용자 확장과 안전 경계

1. Subscriber 커스텀 액션을 승인 요청 또는 비파괴 내부 알림으로 연결한다.
2. 동일 요청 재처리와 제한된 재시도를 실행한다.
3. 실패·거절·롤백·킬 스위치 결과를 감사 원장과 비교한다.
4. 결과 Renderer 확장이 운영 로직에 영향을 주지 않는지 확인한다.

### Phase D — 권한·호환성 회귀

1. Admin, Operator, 일반 사용자, Guest별 화면·API·Flow 접근을 비교한다.
2. namespace와 패키지 버전이 바뀌어도 공개 계약 호출이 유지되는지 확인한다.
3. 테스트 데이터·Flow·권한·Subscriber 코드의 정리 상태를 점검한다.

## 7. 완료 기준

- Subscriber는 패키지 내부 객체를 직접 수정하지 않고 공개 계약만 사용한다.
- 표준 이벤트 문맥에 필요한 최소 정보만 포함된다.
- `eventKey`·`idempotencyKey`로 반복 처리 중복이 통제된다.
- Invocable·Flow·Apex 후속 처리가 성공·실패를 분리해 처리한다.
- 파괴적 액션은 승인·권한·감사 조건을 통과해야 한다.
- 재시도·롤백·킬 스위치·최종 실패가 감사 원장에 남는다.
- 일반 사용자와 Guest는 운영 설정·파괴적 액션에 접근하지 못한다.
- 결과 화면 확장은 표현 계층에만 영향을 준다.
- 설치 버전의 namespace·접근 수준·선택 라이선스 조건이 문서와 일치한다.

## 8. 결과 기록 템플릿

| 항목 | 결과 | 증적 | 비고 |
|---|---|---|---|
| 계약 Describe | 미실행 | - |  |
| Sensor/Adapter | 미실행 | - |  |
| `Sec` 평가 | 미실행 | - |  |
| Platform Event subscriber | 미실행 | - |  |
| Invocable Flow | 미실행 | - |  |
| Idempotency/retry | 미실행 | - |  |
| Custom action/approval | 미실행 | - |  |
| Renderer 확장 | 미실행 | - |  |
| 권한 매트릭스 | 미실행 | - |  |
| Rollback/kill switch | 미실행 | - |  |
| 호환성 회귀 | 미실행 | - |  |

### 실행 중 발견한 막힘

| 단계 | 증상 | 원인 추정 | 조치 | 최종 상태 |
|---|---|---|---|---|
| - | - | - | - | 미작성 |

## 9. 참고 문서

- [1차 PoC 결과서](./poc-1-results.md)
- [Subscriber 확장 가이드](../extensions/README.md)
- [플랫폼 이벤트·Flow·커스텀 액션](../extensions/events-flows-and-actions.md)
- [센서·비즈니스 로직](../extensions/sensors-and-business-logic.md)
- [외부 연동·정책·브랜딩](../extensions/external-policy-and-branding.md)
- [일상 운영과 권한 모델](../user/operations.md)
