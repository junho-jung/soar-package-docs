# SOAR 패키지 2차 PoC 결과서

## 1. 문서 개요

| 항목 | 내용 |
|---|---|
| PoC | 2차 PoC — Teams 알림 및 외부 Inbound·Zero-Login 검증 |
| 검증일 | 2026-08-23 |
| 대상 환경 | Salesforce Scratch Org alias `soarInstallTest` |
| 패키지 | `SOAR_Operations_Core_Next 0.1.0.1` |
| Namespace | `soarpkg` |
| 선행 문서 | [2차 PoC 계획서](./poc-2-plan.md), [1차 PoC 결과서](./poc-1-results.md) |
| 상세 실행 로그 | [`sandbox-install-test-log.md`](../../sandbox-install-test-log.md) |

이 문서는 1차 PoC에서 보류한 Teams 외부 전달과 외부 Inbound·Zero-Login 경계를 실제 테스트 조직에서 검증한 결과다. 사용자 제공 Webhook URL, 실제 Site 호스트, Secret, 사용자·조직 식별자는 보안상 기록하지 않고 Salesforce 설정 화면에만 보관했다.

## 2. 종합 판정

### 판정: Teams 연결·Principal Access 통과 — 정책 기반 Teams E2E는 부분 실행, Teams 전달 실패

초기 설정 점검과 테스트 카드·Ping에서는 HTTP 202를 확인했고, 후속 UI 점검에서 `IF_Teams_Base`의 `soarpkg` 허용 namespace, 기존 subscriber Permission Set의 Principal Access, 현재 테스트 사용자 할당, 직접 Teams POST 성공을 확인했다. 후속 Outbound E2E에서는 전용 Subscriber Flow harness를 실제로 저장·활성화하고 UI에서 `Send Security Log`를 실행했다. 그 결과 정책 감사 로그와 `POC2_TEAMS` Delivery Ledger는 생성됐지만 최신 행은 `EXHAUSTED`, `3/3`, `DELIVERY_FAILED`였고 실제 Teams 카드와 `SecurityNotifyTeamsAction` 성공 결과는 확인하지 못했다. 따라서 현재 상태는 “Teams 연결·Principal Access는 통과했으나 정책 기반 E2E가 비동기 전달 단계에서 실패”이며, “Teams 연결 실패”로 표현하지 않는다.

Zero-Login은 공개 Site 생성, HTTPS Inbound Base URL, Guest 권한 핀포인트 부여까지 완료했고 Health Center에서 `READY`를 확인했다. 공개 REST 경로도 도달했지만, 패키지가 요구하는 callback action-code/token 입력 형식은 계약 매니페스트의 일반 action schema만으로 재구성할 수 없었다. 따라서 안전한 오류 거부까지만 검증하고 유효 callback 실행은 보류했다.

