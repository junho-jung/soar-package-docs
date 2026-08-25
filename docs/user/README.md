# 사용자 매뉴얼

이 매뉴얼은 Salesforce 관리자가 SOAR 패키지를 설치하고 활성화한 뒤, 보안 운영자가 일상적인 탐지·대응·감사 업무를 수행하는 흐름을 설명합니다.

## 권장 온보딩 순서

1. [설치 후 전체 사용 런북](./end-to-end-runbook.md)
2. [패키지 흐름과 사용자 페르소나](./architecture-and-personas.md)
3. [설치 및 초기 활성화](./installation-and-setup.md)
4. [일상 운영과 권한 모델](./operations.md)
5. [Subscriber 확장 지점 확인](../extensions/README.md)
6. [문제 해결](./troubleshooting.md)

## 가장 빠른 시작

설치자는 먼저 [설치 후 전체 사용 런북](./end-to-end-runbook.md)의 1~4장을 순서대로 완료합니다. 특히 권한 집합만 할당하고 끝내지 말고, Setup & Health Center에서 Site·게스트 권한·Inbound Base URL·Named Credential·정책 초기화 상태를 확인해야 합니다.

첫 테스트는 다음 순서로 진행합니다.

1. Setup & Health Center의 연결 점검으로 Named Credential 연결을 확인합니다.
2. 테스트 보안 이벤트로 앱·감사 로그 진입을 확인합니다.
3. Threat Simulator는 모의 이벤트와 감사 로그 확인용으로 사용합니다.
4. 실제 외부 전달을 판정할 때는 `Send Security Log` 또는 실제 정책 이벤트를 사용하고, Action Ledger와 Delivery Ledger를 차례로 확인합니다.
5. Teams 카드의 최종 수신과 callback 결과까지 확인해야 정책 기반 외부 전달을 완료로 판정합니다.

Health/Webhook/Ping의 성공이나 시뮬레이터의 성공 토스트만으로 Teams 정책 전달이 완료됐다고 판단하지 않습니다.

## 대상별 시작점

| 대상 | 먼저 볼 내용 |
|---|---|
| 패키지 설치 관리자 | 설치 전제조건, 권한 집합, Setup & Health Center |
| 보안 관리자 | 정책, 라우트, 스케줄러, 감사 원장, 킬 스위치 |
| SOC 운영자 | 대시보드, 시뮬레이터, 필터, CSV 내보내기 |
| Salesforce 개발·자동화 담당자 | [확장 가이드](../extensions/README.md)의 공개 계약과 제한 |
| 외부 SIEM/SOAR 담당자 | 외부 신호·원격 대응 연동과 승인 경계 |

## 문서에서 사용하는 표현

- **패키지 관리자**: 설치 후 설정과 권한을 관리하는 사용자
- **운영자**: 대시보드와 감사 로그를 확인하고 시뮬레이션을 수행하는 사용자
- **Subscriber**: SOAR 패키지가 설치된 고객사 Salesforce 조직
- **설치 후 활성화**: 패키지를 설치한 뒤 고객 조직의 관리자 또는 보안 담당자가 직접 설정해야 하는 단계

패키지 설치만으로 외부 채널, Experience Cloud Sites, 스케줄, 운영 정책이 자동으로 활성화되는 것은 아닙니다. 해당 기능을 사용할 조직은 [설치 및 초기 활성화](./installation-and-setup.md)의 체크리스트를 완료해야 합니다.
