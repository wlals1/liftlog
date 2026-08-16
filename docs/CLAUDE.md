# LIFT LOG

개인 근력운동 기록 웹앱. GitHub Pages(`/docs`)로 배포되는 빌드 없는 정적 앱.

## 절대 지킬 것
- 앱 전체가 `docs/index.html` **한 파일**이다. 빌드 도구·npm·번들러를 도입하지 말 것.
- `docs/support.js` 는 런타임이다. 수정 금지.
- **스타일은 인라인만.** CSS 클래스/스타일시트를 만들지 말 것.
- 템플릿 `{{ }}` 안에 JS 표현식 금지. 계산은 `renderVals()` 에서 하고 이름으로 노출한다.
- 반복은 `<sc-for list="{{ x }}" as="item">`, 조건은 `<sc-if value="{{ flag }}">`.
- 저장 스키마를 바꾸면 `store()` 의 `v` 를 올리고 `componentDidMount` 에 마이그레이션을 추가한다.

## 구조
`docs/index.html` 안에 `<x-dc>` 마크업 + `class Component extends DCLogic` 로직.
React 클래스 컴포넌트와 동일하되 `render()` 대신 `renderVals()` 가 템플릿 값을 리턴.

## 도메인 규칙
- 종목 사전 `this.EX`, 프로그램 `this.PROG` (4분할: 상체A/하체A/상체B/하체B).
- `per: '양쪽'` 종목은 입력 무게가 **한쪽 기준**, 볼륨은 `mult`(=2) 배.
- `base: 0` 은 무게 미기록 종목 → '맨몸' 표시, 볼륨 제외.
- 무게는 `snap(v, e)` 로 `base` 기준 `st` 배수에 맞춘다 (머신 스택 간격).
- 주 단위 집계는 `weeksAgo(d, today)`, **월요일 시작**. 수정 시 주간 통계 전체 영향.
- 데이터는 localStorage `liftlog.v1` + GitHub(`liftlog-data/liftlog.json`) 동기화.

## 하면 안 되는 약속
아이폰 건강 앱 자동 연동은 웹앱에서 **불가능**하다(HealthKit 미개방). 유산소는 수동 입력이다.

## 로컬 실행
`python3 -m http.server 8000` → http://localhost:8000/docs/
(`file://` 로 열면 Claude API 호출이 CORS로 막힘)