| 품질 게이트 | 결과 | 판단 |
|---|---|---|
| Teams Named Credential·Principal 연결 | Health `READY`; `IF_Teams_Base`·`soarpkg` namespace·Principal Access·현재 사용자 할당 확인; 직접 POST 약 636ms 성공 | 통과 |
| Outbound 정책 기반 Teams E2E | Flow harness 활성화·정책 감사 로그·새 `POC2_TEAMS` Ledger 생성까지 확인했으나 최신 Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`; `SecurityNotifyTeamsAction` 성공·Teams 카드 미확인 | 부분 실행; 전달 실패 원인 추적 필요 |
| Subscriber Flow harness | `POC2_Send_Security_Log_Harness` V1 저장·활성화 및 Screen 입력 매핑 확인 | 통과; 결과 화면이 없어 액션 출력은 직접 노출되지 않음 |
| Teams 테스트 카드·Ping | 카드 HTTP 202, Ping HTTP 202 | 통과; 실제 수신 스크린샷 대기 |
| Route/Profile 구성 | `POC2_TEAMS` 저장, Preview `ROUTE_VALID` | 통과 |
| 1차 PoC 핵심 기능 회귀 | 시뮬레이터·엔트리포인트·Dashboard·정책·감사·리포트·Flow 확인 | 통과 |
| Zero-Login Site·Guest 권한 | Site Active, Guest 권한 부여, Callback `READY` | 통과 |
| 외부 REST 공개 도달성 | namespace 누락 404, namespace 포함 malformed 요청 400 | 통과 |
| 유효 외부 Inbound callback | `MISSING_ACTION_CODE`로 안전 거부; 공식 fixture 필요 | 보류 |
| UI 실제 정책 액션 | 실제 Flow 이벤트가 정책 감사 로그와 `POC2_TEAMS` Ledger까지 도달했으나 3회 재시도 후 `EXHAUSTED`; 액션 성공·Teams 카드 미확인 | 전달 실패; 비동기 액션 원인 추적 필요 |
| 서명·만료·replay·rate limit 행렬 | 유효 callback 형식 미확정으로 미실행 | 보류 |
| Slack·Retry/Fallback | Slack Named Credential·제어 가능한 실패 endpoint 없음 | 보류 |
| Data Dashboard 회귀 | 저장 이벤트는 있으나 Data 카운터·필터가 0으로 표시 | 개선 필요 |

## 3. 실행 환경과 설정값

### 3.1 Teams 설정

| 항목 | 실제 테스트 값/상태 |
|---|---|
| External Credential | Label `SOAR Teams Webhook POC2`, API Name `SOAR_Teams_Webhook_POC2` |
| 인증 방식 | `인증 안 함` — endpoint 원문은 Salesforce 설정에만 저장 |
| Named Credential | API Name `IF_Teams_Base` |
| Named Credential Label | 가이드 규격에 맞춰 `IF_Teams_Base`로 정리 |
| Named Principal | `SOAR_TEAMS_POC2_PRINCIPAL` |
| Principal 권한 | 기존 subscriber Permission Set `SOAR POC2 Teams Callout`의 Principal Access에 `SOAR_TEAMS_POC2_PRINCIPAL`이 등록되고 현재 테스트 사용자에게 할당됨 |
| 인증 헤더 생성 | 가이드 권장값에 맞춰 해제 |
| 패키지 접근 | Named Credential의 콜아웃 허용 namespace에 `soarpkg` 등록·저장 확인 |
| 테스트 Route | `POC2_TEAMS` / Profile `POC2_TEAMS_DEFAULT` |
| Route 조건 | Channel `TEAMS`, Policy `EXTERNAL_THREAT_CALLBACK`, Action `NOTIFY_TEAMS`, Severity `HIGH`, Priority `100`, Dedupe `300`초, Active |
| UI 시뮬레이터 Route | `POC2_TEAMS_SIM` / `POC2_TEAMS_SIM_DEFAULT`, `LOGIN_BRUTE_FORCE_BURST`, `NOTIFY_TEAMS`, Severity 전체, Active |
| UI 외부 Inbound Route | `POC2_TEAMS_GUEST` / `POC2_TEAMS_GUEST_DEFAULT`, `GUEST_USER_DATA_LEAK`, `NOTIFY_TEAMS`, Severity `HIGH`, Active |

External Credential의 주체와 권한을 만들지 않은 첫 호출은 `CALLOUT_FAILED`였고, 주체를 만든 뒤에도 managed package namespace가 허용되지 않아 두 번째 호출이 실패했다. 이후 Health Center 직접 테스트와 권한·namespace 재확인은 성공했다. 과거 실제 정책 이벤트의 Delivery Ledger에 External Credential 접근 오류가 기록된 사실은 보존하되, 이를 현재 Teams 연결 실패로 일반화하지 않는다. 현재 설정상 effective access와 직접 callout은 통과했고, 활성 Subscriber Flow를 통한 후속 정책 E2E는 실행됐으나 비동기 전달이 `EXHAUSTED/DELIVERY_FAILED`로 종료됐다.

### 3.1.1 실제 정책 이벤트 추가 증적

초기 사용자 제공 실행 결과와 후속 UI 검증을 분리해 기록한다. 초기 실행은 다음 단계까지 확인됐다.

`보안 이벤트 → NOTIFY_TEAMS → Delivery Ledger 기록 → Teams callout 재시도 3회 → EXHAUSTED`

이 증적은 앞서 UI 시뮬레이터에서 `Delivery Ledger`가 비어 있던 결과와 다른 경로다. 시뮬레이터의 성공 토스트와 초기 실제 이벤트의 `EXHAUSTED`는 각각 모의 실행·역사적 실패 증적으로 보존한다. 이후 UI에서 `IF_Teams_Base`의 `soarpkg` namespace, `SOAR_TEAMS_POC2_PRINCIPAL` Principal Access, 현재 테스트 사용자 할당과 직접 Teams POST 성공을 확인했고, 후속 Flow 실행에서는 감사 로그와 새 `POC2_TEAMS` Ledger 생성까지 도달했다. 최종 Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`였으므로 현재 사용자에게는 “Teams 연결 실패”가 아니라 “Teams 연결과 Principal Access는 통과했지만 정책 기반 Teams E2E가 전달 단계에서 실패했다”고 전달한다.

보안 허브의 `1-Click 설정 가이드`는 Named Credential을 자동 생성하지 않고, Salesforce Setup에서 생성할 필수 이름·endpoint·인증 옵션을 안내한다. 가이드 경유로 기존 `IF_Teams_Base`를 `Label/Name = IF_Teams_Base`, `Generate Authorization Header = 해제`, `Allowed Namespace = soarpkg` 상태로 저장했다. 다만 가이드에는 External Credential Principal Access 및 subscriber Permission Set 생성 단계가 없어 별도 권한 설정이 필요하다.

후속 권한 점검에서는 새 `SOAR POC2 Teams Callout` Permission Set 저장 시 레이블 중복 오류가 발생했다. 목록 검색 결과 동일 레이블의 기존 subscriber Permission Set이 이미 생성되어 있었고, 그 상세 화면에서 API Name이 중복 문자열로 표시되는 UI 입력 흔적도 확인했다. 기존 항목의 외부 자격 증명 주체 액세스에는 `SOAR_Teams_Webhook_POC2 - SOAR_TEAMS_POC2_PRINCIPAL`이 등록되어 있었으며 현재 테스트 사용자도 활성 할당 상태였다. 이 기존 항목을 재사용하고 추가 권한집합은 만들지 않았다.

