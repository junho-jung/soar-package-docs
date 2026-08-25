# SOAR PoC 피드백 및 원인 분류

> 기준: 2026-08-26 새 `soarInstallTest` Subscriber Scratch Org, `SOAR_Operations_Core_Next 0.1.0.5`.
> 이 문서는 현재 재실행의 최종 피드백이다. 과거 `0.1.0.1`~`0.1.0.4` 실행 결과는 현재 게이트의 증적으로 승계하지 않는다.

## 1. 결론

패키지는 Sandbox/Developer 조직에서 설치·권한·핵심 운영 화면·Teams 연결성 검증까지는 사용할 수 있다. 다만 현재 증적만으로는 정책 기반 Teams outbound를 운영 완료로 판단할 수 없다.

- Teams Health와 외부 채널 직접 POST: 통과
- 실제 정책 이벤트: Audit 기록과 `NOTIFY_TEAMS` Delivery Ledger 생성까지 확인
- 정책 전달 최종 상태: 최신 행이 `EXHAUSTED 3/3`·`DELIVERY_FAILED`, `DELIVERED` 미확인
- 대상 사용자 분리: 별도 Role/User와 권한 분리는 완료했지만, 실행 후 Audit 대상은 실행 주체로 표시됨
- Inbound·Zero-Login: Site·Guest·Inbound metadata와 Callback Health `READY`까지 통과했지만 공식 callback fixture·Teams 카드가 없어 양성/만료/replay 경로 미실행
- Subscriber Global: Describe와 확장 harness 미실행

따라서 현재 사용 권고는 **비운영 PoC·Sandbox 검증에는 조건부 사용 가능, Production 정책 Teams E2E에는 보류**다.

## 2. 현재 증적 요약

| 영역 | 증적 | 판정 |
|---|---|---|
| 설치 | 패키지 `0.1.0.5`, namespace `soarpkg`, `SOAR_Admin` 확인 | 통과 |
| 핵심 회귀 | 5개 Simulator·4개 Entry point·Dashboard·Report·Policy UI 실행 | 부분 통과 |
| Data 채널 | Simulator는 성공했지만 요약·필터 값이 저장 이벤트와 불일치 | 제품 이슈 |
| Teams 구성 | `IF_Teams_Base`, External Credential Principal Access, `soarpkg` 허용 namespace, `DEFAULT_TEAMS` Match=true | 통과 |
| Teams 직접 연결 | Health Check 및 외부 채널 테스트 POST 성공(약 634ms) | 연결성 통과 |
| 정책 E2E | `OFF_HOURS_DATA_MUTATION`·HIGH·APEX Audit 행 생성; `Event: Unknown` 표시 | Audit 부분 통과 |
| Delivery Ledger | 최신 임시 Route `POC2_TEAMS_ACTION_E2E`의 `ActionType=NOTIFY_TEAMS`, 최종 `EXHAUSTED/3/3`, 오류 `DELIVERY_FAILED` | 실패 |
| Teams 카드 | 정책 이벤트에 대한 수신 화면 미확인 | 미통과 |
| Inbound | Site·Guest·Inbound metadata 구성과 Callback Health `READY`는 확인; fixture·실제 callback은 미실행 | 구성 통과·시나리오 보류 |
| Global | Describe·Sensor·Adapter·Event·Renderer 미실행 | 미실행 |

## 3. 문제 원인 분류

