# 작업 이어받기 메모

이 문서는 집이나 다른 컴퓨터에서 같은 작업을 이어가기 위한 맥락 요약입니다. 원문 대화 전체가 아니라, 이어 작업에 필요한 결정사항과 현재 상태만 정리했습니다.

## 목표

반에서 사용할 수 있는 캐치마인드식 미술 스마트드로잉/그림 퀴즈 사이트를 만든다.

원하는 느낌:

- 교사가 방을 만들고 4자리 방 코드를 공유한다.
- 학생들이 각자 기기로 접속해 이름과 방 코드로 참여한다.
- 교사 화면에는 제시어와 그림판이 보인다.
- 학생 화면에는 그림과 정답 입력창이 보인다.
- 정답이 맞으면 채팅/점수/라운드 상태가 반영된다.
- 수업에서 바로 쓰기 쉬운, 재미있는 미술 활동 도구가 목표다.

참고한 외부 레퍼런스:

- `https://magical-cupcake-f1862a.netlify.app/`
- 해당 사이트는 Firebase Auth + Realtime Database 계열을 쓰는 것으로 보였고, 전체 감성은 참고하되 그대로 복제하지 않고 우리 수업용으로 직접 구현하는 방향으로 결정했다.

## 현재 파일

- `index.html`: 처음 만든 스마트드로잉 미술 활동 프로토타입
- `class_catchmind.html`: 같은 컴퓨터/교실 시연용 오프라인 캐치마인드형 버전
- `class_catchmind_online.html`: 온라인 방 코드 + Firebase 준비형 버전, 현재 핵심 파일
- `README.md`: repo 기본 안내

## 현재 구현 상태

`class_catchmind_online.html`에 들어간 기능:

- 교사/학생 시작 화면
- 교사용 방 만들기
- 학생용 방 코드 참여
- 6학년 고정, 반/번호 입력 기반 학생 입장
- 대기실
- 라운드 시작/다음 라운드
- 교사용 그림판
- 학생용 정답 입력
- 정답 채팅 목록
- 자동 채점 모드
- 점수/참여자 표시
- 같은 브라우저 여러 탭에서 동기화되는 로컬 fallback
- Firebase SDK 및 웹 앱 설정값 연결
- Firebase가 준비되면 `/rooms/{방코드}` 경로로 방 데이터 저장/구독
- Firebase DB가 아직 안 되면 로컬 테스트로 계속 동작

## 2026-07-09 레퍼런스 기반 추가 개선

레퍼런스 사이트에서 수업 진행감을 주는 요소를 확인하고 다음 기능을 추가했다.

- 교사 설정에 `라운드 수` 선택 추가
  - 3, 5, 8, 12라운드 중 선택
- 교사 설정에 `효과음 ON/OFF` 추가
  - 라운드 시작, 정답, 최종 종료에 짧은 톤 재생
- 시간이 끝나거나 교사가 정답 공개를 누르면 5초 후 자동으로 다음 단계로 이동
  - 첫 정답자가 나와도 문제는 바로 공개되지 않아 여러 학생이 같은 라운드에서 계속 맞힐 수 있음
  - 마지막 라운드가 끝나면 최종 결과 상태로 전환
- 학생 참여 화면에 태블릿 가로 모드 안내 추가
- 학생 참여 화면을 이름 입력 대신 `6학년 + 반 + 번호 + 방 코드` 구조로 변경
  - 참가자 id는 `g6-c반-n번호` 형태라 같은 학생이 다시 접속하면 중복이 아니라 기존 참가자로 갱신됨
- 학생 정답 입력에서 일부 부적절한 표현 차단
  - 한글, 초성, 띄어쓰기 우회 표현 일부를 정규화해서 필터링
