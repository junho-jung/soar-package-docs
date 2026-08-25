# SOAR Package Documentation

Salesforce SOAR 패키지의 사용자 매뉴얼, Subscriber 확장 가이드, 아키텍처와 검증 포트폴리오를 한 곳에서 제공합니다.

이 저장소는 패키지를 설치하고 운영하는 사용자가 필요한 내용을 빠르게 찾도록 구성한 공개 문서 저장소입니다. 패키지의 내부 구현 소스나 특정 조직의 운영 자료를 공개하는 저장소는 아닙니다.

> 문서 상태: 베타 검증 기준의 공개 문서입니다. GitHub Release 자산이나 패키지 설치 파일을 관리하지 않습니다.

## 현재 검증 기준 패키지 직접 설치

현재 공개 문서가 기준으로 삼는 설치본은 `SOAR_Operations_Core_Next 0.1.0.5`입니다. Salesforce 관리자 계정으로 Sandbox, Developer 또는 Trial 조직에 먼저 설치해 기능과 권한을 검토하세요. 이 저장소는 GitHub Release 자산을 관리하지 않으므로, 이후 배포에서는 제공자가 전달한 최신 Subscriber Package Version ID를 사용합니다.

| 환경 | 설치 화면 |
|---|---|
| Sandbox | [Salesforce Sandbox 설치](https://test.salesforce.com/packaging/installPackage.apexp?p0=04tdM000000c9OrQAI) |

- **Package2**: `SOAR_Operations_Core_Next`
- **Namespace**: `soarpkg`
- **Subscriber Package Version ID**: `04tdM000000c9OrQAI`
- **CLI**: `sf package install --package 04tdM000000c9OrQAI --target-org <YOUR_ORG_ALIAS> --security-type AdminsOnly --wait 30 --no-prompt`
- 현재 검증본은 운영 Production 조직에 적용하기 전에 Sandbox 또는 Developer 조직에서 검증합니다.
- 설치 후에는 [설치 및 초기 활성화](./docs/user/installation-and-setup.md)의 권한·Named Credential·Sites·스케줄 체크리스트를 완료합니다.
- 화면을 처음부터 끝까지 따라가려면 [설치 후 전체 사용 런북](./docs/user/end-to-end-runbook.md)을 사용합니다.

## 먼저 읽을 문서

- [사용자 매뉴얼](./docs/user/README.md): 설치, 초기 활성화, 권한, 일상 운영, 문제 해결
- [설치 후 전체 사용 런북](./docs/user/end-to-end-runbook.md): UI 기준 설치·설정·테스트·Teams callback·원장 판정
- [확장 가이드](./docs/extensions/README.md): Subscriber에서 사용할 수 있는 공개 확장 계약과 연결 지점
- [포트폴리오](./docs/portfolio/README.md): 문제 정의, 보안 아키텍처, 검증 결과와 설계·검증 자료

## 패키지의 역할

SOAR는 Salesforce 안에서 보안 신호를 수집하고, 정책을 평가하고, 승인 가능한 대응을 실행하며, 감사 원장을 남기는 운영 패키지입니다.

- 여러 보안 신호를 공통 이벤트 흐름으로 수집
- 정책·심각도·채널에 따른 대응 결정
- Teams·Slack·이메일·외부 관제 시스템으로 알림 전달
- 승인, 재시도, 중복 억제, 롤백과 감사 추적 지원
- 대시보드, 시뮬레이터, 정책 매퍼, 설정·헬스체크 화면 제공

## 공개 범위

공개 문서에는 사용자 관점의 설정 방법, 지원되는 확장 계약의 목적과 제약, 운영 절차, 보안 경계를 담습니다.

다음 내용은 공개하지 않습니다.

- 패키지 내부 Apex/LWC/Trigger 전체 소스와 테스트 코드
- 내부 메타데이터 원본, 실행 스크립트, 내부 API 구현
- 특정 Org ID·접속 주소·토큰·서명 키·원본 로그
- 그대로 복사해 내부 구현을 재현할 수 있는 상세 코드와 payload

## 관련 저장소의 역할

| 저장소 | 공개 범위 | 역할 |
|---|---|---|
| `soar-package-docs` | Public | 사용자 매뉴얼과 포트폴리오 |
| `SOAR-subscriber` | Private | 베타테스터용 Subscriber 통합·검증 |
| `SOARPKG-SFDX` | Private | 패키지 제공자 소스와 개발 문서 |

## 문서 갱신 원칙

기능이 추가되거나 패키지 계약이 변경되면 사용자 매뉴얼과 확장 가이드를 먼저 갱신합니다. 문서에는 설치 링크나 조직별 값 대신 배포 채널에서 제공되는 값을 사용하도록 안내하며, 실제 검증 결과는 포트폴리오의 검증 스냅샷으로 구분합니다.
