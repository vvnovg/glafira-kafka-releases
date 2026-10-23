# 시작하기

Glafira for Apache Kafka는 Kafka 클러스터용 데스크톱 클라이언트입니다. 토픽과 컨슈머 그룹을 조회하고, 메시지를 읽고 보내고, 오프셋을 재설정할 수 있습니다. 이 장에서는 설치, 첫 실행, 연습용 로컬 Kafka, 표시 언어를 설명합니다.

<a id="install-macos"></a>

## macOS에 설치

Glafira는 macOS 11 이상에서 실행됩니다. 릴리스 페이지에서 설치 프로그램을 다운로드하세요: https://github.com/vvnovg/glafira-kafka-releases/releases

Mac에 맞는 파일을 선택하세요. Apple 실리콘(M1 이상)은 `aarch64`, Intel은 `x64`입니다.

1. `.dmg` 파일을 엽니다.
2. Glafira for Apache Kafka를 '응용 프로그램' 폴더로 드래그합니다.
3. '응용 프로그램' 또는 Launchpad에서 앱을 실행합니다.

`.pkg` 설치 프로그램을 사용할 수도 있습니다. 이 경우 앱은 모든 사용자를 위해 `/Applications`에 설치됩니다.

빌드는 Apple 서명과 공증을 받았기 때문에 앱이 보안 경고 없이 바로 열립니다.

그래도 macOS에서 앱이 손상되었거나 확인할 수 없다고 표시되면 다운로드한 파일이 손상된 것입니다. 릴리스 페이지에서 `.dmg` 또는 `.pkg` 파일을 다시 다운로드하세요.

<a id="install-windows"></a>

## Windows에 설치

릴리스 페이지에서 `.msi` 설치 프로그램(`x64`)을 다운로드하세요: https://github.com/vvnovg/glafira-kafka-releases/releases

1. `.msi` 파일을 실행합니다.
2. Windows 확인 창에서 설치를 승인합니다. 앱은 컴퓨터의 모든 사용자를 위해 설치되므로 관리자 권한이 필요합니다.
3. 시작 메뉴에서 Glafira for Apache Kafka를 실행합니다.

Beta 빌드는 아직 서명되지 않았습니다. SmartScreen이 설치 프로그램을 차단하면 '추가 정보'를 클릭한 다음 '실행'을 클릭하세요. 실행하기 전에 PowerShell에서 파일 차단을 해제할 수도 있습니다:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Linux에 설치

`x86_64`용 빌드가 세 가지 형식으로 제공됩니다. 릴리스 페이지에서 배포판에 맞는 형식을 다운로드하세요: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu 및 파생 배포판: `sudo apt install ./<file>.deb`
- Fedora, RHEL 및 파생 배포판: `sudo dnf install ./<file>.rpm`
- 모든 배포판: `chmod +x <file>.AppImage`로 AppImage에 실행 권한을 주고 실행합니다.

패키지를 설치한 후에는 애플리케이션 메뉴에서 앱을 실행하거나 터미널에서 `glafira-kafka`를 실행하세요.

Glafira는 비밀번호를 Secret Service(예: GNOME Keyring 또는 KWallet)에 저장합니다. 세션에서 이 중 하나가 실행 중인지 확인하세요. 그렇지 않으면 앱이 연결의 비밀번호를 저장할 수 없습니다.

<a id="first-run"></a>

## 첫 실행

창은 두 부분으로 나뉩니다. 왼쪽은 **연결** 패널입니다. 오른쪽에는 탭이 있으며, 토픽 목록이나 Producer처럼 여는 화면마다 별도의 탭이 생깁니다.

아직 연결이 없으면 패널에 짧은 도움말이 표시되고, 탭 영역에서 연결을 추가하라고 안내합니다. 시작하려면:

1. **연결** 패널 헤더에서 `+` 버튼(**새 연결**)을 클릭합니다.
2. **이름** 필드와 **부트스트랩 서버** 필드를 입력합니다. 예: `localhost:9092`.
3. **연결 테스트**를 클릭하여 설정을 확인한 다음 **저장**을 클릭합니다.

