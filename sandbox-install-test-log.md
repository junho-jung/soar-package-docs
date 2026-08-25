# SOAR Sandbox 설치 테스트 로그 — 역사 기록

> 범위: `soar-package-docs` README와 연결 문서에 기재된 설치 절차를 기준으로 진행한 `soarInstallTest` Scratch Org 도입 테스트 기록입니다.
>
> 문서 성격: 2026-08-23 패키지 `0.1.0.1`의 과거 실행 로그입니다. 새 PoC는 [재실행 계획서](docs/portfolio/poc-rerun-plan.md)와 [재실행 실행 로그](poc-rerun-execution-log.md)를 사용하며 이 파일에 이어 쓰지 않습니다.
>
> 보안: 토큰, 비밀번호, 일회성 로그인 URL, Org ID, 사용자명, 원본 로그는 기록하지 않습니다. 이 파일은 로컬 작업 기록이며 커밋하지 않습니다.

## 테스트 기준

- 패키지: `SOAR_Operations_Core_Next 0.1.0.1`
- Subscriber Package Version ID: `04tdM000000byy5QAA`
- Namespace: `soarpkg`
- 대상 alias: `soarInstallTest`
- 생성 Dev Hub: `JUNHO-STUDY`
- 기준 문서: `README.md`, `docs/user/installation-and-setup.md`, `docs/user/operations.md`, `docs/user/troubleshooting.md`, `docs/extensions/README.md`, `docs/portfolio/validation.md`

## 진행 이력