| ID | 관찰된 문제 | 분류 | 확신도 | 판단 근거·필요 조치 |
|---|---|---|---|---|
| P-01 | Data CDC Simulator 성공 후 Data 요약·필터가 0건이고 Platform 수치도 전체 목록과 다름 | 패키지 기능 결함 가능성 | 높음 | 동일 UI에서 저장 이벤트와 집계·필터가 불일치했다. 채널 분류값, 집계 쿼리, 필터 조건을 Provider가 동일 fixture로 재현해야 한다. |
| P-02 | 직접 Teams POST는 성공했지만 정책 비동기 전달은 임시 Route에서 `EXHAUSTED/3/3/DELIVERY_FAILED`로 종료 | 패키지 런타임/실행 컨텍스트 또는 async callout 관측성 문제 가능성 | 높음 | Principal·namespace·Named Credential·Route `Match=true`를 맞춘 뒤에도 두 경로의 결과가 달랐다. `SecurityDeliveryLedgerService`·`SecurityDeliveryRetryJob`는 완료지만 Apex Jobs UI에 외부 예외가 없었다. `SecurityNotifyTeamsAction`의 실제 비동기 주체, Principal Access, payload/route 결정, retry 오류를 Provider가 재현해야 한다. |
| P-03 | Flow의 `Target_User_Id`를 별도 대상 사용자로 입력·매핑했지만 Audit 대상이 실행 주체로 표시 | 패키지 계약 또는 런타임 바인딩 불일치 | 높음 | 공개 확장 문서는 `targetUserId` 입력을 안내하지만 실행 결과가 대상 분리를 반영하지 않았다. 입력 의미가 “정책 대상”인지 “요청자”인지 명시하고, 실제 바인딩 회귀 테스트와 수정이 필요하다. |
| P-04 | 문서는 `isProcessed`·`statusMessage` 반환값을 안내하지만 Flow Builder Action 설정에 출력이 노출되지 않음 | 패키지 메타데이터/공개 계약과 문서의 불일치 가능성 | 중간 | Subscriber UI에서 반환 출력이 확인되지 않았다. Provider는 Invocable output 선언·Managed Package Flow 노출 여부를 확인하고, 결과 Screen 예제를 제공해야 한다. |
| P-05 | Audit와 Ledger의 상관관계가 `Event: Unknown`으로 보이고 Screen의 Event Key가 Ledger/UI에 연결되지 않음 | 관측성·계약 문서 보완 또는 패키지 기능 이슈 | 중간 | 현재 Action 입력에 직접 `eventKey`/`idempotencyKey`가 없고 marker를 Details/Message에 넣어도 화면 상관관계가 보이지 않았다. 공식 correlation 필드와 사용자 화면 표시 규칙이 필요하다. |
| P-06 | Teams route는 정상인데 Health 전체 화면은 Slack·Inbound 누락 때문에 경고/CRITICAL 상태 | 설정 UX·문서 표현 개선 | 높음 | Slack은 현재 검증본에서 완성된 전달 경로가 아니고 Inbound는 선택 기능인데, 전체 상태가 Teams 준비 상태와 섞여 보인다. 채널별 필수/선택/미지원 상태를 분리 표시해야 한다. |
| P-07 | 공식 Inbound action code/token/signature/expiry/replay fixture가 없음 | Provider 테스트 자산·문서 부족 | 높음 | 공개 문서는 fixture가 필요하다고만 하고 출처·버전·필드표를 제공하지 않는다. fixture manifest와 비파괴 정상 fixture를 제공하기 전에는 Package 결함으로 판정하지 않는다. |
| P-08 | CSV 버튼 실행 후 완료·다운로드 피드백이 화면에서 보이지 않음 | UI/검증성 이슈, 결함 미확정 | 낮음~중간 | 실제 파일 다운로드 여부를 브라우저 다운로드 폴더와 함께 검증하지 못했다. 다운로드 성공 토스트·파일명·실패 오류를 제공하면 판정 가능하다. |
| P-09 | 활성 Flow V1 편집에 새 버전이 필요함 | Salesforce 표준 동작 | 높음 | 활성 Flow는 직접 수정할 수 없어 V2 저장·활성화로 해결했다. 패키지 결함은 아니며, 사용자 런북에 새 버전 절차를 추가하면 된다. |
| P-10 | Permission Set 할당 화면 기본 목록 보기가 삭제되었거나 접근 불가 | 오그 Setup/UI 상태 | 높음 | `최근 조회 항목`으로 전환해 해결했다. 패키지 권한집합 문제로 보지 않는다. |
| P-11 | 초기 Chrome 세션이 로그인 화면에 머묾 | 브라우저·인증 환경 | 높음 | CLI alias 인증으로 Chrome 진입해 해결했다. 패키지 문제도 레포 설명 문제도 아니다. |
| P-12 | Site·Guest·Inbound metadata를 구성한 뒤 Callback Health는 `READY`였지만 실제 callback 화면은 확인하지 못함 | Provider fixture·Teams delivery 미완료 | 높음 | 사용자 조직 설정 자체는 문서 흐름으로 해결됐다. 공식 action code/token fixture와 실제 카드가 도착하기 전에는 Inbound 실행 결함으로 판정하지 않는다. |
| P-13 | `GUEST_USER_DATA_LEAK` 평가 MEDIUM과 처음 만든 임시 Route HIGH가 맞지 않아 원하는 Route가 선택되지 않음 | 사용자 Route 설정 정합성 문제 | 높음 | 정책 평가 Severity와 Route Profile의 Policy Code/Severity를 동일하게 맞추고 Route 목록을 새로고침해 `Match=true`를 확인해야 한다. 문서에 예시를 추가할 필요가 있다. |
| P-14 | Health Center 고정 테스트 이벤트는 성공 토스트와 Audit만 만들고 Delivery Ledger를 만들지 않음 | 검증 UX·문서 구분 부족 가능성 | 높음 | 해당 버튼을 실제 outbound 증적으로 사용하지 않고, 고유 정책 이벤트 → Audit → Action → Delivery → Teams 카드 순서를 명시해야 한다. |
| P-15 | Async Apex 작업은 완료 상태지만 callout 실패의 상세 예외가 Apex Jobs UI에 노출되지 않음 | 패키지 관측성 부족 가능성 | 중간~높음 | Delivery Ledger에 안전한 분류형 실패 원인·attempt별 외부 응답·실행 주체/Principal 진단을 추가하거나, PoC용 Trace Flag/로그 수집 절차를 제공해야 한다. |

