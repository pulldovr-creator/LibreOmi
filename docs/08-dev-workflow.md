# 08 — 개발 워크플로

## 1. 언어 정책

규칙 원본은 [`AGENTS.md`](../AGENTS.md) §3.1과 §4.2이고, 이 표는 그 요약이다.

| 산출물 | 언어 | 이유 |
|--------|------|------|
| 소스 코드, 식별자, 주석, docstring | English | 생태계 기본값. English 맥락에서 Dart 생성 품질이 좋다 |
| `README.md`, `AGENTS.md`, `docs/`, `CHANGELOG.md`, `.github/**/*.md` | **한국어** | 프로젝트 소유자가 읽고 유지보수한다. 매뉴얼은 한국어가 기본이다 |
| `docs/08` §8.1 (Play Console 선언문) | English | Play Console에 그대로 붙여넣는 원문이라 번역하지 않는다 |
| 커밋 메시지 | English, Conventional Commits + gitmoji (`✨ feat(ble): request MTU 512 on Android`) | 도구 연동과 체인지로그 생성. 유형 분류는 §2.2 |
| GitHub 이슈/PR | **한국어** 제목·본문, PR 본문 끝에 English summary 한 줄 | 소유자가 한국어로 읽고 쓴다. English 한 줄은 히스토리 검색용 |
| 앱 문자열 | `lib/l10n/*.arb`의 영어·한국어 (LO-62) | `app_en.arb`가 템플릿이고, `app_ko.arb`를 같은 PR에서 번역한다. `test/l10n/arb_parity_test.dart`가 어긋나면 실패한다 |
| 에이전트가 만드는 소유자용 요약 | 한국어 | 소유자 가독성. 예: `SUMMARY.ko.md` |

기존 영문 문서는 그 문서를 실질적으로 수정하는 PR에서 통째로 한국어로 옮긴다. 한 문서
안에서 언어를 섞지 않는다(§8.1 예외). 공개 이후 외부 기여자의 이슈는 어떤 언어든
허용하며, 메인테이너는 이슈의 언어로 답한다.

## 2. 브랜치와 PR 흐름

- `main`은 보호 브랜치이고 CI가 통과해야 한다. 워크플로는
  `.github/workflows/ci.yml`(LO-04)이다. PR과 `main` push마다 `ubuntu-latest` 잡 하나가
  `flutter pub get` → `flutter analyze` → `flutter test` →
  `flutter build apk --debug --target-platform android-arm64`를 실행한다. Flutter와 JDK
  버전은 `mise.toml`에서 읽어 오므로 CI와 로컬 체크아웃이 어긋나지 않는다. analyze는
  `--no-fatal-infos --no-fatal-warnings`로 실행한다. 에러만 실패시키고, 업스트림에서
  물려받은 경고·정보는 실패시키지 않는다. 그 정리는 LO-04 범위 밖이며 아직 백로그에
  없다. 디버그 APK는 컴파일 확인용이고 업로드하지 않는다. 릴리스 산출물은
  `v*` 태그에서만 도는 `.github/workflows/release.yml`(LO-63)이 만든다. 현재 이 두
  워크플로는 실행되지 않는다. 이 계정은 결제 문제로 GitHub Actions가 비활성화되어 있어,
  둘 다 `actionlint`와 동일 명령의 로컬 실행으로 검증한다.
- 백로그 항목 하나에 브랜치 하나: `lo-12-ble-mtu`. 브랜치 하나에 PR 하나.
- `main` 병합은 **항상 `--no-ff`** 병합 커밋으로 한다. squash merge와 rebase merge는
  쓰지 않는다(절차와 메시지 형식: [`AGENTS.md`](../AGENTS.md) §4.3).
- PR 템플릿(한국어)이 요구하는 항목: 목적, 변경 내용, 테스트 방법(에뮬레이터로 불가한
  항목은 실기기 명시), 스크린샷/로그, 관련 이슈(`Closes #n`).
- `device/`, `audio/`, `session/`, `platform/`을 건드리는 PR은 설명에 **기기 스모크
  체크리스트**(§5)를 채워 넣어야 한다. CI는 BLE를 검증할 수 없다.

### 2.1 커밋

커밋은 브랜치 안에서 유형별로 묶고, 한 커밋에 서로 다른 유형을 섞지 않는다. 형식은
`<gitmoji> <type>(<scope>): <English summary>`이고, 유형 표와 정렬 순서는
[`AGENTS.md`](../AGENTS.md) §4.2에 있다.

### 2.2 PR 제목

PR 제목은 Conventional Commits 접두사로 시작한다(예: `feat(ble): BLE MTU 협상`).
릴리스 초안 스크립트(`scripts/changelog_draft.py`)가 제목의 접두사로
Added/Changed/Fixed/Removed 절을 나누기 때문이다. 접두사가 없으면 `Changed`로 떨어진다.

