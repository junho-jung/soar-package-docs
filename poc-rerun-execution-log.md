# SOAR 패키지 PoC 재실행 실행 로그

> 범위: `poc-rerun-plan.md`에 따른 새 Subscriber PoC 실행 기록.
>
> 보안: Org ID, 사용자 이메일, Secret, token, callback query, 원시 Webhook URL, 원본 payload와 민감한 로그는 기록하지 않는다.

> 아래 초기 실행 이력은 외부 설정 전 기준선이다. `후속 2차 PoC 실행 이력`의 R3-04 이후 행이 Teams 설정 완료 후 현재 판정의 기준이다.

## 실행 기준

| 항목 | 값 |
|---|---|
| 기준 패키지 | `SOAR_Operations_Core_Next 0.1.0.5` |
| Subscriber Package Version ID | `04tdM000000c9OrQAI` |
| Namespace | `soarpkg` |
| 대상 alias | `soarInstallTest` |
| 대상 유형 | Developer Scratch Org, namespace 없음 |
| Dev Hub 역할 | `JUNHO-STUDY` — Dev Hub |
| 계획서 | [PoC 재실행 계획서](docs/portfolio/poc-rerun-plan.md) |
| 결과서 | [PoC 재실행 결과서](docs/portfolio/poc-rerun-results.md) |

## 기록 규칙

- 한 행은 `테스트 ID → 발생 단계 → 증상 → 원인 추정 → 조치 → 결과` 순서로 남긴다.
- 성공 토스트와 실제 정책 전달을 분리한다.
- Health/Ping/직접 POST는 연결성 증적으로만 기록한다.
- `EXHAUSTED`, `DELIVERY_FAILED`, `MISSING_ACTION_CODE` 등 상태·오류 코드는 원문 의미를 보존하되 민감한 ID는 제거한다.
- 막힘이 발생하면 같은 이벤트를 반복하지 않고 원인 추적 후 재실행한다.
- 브라우저 로그인/세션이 막히면 `sf org list`로 alias 인증을 확인하고 `sf org open --target-org <alias> --path ...`로 CLI 인증 기반 Chrome 진입을 시도한다.
- 파괴적 액션은 실제 실행하지 않고 승인·버튼·원장·거절·callback 계약만 확인한다.

## 실행 이력

