# 설치 및 초기 활성화

## 0. 베타 패키지 직접 설치

현재 문서 기준 패키지는 정식 Release 전 베타 버전 `SOAR_Operations_Core 1.11.0.1`입니다. 아래 링크는 Salesforce 로그인 후 패키지 설치 화면으로 이동합니다.

| 환경 | 설치 링크 |
|---|---|
| Production | [Production 설치 화면](https://login.salesforce.com/packaging/installPackage.apexp?p0=04tdM000000bx5xQAA) |
| Sandbox | [Sandbox 설치 화면](https://test.salesforce.com/packaging/installPackage.apexp?p0=04tdM000000bx5xQAA) |

- **Subscriber Package Version ID**: `04tdM000000bx5xQAA`
- **CLI 설치**:

```bash
sf package install --package 04tdM000000bx5xQAA --target-org <YOUR_ORG_ALIAS> --wait 30 --no-prompt
```

설치에는 Salesforce 로그인과 패키지 설치 권한이 필요합니다. 베타 버전은 먼저 Sandbox에서 설치·권한·기능을 확인하고, 운영 조직 적용은 별도 승인 후 진행하세요. 설치 링크가 더 이상 유효하지 않으면 과거 ID를 재사용하지 말고 패키지 제공자에게 최신 배포 정보를 확인합니다.

## 1. 설치 전 확인

| 항목 | 기본 기능 | 해당 기능을 사용할 때 |
|---|---|---|
| Salesforce 관리 권한 | 필수 | 패키지 설치와 권한 집합 할당 |
| SOAR 패키지 설치 링크 또는 버전 ID | 필수 | 제공자가 전달한 배포 채널의 값 사용 |
| Platform Event·Flow 사용 환경 | 권장 | Subscriber 자동화와 커스텀 신호 연결 |
| Named Credential | 선택 | Teams·Slack·외부 HTTP 채널 |
| Experience Cloud Sites | 선택 | Zero-Login 인바운드 대응 |
| Shield/Event Monitoring | 선택 | 플랫폼 수준의 고급 이벤트 신호 |

조직별 도메인, 설치 링크, 패키지 버전 ID, 외부 채널 주소는 공개 문서에 고정하지 않습니다. 설치 시점에 패키지 제공자가 지정한 값을 사용하세요.

## 2. 패키지 설치

Salesforce AppExchange 또는 제공자가 전달한 설치 링크에서 패키지를 설치합니다. CLI를 사용할 경우 아래처럼 조직 별칭과 배포 채널에서 받은 패키지 버전 ID를 사용합니다.

```bash
sf package install --package 04tdM000000bx5xQAA --target-org <YOUR_ORG_ALIAS> --wait 30 --no-prompt
```

설치 완료 후 패키지 네임스페이스가 적용된 앱과 권한 집합이 보이는지 확인합니다. Subscriber 조직에 패키지 소스를 직접 배포하는 방식은 사용하지 않습니다.

## 3. 역할별 권한 집합 할당

| 역할 | 권한 집합 | 용도 |
|---|---|---|
| 관리자 | `soarpkg__SOAR_Admin` | 초기 설정, 정책 복구, 스케줄, 서명 키, 운영 제어 |
| 운영자 | `soarpkg__SOAR_Operator` | 대시보드, 감사 로그, 시뮬레이터, 운영 조회 |
| 인바운드 게스트 | `soarpkg__SOAR_Inbound_Guest` | Sites 기반 Zero-Login 수신 지점 |
| 일반 사용자 | 별도 할당 없음 | 보안 탐지 대상이며 운영 콘솔 권한은 부여하지 않음 |

CLI로 할당하는 경우 네임스페이스를 포함합니다.

```bash
sf org assign permset --name soarpkg__SOAR_Admin --target-org <YOUR_ORG_ALIAS>
sf org assign permset --name soarpkg__SOAR_Operator --target-org <YOUR_ORG_ALIAS>
```

인바운드 게스트 권한은 Zero-Login을 사용할 때만 Experience Cloud Site의 게스트 사용자에게 최소 범위로 할당합니다.

## 4. Setup & Health Center에서 설치 후 활성화

패키지 설치 후 관리자가 다음 순서로 기능을 켭니다.

| 순서 | 설정 | 활성화 기준 |
|---:|---|---|
| 1 | 서명 키와 인바운드 기본 설정 | 키 상태와 인바운드 URL 상태가 정상 |
| 2 | Teams/Slack Named Credential | 사용할 채널의 연결 점검이 성공 |
| 3 | Experience Cloud Site와 게스트 권한 | Zero-Login을 사용할 때만 완료 |
| 4 | 표준 스케줄 | 필요한 보관·정리·폴러 작업이 Active |
| 5 | 기본 정책과 대응 액션 초기화 | 운영 정책이 생성되고 상태가 정상 |
| 6 | 테스트 보안 이벤트 | 대시보드·감사 로그·알림 흐름 확인 |

설정 화면의 1-Click 버튼은 패키지 설치자가 직접 실행하는 운영 설정입니다. 이 단계가 완료되면 해당 기능의 설치 후 활성화 이슈는 완료된 것으로 판단할 수 있지만, 외부 채널과 Sites는 고객 조직의 보안 정책과 승인 절차에 따라 별도 검토가 필요합니다.

## 5. 외부 채널 연결

외부 webhook URL을 애플리케이션 코드나 일반 설정 필드에 직접 저장하지 않습니다. Salesforce Setup에서 Named Credential을 만들고, SOAR 라우트가 해당 Named Credential을 참조하도록 설정합니다.

- 채널별 인증과 endpoint는 고객 조직이 관리합니다.
- 설정 화면에서 실제 POST 연결 점검을 수행합니다.
- 연결 실패 시 전달 원장과 재시도 상태를 확인합니다.
- 사용하지 않는 채널은 비활성화합니다.

## 6. 설치 완료 체크리스트

- [ ] 패키지 앱이 App Launcher에 표시됨
- [ ] 관리자 권한 집합이 할당됨
- [ ] 운영자 권한 집합이 필요한 사용자에게만 할당됨
- [ ] Setup & Health Center의 필수 상태가 정상
- [ ] 사용할 외부 채널의 Named Credential 연결 점검 완료
- [ ] 필요한 표준 스케줄만 활성화
- [ ] 기본 정책과 대응 액션이 조직 정책에 맞게 검토됨
- [ ] 테스트 이벤트가 감사 로그와 대시보드에 반영됨
- [ ] Zero-Login 사용 시 Sites와 게스트 권한 검토 완료
