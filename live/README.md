# LiveTrack 고정 링크

`https://heman-hiker.github.io/He-Man/live/` — 가민 LiveTrack으로 자동 연결되는 고정 주소.

- `index.html` : 페이지 본체 (모바일 우선, 다크 톤). 상태 4가지 = 확인 중 / 라이브 / 지난 세션 / 대기.
- `latest.json` : 페이지가 읽는 데이터. `{ url, updatedAt, subject, endpoint }`
  - `endpoint` 에 Apps Script 웹앱 URL을 넣으면 그쪽에서 실시간 값을 받아옵니다.
  - (또는 `index.html` 안의 `APPS_SCRIPT_URL` 상수에 직접 넣어도 됩니다.)
- 앱스 스크립트 원본: `attachments/2026-11-22/garmin-livetrack-apps-script.js`
- 설정 안내: `attachments/2026-11-22/garmin-livetrack-설정안내.md`

동작: 링크를 열면 → `livetrack.garmin.com` 주소를 찾고 → 24시간 이내면 2초 뒤 자동 이동,
아니면 버튼으로 직접 이동. 세션이 없으면 30초마다 다시 확인.

보안: `livetrack.garmin.com` / `connect.garmin.com`(livetrack 경로) 외의 주소로는 이동하지 않습니다.

옵션: `?maxage=72` — 24시간 대신 72시간까지 "라이브"로 인정(장거리 레이스용, 1~168).
`?auto=0` — 자동 이동 없이 버튼만 표시. `?endpoint=<URL>` — 그 주소에서 JSON을 읽음.
