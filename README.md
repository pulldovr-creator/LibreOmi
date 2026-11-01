# LibreOmi

Omi 웨어러블(Friend / DevKit)을 위한 Android 우선 Flutter 앱입니다. BLE로 오디오를
받아 전사하고, 대화·메모리·태스크를 기기 안 SQLite에 저장합니다. 서버도 계정도 없으며,
API 키는 사용자가 앱에서 직접 넣습니다.

이 문서는 **프로젝트 매뉴얼의 진입점**입니다. 세부 절차는 아래 "문서 지도"의 링크를
따르고, 한 주제가 길어지면 `docs/manual/`로 분리해 여기에는 요약과 링크만 남깁니다
(규칙: [`AGENTS.md`](AGENTS.md) §3.2).

## 핵심 원칙

- **로컬 우선**: 대화, 메모리, 태스크, 녹음은 앱 전용 저장소에 남습니다. 내보내기(JSON)와
  백업에서 가져오기를 지원합니다.
- **키는 사용자 소유**: Deepgram·LLM 키는 앱 설정에서 입력하고 Android Keystore로 보호해
  저장합니다. 키가 없으면 어떤 오디오도 외부로 나가지 않습니다.
- **오프라인 옵션**: sherpa-onnx 온디바이스 모델(영어·한국어)로 기기 안에서 전사할 수
  있습니다.
- **Android 우선**: Android가 1차 타깃이고, iOS는 같은 트리에서 빌드 가능한 상태로
  유지합니다.

## 현재 상태

Android 실기기에서 빌드·실행 가능한 상태이고, 아래 기능이 `main`에 구현되어 있습니다.
공개 릴리스 전 남은 일은 [`docs/06-roadmap.md`](docs/06-roadmap.md)의 마일스톤과
[`docs/09-device-verification.md`](docs/09-device-verification.md)의 실기기 검증
런북에 정리되어 있습니다. 변경 이력은 [`CHANGELOG.md`](CHANGELOG.md)를 봅니다.

## 주요 기능

| 영역 | 내용 |
|------|------|
| 실시간 | 기기 스캔·연결, 캡처 시작/중지, 라이브 자막, 더블탭 대화 저장, 싱글탭 질문 |
| 전사 | 클라우드(Deepgram 스트리밍) 또는 로컬(sherpa-onnx, 영어/한국어) |
| 백그라운드 | 포그라운드 서비스로 화면이 꺼져도 캡처 유지, 배터리 최적화 예외 안내, 자동 재연결 |
| 기록 | 대화 목록·상세, 메모리, 태스크(알림 리마인더 포함), 채팅 |
| 데이터 | SD 카드 녹음 동기화, JSON 내보내기/가져오기, 통계, 기기 설정(배터리·펌웨어·게인) |

## 요구사항

- **Flutter 3.47.2 + JDK 17** — `mise.toml`에 고정되어 있고 `mise install`로 재현합니다.
- **Android SDK** — platform 36/35, build-tools 36.0.0/35.0.0, platform-tools,
  cmdline-tools 22.0. 설치는 [`docs/04-android-platform-notes.md`](docs/04-android-platform-notes.md) §1.
- **물리 Android 기기** — 에뮬레이터에는 BLE가 없습니다. Android 12 이상을 권장합니다.
- **Omi 기기**(Friend 또는 DevKit) — 연결·녹음 검증에 필요합니다.

## 빠른 시작

```bash
mise trust
mise install
mise exec -- flutter pub get
mise exec -- flutter doctor -v     # Flutter / Android toolchain / licenses 초록 확인
mise exec -- flutter run -d <android-device-id>
```

디버그 APK만 빌드하려면:

```bash
mise exec -- flutter build apk --debug --target-platform android-arm64
```

## 앱 사용법 (요약)

1. **첫 실행** — 권한 안내 화면을 확인하고 알림·BLE·마이크 권한을 허용합니다.
2. **전사 방식 선택** — 설정에서 `클라우드 (Deepgram)` 또는 `로컬 (오프라인 모델)`을
   고릅니다. 클라우드면 Deepgram API 키를, 대화 요약을 쓰려면 LLM 키를 입력합니다.
   로컬이면 `모델 관리`에서 영어·한국어 모델을 내려받습니다.
3. **기기 연결** — `실시간` 탭에서 스캔 후 Omi 기기에 연결하고 `시작`을 누릅니다.
4. **대화 다루기** — 더블탭은 대화 저장, 싱글탭은 질문과 답변입니다. 저장된 대화는
   `기록`, 추출된 항목은 `메모리`·`태스크`에서 확인합니다.
