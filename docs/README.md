# LIFT LOG

아이폰 홈 화면에 설치해서 쓰는 개인 근력운동 기록 웹앱입니다.

🔗 https://wlals1.github.io/liftlog/

## 주요 기능

### 오늘 — 운동 기록
- 세트별 무게·횟수 입력, 완료 체크, 세트 간 휴식 타이머
- 웜업 세트 추가, 세트 순서 변경·삭제
- 종목 추가·삭제, 4분할 프로그램(상체A · 하체A · 상체B · 하체B) 전환
- 날짜 칩으로 지난 기록 조회 및 수정

### 유산소
- 러닝·자전거·걷기 등 직접 입력 (시간 · 거리 · 심박수)
- 주간 누적 시간과 목표(150분) 대비 진행률

### 분석
- 일간 · 주간 · 월간 3단 통계
- 주간 볼륨, 세트 수, 부위별 배분
- 종목별 무게 증량 추이 스파크라인, 개인 기록(PR) 목록

### 몸
- 체중 · 골격근량 · 체지방률 기록 및 추이 차트

### AI
- Claude API로 최근 기록을 분석해 진단과 다음 주 계획을 리포트로 받아보고, 자유롭게 질문도 할 수 있습니다

## 기술 스택
- 빌드 도구·번들러·npm 의존성 없이 `docs/index.html` 한 파일로 동작하는 정적 웹앱
- 자체 경량 템플릿/컴포넌트 런타임(`docs/support.js`) 위에서 React 클래스 컴포넌트와 유사한 구조로 작성
- 데이터는 브라우저 localStorage에 저장되고, GitHub 저장소(`liftlog-data`)와 동기화 가능
- GitHub Pages로 배포 (`/docs` 디렉터리)

## 시작하기
```bash
git clone https://github.com/wlals1/liftlog.git
cd liftlog
python3 -m http.server 8000
# http://localhost:8000/docs/ 접속
```
> `file://`로 직접 열면 Claude API 호출이 CORS로 막히므로 반드시 로컬 서버로 실행하세요.

## 배포
`main` 브랜치에 푸시하면 GitHub Pages가 1~2분 내로 반영합니다.
```bash
git add . && git commit -m "..." && git push
```

## 저장소 구조
```
docs/
  index.html   # 앱 전체 (마크업 + 로직 + 스타일)
  support.js   # 템플릿/컴포넌트 런타임
```