| 상태 | 단계 | 결과 및 막힘 | 조치 |
|---|---|---|---|
| 완료 | 로컬 Org 역할 확인 | `SOAR-PKG`는 `soarpkg` 네임스페이스 조직이며 Sandbox가 아니었음 | Subscriber 테스트 대상에서 제외 |
| 완료 | 초기 대상 alias 지정 | alias 없는 활성 Subscriber Scratch Org에 `soarInstallTest`를 지정 | 기존 alias는 변경하지 않음 |
| 막힘→해결 | 설치 전 인증 확인 | `soarInstallTest` 조회 시 `refresh token` 인증 실패 | 웹 로그인 흐름을 시도했으나 CLI 콜백이 완료되지 않음 |
| 완료 | 기존 테스트 Org 제거 | 인증이 불안정한 기존 `soarInstallTest` Scratch Org 삭제 | 사용자 요청에 따라 삭제 |
| 완료 | 새 테스트 Org 생성 | `JUNHO-STUDY` Dev Hub에서 Developer Edition, namespace 없음, 2GP ancestor 없음, 30일 수명으로 생성 | 동일 alias `soarInstallTest` 유지 |
| 완료 | 새 Org 사전 검증 | Developer Edition Scratch Org, namespace 없음, 설치 패키지 0개 확인 | 설치 진행 가능 판정 |
| 완료 | 패키지 설치 | README의 `04tdM000000byy5QAA` 설치 성공 | `SOAR_Operations_Core_Next / soarpkg / 0.1.0.1` 확인 |
| 완료 | 관리자 권한 할당 | `soarpkg__SOAR_Admin` 할당 성공 | 운영자·게스트 권한은 아직 할당하지 않음 |
| 완료 | 메타데이터 기본 검증 | namespaced Apex 129개, LWC 7개, SOAR 권한 집합 3개 확인 | 설치 산출물 노출 확인 |
| 막힘→우회 | SOAR 화면 첫 URL | `/lightning/page/soarpkg__SOAR_Dashboard` 접근 시 `페이지 없음` 표시 | 화면에 노출된 실제 탭 경로 `/lightning/n/soarpkg__SOAR_Dashboard` 확인 |
| 완료 | SOAR Security Hub 화면 | 올바른 탭 경로에서 대시보드·위협 모의 시뮬레이터·정책 빌더·Setup & Health Center가 화면에 렌더링됨 | 화면 검증 진행 |
| 확인 필요 | Setup & Health Center | 서명 토큰은 정상이나 Inbound Base URL과 Teams/Slack Named Credential이 미설정 | 외부 URL·인증정보 없이 보류 |
| 확인 필요 | 권한·Sites 상태 | `SOAR_Admin`은 정상, Sites Guest User는 없음 | Zero-Login을 테스트할 때만 별도 구성 |
| 확인 필요 | 정책·스케줄 상태 | 보안 정책 0개, 대응 액션 9개; 표준 스케줄 4개 모두 미등록 | 기본 정책/필요 스케줄 초기화 전 상태로 기록 |
| 완료 | 1-Click 자동 활성화 | `SOAR 인프라 1-Click 자동 설정 완료` 토스트 확인; 신규 2건 반영, 기존 3건 보존 | Inbound 외부 URL/Named Credential은 그대로 미설정 상태로 보존 |
| 완료 | 표준 스케줄 자동 등록 | 4개 표준 작업이 모두 `정상 가동 중 (Active)`로 변경되고 다음 실행 시각이 표시됨 | 감사 아카이브·AuditTrail 검사·임시 로그 정리 스케줄 활성화 |
| 확인 필요 | 주간 ISMS-P/SOC2 보고서 구독 | 구독 토글은 `미등록 (INACTIVE)`로 유지 | 수신자/메일 발송은 외부 알림 기능이므로 임의 활성화하지 않음 |
| 막힘→우회 | 감사 로그 행 작업 버튼 자동 선택 | `작업 표시` 이름이 열 머리글 작업 버튼까지 8개로 매칭되어 strict-mode 오류 | `exact: true`로 행 작업 버튼을 다시 식별 |
| 완료 | 감사 로그 검색·위험도·응답 메뉴 | 테스트 이벤트 2건 확인; 정책 코드 검색은 1건으로 좁혀졌고, 행 메뉴에서 4개 승인 기반 대응 액션 확인 | `세션 종료` 승인 요청을 1건 생성했으나 실제 세션 종료는 실행하지 않음 |
| 완료 | 감사 리포트 CSV/PDF | CSV 다운로드 버튼 동작 확인; 주간 PDF 생성 후 Salesforce Files 저장 완료 토스트 확인 | 현재 필터 상태로 내보내짐; 월간 보고서는 별도 생성하지 않음 |
| 막힘→우회 | PDF 완료 알림 닫기 | 완료 토스트가 자동 소멸되어 `닫기` 버튼을 찾지 못함 | 알림 닫기 없이 다음 탭으로 이동 |
| 완료 | Threat Simulator 5대 채널 | Platform TSP, Data CDC, Logic Flow/Apex, Identity & Audit, External Signal 시나리오를 각각 1회 실행; 모두 `모의 발송 정상 완료` | 모의 이벤트이므로 실제 업무 데이터/외부 채널은 변경하지 않음 |
| 완료 | 4대 엔트리포인트 검증 | Trigger, Sec Facade API, Inbound REST API, LWC Event Bus 버튼을 각각 실행; 모두 성공 알림 확인 | 마지막 결과는 `LWC Platform Event Published` 성공 |
| 막힘 | Teams Webhook 연결 테스트 | `IF_Teams_Base` Named Credential이 없어 callout endpoint 접근 실패 | 실제 Teams URL/Named Credential을 임의 생성하지 않고 중단 |
| 확인 필요 | 수동 액션 콘솔 | `FORCE_MFA_RESET`, `FREEZE_USER`, `KILL_SESSION`, `REVOKE_TOKEN`만 제공되어 모두 대상 사용자에 영향을 주는 대응 액션 | 테스트 사용자·승인 없이 실행하지 않음 |
| 막힘 | Webhook 실시간 핑(Teams) | HTTP 500; `IF_Teams_Base` Named Credential 부재로 callout 접근 실패 | 외부 endpoint 없이 중단 |
| 막힘 | Webhook 실시간 핑(Slack) | HTTP 500; `IF_Slack_Base` Named Credential 부재로 callout 접근 실패 | 외부 endpoint 없이 중단 |
| 완료 | Policy Pipeline Builder | 12개 시그널·12개 정책·9개 조치 노출; `MASS_DATA` 검색, Data 채널 필터, 미할당 0건 확인 | 첫 정책의 허들 편집 화면을 열어 기본 임계치 확인 후 취소 |
| 완료 | 정책 Drift/프리셋 | Drift 미리보기 후 STRICT 일괄 적용 성공 토스트 확인; 커스텀 정책 보존 안내 확인 | BALANCED 미리보기는 취소하고 최종 상태는 STRICT로 유지 |
| 막힘→우회 | 미할당 토글 접근성 이름 | 토글 전환 후 accessible name이 `...전체시그널`에서 `...미할당만`으로 변경되어 기존 selector 불일치 | 현재 상태명을 재조회해 토글 복귀 |
| 막힘→우회 | Route Preview 키 입력 자동 선택 | `Route Key`가 선택적 `Fallback Route Key`까지 2개로 매칭됨 | `exact: true`로 필수 Route Key 입력란 재선택 |
| 완료 | Notification Route/Delivery Ledger 점검 | 기본 TEAMS/SLACK route가 노출되고 `Route Preview`는 빈 key에서 `ROUTE_INVALID`, 유효한 샌드박스 key에서 `ROUTE_VALID_PENDING_NC` 반환 | Named Credential 미설정으로 delivery ledger는 비어 있음; 테스트 route는 저장하지 않음 |
| 완료 | Flow 템플릿 확인 | Salesforce Flow 목록에서 관리됨-설치됨 상태의 `[SOAR 템플릿] VIP 데이터 접근 이상 알림 플로우`, `[SOAR 템플릿] 보안 조치 승인 에스컬레이션 플로우` 확인 | 사내 전용 복제/활성화는 추가 운영 설계가 필요하므로 실행하지 않음 |
| 확인 필요 | 대시보드 채널 필터 | `MASS_DATA_DELETION` 감사 행은 존재하지만 요약 카운터가 Data 0이며 Data 필터 선택 시 0건으로 숨겨짐 | 이벤트 저장은 성공, 채널 집계/필터 매핑 불일치로 기록 |
| 완료 | 대시보드 실시간/Fullscreen | LIVE STREAMING→STREAM PAUSED→LIVE STREAMING 전환 및 Fullscreen SOC Mode→일반 뷰 복귀 확인 | UI 상태 전환 정상 |
| 확인 필요 | Health Center Callback 재진단 | `CONFIGURATION_MISSING` 유지: 서명 키는 정상, HTTPS Inbound Base URL/Site 미설정 | Experience Cloud URL 없이는 인바운드 실검증 불가 |
| 막힘 | Health Center Teams/Slack POST 점검 | Teams·Slack 모두 `CONFIGURATION_MISSING`으로 Named Credential 부재 표시 | 외부 URL/credential 없이 중단 |
| 완료 | 표준 스케줄 재등록 | 두 번째 1-Click 실행도 4개 Active 유지; `총 4개의 ... 정상 등록` 알림 확인 | 중복 스케줄 생성 없이 idempotent 동작 확인 |
| 막힘→해결 | 정책 허들 일괄 저장 | Medium을 2로 올리고 High 2/Critical 3으로 저장 시 `thresholds must be positive and strictly increasing` 오류 | 원래 값 M1/H2/C3으로 복원 후 재저장 예정 |
| 완료 | 정책 허들·일괄 저장 | `MASS_DATA_EXPORT` 허들을 M1/H2/C3으로 원복하고 `전체 시그널, 보안 정책 및 허들 구성 ... 저장` 성공 확인 | 테스트 변경은 원래 정책으로 되돌림 |
| 완료 | AUDIT 프리셋 미리보기 | AUDIT_ONLY Drift 적용 미리보기 확인 후 취소; 최종 선택은 STRICT | 모니터링 전용 모드로 정책을 약화시키지 않음 |
| 완료 | 중복 시나리오 재실행 | 동일 `MASS_DATA_EXPORT` 시나리오를 즉시 재실행; 감사 로그에 severity별 누적/행이 반영됨 | 시뮬레이터가 새 이벤트를 생성하므로 외부 전달 dedupe 자체는 별도 확인 필요 |
| 막힘→우회 | 월간 PDF 라디오 선택 | semantic radio click 후 Weekly가 유지됨 | 화면 좌표로 Monthly 라디오를 선택해 우회 |
| 완료 | 주간·월간 감사 PDF | Weekly와 Monthly 모두 Salesforce Files 저장 완료 토스트 확인 | 외부 메일 발송은 하지 않음 |
| 막힘→우회 | 대시보드 검색 초기화 | 빈 문자열 `fill`만으로는 검색 상태가 유지됨 | 화면의 `지우기` 버튼으로 15건 전체 로그 복귀 |
| 막힘→우회 | STRICT 재선택 | STRICT 버튼도 정책 Diff 확인 모달을 다시 열어 Threat Simulator 탭 전환이 차단됨 | Diff 모달 `취소` 후 탭 이동 |

