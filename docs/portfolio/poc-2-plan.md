# SOAR 패키지 2차 PoC 계획서

## 1. 문서 개요

| 항목 | 내용 |
|---|---|
| PoC | 2차 PoC — Teams 알림 및 외부 Inbound 검증 |
| 상태 | Outbound Teams E2E 부분 실행 완료 — 비동기 전달 실패 원인 추적 후 재검증 필요; Inbound·Zero-Login은 독립 트랙 |
| 선행 문서 | [1차 PoC 결과서](./poc-1-results.md) |
| 대상 환경 | `soarInstallTest` 또는 별도 초기화된 Sandbox/Developer 조직 |
| 핵심 범위 | Microsoft Teams, 외부 SIEM/EDR Inbound, Zero-Login 전제조건 |
| 제외 범위 | Production 적용, 실제 파괴적 사용자 액션의 무승인 실행 |

이 문서는 1차 PoC에서 보류한 외부 전달과 인바운드 경로를 실제 조직 설정으로 검증하기 위한 계획이다. 후속 검증은 서로 다른 두 트랙으로 분리한다. Outbound Teams는 Subscriber Flow의 `Send Security Log`로 정책 기반 알림을 발생시켜 검증하고, Inbound·Zero-Login은 공식 callback fixture로 action code/token·서명·만료·replay를 검증한다. 어느 한 트랙도 다른 트랙의 성공을 대체하지 않는다.

## 2. 목표

- `IF_Teams_Base` Named Credential을 통한 Teams 카드 전달 검증
- Teams 연결 및 Principal Access의 사전조건 통과 상태를 별도 기록
- 임계값 1·`NOTIFY_TEAMS` 정책에 고유 이벤트를 발행하는 활성 Subscriber Flow harness 검증
- `SecurityNotifyTeamsAction` 성공과 `Delivery Ledger`의 `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED` 확인
- 실제 Teams 카드 수신까지 포함한 Outbound Teams E2E 검증
- 정책 코드·심각도·우선순위 기반 Route 선택 검증
- 외부 SIEM/EDR에서 서명된 보안 신호를 Inbound로 수신하는 흐름 검증
- 만료·재전송·중복·잘못된 서명·비정상 payload의 안전한 거부 검증
- Delivery Ledger, 제한된 재시도, Fallback, 최종 실패 상태 검증
- Experience Cloud Site 기반 Zero-Login의 승인·결과·감사 경계 검증
- 1차 PoC에서 발견한 Data 채널 Dashboard 집계·필터 불일치 회귀검증

## 3. 사전조건

### 3.1 준비해야 할 설정값

아래 값은 2차 PoC에서 사용할 권장 기준값이다. `<...>` 부분은 실제 Sandbox에서 입력할 조직별 값이며, Secret·Webhook 원문·토큰은 문서에 기록하지 않는다.

| 구분 | 키/필드 | 입력값 또는 형식 | 필수 |
|---|---|---|---:|
| Package | Namespace | `soarpkg` | 예 |
| Teams | Named Credential Developer Name | `IF_Teams_Base` | 예 |
| Teams | Endpoint | `https://<approved-teams-endpoint>` | 예 |
| Slack 회귀 | Named Credential Developer Name | `IF_Slack_Base` | 선택 |
| Inbound | Custom Metadata Type | `SecurityInboundConfig__mdt` | 예 |
| Inbound | Record | `Default` | 예 |
| Inbound | `Secret__c` | `<generated-signing-secret>` | 예 |
| Inbound | `InboundBaseUrl__c` | `https://<experience-cloud-site-domain>` | 예 |
| Inbound | `IsSystemEnabled__c` | `true` | 예 |
| Inbound | `EnableWebhookSignature__c` | `true` | 예 |
| Site | Site URL | HTTPS, query/fragment 없는 URL | 예 |
| Site | Guest Permission Set | `soarpkg__SOAR_Inbound_Guest` | Zero-Login 시 |
| Site | Guest Apex 접근 | `IF_SecurityActionController` 실행 권한 검토 | Zero-Login 시 |