## 4. 레포 설명이 부족한 부분

현재 레포의 핵심 경계 설명은 대체로 충분하다. 특히 `Health/Ping/202`와 정책 기반 `Delivery Ledger/Teams 카드`가 서로 다른 검증이라는 점은 사용자 매뉴얼·확장 가이드·문제 해결 문서에 반복해서 명시되어 있다. 다만 처음 읽는 사용자의 실행 실패를 줄이려면 다음을 보완해야 한다.

1. README 첫 시작 부분에 “Teams는 두 게이트” 요약을 추가한다.
   - Gate A: Named Credential·Principal Access·namespace·Health·직접 POST
   - Gate B: 실제 정책 이벤트·Audit·Action·Delivery Ledger·Teams 카드
2. Teams 설정 체크리스트에 External Credential → Named Principal → Permission Set Principal Access → `soarpkg` 허용 namespace 순서를 한 줄 표로 고정한다.
3. `Send Security Log` 문서에 Flow Builder 화면에서 반환값이 보이지 않을 수 있는지, 결과 Screen을 어떻게 구성할지 명시한다.
4. `targetUserId`가 요청자·정책 대상·결정자 중 무엇을 의미하는지 필드 정의를 분리한다.
5. Inbound fixture의 출처·버전·필드표를 제공하거나, 공개할 수 없다면 “Provider가 별도 전달해야 하는 필수 테스트 자산”이라고 전면에 표시한다.
6. Slack이 현재 검증본에서 완성된 outbound 경로가 아니라는 점과, Slack 미설정이 Teams 검증을 무효화하지 않는다는 점을 Health 화면과 README에서 같은 용어로 표시한다.
7. CSV 다운로드 성공 증적과 Flow 활성 버전 교체 절차를 사용자 런북에 추가한다.

반대로 다음은 레포 설명 부족으로 보지 않는다.

- 실제 webhook URL을 Route에 저장하지 않고 Named Credential에만 저장하는 원칙
- 파괴적 액션을 실행하지 않는 안전 경계
- Outbound Teams와 Inbound·Zero-Login fixture를 서로 대체하지 않는 원칙
- 관리자·운영자·액션 대상·결정자를 분리해야 한다는 역할 모델

## 5. Provider/패키지 개선 우선순위

### P0 — 운영 사용 전 해결

- `SecurityNotifyTeamsAction` 비동기 전달 실패를 동일 조건에서 재현하고, 실제 실행 주체·Principal Access·namespace·route/payload·retry job을 추적한다.
- `targetUserId`가 Audit·Action·Delivery 상관관계에 반영되는지 계약을 확정하고 회귀 테스트를 추가한다.
- `SecurityInvocableLogger` output이 Flow Builder에서 노출되는지 확인하고 문서와 패키지 메타데이터를 일치시킨다.

### P1 — 사용성·관측성 개선

- Data 채널 집계·필터 쿼리를 저장 이벤트와 일치시킨다.
- Event Key/correlation/idempotency를 Delivery Ledger와 Dashboard에서 검색 가능하게 한다.
- Health를 채널별 `READY / OPTIONAL / NOT_CONFIGURED / UNSUPPORTED`로 구분한다.
- CSV와 Flow 결과에 성공·실패·다운로드 피드백을 추가한다.

### P2 — 문서·테스트 자산

- Teams 두 게이트 Quick Start를 README에 추가한다.
- 공식 Inbound fixture manifest와 비파괴 정상 fixture를 제공한다.
- `Send Security Log` 결과 Screen, 대상 사용자 분리, retry/fallback 검증 harness를 템플릿으로 제공한다.

## 6. 사용 가능성 판단

| 사용 목적 | 판단 |
|---|---|
| Sandbox/Developer 설치·권한·Health·Simulator·Dashboard 평가 | 사용 가능 |
| Teams Named Credential 연결 및 직접 POST 확인 | 사용 가능 |
| Production 정책 기반 Teams 알림 운영 | 현재 보류 |
| Inbound·Zero-Login 운영 | Site·Guest·Callback Health는 구성 통과; 공식 fixture·실제 callback 검증 전 보류 |
| Subscriber Global 확장 개발 | Describe·계약 회귀 전 보류 |

최종적으로 이 패키지는 “설치와 연결성은 검증된 베타 PoC 대상”이지만, “정책 기반 외부 전달까지 운영 검증이 끝난 패키지”는 아니다.

## 7. 참고 문서

- [재실행 계획서](./poc-rerun-plan.md)
- [재실행 결과서](./poc-rerun-results.md)
- [재실행 실행 로그](../../poc-rerun-execution-log.md)
- [설치 및 초기 활성화](../user/installation-and-setup.md)
- [플랫폼 이벤트·Flow·커스텀 액션](../extensions/events-flows-and-actions.md)
- [Teams·Slack·Zero-Login](../extensions/notifications-and-zero-login.md)
- [문제 해결](../user/troubleshooting.md)
