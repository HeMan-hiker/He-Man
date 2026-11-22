# LiveTrack 고정 링크 + 응원 메시지 (음성·텍스트)

`https://heman-hiker.github.io/He-Man/live/` — 가민 LiveTrack 실시간 위치 + 관전자 응원 메시지.

## 화면 구성
1. **라이브 상태 카드** — `LIVE · 진행 중` 배지, 시작/종료 시각, 큰 **실시간 위치 보기 (Garmin LiveTrack)** 버튼, 페이지 링크 복사.
2. **응원 보내기 카드** — 보낸 사람 이름 + 탭 2개
   - `🎙️ 음성 응원 (최대 15초)` — 마이크 녹음 → 미리 듣기/다시 녹음 → 전송
   - `💬 텍스트 응원 (최대 200자)` — 전송
   - 보내면 ① 히맨 님 Google Drive에 영구 저장 ② 동시에 가민 LiveTrack 관전자 메시지 API로 전송 → 워치 스피커로 재생.
3. **받은 응원 아카이브** — 드라이브에 저장된 최근 300건을 **활동(세션)별로 묶어** 표시.
   - 상단 칩 필터: `전체` + 활동별 칩(`11.22 · 활동명  N개`) — 활동 하나만 골라 그 활동에 온 응원만 모아 보기
   - 활동 헤더: 활동 날짜 · 활동 제목 · `음성 N / 텍스트 M` · 최근 시각 · `세션 열기`(가민 세션 페이지)
   - 라이브 밖에서 보낸 응원은 `일반 응원` 묶음으로 따로 표시

## 파일
- `index.html` : 페이지 본체 (모바일 우선, 다크 톤)
- `apps-script.txt` : Apps Script v2 원본 (붙여넣기용)
- `latest.json` : 페이지가 읽는 데이터 `{ url, updatedAt, subject, endpoint, key }`

## Apps Script v2 가 하는 일
- 1분마다 Gmail의 LiveTrack 메일에서 세션 링크 추출 (v1 그대로)
- `doGet` : 링크 + 라이브 세션 상태(가민 API) + 최근 응원 300건 JSON (JSONP 지원, `?audio=<fileId>` 재생, `?probe=1` 연결 점검)
  - 메시지마다 `sessionUrl` 을 함께 돌려주어 페이지가 활동별로 묶는다 (없는 예전 응답이어도 `세션 + 날짜`로 묶임)
- `doPost` : 응원 1건 수신 → Drive `Garmin LiveTrack 응원 메시지` 폴더 + `LiveTrack 응원 메시지 기록` 시트 저장 → 가민 Spectator API 전송
  - 시트 열: `시각(KST) | 세션 | 보낸사람 | 종류 | 내용 | 오디오파일 | 가민전송 | 세션URL`
    (앞 7칸은 v2 그대로라 기존 시트도 호환 — `setup` 을 다시 실행하면 8번째 칸만 덧붙습니다)
  - 같은 `msgId` 재전송은 자동으로 걸러냄(중복 방지)
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
