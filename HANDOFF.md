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

막힌 부분:

- Firebase Console에서 Realtime Database 생성을 시도했지만 콘솔 UI 오류가 발생했다.
- 시도한 지역:
  - 처음에는 한국과 가까운 `싱가포르(asia-southeast1)` 선택
  - 이후 기본 `미국(us-central1)`로도 재시도
- 두 번 모두 콘솔에서 “데이터베이스를 만드는 중에 오류가 발생했습니다. 다시 시도해 주세요.”가 떴다.
- 그래서 현재 코드는 Firebase 준비가 되어 있지만, 실제 여러 기기 동기화는 Realtime Database 생성이 성공해야 작동한다.

다음 Firebase 작업:

1. Firebase Console에서 `school-debate-1` 프로젝트 열기
2. Realtime Database 메뉴로 이동
3. 데이터베이스 만들기 재시도
4. 수업 테스트 단계에서는 테스트 모드로 시작 가능
5. 생성된 DB URL이 코드의 `databaseURL`과 다르면 `class_catchmind_online.html`에서 수정
6. 배포 전에는 테스트 모드 공개 규칙을 더 안전하게 바꾸기

주의:

- Firebase 웹 API key는 브라우저 앱 공개 설정값이라 서버 비밀번호는 아니다.
- 실제 보안은 Realtime Database Rules에서 해야 한다.
- 테스트 모드는 30일 제한이며 공개 읽기/쓰기가 될 수 있으므로 학생 개인정보는 넣지 않는 것이 좋다.

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

1. Firebase Realtime Database 생성 문제 해결
2. 교사 브라우저 1개 + 학생 브라우저/휴대폰 1개로 실제 multi-device 테스트
3. 방 데이터 구조가 안정적이면 Firebase Rules 정리
4. GitHub Pages, Netlify, Vercel, Firebase Hosting 중 하나로 공개 URL 배포
5. 수업용 UX 다듬기
   - 방 코드 크게 표시
   - 학생 입장 QR 코드
   - 정답자 효과
   - 라운드 기록
   - 단어 목록 프리셋
   - 그림판 선/지우개 안정화

## Codex에게 이어서 시킬 때

다음 프롬프트로 시작하면 된다.

```text
이 repo의 HANDOFF.md와 README.md를 먼저 읽고, class_catchmind_online.html의 Firebase Realtime Database 연결을 이어서 완성해줘. 현재 DB 생성이 Firebase Console 오류로 막혔고, 목표는 반에서 여러 기기로 접속하는 캐치마인드식 그림 퀴즈야.
```