저장하면 클러스터 개요가 열립니다. 패널의 각 연결 아래에는 **클러스터 개요**, **토픽**, **컨슈머 그룹**, **Producer**, **컨슈머 세션** 링크가 있습니다. 모든 연결 설정에 대한 자세한 내용은 [연결](connections.md#create)을 참고하세요.

패널 아래쪽에는 **설정**, **프로젝트 후원**, **도움말**, **Glafira 정보** 링크가 있습니다. 필드 옆의 `?` 표시는 해당 필드에 대한 짧은 도움말을 보여 줍니다. 창 어디에서나 `F1`을 누르면 현재 화면에 해당하는 가이드 장이 열립니다. macOS에서는 '도움말' 메뉴에서도 가이드를 열 수 있습니다.

Kafka Explorer(이 앱의 이전 이름)를 사용했다면 Glafira는 첫 실행 시 그 연결, 템플릿, 전송 기록을 복사합니다. 비밀번호도 시스템 보안 저장소에서 복사됩니다. 이미 있는 항목은 덮어쓰지 않으며 이전 데이터도 삭제하지 않습니다. macOS가 이전 키체인 항목에 대한 접근을 한 번 요청할 수 있습니다.

<a id="local-cluster"></a>

## 연습용 로컬 Kafka

공유 클러스터 없이 앱을 사용해 보려면 컴퓨터의 Docker에서 Kafka를 실행하세요. 브로커는 하나면 충분합니다:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

그런 다음 부트스트랩 서버 `localhost:9092`, 프로토콜 `PLAINTEXT`, 환경 `local`로 연결을 만드세요. 브로커가 더 이상 필요하지 않으면 `docker rm -f kafka`로 제거합니다.

비어 있는 **연결** 패널의 도움말에는 `docker compose -f dev/docker-compose.yml up -d`가 나옵니다. 이 파일은 Glafira 소스 코드의 일부입니다. 소스 코드가 있다면 이 명령으로 리스너가 세 개인 브로커를 실행할 수 있습니다:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `PLAIN`(사용자 `tester` / `tester-secret`, `admin` / `admin-secret`)과 SCRAM(`SCRAM-SHA-256`, `SCRAM-SHA-512`용 `scram-user` / `scram-secret`)을 사용하는 `SASL_PLAINTEXT`.
- `localhost:9095` — 클라이언트 인증서(mTLS)를 사용하는 `SSL`. 테스트 인증서는 `dev/certs/out`에 있습니다.

이 사용자와 인증서는 로컬 테스트 전용입니다.

<a id="language"></a>

## 표시 언어

인터페이스는 영어, 러시아어, 독일어, 프랑스어, 스페인어, 이탈리아어, 폴란드어, 포르투갈어(브라질), 중국어(간체, 번체), 일본어, 한국어로 제공됩니다. 모든 언어는 무료입니다.

언어를 바꾸려면:

1. 왼쪽 패널 아래쪽에서 **설정**을 클릭합니다.
2. **표시 언어**에서 언어를 선택합니다.

변경 사항은 이 가이드와 macOS 메뉴를 포함해 즉시 적용됩니다. 기본값인 **시스템 언어**는 운영 체제의 언어를 따릅니다. 그 언어가 목록에 없으면 인터페이스는 영어로 표시됩니다.

<a id="managed-settings"></a>

## 조직에서 설정한 항목

회사에서는 관리자가 정책을 통해 모든 사용자의 일부 옵션을 설정할 수 있습니다. 이러한 옵션은 앱에서 변경할 수 없습니다. 정책으로 할 수 있는 일은 다음과 같습니다:

- 표시 언어 설정 — 이 경우 **설정**의 언어 목록을 사용할 수 없으며, 언어가 조직에서 설정되었다는 안내가 표시됩니다.
- 업데이트 확인을 끄거나 내부 미러로 지정.

정책이 적용되는지 확인하려면 **Glafira 정보**를 열고 **조직 정책** 항목을 보세요. 정책에 오류가 있으면 그곳에 표시됩니다. 정책에 관한 문의는 관리자에게 하세요. 업데이트는 [업데이트 및 진단](updates.md)에서 설명합니다.
