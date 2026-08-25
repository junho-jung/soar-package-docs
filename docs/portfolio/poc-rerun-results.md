# SOAR 패키지 PoC 재실행 결과서

> 이 문서는 [PoC 재실행 계획서](./poc-rerun-plan.md)에 따라 실행하면서 채우는 현재 결과서다. 과거 `poc-1-results.md`와 `poc-2-results.md`의 결과를 복사하지 않는다.
>
> 최종 원인 분류와 패키지·환경·문서 개선 권고는 [PoC 피드백](./poc-feedback.md)을 함께 본다.

## 1. 실행 정보

| 항목 | 결과 |
|---|---|
| 실행 상태 | 전체 PoC 미완료; Teams 직접 POST/헬스체크와 Site/Guest Callback 구성 `READY`는 통과했으나 정책 기반 Teams 전달은 `EXHAUSTED/3/3/DELIVERY_FAILED`로 미통과, 카드 callback·token 화면·Global 트랙은 미실행 |
| 실행일 | `2026-08-26` |
| 대상 alias | `soarInstallTest` |
| 대상 유형 | Developer Scratch Org, namespace 없음 |
| Package | `SOAR_Operations_Core_Next 0.1.0.5` |
| Subscriber Package Version ID | `04tdM000000c9OrQAI` |
| Namespace | `soarpkg` |
| Dev Hub | `JUNHO-STUDY` — Dev Hub 역할만 기록 |
| 실행 기록 | [재실행 실행 로그](../../poc-rerun-execution-log.md) |

Org ID, 사용자 이메일, Secret, 원시 URL, token, callback query, correlation 값은 이 문서에 기록하지 않는다.

## 2. 게이트 판정

| 게이트 | 결과 | 핵심 증적 | 잔여 이슈 |
|---|---|---|---|
| G0 대상·버전 | 통과 | 새 `soarInstallTest` Subscriber Scratch Org, `0.1.0.5`, `soarpkg` 확인 |  |
| G1 설치·초기화 | 부분 통과 | 패키지 설치·`SOAR_Admin` 할당·1-Click 인프라·Inbound Site/Guest 구성 완료; 정책 12개·표준 스케줄 4개 Active | Slack은 별도 미구성; 전체 Health와 Teams 채널별 판정은 분리 필요 |
| G2 핵심 회귀 | 부분 통과 | 5개 Simulator, 4개 Entry point, Dashboard·Report·Policy UI 실행; Data 요약/필터 불일치 발견 | CSV 가시적 완료 피드백·Data 매핑 원인 추가 확인 |
| G3 Teams 연결성 | 통과 | `DEFAULT_TEAMS`가 `IF_Teams_Base`와 `Match=true`; Principal Access·`soarpkg` namespace 허용 후 Health Check와 외부 채널 테스트 POST 성공(왕복 약 634ms) | Slack은 별도 미구성; Teams 카드 수신 화면은 별도 증적 필요 |
| G4 Teams 정책 E2E | 실패·미통과 | Flow V2 정책 이벤트가 Audit에 기록되고 새 Delivery Ledger가 `NOTIFY_TEAMS`로 생성됐지만 최종 `EXHAUSTED/3/3`, `DELIVERY_FAILED`; `DELIVERED`·Teams 카드 미확인 | 비동기 정책 전달 및 대상 바인딩 원인 분석 필요 |
| G5 Inbound·Zero-Login | 구성 통과·시나리오 미실행 | Site root·Guest Apex·`SOAR_Inbound_Guest`·Inbound metadata가 준비되어 Callback Health `READY`; 실제 카드 callback과 유효·만료·replay 화면은 카드 미수신으로 미실행 | 정책 Teams 전달을 먼저 복구하고 공식 fixture로 callback 화면 검증 |
| G6 Global 확장 | 미실행 | - | Describe·Subscriber harness 필요 |
| G7 정리·보고 | 부분 완료 | 현재 결과·피드백·막힘을 문서화했으나 테스트 Flow/Permission Set/Named Credential의 유지·비활성화 결정은 별도 | 잔여 테스트 자산을 유지할지 정리할지 결정 필요 |