> 참고: 패키지 기본 정책 매핑에서 `EXTERNAL_THREAT_CALLBACK`의 High 기본 조치는 `NOTIFY_SLACK`으로 표시된다. 이번 2차 PoC는 별도 테스트 Route에 `NOTIFY_TEAMS`를 명시해 Teams 전달 경로 자체를 검증한 것이다. 운영 정책 매핑을 Teams로 확정할 때는 이 차이를 별도로 승인해야 한다.

### 3.2 Inbound·Zero-Login 설정

| 항목 | 실제 테스트 값/상태 |
|---|---|
| Custom Metadata | `SecurityInboundConfig__mdt` record `Default` |
| 시스템 활성화 | `Is System Enabled = true` |
| 유효 시간 | `Valid Seconds = 300` |
| Throttle | `Throttle Max Per Minute = 10` |
| Inbound Base URL | `<salesforce-sites-domain>/soarpoc2inbound` 형식으로 저장 |
| 서명 설정 | Setup UI에서 `Enable Webhook Signature` 체크박스가 비활성·미체크 상태로 노출되어 서명 성공 행렬은 보류 |
| Secret | 생성값 확인 및 사용 상태만 확인; 원문은 문서·로그에 기록하지 않음 |
| Site | Label `SOAR POC2 Inbound`, Site Name `SOAR_POC2_Inbound`, 기본 웹 주소 `soarpoc2inbound`, Active |
| Site 기본 페이지 | `InMaintenance` — Inbound 전용 공개 진입점의 운영 화면 비노출 상태 |
| Guest 권한 | 패키지 제공 `SOAR_Inbound_Guest`를 탐지된 Site Guest User에 1-Click으로 할당 |
| Callback 상태 | Health Center `READY` |

### 3.3 액션 계약 확인

읽기 전용 Describe 및 REST 계약 조회로 다음을 확인했다.

- `IF_SecurityActionController` REST mapping: `/api/security/action`
- 관리 패키지 공개 호출 경로 형식: `/services/apexrest/soarpkg/api/security/action`
- `NOTIFY_TEAMS`는 `destructive=false`, `requiresToken=false`, `requiresApproval=false`, `executionMode=ASYNC`로 매니페스트에 표시됨
- 계약 입력에는 `actionType`, `userId`, `eventKey`, `source`, `manifestVersion`가 필수로 표시되고, target·request·correlation·idempotency·policy·dry-run 문맥 필드가 선택적으로 표시됨
- `EXTERNAL_THREAT_CALLBACK` 및 `GUEST_USER_DATA_LEAK` 정책을 비교해 정책 매핑 차이를 확인했으나, 두 정책 모두 공개 controller의 `MISSING_ACTION_CODE` 요구를 해소하지 못함

이 계약 매니페스트는 action type의 메타데이터를 설명하지만, 공개 callback controller가 요구하는 별도 action-code/token 입력 규칙 또는 발신 fixture까지 제공하지는 않았다. 그래서 임의 추측으로 필드를 확장하거나 게스트에게 계약 Apex 접근을 추가하지 않았다.

## 4. 실행 결과