## 3. 완료 정의

1. `06-roadmap.md`의 인수 조건을 충족하고 PR에 명시했다.
2. 변경한 파일에서 `flutter analyze` 경고가 0이다.
3. 파서·상태 머신·저장소 변경에는 단위 테스트를 넣었다.
4. 동작이나 프로토콜 가정이 바뀌면 문서를 갱신했다(특히 `05-omi-ble-protocol.md`).
5. 네이티브 플러그인(BLE, opus, sherpa, foreground task)에 새 `^` 버전 범위를
   추가하지 않았다.

## 4. AI 코딩 에이전트와 작업하기

백로그는 이슈 하나가 에이전트에게 자기완결적이도록 작성되어 있다. 이슈를 위임할 때:

1. 이슈 본문을 붙이고, 에이전트가 [`AGENTS.md`](../AGENTS.md)와 그 문서가 링크하는
   문서를 읽게 한다.
2. 어떤 문서 절을 근거로 삼았는지 보고하고, 틀렸으면 같은 PR에서 고치게 한다.
3. 에이전트는 BLE를 테스트할 수 없다. 순수 로직 단위 테스트, 필요한 경우
   `FakeOmiDevice` 픽스처, 소유자가 폰에서 실행할 정확한 `adb logcat` 필터와 수동 절차를
   산출물로 내야 한다.
4. 소유자가 기기 스모크 체크리스트를 실행하고 결과를 PR에 붙인다(한국어 무방).

분담 제안: 에이전트가 M0, M3(리팩터링+테스트), M4–M6 로직을 맡고, 소유자가 M1·M2를
직접 기기에서 수행한다. 그 마일스톤은 기기별 동작과 사람이 관찰해야 하는 권한
다이얼로그가 지배적이기 때문이다.

## 5. 기기 스모크 체크리스트 (PR에 복사)

`device/`, `audio/`, `session/`, `platform/`을 건드리는 **모든 PR**에 들어가는 짧은
형식이다. 긴 형식은 PR #40–#86의 소유자 검증 단계를 중복 제거·재정렬한 하나의
워크스루로, 기대값과 캡처 방법까지 담고 있다:
**[`09-device-verification.md`](09-device-verification.md)**. 배치 검증 세션에서는 긴
형식을, PR마다는 아래 체크리스트를 쓴다. 둘이 어긋나면 같은 PR에서 둘 다 고친다.

```
기기: <제조사/모델/Android 버전>   펌웨어: <Omi firmware>   빌드: <commit>
[ ] 권한 요청 순서 정상 (알림 → BLE → (마이크))
[ ] 스캔에서 Omi 표시, 연결 10초 이내
[ ] logcat 에서 MTU >= 86 확인, 오디오 패킷 길이 83
[ ] 라이브 자막 2초 이내 표시
[ ] 화면 끄고 10분 후에도 자막 누적 (M2 이후)
[ ] 범위 이탈 → 복귀 시 30초 내 재연결 (M2 이후)
[ ] 더블탭 → 대화 저장 알림, History 에 표시
[ ] 싱글탭 → 질문 → 싱글탭 → 알림으로 답변
[ ] 배터리/펌웨어/게인/LED 읽기·쓰기
[ ] 앱 종료 후 재시작 시 자동 재연결
```

유용한 logcat 필터:

```bash
adb logcat -s flutter:V FlutterBluePlus:V BluetoothGatt:V
adb shell dumpsys activity services | grep -i foreground
adb shell dumpsys deviceidle whitelist | grep libreomi
```

## 6. 비밀키와 테스트 계정

- Deepgram 키와 LLM 키는 앱에서만 입력하며 절대 커밋하지 않는다. CI에는 키가 없고,
  테스트는 페이크를 쓴다.
- 노트북에서 도는 에이전트를 위해 테스트에는 `sqflite_common_ffi`를 써서 기기 DB가
  필요 없다.

## 7. 릴리스 (LO-63)

### 7.1 버전 관리

`pubspec.yaml`이 `version: <name>+<code>`를 갖는다. LibreOmi는 `0.1.0+1`에서 다시
시작한다. 업스트림 omibutfree의 `2.1.0`은 이 포크를 설명하지 않으므로 이어 쓰지 않는다.

- `<name>`은 semver이고, `vX.Y.Z` 태그와 GitHub Release 이름이 이 값을 따른다.
- `<code>`는 Android `versionCode`가 되며 **항상 증가**한다. 버전 이름이 내려가도
  마찬가지다. Android와 Play 모두 code가 뒤로 간 업데이트를 거부한다. 이 머신을 떠나는
  모든 빌드에서 올리고, 버전 이름을 올릴 때만 올리는 것이 아니다.
