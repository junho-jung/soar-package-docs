# SOAR 패키지 1차 PoC 결과서

## 1. 문서 개요

| 항목 | 내용 |
|---|---|
| PoC | 1차 PoC — 패키지 설치 및 핵심 운영 기능 검증 |
| 문서 성격 | 역사 기록 — 2026-08-23 `0.1.0.1` 실행 스냅샷; 현재 재실행 판정에 직접 사용하지 않음 |
| 검증일 | 2026-08-23 |
| 대상 환경 | 새로 생성한 Salesforce Scratch Org `soarInstallTest` |
| 패키지 | `SOAR_Operations_Core_Next 0.1.0.1` |
| Namespace | `soarpkg` |
| 기준 문서 | README, 설치·운영 매뉴얼, 확장 가이드, 검증 포트폴리오 |
| 상세 실행 로그 | [`sandbox-install-test-log.md`](../../sandbox-install-test-log.md) |

이 문서는 공개 문서에 정의된 설치·운영·확장 범위를 기준으로 1차 샌드박스 PoC를 평가한 결과서다. 패키지 내부 소스나 인증정보, 특정 조직 식별정보는 포함하지 않는다.

## 2. 종합 결론

### 판정: 조건부 통과

패키지 설치, 초기 활성화, 핵심 운영 화면, 정책·감사·스케줄·시뮬레이터·승인 흐름은 샌드박스에서 실제 실행되어 사용 가능성을 확인했다.

다만 Teams/Slack 외부 전달과 Zero-Login 인바운드는 조직별 Named Credential과 Experience Cloud Site가 필요하므로 1차 PoC에서 완전 검증하지 않았다. 또한 Data 채널 이벤트가 저장되었음에도 Dashboard의 Data 채널 집계·필터에서 누락되는 불일치를 발견했다. 이 항목은 2차 PoC 전에 원인 확인이 필요하다.

| 품질 게이트 | 결과 | 판단 |
|---|---|---|
| 신규 Subscriber 환경 설치 | 설치 성공 및 패키지 메타데이터 확인 | 통과 |
| 초기 활성화 | 1-Click 인프라 설정, 정책·스케줄 활성화 | 통과 |
| 핵심 운영 UI | Dashboard, Simulator, Policy Builder, Health Center 렌더링 | 통과 |
| 감사·리포트 | 검색, 필터, 승인 요청, CSV, 주간·월간 PDF | 통과 |
| 정책·스케줄 운영 | Drift, STRICT 프리셋, 허들 저장, 4개 표준 스케줄 | 통과 |
| 외부 Teams/Slack 전달 | Named Credential 부재로 실패 조건 확인 | 보류 |
| Zero-Login/외부 Inbound | Site와 HTTPS Inbound Base URL 부재 | 보류 |
| Data 채널 Dashboard 필터 | 이벤트 저장 성공, 요약·필터 매핑 불일치 | 개선 필요 |

## 3. 검증 환경과 방법

1. 기존 인증 불안정 테스트 Scratch Org를 삭제했다.
2. `JUNHO-STUDY` Dev Hub에서 namespace 없는 새 Scratch Org를 동일 alias `soarInstallTest`로 생성했다.
3. README에 기재된 Subscriber Package Version을 설치했다.
4. `soarpkg__SOAR_Admin` 권한 집합을 할당했다.
5. 화면 버튼과 Salesforce Setup 화면을 통해 설치 후 초기화·운영·실패 경로를 실행했다.
6. 외부 URL, Webhook 원문, 토큰, 메일 발송, 파괴적 사용자 액션은 임의로 만들거나 실행하지 않았다.

## 4. 1차 PoC 실행 결과

### 4.1 설치 및 기본 메타데이터

- 패키지 설치 성공: `SOAR_Operations_Core_Next 0.1.0.1`
- Namespace `soarpkg` 확인
- namespaced Apex 클래스 129개 확인
- namespaced LWC bundle 7개 확인
- SOAR 권한 집합 3개 확인
- 관리자 권한 집합 할당 후 Hub 접근 성공
- 잘못된 `/lightning/page/...` 경로는 `페이지 없음`이었으며, 실제 탭 경로 `/lightning/n/soarpkg__SOAR_Dashboard`로 우회했다.

