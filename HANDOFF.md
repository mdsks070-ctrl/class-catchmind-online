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
- 정답이 나오거나 시간이 끝나면 5초 후 자동으로 다음 단계로 이동
  - 마지막 라운드가 끝나면 최종 결과 상태로 전환
- 학생 참여 화면에 태블릿 가로 모드 안내 추가
- 학생 정답 입력에서 일부 부적절한 표현 차단
- 대기실에 참여 링크 표시
- 진행판에 `현재 라운드 / 전체 라운드` 표시
- 최종 결과 상태에서 1등과 점수를 보여주고, 대기실로 돌아가면 새 게임을 시작할 수 있도록 라운드/점수 초기화

확인한 것:

- 스크립트 문법 검사 통과
- 로컬 앱 초기 로드 콘솔 오류 없음
- 교사 설정 화면에서 라운드 수/효과음 옵션 노출 확인
- 방 생성 및 참여 링크 표시 확인
- 게임 시작 후 `1 / 5` 라운드 진행 표시 확인
- Firebase Realtime Database 생성 및 `/rooms` 규칙 게시 완료

검증한 것:

- HTML 내 `<script>` 문법 검사 통과
- 로컬 서버 `http://127.0.0.1:8765/class_catchmind_online.html` 로드 확인
- 교사용 방 생성 확인
- Firebase DB가 없을 때도 로컬 fallback으로 방 생성이 유지되는 것 확인

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
이 repo의 HANDOFF.md와 README.md를 먼저 읽고, class_catchmind_online.html의 Firebase Realtime Database 연결을 실제 기기 2대로 테스트해줘. 현재 DB는 생성됐고 `/rooms` 규칙은 수업 프로토타입용으로 열려 있으며, 목표는 반에서 여러 기기로 접속하는 캐치마인드식 그림 퀴즈야.
```
