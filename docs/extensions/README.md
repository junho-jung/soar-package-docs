# Subscriber 확장 가이드

이 문서는 설치한 Salesforce 조직에서 SOAR를 기존 업무·보안 시스템과 연결할 때 사용할 수 있는 공개 확장 계약을 설명합니다. 구현 코드를 제공하는 문서가 아니라, 어떤 연결 지점이 지원되고 어떤 보안·운영 조건을 지켜야 하는지를 정리한 계약 문서입니다.

## 확장 영역 전체 목록

| 영역 | Subscriber가 할 수 있는 일 | 관련 문서 |
|---:|---|---|
| 1. 보안 센서 | 고객 객체와 업무 이벤트를 보안 신호로 연결 | [센서·비즈니스 로직](./sensors-and-business-logic.md) |
| 2. 비즈니스 로직 평가 | 기존 Apex 업무 흐름에서 정책 평가 지점 호출 | [센서·비즈니스 로직](./sensors-and-business-logic.md) |
| 3. 플랫폼 이벤트·Flow | 표준 보안 이벤트를 Flow와 자동화에 연결 | [이벤트·Flow·액션](./events-flows-and-actions.md) |
| 4. 커스텀 대응 액션 | 내부 승인·티켓·SMS·업무 조치를 후속 처리 | [이벤트·Flow·액션](./events-flows-and-actions.md) |
| 5. Invocable 자동화 | 선언형 Flow에서 보안 이벤트를 기록 | [이벤트·Flow·액션](./events-flows-and-actions.md) |
| 6. Teams·Slack | Named Credential 기반 알림과 라우팅 | [알림·Zero-Login](./notifications-and-zero-login.md) |
| 7. Zero-Login·Sites | 로그인 없이 제한된 인바운드 대응 수신 | [알림·Zero-Login](./notifications-and-zero-login.md) |
| 8. 외부 SIEM/SOAR | 외부 관제 시스템에서 신호·승인 흐름 연결 | [외부 연동·정책·브랜딩](./external-policy-and-branding.md) |
| 9. 정책·채널 | 정책 임계치, 액션, 라우트 운영값 조정 | [외부 연동·정책·브랜딩](./external-policy-and-branding.md) |
| 10. 브랜딩·결과 화면 | Zero-Login 결과 화면의 조직별 표현 변경 | [외부 연동·정책·브랜딩](./external-policy-and-branding.md) |

## 공개 계약 이름

설치 버전에서 Subscriber 확장 대상으로 제공되는 대표 계약은 다음과 같습니다.

| 계약 | 목적 | 처리 성격 |
|---|---|---|
| `soarpkg.ISecuritySensor` | 고객 객체 변경을 보안 신호로 변환 | 트랜잭션 내 신호 생성 |
| `soarpkg.SecuritySensorAdapter` | 센서와 패키지 이벤트 흐름 사이 | 후속 비동기 처리 가능 |
| `soarpkg.Sec` | 업무 로직에서 정책 평가 지점 제공 | 호출 결과는 패키지 정책에 따름 |
| `soarpkg__SecurityAlert__e` | 표준 보안 이벤트를 Flow·Trigger에 전달 | 비동기 이벤트 |
| `soarpkg.SecurityInvocableLogger` | 선언형 자동화에서 보안 이벤트 기록 | Flow 호출 |
| `soarpkg.ISecurityActionHtmlRenderer` | 결과 화면 표현을 조직 기준으로 확장 | 응답 렌더링 |

실제 접근 수준(`global`/`public`), 파라미터와 지원 버전은 설치한 패키지 버전의 공식 계약과 Salesforce Describe 결과를 기준으로 확인합니다. 이름만 보고 내부 클래스나 내부 API를 직접 호출하지 않습니다.

## 모든 확장에 공통으로 적용되는 조건

- **권한**: 확장 코드와 자동화는 필요한 객체·필드·이벤트 권한만 사용합니다.
- **실패 처리**: 외부 전송 실패와 업무 트랜잭션 실패를 구분하고, 재시도 가능한 경우에만 제한적으로 재시도합니다.
- **중복 방지**: 동일 신호나 요청 ID를 반복 처리하지 않도록 idempotency 기준을 둡니다.
- **감사**: 탐지, 정책 평가, 승인, 실행, 실패와 롤백 결과를 추적할 수 있어야 합니다.
- **승인**: 사용자 동결·세션 종료·토큰 회수처럼 영향이 큰 액션은 조직의 이중 승인 절차를 따릅니다.
- **보안 경계**: 서명 키, 인증 정보, 내부 endpoint와 원본 payload를 소스 코드·문서·로그에 남기지 않습니다.
- **호환성**: 네임스페이스, 패키지 버전, Salesforce 에디션과 선택 라이선스를 확인한 뒤 연결합니다.

## 구현 전 체크리스트

- [ ] 사용할 확장 계약이 현재 설치 버전에서 지원됨
- [ ] 필요한 권한 집합과 객체·필드 권한이 최소 범위로 준비됨
- [ ] 동기/비동기 처리와 governor limit 영향을 검토함
- [ ] 재시도·중복·롤백·감사 기준을 정함
- [ ] Sandbox에서 시뮬레이터와 실패 시나리오를 검증함
- [ ] 운영 전 승인자와 비상 중지 절차를 정함