- 학생이 튕겼다가 같은 반/번호로 재입장하면 최신 Firebase 방 데이터를 우선 읽어 기존 점수와 현재 문제 상태를 유지
- 새창 운영창 추가
  - 교사 대기실/게임 화면의 `운영창 열기` 버튼 또는 `?room=방코드&admin=1` 주소로 별도 탭 운영 가능
  - 학생별 `채팅 봉인`은 해제 전까지 정답/채팅 제출을 막음
  - 학생별 `이번 문제 봉인`은 현재 라운드만 정답 제출을 막고 다음 라운드에서 자동 해제됨
- 정답 순서 기반 카훗식 점수로 변경
  - 제한 시간 안 정답만 점수 획득
  - 라운드별 1등 100점, 2등 90점, 3등 80점, 4등 70점, 5등 60점, 6등 이후 50점
  - 무응답이나 시간 종료 후 입력은 점수 없음
  - 정답 채팅에는 `정답 n등 +점수`가 표시됨
- Firebase 동시성 보강
  - 학생 heartbeat는 방 전체를 덮어쓰지 않고 본인 접속 상태만 부분 업데이트
  - 학생 입장은 Firebase transaction으로 처리해 여러 명이 동시에 들어와도 참가자 목록이 누락되지 않도록 보강
  - 정답 제출은 Firebase transaction으로 처리해 여러 학생이 맞혀도 정답 순서/점수 기록이 덮이지 않도록 보강
  - 교사 그림 저장, 정답 공개, 다음 문제, 운영창 봉인 조작도 최신 방 상태 기준 transaction으로 처리
  - 다음 문제 버튼/자동 진행이 중복 실행되어 라운드를 건너뛰지 않도록 진행 중 잠금 추가
- 정답 단어 기본 예시를 72개로 확장
- 대기실에 참여 링크 표시
- 진행판에 `현재 라운드 / 전체 라운드` 표시
- 최종 결과 상태에서 1등과 점수를 보여주고, 대기실로 돌아가면 새 게임을 시작할 수 있도록 라운드/점수 초기화

확인한 것:

- 스크립트 문법 검사 통과
- 로컬 앱 초기 로드 콘솔 오류 없음
- 교사 설정 화면에서 라운드 수/효과음 옵션 노출 확인
- 방 생성 및 참여 링크 표시 확인
- 테스트용 3라운드 전체 진행, 재입장, 최종 결과 표시 확인
- Firebase Realtime Database 생성 및 `/rooms` 규칙 게시 완료

검증한 것:

- HTML 내 `<script>` 문법 검사 통과
- 로컬 서버 `http://127.0.0.1:8765/class_catchmind_online.html` 로드 확인
- 교사용 방 생성 확인
- Firebase DB가 없을 때도 로컬 fallback으로 방 생성이 유지되는 것 확인
- 설치된 Chrome을 헤드리스 테스트 브라우저로 사용해 교사/학생 분리 운영 검증
  - 방 생성, 6학년 3반 12번 입장, 그림판 입력, 부적절한 표현 차단
  - 3라운드 정답 처리와 `100 -> 200 -> 300` 점수 누적
  - 학생 새로고침/재입장 후 점수 유지
  - 최종 결과 `1등 · 300점` 표시
  - 테스트 방 Firebase 데이터 삭제 확인
- 설치된 Chrome 헤드리스로 교사 1명 + 운영창 1개 + 학생 3명 전체 흐름 검증
  - 운영창 채팅 봉인 학생은 강제 제출을 시도해도 0점 유지
  - 운영창 이번 문제 봉인 학생은 현재 라운드에서 0점 유지, 다음 라운드에서 자동 해제
  - 라운드별 정답 순서 점수 `100, 90, 80` 배분 확인
  - 3라운드 최종 점수 `12번 270점, 13번 190점, 14번 190점` 확인
  - 테스트 방 Firebase 데이터 삭제 확인
- 전체 코드 재점검 후 추가 검증
  - 학생 8명 동시 입장 테스트에서 Firebase 참가자 8명 유지 확인
  - 다음 문제 버튼을 빠르게 2번 눌러도 라운드가 1개만 진행되는 것 확인