## 3. 핵심 증적 표

| ID | 시나리오 | 결과 | 증적 위치 | 판정 |
|---|---|---|---|---|
| R0-01 | 대상 조직·패키지 버전 확인 | 새 `soarInstallTest`가 Active Developer Scratch Org이고 namespace 없음; 현재 `0.1.0.5` 기준 확인 | CLI/Installed Packages 조회 | 통과 |
| R0-02 | 기존 대상 교체 | 기존 alias 삭제 후 같은 alias로 새 Subscriber 생성 | 삭제·생성 CLI 결과 | 통과 |
| R1-01 | 설치·Admin 권한 | `SOAR_Operations_Core_Next 0.1.0.5` 설치 및 `soarpkg__SOAR_Admin` 할당 | Installed Packages, 권한 할당 결과 | 통과 |
| R1-02 | 로그인 차단 시 Chrome 진입 복구 | 일반 브라우저 세션이 로그인 화면에 머물렀으나 CLI alias 인증으로 Chrome의 인증된 Lightning 탭을 열어 재개 | `sf org list` 확인 후 `sf org open` 실행, 민감한 URL·토큰은 비기록 | 통과 |
| R1-03 | Health Center 초기 기준선 | Secret 설정 상태는 정상; Inbound Base URL·Experience Cloud Guest·`IF_Teams_Base`/`IF_Slack_Base`는 미설정 | 외부 연동 전 필수 설정으로 분리 기록 | 부분 통과 |
| R1-04 | 1-Click 인프라 활성화 | 성공 토스트 확인; 보안 정책 12개 시딩, 표준 스케줄 4개 모두 Active | 변경 후 Health Center 재조회 | 통과 |
| R2-01 | 5개 Simulator·4개 Entry point | Platform TSP, Data CDC, Logic Flow/Apex, Identity & Audit, External Signal 및 Trigger/Sec facade/REST/LWC Event Bus를 모두 UI에서 성공 응답으로 확인 | Threat Simulator UI | 통과 |
| R2-02 | Dashboard·Audit·Report 회귀 | Simulator 후 10건, Flow 실행 후 15건 조회; 검색·Severity 필터와 주간 PDF 생성/Files 저장을 확인. CSV는 버튼 실행 후 가시적 피드백 없음 | Dashboard·Audit Report UI | 부분 통과 |
| R2-03 | Data 채널 집계·필터 | Data (CDC) 요약 버튼은 0건, 필터도 0건인데 Data CDC Simulator 자체는 성공; Platform 요약 7건과 필터 4건도 불일치 | Dashboard 채널 요약·원클릭 필터 | 제품 이슈 기록 |
| R3-01 | Teams Health·Principal·namespace | 후속 설정 후 `DEFAULT_TEAMS`의 Named Credential이 `IF_Teams_Base`, `Match=true`로 확인. Health 문구에는 Slack만 누락으로 남음 | Setup & Health Center 재진단 | 통과(Teams 기준) |
| R3-02 | Teams External Credential·Named Credential | External Credential `SOAR_Teams_Webhook_POC2`와 Named Principal `SOAR_TEAMS_POC2_PRINCIPAL`을 만들고, `IF_Teams_Base`에 연결. No Authentication, Authorization Header 해제, Header/Body Merge Fields 허용, `soarpkg` namespace 허용 | Salesforce Setup UI, 외부 endpoint 원문은 미기록 | 통과(구성) |
| R3-03 | Route Preview·정책 매칭 | canonical `DEFAULT_TEAMS → IF_Teams_Base` route가 `Match=true`; Slack canonical route는 별도 미구성 | Setup & Health Center | Teams 구성 통과 |
| R3-04 | 정책 대상 Role/User 분리 | `SOAR Policy Test Subject` 역할을 CEO 아래 생성하고, 해당 역할의 활성 대상 사용자를 `Company Communities User` 프로필로 생성. 대상 사용자에게 `SOAR_Admin` 및 Teams callout 권한집합은 할당하지 않음 | Salesforce Setup UI | 통과 |
| R3-05 | Subscriber Teams Principal 권한집합 | `SOAR POC2 Teams Callout` 권한집합 생성, `SOAR_TEAMS_POC2_PRINCIPAL` Principal Access 활성화. 실행 주체 관리자에게만 할당하고 정책 대상 사용자와 분리 | Permission Set·External Credential Principal Access·Assignment UI | 통과 |
| R3-06 | Health 재진단·Delivery 기준선 | Callback은 Site/Guest 구성 후 `READY`; Teams 직접 POST는 성공했으나 정책 전달 Ledger에는 `DELIVERED`가 없음 | SOAR Health Center | 연결성과 정책 전달을 분리해 통과·실패 |
| R3-07 | Teams 직접 POST | Health Center의 `TEAMS Health Check`와 외부 채널 테스트 POST 성공 토스트 확인(왕복 약 634ms) | SOAR Health Center | 통과(연결성만) |
| R4-01 | `Send Security Log` Flow harness 구성 | Screen Flow `POC2 Send Security Log Harness` (`POC2_Send_Security_Log_Harness`)를 Chrome UI에서 생성·저장·활성화; Policy Code/Event Key/Severity/Event Message 입력과 Invocable 매핑 확인 | Flow Builder·Flow 목록 | 통과 |
| R4-02 | 정책 Audit·Action 결과 | Flow V2 제출 후 `OFF_HOURS_DATA_MUTATION`·HIGH·APEX·`Event: Unknown` 감사 행이 최신으로 추가됨. 대상 사용자 입력을 전달했지만 Audit의 대상은 실행 주체 `User User`로 표시됨 | Flow 실행 화면·Dashboard | Audit 통과, 대상 바인딩·Action 출력 미확인 |
| R4-03 | `NOTIFY_TEAMS` Delivery Ledger | 최신 정책 이벤트가 `NOTIFY_TEAMS`/`POC2_TEAMS_ACTION_E2E` 행을 만들었고 최종 상태는 `EXHAUSTED`, `3/3`, 오류 `DELIVERY_FAILED`; `DELIVERED` 없음 | Setup & Health Center | 미통과 |
| R4-04 | 실제 Teams 카드 수신 | Health 직접 POST는 성공했지만 정책 이벤트에 대한 Teams 카드 수신 화면은 확인하지 못함 | Teams 채널 | 미통과 |
| R4-05 | 정책 대상 사용자 매핑 Flow V2 | Text 입력 `Target User Id` (`Target_User_Id`)를 추가하고 Action 입력에 매핑·활성화했으나, 실행 후 Audit/Delivery correlation의 대상은 정책 대상이 아닌 실행 주체로 표시됨 | Flow Builder·Audit·Delivery Ledger | 패키지 계약/런타임 정합성 이슈로 분류 |
| R4-06 | 정상 정책 Teams E2E | 단일 정책 이벤트를 실제 제출해 Audit·Delivery 시도를 확인했으나 `Status=DELIVERED`와 카드 수신을 확보하지 못함 | Flow 실행·Health Center | 실패·재현 완료 |
| R4-07 | 대상 사용자 바인딩 | Flow V2의 `Target_User_Id`를 별도 사용자로 매핑했으나 최신 Audit/Delivery correlation은 실행 주체로 표시됨 | Flow Builder·Audit·Delivery Ledger | 계약·런타임 조사 필요 |
| R4-08 | Teams 카드 수신 | 직접 POST 성공 토스트는 확인했으나 정책 이벤트에 대한 실제 Teams 카드 수신 화면은 확인하지 못함 | Health Center·Teams | 미통과 |
| R5-01 | Retry·Fallback·최종 실패 | 정책 전달에서 제한 재시도와 `RETRYING`은 관찰했지만 제어 가능한 실패 endpoint·Fallback 선택·최종 상태 전체는 별도 시나리오로 실행하지 않음 | Delivery Ledger | 보류 |
| R6-01 | Site·Guest·Callback `READY` | Site Active, 최소 Guest Apex, `SOAR_Inbound_Guest`, Base URL·시스템 활성·Secret을 구성하고 Health `READY` 확인 | Setup & Health Center·Sites UI | 구성 통과 |
| R6-02 | 공식 Inbound 정상 fixture | 카드 미수신·공식 fixture 미확보로 미실행 | - | 보류 |
| R6-03 | 오류·만료·replay·Guest 경계 | callback URL/token을 열 수 없어 미실행 | - | 보류 |
| R6-04 | Teams 카드 action URL과 Site callback 계약 | 미실행 | 카드가 도착하지 않아 링크 host/path와 Power Automate endpoint 분리를 확인하지 못함 | 보류 |
| R6-05 | 유효 token 승인/실행 화면 | 미실행 | Chrome에서 callback 링크를 열어 유효 상태·Audit/Action Ledger를 대조하지 못함 | 보류 |
| R6-06 | 만료 token 화면 | 미실행 | 공식 만료 fixture 또는 TTL 경과 링크가 없음 | 보류 |
| R6-07 | replay token 화면 | 미실행 | 동일 callback 링크 재사용 결과를 확인하지 못함 | 보류 |
| R6-08 | 누락·변조 token 안전 거부 화면 | 미실행 | 공식 오류 fixture가 없음 | 보류 |
| R6-09 | Site root와 callback 경계 | Site root 구성은 확인했지만 카드 action URL과 REST callback의 실제 경계는 미확인 | Health Center·Teams 카드 부재 | 보류 |

