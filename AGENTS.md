# AGENTS.md — LibreOmi 작업 규칙

이 파일은 이 저장소에서 일하는 **모든 코딩 에이전트(Claude Code, Codex, Cursor, Gemini CLI 등)와 사람 기여자**가 따르는 **단일 규칙 원본**이다. 도구별 안내 파일은 이 문서를 가리키는 포인터일 뿐이며, 규칙 본문을 복제하지 않는다.

| 파일 | 역할 |
|------|------|
| `AGENTS.md` | 규칙 원본(이 문서). 규칙을 바꾸려면 여기를 고친다. |
| `CLAUDE.md` | `AGENTS.md`를 가리키는 심볼릭 링크. 별도 내용을 두지 않는다. |
| 새 도구의 안내 파일(`GEMINI.md`, `.cursorrules` 등) | 필요해질 때 `AGENTS.md` 링크 또는 한 줄 포인터만 추가한다. |

규칙이 충돌하면 이 문서가 우선한다. 현실과 맞지 않는 규칙을 발견하면 회피하지 말고 **같은 PR에서 이 문서를 고치고** PR 본문에 이유를 적는다.

## 1. 프로젝트 개요

LibreOmi는 Omi 웨어러블(Friend / DevKit)을 위한 Flutter 앱이다. BLE로 오디오를 받아
전사하고(Deepgram 또는 온디바이스 sherpa-onnx), 대화·메모리·태스크를 로컬 SQLite에
저장한다. 서버도 계정도 없고, API 키는 사용자가 앱에서 직접 넣는다. Android가 1차
타깃이며 iOS는 빌드 가능 상태를 유지한다. `kbdevs/omibutfree`(MIT)에서 포크했다.

- 플랫폼 분기(`Platform.isAndroid`)는 `platform/` 밖에 두지 않는다. `device/`, `audio/`,
  `transcription/`, `intelligence/`는 플랫폼을 모른다.
- `flutter_blue_plus`, `opus_flutter`, `sherpa_onnx`, `flutter_foreground_task`의 고정
  버전은 전용 이슈 없이 바꾸지 않는다.
- 파서나 상태 머신을 바꾸면 단위 테스트를 같은 PR에 넣는다. BLE는 CI에서 검증할 수
  없으므로 `FakeOmiDevice` 픽스처와 실기기 수동 절차를 함께 제공한다.
- `flutter analyze`는 경고 0을 유지하고, PR 전에 `flutter test`를 실행한다.
- 문서가 코드와 어긋나면 같은 PR에서 문서를 고치고 PR에 명시한다.

## 2. 작업 전에 읽는 문서

| 문서 | 읽는 이유 |
|------|-----------|
| [`docs/03-architecture.md`](docs/03-architecture.md) | 모듈 경계와 인터페이스. 이 구조를 어기면 리뷰에서 반려된다. |
| [`docs/04-android-platform-notes.md`](docs/04-android-platform-notes.md) | API 레벨별 권한, 포그라운드 서비스, BLE MTU, 빌드 도구. |
| [`docs/05-omi-ble-protocol.md`](docs/05-omi-ble-protocol.md) | 프로토콜 상수는 이 문서에만 기록한다. |
| [`docs/06-roadmap.md`](docs/06-roadmap.md) | 지금 구현할 작업 항목과 인수 조건. |
| [`docs/08-dev-workflow.md`](docs/08-dev-workflow.md) | 브랜치·PR·릴리스·기기 검증의 세부 절차. |
| [`docs/09-device-verification.md`](docs/09-device-verification.md) | 실기기 검증 런북(한국어). |

## 3. 문서 작성 규칙

### 3.1 언어

| 대상 | 언어 |
|------|------|
| 소스 코드, 식별자, 주석, docstring | English |
| **저장소의 모든 마크다운**(`README.md`, `AGENTS.md`, `docs/**`, `CHANGELOG.md`, `.github/**/*.md`, 각 디렉터리의 `README.md`) | **한국어** |
| 커밋 메시지 | English — Conventional Commits + gitmoji(§4.2) |
| GitHub 이슈 / PR | 한국어 제목·본문, PR 본문 끝에 한 줄 English summary |
| 앱 문자열 | `lib/l10n/*.arb` — `app_en.arb`가 템플릿, `app_ko.arb`를 같은 PR에서 갱신 |