### 4.2 Setup & Health Center

- `1-Click 미완료 인프라 자동 활성화` 성공
- 서명 토큰 상태 정상
- 보안 정책 12개, 대응 조치 9개 생성·노출
- 표준 스케줄 4개가 모두 `정상 가동 중 (Active)` 상태
- 동일 스케줄 1-Click을 재실행해도 4개 Active 상태가 유지되어 중복 등록 방지 동작을 확인했다.
- Inbound Base URL은 `CONFIGURATION_MISSING` 상태로 남았다.
- Teams/Slack Named Credential은 미설정 상태로 남았다.

### 4.3 Dashboard·Audit

다음 기능을 실행했다.

- 실시간 스트리밍 일시정지·재개
- Fullscreen SOC Mode 진입·일반 뷰 복귀
- 관제 데이터 새로고침
- 정책 코드·사용자·리소스 검색
- 보안 채널·위험도 필터
- 감사 로그 행의 승인 기반 대응 메뉴
- CSV 다운로드
- 주간 감사 PDF 생성
- 월간 컴플라이언스 PDF 생성

`세션 종료` 승인 요청을 1건 생성해 영향이 큰 액션이 바로 실행되지 않고 승인 흐름으로 분리되는 것을 확인했다. 계정 동결·세션 종료·토큰 회수·MFA 리셋 자체는 실행하지 않았다.

### 4.4 Threat Simulator

5개 보안 채널 시나리오를 각각 실행했고 모두 모의 발송 성공을 확인했다.

| 채널 | 대표 시나리오 | 결과 |
|---|---|---|
| Platform TSP | `MASS_DATA_EXPORT` | 성공 |
| Data CDC | `MASS_DATA_DELETION` | 성공 |
| Logic Flow/Apex | `OFF_HOURS_DATA_MUTATION` | 성공 |
| Identity & Audit | `LOGIN_BRUTE_FORCE_BURST` | 성공 |
| External Signal | `EXTERNAL_THREAT_CALLBACK` | 성공 |

추가로 다음 Simulator 기능을 실행했다.

- 4대 엔트리포인트: Trigger, Sec Facade API, Inbound REST API, LWC Event Bus
- Teams Webhook 테스트
- 수동 액션 콘솔의 액션 목록 확인
- 실시간 Webhook Ping 테스트

4대 엔트리포인트는 모두 성공했다. Teams/Slack Ping은 Named Credential 부재에 따른 실패 상태를 확인했다.

### 4.5 Policy Pipeline Builder

- 12개 위협 시그널 레지스트리 확인
- 12개 보안 정책과 9개 대응 조치 확인
- 정책 키워드 검색·채널 필터·미할당 시그널 필터 확인
- 정책 허들 정밀 수정 화면 확인
- `BALANCED`, `AUDIT_ONLY` Drift 미리보기 확인
- `STRICT` 프리셋 일괄 적용 성공
- 허들 값 변경 후 일괄 저장 성공
- 잘못된 임계치 순서 입력 시 `positive and strictly increasing` 검증 오류 확인
- 테스트한 허들 값은 원래 값으로 복원

### 4.6 Notification Route와 Flow 템플릿

- 기본 `DEFAULT_TEAMS`, `DEFAULT_SLACK` Route/Profile 노출 확인
- 빈 Route Key에서 `ROUTE_INVALID` 반환
- 유효한 테스트 key에서 `ROUTE_VALID_PENDING_NC` 반환
- Named Credential 미설정으로 Delivery Ledger가 비어 있는 상태 확인
- Salesforce Flow 목록에서 다음 관리 패키지 템플릿 확인
  - `[SOAR 템플릿] VIP 데이터 접근 이상 알림 플로우`
  - `[SOAR 템플릿] 보안 조치 승인 에스컬레이션 플로우`

## 5. 발견 사항과 위험도

### 5.1 개선 필요: Data 채널 집계·필터 불일치

