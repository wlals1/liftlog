# LIFT LOG — Claude Code 인수인계

## 이게 뭔가
아이폰 홈 화면에 설치해서 쓰는 개인 근력운동 기록 웹앱. 이미 **동작하는 앱**이고 GitHub Pages로 서비스 중입니다.

- 저장소: `wlals1/liftlog` (branch `main`, Pages 소스 = `/docs`)
- 라이브 주소: https://wlals1.github.io/liftlog/
- 데이터 저장소(별도, private): `wlals1/liftlog-data` 의 `liftlog.json`

## 저장소 구조
```
docs/
  index.html    ← 앱 전체 (마크업 + 로직 + 스타일 한 파일)
  support.js    ← 런타임 (수정하지 말 것)
```

빌드 없음, 번들러 없음, npm 없음. `docs/index.html` 하나만 고치고 커밋하면 배포됩니다.

## 파일 구조 (docs/index.html)
한 파일 안에 두 덩어리가 있습니다.

1. `<x-dc>` … `</x-dc>` — 마크업. **인라인 스타일만** 사용합니다 (CSS 클래스/스타일시트 없음).
   - `{{ name }}` 은 로직의 `renderVals()` 가 리턴한 값을 꽂는 자리입니다. **표현식은 못 씁니다** (`{{ a + b }}` 불가) — 계산은 전부 JS에서 하고 이름으로 노출하세요.
   - `<sc-for list="{{ items }}" as="item">`, `<sc-if value="{{ flag }}">` 로 반복/조건 처리.
   - 이벤트는 JSX식 카멜케이스: `onClick="{{ handler }}"`.
2. `class Component extends DCLogic { … }` — 로직. React 클래스 컴포넌트와 같습니다 (`state`, `setState`, `componentDidMount`). `render()` 대신 `renderVals()` 가 템플릿에 넘길 값을 리턴합니다.

**중요**: 이 구조는 Claude Code에서도 그대로 유지하는 게 가장 안전합니다. React/Vite로 갈아엎으면 빌드 파이프라인이 필요해지고 GitHub Pages 배포가 복잡해집니다. 기능 추가는 `EX`/`PROG` 테이블과 `renderVals()` 에 붙이는 식으로 하세요.

## 핵심 데이터 모델

### 종목 사전 `this.EX` (constructor)
```js
smithinc: { n: '스미스 인클라인 벤치 (30°)', g: '가슴', base: 15, gain: 0.26,
            st: 2.5, rep: 8, range: '6~10', rest: 120, per: '양쪽', mult: 2 }
```
- `n` 표시명 / `g` 부위 / `base` 시작 무게 / `st` 증량 단위(스택 간격)
- `rep` 기본 횟수 / `range` 목표 구간 / `rest` 휴식 초
- `per: '양쪽'` + `mult: 2` = 입력 무게는 **한쪽 기준**, 볼륨 계산 시 2배. 덤벨·핵스쿼트·스미스가 여기 해당.
- `base: 0` = 무게 미기록 종목(로프 푸시다운 등) → 볼륨에서 '맨몸' 처리.

### 프로그램 `this.PROG`
4분할: 상체 A / 하체 A / 상체 B / 하체 B. `ex: [[종목키, 세트수], …]`.

### 저장 데이터 (localStorage 키 `liftlog.v1`)
```js
{
  v: 2,
  progIdx: 0,                 // 현재 프로그램 인덱스
  session: [ { key, name, group, mult, sets: [{w, r, done, warm}] } ],  // 진행 중인 세션
  elapsed: 0,                 // 경과 초 (첫 세트 체크 후부터 증가)
  finished: false,
  saved: {                    // 완료된 기록. 키는 'M/D'
    '8/15': { date: ISO, prog: '하체 B', sub, mins, at: '오후 8:18',
              vol: 4625, nsets: 10,
              ex: [ { key, name, group, mult, sets: [{w, r}] } ] }
  },
  body:   [ { d: ISO, kg, sm, fat } ],          // 체중 / 골격근량 / 체지방률
  cardio: [ { d: ISO, type, mins, km, hr, kcal } ],
  apiKey: '',                 // Anthropic API 키 (이 기기에만)
  gh: { token, repo, path }   // GitHub 동기화 설정
}
```
`store()` 가 이 객체를 만들고, `componentDidMount` 가 읽습니다. **스키마를 바꾸면 `v` 를 올리고 `componentDidMount` 의 마이그레이션 분기를 추가하세요** (현재 v1 → v2 로직이 예시).

## 화면 (하단 탭 5개)
| 탭 | 하는 일 |
| --- | --- |
| 오늘 | 세트별 무게·횟수 입력, 체크, 휴식 타이머, 웜업 추가, 세트 순서변경/삭제, 종목 추가/삭제, 프로그램 전환, **날짜 칩으로 지난 날짜 입력·수정** |
| 유산소 | 러닝/자전거/걷기 직접 입력(분·km·bpm), 주간 합계·목표 150분 |
| 분석 | 일 / 주 / 월 3단. 주간 볼륨·세트·부위 배분, 월간 볼륨, 종목별 증량 추이 스파크라인, PR 목록 |
| 몸 | 체중·골격근량·체지방률 입력, 추이 차트, 측정 이력 |
| AI | Claude API 호출 → 진단·다음 주 계획 JSON 리포트 + 자유 질문. `context()` 가 보내는 요약을 만듭니다 |

## 알아둘 것 (지뢰)

1. **애플 건강 앱 자동 연동은 불가능합니다.** 웹앱에는 HealthKit이 열려 있지 않습니다. 진짜 연동을 원하면 네이티브(Swift) 앱이어야 합니다. 현재는 수동 입력으로 처리 중.
2. **주 계산은 `weeksAgo(d, today)`** — 월요일 시작. 여기 손대면 주간 집계 전체가 틀어집니다. (한 번 버그 났던 자리)
3. **볼륨 = `mult × Σ(무게 × 횟수)`.** `per: '양쪽'` 종목의 무게는 한쪽 값입니다.
4. **무게 스냅**: `snap(v, e)` 가 `base` 기준 `st` 배수로 맞춥니다. 머신 스택에 없는 숫자가 나오지 않게 하는 장치.
5. **인라인 스타일만.** CSS 클래스를 도입하면 렌더 방식이 깨집니다.
6. **`{{ }}` 안에 JS 금지.** 조건·계산은 `renderVals()` 에서.
7. `docs/support.js` 는 런타임입니다. 건드리지 마세요.

## 로컬에서 돌리기
```bash
cd ~/liftlog
python3 -m http.server 8000
# → http://localhost:8000/docs/
```
`file://` 로 직접 열면 Claude API 호출이 CORS로 막힙니다. 반드시 서버로 띄우세요.

## 배포
```bash
git add . && git commit -m "..." && git push
```
푸시하면 1~2분 뒤 https://wlals1.github.io/liftlog/ 에 반영됩니다.

## 다음에 할 만한 것
- 프로그램(EX/PROG) 자체를 앱 안에서 편집 — 지금은 코드 상수
- 월간 탭에 부위별 볼륨 누적
- 기록 CSV 내보내기
- 유산소를 애플 워치 → 단축어(Shortcuts) → GitHub API 로 밀어넣는 우회 자동화