한국어로 쓰지 않는 예외는 다음뿐이다.

1. 코드, 명령, 경로, 로그, 에러 메시지, 외부 문서 인용 — 원문을 유지하고 필요하면 한국어 설명을 덧붙인다.
2. [`docs/08-dev-workflow.md`](docs/08-dev-workflow.md) §8.1의 Play Console 제출용 선언문 — Play Console에 그대로 붙여넣는 문안이라 English 원문을 유지한다.
3. 도구가 자동 생성하는 파일(`graft/`, `.claude/skills/` 등) — 손으로 고치지 않는다.

기존 영문 문서는 **그 문서를 실질적으로 수정하는 PR에서 통째로 한국어로 옮긴다.** 한 문서 안에서 언어를 섞지 않는다(위 2번 예외 제외). 전환 전까지는 영문 상태로 둔다.

### 3.2 README.md = 프로젝트 매뉴얼

`README.md`는 제품 매뉴얼의 **진입점**이다. 다음 순서를 유지한다.

1. 개요와 핵심 원칙
2. 현재 상태
3. 주요 기능
4. 요구사항
5. 빠른 시작(설치·빌드·실행)
6. 앱 사용법 요약
7. 문제 해결
8. 개발(규칙·CI·테스트 링크)
9. 문서 지도
10. 기여와 라이선스

- README에 긴 절차를 직접 넣지 않는다. 주제마다 **3~5줄 요약 + 링크**만 둔다.
- README가 **150줄 또는 8 KB를 넘으면** 주제별로 `docs/manual/<주제>.md`로 분리한다. 파일명은 ASCII kebab-case를 쓴다: `docs/manual/install.md`, `build.md`, `usage.md`, `troubleshooting.md`, `release.md`.
- 분리한 문서는 상단에 README로 돌아가는 링크를 두고, README의 "문서 지도" 표에 반드시 등록한다. 문서를 만들거나 옮기면 같은 PR에서 링크를 모두 고친다.
- 링크는 상대 경로를 쓴다. 존재하지 않는 파일을 가리키는 링크(스텁)를 남기지 않는다.
- 비밀 키, 토큰, 개인정보, 기기 시리얼을 문서에 쓰지 않는다.

## 4. Git 규칙

### 4.1 브랜치

- 기본 브랜치는 `main`이며 `main`에 직접 커밋·push하지 않는다.
- 작업 브랜치는 이슈 하나당 하나: `lo-<이슈번호>-<슬러그>` (예: `lo-12-ble-mtu`).
  이슈 번호가 없으면 `<type>/<슬러그>` (예: `docs/agents-policy`).
- PR 전에 `main`을 병합해 최신화하고(`git merge main`), 충돌은 브랜치에서 해결한다.
- 병합이 끝난 브랜치는 로컬·원격 모두 삭제한다.

### 4.2 커밋: 유형 분류 + gitmoji

형식:

```
<gitmoji> <type>(<scope>): <English summary>
```

- `summary`: English, 명령형, 72자 이하, 마침표 없음.
- `scope`: 모듈·영역. `ble`, `audio`, `transcription`, `session`, `intelligence`, `platform`, `ui`, `android`, `ios`, `docs`, `ci`, `deps` 등.
- 본문(선택): English로 **왜** 바꿨는지. 이슈 참조는 `Refs #12`, `Closes #12`.

| 유형 | gitmoji | type |
|------|---------|------|
| 기능 추가 | ✨ | `feat` |
| 버그 수정 | 🐛 | `fix` |
| 경미한 수정 | 🩹 | `fix` |
| 문서 | 📝 | `docs` |
| 테스트 | ✅ | `test` |
| 리팩터링 | ♻️ | `refactor` |
| 성능 | ⚡️ | `perf` |
| 포맷·스타일 | 🎨 | `style` |
| 빌드·의존성 | 📦 | `build` |
| 의존성 업그레이드 | ⬆️ | `build` |
| 의존성 다운그레이드 | ⬇️ | `build` |
| CI | 👷 | `ci` |
| 잡무·설정 | 🔧 | `chore` |
| 코드·파일 삭제 | 🔥 | `remove` |
| 보안 | 🔒 | `security` |
| 긴급 수정 | 🚑 | `hotfix` |
| 병합 커밋 | 🔀 | `merge` |
| 릴리스 | 🚀 | `release` |