`IF_Teams_Base`와 `IF_Slack_Base`에는 Salesforce Named Credential의 Developer Name만 사용한다. 실제 URL과 인증 방식은 Named Credential/External Credential 설정에 보관하고 이 문서에는 적지 않는다.

### 3.2 Route 테스트 기준값

Route 저장·Preview·Delivery Ledger 비교에 사용할 고정 테스트값이다. 실제 운영 Route와 충돌하지 않는 `POC2_` 접두사를 사용한다.

| 필드 | 기준값 |
|---|---|
| Route Key | `POC2_TEAMS` |
| Profile Key | `POC2_TEAMS_DEFAULT` |
| Channel | `TEAMS` |
| Named Credential Developer Name | `IF_Teams_Base` |
| Policy Code | `EXTERNAL_THREAT_CALLBACK` |
| Action Type | `NOTIFY_TEAMS` |
| Severity | `HIGH` |
| Priority | `100` |
| Dedupe Window | `300` seconds |
| Fallback Route Key | `POC2_TEAMS_FALLBACK` 또는 미설정 상태에서 1차 확인 |
| Active | Preview/검증 전 `false`, 실제 전달 검증 시 승인 후 `true` |

### 3.3 Outbound Teams E2E 테스트값

아래 값은 Teams 연결 자체가 아니라 정책 기반 Outbound E2E를 재현하기 위한 전용 Subscriber 테스트값이다. 실제 실행 시 고유 이벤트 키를 매번 새로 생성하고, 원시 URL·Secret·토큰은 기록하지 않는다.

| 항목 | 기준값/요구사항 |
|---|---|
| 정책 코드 | 전용 Sandbox 정책 코드 `POC2_TEAMS_E2E` 권장; 기존 패키지 정책을 재사용하면 실제 Policy Code를 증적에 기록 |
| 임계값 | `1` |
| Action | `NOTIFY_TEAMS` |
| Severity | `HIGH` |
| Route/Profile | `POC2_TEAMS` / `POC2_TEAMS_DEFAULT` |
| Named Credential | `IF_Teams_Base` |
| Subscriber Flow harness | `POC2_Send_Security_Log_Harness` 권장, Screen Flow UI 실행 방식 |
| Invocable Action | `soarpkg.SecurityInvocableLogger` — `Send Security Log` |
| 고유 이벤트 marker | `POC2-OUTBOUND-<timestamp-or-uuid>`를 Event Key와 Details/Message에 삽입 |
| Idempotency 입력 | `SecurityInvocableLogger` UI 입력에는 직접적인 `idempotencyKey` 필드가 없어 별도 입력으로 가정하지 않음 |
| 테스트 사용자 | 전용 Sandbox 사용자; 민감정보 없는 대상 사용자 문맥 |
| 메시지/Details | 외부 전송용 민감정보 없는 최소 문맥 |

Flow harness는 Sandbox에서만 활성화하고, 한 번의 실행으로 하나의 고유 이벤트를 발행한다. Health Center의 테스트 이벤트, Threat Simulator, Teams 직접 POST는 이 E2E를 대신하는 증적으로 사용하지 않는다. 별도 커스텀 정책 생성 UI가 확인되지 않으면 기존 Sandbox 정책을 재사용하되, 임계값·Action 변경과 원복 여부를 결과서에 명시한다.

### 3.4 외부 Inbound 테스트 식별자

| 필드 | 기준값 |
|---|---|
| `policyCode` | `EXTERNAL_THREAT_CALLBACK` |
| `severity` | `HIGH` |
| `eventKey` | `POC2-INBOUND-001` |
| `idempotencyKey` | `POC2-IDEMP-001` |
| `targetUserId` | `<dedicated-sandbox-user-id>` |
| `recordId` | `<dedicated-test-record-id>` |
| `message` | `POC2 external inbound connectivity test` |
| `details` | 민감정보 없는 최소 JSON 문맥 |