| ID | 영역 | 실행 결과 | 판정 |
|---|---|---|---|
| T2-01 | Teams Health/POST/Ping | Health `READY`; 외부 채널 테스트 POST HTTP 202, 약 566ms; Webhook Ping HTTP 202, 약 554ms | 통과 |
| T2-02 | Teams 카드 | `SUCCESS (HTTP 202): Teams Card Sent Successfully!` | 통과; 실제 수신 스크린샷 대기 |
| T2-03 | Route/Profile | `POC2_TEAMS` 저장, Route Preview `ROUTE_VALID`, Health Center 목록 노출 | 통과 |
| T2-04 | 핵심 기능 회귀 | 5개 채널, 4개 엔트리포인트, Dashboard 검색·필터·Fullscreen·Live pause/resume, CSV, 주간·월간 PDF, Flow 템플릿, Policy Builder를 실행 | 통과 |
| T2-05 | Retry/Fallback | 제어 가능한 실패 endpoint가 없고 Slack credential도 없어 미실행 | 보류 |
| T2-06 | 공개 REST 경로 | namespace 없는 경로 HTTP 404 `NOT_FOUND`; namespace 포함 malformed 요청 HTTP 400 `MISSING_ACTION_CODE` | 안전 거부 확인 |
| T2-07 | 정상 Inbound | contract-shaped 요청도 HTTP 400 `MISSING_ACTION_CODE`; 공식 callback fixture 미확보 | 보류 |
| T2-08 | 서명 오류 | 서명 기능 설정이 UI에서 비활성·미체크이고 정상 fixture도 없음 | 보류 |
| T2-09 | 만료·replay·중복 | 정상 callback이 먼저 확정되지 않아 실제 외부 재전송 행렬 미실행 | 보류 |
| T2-10 | Zero-Login Guest | 공개 Site root 도달, `InMaintenance` 페이지 표시; 운영 Dashboard는 노출하지 않음 | 통과 |
| T2-11 | 승인·결과 경계 | 기존 Dashboard에서 `세션 종료` 승인 요청까지만 생성; 파괴적 실행 없음 | 안전 통과 |
| T2-12 | Data 채널 회귀 | `MASS_DATA_DELETION` 감사 이벤트는 저장되나 Data 요약·필터가 0으로 표시 | 개선 필요 |
| T2-13 | UI 정책 Route·시뮬레이터 | `LOGIN_BRUTE_FORCE_BURST → NOTIFY_TEAMS` Route를 UI에서 `ROUTE_VALID`로 저장하고 시뮬레이터 버튼 실행 성공 토스트 확인 | UI 실행 통과; 외부 전달 보류 |
| T2-14 | UI Delivery Ledger | 시뮬레이터 실행 후 새로고침·대기에도 `아직 notification delivery 이력이 없습니다.` 유지 | 실제 action callout 증적 없음 |
| T2-15 | UI Teams Webhook 탭 | 고정 MessageCard JSON 프리뷰와 테스트 카드 전파 버튼 확인; 인터랙티브 액션·정책 이벤트 입력 없음 | 연결테스트와 실제 액션 분리 |
| T2-16 | Teams Principal Access 재확인 | 기존 `SOAR POC2 Teams Callout`의 `SOAR_TEAMS_POC2_PRINCIPAL` 접근 행과 현재 테스트 사용자 활성 할당을 Setup UI에서 확인 | 통과 |
| T2-17 | Health Center 테스트 이벤트 | `테스트 보안 이벤트 발송` 성공 토스트; Dashboard에 `WIZARD_TEST_POLICY` 신규 기록 확인 | SOAR 엔진 기록 통과 |
| T2-18 | 실제 Teams Delivery Ledger 재검증 | Health Center 테스트 이벤트 후에도 새 `NOTIFY_TEAMS` Ledger 행은 생성되지 않고 기존 `EXHAUSTED` 행만 표시 | 보류; Teams 실제 POST 또는 공식 정책 이벤트 경로 필요 |
| T2-19 | Teams 실제 POST 재검증 | Health Center `Teams 실제 POST 점검`이 `READY` 및 외부 채널 테스트 POST 성공(약 636ms) 반환 | 통과; 정책 이벤트 Delivery Ledger와 분리 |
| T2-20 | 패키지 Flow 템플릿 실행 가능성 | 두 관리 패키지 Flow가 플랫폼 이벤트 트리거이지만 현재 비활성·초안 상태로 확인됨 | 템플릿 확인 통과; 임의 활성화·실행 보류 |
| T2-21 | LWC Event Bus 진입점 재검증 | `LWC Platform Event Published` 성공; Dashboard 로그 14건, 새 행은 `MASS_DATA_EXPORT / DEMO_SIMULATOR` | 이벤트 발행 통과; 새 Teams Ledger 없음 |
| T2-22 | Subscriber Flow harness | `POC2_Send_Security_Log_Harness` V1을 Screen Flow로 저장·활성화하고 `Send Security Log` 입력 매핑을 확인 | 통과; 결과 화면은 미구성 |
| T2-23 | 정책 기반 Outbound 실행 | UI Flow에서 `EXTERNAL_THREAT_CALLBACK`·`HIGH` 고유 이벤트를 제출하고 Dashboard에 `HIGH / EXTERNAL_THREAT_CALLBACK / APEX` 감사 행 생성 확인 | 정책 평가·감사 로그 통과 |
| T2-24 | Outbound Delivery Ledger | 새 `POC2_TEAMS` 행에서 `NOTIFY_TEAMS`, 시도 `3/3`, `EXHAUSTED`, `DELIVERY_FAILED` 확인 | 전달 실패; 기대값 `DELIVERED` 미달 |
| T2-25 | `SecurityNotifyTeamsAction` 성공 | Flow 결과 화면이 없고 Ledger가 실패 상태여서 액션 성공 출력·실행 상세를 확인하지 못함 | 미확인; 성공으로 판정하지 않음 |
| T2-26 | 실제 Teams 카드 수신 | 이번 정책 기반 실행에 대한 Teams 수신 화면·사용자 스크린샷 미확인 | 미통과 |
| T2-27 | 비동기 Debug Trace 후속 | 현재 사용자 `USER_DEBUG` Trace는 저장됐으나 로그 행이 생성되지 않았고, 추가 실행은 Salesforce 세션 만료로 로그인 화면 전환 | 원인 상세 로그 미확보 |

### 4.1 1차 PoC 기능을 포함한 회귀 범위

2차 PoC에서도 외부 연동만 별도 확인하지 않고, 패키지에서 노출된 운영 기능을 다시 사용했다.

- Threat Simulator: Platform TSP, Data CDC, Logic Flow/Apex, Identity & Audit, External Signal 5개 채널
- Entry point: Trigger, Sec Facade API, REST API, LWC Event Bus 4개
- Dashboard/Audit: 실시간 중지·재개, Fullscreen SOC Mode, 검색, 채널·위험도 필터, 행 작업, CSV
- Report: 주간·월간 PDF 생성 및 Salesforce Files 저장
- Policy Pipeline: 12개 시그널·12개 정책·9개 액션, Drift 미리보기, STRICT 일괄 적용
- Setup/Health: Teams Health, Route Preview, Site/Guest Callback 재진단, 표준 상태 확인
- Flow/Action: 관리 패키지 Flow 템플릿 확인, 수동 액션 콘솔 확인, 승인 기반 세션 종료 요청 생성