브랜치 안에서는 커밋을 **유형별로 묶어** 리뷰하기 좋은 순서로 정렬한다. 한 커밋에 서로 다른 유형을 섞지 않는다.

1. ♻️ `refactor` / 🎨 `style` — 동작 변화 없는 정리
2. ✨ `feat` — 새 동작
3. 🐛 `fix` / 🩹 — 작업 중 발견한 결함
4. ✅ `test` — 테스트
5. 📝 `docs` — 문서
6. 🔧 `chore` / 📦 `build` / 👷 `ci` — 잡무

코드 변경과 그 변경을 직접 설명하는 테스트·문서는 같은 커밋에 두어도 된다(원자적 리뷰 단위). 서로 다른 이슈의 변경은 절대 한 커밋에 섞지 않는다.

```
✨ feat(ble): request MTU 512 on Android
🐛 fix(audio): drop packets with a stale sequence number
✅ test(ble): cover the storage-list parser
📝 docs(readme): rewrite the manual in Korean
🔧 chore(ci): pin actionlint
```

### 4.3 병합: 항상 `--no-ff`

`main`으로의 병합은 **항상 `git merge --no-ff`**로 하고 병합 커밋을 남긴다. squash merge, rebase merge, fast-forward 병합은 쓰지 않는다. 브랜치의 커밋 히스토리가 그대로 보존되어야 추적과 롤백이 가능하다.

```bash
git switch main
git pull --ff-only
git merge --no-ff <branch> -m "🔀 merge: <branch> — <English summary>"
git branch -d <branch>
git push origin main
```

- 병합 전 조건: CI(`flutter analyze`, `flutter test`) 통과, PR 승인, 기기 검증이 필요한 변경은 체크리스트 완료.
- 병합 커밋 메시지는 `🔀 merge: <branch> — <summary>` 형식. 본문에 PR 번호와 주요 커밋을 요약한다.
- 병합 후 원격 브랜치도 삭제한다(`git push origin --delete <branch>`).

### 4.4 PR

- 템플릿은 [`.github/pull_request_template.md`](.github/pull_request_template.md). 한국어로 작성하고 마지막에 English summary 한 줄을 둔다.
- PR 하나 = 이슈 하나 = 브랜치 하나.
- `device/`, `audio/`, `session/`, `platform/`을 건드리면 [`docs/08-dev-workflow.md`](docs/08-dev-workflow.md) §5의 기기 스모크 체크리스트를 PR 본문에 채운다. CI는 BLE를 검증하지 못한다.

## 5. 검증 명령

```bash
mise trust                   # 클론마다 한 번, mise가 mise.toml을 읽기 전에
mise install                 # flutter 3.47.2 + temurin-17
mise exec -- flutter doctor -v
mise exec -- flutter pub get
mise exec -- flutter analyze
mise exec -- flutter test
mise exec -- flutter run -d <android-device-id>
```

Android SDK는 mise 도구가 아니다. 기기마다 한 번 [`docs/04-android-platform-notes.md`](docs/04-android-platform-notes.md) §1의 `sdkmanager` 절차로 설치해야 Android 툴체인 검사가 통과한다.

## 6. 에이전트 작업 순서

1. 이 문서와 §2의 관련 문서를 읽고, 어떤 절을 근거로 삼았는지 작업 보고에 적는다.
2. `main`에서 브랜치를 만든다(§4.1).
3. 변경을 유형별 커밋으로 나눠 쌓는다(§4.2). 커밋은 작게, 메시지는 규칙대로.
4. `flutter analyze`와 `flutter test`를 실행하고, BLE·오디오처럼 CI가 못 잡는 경로는 실기기 수동 절차를 함께 적는다.
5. 문서를 한국어로 갱신한다(§3). 코드가 문서와 달라졌다면 문서를 고친다.
6. PR을 한국어로 연다(§4.4). 병합은 `--no-ff`(§4.3).

<!-- graft:start -->
## Graft — repo context graph

This repo is indexed in `graft/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `graft map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `graft ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `graft grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `graft skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `graft callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `graft/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `graft ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `graft build` (deterministic,
no API key, $0).
<!-- graft:end -->