| 상태 | 테스트 ID | 단계 | 결과 및 막힘 | 조치 | 증적 |
|---|---|---|---|---|---|
| 완료 | R0-01 | 기존 대상 확인 | 기존 `soarInstallTest`가 활성 namespace 없는 Scratch Org임을 확인 | 삭제 대상 범위를 alias로 고정 | CLI org list 결과 |
| 완료 | R0-02 | 기존 대상 삭제 | 사용자 요청에 따라 기존 `soarInstallTest` 삭제 완료 | 같은 alias 재생성으로 진행 | CLI 삭제 결과 |
| 막힘→해결 | R0-03 | 새 Scratch Org 생성 명령 | `--edition=Developer`가 현재 CLI 허용값이 아니어서 생성 전 거부 | `--edition developer`로 수정 | CLI 오류·재시도 기록 |
| 막힘→해결 | R0-04 | 새 Scratch Org 생성 명령 | `org create scratch`가 `--no-prompt` 옵션을 지원하지 않아 생성 전 거부 | 해당 옵션 제거 | CLI 오류·재시도 기록 |
| 완료 | R0-05 | 새 대상 생성 | `JUNHO-STUDY` Dev Hub에서 동일 alias, Developer, namespace 없음, 30일 Scratch Org 생성 | 새 대상 기본 상태 검증 | CLI org list 안전 필드 |
| 완료 | R1-01 | 패키지 설치 | 현재 기준 `SOAR_Operations_Core_Next 0.1.0.5` 설치 확인 | Installed Packages 목록으로 재검증 | Package ID·namespace·버전 |
| 완료 | R1-02 | Admin 권한 | `soarpkg__SOAR_Admin` 할당 성공 | 운영자·Guest는 역할 확정 후 별도 진행 | 권한 할당 결과 |
| 완료(막힘→CLI 전환) | R1-03 | Health·1-Click·스케줄 UI 연결 | 새 오그 UI는 Salesforce 로그인 화면으로 열림; CLI 인증과 Chrome 브라우저 세션이 분리됨 | `sf org list`로 alias 인증을 확인한 뒤 `sf org open --target-org soarInstallTest --path /lightning/n/soarpkg__SOAR_Dashboard`로 Chrome 인증 탭을 열어 재개 | Chrome Health Center 진입 성공 |
| 완료 | R1-04 | Health·1-Click 인프라 | 초기 상태에서 정책 0개, 표준 스케줄 4개 미등록; Secret은 이미 설정되고 Inbound/Named Credential/Site는 미설정 | Chrome Health Center의 `1-Click 미완료 인프라 자동 활성화` 실행 후 상태 재조회 | 정책 12개 시딩, 표준 스케줄 4개 Active; 외부 연동 미설정은 잔여 상태로 보존 |
| 완료 | R2-01 | 핵심 Simulator·Entry point | 5개 Simulator 채널과 4개 Entry point를 Chrome UI에서 모두 성공 응답으로 확인 | Threat Simulator에서 채널별 실행 후 Entry point 탭을 순서대로 실행 | 성공 응답 9건; 감사 화면 기준선 확보 |
| 부분 완료 | R2-02 | Dashboard·Audit·Report | Dashboard 10건 기준선, 검색·Severity 필터, 주간 PDF 생성/Files 저장 확인; CSV는 가시적 피드백 없음 | Dashboard UI에서 필터·PDF·CSV를 각각 실행 | 10건 기준선·주간 PDF 통과, CSV 확인 필요 |
| 제품 이슈 | R2-03 | Data 채널 회귀 | Data CDC Simulator는 성공했지만 Data 요약/필터는 0건; Platform 요약 7건과 필터 4건도 불일치 | 원클릭 채널 필터와 전체 목록 결과를 분리 비교 | 매핑/집계 불일치 기록 |
| 부분 완료 | R3-01 | Teams Named Credential·Principal | 1-Click 가이드에서 `IF_Teams_Base`/`IF_Slack_Base`, No Authentication, Authorization Header 해제, Merge Fields 허용값을 확인; 실제 Named Credential·Principal은 생성하지 않음 | 설정값만 읽고 외부 endpoint 저장은 보류 | 설정값 확보, 연결성 미실행 |
| 보류 | R3-02 | Teams Health·Ping·직접 POST | `IF_Teams_Base` 미구성으로 Teams 연결성 점검 버튼을 실행하지 않음 | 외부 endpoint·Principal 설정 후 action-time 확인을 받고 1회 실행 | 미실행 |
| 부분 확인 | R3-03 | Route Preview·정책 매칭 | `DEFAULT_TEAMS`/`DEFAULT_SLACK` route가 표시되지만 두 Named Credential의 Match가 false이고 Delivery Ledger는 비어 있음 | 새 Route를 만들지 않고 canonical route 상태만 기록 | 외부 설정 대기 |
| 완료 | R4-01 | Screen Flow harness 구성 | `POC2 Send Security Log Harness` / `POC2_Send_Security_Log_Harness`를 Chrome Flow Builder에서 생성·저장·활성화; Screen 입력 4개와 `Send Security Log` 입력 매핑 완료 | `PolicyCode`, `EventKey`, `Severity`, `EventMessage`를 입력으로 만들고 Invocable 입력에 매핑 | 활성 버전 1 확인; Flow UI에는 반환 출력이 표시되지 않음 |
| 부분 완료 | R4-02 | 정상 정책 이벤트 발행 | Flow 화면에서 `OFF_HOURS_DATA_MUTATION`·HIGH 이벤트를 실행; Dashboard에 `OFF_HOURS_DATA_MUTATION`·HIGH·APEX·`Event: Unknown` 감사 로그 3건 추가 | 외부 Chrome Flow 실행 화면에서 이벤트를 제출하고 Dashboard를 새로고침 | 감사 로그 생성 확인; marker 상관관계와 정확한 1회성은 UI상 미확인 |
| 부분 완료 | R4-03 | Audit·Action Ledger | 정책 감사 로그는 확인했으나 Flow UI 반환값과 별도 Action Ledger 상태는 확인하지 못함 | Dashboard 및 Flow 화면을 확인하고, 파괴적 수동 액션은 실행하지 않음 | Audit 부분 통과, Action 미확인 |
| 보류 | R4-04 | Delivery Ledger `DELIVERED` | Setup Health Center의 최근 Delivery Ledger가 비어 있음; `IF_Teams_Base`/`IF_Slack_Base` 미구성 | 외부 Named Credential 없이 전달 성공으로 추정하지 않음 | 외부 연결 후 재검증 필요 |
| 보류 | R4-05 | Teams 카드 수신 | 실제 외부 Teams 카드 수신은 실행하지 않음 | 실제 endpoint·Principal·namespace 설정 후 action-time 확인을 거쳐 1회 실행 | 미검증 |
| 예정 | R5-01 | Retry·Fallback | - | - | - |
| 예정 | R6-01 | Site·Guest·Callback `READY` | - | - | - |
| 예정 | R6-02 | 공식 Inbound 정상 fixture | - | - | - |
| 예정 | R6-03 | 오류·만료·replay·Guest 경계 | - | - | - |
| 예정 | R7-01 | Global Describe | - | - | - |
| 예정 | R7-02 | Sensor·Adapter·Event·Flow | - | - | - |
| 예정 | R7-03 | idempotency·권한·실패·Renderer | - | - | - |
| 예정 | R8-01 | 원복·정리·결과서 | - | - | - |