### 현재 재실행 후속 결과 — Site·Guest·Teams 결정 Route

위 표의 초기/직전 결과는 과거 실행 증적이다. 아래는 2026-08-26 후속 재실행에서 추가로 확인한 현재 상태이며, 실제 Teams 카드 발송 전 기준이다.

| 테스트 | 현재 결과 | 증적 및 해석 |
|---|---|---|
| Site 도메인·Inbound Site | 통과 | Salesforce Sites 도메인 등록, `SOAR Inbound PoC` Active Site 생성, 영숫자-only prefix 규칙을 확인했다. Site 홈 페이지는 `InMaintenance`다. |
| Guest Apex 접근 | 통과 | 레포 지침의 최소 Controller `soarpkg.IF_SecurityActionController`만 공개 액세스 프로필에 추가했다. |
| Inbound Config | 통과 | `SecurityInboundConfig__mdt.Default`에 Site root를 저장하고 시스템 활성·Secret 존재를 확인했다. `Enable Webhook Signature`는 관리 화면에서 선택 해제·비활성 상태라 임의 변경하지 않았다. |
| Guest Permission | 통과 | Health Center의 패키지 제공 `1-Click 게스트 권한 부여`로 `SOAR_Inbound_Guest`를 Site Guest에 핀포인트 할당했다. |
| Callback Health | 통과 | 재진단 결과 `Callback 종합 판정 (READY)`를 확인했다. 이는 연결·구성 판정이며 유효 callback 실행 판정은 아니다. |
| Teams action Route | 통과 | canonical `DEFAULT_TEAMS`를 보존하고 `POC2_TEAMS_ACTION_E2E`를 UI에서 저장했다. `NOTIFY_TEAMS`, `TEAMS 결정`, 관리자 결정자, Dashboard fallback, Active, `Match=true`, `ROUTE_VALID`를 확인했다. |
| 실제 Teams 카드·Site callback | 미통과·후속 보류 | UI 정책 이벤트 발송은 실행했으나 임시 Route Delivery가 `EXHAUSTED/3/3/DELIVERY_FAILED`로 끝나 카드가 도착하지 않았다. 카드 action URL의 Site host/path, 유효·만료·replay 화면, 원장 대조는 미판정이다. |
| Async Apex·Trace | 진단 준비 | `SecurityDeliveryLedgerService`·`SecurityDeliveryRetryJob`는 완료 상태였고 Apex Jobs UI에 세부 예외가 없었다. `Process Automated` Trace Flag를 한시 생성했으나 로그 수집용 추가 이벤트는 아직 재확인 전이다. |
| R7-01 | Global 계약 Describe | 미실행 | - |  |
| R7-02 | Sensor/Adapter/Event/Flow | 미실행 | - |  |
| R7-03 | idempotency·권한·실패·Renderer | 미실행 | - |  |
| R8-01 | 정리·원복·잔여 상태 | 결과 문서화 완료; 테스트 Flow V2·Teams Permission Set·Named Credential은 현재 오그에 남아 있고 운영 전환/비활성화 결정은 하지 않음 | Setup·문서 | 부분 완료 |