실제 계정 동결·세션 종료·토큰 회수·MFA 리셋은 실행하지 않았다.

## 5. 외부 Inbound 상세 결과

### 통과한 경계

1. 공개 Site와 Inbound Base URL을 생성·저장했다.
2. 처음에는 Health Center가 `PERMISSION_MISSING`을 표시했지만, 사이트 선택 후 `1-Click 게스트 권한 부여`를 실행해 `SOAR_Inbound_Guest`를 Guest User에 할당했다.
3. 재진단 결과는 `READY`였고, 시스템·Secret·HTTPS URL·Site Guest가 준비됐다고 표시됐다.
4. namespace 없는 REST 경로는 404로 거부됐다.
5. namespace 포함 REST 경로는 controller까지 도달해 `MISSING_ACTION_CODE`를 반환했다. 이는 공개 Site·managed namespace·Apex REST 매핑이 동작한다는 증거다.
6. 게스트의 action-contract REST 조회는 403으로 거부됐다. 최소 권한 경계가 유지됐다.

### UI 실제 액션 검증 결과

시스템 진단의 원클릭 설정 가이드와 UI 기능을 확인했다. 원클릭 씨딩/초기화는 정책·대응 액션 메타데이터를 준비하는 설치 후 활성화 기능이고, Teams 외부 액션을 직접 발생시키지 않는다. Teams Webhook 탭은 고정 테스트 카드 전파만 수행한다.

UI에서 다음 Route를 추가했다.

- `POC2_TEAMS_GUEST`: `GUEST_USER_DATA_LEAK` → `NOTIFY_TEAMS`, High
- `POC2_TEAMS_SIM`: `LOGIN_BRUTE_FORCE_BURST` → `NOTIFY_TEAMS`, 전체 Severity

이후 위협 시뮬레이터에서 `Identity & Audit`의 `LOGIN_BRUTE_FORCE_BURST` 모의 발송 버튼을 눌러 성공 토스트를 확인했지만 `최근 Delivery Ledger`에는 아무 이력도 생성되지 않았다. 따라서 시뮬레이터 성공을 실제 Teams 액션 전달 성공으로 판정하지 않는다.

추가로 Health Center의 `테스트 보안 이벤트 발송`을 실행했다. UI에는 “테스트 성공” 토스트가 표시됐고 Dashboard 새 감사 행도 추가됐지만 정책 코드는 `WIZARD_TEST_POLICY`였다. Delivery Ledger에는 새 행이 생성되지 않아, 이 버튼 역시 엔진 이벤트 기록 확인용이며 `NOTIFY_TEAMS` 외부 callout을 직접 증명하는 진입점은 아닌 것으로 판정했다.

권한 점검 후 `Teams 실제 POST 점검`을 다시 실행한 결과는 `READY · TEAMS 외부 채널 테스트 POST가 성공했습니다`였고 왕복 시간은 약 636ms였다. 이 결과로 Named Credential·External Credential·Principal Access·namespace 경로의 직접 callout은 통과했지만, 정책 이벤트의 비동기 Delivery Ledger 성공까지 의미하지는 않는다.

동일 세션에서 `LWC Event Bus 진입점 테스트`도 재실행해 `SUCCESS: LWC Platform Event Published`를 확인했다. Dashboard 감사 로그는 14건으로 증가했지만 새 이벤트는 `MASS_DATA_EXPORT / DEMO_SIMULATOR`로 표시됐고 Delivery Ledger에는 새 `NOTIFY_TEAMS` 행이 없었다. 따라서 LWC 발행 성공과 실제 정책 notification delivery를 별도 품질 게이트로 유지한다.

### 3.1.2 후속 Outbound Teams E2E 검증 기준

다음 실행은 Health Center 직접 POST나 시뮬레이터 재실행이 아니라, 전용 Subscriber Flow harness에서 `Send Security Log`를 호출하는 방식으로 진행한다.

1. 임계값 `1`, Action `NOTIFY_TEAMS`, Severity `HIGH`, `POC2_TEAMS` Route가 설정된 Sandbox 정책을 준비한다.
2. `POC2_Send_Security_Log_Harness`를 활성화하고 `soarpkg.SecurityInvocableLogger`의 `Send Security Log` 입력을 UI에서 준비한다.
3. 화면의 Event Key와 메시지/Details에 고유 marker를 넣어 Flow를 1회 실행한다. `SecurityInvocableLogger`의 문서화된 UI 입력에는 직접적인 `eventKey`·`idempotencyKey` 입력란이 없으므로, 이 harness에서는 고유 marker를 Details/Message에 보존한다.
4. 같은 고유 이벤트의 감사 로그 생성과 `SecurityNotifyTeamsAction` 성공을 확인한다.
5. 새 Delivery Ledger에서 `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED`를 확인한다.
6. 실제 Teams 카드 수신을 확인한다.

위 여섯 증적 중 하나라도 없으면 “정책 기반 Teams E2E 미검증”으로 유지한다. Teams 연결·Principal Access 통과 여부와 이 E2E 판정은 별도 게이트다.

### 3.1.3 실제 Outbound E2E 실행 결과

