# LiveTrack 고정 링크 + 응원 메시지 (음성·텍스트)

`https://heman-hiker.github.io/He-Man/live/` — 가민 LiveTrack 실시간 위치 + 관전자 응원 메시지.

## 화면 구성
- 상단 **☰ 메뉴** — `📡 현재 라이브 & 응원` / `📂 활동별 응원 아카이브`, 최근 활동 바로가기(누르면 그 활동의 응원이 바로 열립니다), `📺 유튜브 아카이브` · `📊 가민 대시보드` 링크.
- **라이브 뷰** (`#/live`, 기본)
  1. **라이브 상태 카드** — `LIVE · 진행 중` 배지, 시작/종료 시각, 큰 **실시간 위치 보기 (Garmin LiveTrack)** 버튼, 페이지 링크 복사.
  2. **응원 보내기 카드** — 보낸 사람 이름 + 탭 2개
     - `🎙️ 음성 응원 (최대 15초)` — 마이크 녹음 → 미리 듣기/다시 녹음 → 전송
     - `💬 텍스트 응원 (최대 200자)` — 전송
     - 보내면 ① 히맨 님 Google Drive에 영구 저장 ② 동시에 가민 LiveTrack 관전자 메시지 API로 전송 → 워치 스피커로 재생.
  3. **최근 받은 응원** — 최근 3개 미리보기 + 활동별 아카이브로 가는 버튼.
- **활동별 아카이브 뷰** (`#/archive`) — 유튜브 아카이브와 같은 톤의 **카드 그리드**
  - 상단 필터: 검색(활동명·보낸사람·내용) · 연도 select(2년 이상일 때) · 유형 칩(`전체 / 🎙️ 음성 포함 / 💬 텍스트만`)
  - 카드: 활동명(직접 붙인 대회명 우선) · 날짜 · `음성 N / 텍스트 M` · 최근 응원 미리보기 · 최근 시각 · `✏️ 이름 수정`
  - 카드를 누르면 **활동 상세 모달** — 그 활동의 응원 전체 + 음성 바로 재생 + `세션 열기`
  - 주소로 바로 열기: `#/archive/<세션키>` (모달까지 열림)
- **✏️ 이름 수정** — 관리자 비밀번호 + 새 활동명(실제 대회명) → Apps Script `doPost {action:'renameSession'}` → `ScriptProperties.sessionNames` 에 영구 저장 → `doGet` 응답의 `sessionNames` 로 모든 기기에서 그 이름으로 표시. 이름을 비우면 가민 기본 이름으로 돌아갑니다.

## 파일
- `index.html` : 페이지 본체 (모바일 우선, 다크 톤, 바닐라 JS)
- `apps-script.txt` : Apps Script v3 원본 (붙여넣기용) — `setupPin('비번')` 안내 포함
- `latest.json` : 페이지가 읽는 데이터 `{ url, updatedAt, subject, endpoint, key }`

## Apps Script v3 가 하는 일
- 1분마다 Gmail의 LiveTrack 메일에서 세션 링크 추출 (v1 그대로)
- `doGet` : 링크 + 라이브 세션 상태(가민 API) + 최근 응원 300건 + `sessionNames` + `features:['renameSession']` JSON (JSONP 지원, `?audio=<fileId>` 재생, `?probe=1` 연결 점검)
  - 메시지마다 `sessionUrl` 을 함께 돌려주어 페이지가 활동별로 묶는다 (없는 예전 응답이어도 `세션 + 날짜`로 묶임)
  - `sessionNames`/`features` 가 없는 **구버전 v2 응답도 그대로 동작** (이름 수정만 "스크립트 업데이트 필요" 안내)
- `doPost` : 두 갈래
  - `action: 'renameSession'` → 관리자 비밀번호 확인 후 활동 이름 저장 (`sessionNames`), 5분에 8회 실패 시 5분 잠금
  - 그 외 → 응원 1건 수신 → Drive `Garmin LiveTrack 응원 메시지` 폴더 + `LiveTrack 응원 메시지 기록` 시트 저장 → 가민 Spectator API 전송
  - 시트 열: `시각(KST) | 세션 | 보낸사람 | 종류 | 내용 | 오디오파일 | 가민전송 | 세션URL`
    (`세션` 칸에는 직접 붙인 대회명이 있으면 그 이름이 들어갑니다)
  - 같은 `msgId` 재전송은 자동으로 걸러냄(중복 방지)
- 관리자 비밀번호는 **공개 파일에 적지 않습니다** — `ScriptProperties.ADMIN_PIN`, 미설정 시 초기값 `0000` 만 허용하고 `setupPin('새비번')` 으로 바꾸도록 안내.
- 가민 API: 세션 페이지에서 `csrf-token` + `livetrack_csrf` 쿠키를 받아
  `POST /api/messages/spectator/{text|audio}?sessionId=..&token=..` (헤더 `Livetrack-Csrf-Token`, `Cookie`, `X-Garmin-LiveTrack-User-Guid`)

## 옵션
- `?maxage=72` — 24시간 대신 72시간까지 "라이브"로 인정 (1~168)
- `?auto=1` — 라이브일 때 자동으로 가민으로 이동 (기존 동작)
- `?endpoint=<URL>` — 그 주소에서 JSON을 읽음
- `?k=<문자열>` — `latest.json` 의 `"key"` 가 채워져 있을 때 초대 전용
- `?probe=1` — (웹앱 URL에 붙여서) 가민 API 연결 상태 점검

## 보안
- `livetrack.garmin.com` / `connect.garmin.com`(livetrack 경로) 외의 주소로는 이동하지 않습니다.
- 응원 전송 테스트는 **없는 세션 ID**로만 하며, 실제 시계로 아무것도 보내지 않습니다.
- 활동 이름 변경은 서버(Apps Script)에서 비밀번호를 확인한 뒤에만 저장됩니다. 비밀번호는 페이지·저장소에 남지 않습니다.