## 3-1. 가이드에서 확인한 필요 설정값

새 Subscriber 오그의 `1-Click 설정 가이드`를 Chrome UI에서 읽기 전용으로 확인했다. 외부 URL 자체는 기록하지 않는다.

| 대상 | 필수값 | 권장 옵션 |
|---|---|---|
| Teams Named Credential | Label/Name `IF_Teams_Base`; Power Automate/Teams Webhook 전체 endpoint는 Salesforce Named Credential에만 저장 | Authentication `No Authentication`, Generate Authorization Header 해제, HTTP Header/Body Merge Fields 허용 |
| Slack Named Credential | Label/Name `IF_Slack_Base`; Slack Incoming Webhook 전체 endpoint는 Salesforce Named Credential에만 저장 | Authentication `No Authentication`, Generate Authorization Header 해제, HTTP Header/Body Merge Fields 허용 |
| Inbound Custom Metadata | `SecurityInboundConfig__mdt.Default`의 `Secret__c`, `InboundBaseUrl__c`, `IsSystemEnabled__c` | Base URL은 query/fragment 없는 HTTPS Experience Cloud Site URL |
| Guest 권한 | Experience Cloud Guest User에 `SOAR_Inbound_Guest`; 필요 시 `IF_SecurityActionController` 실행 권한 | 실제 Site 활성화 후에만 검증 |
| Route/Profile | 패키지 canonical `DEFAULT_TEAMS`/`DEFAULT_SLACK`; Named Credential Developer Name은 각 채널과 일치 | 새 PoC Route를 만들 경우 `POC_RERUN_` 접두사 사용 |
| 실행 주체 권한 | 현재 관리자 실행 사용자: `soarpkg__SOAR_Admin` + `SOAR POC2 Teams Callout`; External Credential Principal Access는 이 권한집합에서만 활성화 | 실제 Teams POST와 Flow 제출은 실행 주체로 수행 |
| 정책 대상 권한 | 역할 `SOAR Policy Test Subject` 아래 별도 활성 사용자; `Company Communities User` 프로필; Admin·Teams callout 권한집합 미할당 | Flow V2의 `Target_User_Id` 입력으로 이벤트 대상 지정 |

