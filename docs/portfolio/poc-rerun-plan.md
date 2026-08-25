# SOAR 패키지 PoC 재실행 계획서

## 1. 문서 성격과 기준

| 항목 | 내용 |
|---|---|
| 문서 상태 | 실행 종료 기준 계획서·판정표 |
| 작성일 | 2026-08-26 |
| 기준 패키지 | `SOAR_Operations_Core_Next 0.1.0.5` |
| Subscriber Package Version ID | `04tdM000000c9OrQAI` |
| Namespace | `soarpkg` |
| 권장 대상 | 새로 선정한 Subscriber Sandbox/Developer/Trial 조직 |
| 새 PoC alias | `soarInstallTest` — 기존 오그를 삭제한 뒤 2026-08-26 새로 생성 |
| 선행 자료 | [README](../../README.md), [설치 후 전체 사용 런북](../user/end-to-end-runbook.md), [검증 기준](./validation.md) |

이 문서는 기존 1~3차 PoC를 덮어쓰지 않고, 현재 공개 문서와 현재 기준 패키지로 PoC를 다시 시작하기 위한 실행 흐름이다. 기존 `poc-1-results.md`, `poc-2-plan.md`, `poc-2-results.md`, `poc-3-plan.md`, `sandbox-install-test-log.md`는 2026-08-23의 `0.1.0.1` 실행 스냅샷으로 보존한다.

현재 실행의 판정은 과거 결과를 승계하지 않는다. 모든 통과 증적은 새 대상 조직, 새 설치 버전, 새 실행 식별자로 다시 확보한다.

### 현재 실행 종료 메모