Data CDC 시뮬레이션 이벤트 자체는 감사 로그에 저장되었다. 그러나 Dashboard 채널 요약은 Data를 0건으로 표시했고, Data 채널 필터를 선택하면 해당 감사 행이 보이지 않았다.

현재 판단은 다음과 같다.

- 이벤트 생성·저장: 성공
- 정책 코드 검색: 성공
- 채널 요약 카운터: 불일치
- 채널 필터 조회: 불일치

이는 외부 설정 문제가 아닌 Dashboard의 이벤트 채널 매핑 또는 필터 조건 문제일 가능성이 있으므로 2차 PoC 시작 전에 재현·수정·회귀검증이 필요하다.

### 5.2 외부 연동 미검증은 사전조건 부족에 따른 보류

Teams/Slack 실패는 패키지 내부 오류로 단정할 수 없다. Health Center와 Callout 테스트가 모두 `IF_Teams_Base`, `IF_Slack_Base` Named Credential 부재를 명시했다. 실제 URL과 인증값을 임의로 만들지 않았으므로 외부 전달 성공·재시도·Fallback·Delivery Ledger는 2차 PoC로 이관한다.

### 5.3 파괴적 대응은 안전 경계에 따라 보류

수동 액션 콘솔에는 `FORCE_MFA_RESET`, `FREEZE_USER`, `KILL_SESSION`, `REVOKE_TOKEN`이 제공된다. 1차 PoC에서는 승인 요청 생성까지만 검증하고, 실제 사용자 영향 액션은 전용 테스트 사용자와 요청자·승인자 분리 조건이 준비될 때까지 실행하지 않는다.

## 6. 1차 PoC 판정

### 통과한 범위

- Sandbox/Developer 계열 환경에 설치 가능
- 관리자 초기화와 운영 화면 사용 가능
- 정책·감사·스케줄·시뮬레이션·승인 흐름 사용 가능
- Subscriber 확장에 필요한 이벤트·Flow·Apex 계약이 문서화되어 있고 Flow 템플릿이 설치됨
- 외부 설정 누락을 Health Center와 실패 메시지로 식별 가능

### 다음 단계 전 조건

- Data 채널 집계·필터 불일치 원인 확인
- Teams Named Credential과 테스트 endpoint 준비
- Experience Cloud Site 및 HTTPS Inbound Base URL 준비
- 외부 테스트 payload, 서명, replay/expiry 테스트 fixture 준비
- 전용 테스트 사용자와 이중 승인자 준비

## 7. 2차 PoC 계획 — Teams·외부 Inbound

### 목표

Teams 외부 알림과 외부 SIEM/EDR 인바운드가 실제 조직 설정에서 탐지·정책 평가·알림·승인·감사 흐름으로 연결되는지 검증한다.

### 사전조건

- Salesforce Named Credential `IF_Teams_Base`
- Teams 테스트 채널 또는 전용 Webhook endpoint
- HTTPS Experience Cloud Site URL
- `SecurityInboundConfig__mdt`의 `Secret__c`, `InboundBaseUrl__c`, `IsSystemEnabled__c` 설정
- 필요 시 Guest User의 `SOAR_Inbound_Guest` 권한과 `IF_SecurityActionController` 접근 권한
- 외부 발신 시스템의 서명·만료·이벤트 키 규칙
- 외부 테스트용 비파괴 정책과 전용 사용자

### 검증 시나리오

| ID | 시나리오 | 확인 내용 |
|---|---|---|
| T2-01 | Teams Health/POST/Ping | HTTP 응답, 카드 payload, 왕복 시간, 감사 상태 |
| T2-02 | Teams Route Match | 정책 코드·심각도·우선순위별 Route 선택 |
| T2-03 | Teams 전달 성공 | Delivery Ledger 성공 상태와 감사 추적 ID |
| T2-04 | 일시적 Teams 장애 | 제한된 백오프 재시도와 최종 상태 |
| T2-05 | 최종 실패·Fallback | Fallback Route와 관리자 알림, 원인 기록 |
| T2-06 | 유효한 외부 Inbound | 서명 검증, 정책 평가, 감사 로그, 결과 반환 |
| T2-07 | 잘못된 서명 | 거부, 감사 기록, 대응 액션 미실행 |
| T2-08 | 만료·재전송·중복 이벤트 | expiry/replay/idempotency 처리 |
| T2-09 | malformed/rate limit | 비정상 payload와 과도한 요청의 안전한 거부 |
| T2-10 | Zero-Login 승인 흐름 | 조회·승인·실행·결과·감사 원장 상태 |
| T2-11 | Data 채널 회귀 | Data 카운터와 필터가 저장 이벤트와 일치하는지 확인 |