### 후속 2차 PoC 실행 이력 — Teams 연결·역할 분리

| 상태 | 테스트 ID | 단계 | 결과 및 막힘 | 조치 | 증적 |
|---|---|---|---|---|---|
| 완료 | R3-04 | Teams External Credential | 외부 endpoint 원문은 문서·로그에 남기지 않고 `SOAR_Teams_Webhook_POC2` External Credential과 `SOAR_TEAMS_POC2_PRINCIPAL` Named Principal을 Salesforce UI에서 생성 | 인증 프로토콜은 No Authentication으로 구성 | Setup UI 저장·구성됨 상태 |
| 완료 | R3-05 | Teams Named Credential | `IF_Teams_Base`에 Teams External Credential 연결, Callout 허용, Authorization Header 생성 해제, Header/Body Merge Fields 허용, `soarpkg` namespace 허용 | Named Credential Developer Name을 canonical Teams route와 일치시킴 | Setup 상세·Health route match |
| 완료 | R3-06 | 정책 대상 역할·사용자 | CEO 아래 `SOAR Policy Test Subject` 역할을 만들고 별도 활성 정책 대상 사용자를 생성. 대상 사용자는 Company Communities User 프로필이며 Admin·Teams callout 권한집합을 받지 않음 | 관리자 실행 주체와 정책 대상 주체를 분리 | Role/User 상세 화면 |
| 완료 | R3-07 | Subscriber Principal 권한집합 | `SOAR POC2 Teams Callout` 권한집합 생성 후 `SOAR_TEAMS_POC2_PRINCIPAL` External Credential Principal Access를 활성화 | 실행 주체 관리자에게만 할당; 정책 대상 사용자에는 할당하지 않음 | Permission Set Principal Access·Assignment 성공 |
| 막힘→해결 | R3-08 | Permission Set 사용자 선택 | 할당 화면의 기본 목록 보기가 삭제되었거나 접근 권한이 없다는 경고로 사용자 목록이 비어 보임 | 목록 보기에서 `최근 조회 항목`으로 전환해 실행 주체 관리자만 선택 | 할당 요약 Success, 최종 할당 목록 |
| 완료 | R3-09 | SOAR Health·Route 재진단 | `DEFAULT_TEAMS → IF_Teams_Base`가 `Match=true`로 확인. Health 잔여 문구는 Slack NC 누락과 Inbound Base URL/Site 미설정이며 Teams route와 분리됨 | Callback 재진단 후 Teams Health Check·외부 채널 테스트 POST를 1회 실행 | 성공 토스트, 왕복 약 634ms |
| 완료 | R4-05 | 정책 대상 사용자 매핑 Flow V2 | 활성 Flow V1은 직접 편집할 수 없어 새 V2를 만들고 Text 입력 `Target User Id`를 추가, `Send Security Log` Action의 Target User Id에 동일 Screen 리소스를 매핑 | V2 저장·활성화 성공 토스트 확인 | Flow Builder V2, 활성 상태 |
| 실패 | R4-06 | Teams 직접 POST·정책 E2E | 활성 Flow V2에 정책 코드·고유 Event Key·HIGH·메시지·분리 대상 사용자를 입력하고 제출. Audit 행은 생성됐지만 최신 Delivery Ledger는 `NOTIFY_TEAMS`, `EXHAUSTED`, `3/3`, `DELIVERY_FAILED`로 종료 | 새 이벤트를 반복하지 않고 비동기 retry 상태와 직접 POST 결과를 분리 판정 | `DELIVERED` 미확인 |
| 확인 필요 | R4-07 | 대상 사용자 바인딩 | Flow의 `Target_User_Id` 매핑은 저장·활성화됐지만 최신 Audit 대상과 Delivery correlation은 실행 주체 `User User`로 표시됨 | 입력 계약 의미와 패키지 런타임 바인딩을 Provider 재현 항목으로 분리 | 정책 대상 Role/User 분리 자체는 유지 |
| 미통과 | R4-08 | Teams 카드 수신 | 직접 POST 성공 토스트는 확인했으나 정책 이벤트에 대한 실제 Teams 카드 수신 화면은 확인하지 못함 | `DELIVERED` Ledger와 카드 수신을 함께 확보하기 전 E2E 통과 금지 | 수신 증적 없음 |

