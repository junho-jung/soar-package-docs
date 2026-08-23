# Teams·Slack 알림과 Zero-Login 대응

## 알림 라우팅

SOAR는 정책 코드, 심각도, 우선순위와 조직이 선택한 채널을 기준으로 알림 경로를 결정합니다.

| 기능 | 사용자 설정 |
|---|---|
| 채널 연결 | Salesforce Named Credential 등록 |
| 라우팅 | 정책·심각도·우선순위별 Route 설정 |
| 중복 억제 | 조직이 정한 dedupe window 설정 |
| 실패 추적 | 전달 원장과 제한된 지수 백오프 재시도 |
| 최종 실패 | Fallback 경로와 관리자 알림 검토 |

패키지의 표준 채널 점검은 다음 Named Credential Developer Name을 기준으로 합니다.

- Microsoft Teams: `IF_Teams_Base`
- Slack: `IF_Slack_Base`

Route에 원시 webhook URL을 저장하지 말고, 위 canonical Named Credential 또는 조직 보안 정책에 맞는 허용된 Named Credential을 사용합니다.

원시 webhook URL이나 인증값은 Route 레코드와 문서에 직접 저장하지 않습니다. Named Credential의 인증 방식과 권한은 고객 조직이 관리합니다.

## Teams·Slack 확장 범위

Subscriber는 표준 보안 경보의 표현과 전달 경로를 조직의 채널 정책에 맞게 조정할 수 있습니다. 채널별 메시지 변환은 외부 시스템의 요구사항에 맞추되, 원본 이벤트의 승인 상태와 감사 추적 ID를 누락하지 않습니다.

## Zero-Login 전제조건

Zero-Login은 Salesforce Experience Cloud Site와 인바운드 전용 게스트 권한을 고객 조직이 직접 활성화했을 때 사용할 수 있습니다.

1. 활성 Experience Cloud Site 준비
2. 인바운드 기본 설정과 서명 검증 준비
3. 게스트 사용자에게 최소 권한 집합 할당
4. 외부 채널 카드에서 테스트 이벤트 확인
5. 승인·실행·결과 화면·감사 기록 검증

사이트를 활성화하지 않은 조직에서는 표준 Salesforce 로그인 후 승인 화면으로 대응하는 흐름을 사용합니다.

## Teams callback은 Site 홈 주소와 다르다

Zero-Login Site는 공개 진입점의 호스트와 경로를 제공하지만, 카드의 실행 버튼은 Site 홈 페이지로 이동하지 않습니다. 패키지는 다음 형태의 REST callback endpoint를 자동으로 생성합니다.

```text
<InboundBaseUrl>/services/apexrest/api/security/action
```

여기서 `InboundBaseUrl`은 `https://<site-domain>/<site-prefix>`까지만 설정하고, REST suffix는 패키지가 붙입니다. 카드가 브라우저에서 이 경로로 열리면 `@HttpGet` callback이 요청을 검증하고, 성공·승인 필요·만료·재사용·오류 결과를 HTML로 보여줍니다. 이 구조 때문에 Site의 기본 페이지를 `InMaintenance`로 설정한 Inbound 전용 Site에서도 callback은 별도로 처리됩니다.

카드 버튼에 포함되는 opaque code, correlation, idempotency, approval binding 값은 단기·단회 보안 값입니다. 공개 이슈·문서·스크린샷에 원문을 남기지 않고, 만료·재사용 오류가 나면 새 정책 이벤트에서 새 카드를 발송합니다.

## 안전 제어

- 파괴적 액션은 요청자와 승인자를 분리하는 이중 승인 흐름을 적용합니다.
- 외부 링크의 토큰은 단일 사용과 만료 정책을 따릅니다.
- 링크 미리보기 봇이나 자동 크롤러가 조치를 실행하지 않도록 안전한 조회와 실행 단계를 분리합니다.
- 실행 결과는 성공·대기·거절·실패 상태로 감사 원장에 남깁니다.
- 사용하지 않는 Sites endpoint와 채널은 즉시 비활성화합니다.

## 확장 시 금지 사항

- 카드 payload에 비밀번호, 서명 키, OAuth 토큰을 넣지 않습니다.
- URL query string에 장기 인증 정보를 넣지 않습니다.
- 승인 없이 계정 동결·세션 종료·토큰 회수를 자동화하지 않습니다.
- 외부 채널의 재시도 폭주가 Salesforce governor limit을 소모하지 않도록 제한합니다.