- 정책 준비: 별도 커스텀 정책 생성 UI를 확인하지 못해 기존 `EXTERNAL_THREAT_CALLBACK`을 Sandbox PoC 정책으로 재사용했다. `HIGH` 임계값을 `2`에서 `1`로 바꾸고, High 구간의 `KILL_SESSION`을 제거해 `NOTIFY_TEAMS`만 남긴 뒤 일괄 저장·새로고침으로 지속 상태를 확인했다. 이 변경은 PoC 전용 임시 설정이며 원복 여부를 정해야 한다.
- Route: `POC2_TEAMS` / `POC2_TEAMS_DEFAULT` / `TEAMS` / `IF_Teams_Base` / `NOTIFY_TEAMS + EXTERNAL_THREAT_CALLBACK + HIGH` / Active가 확인됐다. Route Health는 `READY`와 직접 POST 성공을 반환했지만 E2E 상태는 `NOT_VERIFIED`였다.
- Flow: `POC2_Send_Security_Log_Harness` V1을 저장·활성화했다. Screen의 Policy Code, Event Key, Severity, Event Message를 `Send Security Log`에 매핑했고 Event Key는 Details에 보존했다.
- 감사: Flow UI 실행 후 Dashboard에 `HIGH / EXTERNAL_THREAT_CALLBACK / APEX` 감사 행이 추가됐다. 정책 이벤트와 감사 로그 생성은 통과로 기록한다.
- Delivery Ledger: 최신 `POC2_TEAMS` 행에 `ActionType=NOTIFY_TEAMS`, 시도 `3/3`, `Status=EXHAUSTED`, 오류 `DELIVERY_FAILED`가 표시됐다. 따라서 `Channel=TEAMS`인 최종 `DELIVERED` 증적은 확보하지 못했다.
- 액션·수신: Flow에 결과 화면을 추가하지 않아 `SecurityNotifyTeamsAction`의 성공 반환값은 직접 확인하지 못했다. Ledger 실패와 Teams 카드 수신 화면 부재를 함께 고려해 O-05와 O-07은 통과 처리하지 않는다.
- Trace: 비동기 원인 확인을 위해 현재 사용자 Trace를 저장했지만 로그 행이 보이지 않았고, 후속 실행은 Salesforce 세션 만료로 로그인 화면으로 전환돼 증적에 포함하지 않았다.

따라서 이번 실행은 정책 이벤트가 감사와 Delivery Ledger 생성까지 도달했다는 점은 확인했지만, 비동기 Teams 전달 성공을 입증하지 못한 부분 실행이다. 다음 시도는 새 이벤트를 반복하기 전에 `DELIVERY_FAILED`의 비동기 실행 주체·payload·External Credential 접근 경로를 추적하는 순서로 진행한다.

공식 callback fixture는 이 Outbound 검증에 사용하지 않는다. action code/token·서명·만료·replay fixture는 Inbound·Zero-Login 전용 후속 트랙으로 별도 확보하고 검증한다.

### 남은 계약 gap

`NOTIFY_TEAMS` 계약 매니페스트의 JSON 필드를 사용한 최소·비파괴 요청을 보냈지만 공개 controller는 계속 `MISSING_ACTION_CODE`를 반환했다. `actionType`, `actionCode`, query parameter, form body, policy 변경 등 추측성 변형도 정상 callback으로 인정되지 않았다.

현재 필요한 것은 더 많은 추측이 아니라 다음 중 하나다.

- 패키지 문서/소유자가 제공하는 공식 외부 발신 payload fixture
- callback에서 요구하는 정확한 action-code와 token/서명 헤더 생성 규칙
- 정상 비파괴 `LOG_ONLY` 또는 `NOTIFY_TEAMS`를 재현하는 공식 sender 예제

이 정보가 없는 상태에서 Guest Apex 접근을 넓히거나, token·action-code를 임의 생성하거나, 파괴적 액션으로 성공 여부를 확인하는 것은 수행하지 않았다.

## 6. 발생한 막힘과 조치