### 현재 재실행 추가 이력 — Site/Inbound·Teams 결정 Route

| 상태 | 테스트 ID | 단계 | 결과 및 막힘 | 조치 | 증적 |
|---|---|---|---|---|---|
| 완료(확인창 문구 미노출) | R6-01 | Salesforce Sites 도메인 등록 | Salesforce Sites 약관 확인 후 등록 과정에서 브라우저 `confirm` 대화상자가 발생했으나 자동화 화면에는 확인창 유형만 노출되고 문구는 노출되지 않음 | 확인을 수락하고 등록 후 도메인 표시 상태를 재조회 | Setup Sites 화면에서 등록 상태 확인; 문구는 추정하지 않음 |
| 막힘→해결 | R6-02 | Inbound Site 생성 | Site URL prefix에 하이픈을 넣자 Salesforce가 영숫자만 허용한다는 오류를 반환 | 하이픈 없는 prefix로 수정해 `SOAR Inbound PoC`를 Active로 저장하고 홈 페이지는 `InMaintenance`로 설정 | Sites 목록·상세 화면 |
| 완료 | R6-03 | Guest Apex 경계 | Site 공개 액세스 프로필의 Apex 클래스가 비어 있었음. 레포 지침의 최소 후보 `soarpkg.IF_SecurityActionController`를 확인 | 해당 패키지 Controller만 프로필에 추가·저장; 운영 Apex/`SOAR_Admin`은 추가하지 않음 | 공개 액세스 설정의 Apex 클래스 1개 |
| 완료 | R6-04 | Inbound Custom Metadata | `SecurityInboundConfig__mdt.Default`의 시스템 활성화·Secret 존재를 확인하고 Base URL은 비어 있었음 | Site root만 입력하고 기존 Secret은 보존. `Enable Webhook Signature`는 관리 화면에서 선택 해제·비활성 상태라 변경하지 않음 | Custom Metadata 상세 화면; 원문 URL·Secret 미기록 |
| 완료 | R6-05 | 패키지 1-Click Guest 권한 | Health Center는 Site를 찾았지만 `SOAR_Inbound_Guest`가 없다고 진단 | Site를 선택한 뒤 패키지 제공 `1-Click 게스트 권한 부여` 실행 | Health 재진단 `Callback 종합 판정 (READY)` |
| 완료 | R4-09 | Teams action Route/Profile | canonical `DEFAULT_TEAMS`는 `DASHBOARD` 결정 경로였음 | 기존 route를 변경하지 않고 `POC2_TEAMS_ACTION_E2E` 임시 Route/Profile을 UI에서 저장. `NOTIFY_TEAMS`, `TEAMS 결정`, 관리자 결정자, Dashboard fallback, 활성 상태 | Route Preview `ROUTE_VALID`; 저장 후 목록 `Match=true`, `TEAMS / DASHBOARD` |
| 부분 완료→실패 | R4-10 | UI 정책 이벤트 발송 | `OFF_HOURS_DATA_MUTATION`·HIGH 정책 이벤트를 Screen Flow UI에서 제출. Audit 행은 생성되었고 임시 Route의 `NOTIFY_TEAMS` Delivery Ledger가 생성되었으나 최종 `EXHAUSTED`, `3 / 3`, `DELIVERY_FAILED` | 추가 발송은 중단하고 Audit·Delivery·Async Apex 작업을 대조 | `DELIVERED`·Teams 카드 미확인 |
| 예정 | R6-06 | Site root·callback URL 화면 | Site root가 `InMaintenance`로 설정된 상태는 확인했으나 외부 Chrome에서 root와 callback 화면을 아직 열지 않음 | 카드 URL 수신 후 Site root·REST callback을 각각 확인 | 미실행 |
| 예정 | R6-07 | 유효·만료·replay 화면 | 공식 action code/token fixture가 아직 없음 | 유효 카드 URL 1회, 동일 URL replay, 공식 만료 fixture 순으로 실행; fixture 없으면 임의 token을 만들지 않음 | 미실행 |