### 완료 기준

- Teams 테스트 전달이 성공하고 원시 Webhook URL이 로그·Route에 저장되지 않는다.
- 실패 시 재시도 횟수와 최종 실패가 Delivery Ledger에 남는다.
- 유효하지 않은 서명·만료·재전송 이벤트가 실행되지 않는다.
- 외부 Inbound 결과가 정책 코드·심각도·이벤트 키와 함께 감사 원장에 남는다.
- Zero-Login은 승인 없는 파괴적 액션을 실행하지 않는다.
- Data 채널 집계·필터 불일치가 해소되거나 명확한 제한사항으로 문서화된다.

## 8. 3차 PoC 계획 — Subscriber 사용자 확장·Global 계약

### 목표

Subscriber 조직의 업무 객체·Apex·Flow·Platform Event·커스텀 액션이 SOAR의 공개 확장 계약을 통해 연결되는지 검증한다. 여기서 Global 기능은 패키지 내부 구현을 직접 호출하는 것이 아니라, 문서에 공개된 Subscriber 계약과 설치 버전의 Describe 결과를 기준으로 평가한다.

### 검증 대상

| 영역 | 계약/기능 | 검증 방향 |
|---|---|---|
| 센서 | `soarpkg.ISecuritySensor`, `SecuritySensorAdapter` | 업무 객체 변경을 최소 보안 문맥으로 변환 |
| 문맥 | `SecuritySubscriberEventContext` | `eventKey`, `idempotencyKey`, 사용자·리소스 문맥 정규화 |
| 정책 평가 | `soarpkg.Sec` | `log`, `triggerAction`, `evaluateEvent` 호출 결과 |
| 표준 이벤트 | `soarpkg__SecurityAlert__e` | Flow·Trigger·Queueable 구독과 읽기 전용 처리 |
| 선언형 확장 | `SecurityInvocableLogger` / `Send Security Log` | 입력, `isProcessed`, `statusMessage`, 오류 분기 |
| 업무 후속 처리 | Subscriber Flow/Apex/Queueable | 티켓·승인·내부 알림 연결과 실패 분리 |
| 결과 화면 | `ISecurityActionHtmlRenderer` | 조직별 결과 표현 확장과 보안 경계 |
| 권한 모델 | Admin/Operator/Inbound Guest | 최소 권한·운영자 격리·게스트 제한 |
| 운영 계약 | Route, dedupe, retry, rollback, audit | 중복·실패·복구·감사 상태 보존 |

### 사전조건

- Subscriber 전용 테스트 코드 또는 Flow harness
- 전용 테스트 객체·레코드와 비파괴 이벤트 데이터
- Admin, Operator, 일반 사용자, Inbound Guest 역할별 테스트 계정
- 승인자와 요청자를 분리한 테스트 승인 흐름
- 재시도·중복·롤백을 확인할 수 있는 외부 또는 내부 mock endpoint
- 설치 버전의 Apex/Platform Event Describe 결과와 계약 버전 기록

### 검증 시나리오