| 순서 | 막힘 | 조치 | 최종 상태 |
|---:|---|---|---|
| 1 | 첫 Teams POST에서 External Credential 주체·접근 매핑 누락 | Named Principal 생성, 전용 subscriber Permission Set Principal Access와 현재 테스트 사용자 할당 | 해결; Setup UI에서 재확인 |
| 2 | 주체 매핑 후에도 managed package namespace 미허용 | Named Credential 허용 namespace에 `soarpkg` 등록 | 해결; Setup UI에서 재확인 |
| 11 | 과거 실제 정책 이벤트가 `NOTIFY_TEAMS` 이후 `EXHAUSTED` | 당시 Delivery Ledger에서 External Credential 접근 오류와 3회 재시도 확인 | 당시 정책 E2E 실패 증적으로 보존; 현재 Teams 연결 실패로 일반화하지 않고 Subscriber Flow E2E로 재검증 |
| 12 | 대상 오그 UI 세션 부재 | 연결된 브라우저에서 Salesforce 로그인 화면 확인 | 사용자 로그인 후 같은 탭에서 재개 |
| 13 | 1-Click 가이드와 기존 Named Credential 옵션 불일치 | 기존 Label과 인증 헤더 옵션이 가이드 권장값과 달랐음 | 가이드 경유 Setup에서 `IF_Teams_Base` Label 통일·인증 헤더 해제·namespace 저장 | 해결 |
| 14 | Teams Principal Access용 subscriber Permission Set을 새로 만들려는 과정에서 레이블 중복 | 목록에서 기존 `SOAR POC2 Teams Callout`을 발견했고, 해당 항목의 Principal Access·현재 사용자 할당을 확인 | 기존 권한집합 재사용; 추가 생성 없음 | 해결 |
| 15 | 신규 Permission Set 상세 API Name이 중복 문자열로 표시 | 기존 항목의 상세 화면에서 UI 입력 흔적을 확인했으나 기능에 필요한 레이블·Principal Access·할당은 정상 | 삭제·재생성하지 않고 기능 검증을 우선; 제품 UI 개선 후보 | 관찰 |
| 16 | Health Center 테스트 이벤트가 Teams Ledger를 만들지 않음 | 성공 토스트와 `WIZARD_TEST_POLICY` 감사 기록은 생성됐지만 `NOTIFY_TEAMS`·신규 Ledger 행은 없음 | 엔진 기록용 버튼과 실제 Teams POST/정책 이벤트 경로를 분리 | 후속 필요 |
| 17 | 패키지 Flow 템플릿을 직접 실행할 수 없음 | 두 템플릿이 플랫폼 이벤트 트리거·비활성·초안 상태로 표시됨 | 공식 subscriber Flow harness 없이 임의 활성화하지 않음 | 보류 |
| 18 | LWC Event Bus 발행 후 Teams Ledger 미생성 | 진입점은 `SUCCESS`였으나 새 감사 행이 `DEMO_SIMULATOR`로 기록되고 Ledger는 기존 실패 행만 유지 | Platform Event 발행 성공과 실제 정책 callout 성공을 분리; 공식 callback/Flow harness 확보 필요 | 후속 필요 |
| 19 | 정책 기반 Flow 실행 후 Teams 전달 실패 | 활성 Flow가 감사 로그와 `POC2_TEAMS` Ledger를 만들었지만 최신 행이 `EXHAUSTED`, `3/3`, `DELIVERY_FAILED`로 종료 | 직접 Teams POST 성공과 비동기 정책 액션을 분리하고 Delivery 실패의 실행 주체·payload·credential 접근을 추적 | 미해결 |
| 20 | 액션 성공·Teams 카드 증적 부재 | Flow에 결과 화면이 없어 `SecurityNotifyTeamsAction` 반환값을 노출하지 않았고, 실제 Teams 수신 스크린샷도 없음 | `DELIVERED` Ledger와 Teams 수신 화면을 모두 확보하기 전 성공 판정 금지 | 미해결 |
| 21 | 비동기 Trace 로그 미확보 | 현재 사용자 `USER_DEBUG` Trace는 저장됐으나 로그 행이 없었고 후속 실행은 Salesforce 세션 만료로 중단됨 | 새 로그인 세션에서 비동기 실행 주체까지 포함한 Trace 설정 후 단 1회 재실행 | 후속 필요 |
| 3 | Site 기본 웹 주소에 하이픈 사용 | 영숫자 slug `soarpoc2inbound`로 변경 | 해결 |
| 4 | Callback `PERMISSION_MISSING` | 사이트 선택 후 `SOAR_Inbound_Guest` 1-Click 부여 | 해결 |
| 5 | namespace 없는 공개 REST URL 404 | `/services/apexrest/soarpkg/...` 형식으로 수정 | 해결 |
| 6 | namespace 포함 요청도 `MISSING_ACTION_CODE` | REST mapping·계약을 읽기 전용으로 확인했으나 공식 fixture 없이는 해결 불가 | 후속 필요 |
| 7 | Guest action-contract 403 | 최소 권한 경계를 유지하고 추가 Apex 접근을 부여하지 않음 | 의도된 제한 |
| 8 | Slack `IF_Slack_Base` 부재 | 임의 credential을 만들지 않고 Slack·Fallback을 보류 | 후속 필요 |
| 9 | Data 이벤트 Dashboard 매핑 불일치 | 저장·검색 성공과 요약·필터 0건을 함께 기록 | 제품 개선 필요 |
| 10 | 기본 sandbox CLI 실행에서 SF 설정/로그 경로 권한 오류 | 로컬 파일 변경 없는 읽기 전용 조회를 승인된 권한으로 재시도 | 우회 완료 |

## 7. 잔여 상태와 위험 통제