5. **백그라운드 유지** — 설정에서 배터리 최적화 예외를 허용하는 것을 권장합니다.

권한별 근거와 API 레벨 차이는
[`docs/04-android-platform-notes.md`](docs/04-android-platform-notes.md) §3을 봅니다.

## 문제 해결

- **기기가 보이지 않음** — BLE·위치 권한, 이미 연결된 다른 앱, 기기 페어링 상태를
  확인합니다. Android 11 이하에서는 BLE 스캔에 위치 권한이 필요합니다.
- **자막이 끊기거나 늦음** — logcat에서 MTU(`>= 86`)와 오디오 패킷 길이(`83`)를
  확인합니다. 필터는 [`docs/08-dev-workflow.md`](docs/08-dev-workflow.md) §5에 있습니다.
- **백그라운드에서 멈춤** — 배터리 최적화 예외와 제조사별 절전 정책을 확인합니다.
- **키 관련 오류** — 키는 앱에만 저장되며 저장소·CI에는 없습니다. 설정에서 다시
  입력하고 연결 테스트를 실행합니다.

실기기 전체 점검은 [`docs/09-device-verification.md`](docs/09-device-verification.md)를
순서대로 따라갑니다.

## 개발

- **규칙 원본**: [`AGENTS.md`](AGENTS.md) — 브랜치, gitmoji 커밋, `--no-ff` 병합,
  문서 언어 정책.
- **아키텍처**: [`docs/03-architecture.md`](docs/03-architecture.md).
- **검사**: `mise exec -- flutter analyze`, `mise exec -- flutter test`.
  CI는 `.github/workflows/ci.yml`이 같은 명령을 실행합니다.
- **언어**: 코드·주석·커밋 메시지는 English, 마크다운 문서는 한국어, 이슈/PR은 한국어
  (PR 끝에 English summary 한 줄).

## 문서 지도

| 문서 | 내용 |
|------|------|
| [`docs/01-upstream-analysis.md`](docs/01-upstream-analysis.md) | 업스트림 omibutfree 분석, 이식 가능/불가능한 부분 |
| [`docs/02-tech-stack-decision.md`](docs/02-tech-stack-decision.md) | 기술 선택과 포크 결정의 근거 |
| [`docs/03-architecture.md`](docs/03-architecture.md) | 모듈 구조, 인터페이스, 데이터 흐름, 백그라운드 설계 |
| [`docs/04-android-platform-notes.md`](docs/04-android-platform-notes.md) | 권한, 포그라운드 서비스, BLE MTU, 빌드 도구, 함정 |
| [`docs/05-omi-ble-protocol.md`](docs/05-omi-ble-protocol.md) | 이 앱이 쓰는 Omi BLE GATT 레퍼런스 |
| [`docs/06-roadmap.md`](docs/06-roadmap.md) | 마일스톤, 작업 항목, 인수 조건 |
| [`docs/07-backlog.ko.md`](docs/07-backlog.ko.md) | GitHub 이슈 백로그(작업 항목당 한 항목) |
| [`docs/08-dev-workflow.md`](docs/08-dev-workflow.md) | 브랜치·PR·릴리스·기기 스모크 체크리스트 |
| [`docs/09-device-verification.md`](docs/09-device-verification.md) | 실기기 검증 런북(소유자용) |
| [`SUMMARY.ko.md`](SUMMARY.ko.md) | 결정과 근거, 소유자가 직접 해야 하는 일 요약 |
| [`AGENTS.md`](AGENTS.md) | 코딩 에이전트·기여자를 위한 작업 규칙 |

## 기여

이슈와 PR은 한국어로 작성하고, PR 본문 마지막에 English summary 한 줄을 남깁니다.
브랜치·커밋·병합 규칙은 [`AGENTS.md`](AGENTS.md) §4를 따릅니다. PR 템플릿은
[`.github/pull_request_template.md`](.github/pull_request_template.md)입니다.

## 출처와 라이선스

MIT 라이선스이며 업스트림 저작권 고지를 유지합니다([`LICENSE`](LICENSE)).

애플리케이션 코드(`lib/`, `assets/`, `test/`, `ios/`, `pubspec.yaml` 등)는
[kbdevs/omibutfree](https://github.com/kbdevs/omibutfree)에서 커밋
`98b912f4e46f03bcad818646b51c8f10f35563fe` 기준으로 가져왔습니다. 업스트림 히스토리는
가져오지 않았고, 위 커밋 해시가 출처 기록입니다. 이후 패키지명은 `libreomi`,
Android application id는 `org.libreomi.app`으로 바뀌었습니다.