## 4. Teams E2E 최종 판정

### 필수 일치 조건

- [ ] Flow 결과 `isProcessed`·`statusMessage` 확인
- [x] 정책 코드·Severity·Source가 Audit에 기록
- [ ] 필요한 경우 Action Ledger 결정·승인 상태 확인
- [x] Delivery Ledger `ActionType=NOTIFY_TEAMS`
- [x] Delivery Ledger `Channel=TEAMS`
- [ ] Delivery Ledger `Status=DELIVERED`
- [ ] 실제 Teams 카드 수신 화면 확인
- [ ] Teams 카드 action URL이 Power Automate endpoint가 아닌 HTTPS Site callback을 가리킴

최종 판정: **Teams 연결성은 통과; 정책 이벤트는 Audit·Delivery 시도까지 확인했으나 `DELIVERED`·카드 수신이 없어 outbound Teams E2E는 미통과**

Health/Ping/직접 POST 성공만으로 정책 기반 Teams E2E를 통과시키지 않는다.

## 5. Inbound·Zero-Login·Site callback 화면 판정

Power Automate endpoint는 outbound 수신 주소이고, Zero-Login 승인·실행 링크를 제공하는 Site URL이 아니다. PoC에서는 두 URL을 분리해 확인해야 한다. 현재는 Site/Guest 구성을 `READY`까지 만들었지만 정책 카드가 도착하지 않아 아래 화면 검증을 수행할 수 없었다.