## Firebase 상태

Firebase 프로젝트:

- 프로젝트 ID: `school-debate-1`
- Firebase 웹 앱 설정값은 `class_catchmind_online.html`에 이미 추가되어 있다.
- Google Analytics는 이 수업용 앱에 필요하지 않아서 꺼두는 방향으로 진행했다.

완료된 부분:

- Firebase Console에서 Realtime Database를 생성했다.
- 생성 위치: `미국(us-central1)`
- DB URL: `https://school-debate-1-default-rtdb.firebaseio.com`
- 코드의 `databaseURL` 값과 생성된 DB URL이 일치한다.
- Realtime Database Rules는 앱이 쓰는 `/rooms/{방코드}` 경로만 읽기/쓰기를 허용하고, 루트는 닫아두는 방식으로 게시했다.
- REST 테스트로 `/rooms/__codex_rule_test__`에 쓰기/읽기/삭제가 모두 `HTTP 200`으로 통과하는 것을 확인했다.

현재 규칙:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": true,
        ".write": true
      }
    },
    ".read": false,
    ".write": false
  }
}
```

다음 Firebase 작업:

1. 교사 브라우저 1개 + 학생 브라우저/휴대폰 1개로 실제 multi-device 테스트
2. 수업 전에 방 코드, 닉네임, 그림 데이터가 의도대로 동기화되는지 확인
3. 실제 배포 전에는 필요하면 Firebase Auth 또는 더 엄격한 Rules 검토

주의:

- Firebase 웹 API key는 브라우저 앱 공개 설정값이라 서버 비밀번호는 아니다.
- 실제 보안은 Realtime Database Rules에서 해야 한다.
- 현재 `/rooms` 경로는 수업 프로토타입용 공개 읽기/쓰기 상태라 학생 개인정보는 넣지 않는 것이 좋다.

## GitHub 상태

비공개 저장소:

- `https://github.com/mdsks070-ctrl/class-catchmind-online`

현재 브랜치:

- `main`

처음 업로드한 커밋:

- `76c41a2`
- 메시지: `Upload classroom drawing apps`

집에서 이어받기:

```bash
git clone https://github.com/mdsks070-ctrl/class-catchmind-online.git
cd class-catchmind-online
```

로컬 미리보기 예시:

```bash
python -m http.server 8765
```

브라우저에서 열기:

```text
http://127.0.0.1:8765/class_catchmind_online.html
```

## 다음 개발 추천 순서

1. 교사 브라우저 1개 + 학생 브라우저/휴대폰 1개로 실제 multi-device 테스트
2. 방 데이터 구조가 안정적이면 Firebase Rules 정리
3. GitHub Pages, Netlify, Vercel, Firebase Hosting 중 하나로 공개 URL 배포
4. 수업용 UX 다듬기
   - 방 코드 크게 표시
   - 학생 입장 QR 코드
   - 정답자 효과
   - 라운드 기록
   - 단어 목록 프리셋
   - 그림판 선/지우개 안정화

## Codex에게 이어서 시킬 때

다음 프롬프트로 시작하면 된다.

```text
이 repo의 HANDOFF.md와 README.md를 먼저 읽고, class_catchmind_online.html의 Firebase Realtime Database 연결을 실제 기기 2대로 테스트해줘. 현재 DB는 생성됐고 `/rooms` 규칙은 수업 프로토타입용으로 열려 있으며, 학생 입장은 6학년 고정 + 반/번호 입력 방식이야. 목표는 반에서 여러 기기로 접속하는 캐치마인드식 그림 퀴즈야.
```

## Deployment

- Public app URL: `https://mdsks070-ctrl.github.io/class-catchmind-online/`
- Student join URL format: `https://mdsks070-ctrl.github.io/class-catchmind-online/?room=ROOMCODE`
- Admin URL format: `https://mdsks070-ctrl.github.io/class-catchmind-online/?room=ROOMCODE&admin=1`
- GitHub Pages source: `gh-pages` branch, `/ (root)`.
- Repository visibility was changed to public so GitHub Pages can serve the app.
- Current deployed branch contains `index.html` and `class_catchmind_online.html`, both pointing to the classroom app.