| Fixture 항목 | 요구사항 |
|---|---|
| Fixture 출처 | 패키지 소유자 또는 공식 발신 시스템이 제공한 공식 fixture |
| action code | 공식 값만 사용; 추측·임의 생성 금지 |
| token/signature | 공식 생성 규칙과 헤더명 확인 후 사용 |
| 만료 | 공식 만료 필드/검증 규칙에 맞춘 정상·만료 fixture 각각 준비 |
| replay | 동일 event/idempotency 값 재전송 fixture 준비 |

이 Inbound fixture는 Outbound Teams 검증에 사용하지 않는다. Outbound E2E는 Subscriber Flow로 시작하고, Inbound 검증은 공식 callback으로 별도 시작한다.

실제 Inbound payload의 필드명·서명 헤더·응답 계약은 설치 버전의 공식 계약과 Describe 결과로 실행 전에 확정한다. 위 값은 재현 가능한 테스트 식별자이며 인증값이 아니다.

### Salesforce 설정

- `IF_Teams_Base` Named Credential
- 필요 시 `IF_Slack_Base`를 회귀 비교용으로 추가
- HTTPS Experience Cloud Site URL
- `SecurityInboundConfig__mdt`의 다음 설정
  - `Secret__c`
  - `InboundBaseUrl__c`
  - `IsSystemEnabled__c`
  - `EnableWebhookSignature__c`
- 외부 Inbound 전용 최소 권한 집합
- 필요 시 Guest User의 `SOAR_Inbound_Guest` 권한
- `IF_SecurityActionController` 실행 권한 검토

### 외부 테스트 자산

- Teams 전용 테스트 채널 또는 Webhook endpoint
- `Send Security Log`를 호출하는 Subscriber Flow harness와 UI 실행 화면
- 외부 SIEM/EDR에서 보낼 수 있는 공식 callback sender
- Inbound 정상·오류·재전송 payload fixture
- Inbound 서명 생성, 만료, 단일 사용, replay 검증 방법
- 장애를 모의할 수 있는 테스트 endpoint 또는 제어 가능한 응답 환경

### 안전 조건

- 테스트 대상은 전용 Sandbox 사용자와 비파괴 정책으로 제한
- 원시 Webhook URL·Secret·OAuth 토큰을 문서와 로그에 기록하지 않음
- 계정 동결·세션 종료·토큰 회수·MFA 리셋은 승인 흐름까지만 검증
- 테스트 종료 후 Site endpoint와 외부 채널을 비활성화하거나 상태를 명시

## 4. 검증 범위와 시나리오

| ID | 영역 | 시나리오 | 예상 확인 결과 |
|---|---|---|---|
| T2-01 | Teams | Health Check | `IF_Teams_Base` 연결 상태와 누락 원인이 명확히 표시됨 |
| T2-02 | Teams | 테스트 카드 POST | Teams 응답 코드, 카드 구조, 전송 시간 확인 |
| T2-03 | Routing | 정책·심각도·우선순위 Route Match | 의도한 Route/Profile과 Named Credential 선택 |
| T2-04 | Delivery | 정상 전달 | Delivery Ledger에 성공 상태와 추적 ID 기록 |
| T2-05 | Delivery | 일시적 외부 오류 | 제한된 백오프 재시도와 재시도 횟수 기록 |
| T2-06 | Delivery | 최종 실패 | Fallback Route, 관리자 알림, 최종 실패 상태 기록 |
| T2-07 | Inbound | 유효한 서명 payload | 수신→정규화→정책 평가→감사 원장 흐름 성공 |
| T2-08 | Inbound | 서명 누락·변조 | 요청 거부, 대응 액션 미실행, 실패 감사 기록 |
| T2-09 | Inbound | 만료·replay | 만료 토큰과 동일 이벤트 키 재전송 거부 |
| T2-10 | Inbound | malformed/rate limit | 비정상 payload와 과도한 요청 안전 거부 |
| T2-11 | Zero-Login | Guest 진입점 | 제한된 지점만 접근 가능하고 운영 화면은 노출되지 않음 |
| T2-12 | Zero-Login | 승인·결과 조회 | 승인 대기·승인·거절·실행 결과가 분리되어 기록됨 |
| T2-13 | Regression | Data CDC 이벤트 | Dashboard Data 카운터와 필터가 감사 로그와 일치함 |
| T2-14 | Regression | 외부 채널 실패 후 복구 | 재전송·복구 결과가 원래 탐지 이벤트와 연결됨 |