### 최신 UI 후속 진단 이력 — 정책 전달 실패 원인 추적

| 상태 | 테스트 ID | 단계 | 결과 및 막힘 | 조치 | 증적 |
|---|---|---|---|---|---|
| 완료 | R4-11 | Health Center 테스트 이벤트 구분 | `테스트 보안 이벤트 발송` 버튼은 성공 토스트와 Audit 행만 추가하고 `NOTIFY_TEAMS` Delivery Ledger를 만들지 않음 | 이를 실제 Teams outbound 증적으로 사용하지 않고 고정 진단 경로로 분리 | 성공 토스트·Audit 증가, Delivery Ledger 신규 행 없음 |
| 완료 | R4-12 | 정책·Route Severity 정합성 | `GUEST_USER_DATA_LEAK` 이벤트가 정책 평가에서 MEDIUM으로 관찰되었고, 처음 저장한 임시 Route의 HIGH 매칭과 어긋남 | Route를 알려진 전달 정책 `OFF_HOURS_DATA_MUTATION`·HIGH로 교체하고 목록 새로고침으로 저장값을 재확인 | Route 목록 `NOTIFY_TEAMS / OFF_HOURS_DATA_MUTATION / HIGH`, `Match=true` |
| 완료 | R4-13 | 최신 정책 Delivery 대조 | 최신 UI 이벤트는 Audit에 `OFF_HOURS_DATA_MUTATION`·HIGH·APEX로 기록되었고, 같은 Route에 `NOTIFY_TEAMS` Ledger가 생성됨 | 3회 재시도 완료 후 상태를 재조회 | `EXHAUSTED / 3 / 3 / DELIVERY_FAILED`; `DELIVERED` 없음 |
| 완료 | R4-14 | Async Apex 작업 확인 | `SecurityDeliveryLedgerService`와 `SecurityDeliveryRetryJob` 작업은 완료 상태였지만 Apex 작업 목록에 callout 예외 세부 내용이 노출되지 않음 | 성공적인 job 실행과 외부 전달 성공을 분리하고, 다음 실행의 자동 프로세스 로그를 준비 | Apex Jobs UI, 민감한 작업 ID 미기록 |
| 완료 | R4-15 | Process Automated Trace Flag | 기존 Debug Log 목록에 현재 실행을 포착할 유효 Trace Flag가 없었음. Debug Level 조회에서 `SFDC_DevConsole`을 선택하고 Process Automated 대상 Trace Flag를 30분 한시 생성 | 다음 외부 정책 이벤트 직전에 Callout/Apex 로그를 수집할 수 있도록 설정. 이번 단계에서는 추가 발송하지 않음 | Debug Logs UI의 Process Automated Trace Flag |
| 완료 | R4-16 | Callback 구성 최종 재진단 | Site root·Secret·시스템 활성·Guest 권한을 다시 진단해 `Callback 종합 판정 (READY)` 확인 | outbound Teams 실패와 Inbound callback 준비 상태를 분리 | Health Center READY 문구 |

## 이번 실행에서 확인한 추가 막힘