- Policy Pipeline의 `STRICT` 프리셋을 실제로 적용한 상태다. 이번 Outbound 검증을 위해 기존 `EXTERNAL_THREAT_CALLBACK`의 High 임계값을 `1`로 변경하고 High의 `KILL_SESSION`을 제거해 `NOTIFY_TEAMS`만 남겼다. 이는 Sandbox PoC용 임시 변경이며 원래 값으로 원복할지 별도 결정이 필요하다.
- `POC2_TEAMS` Route는 Active 상태로 남아 있다.
- UI 검증용 `POC2_TEAMS_GUEST`, `POC2_TEAMS_SIM` Route도 Active 상태로 남아 있다.
- `SOAR POC2 Inbound` Site는 Active이며 기본 페이지는 `InMaintenance`다.
- Teams Named Credential, External Credential Principal, `soarpkg` namespace, 로컬 Callout Permission Set의 Principal Access와 현재 테스트 사용자 할당은 Setup UI에서 재확인됐다. Health Center의 직접 Teams POST도 성공했다.
- 실제 정책 기반 Teams E2E는 활성 Subscriber Flow harness에서 실행했다. 감사 로그와 새 `POC2_TEAMS` Ledger 생성까지는 확인했지만 최신 Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`로 종료됐고 `SecurityNotifyTeamsAction` 성공 및 실제 Teams 카드 수신은 확인하지 못했다.
- Inbound·Zero-Login의 공식 callback fixture(action code/token·서명·만료·replay)는 Outbound Teams E2E와 별도 트랙이며, 어느 하나가 다른 하나를 대체하지 않는다.
- `SOAR_Inbound_Guest`는 Site Guest User에 할당돼 있다.
- 파괴적 수동 액션은 승인 요청 생성까지만 수행했고 실제 실행하지 않았다.
- Slack, 외부 실패 endpoint, 주간 이메일 수신자·발송은 연결하거나 활성화하지 않았다.
- 공개 Site·Route·권한을 유지할 경우 테스트 조직의 외부 노출 범위를 계속 검토해야 한다.

## 8. 후속 실행 계획

### 8.1 Outbound Teams E2E

1. 새 로그인 세션에서 비동기 실행 주체와 `DELIVERY_FAILED` 원인을 추적할 Debug Trace를 준비한다.
2. 기존 PoC 임시 정책 변경(`EXTERNAL_THREAT_CALLBACK` High 임계값·High Action)을 원복할지 결정하고, 재실행 시에는 `HIGH ≥ 1`·`NOTIFY_TEAMS`·`POC2_TEAMS`를 다시 확인한다.
3. 활성 Subscriber Flow harness `POC2_Send_Security_Log_Harness`에서 Event Key와 Details/Message에 새 고유 marker를 넣어 1회 발행한다.
4. 감사 로그와 `SecurityNotifyTeamsAction` 성공 결과를 확인한다. 필요하면 결과 화면 또는 실행 상세 증적을 추가한다.
5. 새 Delivery Ledger의 `ActionType=NOTIFY_TEAMS`, `Channel=TEAMS`, `Status=DELIVERED`를 확인한다.
6. 실제 Teams 카드 수신 스크린샷을 확보하고 동일 실행의 Ledger와 연결한다.
7. 위 단계가 통과한 뒤에만 Retry/Fallback/최종 실패를 별도 제어 endpoint로 검증한다.

### 8.2 Inbound·Zero-Login 독립 트랙

1. 패키지 소유자 또는 공식 발신 시스템에서 action code/token·서명·만료·replay가 포함된 공식 callback fixture를 확보한다.
2. 정상 fixture를 Zero-Login Site로 전송하고 HTTP 응답·감사·정책 결과를 확인한다.
3. action code/token 누락·변조, 서명 오류, 만료, replay, malformed/rate limit을 각각 검증한다.
4. Guest가 Inbound 허용 지점 외 운영 화면·설정·파괴적 액션에 접근하지 못하는지 확인한다.

이 트랙은 Outbound Teams E2E의 대체가 아니며, Outbound 성공 여부와 별도로 완료 판정한다.

### 8.3 공통 회귀·정리

1. Data 채널 Dashboard 집계·필터 불일치를 재현하고 수정본 회귀를 실행한다.
2. Teams 수신 증적·Flow·Route·Delivery Ledger·Inbound fixture 버전을 결과서에 연결한다.
3. PoC 종료 시 Active Site·POC Route·테스트 Permission Set·Subscriber Flow harness의 유지/비활성화 여부를 결정한다.

## 9. 결론

2차 PoC는 Teams endpoint 연결, External Credential Principal Access, `soarpkg` namespace, 현재 사용자 할당, 직접 Teams POST를 통과했다. 따라서 현재 사용자에게 “Teams 연결 실패”라고 전달하지 않는다. 실제 정책 기반 Outbound E2E도 Subscriber Flow 실행, 정책 감사 로그, 새 `POC2_TEAMS` Delivery Ledger 생성까지 진행했지만 최신 Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`로 종료됐다. `SecurityNotifyTeamsAction` 성공과 실제 Teams 카드 수신은 확인하지 못했으므로 정확한 상태는 “Teams 연결과 Principal Access는 통과했지만, 정책 기반 Teams E2E는 비동기 전달 단계에서 실패했고 최종 수신은 미확인”이다. 공식 callback fixture는 Inbound·Zero-Login의 action code/token·서명·만료·replay 검증을 위한 독립 트랙이며 Outbound 검증을 대체하지 않는다.

다만 외부 Inbound의 최종 성공은 공개 controller가 요구하는 공식 action-code/token callback 형식을 확보해야 판정할 수 있다. 현재 증거로는 “공개 경로가 안전하게 도달하고 잘못된 요청을 거부한다”까지가 정확한 결론이며, 유효 callback 성공으로 확대해석하지 않는다.
