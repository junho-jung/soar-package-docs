# 플랫폼 이벤트·Flow·커스텀 액션

## 표준 이벤트 흐름

SOAR가 생성한 표준 보안 이벤트는 Subscriber 조직의 선언형 자동화와 프로코드 후속 처리에 연결할 수 있습니다.

`탐지 신호 → 정책 평가 → 표준 보안 이벤트 → Flow/후속 액션 → 감사 원장`

표준 이벤트의 구체적인 필드와 설치 버전별 계약은 패키지 문서와 Salesforce Describe 결과를 기준으로 확인합니다. 이 공개 문서는 내부 스키마를 복사해 제공하지 않습니다.

Subscriber 자동화에서 주로 사용하는 표준 이벤트 필드는 다음과 같습니다.

| 필드 | 용도 |
|---|---|
| `EventKey__c` | 이벤트와 재시도 상관관계 식별 |
| `PolicyCode__c` | 평가된 정책 코드 |
| `ActionName__c` | 연결된 대응 액션 |
| `Severity__c` | 위험도 |
| `UserId__c`, `UserName__c` | 대상 또는 요청 사용자 |
| `ResourceId__c`, `ResourceName__c`, `ResourceType__c` | 대상 업무 리소스 |
| `Payload__c` | 패키지가 정규화한 안전한 이벤트 문맥 |

이벤트 필드는 Subscriber 조직의 Flow, Platform Event Trigger, Queueable 후속 처리에서 읽기 전용 입력으로 취급합니다. 패키지 내부 감사 원장이나 관리 패키지 객체를 직접 수정하지 않습니다.

## Flow 기반 확장

Event-Triggered Flow 또는 패키지가 제공하는 Invocable Action을 사용해 다음과 같은 후속 업무를 연결할 수 있습니다.

- 보안팀 또는 직속 관리자 알림
- 내부 Incident·Jira·티켓 시스템 등록
- 승인 요청 생성
- 고객사 정책에 따른 사용자 접근 검토
- 추가 감사 태그와 업무 문맥 기록

Flow에서 파괴적 조치를 직접 연결할 때는 승인·중복·롤백 조건을 별도로 설계해야 합니다.

## 프로코드 커스텀 액션

Subscriber의 Apex Trigger, Queueable, Platform Event subscriber 등은 표준 보안 이벤트를 받아 고객 조직의 후속 업무를 수행할 수 있습니다.

권장 경계는 다음과 같습니다.

- 패키지는 탐지·정책·표준 대응·감사를 담당합니다.
- Subscriber는 고객 조직의 티켓, 승인, 내부 통지와 업무별 후속 처리를 담당합니다.
- Subscriber 코드는 패키지 내부 객체를 직접 변경하지 않고 공개 이벤트·계약을 사용합니다.
- 실패한 후속 업무는 패키지의 원래 탐지 결과와 분리해 재처리합니다.

## Invocable 자동화 계약

`soarpkg.SecurityInvocableLogger`의 `Send Security Log` 액션은 Screen Flow 또는 Record-Triggered Flow에서 정책 코드·심각도·업무 문맥을 SOAR 평가 흐름에 전달하는 용도입니다. 입력은 `policyCode`, `severity`, `message` 또는 `details`, `recordId`, `targetUserId`로 구성하고, 결과의 `isProcessed`와 `statusMessage`를 후속 분기 조건으로 사용합니다.

입력 문맥에는 최소한의 업무 정보만 넣고, 비밀번호·서명 키·액세스 토큰·민감한 원문을 포함하지 않습니다. Flow 실행이 반복될 수 있으므로 레코드 ID나 업무 요청 ID를 기준으로 중복 정책을 정합니다.

## 검증 체크리스트

- [ ] 이벤트 구독 또는 Flow가 올바른 패키지 이벤트를 대상으로 함
- [ ] 관리자와 운영자의 권한 범위를 넘어서는 조치를 자동 실행하지 않음
- [ ] 반복 이벤트가 중복 티켓·알림을 만들지 않음
- [ ] 비동기 실행 실패가 감사 원장에 남음
- [ ] 재시도 횟수와 비상 중지 방법이 문서화됨
