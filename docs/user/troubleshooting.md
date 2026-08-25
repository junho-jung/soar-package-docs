# 문제 해결

## 설치 후 앱이나 권한 집합이 보이지 않음

1. 패키지 설치가 완료 상태인지 Setup에서 확인합니다.
2. 네임스페이스가 포함된 권한 집합 이름을 사용했는지 확인합니다.
3. 사용자에게 필요한 권한 집합을 직접 할당했는지 확인합니다.
4. App Launcher와 권한 집합 할당 화면을 새로고침합니다.

## Setup & Health Center에 경고가 남음

경고 카드의 연결 대상과 다음 조치를 확인합니다. 서명 키, 인바운드 기본 설정, Named Credential, Sites 게스트 권한, 정책 초기화, 스케줄은 패키지 설치자가 직접 활성화해야 하는 항목입니다.

Setup & Health Center는 `SOAR Dashboard → 추가 탭 → 시스템 진단 & 설정 (Setup & Health Center)`에서 엽니다. 추가 탭 메뉴가 보이지 않으면 패키지 앱을 새로고침하고 `soarpkg__SOAR_Dashboard` 앱 페이지에 다시 진입합니다.

Teams와 Slack의 상태가 준비되지 않으면 `IF_Teams_Base`와 `IF_Slack_Base`라는 Named Credential Developer Name이 정확한지 먼저 확인합니다.

## Teams 또는 Slack 알림이 도착하지 않음

- 채널의 Named Credential이 존재하고 활성화되어 있는지 확인합니다.
- 라우트가 올바른 채널과 우선순위를 가리키는지 확인합니다.
- 설정 화면의 연결 점검 결과를 확인합니다.
- 동일 이벤트가 중복 억제 창 안에서 제외된 것은 아닌지 확인합니다.
- 전달 원장에서 실패와 재시도 횟수를 확인합니다.

## Health는 성공하지만 Delivery Ledger가 실패함

Health/Webhook/Ping의 HTTP 202는 연결성 점검입니다. 정책 기반 비동기 전달이 `DELIVERY_FAILED` 또는 `EXHAUSTED`로 끝나면 다음을 확인합니다.

1. Named Credential Developer Name이 `IF_Teams_Base`와 일치하는지 확인합니다.
2. External Credential Principal Access가 실제 실행 사용자·Flow 컨텍스트에 부여됐는지 확인합니다.
3. Named Credential의 관리 패키지 허용 namespace에 `soarpkg`가 등록됐는지 확인합니다.
4. 정책 Route가 `TEAMS`와 `NOTIFY_TEAMS`를 가리키고, fallback Route가 필요하면 별도로 활성화됐는지 확인합니다.
5. Delivery Ledger의 Attempt, 다음 재시도 시각, 오류 코드를 확인합니다.

시뮬레이터의 성공 토스트나 Health 202만으로 정책 기반 Teams E2E 성공을 판정하지 않습니다.

## 시뮬레이터는 성공했지만 Delivery Ledger가 없음

시뮬레이터와 일부 Entry point 버튼은 모의 이벤트·감사 기록 확인용입니다. 실제 외부 전달 검증은 Subscriber Flow의 `Send Security Log`, 실제 정책 이벤트, 또는 승인 후 수동 액션처럼 정책 평가를 거치는 경로로 수행합니다.

## 패키지 Flow가 설치됐지만 실행되지 않음

설치 후 `[SOAR 템플릿]` Flow가 `템플릿=true`, `활성=false`, `관리됨-설치됨`으로 표시될 수 있습니다. 이는 패키지에서 빠진 상태가 아니라 조직별 자동화 충돌과 업무 정책을 검토한 뒤 관리자가 선택적으로 활성화하도록 배포된 상태입니다.