| 항목 | 결과 | 비고 |
|---|---|---|
| 공식 fixture 출처·버전 | 미확보 | 없으면 정상 callback 보류 |
| Site·Guest·Callback Health | 통과 | Health Center `Callback 종합 판정 (READY)`; 이는 구성 판정이며 callback 실행 판정은 아님 |
| 카드 action URL이 Site callback을 가리키는지 | 미실행 | 실제 Teams 카드 자체를 수신하지 못함 |
| 유효 token 화면 | 미실행 | Chrome에서 승인 필요/정상 처리 HTML을 확인하지 못함 |
| 만료 token 화면 | 미실행 | 공식 만료 fixture 또는 TTL 경과 후의 만료 안내를 확인하지 못함 |
| replay token 화면 | 미실행 | 동일 링크 재사용 거부와 중복 실행 방지를 확인하지 못함 |
| 정상 callback | 미실행 |  |
| action code/token 오류 | 미실행 | 안전 거부와 정상 경로 분리 |
| 서명·malformed·Guest 경계 | 미실행 |  |
| 운영 화면·Guest 경계 | 미실행 |  |

## 6. Global 계약 판정

| 항목 | 결과 | 비고 |
|---|---|---|
| Describe·접근 수준·버전 | 미실행 | 설치 버전 기준 |
| Sensor/Adapter | 미실행 | 최소 문맥 |
| Platform Event·Flow/Trigger | 미실행 | 읽기 전용 구독 |
| Invocable 결과 분기 | 미실행 | 성공·실패 분기 |
| idempotency/retry | 미실행 | 중복 티켓·알림 금지 |
| 권한 매트릭스 | 미실행 | Admin/Operator/일반/Guest |
| Renderer | 미실행 | 표현 계층만 확장 |

## 7. 막힘과 조치