### 4.1 후속 검증 트랙 — Outbound Teams E2E

| ID | 단계 | 실행 내용 | 통과 증적 |
|---|---|---|---|
| O-01 | 정책 준비 | 임계값 `1`, Action `NOTIFY_TEAMS`, `HIGH`, `POC2_TEAMS` Route를 활성화하고 설정 화면을 캡처 | 정책·임계값·Action·Route 일치 |
| O-02 | Flow harness | `POC2_Send_Security_Log_Harness`를 활성화하고 `Send Security Log` 입력을 UI에서 준비 | 활성 Flow, 입력값, 실행 사용자 |
| O-03 | 고유 이벤트 발행 | Event Key와 Details/Message에 고유 marker를 넣어 Flow를 1회 실행; 액션 UI 입력에는 직접적인 `eventKey`·`idempotencyKey` 필드가 없음 | Flow 실행 및 고유 marker 보존 |
| O-04 | 감사 로그 | 동일 이벤트의 정책 코드·심각도·Action·source를 Dashboard/Audit에서 확인 | 감사 로그 1건 |
| O-05 | 액션 실행 | `SecurityNotifyTeamsAction`의 성공 결과를 Flow 결과/Action 실행 증적에서 확인 | Action 성공 |
| O-06 | Delivery Ledger | 새 행의 `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED`를 확인 | 세 필드 정확 일치 |
| O-07 | 외부 수신 | Teams 테스트 채널에서 동일 이벤트 카드 수신을 확인 | 사용자 스크린샷 또는 수신 증적 |

O-04~O-07 중 하나라도 빠지면 “Teams 연결 성공”은 유지하되 “정책 기반 알림 E2E 통과”로 판정하지 않는다.

### 4.1.1 현재 실행 결과

| 단계 | 현재 결과 | 판정 |
|---|---|---|
| O-01 정책 준비 | 기존 `EXTERNAL_THREAT_CALLBACK` 재사용, High 임계값 `1`, High Action은 `NOTIFY_TEAMS`만 유지, `POC2_TEAMS` Route Active 확인 | 통과; Sandbox 임시 변경 원복 결정 필요 |
| O-02 Flow harness | `POC2_Send_Security_Log_Harness` V1 저장·활성화, Screen 입력 매핑 확인 | 통과 |
| O-03 고유 이벤트 발행 | UI Flow로 고유 Event Key/Details/Message를 넣어 1회 실행 | 통과; 별도 idempotency 입력은 없음 |
| O-04 감사 로그 | Dashboard에 `HIGH / EXTERNAL_THREAT_CALLBACK / APEX` 감사 행 생성 | 통과 |
| O-05 액션 실행 | Flow 결과 화면이 없어 `SecurityNotifyTeamsAction` 성공 출력은 직접 확인하지 못함 | 미확인 |
| O-06 Delivery Ledger | 새 `POC2_TEAMS` 행이 `NOTIFY_TEAMS`, `3/3`, `EXHAUSTED`, `DELIVERY_FAILED`로 종료 | 실패; `DELIVERED` 아님 |
| O-07 외부 수신 | Teams 카드 수신 화면·스크린샷 미확인 | 미통과 |

현재는 O-01~O-04까지 확인하고 O-05~O-07에서 완주하지 못한 상태다. 직접 Route Health/POST가 `READY`였다는 사실은 유지하되, 비동기 정책 전달 실패와 동일시하지 않는다.

### 4.2 후속 검증 트랙 — Inbound·Zero-Login