- 외부 Chrome의 Flow 실행 화면에서 Playwright `fill`만으로는 입력값이 화면에 유지되지 않았다. 화면 입력이 실제 DOM에 반영되는지 확인한 뒤 Chrome UI 키 입력으로 재실행했고, 이후 화면이 초기화되면서 Flow 제출이 완료됐다. 이 현상은 패키지 기능 실패가 아니라 UI 자동화 입력 경로의 차이로 기록한다.
- 현재 Flow Builder의 `Send Security Log` Action 설정에는 문서에 기술된 반환값 `isProcessed`·`statusMessage` 출력이 보이지 않았다. 따라서 화면 결과값을 성공으로 추정하지 않고 미확인으로 남겼다.
- Flow로 생성된 감사 행의 대상 리소스가 `Event: Unknown`으로 표시되어 입력 marker와 감사 행의 직접 상관관계를 화면에서 확인할 수 없었다.
- 사전 설정 단계의 Delivery Ledger는 비어 있었지만, 후속 정책 이벤트 제출 후 최신 행은 `NOTIFY_TEAMS`·`EXHAUSTED 3/3`·`DELIVERY_FAILED`로 종료됐다. 직접 POST 성공과 비동기 정책 전달 실패를 같은 결과로 합치지 않았다.
- 후속 설정 후 `DEFAULT_TEAMS`의 `IF_Teams_Base` Match는 true가 되었지만, Slack과 Inbound Site가 없어 Health 전체 상태가 완전 READY가 되지는 않았다. 이 잔여 상태를 Teams 연결 실패로 해석하지 않았다.
- Permission Set 할당의 기본 목록 보기가 유효하지 않아 `최근 조회 항목`으로 전환해야 했다. 최종적으로 실행 주체 관리자만 할당되었고 정책 대상 사용자와 분리되었다.
- 활성 Flow V1은 직접 수정할 수 없어 대상 사용자 매핑을 V2로 저장·활성화했다. V2는 정상 제출됐고 Audit/Delivery 시도까지 확인했지만 `DELIVERED`와 Teams 카드 수신은 확인하지 못했다.
- 사용자 승인 후 Health 직접 POST와 정책 Flow 제출을 각각 1회 실행했다. 이후 파괴적 액션이나 추가 반복 이벤트는 실행하지 않았다.
- Flow에 입력한 분리 대상 사용자와 달리 Audit/Delivery correlation이 실행 주체로 표시되어 `targetUserId` 계약/런타임 불일치를 별도 기록했다.
- 현재 재실행에서는 문서의 Inbound 절차에 따라 Site 생성 → Guest Apex 최소 접근 → `SecurityInboundConfig__mdt.Default` Base URL → 패키지 1-Click Guest 권한 → Callback Health 재진단 순서로 진행했고 `READY`를 확인했다.
- Site 도메인 등록 확인창의 정확한 문구는 브라우저 자동화 표면에서 얻지 못했다. 확인창을 보지 못했다고 기록하지 않고, 확인 수락 후 도메인 등록 완료라는 사후 상태만 증적으로 남겼다.
- Site Guest 사용자는 일반 사용자 검색에 나타나지 않았지만 Health Center가 Site와 Guest User를 탐지했고, 패키지 제공 1-Click으로 `SOAR_Inbound_Guest`를 핀포인트 할당했다.
- `IF_SecurityActionController`는 레포에서 요구하는 최소 공개 Apex 접근으로 확인했다. Inbound 내부 구현 클래스 전체를 게스트 프로필에 공개하지 않았다.
- Outbound Teams와 Inbound Site URL을 분리했다. Teams Named Credential는 Power Automate endpoint를 outbound로 사용하고, Inbound Base URL은 별도 Salesforce Site root로 설정했다.
- Teams 결정 Route는 canonical `DEFAULT_TEAMS`를 덮어쓰지 않는 임시 Route로 저장했다. 실제 카드 발송·링크 host·callback 화면은 아직 결과를 만들지 않았다.
- 최신 Route는 화면 새로고침 후 `NOTIFY_TEAMS / OFF_HOURS_DATA_MUTATION / HIGH`, `Match=true`로 재확인했다. 초기 `GUEST_USER_DATA_LEAK`·HIGH 설정은 실제 정책 평가 MEDIUM과 맞지 않아 정합성 문제로 기록했다.
- Health Center의 고정 테스트 이벤트는 Audit만 만들고 Delivery Ledger를 만들지 않았다. 이를 실제 Teams E2E로 오인하지 않았다.
- 최신 정책 Delivery에 대해 Async Apex 작업은 완료됐지만 예외 세부 내용은 Apex Jobs UI에 노출되지 않았다. Process Automated Trace Flag와 Callout/Apex 세밀 로그 수준을 다음 재현용으로 설정했으며, 설정 직후 외부 발송은 반복하지 않았다.
- 최신 정책 이벤트는 `NOTIFY_TEAMS`·임시 Route Ledger까지 도달했지만 `EXHAUSTED / 3 / 3 / DELIVERY_FAILED`로 끝났다. 따라서 Teams 연결 실패가 아니라 정책 기반 비동기 전달 실패로 분류한다.
- Site/Guest/Inbound는 문서 흐름대로 구성되어 Callback Health `READY`까지 확인했다. 다만 카드가 전달되지 않아 Site callback URL·유효/만료/replay 화면은 아직 열지 못했다.
- Dashboard의 Data 채널 집계/필터 불일치는 핵심 UI 회귀의 제품 이슈로 남겼고, Teams outbound 및 Inbound·Zero-Login 판정과 섞지 않았다.