1. Setup & Health Center와 Setup의 Flow 목록에서 이름·트리거·활성 버전을 확인합니다.
2. VIP 데이터 알림은 `SecurityAlert__e`의 `HIGH/CRITICAL` 이벤트에서 보조 Task를 만들고, 승인 에스컬레이션은 `SecurityActionRequest__e`의 파괴적 액션에서 보조 Task를 만듭니다.
3. 사용할 템플릿만 Flow Builder에서 활성화하고, 실행 주체의 권한과 중복 Task 여부를 먼저 확인합니다.
4. Flow 활성화 여부와 별개로 `DASHBOARD/TEAMS/BLOCK` 정책 결정, Action Ledger, Delivery Ledger는 패키지 Apex 경로에서 검증합니다.

Flow 템플릿은 결정자·Fallback·신원 검증·Teams 전달 여부를 판정하지 않습니다. 따라서 Flow가 비활성인 것은 핵심 정책·callback 경로의 장애를 의미하지 않으며, Flow를 활성화하면 정책 모드와 무관하게 보조 Task가 생성될 수 있습니다.

## 스케줄이 실행되지 않음

- 관리자 권한으로 스케줄 관리자에 접근했는지 확인합니다.
- 해당 작업이 Active인지 확인합니다.
- 같은 작업이 중복 등록되어 있지 않은지 확인합니다.
- 보존·정리 정책과 실행 주기가 조직 정책에 맞는지 확인합니다.

## 정책 Drift가 감지됨

정책 매퍼에서 변경된 항목과 누락된 항목을 확인합니다. 조직이 의도적으로 바꾼 값이면 문서화하고 유지하며, 표준값으로 돌아가야 하는 항목만 Reconcile합니다. 복구 후 감사 원장에 기록됐는지 확인합니다.

## Zero-Login 링크가 동작하지 않음

- Experience Cloud Site가 활성 상태인지 확인합니다.
- 게스트 사용자에게 인바운드 전용 권한 집합이 최소 범위로 할당되어 있는지 확인합니다.
- 인바운드 기본 URL과 서명 설정이 일치하는지 확인합니다.
- 링크가 이미 사용되었거나 만료된 경우 새 이벤트를 생성합니다.
- 파괴적 대응은 승인 대기 상태가 정상인지 확인합니다.

## Teams 버튼이 Site 홈이 아닌 Apex REST 주소로 열림

다음과 같은 형태로 열리는 것은 정상입니다.

```text
https://<site-domain>/<site-prefix>/services/apexrest/api/security/action?...단기 callback 값...
```

Teams 대응 버튼은 Site 홈 페이지를 표시하는 링크가 아니라 공개 callback endpoint를 호출합니다. `InboundBaseUrl__c`에 REST suffix를 직접 입력하지 말고 Site 기본 주소만 저장했는지 확인합니다. 그 다음을 점검합니다.

1. Site가 Active인지 확인합니다.
2. Site Guest User에 `SOAR_Inbound_Guest`가 할당되어 있는지 확인합니다.
3. Setup & Health Center에서 Callback 상태가 `READY`인지 확인합니다.
4. 파괴적 액션이면 요청자와 다른 관리자가 먼저 승인했는지 확인합니다.
5. 링크가 만료·재사용된 경우 기존 URL을 재시도하지 말고 새 카드를 발송합니다.

브라우저 callback 결과는 HTML이고, `404`·`403`·`ACTION_CODE_EXPIRED`·`ACTION_ALREADY_DONE`은 각각 경로/게스트 권한/유효시간/단회 사용 상태를 뜻합니다. callback URL의 code·token 원문은 지원 요청에 복사하지 않습니다.

## 로그가 쌓이지 않음

일반 사용자에게 운영 권한 집합이 없어도 백그라운드 이벤트 처리가 동작하는 구조인지 확인합니다. 그 다음 탐지 신호의 연결 상태, 정책 활성화 여부, 킬 스위치 상태, 감사 로그 보존 설정을 순서대로 확인합니다.