| 순서 | 테스트 ID | 증상 | 원인 추정 | 조치 | 최종 상태 |
|---:|---|---|---|---|---|
| 1 | R1-02 | 새 Subscriber UI가 Salesforce 로그인 화면으로 열려 Health Center에 진입하지 못함 | CLI 인증과 Chrome 브라우저 세션이 별도 | `sf org list`로 alias 인증을 확인한 뒤 `sf org open --target-org soarInstallTest --path /lightning/n/soarpkg__SOAR_Dashboard`로 Chrome 인증 탭을 열어 재개 | 해결 |
| 2 | R1-03 | 새 조직의 초기 Health Center에서 Inbound Base URL, Guest Site, Teams/Slack Named Credential이 비어 있음 | 새 Scratch Org라 외부 연결·Experience Cloud 설정이 아직 없음 | Site·Guest·Inbound metadata를 구성하고 Health 재진단; Slack 미구성은 Teams 판정과 분리 | 해결(Teams/Inbound 기준), Slack은 별도 보류 |
| 3 | R2-03 | Dashboard의 채널 요약 숫자와 원클릭 필터 결과가 일치하지 않음; Data CDC Simulator 성공 후에도 Data 필터는 0건 | 저장 로그의 채널 분류 또는 집계/필터 쿼리 불일치 가능성 | Teams·Inbound 판정과 분리한 제품 이슈로 기록; 재현 입력과 화면 결과 보존 | 미해결 |
| 4 | R4-01 | Flow Builder의 `Send Security Log` Action 설정 화면에 문서상 반환값(`isProcessed`, `statusMessage`)이 표시되지 않음 | 현재 Subscriber Flow Builder UI에서 반환 출력 계약이 노출되지 않음 | 입력 매핑과 활성 버전을 증적화하고, 결과 출력은 미확인으로 판정 | 미해결·Global 계약에서 재확인 |
| 5 | R4-02~R4-03 | Flow 제출 후 Audit은 생성됐지만 `Event: Unknown`으로 표시되고 Delivery Ledger는 `EXHAUSTED 3/3 / DELIVERY_FAILED`로 종료 | 정책 실행 컨텍스트·대상 바인딩·비동기 callout 경로가 직접 POST 경로와 다를 가능성 | 직접 POST와 정책 E2E를 분리 판정하고, 최신 Ledger·Audit 증적을 보존 | 미해결·패키지 런타임 원인 분석 필요 |
| 6 | R3-05 | Permission Set 할당 화면의 기본 목록 보기가 삭제되었거나 접근 권한이 없다는 경고를 표시 | Salesforce Setup의 기본 list view 상태 문제 | 목록 보기에서 `최근 조회 항목`으로 전환한 뒤 실행 주체 관리자만 선택하여 할당 | 해결 |
| 7 | R4-05 | 활성 Flow 버전은 직접 수정할 수 없어 대상 사용자 입력 추가에 새 버전이 필요 | Salesforce Flow Builder의 활성 버전 편집 제한 | V2를 새 버전으로 저장하고 `Target User Id`를 Action에 매핑한 뒤 활성화 | 해결 |
| 8 | R4-05 | Flow에 분리된 대상 사용자 ID를 입력·매핑했지만 최신 Audit 대상과 Delivery correlation은 실행 주체 `User User`로 표시됨 | `SecurityInvocableLogger`의 `targetUserId` 계약이 정책 대상 바인딩에 적용되지 않거나 UI/런타임 매핑이 손실되는지 미확인 | 사용자 대상 분리 자체는 유지하고, 이 계약 불일치를 패키지 제공자 재현 항목으로 분리 | 미해결·패키지 계약/런타임 조사 |
| 9 | R3-07/R4-06 | Teams Health/직접 POST는 성공했지만 정책 비동기 행은 실패함 | 연결성 경로와 `SecurityNotifyTeamsAction` 비동기 실행 경로가 다름 | Health 성공을 E2E 성공으로 확대하지 않고 G3 통과·G4 실패로 분리 | 미해결·패키지 또는 실행 컨텍스트 |
| 10 | R4-11 | Health Center의 `테스트 보안 이벤트 발송`은 성공 토스트·Audit만 만들고 Delivery Ledger를 만들지 않음 | 고정 진단 이벤트 경로가 실제 정책 Route를 거치지 않는 것으로 보임 | 실제 정책 Flow 이벤트와 별도 증적으로 기록 | 해결·검증 규칙 보완 |
| 11 | R4-12 | 처음 만든 임시 Route의 HIGH 매칭과 `GUEST_USER_DATA_LEAK` 정책 평가 MEDIUM이 어긋남 | 정책 평가 Severity와 Route 매칭값 불일치 | 알려진 `OFF_HOURS_DATA_MUTATION`·HIGH Route로 저장 후 새로고침 검증 | 해결·사용자 설정 주의사항 |
| 12 | R4-14/R4-15 | Async Apex job은 완료됐지만 외부 callout 예외가 Apex Jobs UI에 노출되지 않음 | 패키지 전달 실패의 세부 관측성이 부족하고 기본 Trace Flag가 만료됨 | Process Automated Trace Flag와 Callout/Apex 로그 수준을 UI에서 준비; 추가 발송은 확인 후 수행 | 진단 준비·재현 필요 |

## 8. 최종 결론

재실행 결과는 현재 기준 패키지와 새 Subscriber 조직에 대한 증적만으로 판정한다. 과거 PoC의 통과·실패 상태를 현재 상태로 간주하지 않는다. 현재는 설치·권한·1-Click·핵심 UI·Flow 감사 이벤트·Teams 연결 구성·Principal/namespace·직접 Teams POST·정책 대상 역할 분리·Site/Guest Callback `READY`까지 확인했다. 그러나 최신 정책 이벤트의 Delivery Ledger가 `EXHAUSTED/3/3/DELIVERY_FAILED`이고 대상 바인딩도 실행 주체로 표시되어 `DELIVERED`·Teams 카드 수신을 확보하지 못했다. 따라서 카드 action URL이 Site callback을 가리키는지, token 유효·만료·replay 상태를 브라우저 화면에서 확인하지 않았다. 현재 판정은 G3 통과, G4 미통과, G5 구성 통과·시나리오 미실행이며 **전체 PoC는 완료로 판정할 수 없다**.