| ID | 단계 | 실행 내용 | 통과 증적 |
|---|---|---|---|
| I-01 | Fixture 확보 | 공식 action code/token/signature/expiry/replay fixture를 확보 | fixture 출처·버전·필드표 |
| I-02 | 정상 callback | Zero-Login Site로 정상 fixture 전송 | HTTP 응답, 감사 로그, 정책 결과 |
| I-03 | action code/token 오류 | 누락·변조·잘못된 값을 각각 전송 | 안전 거부 및 실패 감사 |
| I-04 | 만료 | 만료된 fixture 전송 | 만료 거부 |
| I-05 | replay | 동일 event/idempotency fixture 재전송 | 중복 거부 또는 idempotent 처리 |
| I-06 | malformed/rate limit | 잘못된 body와 과도한 요청 전송 | 안전 거부, throttle 기록 |
| I-07 | Guest 경계 | 운영 화면·설정·파괴적 액션 접근을 확인 | 허용된 Inbound 지점만 접근 |

I-01~I-07은 O-01~O-07의 대체 단계가 아니다. Outbound Teams E2E가 완료되어도 Inbound fixture 없이는 Inbound·Zero-Login 정상 callback을 완료로 판정하지 않는다.

## 5. 단계별 진행 순서

### Phase A — 외부 채널 연결

1. Named Credential을 생성하고 원시 URL이 Route 레코드에 저장되지 않는지 확인한다.
2. Health Center에서 Teams 연결 상태를 확인한다.
3. 테스트 카드와 Webhook Ping을 전송한다.
4. Route Preview와 실제 Delivery Ledger를 비교한다.

### Phase A-1 — 정책 기반 Outbound Teams E2E

1. 임계값 1·`NOTIFY_TEAMS` 정책과 `POC2_TEAMS` Route를 준비한다. (완료; 기존 정책 임시 변경 여부 기록)
2. 활성 Subscriber Flow harness에서 `Send Security Log`를 호출한다. (완료)
3. 고유 이벤트의 감사 로그와 `SecurityNotifyTeamsAction` 성공을 확인한다. (감사 통과, 액션 성공 미확인)
4. 새 Delivery Ledger의 `ActionType`, `Channel`, `Status` 세 필드를 확인한다. (Ledger 생성은 확인했으나 `EXHAUSTED/DELIVERY_FAILED`)
5. Teams 카드 수신을 확인하고, 다섯 단계 증적을 하나의 이벤트 키로 묶는다. (미완료)
6. 새 로그인 세션에서 비동기 실행 주체·payload·External Credential 접근 경로를 Trace하고, 원인 확인 후에만 새 이벤트를 재실행한다.

### Phase B — 실패·복구

1. 외부 endpoint의 일시적 오류를 모의한다.
2. 제한된 재시도 횟수와 백오프 간격을 기록한다.
3. Fallback Route가 선택되는지 확인한다.
4. 최종 실패가 관리자 알림과 감사 원장에 남는지 확인한다.

### Phase C — 외부 Inbound

1. HTTPS Site와 Inbound Custom Metadata를 설정한다.
2. 공식 callback fixture의 action code/token/signature를 확인한다.
3. 유효한 fixture를 전송한다.
4. action code/token 오류·서명 변조·만료·replay·malformed payload를 각각 전송한다.
5. 정책 평가, 승인 대기, 결과 조회, 감사 원장을 상호 대조한다.

### Phase D — 회귀 및 정리

1. 1차 PoC에서 발견한 Data 채널 필터 불일치를 재현한다.
2. 수정본 또는 설정 변경 후 동일 시나리오를 재실행한다.
3. 외부 테스트 endpoint, Site, 권한, 테스트 데이터의 잔여 상태를 점검한다.

## 6. 완료 기준