- `--split-per-abi`를 쓰면 Flutter Gradle 플러그인이 APK마다 `1000 * ABI_VERSION`을
  더하므로, 한 빌드의 arm64·armeabi APK가 자동으로 서로 다른 순서의 code를 갖는다.

### 7.2 서명 키

키는 소유자의 것이고 커밋되지 않으며 재생성할 수 없다. **키를 잃으면 이후 어떤 빌드도
설치된 앱을 업데이트할 수 없다.** 키스토어와 비밀번호를 이 저장소도, 이 노트북 하나도
아닌 곳에 백업한다.

한 번만 생성한다:

```bash
keytool -genkey -v -keystore ~/libreomi-release.jks \
  -keyalg RSA -keysize 4096 -validity 10000 -alias libreomi
```

그다음 `android/key.properties`로 빌드가 키를 찾게 한다. 이 파일은 git-ignore이며
`.gitignore`가 `*.jks` / `*.keystore`도 덮어 키스토어가 실수로 추가되지 않는다:

```properties
storeFile=/absolute/path/to/libreomi-release.jks
storePassword=<store password>
keyAlias=libreomi
keyPassword=<key password>
```

`android/app/build.gradle.kts`는 서명 자료를 다음 순서로 찾는다.

1. `LIBREOMI_KEYSTORE_PATH`, `LIBREOMI_KEYSTORE_PASSWORD`, `LIBREOMI_KEYSTORE_ALIAS`,
   `LIBREOMI_KEY_PASSWORD` 환경변수 — CI가 시크릿으로 주입하는 경로
2. `android/key.properties`
3. 둘 다 없으면 릴리스 빌드가 **디버그** 키로 대체되고, 빠진 항목을 이름으로 알리는
   Gradle 경고를 남긴다. 기여자와 CI 컴파일 확인에서 `flutter build apk --release`가
   계속 동작하게 하기 위해서다. 디버그로 서명된 APK는 절대 배포하지 않는다.

`flutter build`는 Gradle 출력을 걸러 내므로 그 경고는 직접 Gradle을 실행할 때만 보인다
(`cd android && ./gradlew :app:signingReport`). **권위 있는 확인은 서명 인증서이지,
경고의 부재가 아니다.** `scripts/release.sh`는 실행할 때마다 인증서를 출력하고,
디버그로 서명된 산출물의 GitHub Release 생성을 거부한다.

어떤 키가 APK에 서명했는지 확인한다. `scripts/release.sh`가 이 값을 출력하고
`CN=Android Debug`를 보면 크게 경고한다:

```bash
# newest installed build-tools, the same one release.yml picks (release.sh prefers an
# apksigner already on PATH, then falls back to this)
"$(find "$ANDROID_HOME/build-tools" -mindepth 2 -maxdepth 2 -name apksigner | sort -V | tail -n1)" \
  verify --print-certs build/app/outputs/flutter-apk/app-arm64-v8a-release.apk
```

CI에서는 같은 키스토어를 저장소 시크릿 `LIBREOMI_KEYSTORE_BASE64`
(`base64 -i ~/libreomi-release.jks`), `LIBREOMI_KEYSTORE_PASSWORD`,
`LIBREOMI_KEYSTORE_ALIAS`, `LIBREOMI_KEY_PASSWORD`에 넣는다.

### 7.3 릴리스 체크리스트

1. 릴리스 노트 초안을 만든다: `scripts/changelog_draft.py --tag vX.Y.Z`가 병합된 PR에서
   Keep a Changelog 절을 출력한다. (한국어) PR 제목을 사용자 관점 문장으로 다듬어
   `CHANGELOG.md`에 붙여넣는다. 개별 PR은 의도적으로 `CHANGELOG.md`를 건드리지 않는다.
   병렬 PR이 병합마다 충돌하기 때문이다.
2. §7.1에 따라 `pubspec.yaml`의 `version:`을 올리고, 그 변경을 체인지로그 수정과 함께
   병합한다.
3. `scripts/release.sh --dry-run` — `app-arm64-v8a-release.apk`,
   `app-armeabi-v7a-release.apk`, `app-release.aab`을 빌드하고 크기·sha256·서명 인증서를
   출력하며 아무것도 업로드하지 않는다. 인증서가 디버그 키가 **아님**을 확인한다.
4. arm64 APK를 폰에 설치하고 §5 스모크 체크리스트를 기기 두 대에서 실행한다.
5. `git tag vX.Y.Z && git push origin vX.Y.Z`. 이 태그 push에
   `.github/workflows/release.yml`이 반응한다. 이 계정에서 Actions가 비활성인 동안에는
   `scripts/release.sh`를 `--dry-run` 없이 로컬에서 실행한다. 같은 세 산출물이 붙은
   드래프트 릴리스가 만들어진다.
