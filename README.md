# MediGuide

`index.html` 하나로 구성된 클라이언트 전용 간호/의학 약어 퀴즈 앱입니다. 비밀번호 입력 화면을 거친 뒤 퀴즈를 풀고, 마지막에 점수와 오답 노트를 보여주는 구조입니다.

## 파일 구성

- `index.html`: HTML, CSS, JavaScript가 모두 포함된 단일 페이지 앱
- 외부 의존성: `crypto-js` CDN이 선언되어 있으나 현재 코드에서는 사용되지 않음

## 화면 구조

`index.html`에는 세 개의 주요 섹션이 있습니다.

- `auth-section`: 비밀번호 입력 화면
- `quiz-section`: 문제, 보기, 피드백, 다음 문제 버튼 표시
- `result-section`: 최종 점수와 오답 노트 표시

CSS는 `<style>` 태그 안에 내장되어 있으며, 중앙 카드형 레이아웃과 정답/오답 버튼 색상을 정의합니다.

## JavaScript 흐름

주요 데이터와 함수는 다음과 같습니다.

- `quizData`: 퀴즈 문항 배열
- `currentIdx`: 현재 문제 번호
- `score`: 정답 수
- `wrongAnswers`: 오답 기록
- `savedPw`: `localStorage`에 저장된 비밀번호

동작 순서:

1. 사용자가 비밀번호를 입력하고 `checkAuth()`를 실행합니다.
2. 인증 조건을 통과하면 `auth-section`을 숨기고 `quiz-section`을 표시합니다.
3. `initQuiz()`가 상태를 초기화한 뒤 `loadNextQuestion()`으로 첫 문제를 표시합니다.
4. 사용자가 보기를 선택하면 `checkAnswer()`가 정답 여부를 판정합니다.
5. 모든 문제를 풀면 `showResults()`가 결과 화면과 오답 노트를 렌더링합니다.

## 현재 확인된 문제

### 1. 한글 인코딩 깨짐

파일에는 `meta charset="UTF-8"`이 선언되어 있지만, 제목, 문항, 안내 문구 등 대부분의 한글이 깨져 있습니다.

예:

```html
<title>媛꾪샇?숆낵 ?섑븰?⑹뼱 ?댁쫰 (?쒗솚湲곌퀎)</title>
```

파일 저장 인코딩 또는 이전 변환 과정에서 한글이 잘못 해석된 것으로 보입니다. 기능 개선보다 먼저 원문 한글 복구가 필요합니다.

### 2. 사용되지 않는 CryptoJS 의존성

다음 CDN 스크립트가 선언되어 있습니다.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
```

하지만 현재 코드에서 `CryptoJS`는 사용되지 않습니다. 제거해도 현재 기능에는 영향이 없습니다.

### 3. 비밀번호 보호 수준이 낮음

비밀번호는 `localStorage`의 `quiz_pw` 키에 저장됩니다.

```js
let savedPw = localStorage.getItem('quiz_pw') || "";
```

이 방식은 브라우저 개발자 도구에서 쉽게 확인하거나 삭제할 수 있으므로 실제 보안 기능으로 보기 어렵습니다. 현재 구조에서는 간단한 진입 제한 정도로 이해해야 합니다.

### 4. 저장된 비밀번호가 없으면 빈 입력으로도 입장 가능

인증 조건은 다음과 같습니다.

```js
if (savedPw === "" || inputPw === savedPw) {
```

따라서 저장된 비밀번호가 없으면 사용자가 아무 값도 입력하지 않아도 퀴즈 화면으로 이동할 수 있습니다. 안내 문구상 의도일 수 있지만, 비밀번호 입력을 필수로 만들려면 조건을 수정해야 합니다.

### 5. `innerHTML` 사용

피드백과 오답 노트 렌더링에 `innerHTML`이 사용됩니다.

현재는 퀴즈 데이터가 코드 내부 상수이므로 위험이 제한적입니다. 하지만 나중에 문제 데이터를 외부 JSON이나 API에서 받게 되면 XSS 위험이 생길 수 있습니다.

### 6. UX 개선 여지

- 문제 순서가 항상 동일합니다.
- 선택 후 반드시 "다음 문제" 버튼을 눌러야 진행됩니다.
- 다시 시작 버튼이 `location.reload()`로 전체 페이지를 새로고침합니다.
- 긴 문항이나 해설이 모바일 화면에서 넘칠 가능성이 있습니다.

## 개선 우선순위

1. 깨진 한글 문구와 퀴즈 데이터 복구
2. 사용하지 않는 CryptoJS CDN 제거
3. 비밀번호 정책 명확화
4. `innerHTML` 기반 렌더링을 DOM API 기반으로 정리
5. 퀴즈 데이터와 앱 로직 분리
6. 문제 랜덤화, 진행률 표시 개선, 다시 시작 로직 개선

## GitHub Pages 배포

`.github/workflows/pages.yml` 워크플로가 `master` 브랜치에 push될 때 정적 사이트를 GitHub Pages로 배포합니다.

GitHub 저장소 설정에서 `Settings > Pages > Build and deployment > Source`를 `GitHub Actions`로 설정해야 합니다.

배포 후 주요 페이지:

- `/index.html`: 현재 퀴즈 앱
- `/plans.html`: 버전별 로드맵과 무료/유료 등급 안내
- `/MARKETING.md`: 글로벌 마케팅 및 수익화 계획