| ID | 시나리오 | 완료 기준 |
|---|---|---|
| T3-01 | 업무 객체 센서 어댑터 | 최소 문맥으로 표준 이벤트가 발행됨 |
| T3-02 | `Sec` 직접 평가 | 정책 코드·심각도·대상 리소스가 일관되게 기록됨 |
| T3-03 | Platform Event subscriber | Flow/Trigger가 이벤트를 읽고 업무 후속 처리만 수행함 |
| T3-04 | Invocable Flow | `Send Security Log` 결과를 성공·실패 분기로 사용함 |
| T3-05 | idempotency/retry | 같은 요청 ID 재처리 시 중복 티켓·알림이 생성되지 않음 |
| T3-06 | 커스텀 액션 | 승인·권한·롤백 경계를 통과한 후 후속 액션 실행 |
| T3-07 | 결과 Renderer | 결과 표현만 확장하고 인증·승인·대응 로직은 침범하지 않음 |
| T3-08 | 권한 매트릭스 | Admin/Operator/일반 사용자/Guest 접근이 문서와 일치함 |
| T3-09 | 실패·복구 | 비동기 실패, 재시도, 롤백, 킬 스위치가 감사 원장에 남음 |
| T3-10 | 계약 호환성 | namespace·패키지 버전·선택 라이선스 조건을 확인함 |

### 완료 기준

- Subscriber는 패키지 내부 객체를 직접 수정하지 않고 공개 이벤트·계약만 사용한다.
- 반복 이벤트가 중복 업무 처리로 확장되지 않는다.
- 파괴적 액션은 승인·권한·감사 조건을 모두 만족해야 한다.
- 외부 전송 실패와 업무 트랜잭션 실패가 구분된다.
- 재시도·롤백·최종 실패·승인 대기 상태가 감사 원장과 후속 Flow에 전달된다.
- 일반 사용자와 Guest는 운영 설정·파괴적 액션에 접근할 수 없다.

## 9. 권고사항

1. 2차 PoC 착수 전 Data 채널 집계·필터 불일치를 제품 이슈로 재현하고 수정 여부를 결정한다.
2. Teams Named Credential과 HTTPS Experience Cloud Site를 별도 테스트 자격증명으로 준비한다.
3. 외부 Inbound는 정상·오류·재전송·만료 payload fixture를 먼저 확정한다.
4. 3차 PoC는 별도 Subscriber 테스트 harness를 만들어 Global 계약과 권한 매트릭스를 자동 검증한다.
5. Production 적용은 2차·3차 PoC 결과와 운영 승인 이후 별도 단계로 둔다.

## 10. 다음 PoC 설정값 요약

| PoC | 반드시 준비할 값 |
|---|---|
| 2차 | `IF_Teams_Base`, 선택적 `IF_Slack_Base`, `SecurityInboundConfig__mdt.Default`의 `Secret__c`·`InboundBaseUrl__c`·`IsSystemEnabled__c`·`EnableWebhookSignature__c`, HTTPS Experience Cloud Site, `soarpkg__SOAR_Inbound_Guest` |
| 2차 테스트 Route | `POC2_TEAMS`, `POC2_TEAMS_DEFAULT`, `TEAMS`, `IF_Teams_Base`, `NOTIFY_TEAMS`, `EXTERNAL_THREAT_CALLBACK`, `HIGH`, dedupe `300`초 |
| 3차 | namespace `soarpkg`, `soarpkg__SecurityAlert__e`, `Send Security Log`, 전용 Admin/Operator/일반 사용자 테스트 계정 |
| 3차 테스트 식별자 | `POC3_CUSTOM_EVENT`, `POC3-EVENT-001`, `POC3-IDEMP-001`, 전용 test record/user ID |

실제 Secret, Webhook URL, OAuth 토큰, 사용자 ID와 Org 식별자는 계획 문서에 기록하지 않고 각 PoC 실행 환경에만 입력한다. 상세 입력 형식은 [2차 PoC 계획서](./poc-2-plan.md)와 [3차 PoC 계획서](./poc-3-plan.md)에 정의했다.

## 11. 참고 문서

- [패키지 README](../../README.md)
- [설치 및 초기 활성화](../user/installation-and-setup.md)
- [일상 운영과 권한 모델](../user/operations.md)
- [Subscriber 확장 가이드](../extensions/README.md)
- [플랫폼 이벤트·Flow·커스텀 액션](../extensions/events-flows-and-actions.md)
- [Teams·Slack 알림과 Zero-Login](../extensions/notifications-and-zero-login.md)
- [검증 결과와 품질 기준](./validation.md)