### 최신 Chrome UI 재현 이력 — 정책 이벤트·관측성 후속

| 상태 | 테스트 ID | 단계 | 결과 및 막힘 | 조치 | 증적 |
|---|---|---|---|---|---|
| 부분 완료→실패 재확인 | R4-17 | Screen Flow UI 정책 이벤트 재현 | Lightning의 Flow 실행 버튼이 로그인 화면으로 이동해 CLI alias 인증으로 Visualforce Flow runtime을 다시 열었다. Chrome UI에서 `OFF_HOURS_DATA_MUTATION`·HIGH·Event Key `POC2_TEAMS_TRACE_UI_20260826_01`·비파괴 메시지·분리 대상 사용자를 입력해 제출했다. Audit에는 새 행이 생겼지만 대상은 다시 실행 주체 `User User`, 리소스는 `Event: Unknown`으로 표시됐다. | 로그인 차단 규칙에 따라 `sf org open --target-org soarInstallTest --path ...`로 인증된 Chrome 화면을 복구하고, 입력값이 보이는 상태에서 `다음`을 1회 실행 | Dashboard Audit 최신 행; Flow 화면 초기화 후 제출 완료 |
| 완료 | R4-18 | Delivery·Async 재조회 | 새 이벤트의 `POC2_TEAMS_ACTION_E2E` Ledger는 처음 `RETRYING 2/3`으로 보였고 전체 Dashboard 재조회 후 `EXHAUSTED 3/3 / DELIVERY_FAILED`가 됐다. `SecurityDeliveryLedgerService`·`SecurityDeliveryRetryJob`은 완료 상태였으나 Apex Jobs에 상세 외부 예외는 없었다. Teams 카드와 Site callback URL은 수신되지 않았다. | 재시도 상태가 끝날 때까지 추가 이벤트를 만들지 않고 Audit·Ledger·Async Jobs를 분리 대조 | Setup & Health·Async Apex Jobs·Chrome 탭 목록 |
| 막힘 | R4-19 | Process Automated Trace 재설정 | 이전 Process Automated Trace Flag는 이미 만료되어 이번 이벤트의 Debug Log가 생성되지 않았다. 새 Trace Flag 화면에서 자동 프로세스 대상을 선택하고 전용 Debug Level 조회를 시도했으나 Salesforce 조회 팝업이 별도 Chrome 탭으로 열려 부모 폼에 선택값을 전달하지 못했고, 저장 시 `디버그 수준: 일치하는 항목이 없습니다`가 표시됐다. 새 Debug Level 이름은 재제출 시 중복 오류가 반환되어 존재 상태를 확인했지만, Trace Flag 자체 저장은 확정하지 않았다. | 추가 외부 Teams 이벤트는 실행하지 않고 팝업 선택 전달 문제를 진단 장애로 기록 | Debug Logs UI·Debug Level UI; 민감한 ID/토큰 미기록 |

이번 재현의 최종 증거는 **Teams 연결성 통과 + 정책 기반 비동기 전달 실패 재확인**이다. `DELIVERED`, 실제 Teams 카드, 카드 action URL, Site callback의 유효·만료·replay 화면은 여전히 만들지 못했다.

## 보안 점검

- [x] 원시 Webhook URL을 기록하지 않음
- [x] Secret·token·callback query를 기록하지 않음
- [x] 사용자 이메일·Org ID·원본 payload를 기록하지 않음
- [x] 관리자 계정을 파괴적 액션 대상에 지정하지 않음
- [x] 파괴적 액션을 실제 실행하지 않음
- [ ] 테스트 종료 시 임시 Route·Site·Flow·권한·정책 상태를 확인함