- Teams 연결 및 Principal Access가 `READY`이고 직접 POST가 성공한다.
- Subscriber Flow가 임계값 1·`NOTIFY_TEAMS` 정책의 고유 이벤트를 발행한다.
- 동일 이벤트의 감사 로그와 `SecurityNotifyTeamsAction` 성공이 확인된다.
- 새 Delivery Ledger에 `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED`가 정확히 남는다.
- 실제 Teams 카드 수신까지 확인되어야 Outbound Teams 정책 E2E를 통과로 판정한다.
- 공식 callback fixture로 Inbound·Zero-Login의 action code/token·만료·replay를 별도로 검증한다.
- 외부 전송 실패가 제한된 재시도로 종료되며 무한 재시도가 발생하지 않는다.
- Fallback과 최종 실패 상태가 운영자가 재구성 가능한 형태로 기록된다.
- 유효하지 않은 서명·만료·replay payload가 대응 액션으로 이어지지 않는다.
- Zero-Login Guest는 허용된 Inbound 지점만 접근할 수 있다.
- 승인 없는 파괴적 액션이 실행되지 않는다.
- Data 채널 카운터와 필터 결과가 저장된 이벤트와 일치한다.
- 외부 URL·Secret·토큰이 로그·문서·Route에 노출되지 않는다.

## 7. 결과 기록 템플릿

| 항목 | 결과 | 증적 | 비고 |
|---|---|---|---|
| Teams 연결·Principal Access | 통과 | Health `READY`, 직접 POST 성공 | 정책 E2E와 별도 |
| Outbound Subscriber Flow | 통과 | `POC2_Send_Security_Log_Harness` V1 활성화 | Screen Flow UI 실행 가능 |
| Outbound 감사 로그·Action | 감사 로그 통과; Action 성공 미확인 | Dashboard 감사 행, Flow 결과 화면 부재 | 액션 실행 상세 필요 |
| Outbound Delivery Ledger | 생성 후 `EXHAUSTED/3/3/DELIVERY_FAILED` | 새 `POC2_TEAMS` Ledger 행 | `DELIVERED` 미달 |
| Teams 카드 수신 | 미통과 | 수신 화면·스크린샷 없음 | 최종 E2E 재실행 필요 |
| Inbound callback fixture | 미확보 | - | Outbound와 별도 |
| Retry/Fallback | 미실행 | - |  |
| 정상 Inbound | 미실행 | - |  |
| 오류 Inbound | 미실행 | - |  |
| Zero-Login | 미실행 | - |  |
| Data 채널 회귀 | 미실행 | - |  |

### 실행 중 발견한 막힘

| 단계 | 증상 | 원인 추정 | 조치 | 최종 상태 |
|---|---|---|---|---|
| 정책 Flow 실행 | 감사 로그와 Delivery Ledger는 생성됐지만 Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED` | 직접 Route POST와 비동기 `SecurityNotifyTeamsAction` 실행 컨텍스트·payload가 다를 가능성 | 비동기 실행 주체·payload·External Credential 접근을 Trace 후 재실행 | 미해결 |
| Action 성공 증적 | Flow 결과 화면이 없어 성공 반환값을 볼 수 없음 | harness에 결과 Screen을 추가하지 않음 | 결과 Screen 또는 Action 실행 상세를 추가 | 후속 필요 |
| Teams 카드 수신 | 정책 기반 실행에서 카드 수신을 확인하지 못함 | Ledger가 전달 실패로 종료 | `DELIVERED` Ledger와 사용자 수신 스크린샷을 함께 확보 | 미해결 |
| Debug Trace | 현재 사용자 Trace 저장 후에도 로그 행이 없고 후속 실행은 세션 만료 | 비동기 실행 주체 미포함 또는 세션 만료 | 새 로그인 세션에서 관련 실행 주체까지 Trace 설정 | 후속 필요 |

## 8. 참고 문서

- [1차 PoC 결과서](./poc-1-results.md)
- [설치 및 초기 활성화](../user/installation-and-setup.md)
- [일상 운영과 권한 모델](../user/operations.md)
- [Teams·Slack 알림과 Zero-Login](../extensions/notifications-and-zero-login.md)
- [외부 연동·정책·브랜딩](../extensions/external-policy-and-branding.md)
- [Subscriber 확장 가이드](../extensions/README.md)
