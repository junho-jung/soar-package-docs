# 설치 후 전체 사용 런북

이 문서는 패키지 설치자가 Salesforce 조직에 SOAR를 설치한 뒤, 설정·정책·외부 채널·대화형 카드·감사 원장을 실제 사용 순서대로 확인하기 위한 UI 중심 런북입니다. 내부 Apex 구현을 재현하는 문서가 아니라, 패키지 사용자가 어떤 화면에서 무엇을 설정하고 어떤 상태를 성공으로 판정하는지 설명합니다.

## 0. 현재 문서 기준

| 항목 | 값 |
|---|---|
| Package2 | `SOAR_Operations_Core_Next 0.1.0.5` |
| 검증 기준 버전 | `0.1.0.5` |
| Subscriber Package Version ID | `04tdM000000c9OrQAI` |
| Namespace | `soarpkg` |
| 검증 대상 | Sandbox/Developer/Trial 조직 |

배포자가 더 새로운 Subscriber Package Version ID를 전달하면 설치 링크와 명령의 ID를 새 값으로 바꿉니다. 이 저장소는 GitHub Release 파일을 제공하지 않습니다.

## 1. 먼저 역할을 나누기

설치 전에 다음 네 역할을 분리합니다.

| 역할 | 의미 | 권장 권한 |
|---|---|---|
| 설치 관리자 | 패키지 설치, Site, Named Credential, 권한, 스케줄 설정 | `soarpkg__SOAR_Admin` |
| 보안 운영자 | 대시보드, 감사 로그, 시뮬레이터, 정책 조회 | `soarpkg__SOAR_Operator` |
| 액션 대상 | 정책에 걸린 사용자 또는 리소스 | 운영 권한 집합을 직접 부여하지 않음 |
| 최종 결정자 | Teams 또는 Dashboard에서 승인·거절·실행을 결정하는 담당자 | 조직의 승인 책임자 또는 보안 운영 그룹 |

테스트할 때 관리자 계정을 액션 대상 사용자로 직접 지정하지 않습니다. 대상 사용자와 결정자를 분리해야 권한 경계와 실제 업무 흐름을 함께 검증할 수 있습니다.

## 2. 패키지 설치

### 2.1 브라우저 설치