## 2026-07-09 Student Drawer Update

- `class_catchmind_online.html` now assigns one student as `drawerId` every round.
- The teacher/admin screens still show the secret word for supervision, but only the assigned student sees the word as a student.
- Only the assigned student can draw on the canvas. Other students see the drawing and can submit answers.
- The drawer cannot submit an answer for their own round; this is blocked in the answer permission logic, not only hidden in the UI.
- Teacher game controls now include `그릴 학생 바꾸기`; the admin panel also has `그리기 지정`.
- Changing the drawer clears the current canvas so two students' drawings do not mix.
- Verified with an automated multi-tab E2E test: teacher + students 12/13/14, first drawer 12, correct answer score 100 for 13, drawer switched to 13, admin drawer badge visible.

## 2026-07-09 Landscape Answer Fix

- Small mobile landscape screens now pin the student answer form to the bottom of the viewport while the student is a guesser.
- Drawer students still do not see the answer form; their drawing toolbar stays available.
- Verified with an automated 844x390 landscape E2E test: answer input visible inside viewport, fixed bottom bar active, answer submission scored correctly.

## 2026-07-09 Student Answer Log Privacy

- Student answer logs no longer reveal other students' submitted answer text during an active round.
- Students still see their own submitted text, and other students' entries appear as `답안 제출됨` or `정답 n등 +점수`.
- Teacher game view and admin view still show the raw answer text for supervision.
- Verified with an automated multi-tab E2E test: student 13 answered correctly, student 14 saw only `정답 1등 +100점`, teacher/admin saw the raw selected word.

## 2026-07-09 Round And Answer Reliability

- Student heartbeat no longer bumps the room-wide `updatedAt`, preventing stale local snapshots from blocking Firebase round/answer updates.
- Remote room updates are accepted when round, guesses, winners, strokes, reveal state, or drawer state differ, even if a stale local timestamp is newer.
- `nextRound()` now checks the expected round/status inside the transaction, preventing double-click or admin/teacher simultaneous clicks from skipping a round.
- Admin window now has `정답 공개` and `다음 문제/게임 시작/결과 보기` controls.
- Teacher/admin answer logs now show `정답 인정` for unscored guesses; accepting a guess awards rank-based points through the same scoring helper as auto scoring.
- Manual answer mode label changed to `교사 확인 후 인정`.
- Verified with a Firebase multi-context E2E test: admin started game, student 13 auto-scored 100, student 14 was manually accepted for 90, simultaneous teacher/admin next clicks advanced only to round 2.

## 2026-07-09 Correct Countdown Flow

- The app no longer reveals the actual prompt to students when a correct answer is processed.
- First accepted correct answer starts a 5-second countdown with the timer title `1등 정답`.
- During those 5 seconds, other students can still answer and receive rank-based points.
- After the countdown, teacher/admin auto-advances to the next prompt. Existing double-advance protection prevents skipped rounds if both windows are open.
- The old `정답 보기/정답 공개` buttons now read `5초 후 넘기기` and start the same countdown without revealing the prompt.
- Timeout also starts the countdown without revealing the prompt.
- Verified with a Firebase multi-context E2E test: student 13 got 100, student 14 got 90 during countdown, student view kept the prompt hidden, then round 2 loaded a new prompt.

## 2026-07-09 Connection Reliability Hardening

- Added visible Firebase sync status badges in the lobby, game, and admin screens.
- Room creation now waits for the first remote Firebase write before announcing that the room is ready.
- Student join and admin URL hydration now retry remote reads several times before showing a room-code failure.
- Student join now retries the Firebase transaction before falling back to local state and background resync.
- The join/create buttons are temporarily disabled while the operation is in progress to prevent duplicate actions.
- Verified with a Firebase multi-context E2E test: teacher/admin/student 12/13/14 all reached online sync, student 13 scored 100, student 14 scored 90 during the 5-second countdown, round 2 auto-started, and student 14 rejoined with the 90-point score preserved.