6. GitHub에서 드래프트 릴리스 편집: §1에 따라 한국어 노트와 English 한 줄 요약을 넣고
   공개한다.
7. Play 내부 테스트에는 `app-release.aab`를 쓴다. 이 저장소의 자동화 범위 밖이므로 Play
   Console에서 수동 업로드한다
   (<https://support.google.com/googleplay/android-developer/answer/9859152>). 업로드
   전에 §8을 다시 읽고 현재 앱 내 고지 문안을 Play Console의 민감 권한 선언에 붙여넣는다.
   Play는 양식의 주장과 앱이 보여 주는 내용을 대조한다.

## 8. Play 민감 권한과 고지 (LO-64)

Play는 마이크·위치·항상 백그라운드 접근을 쓰는 앱이 **데이터 사용 전에, 앱 안에서**,
사용자가 놓칠 수 없는 화면으로 무엇을 왜 수집하는지 알리도록 요구한다. LibreOmi는
`lib/pages/permissions_rationale_page.dart`로 이를 수행한다. 첫 실행에서 한 번
(`SettingsService.rationaleShown`) 보여 주고, 이후에는 실시간 탭에서 다시 열 수 있다.
이 화면은 권한 상태를 **표시**만 하고, 각 권한은 실제 사용 시점에 요청한다.
`docs/04-android-platform-notes.md` §3이 요구하는 방식이다.

**이 절과 그 화면을 함께 유지한다.** 화면이 고지이고, 아래 문안은 그 화면을 Play
Console용으로 바꾼 것이다. 한쪽을 바꾸면 같은 PR에서 다른 쪽도 바꾼다.

### 8.1 선언 초안 (English, Play Console에 그대로 붙여넣기)

> LibreOmi records audio from a connected Omi wearable or from the phone's microphone, and only
> while the user has started a capture. The audio is transcribed either entirely on the device or,
> if the user enters their own Deepgram API key, by sending it to Deepgram. Conversation text is
> sent to an LLM endpoint only when the user supplies a key for one. LibreOmi has no server and no
> account: with no keys entered, no capture is sent to any service. Transcripts, conversations,
> memories, tasks and chat history are stored in the app's private SQLite database, and recordings
> synced from the wearable's SD card are stored as audio files in the app's private support
> directory. API keys are stored encrypted under a key held in the Android keystore. Android's
> automatic backup is left on (`allowBackup` default), so the database and the synced recordings
> are part of the user's Google account backup; the API keys and the downloaded speech models are
> excluded from it (`res/xml/backup_rules.xml`, `res/xml/data_extraction_rules.xml`). The user can
> export the four content tables as JSON or remove everything by clearing the app's data.
>
> Permissions and their purpose:
>
> - `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` — discover and connect to the user's Omi wearable.
>   The scan is filtered to Omi devices and is never used to derive location; the manifest
>   carries `android:usesPermissionFlags="neverForLocation"`.
> - `ACCESS_FINE_LOCATION` (Android 11 and below only, `maxSdkVersion="30"`) — required by those
>   OS versions for any BLE scan. Location is never read.
> - `RECORD_AUDIO` — capture from the phone microphone when the user picks it as the source.
> - `POST_NOTIFICATIONS` — the ongoing capture notification and conversation-saved alerts.
> - `FOREGROUND_SERVICE` (`microphone` / `connectedDevice` types) — keep capture and transcription
>   running while the screen is off. No data is collected that is not already covered above.
> - `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` — optional; OEM battery managers otherwise stop the
>   capture service.
> - `SCHEDULE_EXACT_ALARM` — optional, and only used when the user turns on exact task reminders.
>   A reminder is delivered inexactly when the permission is absent; no additional data is
>   collected either way.

### 8.2 제출 전

- 각 항목을 `android/app/src/main/AndroidManifest.xml`과 대조한다. 매니페스트가
  권위이고 이 목록은 사본이다.
- 매니페스트의 모든 권한이 위 목록에 있고 고지 화면에 행이 있어야 한다. 앞으로 이슈가
  권한을 추가하면 같은 PR에서 둘 다 추가한다. 매니페스트보다 좁은 선언은 Play가
  거부하는 바로 그 형태다.
- 같은 사실로 데이터 안전성 양식을 채운다. 오디오와 "기타 사용자 생성 콘텐츠"는
  수집되며, 사용자가 직접 설정한 전사/LLM 엔드포인트 외에는 제3자와 공유하지 않고,
  광고나 분석에 쓰지 않는다.
- 백업 주장은 두 XML 규칙 파일이 사실일 때만 참이다. 앞으로 이슈가 데이터
  디렉터리를 추가하면(LO-50이 `<app support>/sdcard/`를 추가한 것처럼) 제외 여부를
  정하고 이 절, 고지 화면, `docs/04-android-platform-notes.md` §9를 함께 고친다.