1. Salesforce Sandbox 로그인 상태에서 [현재 검증 기준 패키지 설치 화면](https://test.salesforce.com/packaging/installPackage.apexp?p0=04tdM000000c9OrQAI)을 엽니다.
2. 패키지 이름이 `SOAR_Operations_Core_Next`이고 Namespace가 `soarpkg`인지 확인합니다.
3. 설치 보안 수준은 조직의 정책에 맞게 선택합니다. 현재 UAT는 `AdminsOnly`로 설치한 뒤 필요한 사용자에게 패키지 권한 집합을 별도 할당했습니다.
4. 설치 완료 후 Setup의 Installed Packages에서 패키지 버전과 Namespace를 확인합니다.

### 2.2 CLI 설치

```bash
sf package install --package 04tdM000000c9OrQAI --target-org <YOUR_ORG_ALIAS> --security-type AdminsOnly --wait 30 --no-prompt
```

설치 결과가 성공이어도 사용자에게 권한 집합이 자동으로 필요한 범위까지 할당됐다고 보지 않습니다. 다음 장의 역할별 할당을 완료해야 합니다.

Subscriber 조직에는 Provider 소스를 직접 배포하지 않습니다. 패키지 변경은 새 패키지 버전을 설치하는 방식으로 반영합니다.

## 3. 역할별 권한 집합 할당

Setup → Users → Permission Set Assignments에서 필요한 사용자에게만 할당합니다.

```bash
sf org assign permset --name soarpkg__SOAR_Admin --target-org <YOUR_ORG_ALIAS>
sf org assign permset --name soarpkg__SOAR_Operator --target-org <YOUR_ORG_ALIAS>
```

- `SOAR_Admin`: 초기 설정·정책·라우트·승인 대기열·복구가 필요한 관리자에게만 할당합니다.
- `SOAR_Operator`: 운영 조회와 시뮬레이션이 필요한 사용자에게 할당합니다.
- `SOAR_Inbound_Guest`: 일반 사용자에게 주는 권한이 아닙니다. Experience Cloud Site의 Guest User에 최소 범위로 할당합니다.
- 패키지 권한 집합을 주지 않은 일반 사용자는 탐지 대상이 될 수 있지만 운영 콘솔 사용자가 되지는 않습니다.

## 4. Setup & Health Center 초기 설정

패키지 앱에서 `SOAR Dashboard → 시스템 진단 & 설정 (Setup & Health Center)`로 이동합니다. 메뉴가 보이지 않으면 App Launcher에서 패키지 앱을 다시 열고 `soarpkg__SOAR_Dashboard` 탭으로 진입합니다.

### 4.1 서명 키와 인바운드 설정

인바운드 대응을 사용하지 않더라도 상태를 먼저 확인합니다.

1. 서명 키 생성 또는 회전 상태를 확인합니다.
2. `SecurityInboundConfig__mdt`의 `Default` 설정이 존재하는지 확인합니다.
3. `IsSystemEnabled__c`가 `false`이면 전체 SOAR 액션과 인바운드 처리가 멈추므로 운영 중 Kill Switch 상태를 확인합니다.
4. 서명 검증을 사용할 때 `Secret__c`는 조직 전용 값으로 관리하고 문서·스크린샷·일반 로그에 남기지 않습니다.

### 4.2 Site와 Guest 권한

Zero-Login callback을 사용할 때만 설정합니다.

1. Salesforce Setup의 Quick Find에서 `사이트`를 검색합니다.
2. `사이트 및 도메인`에서 조직 Site 도메인을 등록합니다.
3. 인바운드 전용 Site를 생성하고 Active 상태로 만듭니다.
4. Setup & Health Center의 Site/Guest 설정에서 Active Site를 선택합니다.
5. 패키지가 제공하는 Guest 권한 부여 기능 또는 Setup의 Permission Set Assignment를 사용해 해당 Site Guest에 `soarpkg__SOAR_Inbound_Guest`를 할당합니다.
6. Callback 진단 상태가 `READY`인지 확인합니다.

`InboundBaseUrl__c`에는 다음처럼 Site의 기본 주소까지만 저장합니다.

```text
https://<site-domain>/<site-prefix>
```

`/services/apexrest/api/security/action` suffix는 직접 입력하지 않습니다. 패키지가 카드 callback URL을 만들 때 자동으로 붙입니다.

### 4.3 정책과 액션 초기화

Setup & Health Center 또는 Policy Builder에서 `1-Click 씨딩/초기화`를 실행합니다.

- 현재 검증 기준 화면에는 정책 12개와 액션 9개가 표시되었습니다.
- 조직에서 이미 운영 정책을 관리하고 있다면 초기화 전에 기존 값을 백업·검토합니다.
- 초기화 후 Policy Builder에서 정책 활성 상태, 심각도 임계치, High/Critical 액션을 확인합니다.
- 초기화 버튼의 성공 토스트는 데이터 초기화 성공을 뜻하며 Teams 외부 전달 성공을 뜻하지 않습니다.

### 4.4 Teams Named Credential

Teams를 사용할 때 Salesforce Setup에서 다음 순서로 구성합니다.

1. External Credential과 사용할 Principal을 준비합니다.
2. Named Credential Developer Name을 `IF_Teams_Base`로 맞춥니다.
3. Named Credential이 올바른 External Credential Principal을 참조하는지 확인합니다.
4. 실제 정책 이벤트를 발행하는 사용자 또는 실행 컨텍스트에 External Credential Principal Access를 부여합니다.
5. 관리 패키지가 Named Credential을 사용할 수 있도록 허용 namespace에 `soarpkg`를 등록합니다.
6. Setup & Health Center에서 Teams Health Check를 실행합니다.

Health의 HTTP 202는 외부 endpoint가 요청을 수락했다는 연결성 증적입니다. 정책 평가, Delivery Ledger 생성, 외부 카드 수신까지 확인해야 실제 전달 성공입니다.

원시 webhook URL·서명값·토큰은 Route 레코드, 문서, 스크린샷, 로그에 저장하지 않습니다.

### 4.5 Slack과 기타 채널

- Slack은 Teams와 동일한 typed notification route와 Block Kit callback 계약을 제공하는 선택 채널입니다. `IF_Slack_Base` Named Credential과 Slack 서명 검증용 조직 설정을 구성하면 정책 알림, 승인·거절·실행 callback, Delivery Ledger 추적을 사용할 수 있습니다.
- Slack은 Teams와 별도의 채널·원장 증적으로 판정합니다. `ACCEPTED` 또는 연결성 Ping만으로 완료 처리하지 말고 Slack 카드 수신, callback 결과, 최종 `DELIVERED`를 확인합니다.
- Generic Webhook·SIEM·Ticketing·Custom은 현재 Ping/연결성 진단과 Subscriber 확장 대상으로 이해합니다.

### 4.6 Flow 템플릿

Flow 목록에서 `[SOAR 템플릿]`으로 검색합니다.

| Flow | 트리거 | 설치 직후 상태 | 동작 |
|---|---|---|---|
| 보안 조치 승인 에스컬레이션 | `soarpkg__SecurityActionRequest__e` | Draft/비활성일 수 있음 | 파괴적 액션 요청에 보조 Task 생성 |
| VIP 데이터 접근 이상 알림 | `soarpkg__SecurityAlert__e` | Draft/비활성일 수 있음 | HIGH/CRITICAL 이벤트에 보조 Task 생성 |

두 Flow는 선택형 Subscriber 자동화입니다. 실제 `DASHBOARD/TEAMS/BLOCK` 정책 결정, 결정자, Fallback, Action Ledger, Delivery Ledger는 패키지 Apex 경로가 담당합니다. Flow를 활성화하면 정책 모드와 별개로 Task가 생성될 수 있으므로, 정책 결과에 종속된 Task가 필요하면 Subscriber Flow에 별도 조건과 원장 조회를 추가합니다.

### 4.7 스케줄

로그 보존, 정리, 감사 폴링, 실패 재전송이 조직에서 필요한 경우에만 Scheduler Manager에서 활성화합니다. 중복 등록 여부와 실행 주기를 확인하고, 테스트 조직에서는 불필요한 스케줄을 끕니다.

## 5. 첫 사용 검증 순서

다음 순서를 지키면 연결성 점검과 실제 정책 전달을 혼동하지 않습니다.

| 순서 | 화면/기능 | 성공 판정 |
|---:|---|---|
| 1 | Setup & Health Center | 필수 경고 원인과 다음 조치가 이해됨 |
| 2 | Teams Health/Webhook/Ping | Named Credential 연결·외부 수락 확인 |
| 3 | 테스트 보안 이벤트 발송 | Dashboard/Audit Log 진입 확인 |
| 4 | Threat Simulator | 시나리오 실행과 감사 로그 확인. 외부 전달 증적 아님 |
| 5 | `Send Security Log` 또는 실제 정책 이벤트 | 정책이 활성이고 Action Ledger 평가가 시작됨 |
| 6 | Action Ledger | `PENDING` 또는 정책 결정 상태 생성 |
| 7 | 승인 대기열/결정 화면 | 승인·거절·거절 사유가 원장에 남음 |
| 8 | Delivery Ledger | Route, Attempt, 상태가 생성됨 |
| 9 | Teams 카드 | 외부 카드 수신과 최종 `DELIVERED` 확인 |
| 10 | callback | 승인·실행·결과 HTML과 감사 상태 확인 |

Threat Simulator의 성공 토스트나 감사 로그 1건만으로 5~10단계를 통과했다고 기록하지 않습니다.

## 6. 화면별 사용법

### 6.1 Dashboard

1. 최근 보안 이벤트와 심각도 필터를 확인합니다.
2. 이벤트 상세에서 정책 코드, 대상 사용자/리소스, 유입 채널, 상관관계 정보를 확인합니다.
3. Action Ledger와 Delivery Ledger를 분리해 확인합니다.
4. `PENDING` 파괴적 요청은 승인 전 실행하지 않습니다.
5. 대상 사용자는 액션의 영향 대상이고, 결정자는 승인·실행 담당자입니다.

### 6.2 Threat Simulator

1. Platform, Data, Logic, Identity, External Signal 중 시나리오 영역을 선택합니다.
2. 대상 사용자는 관리자 계정이 아닌 별도 테스트 사용자로 선택합니다.
3. 정책 코드와 심각도, 대상 정보를 확인하고 실행합니다.
4. 성공 토스트와 Audit Log를 확인합니다.
5. Action/Delivery Ledger가 없으면 실제 정책 E2E가 아니라 모의 이벤트 경로로 기록합니다.

### 6.3 Policy Builder

1. 정책의 활성 상태를 확인합니다.
2. Threshold와 Severity별 High/Critical 액션을 확인합니다.
3. 파괴적 액션에는 승인과 대상 재검증이 필요하다는 점을 전제로 변경합니다.
4. 정책을 저장한 뒤 Drift 상태와 감사 기록을 확인합니다.
5. 변경 전·후 의미, 승인자, 원복 기준을 기록합니다.

### 6.4 Notification Route Manager

1. 정책 코드·심각도·우선순위에 연결할 Route를 선택합니다.
2. Teams Route는 `NOTIFY_TEAMS`와 `IF_Teams_Base` 연결을 확인합니다.
3. Fallback이 필요하면 조직에서 허용한 보조 Route를 별도로 활성화합니다.
4. Decision Mode는 `DASHBOARD`, `TEAMS`, `BLOCK` 중 업무 승인 흐름에 맞춰 선택합니다.
5. 원시 webhook URL은 입력하지 않습니다.

라우트가 `DASHBOARD`이면 Dashboard 승인/결정 경로를 기준으로 확인하고, `TEAMS`이면 Teams 수신자에게 카드가 도착하는지 확인합니다. `BLOCK`이면 액션 실행·외부 전달이 차단되는 것이 정상입니다.

### 6.5 연결성·대화형 카드 테스트

1. `채널 Webhook 테스트`에서 Teams 채널이 등록되어 있는지 확인합니다.
2. `대화형 채널 액션 카드 테스트`에서 대상 사용자를 선택합니다. 관리자 계정을 대상에 지정하지 않습니다.
3. 활성 Manifest에서 테스트할 액션을 선택합니다.
4. 테스트 카드 발송 결과의 Route, Delivery 상태, Inbound 설정을 기록합니다.
5. 비파괴적 액션은 알림 전달을, 파괴적 액션은 승인 전 버튼 숨김을 확인합니다.
6. 승인된 파괴적 액션만 callback 버튼이 실행 단계로 열리는지 확인합니다.

`ACCEPTED`는 접수 결과이고 외부 Teams 수신 성공과 동일하지 않습니다. 카드가 실제로 도착했는지 Teams에서 확인하고, 이후 Delivery Ledger와 callback 결과를 대조합니다.

## 7. 액션 선택 기준

현재 활성 Manifest에는 다음 9개 액션이 포함됩니다.

| 액션 | 성격 | 사용자 관점 |
|---|---|---|
| `FREEZE_USER` | 파괴적 | 사용자 계정 동결. 독립 승인 후 실행 |
| `KILL_SESSION` | 파괴적 | 활성 세션 종료. 독립 승인 후 실행 |
| `FORCE_MFA_RESET` | 파괴적 | MFA 재설정 강제. 독립 승인 후 실행 |
| `REVOKE_TOKEN` | 파괴적 | OAuth 토큰 회수. 조직 승인 정책 적용 |
| `QUARANTINE_LOGIN_IP` | 파괴적 | 로그인 IP 격리. 조직 승인 정책 적용 |
| `LOG_ONLY` | 감사 | 기록만 남기고 파괴적 대응을 하지 않음 |
| `NOTIFY_MANAGER` | 통지 | 내부 결정자/관리자에게 알림 |
| `NOTIFY_TEAMS` | 통지 | Teams Route로 보안 카드 전달 |
| `NOTIFY_SLACK` | 통지 | Slack Block Kit 카드와 서명된 단회 callback 전달. `IF_Slack_Base`와 외부 수신 E2E는 조직별 선택 구성 |

파괴적 액션은 목록에 표시되더라도 승인·신원·대상 재검증을 통과하기 전에는 실행 버튼이 열리지 않습니다. 테스트에서는 Dry Run 또는 카드 접수·버튼 숨김·원장 상태까지만 검증하고 실제 대상 계정에 파괴적 조치를 실행하지 않습니다.

## 8. Action Ledger와 Delivery Ledger

두 원장은 서로 다른 업무 상태를 기록합니다.

### Action Ledger

정책이 어떤 액션을 선택했고 누가 결정할지 기록합니다.

```text
PENDING → APPROVED → EXECUTED
       └→ REJECTED
```

- `PENDING`: 승인 또는 결정 대기
- `APPROVED`: 독립 승인 통과
- `REJECTED`: 거절 및 사유 기록
- `EXECUTED`: callback 또는 Dashboard에서 실행 완료

### Delivery Ledger

외부 채널 전송 상태를 기록합니다.

```text
ACCEPTED → DELIVERED
        └→ FAILED → 재시도 → EXHAUSTED
```

- `ACCEPTED`: 패키지가 외부 전송을 접수
- `DELIVERED`: 외부 채널 전달 성공
- `FAILED`: 현재 시도 실패
- `EXHAUSTED`: 제한된 재시도 후 최종 실패

Action Ledger가 `APPROVED`라고 해서 외부 Teams 전달이 성공한 것은 아니며, Delivery Ledger가 `DELIVERED`라고 해서 파괴적 액션이 실행됐다는 뜻도 아닙니다.

## 9. Teams callback과 비로그인 실행

Teams의 실행 버튼은 Site 홈으로 이동하는 링크가 아닙니다. 다음 REST callback 형식으로 보이는 것이 정상입니다.

```text
https://<site-domain>/<site-prefix>/services/apexrest/api/security/action?...단기 callback 값...
```

1. Site가 Active인지 확인합니다.
2. Site Guest에 `soarpkg__SOAR_Inbound_Guest`가 있는지 확인합니다.
3. Inbound Base URL에 REST suffix를 직접 붙이지 않았는지 확인합니다.
4. Callback 상태가 `READY`인지 확인합니다.
5. 파괴적 액션이면 별도 승인 후 새 카드를 발송합니다.
6. 기존 callback URL은 재사용하지 않습니다.

대표 오류는 다음처럼 해석합니다.

| 결과 | 의미 |
|---|---|
| `404` | Site/경로가 존재하지 않거나 주소가 잘못됨 |
| `403` | Guest 권한·서명·승인 경계 확인 필요 |
| `ACTION_CODE_EXPIRED` | 단기 callback 값 만료 |
| `ACTION_CODE_REPLAY` 또는 `ACTION_ALREADY_DONE` | 이미 사용된 callback 재사용 |
| `MISSING_ACTION_CODE` | 공식 카드/Manifest envelope 없이 공개 경로에 접근 |

callback code, token, correlation, idempotency, approval 식별자는 일반 로그·문서·스크린샷에 복사하지 않습니다.

## 10. 운영 종료 전 체크리스트

- [ ] 설치된 패키지 버전과 Namespace 확인
- [ ] Admin/Operator/Guest 권한이 역할별 최소 범위로 할당됨
- [ ] Active Site와 Guest 권한 확인
- [ ] Inbound Base URL과 Callback `READY` 확인
- [ ] Teams Named Credential·Principal Access·`soarpkg` namespace 허용 확인
- [ ] Slack을 사용할 경우 `IF_Slack_Base`, 서명 설정, Route source, callback 및 `DELIVERED` 증적 확인 (미사용 시 optional 상태로 유지)
- [ ] 정책·액션·Route·Decision Mode 검토
- [ ] 필요한 Flow 템플릿만 활성화
- [ ] Health/Webhook/Ping와 실제 정책 전달을 별도로 기록
- [ ] Action Ledger와 Delivery Ledger 상태를 모두 확인
- [ ] 대상 사용자와 최종 결정자를 분리
- [ ] 실제 파괴적 액션은 실행하지 않고 승인·callback 계약만 검증
- [ ] 문서·로그·스크린샷에 URL 서명, Secret, token 원문이 없음

## 11. 지원 요청에 포함할 정보

지원 요청에는 다음의 비민감 정보만 포함합니다.

- 패키지 버전과 Subscriber Package Version ID
- 실행한 화면과 단계
- 정책 코드·액션 이름·심각도
- Action/Delivery Ledger의 상태와 Attempt 수
- 오류 코드와 발생 시각
- Named Credential Developer Name
- Site/Guest/Callback 상태

원시 webhook URL, callback URL 전체, query code, token, Secret, 사용자 이메일과 원본 payload는 포함하지 않습니다.