## 2026-07-09 Answer Input Dock

- Removed the unused `학생 답변이 여기에 쌓입니다.` helper text from the answer panel.
- Guessing students now get a large fixed answer bar at the bottom of the screen in both portrait and landscape layouts.
- The answer input keeps focus after submit so students can type several guesses quickly.
- Verified with an automated Chrome UI test: portrait and landscape answer bars stayed inside the viewport, input and send button remained aligned, and focus returned to the answer field after submission.

## 2026-07-09 Reference-Style Student UI And QR Entry

- Student game screens now hide the teacher-style progress panel and answer log, leaving the drawing area, drawer identity, timer meta, and answer dock.
- The drawing canvas shows a prominent drawer banner so students can immediately see who is drawing.
- Drawer students see `내 차례` and do not see the answer panel.
- The lobby now shows a QR code for the room join URL. Students who open the QR link skip room-code entry and only enter class/number.
- Removed the technical Firebase explanation from the home screen.
- Verified with an automated Chrome E2E test: QR URL included the room code, room-link entry hid the code field, guesser UI hid extra panels, drawer banner was visible, and drawer UI did not show the answer panel.

## 2026-07-09 Answer Area Layout Correction

- Guessing students now see the answer input as a dedicated bottom section below the drawing area, not as a floating overlay on top of the canvas.
- The canvas height is constrained so the answer field always stays inside the viewport in portrait and landscape.
- The answer input is larger and visually dominant, with the send button aligned on the same row.
- Verified with an automated Chrome E2E test: portrait and landscape layouts kept the answer section below the canvas, visible inside the viewport, and focused after submit.

## 2026-07-09 Olive Game Board Skin

- The game screen now uses a reference-style olive status board, warm cream panels, brown canvas frame, and muted classroom-game palette.
- Teacher game layout now uses a full-width top status board above the drawing and answer panels.
- Student game layout keeps the top status board visible, with drawing in the middle and the answer input as a dedicated bottom section.
- Student landscape mode hides the redundant drawing title for guessers so the canvas and answer field both fit comfortably.
- Verified with an automated Chrome E2E test: student portrait/landscape layouts kept status, canvas, and answer input inside the viewport; teacher status bar spanned the board; the answer input stayed non-floating and focused after submit.

## 2026-07-09 UI Test Mode

- Added a `화면 테스트` button on the home screen.
- Test mode creates a local-only sample room with code `TEST`, sample students, a sample drawing, answer log entries, and rank-based scores.
- The fixed test switcher can preview `교사`, `그리는 학생`, `맞히는 학생`, and `운영창` views without touching Firebase.
- Test mode bypasses Firebase writes, Firebase room watches, and remote transactions; all test actions stay in local storage.
- The answer input now explicitly submits on Enter so keyboard entry behaves like the send button.
- Admin view now stays on the admin screen when a correct-answer countdown auto-advances to the next round.
- Verified with automated in-app browser tests: teacher view showed the prompt and drawer, drawer view showed tools and hid the answer form, guesser view submitted `바나나` by Enter for 90 points, admin view stayed open after the 5-second auto-advance, reset restored the sample room, and exit returned to home.

## 2026-07-09 PC Layout Correction

- Desktop game layout now keeps the drawing canvas at the original 14:9 ratio instead of stretching across very wide PC screens.
- The teacher status board is more compact on desktop: helper text is hidden, cards and timer are tightened, and controls fit in one row.
- UI test mode no longer consumes a separate top strip on desktop; the switcher is compact and overlays only the status header area.
- Verified at 1920x900 and 1366x768: no horizontal/vertical page overflow, canvas ratio stayed at 1.556, and the canvas was no longer cut off at the bottom.
