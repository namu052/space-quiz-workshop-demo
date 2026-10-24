# ORBIT 우주 탐사 퀴즈

## 실행

- 애플리케이션은 `index.html` 하나로 구성된다. 브라우저에서 파일을 직접 연다. macOS에서는 저장소 루트에서 `open index.html`을 사용한다.
- HTML, CSS, JavaScript, SVG를 이 파일 안에 유지한다. 서버, 프레임워크, 패키지 설치, CDN, 외부 폰트나 네트워크 요청 없이 `file://`에서 동작해야 한다.
- 코드 변경 후 브라우저에서 퀴즈 시작, 답 선택, 다음 문항 이동, 결과 화면, 다시 시작, 키보드 조작을 확인한다.

## 구조와 상태 흐름

- 문서 앞부분의 `<style>`이 테마와 레이아웃을 정의하고, HTML이 퀴즈·결과 화면의 틀을 제공한다. 문서 끝의 일반 `<script>`가 문항 데이터, 상태, DOM 렌더링을 담당한다.
- `questions`의 각 항목은 `category`, `question`, 4개 문자열의 `choices`, 0부터 시작하는 정답 인덱스 `answer`, `explanation`으로 구성된다. 원본 배열은 유지하며, 최초 진입과 재시작 때 Fisher–Yates 방식으로 `runQuestions`의 실행별 출제 순서를 만든다.
- `current`는 현재 실행 순서의 문항 인덱스, `score`는 맞힌 개수, `checked`는 현재 문항의 채점 여부다. `responses`는 제출한 선택지 인덱스를 `runQuestions` 순서대로 보관한다. 상태는 메모리에만 유지한다.
- `quiz-form`의 제출 이벤트는 `checkAnswer()` → 다음 `renderQuestion()` → 마지막 문항 이후 `showResults()`의 흐름을 제어한다. 선택 직후 자동으로 넘어가지 않으며, 첫 제출은 채점과 해설 표시, 두 번째 제출은 다음 화면 이동이다.
- `checkAnswer()`는 선택지를 비활성화하고 응답·점수를 한 번 반영한다. `updateProgress()`의 진행률은 현재 문항 번호가 아니라 **제출한 문항 수**다.
- `showResults()`는 `runQuestions`와 `responses`를 같은 인덱스로 조합해 실제 출제 순서의 전체 복습 목록을 만든다. 재시작은 새 출제 순서를 만들고 인덱스·점수·응답·진행률·복습 펼침 상태를 초기화한다.

## 변경 시 유지할 계약

- 10문항·4지선다·정답당 1점이 현재 퀴즈의 계약이다. 문항 수를 변경하는 요청에서는 배열뿐 아니라 HTML의 총 문항 수와 `progress.max`, 문항 번호 표시, 결과 분모, `score * 10` 정답률, 결과 등급 기준 및 안내 문구도 함께 맞춘다.
- DOM 조회는 `$` (`document.getElementById`)를 사용한다. HTML ID를 변경할 때 JavaScript 조회와 `aria-labelledby`·`aria-describedby` 참조도 함께 수정한다.
- 동적 내용은 `createElement`, `textContent`, `append`, `replaceChildren`으로 구성한다. 질문과 결과 화면 전환은 `hidden` 속성을 사용하며, `[hidden]` CSS 규칙을 유지한다.
- 한국어 질문·해설·피드백과 `lang="ko"`를 유지한다. 영어는 기존 브랜드 및 짧은 장식 문구에 사용한다.
- 선택지는 같은 `name="answer"`를 갖는 필수 네이티브 라디오 버튼으로 만들고 `label`·`fieldset`·`legend`로 묶는다. 미선택 제출은 브라우저의 필수 입력 검증을 사용한다. 방향키 선택과 폼 제출을 임의의 클릭 전용 컨트롤로 대체하지 않는다.
- 첫 렌더링은 `renderQuestion(false)`로 포커스를 가져가지 않는다. 다음 문항과 재시작은 질문 제목, 채점 후에는 액션 버튼, 결과 화면에서는 결과 제목으로 포커스를 옮긴다. 제목의 `tabindex="-1"`과 보이는 포커스 표시를 유지한다.
- 피드백 영역의 `role="status"`, `aria-live="polite"`, `aria-atomic="true"`를 유지한다. 정오답은 색상뿐 아니라 텍스트, 기호, 스크린리더용 설명으로도 구분한다.
- 색상은 `:root`의 의미별 CSS 변수와 `prefers-color-scheme: dark` 재정의를 따른다. 정답은 `.correct`의 녹색 강조, 오답은 `.wrong`의 빨간 흔들림으로 표현한다. 새 애니메이션도 `prefers-reduced-motion: reduce`에서 비활성화되어야 한다.
- `760px` 이하에서는 단일 열로 전환하고 장식용 우주 그림을 숨긴다. 시스템 폰트와 인라인 SVG를 사용한다.