## 최종 미해결·보류 항목

1. `IF_Teams_Base`와 실제 Teams endpoint, External Credential Principal, `soarpkg` namespace 허용은 구성됐고 직접 POST도 성공했지만, 정책 기반 비동기 Teams 전달은 `EXHAUSTED/3/3/DELIVERY_FAILED`로 끝남. `IF_Slack_Base`는 없어 Slack 전달·Fallback은 보류
2. Experience Cloud Site와 HTTPS Inbound Base URL은 구성됐고 Guest 권한·Health `READY`까지 확인했지만, 공식 action code/token callback fixture가 없어 정상 Inbound·Zero-Login 실행은 미검증
3. 주간 이메일 구독/샘플 발송은 외부 수신자에게 메일을 보낼 수 있어 실행하지 않음
4. 수동 액션 콘솔의 `FORCE_MFA_RESET`, `FREEZE_USER`, `KILL_SESSION`, `REVOKE_TOKEN`은 테스트 사용자·이중 승인 없이 실행하지 않음
5. `MASS_DATA_DELETION` 감사 행이 저장되었지만 Dashboard의 Data 채널 집계가 0으로 표시되고 Data 필터에서 숨겨지는 매핑 불일치가 남음
6. 시뮬레이터 재실행은 감사 누적을 확인했으나, 실제 외부 전달 dedupe/retry/rollback은 외부 설정과 승인 경계 때문에 별도 운영 검증 필요