2026-08-26 후속 실행에서 G0~G3와 핵심 G2 회귀를 판정했다. Teams Health/직접 POST는 성공했지만, 실제 정책 Flow 이벤트는 Audit 이후 Delivery Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`로 종료했고 `DELIVERED` 및 Teams 카드 수신을 확보하지 못했다. 따라서 G4는 실패로 종료한다. 공식 Inbound fixture와 Subscriber Global Describe가 없는 상태에서 G5·G6을 통과로 추정하지 않고 보류·미실행으로 종료한다. 세부 증적은 [재실행 결과서](./poc-rerun-results.md), 원인 분류는 [PoC 피드백](./poc-feedback.md)에 기록한다.

## 2. 문서 우선순위

재실행 중 문서 간 값이나 표현이 다르면 다음 순서로 판단한다.

1. 현재 [README](../../README.md)와 사용자 설치·운영 런북
2. 현재 설치 버전의 Salesforce Setup/Describe 결과
3. 확장 계약과 검증 기준 문서
4. 과거 PoC 결과·실행 로그

과거 PoC 문서는 실패 원인과 위험 통제를 참고하는 자료이지, 현재 오그의 상태나 현재 패키지 기능을 보증하는 자료가 아니다. 특히 과거 `0.1.0.1`의 `soarInstallTest` 결과와 현재 문서의 `0.1.0.5` UAT 기준을 섞지 않는다.

### 로그인 차단 시 CLI 전환 규칙

브라우저에서 대상 Subscriber 오그 로그인이 막히거나 세션이 만료되면, 브라우저 로그인 재시도만 반복하지 않고 CLI에 저장된 동일 대상 alias 인증으로 전환한다.

1. `sf org list --all --json --skip-connection-status`로 대상 alias가 활성 인증 상태인지 확인한다.
2. `sf org open --target-org soarInstallTest --path /lightning/n/soarpkg__SOAR_Dashboard`로 CLI 인증 기반 Salesforce 진입점을 Chrome에 연다.
3. CLI 인증이 유효하면 패키지 설치·권한·Installed Packages·Describe·데이터 조회를 CLI로 계속 진행한다.
4. UI에서만 가능한 Setup/Health/Flow 조작은 CLI가 연 인증 탭에서 상태를 확인한 뒤 수행한다.
5. CLI 인증도 실패하면 원인과 명령 결과를 기록하고, 새 인증 없이는 임의 우회하지 않는다.

CLI 출력·frontdoor URL·access token·refresh token은 사용자 메시지·문서·실행 로그에 복사하지 않는다. 이 규칙은 브라우저 로그인을 우회해 권한을 확대하는 절차가 아니라, 사용자가 이미 연결한 대상 오그의 CLI 인증을 같은 조직 UI 진입에 사용하는 복구 절차다.

## 3. 오그 선정과 역할 분리

### 3.1 착수 전에 확인할 오그 분류

| 역할 | 예시 | 재실행에서의 사용 |
|---|---|---|
| Dev Hub | `JUNHO-STUDY` | 새 Subscriber 테스트 오그 생성에만 사용 |
| Provider/namespace 조직 | `SOAR-PKG` | 패키지 제공·namespace 기준 확인; Subscriber PoC 대상 아님 |
| 기존 UAT 또는 패키지 관련 조직 | `PackageSoarOrg`, `soarPkgInstall` 등 | 현재 로그인·패키지 버전을 확인하기 전에는 대상 확정 금지 |
| 새 PoC Subscriber | `soarInstallTest` | 기존 오그 삭제 후 새로 생성한 패키지 설치·전체 재실행 대상 |

실제 착수 시 `sf org list`와 Salesforce Setup의 Installed Packages를 함께 확인한다. Dev Hub나 namespace 조직을 Subscriber PoC 대상으로 사용하지 않는다. 기존 조직을 재사용해야 한다면 설치 버전·정책·Route·Flow·Site·권한·외부 자격 증명을 먼저 스냅샷하고, 새 조직과 동일한 초기화 기준을 만족할 때만 사용한다.

### 3.2 테스트 역할

| 역할 | 요구사항 |
|---|---|
| 설치 관리자 | `soarpkg__SOAR_Admin`; 설치·초기화·정책·Route·권한 설정 |
| 운영자 | `soarpkg__SOAR_Operator`; Dashboard·Audit·Simulator 조회·검증 |
| 액션 대상 | 별도 Sandbox 테스트 사용자; Admin/Operator 권한을 직접 부여하지 않음 |
| 최종 결정자 | 요청자·대상자와 분리된 승인자 또는 보안 운영 그룹 |
| Site Guest | Zero-Login 트랙에서만 `soarpkg__SOAR_Inbound_Guest` 최소 권한 |

관리자 계정을 액션 대상이나 최종 결정자로 겸용하지 않는다. 파괴적 액션은 실행하지 않고 승인 대기·버튼 노출 조건·거절·감사 상태만 검증한다.

## 4. 재실행 목표와 품질 게이트

이번 PoC는 설치 후 핵심 운영 회귀, Teams 정책 기반 전달, Zero-Login 경계, Subscriber Global 계약을 하나의 흐름으로 확인하되 서로 대체하지 않는 게이트로 분리한다.

| 게이트 | 통과 기준 | 실패 시 조치 |
|---|---|---|
| G0 대상·버전 | 새 Subscriber 조직, 패키지 `0.1.0.5`, namespace `soarpkg`, 역할 분리 확인 | 대상 확정 전 실행 중단 |
| G1 설치·초기화 | 설치, Admin/Operator 권한, Health, 정책·액션·필요 스케줄 상태 확인 | 설정 원인 기록 후 보완 |
| G2 핵심 회귀 | 5개 Simulator, 4개 Entry point, Dashboard/Audit/Report/Policy 경로 확인 | 기능별 통과·보류 분리; Data 매핑은 별도 이슈화 |
| G3 Teams 연결성 | `IF_Teams_Base`, Principal Access, `soarpkg` 허용 namespace, Health/Ping/직접 POST 확인 | 연결성만 통과시키고 정책 E2E로 확대하지 않음 |
| G4 Teams 정책 E2E | 실제 정책 이벤트, Audit, Action 결과, `DELIVERED` Ledger, 실제 Teams 카드 수신 | 새 이벤트 반복 전 비동기 실패 원인 추적 |
| G5 Inbound·Zero-Login | 공식 fixture 기반 정상·오류·만료·replay와 Guest 경계 확인 | fixture 없으면 양성 경로 보류, 안전 거부만 기록 |
| G6 Global 확장 | Describe, Sensor/Adapter, Event, Flow/Invocable, 권한·중복·실패·Renderer 계약 확인 | 공개 계약 밖의 내부 호출은 금지하고 호환성 이슈로 기록 |
| G7 정리·보고 | 임시 정책·Route·Site·Flow·권한·외부 endpoint 상태 정리, 결과서 연결 | 잔여 상태를 명시한 뒤 종료 |

`Health READY`, Webhook/Ping HTTP 202, Simulator 성공 토스트는 G3 연결성 증적일 뿐 G4 정책 전달 통과 증적이 아니다. G4는 다음 다섯 증적을 모두 확보해야 통과로 판정한다.

1. 정책 이벤트와 감사 로그
2. `SecurityInvocableLogger` 결과의 `isProcessed`·`statusMessage` 또는 동등한 Action 실행 증적
3. Action Ledger의 결정·승인 상태(생성되는 경로인 경우)
4. `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED` Delivery Ledger
5. 실제 Teams 카드 수신 화면

## 5. 단계별 실행 흐름

### Phase 0 — 과거 기록 고정과 착수 스냅샷

1. 기존 PoC 문서와 로그를 변경하지 않고 역사 자료로 표시한다.
2. `sf org list`에서 로그인된 조직·alias·역할을 확인한다.
3. Dev Hub, Provider/namespace 조직, 기존 UAT, 새 Subscriber 후보를 분류한다.
4. 새 대상 alias를 확정하고 Org ID·사용자 이메일·토큰·원시 URL은 문서에 기록하지 않는다.
5. 패키지 버전 `0.1.0.5`와 Subscriber Package Version ID를 설치 직전 화면에서 재확인한다.

**중단 기준:** 대상 조직이 Subscriber가 아니거나, 현재 설치본이 `0.1.0.5`가 아니거나, 테스트 대상·결정자 분리가 불가능하면 설치하지 않는다.

### Phase 1 — 깨끗한 Subscriber 설치

1. 새 Sandbox/Developer/Trial 조직에 `04tdM000000c9OrQAI`를 `AdminsOnly`로 설치한다.
2. Installed Packages에서 패키지명, 버전, namespace를 확인한다.
3. `soarpkg__SOAR_Admin`을 설치 관리자에게 할당한다.
4. 운영 검증 사용자에게만 `soarpkg__SOAR_Operator`를 할당한다.
5. 일반 사용자에는 운영 권한을 할당하지 않고, 별도 테스트 대상을 준비한다.
6. 패키지 탭·Dashboard·Setup & Health Center 진입을 확인한다.
7. 설치 메타데이터와 권한 집합 목록을 결과 기록에 남긴다.

**통과 증적:** 설치 완료, namespace, 권한 집합, 앱/탭, 관리자·운영자·대상자 분리.

### Phase 2 — 설치 후 활성화와 기준선 확정

1. Setup & Health Center에서 서명 키·Inbound 기본 설정·시스템 활성 상태를 확인한다.
2. 1-Click 인프라 설정을 실행하고 정책·액션·스케줄의 변경 전·후를 기록한다.
3. 필요한 표준 스케줄만 Active로 두고 중복 등록 여부를 확인한다.
4. Policy Builder에서 정책·Severity 허들·기본 액션을 읽기 전용으로 먼저 캡처한다.
5. Route Manager에서 기존 Route/Profile 목록과 활성 상태를 기록한다.
6. 새 테스트 Route는 기존 운영 Route와 충돌하지 않는 `POC_RERUN_` 접두사로 만든다.
7. 설치 버전의 공개 Global 계약과 Platform Event를 Describe하고, 문서와 실제 접근 수준을 비교한다.

**안전 규칙:** 기존 정책을 바로 수정하지 않는다. 전용 정책 생성 UI가 없으면 가장 영향이 작은 기존 정책을 선택하고, 변경 전 값을 결과서에 저장한 뒤 명시적 원복 단계를 둔다.

### Phase 3 — 핵심 운영 회귀

다음 기능은 외부 채널과 분리한 기준선으로 먼저 실행한다.

| 묶음 | 실행 항목 | 성공 증적 |
|---|---|---|
| Simulator | Platform TSP, Data CDC, Logic Flow/Apex, Identity & Audit, External Signal | 시나리오 결과와 Audit Log |
| Entry point | Trigger, Sec Facade, Inbound REST, LWC Event Bus | 진입점 결과와 이벤트 기록 |
| Dashboard | 검색, 채널·심각도 필터, Live pause/resume, Fullscreen | 화면 상태와 감사 행 |
| Report | CSV, 주간·월간 PDF | 다운로드·Salesforce Files 저장 |
| Policy | 허들 확인, Drift 미리보기, 승인된 변경·원복 | 전후 값과 저장 결과 |
| Safety | 수동 액션 콘솔·승인 대기 | 실제 파괴적 실행 없이 `PENDING/REJECTED` 등 상태 |

1차 PoC에서 발견된 Data 채널 집계·필터 불일치는 이 단계에서 재현한다. 이벤트 저장과 Dashboard 요약·필터가 일치하지 않으면 제품 이슈로 기록하되, Teams·Inbound 결과와 섞지 않는다.

**G2 판정:** 핵심 기능은 영역별로 통과·보류를 나눈다. Data 매핑 불일치가 외부 전달 경로를 막지 않으면 별도 개선 이슈로 남기고 다음 단계로 진행할 수 있다.

### Phase 4 — Teams 연결성만 검증

1. Salesforce Setup에서 External Credential과 Named Principal을 준비한다.
2. Named Credential Developer Name을 `IF_Teams_Base`로 맞춘다.
3. 테스트 실행 사용자 또는 실제 비동기 실행 컨텍스트에 Principal Access를 부여한다.
4. Named Credential의 관리 패키지 허용 namespace에 `soarpkg`를 등록한다.
5. Setup & Health Center에서 Teams Health, Webhook/Ping, 직접 POST를 순서대로 실행한다.
6. 실제 외부 카드가 보이면 수신 화면을 저장하되, 이 단계의 성공을 정책 E2E로 표현하지 않는다.

**G3 통과 증적:** Named Credential/Principal/namespace 설정, Health 결과, 직접 POST 결과, 원시 URL 비노출.

### Phase 5 — 정상 Teams 정책 E2E

#### 5.1 테스트 자산

| 항목 | 새 실행 기준값 |
|---|---|
| Route Key | `POC_RERUN_TEAMS` |
| Profile Key | `POC_RERUN_TEAMS_DEFAULT` |
| Channel | `TEAMS` |
| Named Credential | `IF_Teams_Base` |
| Action | `NOTIFY_TEAMS` |
| Severity | `HIGH` 또는 승인된 비파괴 Severity |
| Dedupe Window | `300`초부터 시작; 변경 시 이유 기록 |
| Flow | 계획명 `POC_RERUN_Send_Security_Log`; 실제 실행 harness `POC2_Send_Security_Log_Harness` |
| Invocable | `soarpkg.SecurityInvocableLogger` — `Send Security Log` |
| Event marker | `POC-RERUN-<고유값>`; 원시 token·correlation은 기록하지 않음 |

전용 정책 코드가 지원되면 `POC_RERUN_TEAMS_E2E`를 사용한다. 지원되지 않으면 기존 정책을 재사용하되 변경 전·후 값과 원복을 결과서에 남긴다. 과거 `POC2_TEAMS` Route와 과거 `EXTERNAL_THREAT_CALLBACK` 임시 변경은 새 기준으로 간주하지 않는다.

#### 5.2 Flow harness 구성

이번 재실행에서 Subscriber Flow Builder UI로 생성한 실제 harness는 `POC2 Send Security Log Harness` (`POC2_Send_Security_Log_Harness`)이며, 후속 대상 사용자 매핑을 반영한 활성 버전 2를 사용한다. Screen 입력은 `PolicyCode`, `EventKey`, `Severity`, `EventMessage`, `Target_User_Id`이고, `Send Security Log`의 Policy Code/Details/Message/Severity/Target User Id에 각각 매핑했다. 현재 UI에는 문서상 반환값 `isProcessed`·`statusMessage` 출력이 노출되지 않았고, 실행 후 Audit 대상도 입력한 정책 대상이 아닌 실행 주체로 표시되어 계약·런타임 정합성 이슈로 기록한다.

Screen Flow에 다음 입력 화면과 결과 화면을 모두 만든다.

- 입력: `policyCode`, `severity`, `message` 또는 `details`, 선택적 `recordId`, `targetUserId`
- 고유 marker: Event Key 필드와 Details/Message에 동일한 비민감 marker
- 실행 대상: 별도 Sandbox 대상 사용자 문맥
- 결과: Invocable 반환값 `isProcessed`, `policyCode`, `statusMessage`
- 예외 분기: 처리되지 않음, 정책 미매칭, 필수 입력 누락, 예외 메시지를 별도 표시

`SecurityInvocableLogger`의 문서화된 입력에 직접적인 `eventKey`·`idempotencyKey`가 없다면 이를 있다고 가정하지 않는다. 해당 트랙에서는 marker를 Details/Message에 보존하고, 실제 `eventKey`·`idempotencyKey` 계약 검증은 Phase 8 Global 확장에서 `SecuritySubscriberEventContext`와 Adapter로 수행한다.

#### 5.3 정상 실행과 판정 순서

1. Policy Builder에서 테스트 정책·허들·Action을 읽고 Route Preview를 확인한다.
2. Flow를 활성화하고 활성 버전·실행 사용자·입력 매핑을 기록한다.
3. 고유 marker를 넣은 이벤트를 정확히 1회 제출한다.
4. Flow 결과 화면에서 `isProcessed`와 `statusMessage`를 저장한다.
5. Dashboard/Audit에서 정책 코드·Severity·Source·대상 문맥을 확인한다.
6. Action Ledger가 생성되면 요청·승인·결정 상태를 확인한다.
7. Delivery Ledger에서 `ACCEPTED` 이후 Attempt와 최종 `DELIVERED`를 확인한다.
8. Teams 테스트 채널에서 실제 카드 수신을 확인하고 화면 증적을 저장한다.
9. 동일 실행의 Audit·Action Ledger·Delivery Ledger·Teams 카드가 같은 marker/상관관계를 가리키는지 대조한다.

#### 5.4 실패 시 순서

정상 E2E가 실패하면 새 이벤트를 반복하지 않는다.

1. Ledger의 Attempt, 오류 코드, 최종 상태를 기록한다.
2. Direct Route Health와 비동기 Action 결과를 별도 판정한다.
3. 실행 사용자·비동기 주체·Principal Access·namespace를 재확인한다.
4. 필요한 실행 주체에 Trace를 설정하고 한 번만 재실행한다.
5. 원인이 밝혀지기 전에는 Retry/Fallback 장애 시험으로 넘어가지 않는다.

`EXHAUSTED`나 `DELIVERY_FAILED`는 연결 실패로 단정하지 않고 “비동기 전달 실패”로 기록한다. `DELIVERED`와 Teams 수신이 없으면 G4를 통과시키지 않는다.

### Phase 6 — 실패·복구·Fallback

정상 Teams E2E가 통과한 뒤에만 진행한다.

1. 제어 가능한 mock endpoint 또는 승인된 실패 응답 환경을 준비한다.
2. 일시적 5xx/timeout을 1회 발생시킨다.
3. 제한된 재시도 횟수와 backoff를 Delivery Ledger에서 확인한다.
4. Fallback Route와 관리자 알림이 선택되는지 확인한다.
5. 최종 `FAILED`/`EXHAUSTED`와 원래 이벤트의 상관관계를 기록한다.
6. 실패 endpoint와 테스트 Route를 비활성화한다.

제어 가능한 실패 endpoint가 없으면 이 Phase는 보류한다. 실제 운영 Teams endpoint에 장애를 의도적으로 만들지 않는다.

### Phase 7 — Inbound·Zero-Login 독립 트랙

#### 7.0 URL 계약과 화면 증적 게이트

이 트랙에서는 외부 전송 endpoint와 사용자에게 노출되는 callback Site URL을 같은 링크로 취급하지 않는다.

| 구분 | 역할 | PoC 확인 방법 |
|---|---|---|
| Teams/Power Automate endpoint | Salesforce Named Credential이 outbound 알림 payload를 보내는 수신 주소 | Named Credential route match와 직접 POST 수락을 확인. 원문 URL은 문서·스크린샷에 남기지 않음 |
| Experience Cloud Site callback | Teams 카드의 승인·실행 링크가 여는 Zero-Login 화면 | 카드 링크의 host/path가 활성 HTTPS Site의 callback 경로인지 확인하고 Chrome에서 화면을 열어 상태를 판정 |

사용자가 제공한 Power Automate endpoint가 Site 주소로 표시되거나 Teams 카드 action URL로 노출되는 것은 성공 조건이 아니다. 정상 조건은 외부 전송은 Named Credential 내부에만 있고, 카드 action은 `InboundBaseUrl`과 패키지 callback suffix로 만들어진 Site URL을 가리키는 것이다.

카드와 링크 검증은 다음 세 화면 상태를 각각 남겨야 한다. 링크 원문·token·query는 마스킹한 화면만 증적으로 보존한다.

1. 유효한 단일 사용 링크: 승인 필요 또는 정상 처리 화면, Audit/Action Ledger 상관관계 확인
2. 만료 링크: 만료 안내 화면, 승인·실행이 발생하지 않음 확인
3. 재사용(replay) 링크: 재사용 거부 화면, 중복 승인·실행이 발생하지 않음 확인

카드가 실제로 도착하지 않으면 위 화면 검증은 수행할 수 없으며, Teams 연결성 통과만으로 이 게이트를 통과시키지 않는다.

#### 7.1 기반 설정

1. HTTPS Experience Cloud Site를 Inbound 전용으로 만든다.
2. Site 기본 주소만 `InboundBaseUrl__c`에 저장하고 REST suffix를 직접 붙이지 않는다.
3. `SecurityInboundConfig__mdt.Default`의 `IsSystemEnabled__c`, `EnableWebhookSignature__c`, Secret 상태를 확인한다.
4. Site Guest에 `soarpkg__SOAR_Inbound_Guest`만 최소 범위로 할당한다.
5. Health Center Callback이 `READY`인지 확인한다.
6. Site root는 운영 Dashboard가 아닌 `InMaintenance` 등 비노출 페이지로 유지한다.

#### 7.2 Fixture 게이트

공식 action code/token/signature/expiry/replay fixture가 없으면 정상 callback을 시도하지 않는다. 임의 payload 변형은 공식 성공 증적이 아니다.

- Fixture 출처·버전·필드표를 기록한다.
- 정상 비파괴 fixture를 먼저 보낸다.
- 누락·변조 action code/token, 서명 오류, 만료, replay, malformed, rate limit을 각각 분리한다.
- `404`, `403`, `MISSING_ACTION_CODE`, 만료·재사용 오류는 안전 거부 증적으로만 기록한다.
- 정상 경로에서 승인 대기·승인·실행·결과 HTML·Audit 상태를 대조한다.
- Teams 카드가 수신된 뒤 카드 action URL이 Power Automate endpoint가 아닌 Site callback URL인지 확인한다.
- 동일한 callback 링크를 Chrome에서 열어 유효·만료·replay 화면을 각각 확인하고, 화면 상태와 원장을 대조한다.

Outbound Teams fixture와 Inbound callback fixture는 서로 대체하지 않는다.

### Phase 8 — Subscriber Global 계약과 사용자 확장

3차 PoC 계획을 현재 기준 패키지로 재실행한다.

1. `soarpkg.ISecuritySensor`, `SecuritySensorAdapter`, `SecuritySubscriberEventContext`, `Sec`, `soarpkg__SecurityAlert__e`, `SecurityInvocableLogger`, `ISecurityActionHtmlRenderer`를 Describe한다.
2. 문서의 `global/public` 접근 수준·파라미터·반환값과 설치 버전을 비교한다.
3. 최소 테스트 객체와 전용 레코드를 준비한다.
4. 업무 객체 센서 → Adapter → 표준 이벤트 → Flow/Trigger/Queueable 흐름을 만든다.
5. `SecuritySubscriberEventContext`의 `eventKey`·`idempotencyKey`로 동일 요청 재처리를 검증한다.
6. `Sec.log`, `Sec.evaluateEvent`, `Sec.triggerAction`을 비파괴 정책으로 검증한다.
7. Invocable 결과를 성공·실패 분기로 연결한다.
8. Subscriber 후속 업무는 Task·티켓·내부 알림 mock 범위에서 실행하고 패키지 원장을 직접 수정하지 않는다.
9. Renderer는 결과 표현만 바꾸고 인증·승인·대응 로직을 넣지 않는다.
10. Admin·Operator·일반 사용자·Guest의 화면·API·Flow 접근을 비교한다.
11. 비동기 실패·제한 재시도·롤백·킬 스위치·감사 결과를 기록한다.

### Phase 9 — 정리와 결과서 확정

1. 테스트 정책·임시 허들·Route를 원래 값으로 복원하거나 유지 이유를 승인받는다.
2. Flow harness와 Subscriber 테스트 코드를 비활성화·보관한다.
3. Site, Guest 권한, Named Credential, Principal Access, 외부 endpoint의 유지·비활성화 상태를 기록한다.
4. 테스트 이벤트·Task·파일·승인 요청·원장을 정리할지 보존할지 결정한다.
5. 새 결과서에는 통과·보류·실패·미실행을 게이트별로 작성하고, 과거 결과와 섞지 않는다.
6. 결과서·실행 로그·스크린샷에 Secret, URL query, token, 사용자 이메일, Org ID가 없는지 점검한다.

## 6. 새 PoC 테스트 ID

| ID 범위 | 검증 영역 |
|---|---|
| R0-01~R0-05 | 오그·버전·역할·보안 기준선 |
| R1-01~R1-08 | 설치·Health·정책·스케줄 |
| R2-01~R2-14 | 핵심 UI·Simulator·Entry point·Data 회귀 |
| R3-01~R3-09 | Teams 연결성·Principal·Route |
| R4-01~R4-08 | 정상 Teams 정책 E2E·Ledger·카드 |
| R5-01~R5-06 | Retry·Fallback·최종 실패 |
| R6-01~R6-12 | Inbound·Zero-Login·Site callback·token 상태·fixture·Guest 경계 |
| R7-01~R7-15 | Global 계약·Subscriber 이벤트·권한·호환성 |
| R8-01~R8-06 | 정리·결과·문서·잔여 위험 |

모든 테스트 ID는 `poc-rerun-execution-log.md`와 `poc-rerun-results.md`에서 동일하게 사용한다.

## 7. 필수 설정값

| 구분 | 값/형식 | 비고 |
|---|---|---|
| Package | `SOAR_Operations_Core_Next 0.1.0.5` | 설치 직전 재확인 |
| Package ID | `04tdM000000c9OrQAI` | 제공자가 새 ID를 주면 교체 |
| Namespace | `soarpkg` | API/권한/허용 namespace |
| Target alias | `soarInstallTest` | 2026-08-26 새로 생성한 Subscriber Scratch Org |
| Admin | `soarpkg__SOAR_Admin` | 설치 관리자에게만 |
| Operator | `soarpkg__SOAR_Operator` | 운영 검증 사용자 |
| Guest | `soarpkg__SOAR_Inbound_Guest` | Inbound 트랙에서만 |
| Teams Named Credential | `IF_Teams_Base` | 원시 URL은 Setup에만 |
| Teams Permission | 기존 또는 새 최소 Subscriber Permission Set | Principal Access만 부여 |
| Teams Route | 실행은 canonical `DEFAULT_TEAMS` 사용 | 새 `POC_RERUN_` Route는 생성하지 않음 |
| Teams Action | `NOTIFY_TEAMS` | 정상·비파괴 전달부터 |
| Flow | 실제 `POC2_Send_Security_Log_Harness` V2 | 결과 Screen 미구성; `Target_User_Id` 매핑 후에도 대상 표시 불일치 |
| Inbound Config | `SecurityInboundConfig__mdt.Default` | Base URL·Secret·활성·서명 |
| Global Event | `soarpkg__SecurityAlert__e` | Describe 후 사용 |
| Global test policy | `POC_RERUN_CUSTOM_EVENT` 권장 | 지원되지 않으면 기존 정책을 명시적으로 재사용 |

실제 Secret, Webhook URL, OAuth token, callback code, correlation, 사용자·조직 식별자는 계획서와 실행 로그에 적지 않는다.

## 8. 새 PoC 완료 기준

- 현재 기준 패키지 설치와 역할별 최소 권한이 확인된다.
- 핵심 운영 회귀와 Data 채널 상태가 별도 판정된다.
- Teams 연결성과 정책 기반 Teams 전달이 분리된 증적으로 검증된다.
- 정상 Teams E2E에서 Flow 결과, Audit, Action/Delivery Ledger, 실제 카드 수신이 모두 일치한다.
- Teams 카드 action URL이 활성 Site callback을 가리키고, Chrome에서 유효·만료·replay 상태 화면을 각각 확인한다.
- 실패·Retry·Fallback은 정상 전달 후 제어 가능한 환경에서만 검증된다.
- Inbound 양성 경로는 공식 fixture가 있을 때만 판정하고, 음성·안전 거부와 분리한다.
- Subscriber Global 계약은 설치 버전 Describe 결과와 문서가 일치한다.
- idempotency·권한·승인·실패·롤백·킬 스위치 경계가 감사 원장과 연결된다.
- Production 무승인 변경이나 실제 파괴적 액션 실행이 없다.
- 임시 설정과 외부 노출 잔여 상태가 결과서에 명시된다.

## 9. 현재 계획 종료 판정

| 트랙 | 종료 상태 | 다음 진행 조건 |
|---|---|---|
| 설치·핵심 UI | 부분 통과 | Data 집계/필터와 CSV 피드백을 제품 개선 항목으로 추적 |
| Teams 연결성 | 통과 | `IF_Teams_Base` Health·직접 POST 성공 증적 확보 |
| Teams 정책 E2E | 실패 | `SecurityNotifyTeamsAction` 비동기 실행·대상 바인딩 원인 수정 또는 재현 후 `DELIVERED`·카드 수신 재검증 |
| Retry/Fallback | 보류 | 정상 전달 통과 후 제어 가능한 실패 endpoint 필요 |
| Inbound·Zero-Login | 보류 | HTTPS Site/Guest, 카드 callback 링크, 유효·만료·replay 화면, 공식 action code·token fixture 필요 |
| Subscriber Global | 미실행 | 설치 버전 Describe와 Subscriber harness 필요 |
| 정리·보고 | 문서 완료·환경 보류 | 테스트 Flow V2·Permission Set·Named Credential 유지/비활성화 결정 |

이 표로 현재 재실행 계획을 종료한다. “종료”는 모든 게이트 통과를 의미하지 않으며, 실패·보류·미실행 항목을 다음 작업의 명시적 진입 조건으로 남긴다는 뜻이다.

## 10. 참고 문서

- [패키지 README](../../README.md)
- [사용자 매뉴얼](../user/README.md)
- [설치 후 전체 사용 런북](../user/end-to-end-runbook.md)
- [설치 및 초기 활성화](../user/installation-and-setup.md)
- [일상 운영과 권한 모델](../user/operations.md)
- [문제 해결](../user/troubleshooting.md)
- [Subscriber 확장 가이드](../extensions/README.md)
- [플랫폼 이벤트·Flow·커스텀 액션](../extensions/events-flows-and-actions.md)
- [센서·비즈니스 로직](../extensions/sensors-and-business-logic.md)
- [Teams·Slack·Zero-Login](../extensions/notifications-and-zero-login.md)
- [외부 연동·정책·브랜딩](../extensions/external-policy-and-branding.md)
- [검증 기준](./validation.md)
- [과거 1차 PoC 결과](./poc-1-results.md)
- [과거 2차 PoC 결과](./poc-2-results.md)
- [과거 3차 PoC 계획](./poc-3-plan.md)