## 최종 상태

- Sandbox 패키지 설치·Admin 권한·1-Click 인프라 초기화·12개 정책·9개 액션·4개 표준 스케줄을 확인함
- 5대 채널 시뮬레이션, 4대 엔트리포인트, 대시보드/감사/검색/필터/CSV/PDF, Policy Drift/STRICT, Route Preview, Flow 템플릿을 실행함
- 테스트 대상은 정상 상태로 유지하고, 공개 Site는 Inbound 전용 `InMaintenance` 상태로 두었으며 파괴적 사용자 액션은 생성/실행하지 않음. Teams 정책 검증을 위해 임시로 바꾼 정책 임계값·High Action 원복 여부는 후속 결정 필요

## 2차 PoC 실행 추가 기록

| 상태 | 단계 | 결과 및 막힘 | 조치 |
|---|---|---|---|
| 완료 | Teams endpoint 보안 연결 | 사용자 제공 Power Automate endpoint를 원문 로그에 남기지 않고 `IF_Teams_Base` Named Credential에 연결 | 인증 안 함 External Credential을 만들고 외부 URL은 Named Credential에만 저장 |
| 막힘→해결 | 첫 Teams 실제 POST | `CALLOUT_FAILED`; `IF_Teams_Base`를 찾았지만 External Credential 주체/접근 매핑이 없어 호출 완료 불가 | Named Principal 생성, 전용 `SOAR POC2 Teams Callout` Permission Set에 Principal Access 매핑, 현재 테스트 사용자에게 할당 |
| 막힘→해결 | 두 번째 Teams 실제 POST | 주체 매핑 후에도 `CALLOUT_FAILED`; managed package callout 허용 namespace가 비어 있었음 | Named Credential의 허용 namespace에 `soarpkg` 입력 |
| 완료 | Teams 실제 POST 재검증 | `READY`; Teams 외부 채널 테스트 POST 성공, 왕복 약 566ms | Teams 수신 화면 증적은 사용자 스크린샷 대기 |
| 확인 필요 | Slack 회귀 | `IF_Slack_Base` Named Credential이 없어 Slack Health/POST는 계속 보류 | Slack credential 없이 임의 생성하지 않음 |
| 막힘→해결 | Salesforce Sites 도메인·사이트 생성 | 기본 웹 주소에 하이픈을 넣어 저장하자 `기본 웹 주소는 영숫자여야 합니다.` 오류 발생 | 영숫자 slug `soarpoc2inbound`로 수정하여 Active 사이트 생성 |
| 완료 | Inbound 설정값 입력 | `SecurityInboundConfig__mdt.Default`에 테스트 사이트 기반 HTTPS Inbound Base URL을 저장하고 시스템을 활성화 | 실제 사이트 호스트와 생성 Secret은 이 로그에 기록하지 않음 |
| 막힘→해결 | Callback 게스트 권한 진단 | Health Center가 `PERMISSION_MISSING`으로 탐지된 Site Guest User의 `SOAR_Inbound_Guest` 부여 누락을 표시 | 사이트를 선택하고 `1-Click 게스트 권한 부여` 실행 |
| 완료 | Callback 재진단 | `READY`; 시스템·Secret·HTTPS URL·Site Guest 구성요소 준비 완료 | Site Guest에 패키지 제공 `SOAR_Inbound_Guest` 권한 집합이 핀포인트 할당됨 |
| 막힘→우회 | Salesforce CLI 읽기 전용 계약 조회 | 기본 실행 환경에서 SF CLI 로그/설정 경로 권한 오류로 조회 실패 | 로컬 파일을 변경하지 않는 읽기 전용 재시도를 승인된 권한으로 수행 |
| 완료 | Apex REST 계약·라우팅 확인 | `SecurityActionContractRestApi` 매니페스트와 `IF_SecurityActionController`의 `/api/security/action` 매핑을 확인 | 전역 입력 필드와 `NOTIFY_TEAMS` 계약 메타데이터를 정리 |
| 막힘→해결 | 외부 REST 경로 1차 호출 | namespace 없는 공개 경로는 HTTP 404 `NOT_FOUND` | 관리 패키지 namespace를 포함한 경로로 수정 |
| 확인 필요 | 외부 REST 유효 callback | namespace 포함 공개 경로는 도달했으나 추정 payload가 HTTP 400 `MISSING_ACTION_CODE`로 거부됨 | 파괴적 액션을 실행하지 않고, 공식 callback action-code/token fixture 확보 전까지 보류 |
| 완료 | 외부 REST 안전 거부·권한 경계 | malformed 요청은 400, 게스트의 action-contract 조회는 403으로 거부됨 | 게스트에 계약 Apex 접근을 추가하지 않고 최소 권한 유지 |
| 완료 | Zero-Login 공개 진입점 | 공개 Site root가 `InMaintenance` 기본 페이지로 열려 게스트 진입 및 기본 페이지 상태 확인 | Inbound 전용 사이트이므로 운영 화면은 노출하지 않음 |
| 확인 필요 | Inbound 서명·만료·replay·rate limit | 유효 callback fixture가 확정되지 않아 정상 처리 이후의 오류 행렬은 실행하지 못함 | 공식 발신 형식과 서명 기능 활성화 가능 여부 확인 후 재실행 |
| 완료 | 2차 PoC 잔여 위험 통제 | 수동 파괴적 액션, Slack credential, 실패 endpoint, 외부 메일은 실행하지 않음 | 승인 경계와 외부 시스템 범위를 넘지 않도록 테스트 종료 |
| 확인 필요 | 시스템 진단 원클릭 범위 | 설정 가이드에서 원클릭은 Named Credential 안내·정책/액션 씨딩·표준 스케줄 설정을 지원하며 실제 외부 정책 액션을 직접 발송하지 않음을 확인 | 원클릭 반복 실행 대신 실제 이벤트 진입점 검증으로 전환 |
| 완료 | UI 정책 Route 추가 | `POC2_TEAMS_GUEST`와 `POC2_TEAMS_SIM`을 UI에서 각각 저장하고 `ROUTE_VALID` 확인 | `GUEST_USER_DATA_LEAK` 및 `LOGIN_BRUTE_FORCE_BURST`를 `NOTIFY_TEAMS`와 연결; 원시 URL은 입력하지 않음 |
| 확인 필요 | UI 시뮬레이터 실제 전달 | `LOGIN_BRUTE_FORCE_BURST` 시나리오의 `모의 발송 정상 완료` 토스트는 확인했지만 `Delivery Ledger`는 계속 비어 있음 | 모의 발송은 외부 callout 증적이 아니며 UI-only 실제 액션 통과로 판정하지 않음 |
| 확인 필요 | UI Teams Webhook 탭의 액션성 | Teams 테스트 탭은 고정 MessageCard 전파 버튼과 JSON 프리뷰만 제공하고 정책 이벤트·인터랙티브 액션 버튼은 제공하지 않음 | Health/연결테스트와 실제 정책 액션을 별도 증적으로 분리 |
| 확인 필요 | 사용자 테스트 진입점 | 패키지 UI에는 `NOTIFY_TEAMS`만 단독으로 실행하는 비파괴 수동 버튼이 없고 수동 액션 콘솔은 파괴적 액션만 노출 | UI 기반 실제 검증은 공식 외부 callback 또는 Flow `Send Security Log` 진입점으로 후속 진행 |
| 확인 필요 | 탐색용 직접 계약 호출 | 공개 `Sec.evaluateEvent` 계약을 비파괴 정책으로 탐색했으나 사용자 UI 증적으로는 채택하지 않음 | UI-only 판정·스크린샷·Delivery Ledger를 최종 증적으로 사용 |
| 사용자 증적 반영 | 실제 보안 이벤트의 정책 진입 | 사용자가 제공한 실행 결과에서 실제 이벤트가 `NOTIFY_TEAMS`까지 도달하고 `Delivery Ledger` 행이 생성됨 | 앞서 시뮬레이터에서 Ledger가 비어 있던 기록과 분리하여, 실제 이벤트 경로의 증적으로 추가 |
| 막힘 | 실제 Teams 전달(초기 증적) | `Delivery Ledger`의 Teams 전송이 External Credential 접근 오류로 3회 재시도 후 `EXHAUSTED` 상태가 됨 | 후속 Setup UI에서 Principal Access·`soarpkg` namespace·직접 POST는 재확인했으나 정책 Flow의 최신 Ledger도 `EXHAUSTED/DELIVERY_FAILED`로 종료됨 |
| 진행 보류→재개 | UI 재검증 세션 | 대상 오그가 브라우저에서 로그인 화면으로 열려 설정 화면 접근이 일시 중단됨 | 사용자 로그인 후 같은 탭에서 Principal·namespace·Flow harness·정책 실행을 재개 |
| 확인→진행 | 보안 허브 1-Click 설정 가이드 | 가이드가 Named Credential을 자동 생성하지 않고 필수 Label/Name, endpoint 예시, 인증 옵션을 안내한 뒤 Salesforce Setup으로 이동시킴 | README의 설치 흐름에 맞춰 가이드 경유로 기존 `IF_Teams_Base`를 수정 |
| 완료 | 가이드 규격과 기존 Teams Named Credential 정합화 | 기존 Label이 `SOAR Teams POC2`, 인증 헤더 생성이 체크 상태였음 | Label을 `IF_Teams_Base`로 통일하고 인증 헤더 생성을 해제, `soarpkg` 허용 namespace는 유지 후 저장 성공 |
| 막힘→확인 | Teams Principal subscriber 권한집합 생성 시도 | 새 권한집합 저장 화면에서 `SOAR POC2 Teams Callout` 레이블 중복 오류가 발생했고, 상세 화면에는 API Name이 중복 문자열로 표시됨 | 목록의 `S` 필터에서 기존 `SOAR POC2 Teams Callout`을 찾아 재사용; 삭제·재생성하지 않음 |
| 완료 | Teams Principal Access·사용자 할당 재확인 | 기존 `SOAR POC2 Teams Callout`의 `외부 자격 증명 주체 액세스`에 `SOAR_Teams_Webhook_POC2 - SOAR_TEAMS_POC2_PRINCIPAL` 행이 존재하고 현재 테스트 사용자 할당도 `활성`으로 확인 | 별도 권한집합을 추가 생성하지 않고 기존 subscriber Permission Set을 유효 설정으로 채택 |
| 완료→막힘 | 실제 Teams 정책 이벤트 재검증 | Health Center에서 `IF_Teams_Base`·활성 Teams Route를 재확인하고 UI Flow에서 비파괴 정책 이벤트를 실행 | 새 `POC2_TEAMS` Ledger는 생성됐으나 `EXHAUSTED/3/3/DELIVERY_FAILED`; 추가 원인 추적 필요 |
| 완료 | Health Center 테스트 보안 이벤트 | UI `테스트 보안 이벤트 발송` 성공 토스트와 SOAR 엔진 기록을 확인; Dashboard 새 행은 `WIZARD_TEST_POLICY`로 추가됨 | 내부 엔진 기록 성공으로만 판정하고 Teams 전달 성공으로 확대하지 않음 |
| 확인 필요 | Health Center 이벤트와 Teams Delivery 경로 차이 | 이벤트 실행 후 Dashboard 로그는 13건이 되었지만 `NOTIFY_TEAMS`·새 Delivery Ledger 행은 생성되지 않음 | `테스트 보안 이벤트 발송`은 엔진 기록용 경로로 분리 기록; Teams 실제 POST 또는 공식 정책 이벤트 경로를 별도 검증 |
| 완료 | Teams 실제 POST 재검증 | 권한·namespace 정합화 후 Health Center `Teams 실제 POST 점검`이 `READY`와 외부 채널 테스트 POST 성공(약 636ms)을 반환 | External Credential callout 자체는 통과; 실제 정책 이벤트 Ledger와는 별도 증적으로 기록 |
| 확인 필요 | 패키지 Flow 실행 경로 | Salesforce Flow 목록에서 `[SOAR 템플릿] VIP 데이터 접근 이상 알림 플로우`, `[SOAR 템플릿] 보안 조치 승인 에스컬레이션 플로우`를 확인했으나 모두 플랫폼 이벤트 트리거·`비활성`·`초안` 상태 | 템플릿을 임의 활성화하지 않고, 공식 Platform Event subscriber/Flow harness 없이는 정책 이벤트 재현 경로로 사용하지 않음 |
| 완료 | LWC Event Bus 진입점 재검증 | UI에서 `LWC Event Bus 진입점 테스트`를 다시 실행해 `SUCCESS: LWC Platform Event Published` 확인; Dashboard 로그는 14건으로 증가 | 이벤트 발행 성공으로 판정하고 외부 Teams 전달 성공으로 확대하지 않음 |
| 확인 필요 | LWC Event Bus와 Delivery Ledger | 재실행 후 새 감사 행은 `MASS_DATA_EXPORT / DEMO_SIMULATOR`였고 Delivery Ledger에는 기존 `EXHAUSTED` 행만 유지 | UI 진입점 발행과 실제 정책 `NOTIFY_TEAMS` callout을 별도 경로로 분리 |
| 완료→정정 | 2차 PoC 판정 표현 정정(이전 상태) | 당시에는 Teams 연결·Principal Access·`soarpkg` namespace·직접 Teams POST가 통과했지만 정책 기반 E2E가 아직 미검증이었음 | 후속 Flow 실행 결과를 반영해 현재 문구를 `Teams 연결과 Principal Access는 통과했지만 정책 기반 Teams E2E가 전달 단계에서 실패했다`로 갱신 |
| 부분 실행 | 후속 검증 트랙 분리 | Outbound는 `Send Security Log`로 임계값 1·`NOTIFY_TEAMS` 고유 이벤트를 발행해 감사·Ledger까지 확인했으나 전달이 실패함 | Action 성공·`DELIVERED`·Teams 카드는 미확인; Inbound·Zero-Login fixture는 별도 트랙 유지 |
| 완료 | Subscriber Flow harness 구성 | Flow Builder에서 `POC2_Send_Security_Log_Harness` V1을 Screen Flow로 저장·활성화하고 Policy Code·Event Key·Severity·Event Message 입력을 `Send Security Log`에 매핑 | Event Key는 Details에 보존; 결과 Screen은 추가하지 않음 |
| 확인→진행 | 정책 기반 Outbound 안전 설정 | 별도 커스텀 정책 생성 UI가 확인되지 않아 `EXTERNAL_THREAT_CALLBACK`을 재사용; High 임계값을 1로 변경하고 High `KILL_SESSION`을 제거해 `NOTIFY_TEAMS`만 유지 | Sandbox PoC 임시 변경으로 기록; 원래 값 원복 여부 후속 결정 |
| 완료 | UI 정책 이벤트 실행 | 활성 Flow에서 `EXTERNAL_THREAT_CALLBACK`·`HIGH`·고유 marker 이벤트를 1회 제출; Dashboard에 `HIGH / EXTERNAL_THREAT_CALLBACK / APEX` 감사 행 생성 | 정책 평가·감사 로그 생성 통과 |
| 막힘 | Outbound Delivery Ledger | 새 `POC2_TEAMS` 행에 `ActionType=NOTIFY_TEAMS`, 시도 `3/3`, `Status=EXHAUSTED`, 오류 `DELIVERY_FAILED` 표시 | `DELIVERED` 미달; 비동기 전달 실패로 판정 |
| 확인 필요 | `SecurityNotifyTeamsAction` 성공 증적 | Flow 결과 Screen이 없어 성공 반환값·Action 실행 상세를 직접 확인하지 못함 | 성공으로 주장하지 않고 Action/비동기 로그 추적 필요 |
| 확인 필요 | Direct Route와 정책 Action 차이 | Route Health/직접 Teams POST는 `READY`·성공이지만 정책 Flow의 Ledger는 `EXHAUSTED` | 직접 연결 성공과 비동기 정책 전달을 별도 게이트로 유지 |
| 막힘 | Teams 카드 수신 | 정책 기반 실행에 대한 실제 Teams 수신 화면·사용자 스크린샷 미확인 | `DELIVERED` Ledger와 카드 수신을 함께 확보할 때까지 미통과 |
| 막힘 | 비동기 Debug Trace | 현재 사용자 `USER_DEBUG` Trace는 저장됐으나 로그 행이 없었고 후속 실행은 Salesforce 세션 만료로 로그인 화면 전환 | 새 로그인 세션에서 비동기 실행 주체까지 Trace 후 단 1회 재실행 |

## 기록 규칙

- 실패 원문은 토큰·개인 식별자·Org ID를 제거한 형태로 기록
- 각 막힘마다 `발생 단계 → 증상 → 원인 추정 → 조치 → 결과`를 남김
- 설치·권한·화면·기능 검증을 각각 독립적인 통과/보류로 판정
- Production 적용이나 외부 운영 채널 연결은 이 테스트 범위에 포함하지 않음
